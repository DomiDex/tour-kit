# NPM Publishing Guide

## Overview

This guide covers everything you need to know about publishing TourKit packages to NPM, including setup, automation, troubleshooting, and best practices.

## Package Publishing Strategy

### Release Channels

| Channel | Tag      | Purpose          | Example               |
| ------- | -------- | ---------------- | --------------------- |
| Stable  | `latest` | Production-ready | `0.1.0`               |
| Beta    | `beta`   | Feature preview  | `0.2.0-beta.1`        |
| Alpha   | `alpha`  | Early testing    | `0.2.0-alpha.1`       |
| Canary  | `canary` | Nightly builds   | `0.0.0-canary.abc123` |

### Version Strategy

We follow [Semantic Versioning](https://semver.org/):

- **Major (X.0.0)**: Breaking changes to public API
- **Minor (0.X.0)**: New features, backward compatible
- **Patch (0.0.X)**: Bug fixes, backward compatible

**Pre-1.0 Convention:**

- `0.0.x` - Initial development, anything may change
- `0.x.0` - API stabilizing, minor = breaking changes

## Publishing Infrastructure

### Required Accounts & Tokens

```
┌─────────────────────────────────────────────────────────────┐
│                    Publishing Flow                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Developer                                                  │
│      │                                                      │
│      ▼                                                      │
│  [Create Changeset] ──► .changeset/unique-id.md            │
│      │                                                      │
│      ▼                                                      │
│  [Merge to main] ──────► GitHub Actions                    │
│                              │                              │
│                              ▼                              │
│                    [Changesets Action]                      │
│                              │                              │
│                    ┌────────┴────────┐                     │
│                    ▼                  ▼                     │
│          [Create Version PR]   [OR Publish]                │
│                    │                  │                     │
│                    ▼                  ▼                     │
│             [Merge PR] ────► [npm publish]                 │
│                                       │                     │
│                                       ▼                     │
│                               [NPM Registry]                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Token Setup Checklist

- [ ] **NPM Token** (Automation type)
  - Created at: npmjs.com/settings/~/tokens
  - Added to GitHub as: `NPM_TOKEN`
- [ ] **GitHub Token** (Automatic)
  - `GITHUB_TOKEN` is automatic in Actions
  - Needs `contents: write` permission

## Package Configuration

### Essential package.json Fields

```json
{
  "name": "@tour-kit/core",
  "version": "0.0.1",
  "description": "Clear, searchable description",

  "// ENTRY POINTS": "",
  "type": "module",
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": {
        "types": "./dist/index.d.ts",
        "default": "./dist/index.js"
      },
      "require": {
        "types": "./dist/index.d.cts",
        "default": "./dist/index.cjs"
      }
    },
    "./package.json": "./package.json"
  },

  "// PUBLISHING": "",
  "files": ["dist", "README.md", "CHANGELOG.md"],
  "sideEffects": false,
  "publishConfig": {
    "access": "public",
    "registry": "https://registry.npmjs.org/"
  },

  "// METADATA": "",
  "license": "MIT",
  "author": "TourKit Team",
  "homepage": "https://tour-kit.dev",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/tour-kit/tour-kit.git",
    "directory": "packages/core"
  },
  "bugs": {
    "url": "https://github.com/tour-kit/tour-kit/issues"
  },
  "keywords": ["react", "onboarding", "tour", "shadcn"]
}
```

### Files Whitelist Strategy

```json
{
  "files": ["dist", "README.md", "CHANGELOG.md", "LICENSE"]
}
```

**Always exclude (via .npmignore or files whitelist):**

- `src/` - Source files
- `*.test.ts` - Test files
- `*.config.*` - Config files
- `node_modules/`
- `.turbo/`
- `coverage/`

## Manual Publishing

### First-Time Setup

```bash
# 1. Login to NPM
npm login

# 2. Verify organization exists (for scoped packages)
npm org ls tour-kit

# 3. Build all packages
pnpm build

# 4. Navigate to package
cd packages/core
```

### Pre-Publish Checklist

```bash
# Check what will be published
npm pack --dry-run

# Verify package size
npm pack
ls -lh *.tgz
tar -tzf *.tgz | head -20
rm *.tgz

