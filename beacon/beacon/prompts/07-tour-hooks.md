# Prompt 07: Tour Hooks

## Problem

Implement the main tour hooks: `useTour`, `useStep`, `useSpotlight`, and supporting hooks.

## Dependencies on Previous Prompts

- Prompt 04-06 complete (types, context, utilities)

## Supporting Information

### Target File Structure

```
packages/core/src/
├── hooks/
│   ├── use-tour.ts
│   ├── use-step.ts
│   ├── use-spotlight.ts
│   ├── use-element-position.ts
│   ├── use-keyboard.ts
│   ├── use-focus-trap.ts
│   ├── use-persistence.ts
│   ├── use-media-query.ts
│   └── index.ts
└── index.ts (updated)
```

### Reference Specification

See `../specs/hooks.md` for full implementations.

## Steps To Complete

### Step 1: Create useElementPosition hook

Create `packages/core/src/hooks/use-element-position.ts`:

```typescript
import { useState, useEffect, useCallback, useRef } from 'react'

export function useElementPosition(target: string | HTMLElement | null) {
  const [element, setElement] = useState<HTMLElement | null>(null)
  const [rect, setRect] = useState<DOMRect | null>(null)
  const observerRef = useRef<ResizeObserver | null>(null)

  // Resolve target to element
  useEffect(() => {
    if (!target) {
      setElement(null)
      return
    }

    if (typeof target === 'string') {
      const el = document.querySelector<HTMLElement>(target)
      setElement(el)
    } else {
      setElement(target)
    }
  }, [target])

  const update = useCallback(() => {
    if (element) {
      setRect(element.getBoundingClientRect())
    }
  }, [element])

  useEffect(() => {
    if (!element) {
      setRect(null)
      return
    }

    update()

    // Observe element resize
    observerRef.current = new ResizeObserver(update)
    observerRef.current.observe(element)

    // Listen for scroll/resize
    window.addEventListener('scroll', update, true)
    window.addEventListener('resize', update)

    return () => {
      observerRef.current?.disconnect()
      window.removeEventListener('scroll', update, true)
      window.removeEventListener('resize', update)
    }
  }, [element, update])

  return { element, rect, update }
}
```

### Step 2: Create useMediaQuery hook

Create `packages/core/src/hooks/use-media-query.ts`:

```typescript
import { useState, useEffect } from 'react'

export function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(() => {
    if (typeof window === 'undefined') return false
    return window.matchMedia(query).matches
  })

  useEffect(() => {
    if (typeof window === 'undefined') return

    const mediaQuery = window.matchMedia(query)
    const handler = (e: MediaQueryListEvent) => setMatches(e.matches)

    setMatches(mediaQuery.matches)
    mediaQuery.addEventListener('change', handler)

    return () => mediaQuery.removeEventListener('change', handler)
  }, [query])

  return matches
}

export function usePrefersReducedMotion(): boolean {
  return useMediaQuery('(prefers-reduced-motion: reduce)')
}
```

### Step 3: Create useTour hook

Create `packages/core/src/hooks/use-tour.ts`:

```typescript
import { useContext, useCallback, useMemo } from 'react'
import { TourContext } from '../context/tour-context'
import type { TourStep } from '../types'

export interface UseTourReturn {
  // State
  isActive: boolean
  isLoading: boolean
  isTransitioning: boolean
  currentStep: TourStep | null
  currentStepIndex: number
  totalSteps: number
  isFirstStep: boolean
  isLastStep: boolean
  progress: number

  // Actions
  start: (stepIndex?: number) => void
  next: () => void
  prev: () => void
  goTo: (stepIndex: number) => void
  skip: () => void
  complete: () => void
  stop: () => void

  // Utilities
  isStepActive: (stepId: string) => boolean
  getStep: (stepId: string) => TourStep | undefined
}

export function useTour(tourId?: string): UseTourReturn {
  const context = useContext(TourContext)

  if (!context) {
    throw new Error('useTour must be used within a TourProvider')
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
  } = context

  // If tourId provided, only active if it matches
  const isThisTourActive = tourId ? isActive && activeTourId === tourId : isActive

  const start = useCallback(
    (stepIndex?: number) => {
      contextStart(tourId, stepIndex)
    },
    [contextStart, tourId]
  )

  const isFirstStep = currentStepIndex === 0
  const isLastStep = totalSteps > 0 && currentStepIndex === totalSteps - 1
  const progress = totalSteps > 0 ? (currentStepIndex + 1) / totalSteps : 0

  const isStepActive = useCallback(
    (stepId: string) => currentStep?.id === stepId,
    [currentStep]
  )

  const getStep = useCallback(
    (stepId: string) => tour?.steps.find((s) => s.id === stepId),
    [tour]
  )

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
  )
}
```

