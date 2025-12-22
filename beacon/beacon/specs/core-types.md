# Core Types Specification

## Overview

This document specifies all TypeScript types for `@tour-kit/core`. These types form the foundation for the entire library.

## File: `packages/core/src/types/index.ts`

```typescript
// Re-export all types
export * from './tour';
export * from './step';
export * from './config';
export * from './state';
export * from './hints';
```

## File: `packages/core/src/types/tour.ts`

```typescript
import type { TourStep } from './step';
import type { KeyboardConfig, SpotlightConfig, PersistenceConfig, A11yConfig, ScrollConfig } from './config';
import type { TourContext } from './state';

/**
 * Tour definition containing all steps and configuration
 */
export interface Tour {
  /** Unique tour identifier */
  id: string;

  /** Tour steps in order */
  steps: TourStep[];

  /** Start tour automatically when provider mounts */
  autoStart?: boolean;

  /** Index of step to start from (default: 0) */
  startAt?: number;

  /** Keyboard navigation configuration */
  keyboard?: KeyboardConfig | boolean;

  /** Spotlight/overlay configuration */
  spotlight?: SpotlightConfig | boolean;

  /** Persistence configuration */
  persistence?: PersistenceConfig | boolean;

  /** Accessibility configuration */
  a11y?: A11yConfig;

  /** Scroll behavior configuration */
  scroll?: ScrollConfig;

  // Lifecycle callbacks
  onStart?: (context: TourContext) => void;
  onComplete?: (context: TourContext) => void;
  onSkip?: (context: TourContext) => void;
  onStepChange?: (step: TourStep, index: number, context: TourContext) => void;
}

/**
 * Options for creating a tour programmatically
 */
export type TourOptions = Omit<Tour, 'id' | 'steps'>;
```

## File: `packages/core/src/types/step.ts`

```typescript
import type React from 'react';
import type { Placement } from './config';
import type { TourContext } from './state';

/**
 * Single step in a tour
 */
export interface TourStep {
  /** Unique step identifier */
  id: string;

  /** Target element - CSS selector or React ref */
  target: string | React.RefObject<HTMLElement | null>;

  /** Step title (optional) */
  title?: React.ReactNode;

  /** Step content/description (required) */
  content: React.ReactNode;

  /** Tooltip placement relative to target */
  placement?: Placement;

  /** Custom offset from target [x, y] in pixels */
  offset?: [number, number];

  /** Show navigation buttons (prev/next) */
  showNavigation?: boolean;

  /** Show close button */
  showClose?: boolean;

  /** Show progress indicator */
  showProgress?: boolean;

  /** Custom CSS class for this step's card */
  className?: string;

  /** Spotlight padding around target (pixels) */
  spotlightPadding?: number;

  /** Spotlight border radius (pixels) */
  spotlightRadius?: number;

  /** Allow interaction with target element through overlay */
  interactive?: boolean;

  /** Auto-advance configuration */
  advanceOn?: {
    /** Event type to listen for */
    event: 'click' | 'input' | 'custom';
    /** Selector for event target (defaults to step target) */
    selector?: string;
    /** Custom handler returning true to advance */
    handler?: () => boolean;
  };

  /** Route to navigate to before showing step */
  route?: string;

  /** Delay after navigation before showing step (ms) */
  routeDelay?: number;

  /** Condition to show this step */
  when?: (context: TourContext) => boolean | Promise<boolean>;

  /** Wait for target element to exist before showing */
  waitForTarget?: boolean;

  /** Timeout for waitForTarget (ms, default: 5000) */
  waitTimeout?: number;

  // Lifecycle hooks
  /** Called before step is shown, return false to prevent */
  onBeforeShow?: (context: TourContext) => void | boolean | Promise<void | boolean>;
  /** Called after step is shown */
  onShow?: (context: TourContext) => void;
  /** Called before step is hidden, return false to prevent */
  onBeforeHide?: (context: TourContext) => void | boolean | Promise<void | boolean>;
  /** Called after step is hidden */
  onHide?: (context: TourContext) => void;
}

/**
 * Options for creating a step programmatically
 */
export type StepOptions = Omit<TourStep, 'id'>;
```

