# BeaconJS - Technical Briefing

## Project Overview

**BeaconJS** is a modern React onboarding and product tour library designed natively for Shadcn UI. It provides both headless hooks and pre-styled components for creating guided user experiences.

### Mission Statement

> The most developer-friendly, accessible, and Shadcn-native onboarding library for React.

### Key Differentiators

1. **Shadcn-native** - Uses Shadcn primitives, follows copy-paste philosophy
2. **Hints system** - Persistent UI beacons (unique in Shadcn ecosystem)
3. **Headless + Styled** - Full flexibility or quick start
4. **Accessible** - WCAG 2.1 AA compliant
5. **TypeScript-first** - Full type safety

---

## Tech Stack

| Category             | Tool                   | Version |
| -------------------- | ---------------------- | ------- |
| Runtime              | Node.js                | ≥18.0.0 |
| Package Manager      | pnpm                   | ≥9.0.0  |
| Language             | TypeScript             | ^5.5.0  |
| Bundler              | tsup                   | ^8.0.0  |
| Monorepo             | Turborepo              | ^2.0.0  |
| Testing              | Vitest                 | ^2.0.0  |
| Testing (Components) | @testing-library/react | ^16.0.0 |
| Linting/Formatting   | Biome                  | ^1.8.0  |
| Versioning           | Changesets             | ^2.27.0 |
| CI/CD                | GitHub Actions         | -       |
| Documentation        | Fumadocs               | ^14.0.0 |

### Core Dependencies

```json
{
  "dependencies": {
    "@floating-ui/react": "^0.26.0"
  },
  "peerDependencies": {
    "react": "^18.0.0 || ^19.0.0",
    "react-dom": "^18.0.0 || ^19.0.0"
  },
  "optionalPeerDependencies": {
    "framer-motion": "^11.0.0",
    "next": "^14.0.0 || ^15.0.0"
  }
}
```

---

## Architecture

### Package Structure

```
beaconjs/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── release.yml
├── apps/
│   ├── docs/                    # Fumadocs documentation site
│   │   ├── app/
│   │   ├── content/
│   │   └── package.json
│   └── demo/                    # Next.js demo/playground
│       ├── app/
│       ├── components/
│       └── package.json
├── packages/
│   ├── core/                    # @beaconjs/core
│   │   ├── src/
│   │   │   ├── context/
│   │   │   ├── hooks/
│   │   │   ├── types/
│   │   │   ├── utils/
│   │   │   └── index.ts
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── tsup.config.ts
│   ├── react/                   # @beaconjs/react (Shadcn components)
│   │   ├── src/
│   │   │   ├── components/
│   │   │   ├── styles/
│   │   │   └── index.ts
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── tsup.config.ts
│   └── hints/                   # @beaconjs/hints
│       ├── src/
│       │   ├── context/
│       │   ├── components/
│       │   └── index.ts
│       ├── package.json
│       ├── tsconfig.json
│       └── tsup.config.ts
├── tooling/
│   ├── tsconfig/                # Shared TypeScript configs
│   │   ├── base.json
│   │   ├── react-library.json
│   │   └── nextjs.json
│   └── biome/                   # Shared Biome config
│       └── biome.json
├── biome.json
├── package.json
├── pnpm-workspace.yaml
├── turbo.json
├── tsconfig.json
└── README.md
```

### Package Dependency Graph

```
@beaconjs/react
    └── @beaconjs/core

@beaconjs/hints
    └── @beaconjs/core

apps/demo
    ├── @beaconjs/core
    ├── @beaconjs/react
    └── @beaconjs/hints

apps/docs
    ├── @beaconjs/core
    ├── @beaconjs/react
    └── @beaconjs/hints
```

---

## Package Specifications

### @beaconjs/core

The headless core containing all logic, hooks, and utilities.

**Exports:**

```typescript
// Context
export { BeaconProvider } from './context/beacon-provider';
export { TourProvider } from './context/tour-provider';

// Hooks
export { useTour } from './hooks/use-tour';
export { useTourState } from './hooks/use-tour-state';
export { useStep } from './hooks/use-step';
export { useBeacon } from './hooks/use-beacon';
export { useSpotlight } from './hooks/use-spotlight';
export { useKeyboardNavigation } from './hooks/use-keyboard-navigation';
export { useFocusTrap } from './hooks/use-focus-trap';
export { usePersistence } from './hooks/use-persistence';

// Utilities
export { createTour } from './utils/create-tour';
export { createStep } from './utils/create-step';
export { scrollIntoView } from './utils/scroll';
export { getElementPosition } from './utils/position';

// Types
export type {
  Tour,
  TourStep,
  TourState,
  TourConfig,
  StepConfig,
  Position,
  Side,
  Alignment,
  SpotlightConfig,
  PersistenceConfig,
  BeaconConfig,
  KeyboardConfig,
  A11yConfig,
} from './types';
```

