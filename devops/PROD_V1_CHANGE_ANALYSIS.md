# `prod-v1` branch — pre-production namespace change analysis

## Goal
Use Kubernetes namespace `adp-preprod` for isolated validation while preserving the existing live namespace `adp` and `www.globalautotrade.com` until an approved cutover.

## Repository findings checked

- `k8s/namespace.yaml` currently defines `adp`.
- `k8s/backend/configmap-aks.yaml` uses production profile and a production Elasticsearch endpoint; Redis and other dependencies need explicit isolation checks.
- `k8s/backend/secrets.yaml` contains credential-like values and a hard-coded default password. Treat them as exposed until verified; rotate any real credentials and remove secrets from version control/history through the approved process.
- `k8s/backend/configmap.yaml` contains development settings and a JWT-like value; it must not be treated as a production-ready preprod configuration.
- `k8s/frontend/frontend.yaml` targets namespace `adp`, uses `:latest`, and points API/WebSocket traffic at `www.globalautotrade.com`.
- `k8s/ingress.yaml` targets namespace `adp` and host `www.globalautotrade.com`; it must not be reused unchanged for preprod.
- `deploy/azurepipeline/inject-secrets.yaml` targets namespace `adp`, uses the production variable group/service connection and restarts live deployments. Do not run it for preprod.
- The sampled service pipelines (`pipeline-frontend.yaml`, `pipeline-actions.yaml`, `pipeline-chat.yaml`, `pipeline-orders.yaml`, `pipeline-analytics.yaml`) use `refs/heads/prod` and production variable groups/registry/cluster values. Create separate preprod pipeline definitions rather than editing these production pipelines in place.
- A deployment manifest named `k8s/backend/deployment.yaml` was not found at that path; inspect the actual per-service manifests before creating a complete overlay.

## Changes made in this branch

1. Added `k8s/preprod/namespace.yaml` defining namespace `adp-preprod` with environment labels.
2. Added `k8s/preprod/README.md` documenting deployment blockers, isolation requirements, and safe next steps.
3. Added this analysis document.

## Not changed intentionally

- Existing `k8s/namespace.yaml` and resources in namespace `adp`.
- Existing production ingress/hostname.
- Existing production pipeline definitions and variable groups.
- Secret-bearing manifests and production dependency endpoints.

## Outstanding inputs / blockers

- Approved preprod hostname and certificate/TLS ownership.
- Approved preprod variable group/Key Vault scope and Azure DevOps service connection.
- Whether preprod has its own ACR or may use the existing registry with separate immutable tags and permissions.
- Isolated database, Redis, Elasticsearch, Service Bus and Blob Storage resources/identities.
- Exact source branch policy for application repositories and confirmation that `prod-v1` is the approved source branch (branch and namespace are separate names).
- Full inventory of per-service Kubernetes manifests and pipeline files, followed by YAML validation, secret scanning, and a reviewed PR.

## Execution status

No AKS commands were run and no cluster resources were created. The `adp-preprod` namespace manifest is committed only to this Git branch; it must be reviewed and explicitly applied by an authorized operator after the blockers above are resolved.
