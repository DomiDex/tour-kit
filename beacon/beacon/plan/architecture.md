# TourKit - Architecture Design

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                         │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │  apps/docs  │  │  apps/demo   │  │  User's App      │   │
│  └──────┬──────┘  └──────┬───────┘  └────────┬─────────┘   │
└─────────┼────────────────┼───────────────────┼─────────────┘
          │                │                   │
┌─────────┼────────────────┼───────────────────┼─────────────┐
│         ▼                ▼                   ▼             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              @tour-kit/react                         │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐  │   │
│  │  │   Tour   │ │ TourCard │ │ Overlay  │ │Progress│  │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └────────┘  │   │
│  └───────────────────────┬─────────────────────────────┘   │
│                          │                                  │
│  ┌───────────────────────┼─────────────────────────────┐   │
│  │                       ▼                              │   │
│  │              @tour-kit/hints                         │   │
│  │  ┌──────────┐ ┌────────────┐ ┌────────────────────┐ │   │
│  │  │   Hint   │ │ HintBeacon │ │    HintTooltip     │ │   │
│  │  └──────────┘ └────────────┘ └────────────────────┘ │   │
│  └───────────────────────┬─────────────────────────────┘   │
│                          │                                  │
│  ┌───────────────────────▼─────────────────────────────┐   │
│  │              @tour-kit/core                          │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌──────────┐   │   │
│  │  │ Context │ │  Hooks  │ │  Utils  │ │  Types   │   │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └──────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
│                          │                                  │
│                          ▼                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              External Dependencies                   │   │
│  │  ┌──────────────────┐  ┌──────────────────────┐     │   │
│  │  │ @floating-ui/react│  │   framer-motion     │     │   │
│  │  │    (required)     │  │    (optional)       │     │   │
│  │  └──────────────────┘  └──────────────────────┘     │   │
│  └─────────────────────────────────────────────────────┘   │
│                      Package Layer                          │
└─────────────────────────────────────────────────────────────┘
```

## Data Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                        BeaconProvider                             │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                      TourContext                            │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐   │  │
│  │  │  TourState   │  │ TourActions  │  │ TourConfig      │   │  │
│  │  │  - isActive  │  │ - start()    │  │ - keyboard      │   │  │
│  │  │  - stepIndex │  │ - next()     │  │ - spotlight     │   │  │
│  │  │  - tourId    │  │ - prev()     │  │ - persistence   │   │  │
│  │  └──────────────┘  └──────────────┘  └─────────────────┘   │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                      HintsContext                           │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐   │  │
│  │  │  HintsState  │  │HintsActions  │  │ HintsConfig     │   │  │
│  │  │  - hints[]   │  │ - show()     │  │ - persistence   │   │  │
│  │  │  - activeId  │  │ - dismiss()  │  │                 │   │  │
│  │  └──────────────┘  └──────────────┘  └─────────────────┘   │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                          Hooks Layer                              │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌────────────┐  │
│  │  useTour()  │ │ useStep()   │ │useSpotlight │ │ useHints() │  │
│  └─────────────┘ └─────────────┘ └─────────────┘ └────────────┘  │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌────────────┐  │
│  │useKeyboard()│ │useFocusTrap │ │usePersist() │ │usePosition │  │
│  └─────────────┘ └─────────────┘ └─────────────┘ └────────────┘  │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                       Components Layer                            │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    @tour-kit/react                           │ │
│  │  <Tour> → <TourStep> → <TourCard> + <TourOverlay>            │ │
│  └─────────────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    @tour-kit/hints                           │ │
│  │  <Hint> → <HintBeacon> + <HintTooltip>                       │ │
│  └─────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

## State Management

### Tour State Machine

```
┌─────────┐     start()      ┌──────────┐
│  IDLE   │ ────────────────▶│  ACTIVE  │
└─────────┘                   └────┬─────┘
     ▲                             │
     │                             ▼
     │                    ┌────────────────┐
     │                    │   STEP: N      │
     │                    │  ┌──────────┐  │
     │    skip()/         │  │ Loading  │  │
     │    complete()      │  └────┬─────┘  │
     │                    │       ▼        │
     │                    │  ┌──────────┐  │
     └────────────────────│  │ Visible  │  │
                          │  └────┬─────┘  │
                          │       │        │
                          │  next()/prev() │
                          │       │        │
                          └───────┼────────┘
                                  │
                                  ▼
                          ┌───────────────┐
                          │  COMPLETED /  │
                          │   SKIPPED     │
                          └───────────────┘
