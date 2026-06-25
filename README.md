# Kairos Homelab GitOps

This repository is the GitOps source of truth for a Kairos/K3s homelab cluster.

It is structured around ArgoCD using an App-of-Apps pattern:

- `bootstrap/argocd/` installs ArgoCD and the root ArgoCD application.
- `clusters/homelab/` defines what belongs in the homelab cluster.
- `namespaces/` defines baseline namespaces.
- `infrastructure/` contains cluster-level services such as cert-manager, Gitea, ingress, storage, and secret management.
- `apps/internal/` contains internal-only workloads.
- `apps/external/` contains externally exposed workloads.

## Prerequisites

Use your Kairos kubeconfig when applying bootstrap manifests:

```bash
export KUBECONFIG=~/.kube/kairos-config
```

You also need:

- `kubectl`
- `kustomize` support, either standalone or via `kubectl apply -k`
- A Git remote for this repository

## Important TODOs before applying

Search for `TODO` and update values for your environment, especially:

- Git repository URL in ArgoCD `Application` manifests
- External domain names
- Internal domain names
- Ingress class names
- Storage class choices
- Gitea hostname and persistence settings

Useful search command:

```bash
grep -R "TODO" -n .
```

## Bootstrap ArgoCD

Install ArgoCD using server-side apply. This avoids oversized CRD annotation errors during bootstrap:

```bash
kubectl apply --server-side -k bootstrap/argocd/install
```

If you previously attempted a normal client-side apply and hit field-manager conflicts, run once with:

```bash
kubectl apply --server-side --force-conflicts -k bootstrap/argocd/install
```

Wait for ArgoCD to become ready:

```bash
kubectl -n argocd rollout status deployment/argocd-server
```

Apply the root app after updating its `repoURL`:

```bash
kubectl apply -f bootstrap/argocd/root-app.yaml
```

## Access ArgoCD

Port-forward ArgoCD locally:

```bash
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

Get the initial admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath='{.data.password}' | base64 -d && echo
```

Then open:

```text
https://localhost:8080
```

## Recommended rollout order

1. Bootstrap ArgoCD.
2. Sync namespaces.
3. Sync sealed-secrets or your preferred secret manager.
4. Sync cert-manager.
5. Confirm ingress strategy.
6. Deploy Gitea internally first.
7. Move this repository into Gitea.
8. Update ArgoCD `repoURL` values to point to Gitea.
9. Add internal apps.
10. Add external apps.

## Notes

The included `whoami` apps are starter examples for validating internal and external ingress paths. They are intentionally simple and should be adjusted or removed once your ingress strategy is finalized.
