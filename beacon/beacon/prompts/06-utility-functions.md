# Prompt 06: Utility Functions

## Problem

Create all utility functions for DOM manipulation, positioning, scrolling, storage, and accessibility.

## Dependencies on Previous Prompts

- Prompt 04 complete (types defined)
- Prompt 05 complete (context created)

## Supporting Information

### Target File Structure

```
packages/core/src/
├── utils/
│   ├── dom.ts
│   ├── position.ts
│   ├── scroll.ts
│   ├── storage.ts
│   ├── a11y.ts
│   ├── create-tour.ts
│   ├── create-step.ts
│   └── index.ts
└── index.ts (updated)
```

### Reference Specification

See `../specs/utilities.md` for full implementations.

## Steps To Complete

### Step 1: Create DOM utilities

Create `packages/core/src/utils/dom.ts`:

```typescript
import type React from 'react';

/**
 * Safely get an element from various target types
 */
export function getElement(
  target: string | HTMLElement | React.RefObject<HTMLElement | null> | null
): HTMLElement | null {
  if (!target) return null;

  if (typeof target === 'string') {
    return document.querySelector<HTMLElement>(target);
  }

  if ('current' in target) {
    return target.current;
  }

  return target;
}

/**
 * Check if element is fully visible in viewport
 */
export function isElementVisible(element: HTMLElement): boolean {
  const rect = element.getBoundingClientRect();
  const windowHeight =
    window.innerHeight || document.documentElement.clientHeight;
  const windowWidth = window.innerWidth || document.documentElement.clientWidth;

  return (
    rect.top >= 0 &&
    rect.left >= 0 &&
    rect.bottom <= windowHeight &&
    rect.right <= windowWidth
  );
}

/**
 * Check if element is at least partially visible
 */
export function isElementPartiallyVisible(element: HTMLElement): boolean {
  const rect = element.getBoundingClientRect();
  const windowHeight =
    window.innerHeight || document.documentElement.clientHeight;
  const windowWidth = window.innerWidth || document.documentElement.clientWidth;

  return (
    rect.top < windowHeight &&
    rect.bottom > 0 &&
    rect.left < windowWidth &&
    rect.right > 0
  );
}

/**
 * Wait for element to appear in DOM
 */
export function waitForElement(
  selector: string,
  timeout = 5000
): Promise<HTMLElement> {
  return new Promise((resolve, reject) => {
    const element = document.querySelector<HTMLElement>(selector);
    if (element) {
      resolve(element);
      return;
    }

    const observer = new MutationObserver((_, obs) => {
      const el = document.querySelector<HTMLElement>(selector);
      if (el) {
        obs.disconnect();
        resolve(el);
      }
    });

    observer.observe(document.body, {
      childList: true,
      subtree: true,
    });

    const timeoutId = setTimeout(() => {
      observer.disconnect();
      reject(new Error(`Element "${selector}" not found within ${timeout}ms`));
    }, timeout);

    // Cleanup on success
    const originalResolve = resolve;
    resolve = (el) => {
      clearTimeout(timeoutId);
      originalResolve(el);
    };
  });
}

const FOCUSABLE_SELECTOR = [
  'a[href]',
  'button:not([disabled])',
  'textarea:not([disabled])',
  'input:not([disabled])',
  'select:not([disabled])',
  '[tabindex]:not([tabindex="-1"])',
].join(', ');

/**
 * Get all focusable elements within a container
 */
export function getFocusableElements(container: HTMLElement): HTMLElement[] {
  return Array.from(
    container.querySelectorAll<HTMLElement>(FOCUSABLE_SELECTOR)
  ).filter((el) => el.offsetParent !== null);
}

/**
 * Find the scrollable parent of an element
 */
export function getScrollParent(element: HTMLElement): HTMLElement | Window {
  let parent: HTMLElement | null = element.parentElement;

  while (parent) {
    const { overflow, overflowX, overflowY } = getComputedStyle(parent);

    if (/(auto|scroll)/.test(overflow + overflowX + overflowY)) {
      return parent;
    }

    parent = parent.parentElement;
  }

  return window;
}
```

### Step 2: Create position utilities

Create `packages/core/src/utils/position.ts`:

