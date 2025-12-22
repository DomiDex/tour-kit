# Phase 5: Documentation & Release

**Duration:** Week 5-6  
**Goal:** Create documentation site, examples, and prepare for release

## Objectives

1. Set up Fumadocs documentation site
2. Write comprehensive API documentation
3. Create usage examples and guides
4. Build interactive demo site
5. Prepare initial release (v0.1.0)

## Dependencies on Previous Phase

- All packages complete (core, react, hints)
- Full test coverage
- All features implemented

## Tasks

### 5.1 Fumadocs Setup

- [ ] Initialize Next.js app for docs
- [ ] Configure Fumadocs
- [ ] Set up MDX content structure
- [ ] Create documentation layout
- [ ] Add syntax highlighting
- [ ] Add component previews

**Deliverables:**

- `apps/docs/` - Complete Fumadocs site

### 5.2 Documentation Structure

```
apps/docs/content/
├── docs/
│   ├── index.mdx              # Introduction
│   ├── getting-started/
│   │   ├── installation.mdx
│   │   ├── quick-start.mdx
│   │   └── typescript.mdx
│   ├── core/
│   │   ├── overview.mdx
│   │   ├── beacon-provider.mdx
│   │   ├── tour-provider.mdx
│   │   └── hooks/
│   │       ├── use-tour.mdx
│   │       ├── use-step.mdx
│   │       ├── use-spotlight.mdx
│   │       └── ...
│   ├── react/
│   │   ├── overview.mdx
│   │   ├── tour.mdx
│   │   ├── tour-step.mdx
│   │   ├── tour-card.mdx
│   │   └── components/
│   │       └── ...
│   ├── hints/
│   │   ├── overview.mdx
│   │   ├── hints-provider.mdx
│   │   ├── hint.mdx
│   │   └── hooks.mdx
│   ├── guides/
│   │   ├── styling.mdx
│   │   ├── animations.mdx
│   │   ├── persistence.mdx
│   │   ├── accessibility.mdx
│   │   ├── nextjs.mdx
│   │   └── headless.mdx
│   └── api/
│       ├── core.mdx
│       ├── react.mdx
│       └── hints.mdx
```

### 5.3 Core Documentation

- [ ] Write package overview
- [ ] Document all types
- [ ] Document all hooks
- [ ] Document all utilities
- [ ] Add code examples

**Deliverables:**

- Complete `@tour-kit/core` documentation

### 5.4 React Documentation

- [ ] Write package overview
- [ ] Document all components
- [ ] Add interactive examples
- [ ] Document styling/customization
- [ ] Add Shadcn integration guide

**Deliverables:**

- Complete `@tour-kit/react` documentation

### 5.5 Hints Documentation

- [ ] Write package overview
- [ ] Document components
- [ ] Document hooks
- [ ] Add examples

**Deliverables:**

- Complete `@tour-kit/hints` documentation

### 5.6 Guides

- [ ] Getting Started guide
- [ ] Styling guide
- [ ] Animation guide
- [ ] Persistence guide
- [ ] Accessibility guide
- [ ] Next.js integration guide
- [ ] Headless usage guide

**Deliverables:**

- Comprehensive guides section

### 5.7 Demo Site

- [ ] Create Next.js demo app
- [ ] Build interactive playground
- [ ] Create example tours
- [ ] Create example hints
- [ ] Add code view/copy

**Deliverables:**

- `apps/demo/` - Interactive demo site

### 5.8 API Reference

- [ ] Auto-generate API docs from types
- [ ] Document all exports
- [ ] Add type signatures
- [ ] Include examples

**Deliverables:**

- Complete API reference

### 5.9 Release Preparation

- [ ] Write CHANGELOG.md
- [ ] Create migration guides (for future)
- [ ] Update all package READMEs
- [ ] Create GitHub release notes template
- [ ] Configure npm publish

**Deliverables:**

- Release-ready packages

### 5.10 Final Checklist

- [ ] All tests passing
- [ ] Bundle sizes within budget
- [ ] Lighthouse accessibility: 100
- [ ] No TypeScript errors
- [ ] Documentation complete
- [ ] Demo working
- [ ] npm publish dry run

## Documentation Site Structure

```
apps/docs/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── docs/
│   │   └── [[...slug]]/
│   │       └── page.tsx
│   └── api/
│       └── search/
│           └── route.ts
├── content/
│   └── docs/
│       └── (mdx files)
├── components/
│   ├── example-preview.tsx
│   ├── code-block.tsx
│   └── ...
├── lib/
│   └── source.ts
├── public/
│   └── ...
├── next.config.mjs
├── package.json
└── tailwind.config.ts
```

## Demo Site Structure

```
apps/demo/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── tours/
│   │   ├── basic/page.tsx
│   │   ├── advanced/page.tsx
│   │   └── headless/page.tsx
│   └── hints/
│       └── page.tsx
├── components/
│   ├── demo-container.tsx
│   ├── code-viewer.tsx
│   └── ...
├── examples/
│   ├── basic-tour.tsx
│   ├── styled-tour.tsx
│   └── ...
├── package.json
└── tailwind.config.ts
```

## Success Criteria

- [ ] Documentation site deploys successfully
- [ ] All packages documented
- [ ] Demo site functional
- [ ] Examples work correctly
- [ ] Search works in docs
- [ ] Mobile responsive
- [ ] npm publish succeeds

## Prompts for This Phase

1. `../prompts/25-fumadocs-setup.md` - Documentation site
2. `../prompts/26-api-docs.md` - API reference
3. `../prompts/27-guides.md` - Usage guides
4. `../prompts/28-demo-site.md` - Interactive demo
5. `../prompts/29-release.md` - Release preparation

## Release Checklist

```bash
# Pre-release
pnpm lint
pnpm typecheck
pnpm test
pnpm build

# Bundle size check
npx bundlesize

# Dry run publish
pnpm publish --dry-run

# Create changeset
pnpm changeset

# Version packages
pnpm version-packages

# Publish
pnpm release
```
