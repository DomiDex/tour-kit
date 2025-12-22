# Hooks Specification

## Overview

This document specifies all hooks for `@tour-kit/core`. Hooks are the primary API for headless usage.

---

## useTour

The main hook for controlling tours.

### Signature

```typescript
function useTour(tourId?: string): UseTourReturn;

interface UseTourReturn {
  // State
  isActive: boolean;
  isLoading: boolean;
  isTransitioning: boolean;
  currentStep: TourStep | null;
  currentStepIndex: number;
  totalSteps: number;
  isFirstStep: boolean;
  isLastStep: boolean;
  progress: number; // 0 to 1

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
}
```

### Implementation

```typescript
// packages/core/src/hooks/use-tour.ts
import { useContext, useCallback, useMemo } from 'react';
import { TourContext } from '../context/tour-context';
import type { TourStep } from '../types';

export function useTour(tourId?: string): UseTourReturn {
  const context = useContext(TourContext);

  if (!context) {
    throw new Error('useTour must be used within a TourProvider');
  }

  const {
    tourId: activeTourId,
    isActive,
    isLoading,
    isTransitioning,
    currentStep,
    currentStepIndex,
    totalSteps,
    tour,
    start: contextStart,
    next,
    prev,
    goTo,
    skip,
    complete,
    stop,
  } = context;

  // If tourId specified, only return active state if it matches
  const isThisTourActive = tourId 
    ? isActive && activeTourId === tourId 
    : isActive;

  const start = useCallback(
    (stepIndex?: number) => {
      contextStart(tourId, stepIndex);
    },
    [contextStart, tourId]
  );

  const isFirstStep = currentStepIndex === 0;
  const isLastStep = currentStepIndex === totalSteps - 1;
  const progress = totalSteps > 0 ? (currentStepIndex + 1) / totalSteps : 0;

  const isStepActive = useCallback(
    (stepId: string) => currentStep?.id === stepId,
    [currentStep]
  );

  const getStep = useCallback(
    (stepId: string) => tour?.steps.find((s) => s.id === stepId),
    [tour]
  );

  return useMemo(
    () => ({
      isActive: isThisTourActive,
      isLoading,
      isTransitioning,
      currentStep: isThisTourActive ? currentStep : null,
      currentStepIndex: isThisTourActive ? currentStepIndex : 0,
      totalSteps: isThisTourActive ? totalSteps : 0,
      isFirstStep,
      isLastStep,
      progress,
      start,
      next,
      prev,
      goTo,
      skip,
      complete,
      stop,
      isStepActive,
      getStep,
    }),
    [
      isThisTourActive,
      isLoading,
      isTransitioning,
      currentStep,
      currentStepIndex,
      totalSteps,
      isFirstStep,
      isLastStep,
      progress,
      start,
      next,
      prev,
      goTo,
      skip,
      complete,
      stop,
      isStepActive,
      getStep,
    ]
  );
}
```

---

## useStep

Hook for individual step state and actions.

### Signature

```typescript
function useStep(stepId: string): UseStepReturn;

interface UseStepReturn {
  // State
  isActive: boolean;
  isVisible: boolean;
  hasCompleted: boolean;

  // Target element
  targetElement: HTMLElement | null;
  targetRect: DOMRect | null;

  // Actions
  show: () => void;
  hide: () => void;
  complete: () => void;
}
```

### Implementation

