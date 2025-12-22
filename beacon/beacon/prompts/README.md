# TourKit Implementation Prompts

## Overview

This directory contains detailed prompts for generating the TourKit library code. Each prompt builds on the previous ones, creating a complete implementation.

## Prompt Sequence

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

### Sequential Execution

1. Start with Prompt 01 and execute each in order
2. Verify each prompt's output before proceeding
3. Use the "Verification" section to validate

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
# To execute Prompt 01:

1. Read the problem statement
2. Follow each step in order
3. Create all specified files
4. Run verification commands
5. Proceed to next prompt
```

## File References

| Directory    | Description                       |
| ------------ | --------------------------------- |
| `../specs/`  | Detailed technical specifications |
| `../phases/` | Phase breakdown documentation     |
| `../plan/`   | Project plan and architecture     |

## Quality Checklist

Before completing each prompt, verify:

- [ ] All files created at correct paths
- [ ] TypeScript compiles without errors
- [ ] Lint passes
- [ ] Tests pass (where applicable)
- [ ] Build succeeds

## Customization

These prompts use sensible defaults but can be customized:

- **Package names**: Change `@tour-kit/*` to your org
- **Styling**: Modify Tailwind classes for different themes
- **Features**: Add/remove features per requirements

## Support

See the main `briefing.md` for:

- Full technical specifications
- Architecture decisions
- API designs
