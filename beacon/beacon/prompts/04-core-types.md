# Prompt 04: Core Types Definition

## Problem

Define all TypeScript types for `@tour-kit/core`. These types are the foundation for the entire library.

## Dependencies on Previous Prompts

- Prompts 01-03 complete (monorepo and tooling set up)

## Supporting Information

### Target File Structure

```
packages/core/
├── src/
│   ├── types/
│   │   ├── index.ts
│   │   ├── tour.ts
│   │   ├── step.ts
│   │   ├── config.ts
│   │   ├── state.ts
│   │   └── hints.ts
│   └── index.ts
├── package.json
├── tsconfig.json
└── tsup.config.ts
```

### Reference Specification

See `../specs/core-types.md` for full type definitions.

## Steps To Complete

### Step 1: Create core package.json

Create `packages/core/package.json`:

```json
{
  "name": "@tour-kit/core",
  "version": "0.0.1",
  "description": "Headless onboarding and product tour library for React",
  "author": "TourKit",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/your-org/tour-kit.git",
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
    "@tour-kit/tsconfig": "workspace:*",
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

### Step 2: Create tsconfig.json

Create `packages/core/tsconfig.json`:

```json
{
  "extends": "@tour-kit/tsconfig/react-library.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts", "**/*.test.tsx"]
}
```

### Step 3: Create tsup.config.ts

Create `packages/core/tsup.config.ts`:

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

### Step 4: Create config types

Create `packages/core/src/types/config.ts`:

```typescript
/**
 * Side of target element for positioning
 */
export type Side = 'top' | 'bottom' | 'left' | 'right';

/**
 * Alignment along the side
 */
export type Alignment = 'start' | 'center' | 'end';

/**
 * Combined placement (side + optional alignment)
 */
export type Placement = Side | `${Side}-${Alignment}`;

/**
 * Point coordinates
 */
export interface Position {
  x: number;
  y: number;
}

/**
 * Rectangle dimensions
 */
export interface Rect {
  x: number;
  y: number;
  width: number;
  height: number;
}

/**
 * Keyboard navigation configuration
 */
export interface KeyboardConfig {
  enabled?: boolean;
  nextKeys?: string[];
  prevKeys?: string[];
  exitKeys?: string[];
  trapFocus?: boolean;
}

/**
 * Spotlight/overlay configuration
 */
export interface SpotlightConfig {
  enabled?: boolean;
  color?: string;
  padding?: number;
  borderRadius?: number;
  animate?: boolean;
  animationDuration?: number;
  clickToExit?: boolean;
}

/**
 * Storage interface for custom adapters
 */
export interface Storage {
  getItem: (key: string) => string | null | Promise<string | null>;
  setItem: (key: string, value: string) => void | Promise<void>;
  removeItem: (key: string) => void | Promise<void>;
}

/**
 * Persistence configuration
 */
export interface PersistenceConfig {
  enabled?: boolean;
  storage?: 'localStorage' | 'sessionStorage' | 'cookie' | Storage;
  keyPrefix?: string;
  rememberStep?: boolean;
  trackCompleted?: boolean;
  dontShowAgain?: boolean;
}

/**
 * Accessibility configuration
 */
export interface A11yConfig {
  announceSteps?: boolean;
  ariaLive?: 'polite' | 'assertive' | 'off';
  focusTrap?: boolean;
  restoreFocus?: boolean;
  reducedMotion?: 'respect' | 'always-animate' | 'never-animate';
}

/**
 * Scroll behavior configuration
 */
export interface ScrollConfig {
  enabled?: boolean;
  behavior?: 'auto' | 'smooth';
  block?: 'start' | 'center' | 'end' | 'nearest';
  offset?: number;
}

/**
 * Global beacon configuration
 */
export interface BeaconConfig {
  keyboard?: KeyboardConfig;
  spotlight?: SpotlightConfig;
  persistence?: PersistenceConfig;
  a11y?: A11yConfig;
  scroll?: ScrollConfig;
}

// Default configurations
export const defaultKeyboardConfig: Required<KeyboardConfig> = {
  enabled: true,
  nextKeys: ['ArrowRight', 'Enter'],
  prevKeys: ['ArrowLeft'],
  exitKeys: ['Escape'],
  trapFocus: true,
};

export const defaultSpotlightConfig: Required<SpotlightConfig> = {
  enabled: true,
  color: 'rgba(0, 0, 0, 0.5)',
  padding: 8,
  borderRadius: 4,
  animate: true,
  animationDuration: 300,
  clickToExit: false,
};

export const defaultPersistenceConfig: Required<
  Omit<PersistenceConfig, 'storage'>
> & {
  storage: 'localStorage';
} = {
  enabled: true,
  storage: 'localStorage',
  keyPrefix: 'beacon',
  rememberStep: true,
  trackCompleted: true,
  dontShowAgain: false,
};

export const defaultA11yConfig: Required<A11yConfig> = {
  announceSteps: true,
  ariaLive: 'polite',
  focusTrap: true,
  restoreFocus: true,
  reducedMotion: 'respect',
};

export const defaultScrollConfig: Required<ScrollConfig> = {
  enabled: true,
  behavior: 'smooth',
  block: 'center',
  offset: 20,
};
```

### Step 5: Create step types

Create `packages/core/src/types/step.ts`:

```typescript
import type React from 'react';
import type { Placement } from './config';
import type { TourContext } from './state';

/**
 * Single step in a tour
 */