```typescript
import type { Rect, Position, Placement, Side, Alignment } from '../types';

/**
 * Get element's position including scroll offset
 */
export function getElementRect(element: HTMLElement): Rect {
  const rect = element.getBoundingClientRect();

  return {
    x: rect.x + window.scrollX,
    y: rect.y + window.scrollY,
    width: rect.width,
    height: rect.height,
  };
}

/**
 * Get current viewport dimensions
 */
export function getViewportDimensions(): { width: number; height: number } {
  return {
    width: window.innerWidth || document.documentElement.clientWidth,
    height: window.innerHeight || document.documentElement.clientHeight,
  };
}

/**
 * Parse placement string into side and alignment
 */
export function parsePlacement(placement: Placement): {
  side: Side;
  alignment: Alignment;
} {
  const parts = placement.split('-') as [Side, Alignment?];

  return {
    side: parts[0],
    alignment: parts[1] ?? 'center',
  };
}

/**
 * Get the opposite side
 */
export function getOppositeSide(side: Side): Side {
  const opposites: Record<Side, Side> = {
    top: 'bottom',
    bottom: 'top',
    left: 'right',
    right: 'left',
  };

  return opposites[side];
}

/**
 * Calculate position for tooltip relative to target
 */
export function calculatePosition(
  targetRect: Rect,
  tooltipSize: { width: number; height: number },
  placement: Placement,
  offset: [number, number] = [0, 0]
): Position {
  const { side, alignment } = parsePlacement(placement);
  const [offsetX, offsetY] = offset;

  let x = 0;
  let y = 0;

  // Calculate main axis
  switch (side) {
    case 'top':
      y = targetRect.y - tooltipSize.height - offsetY;
      break;
    case 'bottom':
      y = targetRect.y + targetRect.height + offsetY;
      break;
    case 'left':
      x = targetRect.x - tooltipSize.width - offsetX;
      break;
    case 'right':
      x = targetRect.x + targetRect.width + offsetX;
      break;
  }

  // Calculate cross axis (alignment)
  if (side === 'top' || side === 'bottom') {
    switch (alignment) {
      case 'start':
        x = targetRect.x + offsetX;
        break;
      case 'end':
        x = targetRect.x + targetRect.width - tooltipSize.width - offsetX;
        break;
      case 'center':
      default:
        x = targetRect.x + (targetRect.width - tooltipSize.width) / 2 + offsetX;
    }
  } else {
    switch (alignment) {
      case 'start':
        y = targetRect.y + offsetY;
        break;
      case 'end':
        y = targetRect.y + targetRect.height - tooltipSize.height - offsetY;
        break;
      case 'center':
      default:
        y =
          targetRect.y + (targetRect.height - tooltipSize.height) / 2 + offsetY;
    }
  }

  return { x, y };
}

/**
 * Check if position would cause overflow
 */
export function wouldOverflow(
  position: Position,
  dimensions: { width: number; height: number },
  viewport: { width: number; height: number }
): { top: boolean; right: boolean; bottom: boolean; left: boolean } {
  return {
    top: position.y < 0,
    right: position.x + dimensions.width > viewport.width,
    bottom: position.y + dimensions.height > viewport.height,
    left: position.x < 0,
  };
}
```

### Step 3: Create scroll utilities

Create `packages/core/src/utils/scroll.ts`:

```typescript
import type { ScrollConfig } from '../types';
import { getScrollParent, isElementVisible } from './dom';

const defaultScrollConfig: Required<ScrollConfig> = {
  enabled: true,
  behavior: 'smooth',
  block: 'center',
  offset: 20,
};

/**
 * Scroll element into view with configuration
 */
export function scrollIntoView(
  element: HTMLElement,
  config?: ScrollConfig
): Promise<void> {
  const mergedConfig = { ...defaultScrollConfig, ...config };

  if (!mergedConfig.enabled) {
    return Promise.resolve();
  }

  return new Promise((resolve) => {
    if (isElementVisible(element)) {
      resolve();
      return;
    }

    element.scrollIntoView({
      behavior: mergedConfig.behavior,
      block: mergedConfig.block,
    });

    // Estimate scroll duration
    const duration = mergedConfig.behavior === 'smooth' ? 500 : 0;
    setTimeout(resolve, duration);
  });
}

/**
 * Scroll to specific position
 */
export function scrollTo(
  container: HTMLElement | Window,
  position: { top?: number; left?: number },
  behavior: ScrollBehavior = 'smooth'
): void {
  container.scrollTo({
    ...position,
    behavior,
  });
}

/**
 * Get current scroll position
 */
export function getScrollPosition(container: HTMLElement | Window = window): {
  x: number;
  y: number;
} {
  if (container === window) {
    return {
      x: window.scrollX || document.documentElement.scrollLeft,
      y: window.scrollY || document.documentElement.scrollTop,
    };
  }

  const el = container as HTMLElement;
  return {
    x: el.scrollLeft,
    y: el.scrollTop,
  };
}

/**
 * Lock body scroll and return unlock function
 */
export function lockScroll(): () => void {
  const scrollY = window.scrollY;
  const body = document.body;

  body.style.position = 'fixed';
  body.style.top = `-${scrollY}px`;
  body.style.width = '100%';
  body.style.overflowY = 'scroll';

  return () => {
    body.style.position = '';
    body.style.top = '';
    body.style.width = '';
    body.style.overflowY = '';
    window.scrollTo(0, scrollY);
  };
}
```

### Step 4: Create storage utilities

Create `packages/core/src/utils/storage.ts`:

```typescript
import type { Storage, PersistenceConfig } from '../types';

/**
 * Create storage adapter from config
 */
export function createStorageAdapter(
  storageType: PersistenceConfig['storage']
): Storage {
  if (typeof storageType === 'object') {
    return storageType;
  }

  if (typeof window === 'undefined') {
    return createNoopStorage();
  }

  switch (storageType) {
    case 'sessionStorage':
      return window.sessionStorage;
    case 'cookie':
      return createCookieStorage();
    default:
      return window.localStorage;
  }
}

/**
 * No-op storage for SSR
 */
export function createNoopStorage(): Storage {
  return {
    getItem: () => null,
    setItem: () => {},
    removeItem: () => {},
  };
}

/**
 * Cookie-based storage adapter
 */
export function createCookieStorage(
  options: { expires?: number; path?: string } = {}
): Storage {
  const { expires = 365, path = '/' } = options;

  return {
    getItem: (key: string) => {
      if (typeof document === 'undefined') return null;
      const match = document.cookie.match(new RegExp(`(^| )${key}=([^;]+)`));
      return match ? decodeURIComponent(match[2]) : null;
    },

    setItem: (key: string, value: string) => {
      if (typeof document === 'undefined') return;
      const date = new Date();
      date.setTime(date.getTime() + expires * 24 * 60 * 60 * 1000);
      document.cookie = `${key}=${encodeURIComponent(
        value
      )};expires=${date.toUTCString()};path=${path}`;
    },

    removeItem: (key: string) => {
      if (typeof document === 'undefined') return;
      document.cookie = `${key}=;expires=Thu, 01 Jan 1970 00:00:00 GMT;path=${path}`;
    },
  };
}

/**
 * Safe JSON parse with fallback
 */
export function safeJSONParse<T>(value: string | null, fallback: T): T {
  if (!value) return fallback;

  try {
    return JSON.parse(value) as T;
  } catch {
    return fallback;
  }
}

/**
 * Create storage with key prefix
 */
export function createPrefixedStorage(
  storage: Storage,
  prefix: string
): Storage {
  const prefixKey = (key: string) => `${prefix}:${key}`;

  return {
    getItem: (key: string) => storage.getItem(prefixKey(key)),
    setItem: (key: string, value: string) =>
      storage.setItem(prefixKey(key), value),
    removeItem: (key: string) => storage.removeItem(prefixKey(key)),
  };
}
```

### Step 5: Create accessibility utilities

Create `packages/core/src/utils/a11y.ts`:

```typescript
/**
 * Announce message to screen readers
 */
export function announce(
  message: string,
  politeness: 'polite' | 'assertive' = 'polite'
): void {
  if (typeof document === 'undefined') return;

  const announcer = document.createElement('div');

  announcer.setAttribute('role', 'status');
  announcer.setAttribute('aria-live', politeness);
  announcer.setAttribute('aria-atomic', 'true');

  // Visually hidden
  Object.assign(announcer.style, {
    position: 'absolute',
    width: '1px',
    height: '1px',
    padding: '0',
    margin: '-1px',
    overflow: 'hidden',
    clip: 'rect(0, 0, 0, 0)',
    whiteSpace: 'nowrap',
    border: '0',
  });

  document.body.appendChild(announcer);

  // Delay for screen reader registration
  setTimeout(() => {
    announcer.textContent = message;
  }, 100);

  // Cleanup
  setTimeout(() => {
    document.body.removeChild(announcer);
  }, 1000);
}

/**
 * Generate unique ID for accessibility
 */
export function generateId(prefix = 'beacon'): string {
  return `${prefix}-${Math.random().toString(36).substring(2, 11)}`;
}

/**
 * Check if user prefers reduced motion
 */
export function prefersReducedMotion(): boolean {
  if (typeof window === 'undefined') return false;
  return window.matchMedia('(prefers-reduced-motion: reduce)').matches;
}

/**
 * Generate step announcement for screen readers
 */
export function getStepAnnouncement(
  stepTitle: string | undefined,
  currentStep: number,
  totalSteps: number
): string {
  const stepInfo = `Step ${currentStep} of ${totalSteps}`;
  return stepTitle ? `${stepInfo}: ${stepTitle}` : stepInfo;
}
```

### Step 6: Create factory functions

Create `packages/core/src/utils/create-tour.ts`:

```typescript
import type { Tour, TourStep, TourOptions } from '../types';

let tourIdCounter = 0;

/**
 * Create a tour with auto-generated ID
 */
export function createTour(steps: TourStep[], options?: TourOptions): Tour {
  return {
    id: `tour-${++tourIdCounter}`,
    steps,
    ...options,
  };
}

/**
 * Create a tour with explicit ID
 */
export function createNamedTour(
  id: string,
  steps: TourStep[],
  options?: TourOptions
): Tour {
  return {
    id,
    steps,
    ...options,
  };
}
```

Create `packages/core/src/utils/create-step.ts`:

```typescript
import type { TourStep, StepOptions } from '../types';

let stepIdCounter = 0;

/**
 * Create a step with auto-generated ID
 */
export function createStep(
  target: TourStep['target'],
  content: TourStep['content'],
  options?: Partial<StepOptions>
): TourStep {
  return {
    id: `step-${++stepIdCounter}`,
    target,
    content,
    ...options,
  };
}

/**
 * Create a step with explicit ID
 */
export function createNamedStep(
  id: string,
  target: TourStep['target'],
  content: TourStep['content'],
  options?: Partial<StepOptions>
): TourStep {
  return {
    id,
    target,
    content,
    ...options,
  };
}
```

### Step 7: Create utils index

Create `packages/core/src/utils/index.ts`:

```typescript
export {
  getElement,
  isElementVisible,
  isElementPartiallyVisible,
  waitForElement,
  getFocusableElements,
  getScrollParent,
} from './dom';

export {
  getElementRect,
  getViewportDimensions,
  parsePlacement,
  calculatePosition,
  wouldOverflow,
  getOppositeSide,
} from './position';

export {
  scrollIntoView,
  scrollTo,
  getScrollPosition,
  lockScroll,
} from './scroll';

export {
  createStorageAdapter,
  createNoopStorage,
  createCookieStorage,
  safeJSONParse,
  createPrefixedStorage,
} from './storage';

export {
  announce,
  generateId,
  prefersReducedMotion,
  getStepAnnouncement,
} from './a11y';

export { createTour, createNamedTour } from './create-tour';
export { createStep, createNamedStep } from './create-step';
```

### Step 8: Update package entry

Update `packages/core/src/index.ts`:

```typescript
// Types
export * from './types';

// Context
export * from './context';

// Utilities
export * from './utils';
```

## Key Requirements & Constraints

- [ ] All utilities are SSR-safe
- [ ] No side effects at module load
- [ ] Proper TypeScript types
- [ ] Functions are tree-shakeable
- [ ] Utilities are tested (next prompt)

## Verification

```bash
cd packages/core
pnpm typecheck
pnpm build
```

## Next Prompt

Continue with `07-tour-hooks.md` to implement the main hooks.
