# Harbor

Helm wrapper around the **upstream Harbor** chart. Lab-sized setup: **existing PVC** `harbor-registry` (30Gi, pre-staged in Longhorn), external PostgreSQL and Redis. **Volumes are defined in Longhorn** (e.g. `harbor-registry` in Longhorn values). **Trivy** scanner enabled, **OIDC** via nginx (e.g. oauth2-proxy) in front of Harbor UI. **Artifact cleanup:** configure tag retention (e.g. retain 1 year, then delete older) in Harbor UI per project (Projects → Project → Tag Retention).

## Argo CD

Deploy via Argo CD. Example Application (adjust repo/path/namespace):

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: harbor
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/jd4883/harbor
    path: .
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: harbor
  syncPolicy:
    automated: { prune: true, selfHeal: true }
```

---

## Requirements

| Dependency | Notes |
|------------|--------|
| **PostgreSQL** | Same cluster as Nextcloud/Immich: host `postgresql-rw.postgresql.svc.cluster.local`, database **harbor**. Secret **harbor-db** is created from 1Password item **postgresql**; both username and password come from the secret (no hardcoding). |
| **Redis** | Host `redis-master.redis.svc.cluster.local`. Secret **harbor-redis** only if your Redis has auth: create a Secret with key **REDIS_PASSWORD** and set `harbor.redis.external.existingSecret: harbor-redis`. Leave empty for no-auth Redis. |
| **PVC** | Pre-staged **harbor-registry** (30Gi, **ssd-nvme-gen3**) in `homelab/helm/longhorn/values/volumes.yaml`; create namespace `harbor` before install. |
| **Docker Hub** | Secret **dockerhub** is created by this chart from 1Password item **dockerhub** (username + password) via ExternalSecret (type `kubernetes.io/dockerconfigjson`). Used for `imagePullSecrets` and for **pull-through cache** (configure in Harbor UI later). |
| **OIDC** | Use nginx ingress auth (e.g. oauth2-proxy with Okta) in front of Harbor so the web UI uses your IdP; or set Harbor auth mode to OIDC in values. |

---

## Tag retention (artifact cleanup)

In Harbor UI: **Projects** → select project → **Tag Retention**. Add a rule to retain tags for **1 year** (e.g. "retain most recently pulled for 365 days" or "retain by count" and run a schedule). Older artifacts are then cleaned automatically.

---

## Secrets summary

| Secret | Source | Notes |
|--------|--------|------|
| **harbor-db** | ExternalSecret from 1Password item **postgresql** (property **password**). Secret has keys **username** and **password**; both are used by Harbor (no hardcoded credentials). Same 1Password item as Postgres cluster and Immich. |
| **dockerhub** | ExternalSecret from 1Password item **dockerhub** (properties **username** and **password**). Creates a `kubernetes.io/dockerconfigjson` secret. Used for `imagePullSecrets` and for **Docker Hub pull-through cache** (configure in Harbor UI: Registry → New Endpoint → Docker Hub, then Project → New Project → Proxy Cache). |
| **harbor-redis** | Only if Redis has auth. Create a Secret with key **REDIS_PASSWORD**; set `harbor.redis.external.existingSecret: harbor-redis`. Omit for no-auth Redis. |

**1Password items:** **postgresql** (password only; username is set to `postgres` in the secret by the chart). **dockerhub** (username + password). On the Postgres server, create database **harbor** (`CREATE DATABASE harbor;`); the **postgres** user already exists.

---

## Render

> `helm dependency update . && helm template harbor . -f values.yaml -n harbor`

---

## Next steps

1. Ensure **harbor-registry** PVC exists (Longhorn chart with `values/volumes.yaml` creates it in namespace `harbor`).
2. On the shared PostgreSQL, create database **harbor** (`CREATE DATABASE harbor;`). The chart creates **harbor-db** from the same 1Password item as the Postgres cluster (**postgresql**).
3. Add 1Password item **dockerhub** with **username** and **password**; the chart creates Secret **dockerhub** via ExternalSecret. Use it in Harbor UI for Docker Hub pull-through cache when you configure it.
4. If your Redis has auth, create Secret **harbor-redis** with key **REDIS_PASSWORD** and set `harbor.redis.external.existingSecret: harbor-redis`.
5. Protect Harbor ingress with nginx OIDC (e.g. oauth2-proxy) so the web UI uses your IdP.
6. In Harbor UI, configure **Tag Retention** per project to auto-cleanup artifacts older than 1 year.
