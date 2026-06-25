# Storage

This directory is reserved for storage configuration.

Common homelab options:

- `local-path`: simple default K3s storage, node-local only.
- Longhorn: replicated block storage across nodes.
- NFS provisioner: useful if you already have NAS storage.

TODO:

- Pick the default storage class.
- Add Longhorn or NFS manifests if needed.
- Decide backup strategy for Gitea repositories and databases.
