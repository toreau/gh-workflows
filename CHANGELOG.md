# Changelog

## 2026-08-26 (initial library)

### Core
- Initial reusable workflow library, tag `v1`: `manifest-validate`, `dotnet-ci`, `container-build-push`, `container-merge-attest`, `dispatch`, `attestation-gate`, `digest-bump`, `native-pin-watcher`.
- First consumers: k8s-research (`validate.yml` via `manifest-validate`; `astro-digest-bump.yml` via `attestation-gate` + `digest-bump`). Astronomy migrated too (PR #6–#9): `dotnet-ci`, `container-build-push`, `container-merge-attest` + `dispatch`, `native-pin-watcher`.

### Bugs fixed (during astronomy migration, all in tag `v1`)
- `container-merge-attest`: multi-arch digest resolved from a two-line `$first` (`awk|cut` on multiline `merge-tags` → `invalid reference format`) → `xargs` collapse.
- `container-merge-attest`: `gh attestation verify` failed on reusable-workflow-signed attestations (`verifying with issuer "sigstore.dev"`, cli/cli#9045) → new `signer-workflow` input passed as `--signer-workflow` (default `@refs/tags/v1`).
- `dispatch`: `client_payload` sent as a string (HTTP 422 «not an object») → build JSON body with `jq`, pipe via `gh api --input -`.
- `native-pin-watcher`: `grep` parsed a `--`-prefixed pattern as an option (`--branch v2.0.1` → empty pin, silent in the pipeline) → `grep -e`. Latent bug inherited from the original astronomy `native-watcher`.