```typescript
// packages/core/src/hooks/use-step.ts
import { useContext, useCallback, useMemo, useState, useEffect } from 'react';
import { TourContext } from '../context/tour-context';
import { useElementPosition } from './use-element-position';

export function useStep(stepId: string): UseStepReturn {
  const context = useContext(TourContext);
  
  if (!context) {
    throw new Error('useStep must be used within a TourProvider');
  }

  const { currentStep, goTo, tour, next } = context;
  
  const step = tour?.steps.find((s) => s.id === stepId);
  const stepIndex = tour?.steps.findIndex((s) => s.id === stepId) ?? -1;
  
  const isActive = currentStep?.id === stepId;
  const [isVisible, setIsVisible] = useState(false);
  const [hasCompleted, setHasCompleted] = useState(false);

  // Get target element
  const target = step?.target;
  const { element: targetElement, rect: targetRect } = useElementPosition(
    typeof target === 'string' ? target : target?.current ?? null
  );

  useEffect(() => {
    if (isActive) {
      setIsVisible(true);
    }
  }, [isActive]);

  const show = useCallback(() => {
    if (stepIndex >= 0) {
      goTo(stepIndex);
    }
  }, [goTo, stepIndex]);

  const hide = useCallback(() => {
    setIsVisible(false);
  }, []);

  const complete = useCallback(() => {
    setHasCompleted(true);
    next();
  }, [next]);

  return useMemo(
    () => ({
      isActive,
      isVisible,
      hasCompleted,
      targetElement,
      targetRect,
      show,
      hide,
      complete,
    }),
    [isActive, isVisible, hasCompleted, targetElement, targetRect, show, hide, complete]
  );
}
```

---

## useSpotlight

Hook for spotlight overlay calculations and state.

### Signature

```typescript
function useSpotlight(): UseSpotlightReturn;

interface UseSpotlightReturn {
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
}
```

### Implementation

```typescript
// packages/core/src/hooks/use-spotlight.ts
import { useState, useCallback, useMemo, useRef, useEffect } from 'react';
import type { SpotlightConfig } from '../types';
import { defaultSpotlightConfig } from '../types/config';

export function useSpotlight(): UseSpotlightReturn {
  const [isVisible, setIsVisible] = useState(false);
  const [targetRect, setTargetRect] = useState<DOMRect | null>(null);
  const [config, setConfig] = useState<SpotlightConfig>(defaultSpotlightConfig);
  const targetRef = useRef<HTMLElement | null>(null);

  const updateRect = useCallback(() => {
    if (targetRef.current) {
      setTargetRect(targetRef.current.getBoundingClientRect());
    }
  }, []);

  // Update on scroll/resize
  useEffect(() => {
    if (!isVisible) return;

    const handleUpdate = () => updateRect();
    window.addEventListener('scroll', handleUpdate, true);
    window.addEventListener('resize', handleUpdate);

    return () => {
      window.removeEventListener('scroll', handleUpdate, true);
      window.removeEventListener('resize', handleUpdate);
    };
  }, [isVisible, updateRect]);

  const show = useCallback(
    (target: HTMLElement, spotlightConfig?: SpotlightConfig) => {
      targetRef.current = target;
      setConfig({ ...defaultSpotlightConfig, ...spotlightConfig });
      setTargetRect(target.getBoundingClientRect());
      setIsVisible(true);
    },
    []
  );

  const hide = useCallback(() => {
    setIsVisible(false);
    targetRef.current = null;
    setTargetRect(null);
  }, []);

  const overlayStyle = useMemo<React.CSSProperties>(() => {
    return {
      position: 'fixed',
      inset: 0,
      backgroundColor: config.color ?? 'rgba(0, 0, 0, 0.5)',
      transition: config.animate
        ? `all ${config.animationDuration ?? 300}ms ease-out`
        : undefined,
      pointerEvents: 'auto',
    };
  }, [config]);

  const cutoutStyle = useMemo<React.CSSProperties>(() => {
    if (!targetRect) return {};

    const padding = config.padding ?? 8;
    const borderRadius = config.borderRadius ?? 4;

    return {
      position: 'absolute',
      top: targetRect.top - padding,
      left: targetRect.left - padding,
      width: targetRect.width + padding * 2,
      height: targetRect.height + padding * 2,
      borderRadius,
      boxShadow: `0 0 0 9999px ${config.color ?? 'rgba(0, 0, 0, 0.5)'}`,
      transition: config.animate
        ? `all ${config.animationDuration ?? 300}ms ease-out`
        : undefined,
      pointerEvents: 'none',
    };
  }, [targetRect, config]);

  return useMemo(
    () => ({
      isVisible,
      targetRect,
      overlayStyle,
      cutoutStyle,
      show,
      hide,
      update: updateRect,
    }),
    [isVisible, targetRect, overlayStyle, cutoutStyle, show, hide, updateRect]
  );
}
```