### Step 4: Create useStep hook

Create `packages/core/src/hooks/use-step.ts`:

```typescript
import { useContext, useCallback, useMemo, useState, useEffect } from 'react'
import { TourContext } from '../context/tour-context'
import { useElementPosition } from './use-element-position'

export interface UseStepReturn {
  isActive: boolean
  isVisible: boolean
  hasCompleted: boolean
  targetElement: HTMLElement | null
  targetRect: DOMRect | null
  show: () => void
  hide: () => void
  complete: () => void
}

export function useStep(stepId: string): UseStepReturn {
  const context = useContext(TourContext)

  if (!context) {
    throw new Error('useStep must be used within a TourProvider')
  }

  const { currentStep, goTo, tour, next } = context

  const step = tour?.steps.find((s) => s.id === stepId)
  const stepIndex = tour?.steps.findIndex((s) => s.id === stepId) ?? -1

  const isActive = currentStep?.id === stepId
  const [isVisible, setIsVisible] = useState(false)
  const [hasCompleted, setHasCompleted] = useState(false)

  const target = step?.target
  const targetSelector = typeof target === 'string' ? target : null
  const targetRef = typeof target === 'object' ? target?.current : null

  const { element: targetElement, rect: targetRect } = useElementPosition(
    targetSelector ?? targetRef
  )

  useEffect(() => {
    if (isActive) {
      setIsVisible(true)
    }
  }, [isActive])

  const show = useCallback(() => {
    if (stepIndex >= 0) {
      goTo(stepIndex)
    }
  }, [goTo, stepIndex])

  const hide = useCallback(() => {
    setIsVisible(false)
  }, [])

  const complete = useCallback(() => {
    setHasCompleted(true)
    next()
  }, [next])

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
  )
}
```

### Step 5: Create useSpotlight hook

Create `packages/core/src/hooks/use-spotlight.ts`:

```typescript
import { useState, useCallback, useMemo, useRef, useEffect } from 'react'
import type { SpotlightConfig } from '../types'
import { defaultSpotlightConfig } from '../types/config'

export interface UseSpotlightReturn {
  isVisible: boolean
  targetRect: DOMRect | null
  overlayStyle: React.CSSProperties
  cutoutStyle: React.CSSProperties
  show: (target: HTMLElement, config?: SpotlightConfig) => void
  hide: () => void
  update: () => void
}

export function useSpotlight(): UseSpotlightReturn {
  const [isVisible, setIsVisible] = useState(false)
  const [targetRect, setTargetRect] = useState<DOMRect | null>(null)
  const [config, setConfig] = useState<SpotlightConfig>(defaultSpotlightConfig)
  const targetRef = useRef<HTMLElement | null>(null)

  const updateRect = useCallback(() => {
    if (targetRef.current) {
      setTargetRect(targetRef.current.getBoundingClientRect())
    }
  }, [])

  useEffect(() => {
    if (!isVisible) return

    window.addEventListener('scroll', updateRect, true)
    window.addEventListener('resize', updateRect)

    return () => {
      window.removeEventListener('scroll', updateRect, true)
      window.removeEventListener('resize', updateRect)
    }
  }, [isVisible, updateRect])

  const show = useCallback((target: HTMLElement, spotlightConfig?: SpotlightConfig) => {
    targetRef.current = target
    setConfig({ ...defaultSpotlightConfig, ...spotlightConfig })
    setTargetRect(target.getBoundingClientRect())
    setIsVisible(true)
  }, [])

  const hide = useCallback(() => {
    setIsVisible(false)
    targetRef.current = null
    setTargetRect(null)
  }, [])

  const overlayStyle = useMemo<React.CSSProperties>(
    () => ({
      position: 'fixed',
      inset: 0,
      backgroundColor: 'transparent',
      transition: config.animate
        ? `all ${config.animationDuration ?? 300}ms ease-out`
        : undefined,
      pointerEvents: 'auto',
    }),
    [config]
  )

  const cutoutStyle = useMemo<React.CSSProperties>(() => {
    if (!targetRect) return {}

    const padding = config.padding ?? 8
    const borderRadius = config.borderRadius ?? 4

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
    }
  }, [targetRect, config])

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
  )
}
```

