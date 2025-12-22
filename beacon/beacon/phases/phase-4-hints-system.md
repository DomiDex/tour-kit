# Phase 4: Hints System

**Duration:** Week 4-5  
**Goal:** Build the standalone hints/hotspots system

## Objectives

1. Create HintsProvider context
2. Implement hints management hooks
3. Build beacon (pulsing dot) component
4. Build hint tooltip component
5. Add persistence for hints

## Dependencies on Previous Phase

- Phase 3 complete (persistence, positioning, a11y)
- Shared utilities from `@tour-kit/core`
- Floating UI integration working

## Overview

The Hints system is a separate but complementary feature to Tours. While tours guide users through a sequence of steps, hints are persistent UI elements that highlight features or provide contextual information.

```
┌────────────────────────────────────────────────────────────┐
│                     HintsProvider                           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  State                                                │  │
│  │  - hints: Map<id, HintState>                         │  │
│  │  - activeHint: string | null                         │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Actions                                              │  │
│  │  - showHint(id)                                       │  │
│  │  - hideHint(id)                                       │  │
│  │  - dismissHint(id)                                    │  │
│  │  - resetHint(id)                                      │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

## Tasks

### 4.1 Hints Types

- [ ] Define HintConfig type
- [ ] Define HintState type
- [ ] Define HintsContext type
- [ ] Define position types for beacon

**Deliverables:**

- `packages/hints/src/types/index.ts`

### 4.2 Hints Context & Provider

- [ ] Create HintsContext
- [ ] Implement HintsProvider
- [ ] Add state management for hints
- [ ] Integrate with persistence

**Deliverables:**

- `packages/hints/src/context/hints-context.ts`
- `packages/hints/src/context/hints-provider.tsx`

### 4.3 Hints Hooks

- [ ] Implement `useHints` hook (manage all hints)
- [ ] Implement `useHint` hook (single hint)
- [ ] Add persistence integration

**Deliverables:**

- `packages/hints/src/hooks/use-hints.ts`
- `packages/hints/src/hooks/use-hint.ts`

### 4.4 HintBeacon Component

- [ ] Create pulsing dot animation
- [ ] Support different positions relative to target
- [ ] Handle click to show tooltip
- [ ] Support custom beacon styles

**Beacon Positions:**

```
┌─────────────────────────────┐
│  ●                       ●  │  ← top-left, top-right
│                             │
│            ●                │  ← center
│                             │
│  ●                       ●  │  ← bottom-left, bottom-right
└─────────────────────────────┘
```

**Deliverables:**

- `packages/hints/src/components/hint-beacon.tsx`
- CSS animations for pulse effect

### 4.5 HintTooltip Component

- [ ] Create tooltip content container
- [ ] Use Floating UI for positioning
- [ ] Handle open/close state
- [ ] Support custom content

**Deliverables:**

- `packages/hints/src/components/hint-tooltip.tsx`

### 4.6 Main Hint Component

- [ ] Combine beacon + tooltip
- [ ] Handle target element attachment
- [ ] Manage visibility state
- [ ] Support configuration options

**Deliverables:**

- `packages/hints/src/components/hint.tsx`

### 4.7 Hints Persistence

- [ ] Track dismissed hints
- [ ] Support per-hint persistence config
- [ ] Reset dismissed hints functionality

**Deliverables:**

- Persistence integration in HintsProvider
- Tests for persistence

### 4.8 Package Configuration

- [ ] Set up package.json
- [ ] Configure tsup build
- [ ] Set up exports
- [ ] Add tests

**Deliverables:**

- `packages/hints/package.json`
- `packages/hints/tsup.config.ts`
- `packages/hints/tsconfig.json`
- Comprehensive tests

## Component API

### Hint Component

```tsx
<Hint
  id='new-feature' // Unique identifier
  target='#feature-button' // Target element
  position='top-right' // Beacon position on target
  tooltipPlacement='bottom' // Tooltip placement when open
  pulse // Enable pulsing animation
  persist // Persist dismissal to storage
  autoShow={false} // Auto-show tooltip on mount
>
  <p>Check out this new feature!</p>
</Hint>
```

### useHints Hook

```tsx
const {
  hints, // All hint states
  activeHint, // Currently open hint ID
  showHint, // (id) => void
  hideHint, // (id) => void
  dismissHint, // (id) => void - persistent
  resetHint, // (id) => void - clear dismissal
  resetAllHints, // () => void
  isHintVisible, // (id) => boolean
  isHintDismissed, // (id) => boolean
} = useHints();
```

### useHint Hook

```tsx
const {
  isOpen, // Tooltip visible
  isDismissed, // Permanently dismissed
  show, // Open tooltip
  hide, // Close tooltip
  dismiss, // Dismiss permanently
  reset, // Clear dismissal
} = useHint('hint-id');
```

## Success Criteria

- [ ] Hints can be rendered on target elements
- [ ] Beacon pulse animation works
- [ ] Clicking beacon shows tooltip
- [ ] Dismissed hints persist
- [ ] Multiple hints can coexist
- [ ] Only one tooltip open at a time (optional)
- [ ] Bundle size < 5KB gzipped
- [ ] Tests cover >80% of code

## File Tree After Phase 4

```
packages/hints/
├── src/
│   ├── context/
│   │   ├── hints-context.ts
│   │   └── hints-provider.tsx
│   ├── components/
│   │   ├── hint.tsx
│   │   ├── hint-beacon.tsx
│   │   ├── hint-tooltip.tsx
│   │   └── index.ts
│   ├── hooks/
│   │   ├── use-hints.ts
│   │   ├── use-hint.ts
│   │   └── index.ts
│   ├── types/
│   │   └── index.ts
│   └── index.ts
├── package.json
├── tsconfig.json
├── tsup.config.ts
└── vitest.config.ts
```

## Prompts for This Phase

1. `../prompts/20-hints-types-context.md` - Types & context
2. `../prompts/21-hints-hooks.md` - useHints & useHint
3. `../prompts/22-hint-beacon.md` - Beacon component
4. `../prompts/23-hint-tooltip.md` - Tooltip component
5. `../prompts/24-hints-integration.md` - Final integration
