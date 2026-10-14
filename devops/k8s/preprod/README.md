# ADP pre-production deployment (`adp-preprod`)

Temporary hostname requested: `prod.globalautotrade.com`.

## Updated on branch `prod-v1`

- Namespace manifest and namespace fields in the sampled Kubernetes resources target `adp-preprod`.
- Frontend API/WebSocket URLs and ingress host rules use `prod.globalautotrade.com`.
- `secrets.template.yaml` contains placeholders only. Populate secrets from an approved preprod Key Vault/variable group.
- `inject-secrets-preprod.yaml` expects an Azure DevOps variable group named `adp-secrets-preprod`.

## Do not deploy until these are confirmed

1. DNS for `prod.globalautotrade.com` points to the intended ingress and the TLS certificate covers this hostname.
2. The preprod variable group/Key Vault contains only preprod credentials.
3. Database, Redis, Elasticsearch, Service Bus and Blob Storage are isolated from live Production. Current `configmap-aks.yaml` still contains production dependency endpoints.
4. Pipeline variable groups, ACR permissions, image tag/digest and service connection are reviewed. Several existing pipeline settings still identify Production resources.
5. Cookie domain, auth callback URLs, CORS, WebSocket routing, ingress and application settings are validated.
6. YAML/schema validation and secret scanning pass.

## Safety notes

The original `deploy/azurepipeline/inject-secrets.yaml`, `deploy/azurepipeline/rollback-prod.yml`, and `k8s/backend/secrets.yaml` remain production-oriented. Do not run/apply them for preprod. Service identifiers such as `adp-authentication` remain unchanged intentionally; they are service names, not namespace names.

No AKS deployment or DNS/TLS change has been performed by these Git commits.
