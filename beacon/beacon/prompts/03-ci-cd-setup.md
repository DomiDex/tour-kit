# Prompt 03: CI/CD Setup

## Problem

Configure GitHub Actions workflows for CI (lint, test, build) and releases with Changesets.

## Dependencies on Previous Prompts

- Prompt 01 complete (monorepo exists)
- Prompt 02 complete (tooling configured)

## Supporting Information

### Target File Structure

```
tour-kit/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── release.yml
└── .changeset/
    └── config.json
```

## Steps To Complete

### Step 1: Create CI workflow

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    name: Build & Test
    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup pnpm
        uses: pnpm/action-setup@v3
        with:
          version: 9

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Lint
        run: pnpm lint

      - name: Type check
        run: pnpm typecheck

      - name: Build
        run: pnpm build

      - name: Test
        run: pnpm test

  size:
    name: Bundle Size
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name == 'pull_request'

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup pnpm
        uses: pnpm/action-setup@v3
        with:
          version: 9

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build
        run: pnpm build

      - name: Check bundle size
        uses: preactjs/compressed-size-action@v2
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          pattern: './packages/*/dist/**/*.js'
          exclude: '{**/*.map,**/node_modules/**}'
```

### Step 2: Create Release workflow

Create `.github/workflows/release.yml`:

```yaml
name: Release

on:
  push:
    branches: [main]

concurrency: ${{ github.workflow }}-${{ github.ref }}

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup pnpm
        uses: pnpm/action-setup@v3
        with:
          version: 9

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
          registry-url: 'https://registry.npmjs.org'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build
        run: pnpm build

      - name: Create Release Pull Request or Publish
        id: changesets
        uses: changesets/action@v1
        with:
          publish: pnpm release
          version: pnpm version-packages
          title: 'chore: version packages'
          commit: 'chore: version packages'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Step 3: Initialize Changesets

Create `.changeset/config.json`:

```json
{
  "$schema": "https://unpkg.com/@changesets/config@3.0.0/schema.json",
  "changelog": "@changesets/cli/changelog",
  "commit": false,
  "fixed": [],
  "linked": [["@tour-kit/core", "@tour-kit/react", "@tour-kit/hints"]],
  "access": "public",
  "baseBranch": "main",
  "updateInternalDependencies": "patch",
  "ignore": [],
  "___experimentalUnsafeOptions_WILL_CHANGE_IN_PATCH": {
    "onlyUpdatePeerDependentsWhenOutOfRange": true
  }
}
```

### Step 4: Create Changesets README

Create `.changeset/README.md`:

```markdown
# Changesets

This project uses [Changesets](https://github.com/changesets/changesets) for version management.

## Adding a Changeset

When you make changes that should be released, run:

\`\`\`bash
pnpm changeset
\`\`\`

This will prompt you to:

1. Select which packages have changed
2. Choose the bump type (major/minor/patch)
3. Write a summary of the changes

## Releasing

Releases are automated via GitHub Actions. When changesets are merged to main:

1. A "Version Packages" PR is created/updated
2. Merging that PR triggers the release workflow
3. Packages are published to npm

## Versioning Strategy

- **Major**: Breaking changes to public API
- **Minor**: New features, non-breaking
- **Patch**: Bug fixes, documentation
```

### Step 5: Create PR template

Create `.github/pull_request_template.md`:

```markdown
## Description

<!-- Describe your changes -->

## Type of Change

- [ ] Bug fix (non-breaking change that fixes an issue)
- [ ] New feature (non-breaking change that adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update

## Checklist

- [ ] My code follows the project's style guidelines
- [ ] I have performed a self-review of my code
- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] New and existing unit tests pass locally with my changes
- [ ] I have added a changeset (if applicable)

## Related Issues

<!-- Link to related issues: Fixes #123, Closes #456 -->
```

### Step 6: Create CODEOWNERS

Create `.github/CODEOWNERS`:

```
# Default owners for everything
* @your-username

# Package owners
/packages/core/ @your-username
/packages/react/ @your-username
/packages/hints/ @your-username
```

## Key Requirements & Constraints

- [ ] CI runs on push and PR to main
- [ ] Workflows use pnpm 9 and Node.js 20
- [ ] Releases are automated with Changesets
- [ ] Bundle size is checked on PRs
- [ ] All packages are linked for versioning
- [ ] Concurrency prevents duplicate runs

## Verification

```bash
# Verify changeset config
pnpm changeset status

# Dry run release
pnpm changeset version --dry-run
```

## Next Prompt

Continue with `04-core-types.md` to define TypeScript types.
