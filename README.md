# gh-workflows

Reusable GitHub Actions workflows for toreau's projects.

**Plasseringsregel:** reusable workflows må ligge **direkte** i `.github/workflows/` — undermapper støttes ikke (GitHub-begrensning).

**Referanse fra kallere:**
`uses: toreau/gh-workflows/.github/workflows/<navn>.yml@v1`

Refs pinnes med tag (f.eks. `@v1`); dependabot (github-actions-økosystemet) holder refs oppdatert.

**Sikkerhet:** workflow-ene inneholder ingen hemmeligheter; kallere sender secrets som inputs.
