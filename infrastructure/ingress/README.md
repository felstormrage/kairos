# Ingress

K3s commonly includes Traefik by default. This directory is reserved for ingress customization once the cluster strategy is confirmed.

Recommended pattern:

- Use one ingress class or entrypoint for external/public services.
- Use one ingress class or entrypoint for internal-only services.
- Keep internal apps on private DNS and firewall them from the internet.

Example host conventions:

- External: `app.example.com`
- Internal: `app.home.arpa` or `app.internal.example.com`

TODO:

- Confirm whether K3s Traefik is enabled.
- Decide whether to keep Traefik or install ingress-nginx.
- Define internal/external ingress classes or Traefik entrypoints.