export interface TourStep {
  id: string;
  target: string | React.RefObject<HTMLElement | null>;
  title?: React.ReactNode;
  content: React.ReactNode;
  placement?: Placement;
  offset?: [number, number];
  showNavigation?: boolean;
  showClose?: boolean;
  showProgress?: boolean;
  className?: string;
  spotlightPadding?: number;
  spotlightRadius?: number;
  interactive?: boolean;
  advanceOn?: {
    event: 'click' | 'input' | 'custom';
    selector?: string;
    handler?: () => boolean;
  };
  route?: string;
  routeDelay?: number;
  when?: (context: TourContext) => boolean | Promise<boolean>;
  waitForTarget?: boolean;
  waitTimeout?: number;
  onBeforeShow?: (
    context: TourContext
  ) => void | boolean | Promise<void | boolean>;
  onShow?: (context: TourContext) => void;
  onBeforeHide?: (
    context: TourContext
  ) => void | boolean | Promise<void | boolean>;
  onHide?: (context: TourContext) => void;
}

export type StepOptions = Omit<TourStep, 'id'>;
```

### Step 6: Create tour types

Create `packages/core/src/types/tour.ts`:

```typescript
import type { TourStep } from './step';
import type {
  KeyboardConfig,
  SpotlightConfig,
  PersistenceConfig,
  A11yConfig,
  ScrollConfig,
} from './config';
import type { TourContext } from './state';

/**
 * Tour definition
 */
export interface Tour {
  id: string;
  steps: TourStep[];
  autoStart?: boolean;
  startAt?: number;
  keyboard?: KeyboardConfig | boolean;
  spotlight?: SpotlightConfig | boolean;
  persistence?: PersistenceConfig | boolean;
  a11y?: A11yConfig;
  scroll?: ScrollConfig;
  onStart?: (context: TourContext) => void;
  onComplete?: (context: TourContext) => void;
  onSkip?: (context: TourContext) => void;
  onStepChange?: (step: TourStep, index: number, context: TourContext) => void;
}

export type TourOptions = Omit<Tour, 'id' | 'steps'>;
```

### Step 7: Create state types

Create `packages/core/src/types/state.ts`:

```typescript
import type { Tour } from './tour';
import type { TourStep } from './step';

/**
 * Current tour state
 */
export interface TourState {
  tourId: string | null;
  isActive: boolean;
  currentStepIndex: number;
  currentStep: TourStep | null;
  totalSteps: number;
  isLoading: boolean;
  isTransitioning: boolean;
  completedTours: string[];
  skippedTours: string[];
}

/**
 * Extended tour context
 */
export interface TourContext extends TourState {
  tour: Tour | null;
  data: Record<string, unknown>;
}

/**
 * Tour action methods
 */
export interface TourActions {
  start: (tourId?: string, stepIndex?: number) => void;
  next: () => void;
  prev: () => void;
  goTo: (stepIndex: number) => void;
  skip: () => void;
  complete: () => void;
  stop: () => void;
  setDontShowAgain: (tourId: string, value: boolean) => void;
  reset: (tourId?: string) => void;
  setData: (key: string, value: unknown) => void;
}

/**
 * Combined context value
 */
export interface TourContextValue extends TourState, TourActions {
  tour: Tour | null;
  data: Record<string, unknown>;
}

/**
 * Initial state
 */
export const initialTourState: TourState = {
  tourId: null,
  isActive: false,
  currentStepIndex: 0,
  currentStep: null,
  totalSteps: 0,
  isLoading: false,
  isTransitioning: false,
  completedTours: [],
  skippedTours: [],
};
```

### Step 8: Create hints types

Create `packages/core/src/types/hints.ts`:

```typescript
import type React from 'react';
import type { Placement } from './config';

export type BeaconPosition =
  | 'top-left'
  | 'top-right'
  | 'bottom-left'
  | 'bottom-right'
  | 'center';

export interface HintConfig {
  id: string;
  target: string | React.RefObject<HTMLElement | null>;
  content: React.ReactNode;
  position?: BeaconPosition;
  tooltipPlacement?: Placement;
  pulse?: boolean;
  autoShow?: boolean;
  persist?: boolean;
  onClick?: () => void;
  onShow?: () => void;
  onDismiss?: () => void;
}

export interface HintState {
  id: string;
  isOpen: boolean;
  isDismissed: boolean;
}

export interface HintsState {
  hints: Map<string, HintState>;
  activeHint: string | null;
}

export interface HintsActions {
  registerHint: (id: string) => void;
  unregisterHint: (id: string) => void;
  showHint: (id: string) => void;
  hideHint: (id: string) => void;
  dismissHint: (id: string) => void;
  resetHint: (id: string) => void;
  resetAllHints: () => void;
}

export interface HintsContextValue extends HintsState, HintsActions {}
```

### Step 9: Create types index

Create `packages/core/src/types/index.ts`:

```typescript
export * from './config';
export * from './step';
export * from './tour';
export * from './state';
export * from './hints';
```

### Step 10: Create package entry point

Create `packages/core/src/index.ts`:

```typescript
// Types
export * from './types';
```

## Key Requirements & Constraints

- [ ] All types use proper JSDoc comments
- [ ] Types are properly exported from index
- [ ] No circular dependencies between type files
- [ ] Default configurations are provided
- [ ] Package builds without errors
- [ ] Types are strict (no implicit any)

## Verification

```bash
cd packages/core
pnpm install
pnpm typecheck
pnpm build
```

## Next Prompt

Continue with `05-context-setup.md` to create React contexts.
