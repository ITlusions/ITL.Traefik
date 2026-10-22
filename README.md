# ITL.Traefik

[Traefik](https://github.com/traefik/traefik) ingress controller for the ITL Talos cluster ([`ITlusions/ITL.K8s`](https://github.com/ITlusions/ITL.K8s)) - a thin wrapper chart around the upstream `traefik/traefik` chart, same pattern as [`ITL.Cilium`](https://github.com/ITlusions/ITL.Cilium), [`ITL.Longhorn`](https://github.com/ITlusions/ITL.Longhorn), and [`ITL.K8s.MetricsServer`](https://github.com/ITlusions/ITL.K8s.MetricsServer).

On top of the upstream chart, this adds:
- **`templates/acl.yaml`** - IP allowlist middleware(s) for restricting access to internal-only routes
- **`templates/waf.yaml`** - an [OWASP ModSecurity CRS](https://github.com/coreruleset/coreruleset) WAF engine, fronted by the [traefik-modsecurity-plugin](https://github.com/acouvreur/traefik-modsecurity-plugin), registered via `traefik.experimental.plugins` in `values.yaml`

## Deploy

```bash
cd itl.traefik
helm dependency update
helm upgrade --install itl-traefik . \
  -n traefik \
  --wait --timeout 3m
```

## WAF caveat

The `waf.*` values deploy a single shared `modsecurity-crs` instance whose `backend` is a single hardcoded origin - see the comment above the `waf:` block in `values.yaml` before attaching the `waf` Middleware to more than one app's Ingress. The plugin also fully reverse-proxies approved traffic through this instance rather than inspecting-then-forwarding separately, and its Go HTTP client follows redirects by default - don't attach it to routes that issue HTTP redirects (login/logout/auth flows) without accounting for that.

## Verify

```bash
kubectl get pods -n traefik
kubectl get ingressclass
```