## File: `packages/core/src/types/config.ts`

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
  /** Enable keyboard navigation (default: true) */
  enabled?: boolean;

  /** Keys to advance to next step (default: ['ArrowRight', 'Enter']) */
  nextKeys?: string[];

  /** Keys to go to previous step (default: ['ArrowLeft']) */
  prevKeys?: string[];

  /** Keys to exit tour (default: ['Escape']) */
  exitKeys?: string[];

  /** Trap focus within tooltip (default: true) */
  trapFocus?: boolean;
}

/**
 * Spotlight/overlay configuration
 */
export interface SpotlightConfig {
  /** Enable spotlight overlay (default: true) */
  enabled?: boolean;

  /** Overlay background color (default: 'rgba(0, 0, 0, 0.5)') */
  color?: string;

  /** Padding around target element (default: 8) */
  padding?: number;

  /** Border radius of spotlight cutout (default: 4) */
  borderRadius?: number;

  /** Animate spotlight movement (default: true) */
  animate?: boolean;

  /** Animation duration in ms (default: 300) */
  animationDuration?: number;

  /** Click overlay to exit tour (default: false) */
  clickToExit?: boolean;
}

/**
 * Storage interface for custom storage adapters
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
  /** Enable persistence (default: true) */
  enabled?: boolean;

  /** Storage type or custom storage adapter */
  storage?: 'localStorage' | 'sessionStorage' | 'cookie' | Storage;

  /** Key prefix for storage (default: 'beacon') */
  keyPrefix?: string;

  /** Remember current step on exit (default: true) */
  rememberStep?: boolean;

  /** Track completed tours (default: true) */
  trackCompleted?: boolean;

  /** Enable "Don't show again" feature (default: false) */
  dontShowAgain?: boolean;
}

/**
 * Accessibility configuration
 */
export interface A11yConfig {
  /** Announce step changes to screen readers (default: true) */
  announceSteps?: boolean;

  /** ARIA live region politeness (default: 'polite') */
  ariaLive?: 'polite' | 'assertive' | 'off';

  /** Enable focus trap (default: true) */
  focusTrap?: boolean;

  /** Return focus to trigger element on exit (default: true) */
  restoreFocus?: boolean;

  /** How to handle prefers-reduced-motion (default: 'respect') */
  reducedMotion?: 'respect' | 'always-animate' | 'never-animate';
}

/**
 * Scroll behavior configuration
 */
export interface ScrollConfig {
  /** Enable auto-scroll to target (default: true) */
  enabled?: boolean;

  /** Scroll behavior (default: 'smooth') */
  behavior?: 'auto' | 'smooth';

  /** Scroll alignment (default: 'center') */
  block?: 'start' | 'center' | 'end' | 'nearest';

  /** Offset from viewport edge (default: 20) */
  offset?: number;
}

/**
 * Global beacon configuration (merged with tour-specific config)
 */
export interface BeaconConfig {
  keyboard?: KeyboardConfig;
  spotlight?: SpotlightConfig;
  persistence?: PersistenceConfig;
  a11y?: A11yConfig;
  scroll?: ScrollConfig;
}
```

## File: `packages/core/src/types/state.ts`

```typescript
import type { Tour, TourStep } from './tour';

/**
 * Current tour state
 */
export interface TourState {
  /** Current tour ID (null if no tour active) */
  tourId: string | null;

  /** Is a tour currently active */
  isActive: boolean;

  /** Current step index (0-based) */
  currentStepIndex: number;

  /** Current step data (null if no step active) */
  currentStep: TourStep | null;

  /** Total number of steps in current tour */
  totalSteps: number;

  /** Tour is loading (waiting for element, route, etc.) */
  isLoading: boolean;

  /** Tour is transitioning between steps */
  isTransitioning: boolean;

  /** List of completed tour IDs */
  completedTours: string[];

  /** List of skipped tour IDs */
  skippedTours: string[];
}

/**
 * Extended tour context with tour config and custom data
 */
export interface TourContext extends TourState {
  /** Current tour configuration */
  tour: Tour | null;

  /** Custom context data set via setData */
  data: Record<string, unknown>;
}

/**
 * Tour action methods
 */
export interface TourActions {
  /** Start a tour (optionally at specific step) */
  start: (tourId?: string, stepIndex?: number) => void;

  /** Advance to next step */
  next: () => void;

  /** Go to previous step */
  prev: () => void;