---

## useKeyboardNavigation

Hook for keyboard event handling.

### Signature

```typescript
function useKeyboardNavigation(config?: KeyboardConfig): void;
```

### Implementation

```typescript
// packages/core/src/hooks/use-keyboard.ts
import { useEffect, useContext } from 'react';
import { TourContext } from '../context/tour-context';
import type { KeyboardConfig } from '../types';
import { defaultKeyboardConfig } from '../types/config';

export function useKeyboardNavigation(config?: KeyboardConfig): void {
  const context = useContext(TourContext);
  
  if (!context) {
    throw new Error('useKeyboardNavigation must be used within a TourProvider');
  }

  const { isActive, next, prev, skip } = context;
  const mergedConfig = { ...defaultKeyboardConfig, ...config };

  useEffect(() => {
    if (!isActive || !mergedConfig.enabled) return;

    const handleKeyDown = (event: KeyboardEvent) => {
      const { key } = event;

      if (mergedConfig.nextKeys?.includes(key)) {
        event.preventDefault();
        next();
      } else if (mergedConfig.prevKeys?.includes(key)) {
        event.preventDefault();
        prev();
      } else if (mergedConfig.exitKeys?.includes(key)) {
        event.preventDefault();
        skip();
      }
    };

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [isActive, mergedConfig, next, prev, skip]);
}
```

---

## useFocusTrap

Hook for trapping focus within an element.

### Signature

```typescript
function useFocusTrap(enabled?: boolean): {
  containerRef: React.RefObject<HTMLElement>;
  activate: () => void;
  deactivate: () => void;
};
```

### Implementation

```typescript
// packages/core/src/hooks/use-focus-trap.ts
import { useRef, useCallback, useEffect } from 'react';

const FOCUSABLE_SELECTOR = [
  'a[href]',
  'button:not([disabled])',
  'textarea:not([disabled])',
  'input:not([disabled])',
  'select:not([disabled])',
  '[tabindex]:not([tabindex="-1"])',
].join(', ');

export function useFocusTrap(enabled = true) {
  const containerRef = useRef<HTMLElement>(null);
  const previousActiveElement = useRef<HTMLElement | null>(null);
  const isTrapping = useRef(false);

  const getFocusableElements = useCallback(() => {
    if (!containerRef.current) return [];
    return Array.from(
      containerRef.current.querySelectorAll<HTMLElement>(FOCUSABLE_SELECTOR)
    ).filter((el) => el.offsetParent !== null);
  }, []);

  const handleKeyDown = useCallback(
    (event: KeyboardEvent) => {
      if (!isTrapping.current || event.key !== 'Tab') return;

      const focusable = getFocusableElements();
      if (focusable.length === 0) return;

      const first = focusable[0];
      const last = focusable[focusable.length - 1];

      if (event.shiftKey && document.activeElement === first) {
        event.preventDefault();
        last.focus();
      } else if (!event.shiftKey && document.activeElement === last) {
        event.preventDefault();
        first.focus();
      }
    },
    [getFocusableElements]
  );

  const activate = useCallback(() => {
    if (!enabled || !containerRef.current) return;
    
    previousActiveElement.current = document.activeElement as HTMLElement;
    isTrapping.current = true;
    
    const focusable = getFocusableElements();
    if (focusable.length > 0) {
      focusable[0].focus();
    }
    
    document.addEventListener('keydown', handleKeyDown);
  }, [enabled, getFocusableElements, handleKeyDown]);

  const deactivate = useCallback(() => {
    isTrapping.current = false;
    document.removeEventListener('keydown', handleKeyDown);
    
    if (previousActiveElement.current) {
      previousActiveElement.current.focus();
      previousActiveElement.current = null;
    }
  }, [handleKeyDown]);

  useEffect(() => {
    return () => {
      document.removeEventListener('keydown', handleKeyDown);
    };
  }, [handleKeyDown]);

  return { containerRef, activate, deactivate };
}
```

