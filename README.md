# o269/ci-workflows — shared CI for the Oasis Studios + Omnia ecosystem

Reusable GitHub Actions workflows consumed by every repo across the
Omnia × Oasis Studios platform. Created 2026-05-15 per audit Phase 1 (closes
CRITICAL-7 — no CI anywhere across 26 repos).

## Why this exists

Before today, no repo in the ecosystem had CI. Type errors, missing imports,
and accidentally-committed secrets reached production silently. The memory
rule "always run `pnpm run build` before pushing" was a habit, not a gate.

This repo provides one workflow that every caller repo can adopt with a
10-line `.github/workflows/ci.yml`. Branch protection on `main` requires the
workflow to pass before merge.

## Usage

In each caller repo, add `.github/workflows/ci.yml`:

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    uses: o269/ci-workflows/.github/workflows/node-ci.yml@main
    with:
      workdir: '.'
      build-script: 'build'
    secrets: inherit
```

### Inputs

| Input | Default | Notes |
|---|---|---|
| `workdir` | `.` | Path to the package. Use `apps/api` for monorepo subpaths. |
| `node-version` | `22.x` | Phase 1 target. Lead-gen + Omnia Dockerfiles pin 22. |
| `pnpm-version` | `10` | Matches existing lockfiles. |
| `install-script` | `pnpm install --frozen-lockfile` | Override for non-pnpm. |
| `typecheck-script` | `typecheck` | Falls back to `pnpm exec tsc --noEmit` if not in package.json. |
| `lint-script` | `lint` | Set to `''` to skip. |
| `build-script` | `build` | Set to `''` to skip. |
| `test-script` | `''` (skipped) | Wire tests as part of Phase 2 coverage push. |
| `secret-scan` | `true` | Runs gitleaks against the diff. Findings surface as annotations; doesn't fail the build until Phase 2. |

## Adoption checklist

- [x] `oasis-command-center` (CRITICAL repo)
- [x] `bookkeeping` (CRITICAL repo — financial data)
- [x] `lead-gen-system` (CRITICAL repo — permit pipeline)
- [x] `omnia` (SaaS bet)
- [x] `chef` (PDF estimator)
- [ ] 6 marketing sites (oasis-roofs/remodels/adus/invests/solar/ai) — Phase 2 monorepo consolidation makes this trivial
- [ ] califirst-brands-repo — uses local `@repo/ui`, needs investigation first

## Branch protection

After CI is adopted by a repo, enable branch protection at GitHub Settings:

1. Branches → Add rule for `main`
2. Require status checks to pass: select `ci / ci` and `ci / secret-scan`
3. Require linear history
4. Restrict pushes that create new branches (no force-push to main)
5. Optional: require pull request before merging (recommended for shared repos)

## Roadmap

Phase 1 (now): typecheck + build + secret-scan annotations.
Phase 2 (Q3): hard-fail on gitleaks findings, add test gate, add Sentry release notification on main push.
Phase 3 (Q4): add AI eval gate (regress >10% blocks merge).
