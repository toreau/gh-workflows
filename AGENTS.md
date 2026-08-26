# AGENTS.md: gh-workflows

Reusable GitHub Actions workflow library for toreau's projects.

## Rules

- Reusable workflows live **directly** in `.github/workflows/` — subdirectories are NOT supported (GitHub limitation).
- Trigger is `on: workflow_call` only; declare typed `inputs`, named `secrets`, and explicit `permissions` (the caller must grant the same permissions).
- No repository secrets/credentials inside workflows — callers pass secrets as inputs.
- All third-party actions pinned by commit SHA (tag kept as `# vN` comment).
- Cross-repo callers reference `@v1`; dependabot (github-actions ecosystem) keeps refs fresh.
- `actionlint` must pass before merge; per-workflow branch + PR (`gh pr merge --squash`); after adding/changing workflows, fast-forward tag `v1`.

## Library

- `manifest-validate` — kubeconform + yamllint over whitespace-separated paths; optional custom CRD `schemas-dir` and yamllint-only paths.
- `dotnet-ci` — restore/build/test with optional XPlat code-coverage collection + upload.
- `container-build-push` — buildx + registry login + build-push for one platform; caller matrix picks the `runner` input (e.g. `ubuntu-24.04-arm` for arm64).
- `container-merge-attest` — merge multi-arch manifest, resolve digest (**output `digest`**), SBOM, attest provenance + SBOM, verify with `gh attestation`; requires `attestation-repo` + `sbom-image` inputs.
- `dispatch` — `repository_dispatch` with a JSON `client_payload`.
- `attestation-gate` — fail-closed "an attestation of a predicate type exists for this digest" check via the GitHub attestations REST API.
- `digest-bump` — sed digest bump in a file + bot commit/push; `commit-message` supports `{digest12}`/`{digest}`/`{sha7}`/`{sha}` tokens.
- `native-pin-watcher` — watch pinned native dependency versions (tag or commit) in a Dockerfile against upstream; opens a deduped issue when one moves.
