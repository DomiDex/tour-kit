# Phase 3: Polish

**Duration:** Week 3-4  
**Goal:** Add persistence, accessibility features, animations, and complete components

## Objectives

1. Implement persistence layer
2. Add progress indicators
3. Integrate animations (optional Framer Motion)
4. Complete accessibility features
5. Add route navigation support
6. Polish all components

## Dependencies on Previous Phase

- Phase 2 complete (core hooks, basic components)
- `useTour`, `useStep`, `useSpotlight` implemented
- Basic `TourCard`, `TourOverlay` components exist

## Tasks

### 3.1 Persistence Layer

- [ ] Implement `usePersistence` hook
- [ ] Support localStorage/sessionStorage
- [ ] Support custom storage adapters
- [ ] Track completed/skipped tours
- [ ] Implement "Don't show again" feature
- [ ] Remember step position on exit

**Deliverables:**

- `packages/core/src/hooks/use-persistence.ts`
- `packages/core/src/utils/storage.ts` (enhanced)
- Tests for persistence functionality

### 3.2 Progress Indicators

- [ ] Create `TourProgress` component
- [ ] Support different styles (dots, bar, numbers)
- [ ] Animate progress changes

**Deliverables:**

- `packages/react/src/components/navigation/tour-progress.tsx`

### 3.3 Navigation Components

- [ ] Create `TourNavigation` component (prev/next buttons)
- [ ] Create `TourClose` component
- [ ] Style components with Shadcn patterns

**Deliverables:**

- `packages/react/src/components/navigation/tour-navigation.tsx`
- `packages/react/src/components/navigation/tour-close.tsx`

### 3.4 Card Composition Components

- [ ] Create `TourCardHeader` component
- [ ] Create `TourCardContent` component
- [ ] Create `TourCardFooter` component
- [ ] Enable flexible composition

**Deliverables:**

- `packages/react/src/components/card/tour-card-header.tsx`
- `packages/react/src/components/card/tour-card-content.tsx`
- `packages/react/src/components/card/tour-card-footer.tsx`

### 3.5 Animation Support

- [ ] Create `useMediaQuery` hook (reduced motion)
- [ ] Add CSS transition support
- [ ] Optional Framer Motion integration
- [ ] Animate spotlight movement
- [ ] Animate card entrance/exit

**Deliverables:**

- `packages/core/src/hooks/use-media-query.ts`
- Animation utilities and examples

### 3.6 Screen Reader Support

- [ ] Create `useAriaAnnounce` hook
- [ ] Add ARIA live regions
- [ ] Announce step changes
- [ ] Create accessibility utility functions

**Deliverables:**

- `packages/core/src/hooks/use-aria-announce.ts`
- `packages/core/src/utils/a11y.ts`

### 3.7 Scroll Management

- [ ] Implement `useScroll` hook
- [ ] Auto-scroll to target elements
- [ ] Configurable scroll behavior
- [ ] Handle scroll containers

**Deliverables:**

- `packages/core/src/hooks/use-scroll.ts`
- Enhanced scroll utilities

### 3.8 Route Navigation Support

- [ ] Handle route changes in tour
- [ ] Wait for elements after navigation
- [ ] Support Next.js router
- [ ] Support React Router

**Deliverables:**

- Route handling in tour context
- Documentation for router integration

### 3.9 Conditional Steps

- [ ] Implement `when` condition evaluation
- [ ] Async condition support
- [ ] Step skipping logic

**Deliverables:**

- Condition evaluation in tour state machine

### 3.10 Advanced Features

- [ ] Interactive step support
- [ ] Auto-advance on events
- [ ] Custom target wait timeout
- [ ] Step lifecycle hooks

**Deliverables:**

- Enhanced `TourStep` configuration handling
- Event listener management

## Success Criteria

- [ ] Tour progress persists across page reloads
- [ ] "Don't show again" works correctly
- [ ] Progress indicators display correctly
- [ ] Animations respect reduced motion preference
- [ ] Screen readers announce step changes
- [ ] Target elements auto-scroll into view
- [ ] Conditional steps work correctly
- [ ] Interactive steps allow target interaction

## File Tree After Phase 3

```
packages/core/src/
├── hooks/
│   ├── (from Phase 2)
│   ├── use-persistence.ts
│   ├── use-media-query.ts
│   ├── use-aria-announce.ts
│   └── use-scroll.ts
├── utils/
│   ├── (from Phase 2)
│   ├── a11y.ts
│   └── storage.ts (enhanced)
└── index.ts (updated exports)

packages/react/src/
├── components/
│   ├── tour/ (from Phase 2)
│   ├── card/
│   │   ├── tour-card.tsx
│   │   ├── tour-card-header.tsx
│   │   ├── tour-card-content.tsx
│   │   ├── tour-card-footer.tsx
│   │   └── index.ts
│   ├── navigation/
│   │   ├── tour-navigation.tsx
│   │   ├── tour-progress.tsx
│   │   ├── tour-close.tsx
│   │   └── index.ts
│   ├── overlay/ (from Phase 2)
│   └── primitives/ (from Phase 2)
├── styles/
│   └── globals.css
└── index.ts (updated exports)
```

## Prompts for This Phase

1. `../prompts/13-persistence-layer.md` - Storage & persistence
2. `../prompts/14-progress-navigation.md` - Progress & nav components
3. `../prompts/15-card-composition.md` - Card sub-components
4. `../prompts/16-animations.md` - Animation support
5. `../prompts/17-accessibility.md` - A11y features
6. `../prompts/18-scroll-routing.md` - Scroll & route handling
7. `../prompts/19-advanced-features.md` - Conditions, events
