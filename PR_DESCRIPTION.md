# Branch: `feature/harbor-chart-wrapper` — Harbor secrets from 1Password, pull-through cache ready

## Summary
- DB and Docker Hub credentials from 1Password via ExternalSecrets (no hardcoding).
- Correct upstream value structure (`database.type` + `database.external`, `redis.type` + `redis.external`) so DB name is **harbor**.
- Docker Hub secret as `dockerconfigjson` for imagePullSecrets and pull-through cache (configure in UI later).

## Changes
- **ExternalSecret harbor-db**: from 1Password item **postgresql** (password). Secret has `username` and `password`; Harbor gets both from secret (core.extraEnvVars for POSTGRESQL_USERNAME).
- **ExternalSecret dockerhub**: from 1Password item **dockerhub** (username + password) → `kubernetes.io/dockerconfigjson`.
- **values**: `database.type: external`, `database.external.coreDatabase: harbor`, `redis.type: external`, `redis.external.*`. Removed hardcoded DB username.
- **.gitignore**: `charts/`, `Chart.lock`.
- **README**: secrets summary, pull-through cache steps, `harbor.redis.external.existingSecret` path.

## 1Password
- **postgresql**: property **password** (same as Postgres cluster).
- **dockerhub**: properties **username** and **password**.

## Longhorn
- **harbor-registry** volume is now **ssd-nvme-gen3** (see longhorn repo PR).
