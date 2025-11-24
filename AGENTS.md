# Repository Guidelines

## Project Structure & Modules
- `charts/dify`: main chart with `Chart.yaml`, `values.yaml`, and `templates/` for all components (api, web, worker, beat, proxy, sandbox, ssrf-proxy, plugin-daemon).
- `charts/dify/templates/tests`: Helm test hooks for Redis/Postgres connectivity; run after install when enabled.
- `charts/dify/charts`: bundled subcharts (e.g., Weaviate).
- `ci/values`: sample values for legacy, ExternalSecret, and other scenarios used in pipeline runs.
- `ci/scripts`: utilities for templating and validation (see `test-helm-template.sh`, `deploy-and-test.sh`).
- `ct.yaml`: chart-testing configuration; repo root contains top-level `README.md` and license.

## Build, Test, and Development Commands
- `helm dependency update charts/dify` — pull chart deps.
- `helm lint charts/dify` — quick lint of chart and values.
- `helm template <release> charts/dify --values ci/values/<file>.yaml --namespace <ns> --debug` — render manifests; mirrors `ci/scripts/test-helm-template.sh`.
- `helm install <release> charts/dify -f values.yaml --dry-run --debug` — simulate install with your overrides.
- `ct lint --config ct.yaml` — repository-wide chart lint (used in CI).
- Optional deep check: `bash ci/scripts/test-helm-template.sh <values-file>` validates rendered YAML with `kubectl --dry-run=client`.

## Coding Style & Naming
- YAML and Helm templates use 2-space indentation; keep resource names lowercase with hyphens (e.g., `api`, `plugin-daemon`, `ssrf-proxy`).
- Prefer helper templates in `_helpers.tpl`/`config.tpl`; keep new values under existing prefixes (`api.*`, `worker.*`, etc.) and document defaults in `values.yaml`.
- Be explicit over clever: readable conditionals, `default` pipes, and quoted strings where needed; avoid hidden side effects.

## Testing Guidelines
- Standard checks: `helm lint`, `helm template ... | kubectl apply --dry-run=client -f -` for generated files, and chart-testing lint.
- Helm tests in `charts/dify/templates/tests` run via `helm test <release>` after deployment when those hooks are enabled.
- Add or adjust sample `ci/values/*.yaml` when introducing new configuration paths; keep tests deterministic and minimal.

## Commit & Pull Request Guidelines
- Commit history follows conventional commits with scopes, e.g., `feat(chart): ...`, `build(dify): ...`, `chore(dify): ...`.
- For PRs: describe intent, values files tested, impact on defaults/dependencies, and link issues. Include results for `helm lint` and template validation; attach logs or screenshots if manifests change behavior.
- Bump `charts/dify/Chart.yaml` version when altering templates/values; update `README.md` if user-facing behavior changes.

## Security & Configuration Tips
- Never commit secrets; prefer ExternalSecret configuration (`externalSecret.enabled`) or existing secretRefs. Validate vault paths with `ci/scripts/validate-vault-paths.sh` when relevant.
- Keep TLS/ingress settings explicit; sanitize public endpoints in `values.yaml`. Avoid widening service ports or RBAC beyond necessity.
