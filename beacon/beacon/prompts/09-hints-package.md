# Prompt 09: Hints Package

## Problem

Build the `@tour-kit/hints` package with HintsProvider, Hint component, and supporting components.

## Dependencies on Previous Prompts

- Prompts 01-07 complete (@tour-kit/core implemented)

## Supporting Information

### Target File Structure

```
packages/hints/
├── src/
│   ├── context/
│   │   ├── hints-context.ts
│   │   └── hints-provider.tsx
│   ├── components/
│   │   ├── hint.tsx
│   │   ├── hint-beacon.tsx
│   │   ├── hint-tooltip.tsx
│   │   └── index.ts
│   ├── hooks/
│   │   ├── use-hints.ts
│   │   ├── use-hint.ts
│   │   └── index.ts
│   ├── utils/
│   │   └── cn.ts
│   ├── types/
│   │   └── index.ts
│   └── index.ts
├── package.json
├── tsconfig.json
└── tsup.config.ts
```

## Steps To Complete

### Step 1: Create hints package.json

Create `packages/hints/package.json`:

```json
{
  "name": "@tour-kit/hints",
  "version": "0.0.1",
  "description": "Persistent hints and hotspots for TourKit",
  "author": "TourKit",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/your-org/tour-kit.git",
    "directory": "packages/hints"
  },
  "keywords": ["react", "hints", "hotspots", "tooltips", "onboarding"],
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
    "typecheck": "tsc --noEmit",
    "clean": "rm -rf dist"
  },
  "dependencies": {
    "@tour-kit/core": "workspace:*",
    "@floating-ui/react": "^0.26.0",
    "clsx": "^2.1.0",
    "tailwind-merge": "^2.3.0"
  },
  "peerDependencies": {
    "react": "^18.0.0 || ^19.0.0",
    "react-dom": "^18.0.0 || ^19.0.0"
  },
  "devDependencies": {
    "@tour-kit/tsconfig": "workspace:*",
    "@types/react": "^18.3.0",
    "@types/react-dom": "^18.3.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "tsup": "^8.0.0",
    "typescript": "^5.5.0",
    "vitest": "^2.0.0"
  }
}
```

### Step 2: Create tsconfig.json and tsup.config.ts

Create `packages/hints/tsconfig.json`:

```json
{
  "extends": "@tour-kit/tsconfig/react-library.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Create `packages/hints/tsup.config.ts`:

```typescript
import { defineConfig } from 'tsup'

export default defineConfig({
  entry: ['src/index.ts'],
  format: ['cjs', 'esm'],
  dts: true,
  clean: true,
  external: ['react', 'react-dom', '@tour-kit/core'],
  treeshake: true,
  splitting: false,
  minify: true,
  sourcemap: true,
  target: 'es2020',
  outDir: 'dist',
  esbuildOptions(options) {
    options.banner = {
      js: '"use client";',
    }
  },
})
```

### Step 3: Create types

Create `packages/hints/src/types/index.ts`:

```typescript
import type React from 'react'
import type { Placement } from '@tour-kit/core'

export type BeaconPosition =
  | 'top-left'
  | 'top-right'
  | 'bottom-left'
  | 'bottom-right'
  | 'center'

export interface HintConfig {
  id: string
  target: string | React.RefObject<HTMLElement | null>
  content: React.ReactNode
  position?: BeaconPosition
  tooltipPlacement?: Placement
  pulse?: boolean
  autoShow?: boolean
  persist?: boolean
  onClick?: () => void
  onShow?: () => void
  onDismiss?: () => void
}

export interface HintState {
  id: string
  isOpen: boolean
  isDismissed: boolean
}

export interface HintsContextValue {
  hints: Map<string, HintState>
  activeHint: string | null
  registerHint: (id: string) => void
  unregisterHint: (id: string) => void
  showHint: (id: string) => void
  hideHint: (id: string) => void
  dismissHint: (id: string) => void
  resetHint: (id: string) => void
  resetAllHints: () => void
}
```

### Step 4: Create HintsContext and Provider

Create `packages/hints/src/context/hints-context.ts`:

```typescript
import { createContext, useContext } from 'react'
import type { HintsContextValue } from '../types'