---

## usePersistence

Hook for persisting tour state.

### Signature

```typescript
function usePersistence(config?: PersistenceConfig): UsePersistenceReturn;

interface UsePersistenceReturn {
  getCompletedTours: () => string[];
  getSkippedTours: () => string[];
  getDontShowAgain: (tourId: string) => boolean;
  getLastStep: (tourId: string) => number | null;
  
  markCompleted: (tourId: string) => void;
  markSkipped: (tourId: string) => void;
  setDontShowAgain: (tourId: string, value: boolean) => void;
  saveStep: (tourId: string, stepIndex: number) => void;
  
  reset: (tourId?: string) => void;
}
```

### Implementation

```typescript
// packages/core/src/hooks/use-persistence.ts
import { useCallback, useMemo } from 'react';
import type { PersistenceConfig, Storage } from '../types';
import { defaultPersistenceConfig } from '../types/config';

function getStorage(storage: PersistenceConfig['storage']): Storage {
  if (typeof storage === 'object') return storage;
  
  if (typeof window === 'undefined') {
    return {
      getItem: () => null,
      setItem: () => {},
      removeItem: () => {},
    };
  }
  
  switch (storage) {
    case 'sessionStorage':
      return window.sessionStorage;
    case 'cookie':
      // Implement cookie storage adapter
      return {
        getItem: (key) => {
          const match = document.cookie.match(new RegExp(`(^| )${key}=([^;]+)`));
          return match ? decodeURIComponent(match[2]) : null;
        },
        setItem: (key, value) => {
          document.cookie = `${key}=${encodeURIComponent(value)};path=/;max-age=31536000`;
        },
        removeItem: (key) => {
          document.cookie = `${key}=;path=/;max-age=0`;
        },
      };
    default:
      return window.localStorage;
  }
}

export function usePersistence(config?: PersistenceConfig): UsePersistenceReturn {
  const mergedConfig = { ...defaultPersistenceConfig, ...config };
  const storage = useMemo(() => getStorage(mergedConfig.storage), [mergedConfig.storage]);
  const prefix = mergedConfig.keyPrefix;

  const getKey = useCallback((key: string) => `${prefix}:${key}`, [prefix]);

  const getCompletedTours = useCallback(() => {
    const data = storage.getItem(getKey('completed'));
    return data ? JSON.parse(data) : [];
  }, [storage, getKey]);

  const getSkippedTours = useCallback(() => {
    const data = storage.getItem(getKey('skipped'));
    return data ? JSON.parse(data) : [];
  }, [storage, getKey]);

  const getDontShowAgain = useCallback(
    (tourId: string) => {
      const data = storage.getItem(getKey(`dontShow:${tourId}`));
      return data === 'true';
    },
    [storage, getKey]
  );

  const getLastStep = useCallback(
    (tourId: string) => {
      const data = storage.getItem(getKey(`step:${tourId}`));
      return data ? parseInt(data, 10) : null;
    },
    [storage, getKey]
  );

  const markCompleted = useCallback(
    (tourId: string) => {
      const completed = getCompletedTours();
      if (!completed.includes(tourId)) {
        completed.push(tourId);
        storage.setItem(getKey('completed'), JSON.stringify(completed));
      }
    },
    [storage, getKey, getCompletedTours]
  );

  const markSkipped = useCallback(
    (tourId: string) => {
      const skipped = getSkippedTours();
      if (!skipped.includes(tourId)) {
        skipped.push(tourId);
        storage.setItem(getKey('skipped'), JSON.stringify(skipped));
      }
    },
    [storage, getKey, getSkippedTours]
  );

  const setDontShowAgain = useCallback(
    (tourId: string, value: boolean) => {
      if (value) {
        storage.setItem(getKey(`dontShow:${tourId}`), 'true');
      } else {
        storage.removeItem(getKey(`dontShow:${tourId}`));
      }
    },
    [storage, getKey]
  );

  const saveStep = useCallback(
    (tourId: string, stepIndex: number) => {
      storage.setItem(getKey(`step:${tourId}`), String(stepIndex));
    },
    [storage, getKey]
  );

  const reset = useCallback(
    (tourId?: string) => {
      if (tourId) {
        storage.removeItem(getKey(`step:${tourId}`));
        storage.removeItem(getKey(`dontShow:${tourId}`));
        
        const completed = getCompletedTours().filter((id: string) => id !== tourId);
        storage.setItem(getKey('completed'), JSON.stringify(completed));
        
        const skipped = getSkippedTours().filter((id: string) => id !== tourId);
        storage.setItem(getKey('skipped'), JSON.stringify(skipped));
      } else {
        storage.removeItem(getKey('completed'));
        storage.removeItem(getKey('skipped'));
      }
    },
    [storage, getKey, getCompletedTours, getSkippedTours]
  );

  return {
    getCompletedTours,
    getSkippedTours,
    getDontShowAgain,
    getLastStep,
    markCompleted,
    markSkipped,
    setDontShowAgain,
    saveStep,
    reset,
  };
}
```