**Size Budget:** < 8KB gzipped

---

### @beaconjs/react

Pre-built Shadcn-styled components.

**Exports:**

```typescript
// Components
export { Tour } from './components/tour';
export { TourStep } from './components/tour-step';
export { TourCard } from './components/tour-card';
export { TourOverlay } from './components/tour-overlay';
export { TourProgress } from './components/tour-progress';
export { TourArrow } from './components/tour-arrow';
export { TourClose } from './components/tour-close';
export { TourNavigation } from './components/tour-navigation';

// Re-export core
export * from '@beaconjs/core';
```

**Size Budget:** < 12KB gzipped (including core)

---

### @beaconjs/hints

Standalone hints/hotspots system.

**Exports:**

```typescript
// Context
export { HintsProvider } from './context/hints-provider';

// Components
export { Hint } from './components/hint';
export { HintBeacon } from './components/hint-beacon';
export { HintTooltip } from './components/hint-tooltip';

// Hooks
export { useHints } from './hooks/use-hints';
export { useHint } from './hooks/use-hint';

// Re-export core types
export type { HintConfig, HintState } from './types';
```

**Size Budget:** < 5KB gzipped

---

## Type Definitions

### Core Types

```typescript
// packages/core/src/types/index.ts

// ============================================
// POSITIONING
// ============================================

export type Side = 'top' | 'bottom' | 'left' | 'right';

export type Alignment = 'start' | 'center' | 'end';

export type Placement = Side | `${Side}-${Alignment}`;

export interface Position {
  x: number;
  y: number;
}

export interface Rect {
  x: number;
  y: number;
  width: number;
  height: number;
}

// ============================================
// TOUR CONFIGURATION
// ============================================

export interface TourStep {
  /** Unique step identifier */
  id: string;

  /** Target element - CSS selector or React ref */
  target: string | React.RefObject<HTMLElement>;

  /** Step title */
  title?: React.ReactNode;

  /** Step content/description */
  content: React.ReactNode;

  /** Tooltip placement relative to target */
  placement?: Placement;

  /** Custom offset from target [x, y] */
  offset?: [number, number];

  /** Show/hide navigation buttons */
  showNavigation?: boolean;

  /** Show/hide close button */
  showClose?: boolean;

  /** Show/hide progress indicator */
  showProgress?: boolean;

  /** Custom CSS class for this step's card */
  className?: string;

  /** Spotlight padding around target */
  spotlightPadding?: number;

  /** Spotlight border radius */
  spotlightRadius?: number;

  /** Allow interaction with target element */
  interactive?: boolean;

  /** Auto-advance when condition met */
  advanceOn?: {
    event: 'click' | 'input' | 'custom';
    selector?: string;
    handler?: () => boolean;
  };

  /** Route to navigate to before showing step */
  route?: string;

  /** Delay after navigation before showing step (ms) */
  routeDelay?: number;

  /** Condition to show this step */
  when?: (context: TourContext) => boolean | Promise<boolean>;

  /** Wait for element to exist before showing */
  waitForTarget?: boolean;

  /** Timeout for waitForTarget (ms) */
  waitTimeout?: number;

  // Lifecycle hooks
  onBeforeShow?: (
    context: TourContext
  ) => void | boolean | Promise<void | boolean>;
  onShow?: (context: TourContext) => void;
  onBeforeHide?: (
    context: TourContext
  ) => void | boolean | Promise<void | boolean>;
  onHide?: (context: TourContext) => void;
}

export interface Tour {
  /** Unique tour identifier */
  id: string;

  /** Tour steps */
  steps: TourStep[];

  /** Start tour automatically when mounted */
  autoStart?: boolean;

  /** Start from specific step index */
  startAt?: number;

  /** Keyboard navigation config */
  keyboard?: KeyboardConfig | boolean;

  /** Spotlight/overlay config */
  spotlight?: SpotlightConfig | boolean;

  /** Persistence config */
  persistence?: PersistenceConfig | boolean;

  /** Accessibility config */
  a11y?: A11yConfig;

  /** Scroll behavior config */
  scroll?: ScrollConfig;

  // Lifecycle hooks
  onStart?: (context: TourContext) => void;
  onComplete?: (context: TourContext) => void;
  onSkip?: (context: TourContext) => void;
  onStepChange?: (step: TourStep, index: number, context: TourContext) => void;
}

// ============================================
// CONFIGURATION
// ============================================

export interface KeyboardConfig {
  /** Enable keyboard navigation */
  enabled?: boolean;

  /** Next step keys (default: ['ArrowRight', 'Enter']) */
  nextKeys?: string[];

  /** Previous step keys (default: ['ArrowLeft']) */
  prevKeys?: string[];

  /** Exit keys (default: ['Escape']) */
  exitKeys?: string[];

  /** Enable tab trapping within tooltip */
  trapFocus?: boolean;
}

export interface SpotlightConfig {
  /** Enable spotlight overlay */
  enabled?: boolean;

  /** Overlay color (default: 'rgba(0, 0, 0, 0.5)') */
  color?: string;

  /** Padding around target element */
  padding?: number;

  /** Border radius of spotlight cutout */
  borderRadius?: number;

  /** Animate spotlight movement */
  animate?: boolean;

  /** Animation duration (ms) */
  animationDuration?: number;

  /** Click overlay to exit */
  clickToExit?: boolean;
}

export interface PersistenceConfig {
  /** Enable persistence */
  enabled?: boolean;

  /** Storage type */
  storage?: 'localStorage' | 'sessionStorage' | 'cookie' | Storage;

  /** Storage key prefix */
  keyPrefix?: string;

  /** Remember current step on exit */
  rememberStep?: boolean;

  /** Track completed tours */
  trackCompleted?: boolean;

  /** "Don't show again" feature */
  dontShowAgain?: boolean;
}

export interface A11yConfig {
  /** Announce step changes to screen readers */
  announceSteps?: boolean;

  /** ARIA live region politeness */
  ariaLive?: 'polite' | 'assertive' | 'off';

  /** Enable focus trap */
  focusTrap?: boolean;

  /** Return focus to trigger element on exit */
  restoreFocus?: boolean;

  /** Respect prefers-reduced-motion */
  reducedMotion?: 'respect' | 'always-animate' | 'never-animate';
}

export interface ScrollConfig {
  /** Enable auto-scroll */
  enabled?: boolean;

  /** Scroll behavior */
  behavior?: 'auto' | 'smooth';

  /** Scroll alignment */
  block?: 'start' | 'center' | 'end' | 'nearest';

  /** Offset from viewport edge */
  offset?: number;
}

// ============================================
// STATE
// ============================================

export interface TourState {
  /** Current tour ID */
  tourId: string | null;

  /** Is tour currently active */
  isActive: boolean;

  /** Current step index */
  currentStepIndex: number;

  /** Current step data */
  currentStep: TourStep | null;

  /** Total steps count */
  totalSteps: number;

  /** Tour is loading (waiting for element, route, etc.) */
  isLoading: boolean;

  /** Tour is transitioning between steps */
  isTransitioning: boolean;

  /** Completed tour IDs */
  completedTours: string[];

  /** Skipped tour IDs */
  skippedTours: string[];
}

export interface TourContext extends TourState {
  /** Current tour config */
  tour: Tour | null;

  /** Custom context data */
  data: Record<string, unknown>;
}

// ============================================
// ACTIONS
// ============================================

export interface TourActions {
  /** Start a tour */
  start: (tourId?: string, stepIndex?: number) => void;

  /** Go to next step */
  next: () => void;

  /** Go to previous step */
  prev: () => void;

  /** Go to specific step */
  goTo: (stepIndex: number) => void;

  /** Skip/exit tour */
  skip: () => void;

  /** Complete tour */
  complete: () => void;

  /** Stop tour without marking complete/skipped */
  stop: () => void;

  /** Set "don't show again" for tour */
  setDontShowAgain: (tourId: string, value: boolean) => void;

  /** Reset tour progress */
  reset: (tourId?: string) => void;

  /** Update custom context data */
  setData: (key: string, value: unknown) => void;
}

// ============================================
// HINTS
// ============================================

export interface HintConfig {
  /** Unique hint identifier */
  id: string;

  /** Target element - CSS selector or React ref */
  target: string | React.RefObject<HTMLElement>;

  /** Hint content */
  content: React.ReactNode;

  /** Beacon position relative to target */
  position?:
    | 'top-left'
    | 'top-right'
    | 'bottom-left'
    | 'bottom-right'
    | 'center';

  /** Tooltip placement when opened */
  tooltipPlacement?: Placement;

  /** Show pulsing animation */
  pulse?: boolean;

  /** Auto-show tooltip on mount */
  autoShow?: boolean;

  /** Persist dismissal */
  persist?: boolean;

  // Lifecycle
  onClick?: () => void;
  onShow?: () => void;
  onDismiss?: () => void;
}

export interface HintState {
  /** Hint ID */
  id: string;

  /** Is tooltip visible */
  isOpen: boolean;

  /** Has been dismissed */
  isDismissed: boolean;
}
```

