# ADP Pre-production deployment (`adp-preprod`)

This directory is an isolated pre-production starting point. Do not apply the current production manifests directly to `adp-preprod`: several of them contain production hostnames, production dependency endpoints, and credential-like values.

## Implemented in this branch

- `namespace.yaml` defines the dedicated `adp-preprod` namespace.

## Required before deployment

1. Create and approve a pre-production Azure DevOps variable group / Key Vault scope. Do not reuse `adp-secrets-prod` without explicit security approval.
2. Replace credential-like values in tracked Kubernetes files with placeholders and rotate any real credentials that were committed. Inject real values at deploy time from approved secret storage. Do not copy `k8s/backend/secrets.yaml` into this overlay.
3. Provision/approve isolated database, Redis, Elasticsearch indexes/cluster, Service Bus queues/topics/subscriptions, and Blob Storage container/prefix. Current `configmap-aks.yaml` points to production dependency endpoints.
4. Approve a temporary pre-production hostname and TLS certificate. Do not reuse or overwrite `www.globalautotrade.com`. Update frontend API/WebSocket URLs, CORS, cookie domain and auth callback URLs to match the approved hostname.
5. Create pre-production-specific Azure DevOps pipelines or templates. Do not change existing production pipeline definitions in place. They currently reference `refs/heads/prod`, production variable groups and production ACR/AKS resources. Confirm the intended source branch and target registry/service connection before enabling deployment.
6. Pin a unique immutable image tag/digest and verify the pipeline only applies to `adp-preprod`; add a guard that fails if the target namespace is `adp`.
7. Add and validate pre-production-specific Deployment/Service/ConfigMap/NetworkPolicy manifests and ingress rules. Do not apply the current `k8s/ingress.yaml` to this namespace because it uses the live production hostname.
8. Run YAML/schema validation, security/secret scanning, and review the diff before any AKS deployment.

## Safety status

This commit only adds the namespace manifest and this implementation checklist. It does not deploy anything to AKS and does not modify the existing `adp` namespace, production ingress, or production pipelines. Hostname, secret scope, database/dependency isolation, and target ACR/pipeline service connection are not known yet, so a complete deployable pre-production pipeline cannot safely be inferred from repository contents alone.
