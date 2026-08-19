# AdService Kubernetes Manifests

This directory contains Kubernetes manifests and GitOps configuration for the AdService microservice.

## Files

| File | Description |
|------|-------------|
| `adservice-deployment.yaml` | Kubernetes Deployment (image, ports, probes, resources) |
| `adservice-service.yaml` | ClusterIP Service exposing gRPC on port 9555 |
| `adservice-gitops.yaml` | ArgoCD Application manifest for GitOps-based deployment |

## Quick Deploy (manual)

```bash
kubectl apply -f k8s/adservice-deployment.yaml
kubectl apply -f k8s/adservice-service.yaml
kubectl rollout status deployment/adservice --timeout=120s
```

## GitOps Deploy (ArgoCD)

The CD pipeline (`cd.yaml`) automatically:

1. Creates a Kind cluster
2. Installs ArgoCD
3. Applies the ArgoCD Application from `adservice-gitops.yaml`
4. Syncs the application

ArgoCD watches the `k8s/` directory in this repo. Any change pushed to `main` is automatically synced to the cluster (self-heal + prune enabled).

### ArgoCD Application Details

- **Repo:** `https://github.com/lokeshzenbook-coder/Online-Boutique.git`
- **Path:** `k8s`
- **Target Revision:** `main`
- **Destination Namespace:** `default`
- **Sync Policy:** Automated with self-heal and prune

## Image

```
ghcr.io/lokeshzenbook-coder/online-boutique/adservice:main
```

Built and pushed by the CI pipeline (`adservice.yaml`).

## Verify

```bash
kubectl get pods -l app=adservice
kubectl get svc adservice
kubectl logs -l app=adservice
```
