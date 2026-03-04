# Alacenas

Alacenas is a GitHub organization for collaborative software projects. This repo serves as the org's home base — a place to track standards, projects, and shared practices.

## Projects

| Repository | Description |
|---|---|
| [alacenas/.github](https://github.com/alacenas/.github) | Org home repo — standards, docs, and meta information |
| [alacenas/alacenas-lab](https://github.com/alacenas/alacenas-lab) | Experimental projects and sandbox work |

## Standards

### Code

- Prefer small, focused pull requests
- All PRs require at least one approving review before merge
- Write meaningful commit messages (imperative mood, e.g. "Add feature X")
- Keep main/default branches stable; use feature branches for development
- Repos use **squash merges only**; the squash commit title must equal the PR title

### PR Title Convention

All PR titles must follow the [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>[!]: <subject>
```

| Type | When to use | Version bump |
|---|---|---|
| `feat` | New user-facing feature | minor (`0.x.0`) / major after v1.0 |
| `fix` | Bug fix | patch (`0.0.x`) |
| `feat!` / `fix!` | Breaking change | minor pre-v1.0 → major (`X.0.0`) at v1.0+ |
| `docs` | Documentation only | no release |
| `chore` | Maintenance, deps, tooling | no release |
| `refactor` | Code restructure, no behavior change | no release |
| `test` | Adding or fixing tests | no release |
| `ci` | CI/CD configuration | no release |
| `build` | Build system changes | no release |
| `perf` | Performance improvement | no release |

Examples: `feat: add user login`, `fix: handle null session`, `chore: upgrade pnpm`

> The PR title becomes the squash-merge commit message, which is what `semantic-release` reads to determine the next version.

### Documentation

- Every repository should have a `README.md` covering purpose, setup, and usage
- Document non-obvious decisions inline or in a `docs/` folder

### Communication

- Open issues for bugs, feature requests, and discussions
- Use labels to categorize issues and PRs
- Reference related issues and PRs in commit messages and descriptions

## Reusable Workflows

All workflows live in `.github/workflows/` and are callable from any repo in the `alacenas` org.

### `pr-title-lint.yml` — PR Title Convention Check

Validates that the PR title follows the Conventional Commits format (required for automated versioning).

```yaml
# .github/workflows/pr.yml (in caller repo)
on:
  pull_request:
    types: [opened, edited, synchronize, reopened]

jobs:
  title:
    uses: alacenas/.github/.github/workflows/pr-title-lint.yml@main
    secrets: inherit
```

Required branch-protection check name: `pr / title`

---

### `node-pr-checks.yml` — Node.js PR Checks

Runs `lint`, `typecheck`, `test`, and `build` using **pnpm** and **Node 22 LTS**. Each step runs as a separate job so check names are stable for branch-protection rules.

```yaml
# .github/workflows/pr.yml (in caller repo)
on:
  pull_request:

jobs:
  checks:
    uses: alacenas/.github/.github/workflows/node-pr-checks.yml@main
    with:
      node-version: "22"      # optional
      pnpm-version: "9"       # optional
      working-directory: "."  # optional
```

Required branch-protection check names: `pr / lint`, `pr / typecheck`, `pr / test`, `pr / build`

---

### `semantic-release.yml` — Automated Versioning & GitHub Releases

Creates `vX.Y.Z` git tags and GitHub Releases on merge to `main` by reading squash-commit messages.

```yaml
# .github/workflows/release.yml (in caller repo)
on:
  push:
    branches: [main]

jobs:
  release:
    uses: alacenas/.github/.github/workflows/semantic-release.yml@main
    secrets:
      release-token: ${{ secrets.GITHUB_TOKEN }}
    permissions:
      contents: write
      issues: write
      pull-requests: write
```

To customize release behavior, add a `.releaserc.json` in the caller repo.

---

### `publish-image.yml` — Push Docker Image to GHCR

Builds a Docker image and pushes it to the GitHub Container Registry on merge to `main`.

- Always pushes: `ghcr.io/<org>/<repo>:sha-<git-sha>` (immutable, safe for ECS rollbacks)
- Optionally pushes: `ghcr.io/<org>/<repo>:main` (floating tag for dev environments)

```yaml
# .github/workflows/publish.yml (in caller repo)
on:
  push:
    branches: [main]

jobs:
  publish:
    uses: alacenas/.github/.github/workflows/publish-image.yml@main
    with:
      push-main-tag: true     # optional, default true
    permissions:
      contents: read
      packages: write
```

Prerequisite: GHCR packages write permission must be enabled at the org level.

## Branch Protection Checklist

For each repo that uses the org workflows, configure the following in **Settings → Branches → Branch protection rules** for `main`:

- [x] Require a pull request before merging
- [x] Require at least 1 approving review
- [x] Require status checks to pass before merging
  - Add required checks: `pr / title`, `pr / lint`, `pr / typecheck`, `pr / test`, `pr / build`
- [x] Require branches to be up to date before merging
- [x] Do not allow bypassing the above settings
- [x] Allow only squash merging (set in repo Settings → General → Pull Requests)
- [x] Set default squash commit title to **Pull request title**

## Contributing

1. Fork or branch off of `main`
2. Make changes in a focused feature branch
3. Open a pull request with a clear description and a valid PR title (see convention above)
4. Address review feedback and merge once approved

## License

Unless otherwise noted, projects in this org are unlicensed. Check individual repositories for their specific license terms.