---

## Component API Design

### BeaconProvider (Root Provider)

```tsx
import { BeaconProvider } from '@beaconjs/react';

<BeaconProvider
  // Global configuration
  config={{
    keyboard: { enabled: true, trapFocus: true },
    spotlight: { color: 'rgba(0,0,0,0.75)', padding: 8 },
    persistence: { storage: 'localStorage', keyPrefix: 'beacon' },
    a11y: { announceSteps: true, focusTrap: true },
    scroll: { behavior: 'smooth', block: 'center' },
  }}
  // Global callbacks
  onTourStart={(tourId) => analytics.track('tour_start', { tourId })}
  onTourComplete={(tourId) => analytics.track('tour_complete', { tourId })}
  onTourSkip={(tourId, stepIndex) =>
    analytics.track('tour_skip', { tourId, stepIndex })
  }
  onStepView={(tourId, step, index) =>
    analytics.track('step_view', { tourId, step: step.id })
  }
>
  <App />
</BeaconProvider>;
```

### Tour Component

```tsx
import { Tour, TourStep } from '@beaconjs/react';

<Tour
  id='onboarding'
  autoStart={isFirstVisit}
  onComplete={() => setOnboardingComplete(true)}
>
  <TourStep
    target='#welcome-button'
    title='Welcome to BeaconJS!'
    content="Let's take a quick tour of the main features."
    placement='bottom'
  />

  <TourStep
    target='#dashboard'
    title='Your Dashboard'
    content="This is where you'll find all your important data."
    placement='right'
    spotlightPadding={16}
  />

  <TourStep
    target={settingsRef}
    title='Settings'
    content='Customize your experience here.'
    placement='left'
    when={(ctx) => ctx.data.userRole === 'admin'}
  />

  <TourStep
    target='#save-button'
    title='Try it out!'
    content='Click the save button to continue.'
    interactive
    advanceOn={{ event: 'click' }}
  />
</Tour>;
```

