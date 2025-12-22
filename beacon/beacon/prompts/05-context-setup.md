# Prompt 05: Context Setup

## Problem

Create the React context providers for TourKit - BeaconProvider and TourProvider.

## Dependencies on Previous Prompts

- Prompt 04 complete (types defined)

## Supporting Information

### Target File Structure

```
packages/core/src/
├── context/
│   ├── beacon-context.ts
│   ├── beacon-provider.tsx
│   ├── tour-context.ts
│   └── tour-provider.tsx
└── index.ts (updated)
```

## Steps To Complete

### Step 1: Create BeaconContext

Create `packages/core/src/context/beacon-context.ts`:

```typescript
import { createContext } from 'react'
import type { BeaconConfig } from '../types'

export interface BeaconContextValue {
  config: BeaconConfig
  // Global callbacks
  onTourStart?: (tourId: string) => void
  onTourComplete?: (tourId: string) => void
  onTourSkip?: (tourId: string, stepIndex: number) => void
  onStepView?: (tourId: string, stepId: string, stepIndex: number) => void
}

export const BeaconContext = createContext<BeaconContextValue | null>(null)

BeaconContext.displayName = 'BeaconContext'
```

### Step 2: Create BeaconProvider

Create `packages/core/src/context/beacon-provider.tsx`:

```typescript
import * as React from 'react'
import { BeaconContext, type BeaconContextValue } from './beacon-context'
import type { BeaconConfig } from '../types'

export interface BeaconProviderProps {
  children: React.ReactNode
  config?: BeaconConfig
  onTourStart?: (tourId: string) => void
  onTourComplete?: (tourId: string) => void
  onTourSkip?: (tourId: string, stepIndex: number) => void
  onStepView?: (tourId: string, stepId: string, stepIndex: number) => void
}

export function BeaconProvider({
  children,
  config = {},
  onTourStart,
  onTourComplete,
  onTourSkip,
  onStepView,
}: BeaconProviderProps) {
  const value = React.useMemo<BeaconContextValue>(
    () => ({
      config,
      onTourStart,
      onTourComplete,
      onTourSkip,
      onStepView,
    }),
    [config, onTourStart, onTourComplete, onTourSkip, onStepView]
  )

  return <BeaconContext.Provider value={value}>{children}</BeaconContext.Provider>
}
```

### Step 3: Create TourContext

Create `packages/core/src/context/tour-context.ts`:

```typescript
import { createContext, useContext } from 'react'
import type { TourContextValue } from '../types'

export const TourContext = createContext<TourContextValue | null>(null)

TourContext.displayName = 'TourContext'

export function useTourContext(): TourContextValue {
  const context = useContext(TourContext)
  
  if (!context) {
    throw new Error('useTourContext must be used within a TourProvider')
  }
  
  return context
}
```

### Step 4: Create TourProvider

Create `packages/core/src/context/tour-provider.tsx`:

