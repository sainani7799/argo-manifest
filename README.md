# ArgoCD Manifest Repository

This repository contains Kubernetes manifests and Kustomization configurations for deploying applications using ArgoCD.

## Directory Structure

- **base/** - Base Kubernetes manifests for application-a
- **overlays/dev/** - Development environment overlay configuration
- **overlays/prod/** - Production environment overlay configuration
- **argocd-apps/** - ArgoCD Application manifests for deployment

## How to Update Image Tags

The image tags are managed through kustomization. To update the image tag for a specific environment:

### Update Dev Image Tag
\\\ash
cd overlays/dev
kustomize edit set image application-a=your-registry/application-a:dev-v1.2.3
\\\

### Update Prod Image Tag
\\\ash
cd overlays/prod
kustomize edit set image application-a=your-registry/application-a:v1.2.3
\\\

Or manually edit the \kustomization.yaml\ files in each overlay folder and update the \
ewTag\ field.

## Deploying with ArgoCD

1. Update your repo URL in ArgoCD applications (\rgocd-apps/application-a-*.yaml\)
2. Apply the ArgoCD applications:

\\\ash
kubectl apply -f argocd-apps/application-a-dev.yaml
kubectl apply -f argocd-apps/application-a-prod.yaml
\\\

Or use ArgoCD UI to sync the applications.

## Building and Pushing Images

From your application-a repository:

\\\ash
# Build image
docker build -t your-registry/application-a:dev-v1.2.3 .

# Push image
docker push your-registry/application-a:dev-v1.2.3

# Then update manifest repo with new tag
\\\

## Quick Commands

### View generated manifests for dev
\\\ash
kustomize build overlays/dev
\\\

### View generated manifests for prod
\\\ash
kustomize build overlays/prod
\\\

### Apply to cluster (with kubectl)
\\\ash
kubectl apply -k overlays/dev
kubectl apply -k overlays/prod
\\\

## Image Updates from Application Repo

When building images from your application-a repository:

1. Build the image with a tag: \docker build -t registry/app-a:TAG .\
2. Push the image: \docker push registry/app-a:TAG\
3. Update the manifest repo image tag:
   - Edit \overlays/dev/kustomization.yaml\ or \overlays/prod/kustomization.yaml\
   - Update the \
ewTag\ field under \images\
   - Commit and push changes
4. ArgoCD will sync automatically (if auto-sync is enabled)
