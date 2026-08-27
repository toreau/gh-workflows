# toreau/gh-workflows

Reusable GitHub Actions workflows shared across repos. Callers use these via
`uses: toreau/gh-workflows/.github/workflows/<name>.yml@v1`.

> **This repository must stay public.** Private reusable workflows cannot be
> used across repos under a personal account (same-owner private→private
> included). If it becomes private, every caller fails at workflow parse with
> «workflow was not found».

## Versioning & release

- Callers pin the **`v1`** tag — a **mutable ref** (a trusted, centrally
  administered channel, part of the trust model). Dependabot keeps caller refs
  fresh.
- To publish a change: open a PR → merge → fast-forward the `v1` tag to the
  new HEAD.
- New workflow files must sit at the repo root of `.github/workflows/`
  (subdirectories are unsupported by GitHub).

## Workflows

### Build & test

| Workflow | What it does | Key inputs / secrets |
| --- | --- | --- |
| `container-build-push.yml` | Builds and pushes a container image (buildx, platform e.g. arm64), docker login. Image is later SLSA-attested by the caller. | `image`, `dockerfile`, `platform`, `tags`, `runner` · secret `token` (packages:write) |
| `dotnet-ci.yml` | .NET restore/build/test with optional XPlat coverage + artifact upload. | `solution`, `coverage`, `upload-artifacts`, `global-json-file` |

### Promotion & gate

| Workflow | What it does | Key inputs / secrets |
| --- | --- | --- |
| `attestation-gate.yml` | Consumer-side attestation gate. Legacy mode = REST existence + predicate type. Strong mode = cryptographic `gh attestation verify --bundle-from-oci` with expected signer-workflow/source-ref (optional `--source-digest`). | `owner-repo`, `digest`, `image` (strong), `signer-workflow`, `source-ref`, `source-digest` · secrets `token`, `registry-token` |
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
- App repos (`toreau/astronomy.aursand.no`, `toreau/frosta-historielag.no`) —
  `ci.yml` (dotnet-ci / container-build-push + attest inline + dispatch).

## Adding a workflow

1. Add the file + header comment + full descriptions.
2. Run `actionlint`.
3. Open a PR, merge, fast-forward `v1`.
4. (Optional) wire a thin caller in a consuming repo and verify end-to-end.