```typescript
import * as React from 'react'
import { TourContext } from './tour-context'
import { BeaconContext } from './beacon-context'
import type { Tour, TourState, TourContextValue, TourStep, initialTourState } from '../types'

// Action types for reducer
type TourAction =
  | { type: 'START_TOUR'; tourId: string; stepIndex?: number }
  | { type: 'NEXT_STEP' }
  | { type: 'PREV_STEP' }
  | { type: 'GO_TO_STEP'; stepIndex: number }
  | { type: 'SKIP_TOUR' }
  | { type: 'COMPLETE_TOUR' }
  | { type: 'STOP_TOUR' }
  | { type: 'SET_LOADING'; isLoading: boolean }
  | { type: 'SET_TRANSITIONING'; isTransitioning: boolean }
  | { type: 'ADD_COMPLETED'; tourId: string }
  | { type: 'ADD_SKIPPED'; tourId: string }
  | { type: 'RESET'; tourId?: string }

interface TourReducerState extends TourState {
  tours: Map<string, Tour>
}

function tourReducer(state: TourReducerState, action: TourAction): TourReducerState {
  switch (action.type) {
    case 'START_TOUR': {
      const tour = state.tours.get(action.tourId)
      if (!tour) return state
      
      const stepIndex = action.stepIndex ?? tour.startAt ?? 0
      const step = tour.steps[stepIndex]
      
      return {
        ...state,
        tourId: action.tourId,
        isActive: true,
        currentStepIndex: stepIndex,
        currentStep: step ?? null,
        totalSteps: tour.steps.length,
        isLoading: false,
        isTransitioning: false,
      }
    }
    
    case 'NEXT_STEP': {
      const tour = state.tours.get(state.tourId ?? '')
      if (!tour || state.currentStepIndex >= tour.steps.length - 1) {
        return state
      }
      
      const nextIndex = state.currentStepIndex + 1
      return {
        ...state,
        currentStepIndex: nextIndex,
        currentStep: tour.steps[nextIndex] ?? null,
        isTransitioning: false,
      }
    }
    
    case 'PREV_STEP': {
      const tour = state.tours.get(state.tourId ?? '')
      if (!tour || state.currentStepIndex <= 0) {
        return state
      }
      
      const prevIndex = state.currentStepIndex - 1
      return {
        ...state,
        currentStepIndex: prevIndex,
        currentStep: tour.steps[prevIndex] ?? null,
        isTransitioning: false,
      }
    }
    
    case 'GO_TO_STEP': {
      const tour = state.tours.get(state.tourId ?? '')
      if (!tour || action.stepIndex < 0 || action.stepIndex >= tour.steps.length) {
        return state
      }
      
      return {
        ...state,
        currentStepIndex: action.stepIndex,
        currentStep: tour.steps[action.stepIndex] ?? null,
        isTransitioning: false,
      }
    }
    
    case 'SKIP_TOUR':
    case 'COMPLETE_TOUR':
    case 'STOP_TOUR': {
      return {
        ...state,
        tourId: null,
        isActive: false,
        currentStepIndex: 0,
        currentStep: null,
        totalSteps: 0,
        isLoading: false,
        isTransitioning: false,
      }
    }
    
    case 'SET_LOADING':
      return { ...state, isLoading: action.isLoading }
    
    case 'SET_TRANSITIONING':
      return { ...state, isTransitioning: action.isTransitioning }
    
    case 'ADD_COMPLETED':
      return {
        ...state,
        completedTours: [...state.completedTours, action.tourId],
      }
    
    case 'ADD_SKIPPED':
      return {
        ...state,
        skippedTours: [...state.skippedTours, action.tourId],
      }
    
    case 'RESET': {
      if (action.tourId) {
        return {
          ...state,
          completedTours: state.completedTours.filter((id) => id !== action.tourId),
          skippedTours: state.skippedTours.filter((id) => id !== action.tourId),
        }
      }
      return {
        ...state,
        completedTours: [],
        skippedTours: [],
      }
    }
    
    default:
      return state
  }
}

export interface TourProviderProps {
  children: React.ReactNode
  tours?: Tour[]
}

export function TourProvider({ children, tours = [] }: TourProviderProps) {
  const beaconContext = React.useContext(BeaconContext)
  const [data, setDataState] = React.useState<Record<string, unknown>>({})
  
  const initialState: TourReducerState = {
    tourId: null,
    isActive: false,
    currentStepIndex: 0,
    currentStep: null,
    totalSteps: 0,
    isLoading: false,
    isTransitioning: false,
    completedTours: [],
    skippedTours: [],
    tours: new Map(tours.map((t) => [t.id, t])),
  }
  
  const [state, dispatch] = React.useReducer(tourReducer, initialState)
  
  // Get current tour
  const currentTour = state.tourId ? state.tours.get(state.tourId) ?? null : null
  
  // Actions
  const start = React.useCallback(
    (tourId?: string, stepIndex?: number) => {
      const id = tourId ?? tours[0]?.id
      if (!id) return
      
      dispatch({ type: 'START_TOUR', tourId: id, stepIndex })
      beaconContext?.onTourStart?.(id)
      
      const tour = state.tours.get(id)
      tour?.onStart?.({ ...state, tour, data })
    },
    [tours, state, data, beaconContext]
  )
  
  const next = React.useCallback(() => {
    if (!state.isActive || !currentTour) return
    
    const isLastStep = state.currentStepIndex >= currentTour.steps.length - 1
    
    if (isLastStep) {
      dispatch({ type: 'ADD_COMPLETED', tourId: currentTour.id })
      dispatch({ type: 'COMPLETE_TOUR' })
      beaconContext?.onTourComplete?.(currentTour.id)
      currentTour.onComplete?.({ ...state, tour: currentTour, data })
    } else {
      dispatch({ type: 'SET_TRANSITIONING', isTransitioning: true })
      dispatch({ type: 'NEXT_STEP' })
      
      const nextStep = currentTour.steps[state.currentStepIndex + 1]
      if (nextStep) {
        beaconContext?.onStepView?.(currentTour.id, nextStep.id, state.currentStepIndex + 1)
        currentTour.onStepChange?.(nextStep, state.currentStepIndex + 1, {
          ...state,
          tour: currentTour,
          data,
        })
      }
    }
  }, [state, currentTour, data, beaconContext])
  
  const prev = React.useCallback(() => {
    if (!state.isActive || state.currentStepIndex <= 0 || !currentTour) return
    
    dispatch({ type: 'SET_TRANSITIONING', isTransitioning: true })
    dispatch({ type: 'PREV_STEP' })
    
    const prevStep = currentTour.steps[state.currentStepIndex - 1]
    if (prevStep) {
      beaconContext?.onStepView?.(currentTour.id, prevStep.id, state.currentStepIndex - 1)
      currentTour.onStepChange?.(prevStep, state.currentStepIndex - 1, {
        ...state,
        tour: currentTour,
        data,
      })
    }
  }, [state, currentTour, data, beaconContext])
  
  const goTo = React.useCallback(
    (stepIndex: number) => {
      if (!state.isActive || !currentTour) return
      
      dispatch({ type: 'SET_TRANSITIONING', isTransitioning: true })
      dispatch({ type: 'GO_TO_STEP', stepIndex })
      
      const step = currentTour.steps[stepIndex]
      if (step) {
        beaconContext?.onStepView?.(currentTour.id, step.id, stepIndex)
        currentTour.onStepChange?.(step, stepIndex, { ...state, tour: currentTour, data })
      }
    },
    [state, currentTour, data, beaconContext]
  )
  
  const skip = React.useCallback(() => {
    if (!state.isActive || !currentTour) return
    
    dispatch({ type: 'ADD_SKIPPED', tourId: currentTour.id })
    dispatch({ type: 'SKIP_TOUR' })
    beaconContext?.onTourSkip?.(currentTour.id, state.currentStepIndex)
    currentTour.onSkip?.({ ...state, tour: currentTour, data })
  }, [state, currentTour, data, beaconContext])
  
  const complete = React.useCallback(() => {
    if (!state.isActive || !currentTour) return
    
    dispatch({ type: 'ADD_COMPLETED', tourId: currentTour.id })
    dispatch({ type: 'COMPLETE_TOUR' })
    beaconContext?.onTourComplete?.(currentTour.id)
    currentTour.onComplete?.({ ...state, tour: currentTour, data })
  }, [state, currentTour, data, beaconContext])
  
  const stop = React.useCallback(() => {
    dispatch({ type: 'STOP_TOUR' })
  }, [])
  
  const setDontShowAgain = React.useCallback((_tourId: string, _value: boolean) => {
    // Implemented in usePersistence hook
  }, [])
  
  const reset = React.useCallback((tourId?: string) => {
    dispatch({ type: 'RESET', tourId })
  }, [])
  
  const setData = React.useCallback((key: string, value: unknown) => {
    setDataState((prev) => ({ ...prev, [key]: value }))
  }, [])
  
  const contextValue = React.useMemo<TourContextValue>(
    () => ({
      ...state,
      tour: currentTour,
      data,
      start,
      next,
      prev,
      goTo,
      skip,
      complete,
      stop,
      setDontShowAgain,
      reset,
      setData,
    }),
    [state, currentTour, data, start, next, prev, goTo, skip, complete, stop, reset, setData]
  )
  
  return <TourContext.Provider value={contextValue}>{children}</TourContext.Provider>
}
```

### Step 5: Create context index

Create `packages/core/src/context/index.ts`:

```typescript
export { BeaconContext, type BeaconContextValue } from './beacon-context'
export { BeaconProvider, type BeaconProviderProps } from './beacon-provider'
export { TourContext, useTourContext } from './tour-context'
export { TourProvider, type TourProviderProps } from './tour-provider'
```

### Step 6: Update package entry point

Update `packages/core/src/index.ts`:

```typescript
// Types
export * from './types'

// Context
export * from './context'
```

## Key Requirements & Constraints

- [ ] Contexts are properly typed
- [ ] Providers are composable
- [ ] State reducer handles all actions
- [ ] Lifecycle callbacks are invoked
- [ ] No memory leaks (proper cleanup)
- [ ] SSR safe (no window/document access in module scope)

## Verification

```bash
cd packages/core
pnpm typecheck
pnpm build
```

## Next Prompt

Continue with `06-utility-functions.md` to create utility functions.