export const HintsContext = createContext<HintsContextValue | null>(null)

HintsContext.displayName = 'HintsContext'

export function useHintsContext(): HintsContextValue {
  const context = useContext(HintsContext)
  if (!context) {
    throw new Error('useHintsContext must be used within a HintsProvider')
  }
  return context
}
```

Create `packages/hints/src/context/hints-provider.tsx`:

```typescript
import * as React from 'react'
import { HintsContext } from './hints-context'
import type { HintState, HintsContextValue } from '../types'

type HintsAction =
  | { type: 'REGISTER'; id: string }
  | { type: 'UNREGISTER'; id: string }
  | { type: 'SHOW'; id: string }
  | { type: 'HIDE'; id: string }
  | { type: 'DISMISS'; id: string }
  | { type: 'RESET'; id: string }
  | { type: 'RESET_ALL' }

interface HintsState {
  hints: Map<string, HintState>
  activeHint: string | null
}

function hintsReducer(state: HintsState, action: HintsAction): HintsState {
  switch (action.type) {
    case 'REGISTER': {
      const newHints = new Map(state.hints)
      if (!newHints.has(action.id)) {
        newHints.set(action.id, {
          id: action.id,
          isOpen: false,
          isDismissed: false,
        })
      }
      return { ...state, hints: newHints }
    }

    case 'UNREGISTER': {
      const newHints = new Map(state.hints)
      newHints.delete(action.id)
      return {
        ...state,
        hints: newHints,
        activeHint: state.activeHint === action.id ? null : state.activeHint,
      }
    }

    case 'SHOW': {
      const newHints = new Map(state.hints)
      const hint = newHints.get(action.id)
      if (hint && !hint.isDismissed) {
        // Close currently active hint
        if (state.activeHint && state.activeHint !== action.id) {
          const activeHint = newHints.get(state.activeHint)
          if (activeHint) {
            newHints.set(state.activeHint, { ...activeHint, isOpen: false })
          }
        }
        newHints.set(action.id, { ...hint, isOpen: true })
        return { hints: newHints, activeHint: action.id }
      }
      return state
    }

    case 'HIDE': {
      const newHints = new Map(state.hints)
      const hint = newHints.get(action.id)
      if (hint) {
        newHints.set(action.id, { ...hint, isOpen: false })
        return {
          hints: newHints,
          activeHint: state.activeHint === action.id ? null : state.activeHint,
        }
      }
      return state
    }

    case 'DISMISS': {
      const newHints = new Map(state.hints)
      const hint = newHints.get(action.id)
      if (hint) {
        newHints.set(action.id, { ...hint, isOpen: false, isDismissed: true })
        return {
          hints: newHints,
          activeHint: state.activeHint === action.id ? null : state.activeHint,
        }
      }
      return state
    }

    case 'RESET': {
      const newHints = new Map(state.hints)
      const hint = newHints.get(action.id)
      if (hint) {
        newHints.set(action.id, { ...hint, isDismissed: false })
      }
      return { ...state, hints: newHints }
    }

    case 'RESET_ALL': {
      const newHints = new Map(state.hints)
      newHints.forEach((hint, id) => {
        newHints.set(id, { ...hint, isDismissed: false })
      })
      return { ...state, hints: newHints }
    }

    default:
      return state
  }
}

interface HintsProviderProps {
  children: React.ReactNode
}

export function HintsProvider({ children }: HintsProviderProps) {
  const [state, dispatch] = React.useReducer(hintsReducer, {
    hints: new Map(),
    activeHint: null,
  })

  const contextValue = React.useMemo<HintsContextValue>(
    () => ({
      ...state,
      registerHint: (id) => dispatch({ type: 'REGISTER', id }),
      unregisterHint: (id) => dispatch({ type: 'UNREGISTER', id }),
      showHint: (id) => dispatch({ type: 'SHOW', id }),
      hideHint: (id) => dispatch({ type: 'HIDE', id }),
      dismissHint: (id) => dispatch({ type: 'DISMISS', id }),
      resetHint: (id) => dispatch({ type: 'RESET', id }),
      resetAllHints: () => dispatch({ type: 'RESET_ALL' }),
    }),
    [state]
  )

  return (
    <HintsContext.Provider value={contextValue}>
      {children}
    </HintsContext.Provider>
  )
}
```

### Step 5: Create hooks

Create `packages/hints/src/hooks/use-hints.ts`:

```typescript
import { useHintsContext } from '../context/hints-context'