### Headless Usage

```tsx
import { useTour, TourProvider } from '@beaconjs/core';

function CustomTourUI() {
  const {
    isActive,
    currentStep,
    currentStepIndex,
    totalSteps,
    next,
    prev,
    skip,
    complete,
  } = useTour();

  if (!isActive || !currentStep) return null;

  return (
    <MyCustomTooltip
      target={currentStep.target}
      placement={currentStep.placement}
    >
      <h3>{currentStep.title}</h3>
      <p>{currentStep.content}</p>
      <div>
        <span>
          {currentStepIndex + 1} of {totalSteps}
        </span>
        <button onClick={prev}>Back</button>
        <button onClick={next}>Next</button>
        <button onClick={skip}>Skip</button>
      </div>
    </MyCustomTooltip>
  );
}

function App() {
  return (
    <TourProvider
      tours={[
        {
          id: 'onboarding',
          steps: [{ id: 'step-1', target: '#btn', content: 'Hello!' }],
        },
      ]}
    >
      <CustomTourUI />
      <MainApp />
    </TourProvider>
  );
}
```

### Hints Component

```tsx
import { HintsProvider, Hint } from '@beaconjs/hints';

<HintsProvider>
  <Hint
    id='new-feature'
    target='#feature-button'
    position='top-right'
    pulse
    persist
  >
    <p>New feature available! Click to learn more.</p>
  </Hint>

  <Hint
    id='pro-tip'
    target='#search-input'
    position='bottom-right'
    content='Pro tip: Use keyboard shortcuts for faster navigation.'
  />
</HintsProvider>;
```

---

## Hook Specifications

### useTour

