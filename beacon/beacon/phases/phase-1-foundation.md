# Phase 1: Foundation

**Duration:** Week 1-2  
**Goal:** Set up the monorepo infrastructure and define core types

## Objectives

1. Initialize Turborepo monorepo with pnpm
2. Configure TypeScript, Biome, and tooling
3. Set up CI/CD with GitHub Actions
4. Define all core types
5. Implement basic context providers

## Tasks

### 1.1 Monorepo Setup

- [ ] Initialize pnpm workspace
- [ ] Configure Turborepo
- [ ] Set up shared TypeScript configs
- [ ] Configure Biome linting/formatting
- [ ] Create package scaffolds

**Deliverables:**

- `pnpm-workspace.yaml`
- `turbo.json`
- `biome.json`
- `tooling/tsconfig/*.json`
- `tooling/biome/biome.json`

### 1.2 Package Configuration

- [ ] Set up `@tour-kit/core` package.json
- [ ] Set up `@tour-kit/react` package.json
- [ ] Set up `@tour-kit/hints` package.json
- [ ] Configure tsup for each package
- [ ] Set up Vitest for testing

**Deliverables:**

- `packages/core/package.json`
- `packages/core/tsup.config.ts`
- `packages/core/vitest.config.ts`
- (Same for react and hints packages)

### 1.3 CI/CD Pipeline

- [ ] Create CI workflow (lint, typecheck, test, build)
- [ ] Create Release workflow with Changesets
- [ ] Configure bundle size checks

**Deliverables:**

- `.github/workflows/ci.yml`
- `.github/workflows/release.yml`
- `.changeset/config.json`

### 1.4 Core Types Definition

- [ ] Define positioning types (Side, Alignment, Placement)
- [ ] Define tour types (Tour, TourStep)
- [ ] Define configuration types (KeyboardConfig, SpotlightConfig, etc.)
- [ ] Define state types (TourState, TourContext)
- [ ] Define action types (TourActions)
- [ ] Define hint types (HintConfig, HintState)

**Deliverables:**

- `packages/core/src/types/index.ts`
- `packages/core/src/types/tour.ts`
- `packages/core/src/types/step.ts`
- `packages/core/src/types/config.ts`
- `packages/core/src/types/state.ts`
- `packages/core/src/types/hints.ts`

### 1.5 Context Setup

- [ ] Create BeaconContext definition
- [ ] Implement BeaconProvider
- [ ] Create TourContext definition
- [ ] Implement basic TourProvider

**Deliverables:**

- `packages/core/src/context/beacon-context.ts`
- `packages/core/src/context/beacon-provider.tsx`
- `packages/core/src/context/tour-context.ts`
- `packages/core/src/context/tour-provider.tsx`

## Dependencies

None - This is the foundation phase.

## Success Criteria

- [ ] `pnpm install` succeeds
- [ ] `pnpm build` succeeds for all packages
- [ ] `pnpm lint` passes
- [ ] `pnpm typecheck` passes
- [ ] CI pipeline runs successfully
- [ ] Types are properly exported from @tour-kit/core

## File Tree After Phase 1

```
tour-kit/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── release.yml
├── .changeset/
│   └── config.json
├── packages/
│   ├── core/
│   │   ├── src/
│   │   │   ├── context/
│   │   │   │   ├── beacon-context.ts
│   │   │   │   ├── beacon-provider.tsx
│   │   │   │   ├── tour-context.ts
│   │   │   │   └── tour-provider.tsx
│   │   │   ├── types/
│   │   │   │   ├── index.ts
│   │   │   │   ├── tour.ts
│   │   │   │   ├── step.ts
│   │   │   │   ├── config.ts
│   │   │   │   ├── state.ts
│   │   │   │   └── hints.ts
│   │   │   └── index.ts
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── tsup.config.ts
│   │   └── vitest.config.ts
│   ├── react/
│   │   ├── src/
│   │   │   └── index.ts
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── tsup.config.ts
│   └── hints/
│       ├── src/
│       │   └── index.ts
│       ├── package.json
│       ├── tsconfig.json
│       └── tsup.config.ts
├── tooling/
│   ├── tsconfig/
│   │   ├── base.json
│   │   ├── react-library.json
│   │   └── nextjs.json
│   └── biome/
│       └── biome.json
├── biome.json
├── package.json
├── pnpm-workspace.yaml
├── turbo.json
└── README.md
```

## Prompts for This Phase

1. `../prompts/01-monorepo-init.md` - Initialize monorepo
2. `../prompts/02-tooling-config.md` - Configure tooling
3. `../prompts/03-ci-cd-setup.md` - Set up CI/CD
4. `../prompts/04-core-types.md` - Define types
5. `../prompts/05-context-setup.md` - Create contexts
