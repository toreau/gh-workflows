# Changelog

## 2026-08-26 (initial library)

### Core
- Initial reusable workflow library, tag `v1`: `manifest-validate`, `dotnet-ci`, `container-build-push`, `container-merge-attest`, `dispatch`, `attestation-gate`, `digest-bump`, `native-pin-watcher`.
- First consumers: k8s-research (`validate.yml` via `manifest-validate`; `astro-digest-bump.yml` via `attestation-gate` + `digest-bump`). Astronomy migration planned (Fase 2).
