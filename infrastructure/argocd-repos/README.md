# ArgoCD repo credentials (sealed)

This directory holds the `SealedSecret` that produces the `Secret` ArgoCD uses
to authenticate to the in-cluster Gitea repository.

## What it does

The `SealedSecret` (`sealed-gitea-repo.yaml`) is decrypted by the
sealed-secrets controller into a plain `Secret` named `gitea-repo-creds` in the
`argocd` namespace. That `Secret` is labeled
`argocd.argoproj.io/secret-type: repository`, so ArgoCD picks it up as a repo
credential and uses it whenever an `Application`/`ApplicationSet` references a
`repoURL` matching the `url` field of the secret.

## Generating / rotating the sealed blob

See the comment block at the top of `sealed-gitea-repo.yaml` for the exact
`kubeseal` invocation. The short version:

1. Write a temporary plain `Secret` (with `stringData.url`, `username`,
   `password`) — **do not commit it**.
2. Pipe it through `kubeseal --format=yaml` (pointing at the
   `sealed-secrets` namespace / `sealed-secrets-controller` name) and write the
   output to `sealed-gitea-repo.yaml`.
3. Commit the resulting file. The PAT is never stored in Git.

## Required ordering (chicken-and-egg)

When you move this repo from GitHub to Gitea, ArgoCD must already *have* the
repo credential before the `repoURL` flips to Gitea, otherwise it can no longer
fetch the manifests that contain the credential.

Correct order:

1. Keep all `repoURL` values pointing at **GitHub** for now.
2. Deploy Gitea (infra-gitea Application) and create your PAT in Gitea.
3. Seal the PAT into `sealed-gitea-repo.yaml` and commit. Let the
   `infra-argocd-repos` Application sync it into the `argocd` namespace.
4. Verify ArgoCD registered the repo:
   `kubectl -n argocd get secret gitea-repo-creds` should exist, and
   `argocd repo list` (if you have the CLI) should show your Gitea URL as
   Connected.
5. Only now flip `repoURL` values (in `clusters/homelab/infrastructure.yaml`,
   `clusters/homelab/namespaces.yaml`, `clusters/homelab/apps-*.yaml`, and
   `bootstrap/argocd/root-app.yaml`) to the Gitea URL.

## Deploying the SealedSecret itself

The `Application` that syncs this path into the `argocd` namespace is defined
in `clusters/homelab/infrastructure.yaml` as `infra-argocd-repos`. There is no
nested `application.yaml` here because, unlike the other infra components,
there is no Helm chart — the `SealedSecret` manifest is applied directly.