# Verify types are included
ls dist/*.d.ts

# Test locally before publish
npm link
cd /tmp/test-project
npm link @tour-kit/core
```

### Publishing Commands

```bash
# Dry run (no actual publish)
npm publish --dry-run --access public

# Publish to NPM
npm publish --access public

# Publish with specific tag
npm publish --access public --tag beta

# Publish from CI (uses NPM_TOKEN)
npm publish --access public
```

## Automated Publishing with Changesets

### Creating a Changeset

```bash
# Interactive changeset creation
pnpm changeset

# Or create manually
cat > .changeset/my-change.md << 'EOF'
---
"@tour-kit/core": patch
"@tour-kit/react": patch
---

Fix tooltip positioning on scroll
EOF
```

### Changeset Types

```markdown
---
"@tour-kit/core": major    # Breaking change
"@tour-kit/core": minor    # New feature
"@tour-kit/core": patch    # Bug fix
---
```

### CI/CD Workflow

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches: [main]

concurrency: ${{ github.workflow }}-${{ github.ref }}

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      id-token: write # For provenance

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v3
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
          registry-url: 'https://registry.npmjs.org'

      - run: pnpm install --frozen-lockfile
      - run: pnpm build
      - run: pnpm test

      - name: Create Release Pull Request or Publish
        uses: changesets/action@v1
        with:
          publish: pnpm release
          version: pnpm changeset version
          title: 'chore: release packages'
          commit: 'chore: release packages'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## Troubleshooting Guide

### Authentication Errors

#### "ENEEDAUTH - authentication required"

```bash
# Check if logged in
npm whoami

# Re-login
npm login

# For CI, verify token
echo "//registry.npmjs.org/:_authToken=${NPM_TOKEN}" > ~/.npmrc
npm whoami
```

#### "403 Forbidden"

**Possible causes:**

1. Token doesn't have publish permissions
2. Package name already taken by someone else
3. Organization permissions

**Solutions:**

```bash
# Check package ownership
npm owner ls @tour-kit/core

# Verify token permissions (regenerate if needed)
# Use "Automation" token type for CI

# Check org membership
npm org ls tour-kit
```

### Package Configuration Errors

#### "402 Payment Required"

Scoped packages are private by default:

```json
{
  "publishConfig": {
    "access": "public"
  }
}
```

Or publish with flag:

```bash
npm publish --access public
```

#### "Package name too similar to existing package"

NPM's spam detection. Options:

1. Use more unique name
2. Wait 24 hours and retry
3. Contact NPM support

#### "Cannot publish over existing version"

```bash
# Check existing versions
npm view @tour-kit/core versions

# Bump version
npm version patch  # or minor/major

# Then publish
npm publish --access public
```

### Build/Bundle Errors

#### "Types not found" for consumers

Ensure exports include types FIRST:

```json
{
  "exports": {
    ".": {
      "import": {
        "types": "./dist/index.d.ts", // MUST be first!
        "default": "./dist/index.js"
      }
    }
  }
}
```

#### ESM/CJS Resolution Issues

```json
{
  "type": "module",
  "main": "./dist/index.cjs", // CJS entry
  "module": "./dist/index.js", // ESM entry
  "types": "./dist/index.d.ts" // Types
}
```

### Size Issues

#### Package too large

```bash
# Check what's included
npm pack --dry-run

# Verify files array
cat package.json | jq '.files'

# Common exclusions
# Add to .npmignore or remove from files:
# - src/
# - *.test.ts
# - docs/
# - coverage/
```

## Recovery Procedures

### Unpublish (Within 72 Hours)

```bash
# Unpublish specific version
npm unpublish @tour-kit/core@0.0.1

# Unpublish entire package (use with extreme caution!)
npm unpublish @tour-kit/core --force
```

### Deprecate (Preferred)

```bash
# Deprecate with message
npm deprecate @tour-kit/core@0.0.1 "Critical bug - upgrade to 0.0.2"

# Deprecate version range
npm deprecate "@tour-kit/core@<0.1.0" "Pre-release versions, please upgrade"

# Un-deprecate
npm deprecate @tour-kit/core@0.0.1 ""
```

### Version Recovery

If you need to "re-release" the same version:

```bash
# You cannot. NPM prevents version reuse.
# Instead, publish a new patch version:
npm version patch
npm publish --access public
```

## Security Best Practices

### Token Security

1. **Never commit tokens** - Use environment variables
2. **Use Automation tokens** - Limited scope for CI
3. **Rotate tokens regularly** - Every 6 months
4. **Enable 2FA** - On your NPM account

### Package Security

```bash
# Enable provenance (proves package came from your CI)
npm publish --provenance

# Or in package.json:
{
  "publishConfig": {
    "provenance": true
  }
}
```

### Pre-Publish Security Check

```bash
# Audit for vulnerabilities
npm audit

# Check for secrets in package
npx secretlint "**/*"

# Verify no sensitive files included
npm pack --dry-run | grep -E "(\.env|secret|password|token)"
```

## Quick Reference Card

```bash
# ═══════════════════════════════════════════
#            NPM PUBLISHING CHEATSHEET
# ═══════════════════════════════════════════

# CHECK STATUS
npm whoami                           # Who am I?
npm view @tour-kit/core              # Package info
npm view @tour-kit/core versions     # All versions

# PRE-PUBLISH
npm pack --dry-run                   # Preview contents
npm pack && tar -tzf *.tgz           # Inspect archive

# PUBLISH
npm publish --access public          # Publish public
npm publish --tag beta               # Publish to tag
npm publish --dry-run                # Test publish

# POST-PUBLISH
npm deprecate PKG@VER "message"      # Deprecate
npm unpublish PKG@VER                # Remove (72h)
npm dist-tag add PKG@VER latest      # Change tag

# CHANGESETS
pnpm changeset                       # Create changeset
pnpm changeset version               # Apply versions
pnpm changeset publish               # Publish all
```