```typescript
function useTour(tourId?: string): {
  // State
  isActive: boolean;
  isLoading: boolean;
  isTransitioning: boolean;
  currentStep: TourStep | null;
  currentStepIndex: number;
  totalSteps: number;
  isFirstStep: boolean;
  isLastStep: boolean;
  progress: number; // 0-1

  // Actions
  start: (stepIndex?: number) => void;
  next: () => void;
  prev: () => void;
  goTo: (stepIndex: number) => void;
  skip: () => void;
  complete: () => void;
  stop: () => void;

  // Utilities
  isStepActive: (stepId: string) => boolean;
  getStep: (stepId: string) => TourStep | undefined;
};
```

### useStep

```typescript
function useStep(stepId: string): {
  // State
  isActive: boolean;
  isVisible: boolean;
  hasCompleted: boolean;

  // Target element info
  targetElement: HTMLElement | null;
  targetRect: DOMRect | null;

  // Actions
  show: () => void;
  hide: () => void;
  complete: () => void;
};
```

### useSpotlight

```typescript
function useSpotlight(): {
  // State
  isVisible: boolean;
  targetRect: DOMRect | null;

  // Computed styles
  overlayStyle: React.CSSProperties;
  cutoutStyle: React.CSSProperties;

  // Actions
  show: (target: HTMLElement, config?: SpotlightConfig) => void;
  hide: () => void;
  update: () => void;
};
```

### useHints

```typescript
function useHints(): {
  // State
  hints: HintState[];
  activeHint: string | null;

  // Actions
  showHint: (hintId: string) => void;
  hideHint: (hintId: string) => void;
  dismissHint: (hintId: string) => void;
  resetHint: (hintId: string) => void;
  resetAllHints: () => void;

  // Utilities
  isHintVisible: (hintId: string) => boolean;
  isHintDismissed: (hintId: string) => boolean;
};
```

---

## File Structure Detail

### packages/core/src/

```
src/
├── context/
│   ├── beacon-context.ts        # Main context definition
│   ├── beacon-provider.tsx      # Root provider component
│   ├── tour-context.ts          # Tour-specific context
│   └── tour-provider.tsx        # Tour provider component
├── hooks/
│   ├── use-tour.ts              # Main tour hook
│   ├── use-tour-state.ts        # Internal state management
│   ├── use-step.ts              # Individual step hook
│   ├── use-spotlight.ts         # Spotlight overlay hook
│   ├── use-keyboard.ts          # Keyboard navigation
│   ├── use-focus-trap.ts        # Focus trapping
│   ├── use-scroll.ts            # Scroll management
│   ├── use-persistence.ts       # localStorage/session
│   ├── use-element-position.ts  # Element rect tracking
│   └── use-media-query.ts       # Reduced motion, etc.
├── utils/
│   ├── create-tour.ts           # Tour factory
│   ├── create-step.ts           # Step factory
│   ├── position.ts              # Position calculations
│   ├── scroll.ts                # Scroll utilities
│   ├── dom.ts                   # DOM helpers
│   ├── storage.ts               # Storage abstraction
│   └── a11y.ts                  # Accessibility helpers
├── types/
│   ├── index.ts                 # Main type exports
│   ├── tour.ts                  # Tour types
│   ├── step.ts                  # Step types
│   ├── config.ts                # Config types
│   └── state.ts                 # State types
└── index.ts                     # Package entry point
```

### packages/react/src/

```
src/
├── components/
│   ├── tour/
│   │   ├── tour.tsx             # Tour wrapper component
│   │   ├── tour-step.tsx        # Step definition component
│   │   └── index.ts
│   ├── card/
│   │   ├── tour-card.tsx        # Main tooltip card
│   │   ├── tour-card-header.tsx
│   │   ├── tour-card-content.tsx
│   │   ├── tour-card-footer.tsx
│   │   └── index.ts
│   ├── overlay/
│   │   ├── tour-overlay.tsx     # Spotlight overlay
│   │   └── index.ts
│   ├── navigation/
│   │   ├── tour-navigation.tsx  # Next/Prev buttons
│   │   ├── tour-progress.tsx    # Progress bar/dots
│   │   ├── tour-close.tsx       # Close button
│   │   └── index.ts
│   ├── primitives/
│   │   ├── tour-arrow.tsx       # Tooltip arrow
│   │   ├── tour-portal.tsx      # Portal wrapper
│   │   └── index.ts
│   └── index.ts
├── styles/
│   └── globals.css              # Optional base styles
└── index.ts                     # Package entry point
```

