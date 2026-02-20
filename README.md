# Harbor

Helm wrapper around the **upstream Harbor** chart. Lab-sized setup: **existing PVC** `harbor-registry` (30Gi, pre-staged in Longhorn), external PostgreSQL and Redis, **Trivy** scanner enabled, **OIDC** via nginx (e.g. oauth2-proxy) in front of Harbor UI. **Artifact cleanup:** configure tag retention (e.g. retain 1 year, then delete older) in Harbor UI per project (Projects → Project → Tag Retention).

---

## Requirements

| Dependency | Notes |
|------------|--------|
| **PostgreSQL** | Database `harbor` and user; host `postgresql-rw.postgresql.svc.cluster.local`. Create Secret **harbor-db** (e.g. from 1Password). |
| **Redis** | Host `redis-master.redis.svc.cluster.local`. Create Secret **harbor-redis** if auth enabled. |
| **PVC** | Pre-staged **harbor-registry** (30Gi) in `homelab/helm/longhorn/values/volumes.yaml`; create namespace `harbor` before install. |
| **1Password** | For DockerHub: item with **username** and **password** (or token); sync to a Secret and use for Harbor registry replication. |
| **OIDC** | Use nginx ingress auth (e.g. oauth2-proxy with Okta) in front of Harbor so the web UI uses your IdP; or set Harbor auth mode to OIDC in values. |

---

## Tag retention (artifact cleanup)

In Harbor UI: **Projects** → select project → **Tag Retention**. Add a rule to retain tags for **1 year** (e.g. "retain most recently pulled for 365 days" or "retain by count" and run a schedule). Older artifacts are then cleaned automatically.

---

## DockerHub mapping

Create an ExternalSecret that fetches your DockerHub credentials from 1Password (e.g. item **dockerhub**) into a Secret (e.g. **harbor-dockerhub**). Configure Harbor replication or registry provider to use that Secret. See [Harbor replication](https://goharbor.io/docs/latest/administration/configuring-replication/) and the upstream chart values for `registry.registryConfig` or job-based replication.

---

## Render

> `helm dependency update . && helm template harbor . -f values.yaml -n harbor`

---

## Next steps

1. Ensure **harbor-registry** PVC exists (Longhorn chart with `values/volumes.yaml` creates it in namespace `harbor`).
2. Create database **harbor** and user on the shared PostgreSQL; add credentials to 1Password and ExternalSecret **harbor-db**.
3. Add DockerHub credentials to 1Password; add ExternalSecret template and wire Harbor to use the synced Secret.
4. Protect Harbor ingress with nginx OIDC (e.g. oauth2-proxy) so the web UI uses your IdP.
5. In Harbor UI, configure **Tag Retention** per project to auto-cleanup artifacts older than 1 year.
