# GitOps manifests

Kustomize-based Kubernetes manifests for the flash cards app.

- `base/` — Deployments + Services for `backend` and `frontend`, plus a `PersistentVolumeClaim`
  backing the backend's SQLite database (card review progress), environment-agnostic.
- `overlays/hallen/` — the `hallen` environment: namespace, ingress, and image tags.

The backend Deployment uses `strategy: Recreate` since the SQLite file lives on a
`ReadWriteOnce` volume only one pod can mount at a time — the cluster must support
dynamic provisioning for the default `StorageClass`, or you'll need to set one explicitly
on the PVC.

## Before deploying

1. Push a `v*` tag (e.g. `v0.0.2`) in the `flash-cards-backend` and `flash-cards-frontend`
   repos to trigger their `.github/workflows/build-tagged-master.yml`, which builds and pushes
   `ghcr.io/<owner>/flash-cards-backend` and `-frontend` images tagged to match. Pushes to
   `master` publish `-development` images via `build-development.yml`, and both workflows
   dispatch `renovate.yml` in this repo to bump the image tags.
2. Build the frontend image **without** `VITE_API_BASE_URL` set. The ingress routes
   `/api` to the backend and `/` to the frontend on the same host, so the frontend's
   default (relative `/api` calls) is what production needs.
3. Update the `host` in `overlays/hallen/ingress.yaml` to the real hostname, and set
   `ingressClassName` to match your cluster's ingress controller (the `hallen` cluster
   only has `traefik`; check with `kubectl get ingressclass`).

## Deploy

```bash
kubectl apply -k overlays/hallen
```

Point your GitOps controller (Argo CD `Application` / Flux `Kustomization`) at
`gitops/overlays/hallen` to sync it automatically instead.
