# toreau/gh-workflows

Reusable GitHub Actions workflows shared across repos. Production callers
reference them by immutable full commit SHA:
`uses: toreau/gh-workflows/.github/workflows/<name>.yml@<full-commit-sha>`.

> **This repository must stay public.** Private reusable workflows cannot be
> used across repos under a personal account (same-owner private→private
> included). If it becomes private, every caller fails at workflow parse with
> «workflow was not found».

## Versioning & release

- Production callers reference reusable workflows by immutable full commit
  SHA. A workflow change is adopted by a caller through a reviewed SHA update.
- `v1` is a legacy/compatibility tag, not the production execution-trust
  mechanism. Moving tags may be used for compatibility or update discovery,
  but production callers execute reviewed full commit SHAs.
- Dependabot may propose updates to ordinary GitHub Actions/reusable-workflow
  SHA pins; a proposal is not approval. Security-decision refs such as the
  trusted builder and attestation gate are kept out of ordinary Dependabot
  adoption and change only through their explicit evidence/trust lifecycle.
- New workflow files must sit at the repo root of `.github/workflows/`
  (subdirectories are unsupported by GitHub).

## Workflows

### Build & test

| Workflow | What it does | Key inputs / secrets |
| --- | --- | --- |
| `container-build-attest.yml` | **Trusted builder** (generic): no caller inputs/secrets — build config is read from `.github/container-build.json` in the caller's exact source revision. Validates (main-only, ghcr.io + owner namespace, supported platform→runner), builds, derives the exact digest, generates provenance+SBOM attestations (SBOM in a separate non-signing job), returns only `digest`. Uses caller `github.token`. | no inputs, no secrets · caller ceiling: `contents:read, packages:write, id-token:write, attestations:write` |
| `dotnet-ci.yml` | .NET restore/build/test with optional XPlat coverage + artifact upload. | `solution`, `coverage`, `upload-artifacts`, `global-json-file` |

### Promotion & gate

| Workflow | What it does | Key inputs / secrets |
| --- | --- | --- |
| `attestation-gate.yml` | Consumer-side attestation gate. Legacy mode = REST existence + predicate type. Strong mode = cryptographic `gh attestation verify --bundle-from-oci` with expected signer-workflow/source-ref (optional `--source-digest`). Revision mode = strong mode + explicit trusted signer-digest set (JSON array; one complete PASS among N trusted revisions authorizes, zero = fail-closed, no path-only fallback). | `owner-repo`, `digest`, `image` (strong), `signer-workflow`, `source-ref`, `source-digest`, `trusted-signer-digests-json`, `require-signer-digest`, `deny-self-hosted-runners` · secrets `token`, `registry-token` |
| `container-scan.yml` | Final-container vulnerability gate: scans the exact remote GHCR `IMAGE@DIGEST` with a checksum-pinned Trivy v0.74.0 binary. Fixed policy owned here: vuln-only, severities `HIGH,CRITICAL`, unfixed included, `os,library`; single-image-manifest only (index/list fails closed); no source checkout. | `image`, `digest` · no secrets · caller ceiling: `packages:read` |
| `digest-bump.yml` | Bumps an image digest in manifests; commit/push, or open/update a PR (`open-pr: true`). Token substitution in branch/title/body. | `file-paths`, `image-prefix`, `digest`, `sha`, `open-pr`, `head-branch`, `pr-title` · secret `token` |
| `dispatch.yml` | Sends a `repository_dispatch` event; optional HMAC-v1 signing of the payload. | `target-repo`, `event-type`, `payload` · secrets `token`, `hmac-secret` |

### Validation

| Workflow | What it does | Key inputs / secrets |
| --- | --- | --- |
| `manifest-validate.yml` | kubeconform (optional custom CRD schemas + exclude regex) + yamllint (relaxed) over manifest paths. | `paths`, `schemas-dir`, `strict`, `exclude`, `yamllint-paths` |

### Maintenance

| Workflow | What it does | Key inputs / secrets |
| --- | --- | --- |
| `native-pin-watcher.yml` | Watches pinned native dependencies in a Dockerfile; opens an issue when an upstream moves. | `pins`, `dockerfile`, `issue-title`, `labels` |

## Requirements for new workflows

- File at the root of `.github/workflows/`, declared with `on: workflow_call`.
- **Header comment** at the top describing what the workflow does, which phase
  it belongs to, and who uses it.
- A `description` on **every** input and secret (shown in the Actions UI).
- **Caller token scope:** a reusable workflow that opens PRs needs
  `pull-requests: write` in the **caller's** `permissions:` block — the caller
  defines the GITHUB_TOKEN scope, and the library's own block cannot raise it.
- `actionlint` clean before merge.

## Caller examples

- `toreau/k8s-research` — thin callers in `.github/workflows/`:
  `app-digest-bump.yml` (attestation-gate → digest-bump), `validate.yml`
  (manifest-validate → gate-pr).
- App repos: thin `ci.yml` callers around the trusted
  `container-build-attest` workflow (`build-test -> builder -> dispatch`);
  reference implementation: `toreau/frosta-historielag.no`.

## Adding a workflow

1. Add the file + header comment + full descriptions.
2. Run `actionlint`.
3. Open a PR and merge through protected `main`.
4. When adopting the workflow, reference the reviewed merge/full commit SHA
   from the consuming repo and verify end-to-end where appropriate.