export function useHints() {
  const context = useHintsContext()

  return {
    hints: Array.from(context.hints.values()),
    activeHint: context.activeHint,
    showHint: context.showHint,
    hideHint: context.hideHint,
    dismissHint: context.dismissHint,
    resetHint: context.resetHint,
    resetAllHints: context.resetAllHints,
    isHintVisible: (id: string) => context.hints.get(id)?.isOpen ?? false,
    isHintDismissed: (id: string) => context.hints.get(id)?.isDismissed ?? false,
  }
}
```

Create `packages/hints/src/hooks/use-hint.ts`:

```typescript
import { useEffect } from 'react'
import { useHintsContext } from '../context/hints-context'

export function useHint(id: string) {
  const context = useHintsContext()
  const hint = context.hints.get(id)

  useEffect(() => {
    context.registerHint(id)
    return () => context.unregisterHint(id)
  }, [id, context])

  return {
    isOpen: hint?.isOpen ?? false,
    isDismissed: hint?.isDismissed ?? false,
    show: () => context.showHint(id),
    hide: () => context.hideHint(id),
    dismiss: () => context.dismissHint(id),
    reset: () => context.resetHint(id),
  }
}
```

Create `packages/hints/src/hooks/index.ts`:

```typescript
export { useHints } from './use-hints'
export { useHint } from './use-hint'
```

### Step 6: Create utils

Create `packages/hints/src/utils/cn.ts`:

```typescript
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]): string {
  return twMerge(clsx(inputs))
}
```

### Step 7: Create HintBeacon

Create `packages/hints/src/components/hint-beacon.tsx`:

```typescript
import * as React from 'react'
import type { BeaconPosition } from '../types'
import { cn } from '../utils/cn'

interface HintBeaconProps {
  targetRect: DOMRect
  position: BeaconPosition
  pulse?: boolean
  isOpen?: boolean
  onClick: () => void
  className?: string
}

function getBeaconPosition(position: BeaconPosition, rect: DOMRect) {
  const offset = 4
  
  switch (position) {
    case 'top-left':
      return { top: rect.top - offset, left: rect.left - offset }
    case 'top-right':
      return { top: rect.top - offset, left: rect.right - offset }
    case 'bottom-left':
      return { top: rect.bottom - offset, left: rect.left - offset }
    case 'bottom-right':
      return { top: rect.bottom - offset, left: rect.right - offset }
    case 'center':
      return {
        top: rect.top + rect.height / 2 - 6,
        left: rect.left + rect.width / 2 - 6,
      }
    default:
      return { top: rect.top - offset, left: rect.right - offset }
  }
}

export function HintBeacon({
  targetRect,
  position,
  pulse = true,
  isOpen = false,
  onClick,
  className,
}: HintBeaconProps) {
  const pos = getBeaconPosition(position, targetRect)

  return (
    <button
      type="button"
      onClick={onClick}
      className={cn(
        'fixed z-50 h-3 w-3 rounded-full bg-primary',
        pulse && !isOpen && 'animate-pulse',
        className
      )}
      style={{
        top: pos.top,
        left: pos.left,
      }}
      aria-label="Show hint"
      aria-expanded={isOpen}
    >
      <span className="sr-only">Show hint</span>
    </button>
  )
}
```

### Step 8: Create HintTooltip

Create `packages/hints/src/components/hint-tooltip.tsx`:

```typescript
import * as React from 'react'
import {
  useFloating,
  autoUpdate,
  offset,
  flip,
  shift,
  useDismiss,
  useRole,
  useInteractions,
  FloatingPortal,
} from '@floating-ui/react'
import type { Placement } from '@tour-kit/core'
import { cn } from '../utils/cn'

interface HintTooltipProps {
  target: HTMLElement
  placement?: Placement
  children: React.ReactNode
  onClose: () => void
  className?: string
}