---

## Configuration Files

### Root package.json

```json
{
  "name": "beaconjs",
  "private": true,
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "test": "turbo test",
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "format": "biome format --write .",
    "typecheck": "turbo typecheck",
    "clean": "turbo clean && rm -rf node_modules",
    "changeset": "changeset",
    "version-packages": "changeset version",
    "release": "turbo build --filter=./packages/* && changeset publish"
  },
  "devDependencies": {
    "@biomejs/biome": "^1.8.0",
    "@changesets/cli": "^2.27.0",
    "turbo": "^2.0.0",
    "typescript": "^5.5.0"
  },
  "packageManager": "pnpm@9.0.0",
  "engines": {
    "node": ">=18.0.0"
  }
}
```

### pnpm-workspace.yaml

```yaml
packages:
  - 'packages/*'
  - 'apps/*'
  - 'tooling/*'
```

### turbo.json

```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"]
    },
    "typecheck": {
      "dependsOn": ["^build"]
    },
    "lint": {
      "dependsOn": ["^build"]
    },
    "clean": {
      "cache": false
    }
  }
}
```

### biome.json (root)

```json
{
  "$schema": "https://biomejs.dev/schemas/1.8.0/schema.json",
  "extends": ["./tooling/biome/biome.json"],
  "files": {
    "ignore": ["**/dist/**", "**/node_modules/**", "**/.next/**"]
  }
}
```

### tooling/biome/biome.json

```json
{
  "$schema": "https://biomejs.dev/schemas/1.8.0/schema.json",
  "organizeImports": {
    "enabled": true
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "complexity": {
        "noExcessiveCognitiveComplexity": "warn"
      },
      "correctness": {
        "noUnusedImports": "error",
        "noUnusedVariables": "error"
      },
      "style": {
        "noNonNullAssertion": "warn",
        "useConst": "error"
      },
      "suspicious": {
        "noExplicitAny": "warn"
      }
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "trailingCommas": "es5",
      "semicolons": "asNeeded"
    }
  }
}
```

### packages/core/package.json

```json
{
  "name": "@beaconjs/core",
  "version": "0.0.1",
  "description": "Headless onboarding and product tour library for React",
  "author": "Your Name",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/yourusername/beaconjs.git",
    "directory": "packages/core"
  },
  "keywords": [
    "react",
    "onboarding",
    "tour",
    "product-tour",
    "walkthrough",
    "hints",
    "tooltip",
    "headless"
  ],
  "type": "module",
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": {
        "types": "./dist/index.d.ts",
        "default": "./dist/index.js"
      },
      "require": {
        "types": "./dist/index.d.cts",
        "default": "./dist/index.cjs"
      }
    }
  },
  "files": ["dist", "README.md"],
  "sideEffects": false,
  "scripts": {
    "build": "tsup",
    "dev": "tsup --watch",
    "test": "vitest",
    "test:coverage": "vitest --coverage",
    "typecheck": "tsc --noEmit",
    "clean": "rm -rf dist"
  },
  "dependencies": {
    "@floating-ui/react": "^0.26.0"
  },
  "peerDependencies": {
    "react": "^18.0.0 || ^19.0.0",
    "react-dom": "^18.0.0 || ^19.0.0"
  },
  "devDependencies": {
    "@testing-library/react": "^16.0.0",
    "@types/react": "^18.3.0",
    "@types/react-dom": "^18.3.0",
    "jsdom": "^24.0.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "tsup": "^8.0.0",
    "typescript": "^5.5.0",
    "vitest": "^2.0.0"
  }
}
```

### packages/core/tsup.config.ts

```typescript
import { defineConfig } from 'tsup';

export default defineConfig({
  entry: ['src/index.ts'],
  format: ['cjs', 'esm'],
  dts: true,
  clean: true,
  external: ['react', 'react-dom'],
  treeshake: true,
  splitting: false,
  minify: true,
  sourcemap: true,
  target: 'es2020',
  outDir: 'dist',
  esbuildOptions(options) {
    options.banner = {
      js: '"use client";',
    };
  },
});
```

### packages/core/tsconfig.json

```json
{
  "extends": "../../tooling/tsconfig/react-library.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts", "**/*.test.tsx"]
}
```

### tooling/tsconfig/react-library.json

```json
{
  "extends": "./base.json",
  "compilerOptions": {
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "declaration": true,
    "declarationMap": true,
    "noEmit": false
  }
}
```

