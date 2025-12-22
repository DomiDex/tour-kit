# TourKit - Master Implementation Plan

## Overview

TourKit is a modern React onboarding and product tour library designed natively for Shadcn UI. This plan outlines the systematic implementation approach across 5 phases.

## ⚠️ First Steps (Before Coding)

1. **Set up NPM publishing first** - See `npm-publishing.md` and `../prompts/00-npm-publishing-setup.md`
2. Verify package names are available
3. Create organization on NPM (if using scoped packages)
4. Add NPM_TOKEN to GitHub Secrets

> Publishing issues are the #1 cause of delayed releases. Set this up early!

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
| 0     | Pre-Setup     | Day 1    | NPM account, tokens, verify names   |
| 1     | Foundation    | Week 1-2 | Monorepo setup, CI/CD, core types   |
| 2     | Core Features | Week 2-3 | Floating UI, tour hooks, components |
| 3     | Polish        | Week 3-4 | Persistence, a11y, animations       |
| 4     | Hints System  | Week 4-5 | HintsProvider, Hint components      |
| 5     | Documentation | Week 5-6 | Fumadocs, examples, release         |

### Milestone: First Publish

After completing Phase 1 (prompts 00-03), do a test publish:

```bash
# Build minimal package
cd packages/core
pnpm build

# Test publish (dry run)
npm publish --dry-run --access public

# If dry run works, publish 0.0.1
npm publish --access public
```

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

```bash
# Recommended order:
1. ../prompts/00-npm-publishing-setup.md  # ⚠️ DO THIS FIRST
2. ../prompts/01-monorepo-init.md
3. ../prompts/02-tooling-config.md
4. ../prompts/03-ci-cd-setup.md
5. [TEST PUBLISH] - Verify everything works
6. Continue with 04-10...
```

### Quick Start Commands

```bash
# After initial setup
pnpm install
pnpm build
pnpm test

# Publishing
pnpm changeset          # Create changeset
pnpm changeset version  # Apply version bumps
pnpm release            # Publish to NPM
```

## Files Reference

| File/Directory         | Description                       |
| ---------------------- | --------------------------------- |
| `npm-publishing.md`    | NPM publishing guide & troubleshooting |
| `architecture.md`      | System architecture details       |
| `../phases/`           | Detailed phase breakdowns         |
| `../specs/`            | Technical specifications          |
| `../prompts/`          | Code generation prompts           |

## Publishing Checklist

Before each release:

- [ ] All tests pass (`pnpm test`)
- [ ] Build succeeds (`pnpm build`)
- [ ] Lint passes (`pnpm lint`)
- [ ] Changeset created (`pnpm changeset`)
- [ ] Package size within budget
- [ ] CHANGELOG updated

## Troubleshooting

See `npm-publishing.md` for common issues:

| Issue | Quick Fix |
|-------|-----------|
| 402 Payment Required | Add `publishConfig.access: "public"` |
| ENEEDAUTH in CI | Check `NPM_TOKEN` secret |
| Types not found | Verify `exports` field |

## Success Metrics

| Metric | Target |
|--------|--------|
| Bundle Size (core) | < 8KB |
| Time to First Tour | < 5 minutes |
| Lighthouse a11y | 100 |
| Test Coverage | > 80% |
| First Publish | Within Week 1 |