export function HintTooltip({
  target,
  placement = 'bottom',
  children,
  onClose,
  className,
}: HintTooltipProps) {
  const { refs, floatingStyles, context } = useFloating({
    open: true,
    placement,
    middleware: [offset(8), flip(), shift({ padding: 8 })],
    whileElementsMounted: autoUpdate,
  })

  const dismiss = useDismiss(context)
  const role = useRole(context, { role: 'tooltip' })
  const { getFloatingProps } = useInteractions([dismiss, role])

  React.useEffect(() => {
    refs.setReference(target)
  }, [target, refs])

  return (
    <FloatingPortal>
      <div
        ref={refs.setFloating}
        style={floatingStyles}
        {...getFloatingProps()}
        className={cn(
          'z-50 max-w-xs rounded-lg border bg-popover p-3 text-sm text-popover-foreground shadow-md',
          className
        )}
      >
        <button
          type="button"
          onClick={onClose}
          className="absolute right-2 top-2 rounded-sm opacity-70 hover:opacity-100 transition-opacity"
          aria-label="Dismiss hint"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="12"
            height="12"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            strokeWidth="2"
          >
            <path d="M18 6 6 18" />
            <path d="m6 6 12 12" />
          </svg>
        </button>
        <div className="pr-4">{children}</div>
      </div>
    </FloatingPortal>
  )
}
```

### Step 9: Create Hint

Create `packages/hints/src/components/hint.tsx`:

```typescript
import * as React from 'react'
import { useElementPosition } from '@tour-kit/core'
import { useHint } from '../hooks/use-hint'
import type { HintConfig } from '../types'
import { HintBeacon } from './hint-beacon'
import { HintTooltip } from './hint-tooltip'

type HintProps = HintConfig & {
  children?: React.ReactNode
  className?: string
}

export function Hint({
  id,
  target,
  content,
  children,
  position = 'top-right',
  tooltipPlacement = 'bottom',
  pulse = true,
  autoShow = false,
  persist = false,
  onClick,
  onShow,
  onDismiss,
  className,
}: HintProps) {
  const { isOpen, isDismissed, show, hide, dismiss } = useHint(id)

  const targetSelector = typeof target === 'string' ? target : null
  const targetRef = typeof target === 'object' ? target?.current : null

  const { element: targetElement, rect: targetRect } = useElementPosition(
    targetSelector ?? targetRef
  )

  React.useEffect(() => {
    if (autoShow && !isDismissed) {
      show()
      onShow?.()
    }
  }, [autoShow, isDismissed, show, onShow])

  const handleBeaconClick = () => {
    onClick?.()
    if (isOpen) {
      hide()
    } else {
      show()
      onShow?.()
    }
  }

  const handleDismiss = () => {
    if (persist) {
      dismiss()
    } else {
      hide()
    }
    onDismiss?.()
  }

  if (isDismissed || !targetElement || !targetRect) return null

  return (
    <>
      <HintBeacon
        targetRect={targetRect}
        position={position}
        pulse={pulse}
        isOpen={isOpen}
        onClick={handleBeaconClick}
      />
      {isOpen && (
        <HintTooltip
          target={targetElement}
          placement={tooltipPlacement}
          onClose={handleDismiss}
          className={className}
        >
          {children ?? content}
        </HintTooltip>
      )}
    </>
  )
}
```

Create `packages/hints/src/components/index.ts`:

```typescript
export { Hint } from './hint'
export { HintBeacon } from './hint-beacon'
export { HintTooltip } from './hint-tooltip'
```

### Step 10: Create package entry

Create `packages/hints/src/index.ts`:

```typescript
// Context
export { HintsProvider } from './context/hints-provider'
export { HintsContext } from './context/hints-context'

// Components
export { Hint, HintBeacon, HintTooltip } from './components'

// Hooks
export { useHints, useHint } from './hooks'

// Types
export type { HintConfig, HintState, BeaconPosition, HintsContextValue } from './types'
```

## Key Requirements & Constraints

- [ ] Hints can be shown/hidden independently
- [ ] Only one tooltip open at a time
- [ ] Dismissed hints persist (optionally)
- [ ] Beacon animation works
- [ ] Bundle size < 5KB gzipped

## Verification

```bash
cd packages/hints
pnpm install
pnpm typecheck
pnpm build
```

## Next Prompt

Continue with testing or documentation.

