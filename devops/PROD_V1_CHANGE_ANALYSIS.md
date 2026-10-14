# `prod-v1` branch — pre-production namespace change analysis

## Goal
Use Kubernetes namespace `adp-preprod` and temporary hostname `prod.globalautotrade.com` for validation while preserving live namespace `adp` and live hostname `www.globalautotrade.com`.

## Changes made on this branch

- Updated `k8s/namespace.yaml` to define `adp-preprod`.
- Updated namespace fields in backend ConfigMaps, frontend Deployment/Service, ingress manifests and NetworkPolicies to `adp-preprod`.
- Updated frontend API/WebSocket URLs and ingress host rules to `prod.globalautotrade.com`.
- Updated the CORS origins in `configmap-aks.yaml` to `https://prod.globalautotrade.com`.
- Updated explicit namespace flags in the sampled service pipelines to `adp-preprod`.
- Added `k8s/preprod/secrets.template.yaml` with placeholders only.
- Added `deploy/azurepipeline/inject-secrets-preprod.yaml`, which expects a separate `adp-secrets-preprod` variable group.
- Added `k8s/preprod/README.md` with deployment prerequisites.

## Intentionally not globally renamed

Service names such as `adp-authentication`, `adp-frontend`, resource names, Azure variable group names, and application identifiers were not blindly renamed: these are not Kubernetes namespace values and may be referenced by application routing. Namespace references were changed instead.

The original `deploy/azurepipeline/inject-secrets.yaml` and `deploy/azurepipeline/rollback-prod.yml` remain production-oriented and must not be used for preprod. The original `k8s/backend/secrets.yaml` remains unchanged; it contains credential-like values and targets namespace `adp`. Do not apply it to preprod. Use the placeholder template and an approved preprod secret store.

## Remaining deployment blockers

- Confirm DNS record and TLS certificate coverage/ownership for `prod.globalautotrade.com`.
- Confirm that the `adp-secrets-preprod` Azure DevOps variable group exists and contains only preprod credentials.
- Current pipelines still contain production variable groups, ACR, and AKS service connection values unless separately overridden; verify access and approve a preprod-specific pipeline before running.
- `configmap-aks.yaml` still has production Elasticsearch/Redis endpoints and must be pointed at isolated resources before deploying. Database, Service Bus and Blob Storage isolation also needs confirmation.
- The frontend pipeline still uses the existing image tag strategy; pin an immutable preprod image tag/digest before deployment.
- Validate cookie domain, auth callback URLs, CORS, WebSocket routing, TLS, ingress controller behavior, YAML schemas and secret scanning.

## Execution status

Only GitHub branch files were changed. No AKS commands were run, no namespace was created in the cluster, and no DNS/TLS changes were made. Review the branch diff before running any pipeline.
