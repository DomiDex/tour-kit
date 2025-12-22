# Prompt 00: NPM Publishing Setup

## Problem

Set up NPM publishing infrastructure early to avoid common pitfalls. This should be done **before** or alongside monorepo initialization to ensure packages can be published from day one.

## Why Early?

Publishing early helps you:

1. Validate package names are available
2. Test the entire publish pipeline before complexity grows
3. Catch configuration issues immediately
4. Build confidence with "hello world" releases

## Prerequisites

- NPM account created at [npmjs.com](https://www.npmjs.com/)
- GitHub repository created
- pnpm installed globally

## Steps To Complete

### Step 1: NPM Account & Organization Setup

```bash
# Login to NPM (creates ~/.npmrc with auth token)
npm login

# Verify login
npm whoami
```

**For scoped packages (@tour-kit/\*)**, you have two options:

**Option A: Personal Scope (Recommended for solo projects)**

```bash
# Your username becomes the scope
# If username is "johndoe", packages are @johndoe/core
```

**Option B: Organization Scope (Recommended for team/brand)**

```bash
# Create organization at https://www.npmjs.com/org/create
# Organization name: tour-kit
# This enables @tour-kit/core, @tour-kit/react, etc.
```

### Step 2: Verify Package Name Availability

```bash
# Check if names are available (404 = available)
npm view @tour-kit/core
npm view @tour-kit/react
npm view @tour-kit/hints

# Or use npm-name-cli
npx npm-name @tour-kit/core @tour-kit/react @tour-kit/hints
```

**If names are taken:**

- Try alternative scopes: `@beacon-tour/core`, `@beaconui/core`
- Use unscoped names: `tour-kit-core` (less ideal)

### Step 3: Create NPM Token for CI/CD

1. Go to [npmjs.com/settings/tokens](https://www.npmjs.com/settings/~/tokens)
2. Click "Generate New Token"
3. Select **"Automation"** token type (for CI/CD)
4. Copy the token (starts with `npm_`)

**Add to GitHub Secrets:**

1. Go to your repo → Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Name: `NPM_TOKEN`
4. Value: paste your token

### Step 4: Configure Package.json for Publishing

Each package needs specific fields. Create a template:

**packages/core/package.json:**

```json
{
  "name": "@tour-kit/core",
  "version": "0.0.0",
  "description": "Headless onboarding and product tour library for React",
  "author": "Your Name <your@email.com>",
  "license": "MIT",
  "homepage": "https://github.com/your-org/tour-kit#readme",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/your-org/tour-kit.git",
    "directory": "packages/core"
  },
  "bugs": {
    "url": "https://github.com/your-org/tour-kit/issues"
  },
  "keywords": [
    "react",
    "onboarding",
    "tour",
    "product-tour",
    "walkthrough",
    "hints",
    "tooltip",
    "headless",
    "shadcn"
  ],
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
  "files": ["dist", "README.md", "CHANGELOG.md"],
  "sideEffects": false,
  "publishConfig": {
    "access": "public",
    "registry": "https://registry.npmjs.org/"
  },
  "scripts": {
    "build": "tsup",
    "prepublishOnly": "pnpm build"
  }
}
```

**Critical fields explained:**

- `publishConfig.access`: "public" required for scoped packages
- `files`: whitelist what gets published
- `exports`: modern entry points
- `prepublishOnly`: ensures build before publish

### Step 5: Create .npmrc Files

**Root `.npmrc`:**

```ini
auto-install-peers=true
strict-peer-dependencies=false
# Prevent accidental publish from root
package-lock=false
```

**packages/core/.npmrc** (and each package):

```ini
# Ensure we use the public registry
registry=https://registry.npmjs.org/
# Enable provenance for security (optional but recommended)
provenance=true
```

### Step 6: Create Minimal Publishable Package

Create a "hello world" version to test publishing:

**packages/core/src/index.ts:**

```typescript
export const VERSION = '0.0.1';
export const hello = () => 'Hello from @tour-kit/core!';
```

### Step 7: Test Publishing Locally

```bash
# Build the package
cd packages/core
pnpm build

# Check what would be published
npm pack --dry-run

# Verify package contents
npm pack
tar -tzf tour-kit-core-0.0.0.tgz
rm tour-kit-core-0.0.0.tgz

# Test publish (dry run)
npm publish --dry-run
```

### Step 8: First Real Publish (Manual)

Before using Changesets automation, do one manual publish:

```bash
cd packages/core

# Set initial version
npm version 0.0.1

# Publish
npm publish --access public

# Verify on NPM
npm view @tour-kit/core
```

### Step 9: Setup Changesets for Future Releases

**Root `.changeset/config.json`:**

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
  "ignore": []
}
```

**Test changeset flow:**

```bash
# Create a changeset
pnpm changeset

# Preview version bump
pnpm changeset version --dry-run

# Preview what would be published
pnpm changeset publish --dry-run
```

### Step 10: Verify GitHub Actions Can Publish

Create a test workflow to verify secrets:

**.github/workflows/test-npm-auth.yml:**

```yaml
name: Test NPM Auth

on:
  workflow_dispatch: # Manual trigger only

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Test NPM Token
        env:
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: |
          echo "//registry.npmjs.org/:_authToken=${NPM_TOKEN}" > ~/.npmrc
          npm whoami
          echo "✅ NPM authentication successful!"
```

Run this workflow manually to verify your token works.

## Common Pitfalls & Solutions

### ❌ "402 Payment Required"

**Cause:** Trying to publish scoped package without `access: public`
**Fix:** Add to package.json:

```json
"publishConfig": {
  "access": "public"
}
```

### ❌ "403 Forbidden - Package name too similar"

**Cause:** NPM spam detection or name squatting
**Fix:**

- Use a more unique scope/name
- Wait 24h and try again
- Contact NPM support

### ❌ "404 Not Found" on publish

**Cause:** Organization doesn't exist
**Fix:** Create org at npmjs.com/org/create first

### ❌ "ENEEDAUTH" in CI

**Cause:** NPM_TOKEN secret not set or invalid
**Fix:**

- Verify secret exists in GitHub
- Regenerate token if expired
- Check token type is "Automation"

### ❌ Files missing in published package

**Cause:** `files` array excludes needed files
**Fix:**

```bash
# Check what's included
npm pack --dry-run

# Update files array
"files": ["dist", "README.md", "CHANGELOG.md"]
```

### ❌ "Types not found" for consumers

**Cause:** Missing or wrong `types`/`exports` config
**Fix:** Ensure exports map includes types:

```json
"exports": {
  ".": {
    "import": {
      "types": "./dist/index.d.ts",
      "default": "./dist/index.js"
    }
  }
}
```

### ❌ ESM/CJS compatibility issues

**Cause:** Missing dual format build
**Fix:** Ensure tsup builds both:

```typescript
export default defineConfig({
  format: ['cjs', 'esm'],
  dts: true,
});
```

## Verification Checklist

- [ ] NPM account created and logged in
- [ ] Organization/scope created (if using scoped packages)
- [ ] Package names available
- [ ] NPM_TOKEN added to GitHub Secrets
- [ ] `publishConfig.access: "public"` in each package.json
- [ ] `files` array includes dist and docs
- [ ] `npm pack --dry-run` shows correct files
- [ ] Manual test publish successful
- [ ] CI workflow can authenticate with NPM

## Quick Commands Reference

```bash
# Check NPM login status
npm whoami

# Check package name availability
npm view @tour-kit/core || echo "Available!"

# Preview publish contents
npm pack --dry-run

# Publish scoped package
npm publish --access public

# Unpublish (within 72h only!)
npm unpublish @tour-kit/core@0.0.1

# Deprecate version (preferred over unpublish)
npm deprecate @tour-kit/core@0.0.1 "Use 0.0.2 instead"

# View package info
npm view @tour-kit/core

# List published versions
npm view @tour-kit/core versions --json
```

## Next Steps

After successful test publish:

1. Continue with `01-monorepo-init.md`
2. Build actual package content
3. Use Changesets for version management
4. Automate via CI/CD

## Rollback Plan

If you publish something broken:

```bash
# Within 72 hours: unpublish
npm unpublish @tour-kit/core@0.0.1

# After 72 hours: deprecate and publish fix
npm deprecate @tour-kit/core@0.0.1 "Critical bug - use 0.0.2"
npm version 0.0.2
npm publish --access public
```