### tooling/tsconfig/base.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "allowSyntheticDefaultImports": true
  }
}
```

### packages/core/vitest.config.ts

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.{test,spec}.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'src/test/'],
    },
  },
});
```

---

## CI/CD Configuration

### .github/workflows/ci.yml

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v3
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Lint
        run: pnpm lint

      - name: Type check
        run: pnpm typecheck

      - name: Build
        run: pnpm build

      - name: Test
        run: pnpm test

  size:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v3
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile
      - run: pnpm build

      - name: Check bundle size
        uses: preactjs/compressed-size-action@v2
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          pattern: './packages/*/dist/**/*.js'
```

### .github/workflows/release.yml

```yaml
name: Release

on:
  push:
    branches: [main]

concurrency: ${{ github.workflow }}-${{ github.ref }}

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v3
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
          registry-url: 'https://registry.npmjs.org'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build
        run: pnpm build

      - name: Create Release Pull Request or Publish
        id: changesets
        uses: changesets/action@v1
        with:
          publish: pnpm release
          version: pnpm version-packages
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## Development Workflow

### Initial Setup

```bash
# Clone repository
git clone https://github.com/yourusername/beaconjs.git
cd beaconjs

# Install dependencies
pnpm install

# Start development
pnpm dev
```

### Development Commands

```bash
# Run all packages in dev mode
pnpm dev

# Build all packages
pnpm build

# Run tests
pnpm test

# Run tests with coverage
pnpm test:coverage

# Lint and format
pnpm lint
pnpm lint:fix

# Type check
pnpm typecheck

# Clean all build artifacts
pnpm clean
```

### Creating a Release

```bash
# Create a changeset
pnpm changeset

# Version packages based on changesets
pnpm version-packages

# Publish to npm
pnpm release
```

---

## Implementation Roadmap

### Phase 1: Foundation (Week 1-2)

- [ ] Repository setup with Turborepo
- [ ] TypeScript configuration
- [ ] Biome linting setup
- [ ] CI/CD pipelines
- [ ] Core types definition
- [ ] BeaconProvider context
- [ ] useTour hook (basic)
- [ ] useTourState internal hook

### Phase 2: Core Features (Week 2-3)

- [ ] Floating UI integration
- [ ] useSpotlight hook
- [ ] Overlay component
- [ ] Keyboard navigation
- [ ] Basic TourCard component
- [ ] TourStep component
- [ ] Tour component

### Phase 3: Polish (Week 3-4)

- [ ] Persistence layer
- [ ] Progress indicators
- [ ] Animation integration
- [ ] Focus trapping
- [ ] Screen reader support
- [ ] Route navigation support

### Phase 4: Hints System (Week 4-5)

- [ ] HintsProvider context
- [ ] useHints hook
- [ ] Hint component
- [ ] HintBeacon (pulsing dot)
- [ ] HintTooltip
- [ ] Hints persistence

### Phase 5: Documentation & Release (Week 5-6)

- [ ] Fumadocs setup
- [ ] API documentation
- [ ] Usage examples
- [ ] Demo site
- [ ] Initial release (v0.1.0)

---

## Success Metrics

| Metric                        | Target         |
| ----------------------------- | -------------- |
| Bundle size (@beaconjs/core)  | < 8KB gzipped  |
| Bundle size (@beaconjs/react) | < 12KB gzipped |
| Bundle size (@beaconjs/hints) | < 5KB gzipped  |
| Lighthouse Accessibility      | 100            |
| Test coverage                 | > 80%          |
| TypeScript strict mode        | Enabled        |
| Zero runtime dependencies\*   | ✓              |

\*Excluding @floating-ui/react and peer dependencies

---

## License

MIT License - Open source for personal and commercial use.

---

## Summary

BeaconJS will be built as a modern, TypeScript-first monorepo with three packages:

1. **@beaconjs/core** - Headless hooks and logic
2. **@beaconjs/react** - Shadcn-styled components
3. **@beaconjs/hints** - Persistent hints/hotspots

The stack prioritizes developer experience (pnpm, tsup, Vitest, Biome) while maintaining production quality (CI/CD, changesets, comprehensive types).

Key architectural decisions:

- Headless-first approach enables full customization
- Floating UI for reliable positioning
- Context-based state for clean API
- Optional Framer Motion for animations
- Progressive enhancement for accessibility
