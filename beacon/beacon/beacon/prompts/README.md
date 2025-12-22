# TourKit Implementation Prompts

## Overview

This directory contains detailed prompts for generating the TourKit library code. Each prompt builds on the previous ones, creating a complete implementation.

## Prompt Sequence

### Phase 0: Pre-Setup (Critical)

| #   | Prompt                       | Description                                  |
| --- | ---------------------------- | -------------------------------------------- |
| 00  | `00-npm-publishing-setup.md` | NPM account, tokens, and publishing setup ⚠️ |

> ⚠️ **Do this first!** Setting up NPM publishing early prevents common issues later. See also: `../plan/npm-publishing.md` for troubleshooting.

### Phase 1: Foundation

| #   | Prompt                 | Description                             |
| --- | ---------------------- | --------------------------------------- |
| 01  | `01-monorepo-init.md`  | Initialize Turborepo monorepo with pnpm |
| 02  | `02-tooling-config.md` | Configure TypeScript and Biome          |
| 03  | `03-ci-cd-setup.md`    | Set up GitHub Actions and Changesets    |
| 04  | `04-core-types.md`     | Define all TypeScript types             |
| 05  | `05-context-setup.md`  | Create React context providers          |

### Phase 2: Core Features

| #   | Prompt                    | Description                              |
| --- | ------------------------- | ---------------------------------------- |
| 06  | `06-utility-functions.md` | Create DOM, position, scroll utilities   |
| 07  | `07-tour-hooks.md`        | Implement useTour, useStep, useSpotlight |
| 08  | `08-react-components.md`  | Build styled React components            |

### Phase 3: Hints & Polish

| #   | Prompt                | Description                       |
| --- | --------------------- | --------------------------------- |
| 09  | `09-hints-package.md` | Build the @tour-kit/hints package |
| 10  | `10-testing-setup.md` | Configure Vitest and write tests  |

### Phase 4: Documentation (Coming Soon)

- Documentation site with Fumadocs
- API reference generation
- Interactive examples

## How to Use These Prompts

### Recommended Order

```
00 → 01 → 02 → 03 → [TEST PUBLISH] → 04 → 05 → 06 → 07 → 08 → 09 → 10
                          ↑
                    Validate NPM works!
```

### Sequential Execution

1. **Start with Prompt 00** - Set up NPM publishing first
2. Complete Prompts 01-03 (monorepo setup)
3. **Test publish a minimal package** - Catch issues early
4. Continue with 04-10
5. Verify each prompt's output before proceeding

### Prompt Format

Each prompt includes:

- **Problem**: What needs to be built
- **Dependencies**: Previous prompts required
- **Supporting Information**: File structure, specs
- **Steps To Complete**: Detailed implementation
- **Key Requirements**: Validation checklist
- **Verification**: How to test the output

### Example Usage

```markdown
# To execute Prompt 00 (NPM Setup):

1. Create NPM account at npmjs.com
2. Create organization @tour-kit (if using scoped packages)
3. Generate automation token
4. Add token to GitHub secrets
5. Follow verification steps
```

## File References

| Directory    | Description                       |
| ------------ | --------------------------------- |
| `../specs/`  | Detailed technical specifications |
| `../phases/` | Phase breakdown documentation     |
| `../plan/`   | Project plan and architecture     |

## Publishing Quick Reference

```bash
# Verify NPM setup (from Prompt 00)
npm whoami
npm view @tour-kit/core || echo "Name available!"

# Test publish flow
npm pack --dry-run
npm publish --dry-run --access public

# Create changeset for release
pnpm changeset
```

See `../plan/npm-publishing.md` for detailed troubleshooting.

## Quality Checklist

Before completing each prompt, verify:

- [ ] All files created at correct paths
- [ ] TypeScript compiles without errors
- [ ] Lint passes
- [ ] Tests pass (where applicable)
- [ ] Build succeeds
- [ ] (After 03) NPM publish dry-run works

## Customization

These prompts use sensible defaults but can be customized:

- **Package names**: Change `@tour-kit/*` to your org
- **Styling**: Modify Tailwind classes for different themes
- **Features**: Add/remove features per requirements

## Common Issues

| Issue                  | Solution                              |
| ---------------------- | ------------------------------------- |
| "402 Payment Required" | Add `publishConfig.access: "public"`  |
| "ENEEDAUTH" in CI      | Verify `NPM_TOKEN` secret exists      |
| Types not found        | Check `exports` field in package.json |
| Package too large      | Review `files` array whitelist        |

See `../plan/npm-publishing.md` for full troubleshooting guide.

## Support

See the main `briefing.md` for:

- Full technical specifications
- Architecture decisions
- API designs
