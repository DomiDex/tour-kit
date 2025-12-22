# TourKit - Master Implementation Plan

## Overview

TourKit is a modern React onboarding and product tour library designed natively for Shadcn UI. This plan outlines the systematic implementation approach across 5 phases.

## Project Structure

```
tour-kit/
├── packages/
│   ├── core/          # @tour-kit/core - Headless hooks & logic
│   ├── react/         # @tour-kit/react - Shadcn components
│   └── hints/         # @tour-kit/hints - Persistent hints
├── apps/
│   ├── docs/          # Fumadocs documentation
│   └── demo/          # Next.js demo/playground
└── tooling/
    ├── tsconfig/      # Shared TypeScript configs
    └── biome/         # Shared Biome config
```

## Implementation Phases

| Phase | Focus         | Duration | Key Deliverables                    |
| ----- | ------------- | -------- | ----------------------------------- |
| 1     | Foundation    | Week 1-2 | Monorepo setup, CI/CD, core types   |
| 2     | Core Features | Week 2-3 | Floating UI, tour hooks, components |
| 3     | Polish        | Week 3-4 | Persistence, a11y, animations       |
| 4     | Hints System  | Week 4-5 | HintsProvider, Hint components      |
| 5     | Documentation | Week 5-6 | Fumadocs, examples, release         |

## Package Dependencies

```
@tour-kit/react
    └── @tour-kit/core

@tour-kit/hints
    └── @tour-kit/core

apps/demo
    ├── @tour-kit/core
    ├── @tour-kit/react
    └── @tour-kit/hints
```

## Quality Gates

### Size Budgets

- `@tour-kit/core`: < 8KB gzipped
- `@tour-kit/react`: < 12KB gzipped (including core)
- `@tour-kit/hints`: < 5KB gzipped

### Standards

- TypeScript strict mode enabled
- Test coverage > 80%
- Lighthouse Accessibility: 100
- WCAG 2.1 AA compliant

## Technology Stack

| Category           | Tool                   | Version |
| ------------------ | ---------------------- | ------- |
| Runtime            | Node.js                | ≥18.0.0 |
| Package Manager    | pnpm                   | ≥9.0.0  |
| Language           | TypeScript             | ^5.5.0  |
| Bundler            | tsup                   | ^8.0.0  |
| Monorepo           | Turborepo              | ^2.0.0  |
| Testing            | Vitest                 | ^2.0.0  |
| Component Testing  | @testing-library/react | ^16.0.0 |
| Linting/Formatting | Biome                  | ^1.8.0  |
| Versioning         | Changesets             | ^2.27.0 |
| CI/CD              | GitHub Actions         | -       |
| Positioning        | @floating-ui/react     | ^0.26.0 |

## Getting Started

1. Follow Phase 1 prompts to set up the monorepo
2. Implement core types and providers
3. Build hooks incrementally with tests
4. Add styled components
5. Document and release

## Files Reference

- **Phases**: `../phases/` - Detailed phase breakdowns
- **Specs**: `../specs/` - Technical specifications
- **Prompts**: `../prompts/` - Code generation prompts