### Step 6: Create useKeyboardNavigation hook

Create `packages/core/src/hooks/use-keyboard.ts`:

```typescript
import { useEffect, useContext } from 'react'
import { TourContext } from '../context/tour-context'
import type { KeyboardConfig } from '../types'
import { defaultKeyboardConfig } from '../types/config'

export function useKeyboardNavigation(config?: KeyboardConfig): void {
  const context = useContext(TourContext)

  if (!context) {
    throw new Error('useKeyboardNavigation must be used within a TourProvider')
  }

  const { isActive, next, prev, skip } = context
  const mergedConfig = { ...defaultKeyboardConfig, ...config }

  useEffect(() => {
    if (!isActive || !mergedConfig.enabled) return

    const handleKeyDown = (event: KeyboardEvent) => {
      const { key } = event

      // Ignore if user is typing in an input
      if (
        document.activeElement instanceof HTMLInputElement ||
        document.activeElement instanceof HTMLTextAreaElement
      ) {
        return
      }

      if (mergedConfig.nextKeys?.includes(key)) {
        event.preventDefault()
        next()
      } else if (mergedConfig.prevKeys?.includes(key)) {
        event.preventDefault()
        prev()
      } else if (mergedConfig.exitKeys?.includes(key)) {
        event.preventDefault()
        skip()
      }
    }

    document.addEventListener('keydown', handleKeyDown)
    return () => document.removeEventListener('keydown', handleKeyDown)
  }, [isActive, mergedConfig, next, prev, skip])
}
```

### Step 7: Create useFocusTrap hook

Create `packages/core/src/hooks/use-focus-trap.ts`:

```typescript
import { useRef, useCallback, useEffect } from 'react'
import { getFocusableElements } from '../utils/dom'

export interface UseFocusTrapReturn {
  containerRef: React.RefObject<HTMLElement | null>
  activate: () => void
  deactivate: () => void
}

export function useFocusTrap(enabled = true): UseFocusTrapReturn {
  const containerRef = useRef<HTMLElement | null>(null)
  const previousActiveElement = useRef<HTMLElement | null>(null)
  const isTrapping = useRef(false)

  const handleKeyDown = useCallback((event: KeyboardEvent) => {
    if (!isTrapping.current || event.key !== 'Tab' || !containerRef.current) {
      return
    }

    const focusable = getFocusableElements(containerRef.current)
    if (focusable.length === 0) return

    const first = focusable[0]
    const last = focusable[focusable.length - 1]

    if (event.shiftKey && document.activeElement === first) {
      event.preventDefault()
      last.focus()
    } else if (!event.shiftKey && document.activeElement === last) {
      event.preventDefault()
      first.focus()
    }
  }, [])

  const activate = useCallback(() => {
    if (!enabled || !containerRef.current) return

    previousActiveElement.current = document.activeElement as HTMLElement
    isTrapping.current = true

    const focusable = getFocusableElements(containerRef.current)
    if (focusable.length > 0) {
      focusable[0].focus()
    }

    document.addEventListener('keydown', handleKeyDown)
  }, [enabled, handleKeyDown])

  const deactivate = useCallback(() => {
    isTrapping.current = false
    document.removeEventListener('keydown', handleKeyDown)

    if (previousActiveElement.current) {
      previousActiveElement.current.focus()
      previousActiveElement.current = null
    }
  }, [handleKeyDown])

  useEffect(() => {
    return () => {
      document.removeEventListener('keydown', handleKeyDown)
    }
  }, [handleKeyDown])

  return { containerRef, activate, deactivate }
}
```

### Step 8: Create usePersistence hook

Create `packages/core/src/hooks/use-persistence.ts`:

```typescript
import { useCallback, useMemo } from 'react'
import type { PersistenceConfig } from '../types'
import { defaultPersistenceConfig } from '../types/config'
import { createStorageAdapter, safeJSONParse, createPrefixedStorage } from '../utils/storage'

export interface UsePersistenceReturn {
  getCompletedTours: () => string[]
  getSkippedTours: () => string[]
  getDontShowAgain: (tourId: string) => boolean
  getLastStep: (tourId: string) => number | null
  markCompleted: (tourId: string) => void
  markSkipped: (tourId: string) => void
  setDontShowAgain: (tourId: string, value: boolean) => void
  saveStep: (tourId: string, stepIndex: number) => void
  reset: (tourId?: string) => void
}

export function usePersistence(config?: PersistenceConfig): UsePersistenceReturn {
  const mergedConfig = { ...defaultPersistenceConfig, ...config }

  const storage = useMemo(() => {
    const adapter = createStorageAdapter(mergedConfig.storage)
    return createPrefixedStorage(adapter, mergedConfig.keyPrefix ?? 'beacon')
  }, [mergedConfig.storage, mergedConfig.keyPrefix])

  const getCompletedTours = useCallback(() => {
    const data = storage.getItem('completed')
    return safeJSONParse<string[]>(data as string | null, [])
  }, [storage])

  const getSkippedTours = useCallback(() => {
    const data = storage.getItem('skipped')
    return safeJSONParse<string[]>(data as string | null, [])
  }, [storage])

  const getDontShowAgain = useCallback(
    (tourId: string) => {
      const data = storage.getItem(`dontShow:${tourId}`)
      return data === 'true'
    },
    [storage]
  )

  const getLastStep = useCallback(
    (tourId: string) => {
      const data = storage.getItem(`step:${tourId}`)
      return data ? parseInt(data as string, 10) : null
    },
    [storage]
  )

  const markCompleted = useCallback(
    (tourId: string) => {
      const completed = getCompletedTours()
      if (!completed.includes(tourId)) {
        completed.push(tourId)
        storage.setItem('completed', JSON.stringify(completed))
      }
    },
    [storage, getCompletedTours]
  )

  const markSkipped = useCallback(
    (tourId: string) => {
      const skipped = getSkippedTours()
      if (!skipped.includes(tourId)) {
        skipped.push(tourId)
        storage.setItem('skipped', JSON.stringify(skipped))
      }
    },
    [storage, getSkippedTours]
  )

  const setDontShowAgain = useCallback(
    (tourId: string, value: boolean) => {
      if (value) {
        storage.setItem(`dontShow:${tourId}`, 'true')
      } else {
        storage.removeItem(`dontShow:${tourId}`)
      }
    },
    [storage]
  )

  const saveStep = useCallback(
    (tourId: string, stepIndex: number) => {
      storage.setItem(`step:${tourId}`, String(stepIndex))
    },
    [storage]
  )

  const reset = useCallback(
    (tourId?: string) => {
      if (tourId) {
        storage.removeItem(`step:${tourId}`)
        storage.removeItem(`dontShow:${tourId}`)

        const completed = getCompletedTours().filter((id) => id !== tourId)
        storage.setItem('completed', JSON.stringify(completed))

        const skipped = getSkippedTours().filter((id) => id !== tourId)
        storage.setItem('skipped', JSON.stringify(skipped))
      } else {
        storage.removeItem('completed')
        storage.removeItem('skipped')
      }
    },
    [storage, getCompletedTours, getSkippedTours]
  )

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
  }
}
```

### Step 9: Create hooks index

Create `packages/core/src/hooks/index.ts`:

```typescript
export { useTour, type UseTourReturn } from './use-tour'
export { useStep, type UseStepReturn } from './use-step'
export { useSpotlight, type UseSpotlightReturn } from './use-spotlight'
export { useElementPosition } from './use-element-position'
export { useKeyboardNavigation } from './use-keyboard'
export { useFocusTrap, type UseFocusTrapReturn } from './use-focus-trap'
export { usePersistence, type UsePersistenceReturn } from './use-persistence'
export { useMediaQuery, usePrefersReducedMotion } from './use-media-query'
```

### Step 10: Update package entry

Update `packages/core/src/index.ts`:

```typescript
// Types
export * from './types'

// Context
export * from './context'

// Hooks
export * from './hooks'

// Utilities
export * from './utils'
```

## Key Requirements & Constraints

- [ ] All hooks follow React rules
- [ ] Hooks are properly memoized
- [ ] No memory leaks
- [ ] SSR compatible
- [ ] Proper error boundaries

## Verification

```bash
cd packages/core
pnpm typecheck
pnpm build
```

## Next Prompt

Continue with `08-react-components.md` to build the styled components.