---

## useElementPosition

Hook for tracking element position.

### Signature

```typescript
function useElementPosition(
  target: string | HTMLElement | null
): {
  element: HTMLElement | null;
  rect: DOMRect | null;
  update: () => void;
};
```

### Implementation

```typescript
// packages/core/src/hooks/use-element-position.ts
import { useState, useEffect, useCallback, useRef } from 'react';

export function useElementPosition(target: string | HTMLElement | null) {
  const [element, setElement] = useState<HTMLElement | null>(null);
  const [rect, setRect] = useState<DOMRect | null>(null);
  const observerRef = useRef<ResizeObserver | null>(null);

  // Resolve target to element
  useEffect(() => {
    if (!target) {
      setElement(null);
      return;
    }

    if (typeof target === 'string') {
      const el = document.querySelector<HTMLElement>(target);
      setElement(el);
    } else {
      setElement(target);
    }
  }, [target]);

  const update = useCallback(() => {
    if (element) {
      setRect(element.getBoundingClientRect());
    }
  }, [element]);

  // Update rect on changes
  useEffect(() => {
    if (!element) {
      setRect(null);
      return;
    }

    update();

    // Observe resize
    observerRef.current = new ResizeObserver(update);
    observerRef.current.observe(element);

    // Listen to scroll and resize
    window.addEventListener('scroll', update, true);
    window.addEventListener('resize', update);

    return () => {
      observerRef.current?.disconnect();
      window.removeEventListener('scroll', update, true);
      window.removeEventListener('resize', update);
    };
  }, [element, update]);

  return { element, rect, update };
}
```

---

## useMediaQuery

Hook for checking media queries (e.g., reduced motion).

### Signature

```typescript
function useMediaQuery(query: string): boolean;
function usePrefersReducedMotion(): boolean;
```

### Implementation

```typescript
// packages/core/src/hooks/use-media-query.ts
import { useState, useEffect } from 'react';

export function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(() => {
    if (typeof window === 'undefined') return false;
    return window.matchMedia(query).matches;
  });

  useEffect(() => {
    if (typeof window === 'undefined') return;

    const mediaQuery = window.matchMedia(query);
    const handler = (e: MediaQueryListEvent) => setMatches(e.matches);

    setMatches(mediaQuery.matches);
    mediaQuery.addEventListener('change', handler);

    return () => mediaQuery.removeEventListener('change', handler);
  }, [query]);

  return matches;
}

export function usePrefersReducedMotion(): boolean {
  return useMediaQuery('(prefers-reduced-motion: reduce)');
}
```

---

## Hook Index

All hooks exported from `@tour-kit/core`:

```typescript
// packages/core/src/hooks/index.ts
export { useTour } from './use-tour';
export { useStep } from './use-step';
export { useSpotlight } from './use-spotlight';
export { useKeyboardNavigation } from './use-keyboard';
export { useFocusTrap } from './use-focus-trap';
export { usePersistence } from './use-persistence';
export { useElementPosition } from './use-element-position';
export { useMediaQuery, usePrefersReducedMotion } from './use-media-query';
```