  /** Go to specific step by index */
  goTo: (stepIndex: number) => void;

  /** Skip/exit tour (marks as skipped) */
  skip: () => void;

  /** Complete tour (marks as completed) */
  complete: () => void;

  /** Stop tour without marking as completed or skipped */
  stop: () => void;

  /** Set "don't show again" for a tour */
  setDontShowAgain: (tourId: string, value: boolean) => void;

  /** Reset tour progress (optionally for specific tour) */
  reset: (tourId?: string) => void;

  /** Set custom context data */
  setData: (key: string, value: unknown) => void;
}

/**
 * Combined context value (state + actions)
 */
export interface TourContextValue extends TourState, TourActions {
  tour: Tour | null;
  data: Record<string, unknown>;
}

/**
 * Initial state factory
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

## File: `packages/core/src/types/hints.ts`

```typescript
import type React from 'react';
import type { Placement } from './config';

/**
 * Beacon position relative to target element
 */
export type BeaconPosition =
  | 'top-left'
  | 'top-right'
  | 'bottom-left'
  | 'bottom-right'
  | 'center';

/**
 * Hint configuration
 */
export interface HintConfig {
  /** Unique hint identifier */
  id: string;

  /** Target element - CSS selector or React ref */
  target: string | React.RefObject<HTMLElement | null>;

  /** Hint content for tooltip */
  content: React.ReactNode;

  /** Beacon position relative to target (default: 'top-right') */
  position?: BeaconPosition;

  /** Tooltip placement when opened */
  tooltipPlacement?: Placement;

  /** Show pulsing animation on beacon (default: true) */
  pulse?: boolean;

  /** Auto-show tooltip on mount (default: false) */
  autoShow?: boolean;

  /** Persist dismissal to storage (default: false) */
  persist?: boolean;

  // Lifecycle callbacks
  onClick?: () => void;
  onShow?: () => void;
  onDismiss?: () => void;
}

/**
 * Individual hint state
 */
export interface HintState {
  /** Hint identifier */
  id: string;

  /** Is tooltip currently visible */
  isOpen: boolean;

  /** Has been permanently dismissed */
  isDismissed: boolean;
}

/**
 * Hints context state
 */
export interface HintsState {
  /** Map of hint states by ID */
  hints: Map<string, HintState>;

  /** Currently active/open hint ID */
  activeHint: string | null;
}

/**
 * Hints action methods
 */
export interface HintsActions {
  /** Register a hint */
  registerHint: (id: string) => void;

  /** Unregister a hint */
  unregisterHint: (id: string) => void;

  /** Show hint tooltip */
  showHint: (id: string) => void;

  /** Hide hint tooltip */
  hideHint: (id: string) => void;

  /** Dismiss hint permanently */
  dismissHint: (id: string) => void;

  /** Reset dismissed state for hint */
  resetHint: (id: string) => void;

  /** Reset all dismissed hints */
  resetAllHints: () => void;
}

/**
 * Combined hints context value
 */
export interface HintsContextValue extends HintsState, HintsActions {}
```

## Type Defaults

When no configuration is provided, these defaults are used:

```typescript
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

export const defaultPersistenceConfig: Required<PersistenceConfig> = {
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

## Usage Examples

### Creating a Tour

```typescript
import type { Tour } from '@tour-kit/core';

const onboardingTour: Tour = {
  id: 'onboarding',
  steps: [
    {
      id: 'welcome',
      target: '#welcome-button',
      title: 'Welcome!',
      content: 'Let\'s get started with a quick tour.',
      placement: 'bottom',
    },
    {
      id: 'dashboard',
      target: '#dashboard',
      title: 'Your Dashboard',
      content: 'All your data at a glance.',
      placement: 'right',
      spotlightPadding: 16,
    },
  ],
  keyboard: { enabled: true },
  spotlight: { color: 'rgba(0, 0, 0, 0.7)' },
  onComplete: (ctx) => console.log('Tour completed!'),
};
```

### Creating a Hint

```typescript
import type { HintConfig } from '@tour-kit/core';

const newFeatureHint: HintConfig = {
  id: 'new-feature',
  target: '#feature-button',
  content: 'Check out this new feature!',
  position: 'top-right',
  pulse: true,
  persist: true,
};
```

