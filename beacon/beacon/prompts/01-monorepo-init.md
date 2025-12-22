# Prompt 01: Initialize Monorepo

## Problem

Set up a Turborepo monorepo with pnpm workspaces for the TourKit library. This is the foundation for all packages.

## Supporting Information

### Target File Structure

```
tour-kit/
├── package.json
├── pnpm-workspace.yaml
├── turbo.json
├── .gitignore
├── .npmrc
└── README.md
```

### Relevant Technologies

- pnpm 9.x
- Turborepo 2.x
- Node.js ≥18.0.0

## Steps To Complete

### Step 1: Create root package.json

Create `package.json` with the following:

```json
{
  "name": "tour-kit",
  "private": true,
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "test": "turbo test",
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "format": "biome format --write .",
    "typecheck": "turbo typecheck",
    "clean": "turbo clean && rm -rf node_modules",
    "changeset": "changeset",
    "version-packages": "changeset version",
    "release": "turbo build --filter=./packages/* && changeset publish"
  },
  "devDependencies": {
    "@biomejs/biome": "^1.8.0",
    "@changesets/cli": "^2.27.0",
    "turbo": "^2.0.0",
    "typescript": "^5.5.0"
  },
  "packageManager": "pnpm@9.0.0",
  "engines": {
    "node": ">=18.0.0"
  }
}
```

### Step 2: Create pnpm-workspace.yaml

Create `pnpm-workspace.yaml`:

```yaml
packages:
  - 'packages/*'
  - 'apps/*'
  - 'tooling/*'
```

### Step 3: Create turbo.json

Create `turbo.json`:

```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"]
    },
    "typecheck": {
      "dependsOn": ["^build"]
    },
    "lint": {
      "dependsOn": ["^build"]
    },
    "clean": {
      "cache": false
    }
  }
}
```

### Step 4: Create .gitignore

Create `.gitignore`:

```gitignore
# Dependencies
node_modules
.pnpm-store

# Build outputs
dist
.next
.turbo

# Cache
*.tsbuildinfo
.cache

# IDE
.idea
.vscode/*
!.vscode/extensions.json

# OS
.DS_Store
Thumbs.db

# Env
.env
.env.local
.env*.local

# Testing
coverage

# Logs
*.log
npm-debug.log*
pnpm-debug.log*
```

### Step 5: Create .npmrc

Create `.npmrc`:

```
auto-install-peers=true
strict-peer-dependencies=false
```

### Step 6: Create README.md

Create `README.md`:

```markdown
# TourKit

A modern React onboarding and product tour library designed natively for Shadcn UI.

## Features

- 🎯 **Shadcn-native** - Uses Shadcn primitives, follows copy-paste philosophy
- 💡 **Hints system** - Persistent UI beacons (unique in Shadcn ecosystem)
- 🎨 **Headless + Styled** - Full flexibility or quick start
- ♿ **Accessible** - WCAG 2.1 AA compliant
- 📦 **TypeScript-first** - Full type safety

## Packages

| Package           | Description                  | Size   |
| ----------------- | ---------------------------- | ------ |
| `@tour-kit/core`  | Headless hooks and utilities | < 8KB  |
| `@tour-kit/react` | Shadcn-styled components     | < 12KB |
| `@tour-kit/hints` | Persistent hints/hotspots    | < 5KB  |

## Installation

\`\`\`bash

# Core only (headless)

pnpm add @tour-kit/core

# With styled components

pnpm add @tour-kit/react

# Hints system

pnpm add @tour-kit/hints
\`\`\`

## Quick Start

\`\`\`tsx
import { Tour, TourStep } from '@tour-kit/react';

function App() {
return (
<Tour id="onboarding" autoStart>
<TourStep
        id="welcome"
        target="#welcome-btn"
        title="Welcome!"
        content="Let's take a quick tour."
        placement="bottom"
      />
<TourStep
        id="dashboard"
        target="#dashboard"
        title="Dashboard"
        content="Your data overview."
        placement="right"
      />
</Tour>
);
}
\`\`\`

## Development

\`\`\`bash

# Install dependencies

pnpm install

# Start development

pnpm dev

# Build all packages

pnpm build

# Run tests

pnpm test
\`\`\`

## License

MIT
```

### Step 7: Create directory structure

Create the following empty directories:

```
mkdir -p packages/core/src
mkdir -p packages/react/src
mkdir -p packages/hints/src
mkdir -p apps/docs
mkdir -p apps/demo
mkdir -p tooling/tsconfig
mkdir -p tooling/biome
```

## Key Requirements & Constraints

- [ ] All files use correct syntax (JSON, YAML, markdown)
- [ ] pnpm workspace includes all required directories
- [ ] Turborepo tasks have correct dependencies
- [ ] .gitignore covers all common patterns
- [ ] README is professional and accurate

## Verification

After running this prompt, verify with:

```bash
# Install dependencies
pnpm install

# Verify workspace structure
pnpm ls --depth 0
```

## Next Prompt

Continue with `02-tooling-config.md` to configure TypeScript and Biome.
