# ArgoCD Manifest Repository - Gatex Backend Service

This repository contains Kubernetes manifests and Kustomization configurations for deploying **gatex-be-service** using ArgoCD in the **gtx-dev** namespace.

## Directory Structure

\\\
base/gatex-be-service/
├── namespace.yaml              # gtx-dev namespace
├── kustomization.yaml          # Base kustomization
└── gatex-be/
    ├── deployment.yaml
    └── service.yaml (NodePort: 30081)

overlays/dev/gatex-be-service/
├── kustomization.yaml          # Dev overlay with patches & image tag management
├── config.yaml                 # ConfigMap with environment variables
└── secrets.yaml                # Secret with credentials
\\\

## Configuration

### Base Configuration
- **Namespace**: gtx-dev
- **Image Registry**: ghcr.io/schemax-pte-ltd/gatex/gatex-be
- **Default Image Tag**: 1.9.0-dev
- **Service Type**: NodePort (Port 30081)
- **Container Port**: 8080

### Dev Environment
The dev overlay includes:
- **ConfigMap** (\gatex-be-service-config\): Environment variables like DATABASE_HOST, REDIS_HOST, LOG_LEVEL, etc.
- **Secret** (\gatex-be-service-env\): Sensitive data like DATABASE_PASSWORD, API_KEY, JWT_SECRET
- **Patches**: Inject config and secrets as envFrom in the deployment
- **Kubernetes Metadata**: Automatic pod/node/namespace name injection

## Updating Image Tag

To update the image tag for dev environment:

\\\ash
cd overlays/dev/gatex-be-service
kustomize edit set image ghcr.io/schemax-pte-ltd/gatex/gatex-be=ghcr.io/schemax-pte-ltd/gatex/gatex-be:1.10.0-dev
\\\

Or manually edit \overlays/dev/gatex-be-service/kustomization.yaml\ and update the \
ewTag\ field.

## ConfigMap and Secrets Management

### ConfigMap (config.yaml)
Contains non-sensitive environment variables:
- ENVIRONMENT, LOG_LEVEL, API_PORT
- DATABASE_HOST, DATABASE_PORT, DATABASE_NAME
- REDIS_HOST, REDIS_PORT
- Service discovery and monitoring settings

### Secret (secrets.yaml)
Contains sensitive data:
- Database credentials (DATABASE_USER, DATABASE_PASSWORD)
- API keys and tokens (API_KEY, JWT_SECRET)
- Webhook secrets

**⚠️ Important**: For production use, implement proper secret management:
- Use \itnami/sealed-secrets\ for encrypting secrets at rest
- Use HashiCorp Vault with ArgoCD plugin
- Use External Secrets Operator (ESO)
- Never commit plaintext secrets to git

## Deploying with ArgoCD

1. Update repository URL in \rgocd-apps/argo-gatex-dev.yaml\
2. Apply the ArgoCD Application:

\\\ash
kubectl apply -f argocd-apps/argo-gatex-dev.yaml
\\\

Or via ArgoCD UI:
- Add new application
- Set repo URL to your manifest repository
- Set path to \overlays/dev/gatex-be-service\
- Set destination namespace to \gtx-dev\

## Quick Commands

### View generated manifests
\\\ash
kustomize build overlays/dev/gatex-be-service
\\\

### Apply manifests directly (without ArgoCD)
\\\ash
kubectl apply -k overlays/dev/gatex-be-service
\\\

### Verify deployment
\\\ash
kubectl get deployment -n gtx-dev
kubectl get configmap -n gtx-dev
kubectl get secret -n gtx-dev
kubectl get service -n gtx-dev
\\\

### View service endpoint
\\\ash
kubectl get service gatex-be-service -n gtx-dev
# Access at: http://<node-ip>:30081
\\\

## ArgoCD Application File

See \rgocd-apps/argo-gatex-dev.yaml\ for the ArgoCD Application definition.

The application is configured with:
- Auto-sync enabled (prune and selfHeal)
- Automatic namespace creation
- Foreground deletion for clean resource cleanup