```

## Positioning System (Floating UI Integration)

```
┌─────────────────────────────────────────────────────────────┐
│                    useFloating Hook                          │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  Configuration                                           ││
│  │  - placement: 'top' | 'bottom' | 'left' | 'right'       ││
│  │  - middleware: [offset, flip, shift, arrow]             ││
│  │  - strategy: 'absolute' | 'fixed'                       ││
│  └─────────────────────────────────────────────────────────┘│
│                          │                                   │
│                          ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  Returns                                                 ││
│  │  - refs: { setReference, setFloating }                  ││
│  │  - floatingStyles: CSSProperties                        ││
│  │  - context: FloatingContext                             ││
│  │  - middlewareData: { offset, flip, shift, arrow }       ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Interaction Hooks                         │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌──────────┐  │
│  │ useHover() │ │ useFocus() │ │useDismiss()│ │ useRole()│  │
│  └────────────┘ └────────────┘ └────────────┘ └──────────┘  │
│                          │                                   │
│                          ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  useInteractions([hover, focus, dismiss, role])         ││
│  │  → getReferenceProps(), getFloatingProps()              ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

## File Organization

### @tour-kit/core

```
packages/core/src/
├── context/
│   ├── beacon-context.ts      # Global beacon context
│   ├── beacon-provider.tsx    # Root provider
│   ├── tour-context.ts        # Tour-specific context
│   └── tour-provider.tsx      # Tour provider
├── hooks/
│   ├── use-tour.ts            # Main tour hook
│   ├── use-tour-state.ts      # Internal state management
│   ├── use-step.ts            # Individual step hook
│   ├── use-spotlight.ts       # Spotlight overlay
│   ├── use-keyboard.ts        # Keyboard navigation
│   ├── use-focus-trap.ts      # Focus trapping
│   ├── use-scroll.ts          # Scroll management
│   ├── use-persistence.ts     # Storage abstraction
│   ├── use-element-position.ts# Element rect tracking
│   └── use-media-query.ts     # Reduced motion detection
├── utils/
│   ├── create-tour.ts         # Tour factory
│   ├── create-step.ts         # Step factory
│   ├── position.ts            # Position calculations
│   ├── scroll.ts              # Scroll utilities
│   ├── dom.ts                 # DOM helpers
│   ├── storage.ts             # Storage abstraction
│   └── a11y.ts                # Accessibility helpers
├── types/
│   ├── index.ts               # Main exports
│   ├── tour.ts                # Tour types
│   ├── step.ts                # Step types
│   ├── config.ts              # Config types
│   └── state.ts               # State types
└── index.ts                   # Package entry
```

### @tour-kit/react

```
packages/react/src/
├── components/
│   ├── tour/
│   │   ├── tour.tsx           # Tour wrapper
│   │   ├── tour-step.tsx      # Step definition
│   │   └── index.ts
│   ├── card/
│   │   ├── tour-card.tsx      # Main tooltip
│   │   ├── tour-card-header.tsx
│   │   ├── tour-card-content.tsx
│   │   ├── tour-card-footer.tsx
│   │   └── index.ts
│   ├── overlay/
│   │   ├── tour-overlay.tsx   # Spotlight
│   │   └── index.ts
│   ├── navigation/
│   │   ├── tour-navigation.tsx
│   │   ├── tour-progress.tsx
│   │   ├── tour-close.tsx
│   │   └── index.ts
│   └── primitives/
│       ├── tour-arrow.tsx
│       ├── tour-portal.tsx
│       └── index.ts
├── styles/
│   └── globals.css
└── index.ts
```

### @tour-kit/hints

```
packages/hints/src/
├── context/
│   └── hints-provider.tsx
├── components/
│   ├── hint.tsx
│   ├── hint-beacon.tsx
│   └── hint-tooltip.tsx
├── hooks/
│   ├── use-hints.ts
│   └── use-hint.ts
├── types/
│   └── index.ts
└── index.ts
```

## Design Principles

### 1. Headless First

All logic lives in `@tour-kit/core`. Components in `@tour-kit/react` and `@tour-kit/hints` are thin wrappers.

### 2. Composition Over Configuration

Small, focused components that compose well together rather than one mega-component.

### 3. Progressive Enhancement

- Works without JS (graceful degradation)
- Respects `prefers-reduced-motion`
- Optional animations via Framer Motion

### 4. Type Safety

- Full TypeScript coverage
- Strict mode enabled
- Generic types for extensibility

### 5. Accessibility First

- ARIA attributes built-in
- Focus management
- Screen reader announcements
- Keyboard navigation
