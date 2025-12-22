# Phase 2: Core Features

**Duration:** Week 2-3  
**Goal:** Implement core tour functionality with Floating UI integration

## Objectives

1. Integrate Floating UI for positioning
2. Implement core tour hooks
3. Create spotlight/overlay system
4. Build basic tour components
5. Add keyboard navigation

## Dependencies on Previous Phase

- Phase 1 complete (monorepo, types, basic context)
- Core types defined in `packages/core/src/types/`
- Basic providers in `packages/core/src/context/`

## Tasks

### 2.1 Utility Functions

- [ ] Create DOM utility functions
- [ ] Create position calculation utilities
- [ ] Create scroll utilities
- [ ] Create storage abstraction

**Deliverables:**
- `packages/core/src/utils/dom.ts`
- `packages/core/src/utils/position.ts`
- `packages/core/src/utils/scroll.ts`
- `packages/core/src/utils/storage.ts`
- `packages/core/src/utils/index.ts`

### 2.2 Core Hooks - State Management

- [ ] Implement `useTourState` (internal state reducer)
- [ ] Implement `useTour` (main public hook)
- [ ] Implement `useStep` (individual step hook)
- [ ] Add comprehensive tests

**Deliverables:**
- `packages/core/src/hooks/use-tour-state.ts`
- `packages/core/src/hooks/use-tour.ts`
- `packages/core/src/hooks/use-step.ts`
- `packages/core/src/hooks/__tests__/*.test.ts`

### 2.3 Floating UI Integration

- [ ] Implement `useElementPosition` hook
- [ ] Create positioning hook wrapping Floating UI
- [ ] Handle dynamic repositioning
- [ ] Add middleware configuration

**Deliverables:**
- `packages/core/src/hooks/use-element-position.ts`
- `packages/core/src/hooks/use-floating-position.ts`

### 2.4 Spotlight System

- [ ] Implement `useSpotlight` hook
- [ ] Calculate overlay cutout
- [ ] Handle spotlight animations
- [ ] Support multiple shapes (rounded, circular)

**Deliverables:**
- `packages/core/src/hooks/use-spotlight.ts`
- Tests for spotlight calculations

### 2.5 Keyboard Navigation

- [ ] Implement `useKeyboardNavigation` hook
- [ ] Support configurable keybindings
- [ ] Handle escape to close
- [ ] Arrow keys for navigation

**Deliverables:**
- `packages/core/src/hooks/use-keyboard.ts`

### 2.6 Focus Management

- [ ] Implement `useFocusTrap` hook
- [ ] Trap focus within tooltip
- [ ] Restore focus on exit
- [ ] Handle focus order

**Deliverables:**
- `packages/core/src/hooks/use-focus-trap.ts`

### 2.7 Basic Components (@tour-kit/react)

- [ ] Create `TourPortal` (portal wrapper)
- [ ] Create `TourOverlay` (spotlight overlay)
- [ ] Create `TourCard` (tooltip card)
- [ ] Create `TourArrow` (tooltip arrow)
- [ ] Create `Tour` wrapper component
- [ ] Create `TourStep` definition component

**Deliverables:**
- `packages/react/src/components/primitives/tour-portal.tsx`
- `packages/react/src/components/overlay/tour-overlay.tsx`
- `packages/react/src/components/card/tour-card.tsx`
- `packages/react/src/components/primitives/tour-arrow.tsx`
- `packages/react/src/components/tour/tour.tsx`
- `packages/react/src/components/tour/tour-step.tsx`

### 2.8 Factory Functions

- [ ] Implement `createTour` utility
- [ ] Implement `createStep` utility
- [ ] Add TypeScript inference support

**Deliverables:**
- `packages/core/src/utils/create-tour.ts`
- `packages/core/src/utils/create-step.ts`

## Success Criteria

- [ ] Tours can be started, navigated, and completed
- [ ] Floating UI positions tooltips correctly
- [ ] Spotlight overlay highlights target elements
- [ ] Keyboard navigation works (arrows, escape)
- [ ] Focus is trapped within active tooltip
- [ ] All hooks have >80% test coverage
- [ ] Components render without errors

## File Tree After Phase 2

```
packages/core/src/
├── context/
│   └── (from Phase 1)
├── hooks/
│   ├── use-tour.ts
│   ├── use-tour-state.ts
│   ├── use-step.ts
│   ├── use-spotlight.ts
│   ├── use-keyboard.ts
│   ├── use-focus-trap.ts
│   ├── use-element-position.ts
│   ├── use-floating-position.ts
│   └── index.ts
├── utils/
│   ├── dom.ts
│   ├── position.ts
│   ├── scroll.ts
│   ├── storage.ts
│   ├── create-tour.ts
│   ├── create-step.ts
│   └── index.ts
├── types/
│   └── (from Phase 1)
└── index.ts

packages/react/src/
├── components/
│   ├── tour/
│   │   ├── tour.tsx
│   │   ├── tour-step.tsx
│   │   └── index.ts
│   ├── card/
│   │   ├── tour-card.tsx
│   │   └── index.ts
│   ├── overlay/
│   │   ├── tour-overlay.tsx
│   │   └── index.ts
│   └── primitives/
│       ├── tour-portal.tsx
│       ├── tour-arrow.tsx
│       └── index.ts
└── index.ts
```

## Prompts for This Phase

1. `../prompts/06-utility-functions.md` - Create utilities
2. `../prompts/07-tour-state-hook.md` - Implement state management
3. `../prompts/08-floating-ui-integration.md` - Position hooks
4. `../prompts/09-spotlight-hook.md` - Spotlight system
5. `../prompts/10-keyboard-focus.md` - Keyboard & focus
6. `../prompts/11-basic-components.md` - React components
7. `../prompts/12-factory-functions.md` - Create tour/step utilities

