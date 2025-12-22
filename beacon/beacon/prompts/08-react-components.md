# Prompt 08: React Components

## Problem

Build the styled React components for `@tour-kit/react` - Tour, TourCard, TourOverlay, and supporting components.

## Dependencies on Previous Prompts

- Prompts 01-07 complete (@tour-kit/core fully implemented)

## Supporting Information

### Target File Structure

```
packages/react/
├── src/
│   ├── components/
│   │   ├── tour/
│   │   │   ├── tour.tsx
│   │   │   ├── tour-step.tsx
│   │   │   └── index.ts
│   │   ├── card/
│   │   │   ├── tour-card.tsx
│   │   │   ├── tour-card-header.tsx
│   │   │   ├── tour-card-content.tsx
│   │   │   ├── tour-card-footer.tsx
│   │   │   └── index.ts
│   │   ├── overlay/
│   │   │   ├── tour-overlay.tsx
│   │   │   └── index.ts
│   │   ├── navigation/
│   │   │   ├── tour-navigation.tsx
│   │   │   ├── tour-progress.tsx
│   │   │   ├── tour-close.tsx
│   │   │   └── index.ts
│   │   ├── primitives/
│   │   │   ├── tour-portal.tsx
│   │   │   ├── tour-arrow.tsx
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── utils/
│   │   └── cn.ts
│   └── index.ts
├── package.json
├── tsconfig.json
└── tsup.config.ts
```

### Reference Specification

See `../specs/components.md` for full implementations.

## Steps To Complete

### Step 1: Create react package.json

Create `packages/react/package.json`:

```json
{
  "name": "@tour-kit/react",
  "version": "0.0.1",
  "description": "Shadcn-styled React components for TourKit tours",
  "author": "TourKit",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/your-org/tour-kit.git",
    "directory": "packages/react"
  },
  "keywords": [
    "react",
    "shadcn",
    "onboarding",
    "tour",
    "product-tour",
    "components"
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
    "lucide-react": "^0.400.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "tsup": "^8.0.0",
    "typescript": "^5.5.0",
    "vitest": "^2.0.0"
  }
}
```

### Step 2: Create tsconfig.json and tsup.config.ts

Create `packages/react/tsconfig.json`:

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

Create `packages/react/tsup.config.ts`:

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

### Step 3: Create cn utility

Create `packages/react/src/utils/cn.ts`:

```typescript
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]): string {
  return twMerge(clsx(inputs))
}
```

### Step 4: Create TourPortal

Create `packages/react/src/components/primitives/tour-portal.tsx`:

```typescript
import * as React from 'react'
import { createPortal } from 'react-dom'

interface TourPortalProps {
  children: React.ReactNode
  container?: HTMLElement
}

export function TourPortal({ children, container }: TourPortalProps) {
  const [mounted, setMounted] = React.useState(false)

  React.useEffect(() => {
    setMounted(true)
  }, [])

  if (!mounted) return null

  return createPortal(children, container ?? document.body)
}
```

### Step 5: Create TourArrow

Create `packages/react/src/components/primitives/tour-arrow.tsx`:

```typescript
import * as React from 'react'
import { FloatingArrow } from '@floating-ui/react'
import type { FloatingContext } from '@floating-ui/react'

interface TourArrowProps {
  context: FloatingContext
  className?: string
}

export const TourArrow = React.forwardRef<SVGSVGElement, TourArrowProps>(
  ({ context, className }, ref) => {
    return (
      <FloatingArrow
        ref={ref}
        context={context}
        className={className}
        fill="hsl(var(--popover))"
        stroke="hsl(var(--border))"
        strokeWidth={1}
      />
    )
  }
)

TourArrow.displayName = 'TourArrow'
```

Create `packages/react/src/components/primitives/index.ts`:

```typescript
export { TourPortal } from './tour-portal'
export { TourArrow } from './tour-arrow'
```

### Step 6: Create TourOverlay

Create `packages/react/src/components/overlay/tour-overlay.tsx`:

```typescript
import * as React from 'react'
import { useTour, useSpotlight, usePrefersReducedMotion } from '@tour-kit/core'
import { TourPortal } from '../primitives/tour-portal'
import { cn } from '../../utils/cn'

interface TourOverlayProps {
  className?: string
  onClick?: () => void
}

export function TourOverlay({ className, onClick }: TourOverlayProps) {
  const { isActive, currentStep } = useTour()
  const { overlayStyle, cutoutStyle, show, hide, targetRect } = useSpotlight()
  const prefersReducedMotion = usePrefersReducedMotion()

  const targetElement = React.useMemo(() => {
    if (!currentStep?.target) return null
    if (typeof currentStep.target === 'string') {
      return document.querySelector<HTMLElement>(currentStep.target)
    }
    return currentStep.target.current
  }, [currentStep?.target])

  React.useEffect(() => {
    if (isActive && targetElement) {
      show(targetElement, {
        padding: currentStep?.spotlightPadding,
        borderRadius: currentStep?.spotlightRadius,
        animate: !prefersReducedMotion,
      })
    } else {
      hide()
    }
  }, [isActive, targetElement, currentStep, show, hide, prefersReducedMotion])

  if (!isActive) return null

  return (
    <TourPortal>
      <div
        className={cn('fixed inset-0 z-40', className)}
        style={overlayStyle}
        onClick={onClick}
        aria-hidden="true"
      >
        {targetRect && (
          <div
            className="absolute bg-transparent"
            style={{
              ...cutoutStyle,
              pointerEvents: currentStep?.interactive ? 'auto' : 'none',
            }}
          />
        )}
      </div>
    </TourPortal>
  )
}
```

Create `packages/react/src/components/overlay/index.ts`:

```typescript
export { TourOverlay } from './tour-overlay'
```

### Step 7: Create navigation components

Create `packages/react/src/components/navigation/tour-progress.tsx`:

```typescript
import * as React from 'react'
import { cn } from '../../utils/cn'

interface TourProgressProps {
  current: number
  total: number
  variant?: 'dots' | 'bar' | 'text'
  className?: string
}

export function TourProgress({
  current,
  total,
  variant = 'dots',
  className,
}: TourProgressProps) {
  if (variant === 'text') {
    return (
      <span className={cn('text-sm text-muted-foreground', className)}>
        {current} of {total}
      </span>
    )
  }

  if (variant === 'bar') {
    const percentage = (current / total) * 100
    return (
      <div
        className={cn(
          'h-1.5 w-20 overflow-hidden rounded-full bg-secondary',
          className
        )}
      >
        <div
          className="h-full bg-primary transition-all duration-300"
          style={{ width: `${percentage}%` }}
        />
      </div>
    )
  }

  return (
    <div className={cn('flex gap-1', className)}>
      {Array.from({ length: total }, (_, i) => (
        <div
          key={i}
          className={cn(
            'h-2 w-2 rounded-full transition-colors',
            i + 1 === current ? 'bg-primary' : 'bg-secondary'
          )}
        />
      ))}
    </div>
  )
}
```

Create `packages/react/src/components/navigation/tour-navigation.tsx`:

```typescript
import * as React from 'react'
import { cn } from '../../utils/cn'

interface TourNavigationProps {
  isFirstStep: boolean
  isLastStep: boolean
  onPrev: () => void
  onNext: () => void
  onSkip?: () => void
  className?: string
  prevLabel?: string
  nextLabel?: string
  finishLabel?: string
  skipLabel?: string
}

export function TourNavigation({
  isFirstStep,
  isLastStep,
  onPrev,
  onNext,
  onSkip,
  className,
  prevLabel = 'Back',
  nextLabel = 'Next',
  finishLabel = 'Finish',
  skipLabel = 'Skip',
}: TourNavigationProps) {
  return (
    <div className={cn('flex items-center gap-2', className)}>
      {onSkip && !isLastStep && (
        <button
          type="button"
          onClick={onSkip}
          className="text-sm text-muted-foreground hover:text-foreground transition-colors"
        >
          {skipLabel}
        </button>
      )}
      {!isFirstStep && (
        <button
          type="button"
          onClick={onPrev}
          className="inline-flex items-center justify-center rounded-md border border-input bg-background px-3 py-1.5 text-sm font-medium hover:bg-accent hover:text-accent-foreground transition-colors"
        >
          {prevLabel}
        </button>
      )}
      <button
        type="button"
        onClick={onNext}
        className="inline-flex items-center justify-center rounded-md bg-primary px-3 py-1.5 text-sm font-medium text-primary-foreground hover:bg-primary/90 transition-colors"
      >
        {isLastStep ? finishLabel : nextLabel}
      </button>
    </div>
  )
}
```

Create `packages/react/src/components/navigation/tour-close.tsx`:

```typescript
import * as React from 'react'
import { useTour } from '@tour-kit/core'
import { cn } from '../../utils/cn'

interface TourCloseProps {
  className?: string
  'aria-label'?: string
}

export function TourClose({
  className,
  'aria-label': ariaLabel = 'Close tour',
}: TourCloseProps) {
  const { skip } = useTour()

  return (
    <button
      type="button"
      onClick={skip}
      className={cn(
        'rounded-sm opacity-70 ring-offset-background transition-opacity hover:opacity-100 focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2',
        className
      )}
      aria-label={ariaLabel}
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="16"
        height="16"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        strokeWidth="2"
        strokeLinecap="round"
        strokeLinejoin="round"
      >
        <path d="M18 6 6 18" />
        <path d="m6 6 12 12" />
      </svg>
    </button>
  )
}
```

Create `packages/react/src/components/navigation/index.ts`:

```typescript
export { TourProgress } from './tour-progress'
export { TourNavigation } from './tour-navigation'
export { TourClose } from './tour-close'
```

### Step 8: Create TourCard components

Create `packages/react/src/components/card/tour-card-header.tsx`:

```typescript
import * as React from 'react'
import { cn } from '../../utils/cn'
import { TourClose } from '../navigation/tour-close'

interface TourCardHeaderProps {
  title?: React.ReactNode
  titleId: string
  showClose?: boolean
  onClose?: () => void
  className?: string
}

export function TourCardHeader({
  title,
  titleId,
  showClose = true,
  onClose,
  className,
}: TourCardHeaderProps) {
  if (!title && !showClose) return null

  return (
    <div className={cn('flex items-start justify-between gap-2', className)}>
      {title && (
        <h3 id={titleId} className="font-semibold leading-none tracking-tight">
          {title}
        </h3>
      )}
      {showClose && <TourClose />}
    </div>
  )
}
```

Create `packages/react/src/components/card/tour-card-content.tsx`:

```typescript
import * as React from 'react'
import { cn } from '../../utils/cn'

interface TourCardContentProps {
  content: React.ReactNode
  className?: string
}

export function TourCardContent({ content, className }: TourCardContentProps) {
  return (
    <div className={cn('py-3 text-sm text-muted-foreground', className)}>
      {content}
    </div>
  )
}
```

Create `packages/react/src/components/card/tour-card-footer.tsx`:

```typescript
import * as React from 'react'
import { TourNavigation } from '../navigation/tour-navigation'
import { TourProgress } from '../navigation/tour-progress'
import { cn } from '../../utils/cn'

interface TourCardFooterProps {
  currentStep: number
  totalSteps: number
  showNavigation?: boolean
  showProgress?: boolean
  isFirstStep: boolean
  isLastStep: boolean
  onPrev: () => void
  onNext: () => void
  onSkip: () => void
  className?: string
}

export function TourCardFooter({
  currentStep,
  totalSteps,
  showNavigation = true,
  showProgress = true,
  isFirstStep,
  isLastStep,
  onPrev,
  onNext,
  onSkip,
  className,
}: TourCardFooterProps) {
  return (
    <div className={cn('flex items-center justify-between pt-2', className)}>
      {showProgress && <TourProgress current={currentStep} total={totalSteps} />}
      {showNavigation && (
        <TourNavigation
          isFirstStep={isFirstStep}
          isLastStep={isLastStep}
          onPrev={onPrev}
          onNext={onNext}
          onSkip={onSkip}
        />
      )}
    </div>
  )
}
```

Create `packages/react/src/components/card/tour-card.tsx`:

```typescript
import * as React from 'react'
import {
  useFloating,
  autoUpdate,
  offset,
  flip,
  shift,
  arrow,
} from '@floating-ui/react'
import { useTour, useFocusTrap } from '@tour-kit/core'
import { TourCardHeader } from './tour-card-header'
import { TourCardContent } from './tour-card-content'
import { TourCardFooter } from './tour-card-footer'
import { TourArrow } from '../primitives/tour-arrow'
import { TourPortal } from '../primitives/tour-portal'
import { cn } from '../../utils/cn'

interface TourCardProps {
  className?: string
}

export function TourCard({ className }: TourCardProps) {
  const {
    isActive,
    currentStep,
    currentStepIndex,
    totalSteps,
    next,
    prev,
    skip,
    isFirstStep,
    isLastStep,
  } = useTour()

  const arrowRef = React.useRef<SVGSVGElement>(null)
  const { containerRef, activate, deactivate } = useFocusTrap(isActive)

  const targetElement = React.useMemo(() => {
    if (!currentStep?.target) return null
    if (typeof currentStep.target === 'string') {
      return document.querySelector<HTMLElement>(currentStep.target)
    }
    return currentStep.target.current
  }, [currentStep?.target])

  const { refs, floatingStyles, context } = useFloating({
    open: isActive,
    placement: currentStep?.placement ?? 'bottom',
    middleware: [
      offset(currentStep?.offset?.[1] ?? 12),
      flip({ fallbackAxisSideDirection: 'start' }),
      shift({ padding: 8 }),
      arrow({ element: arrowRef }),
    ],
    whileElementsMounted: autoUpdate,
  })

  React.useEffect(() => {
    if (targetElement) {
      refs.setReference(targetElement)
    }
  }, [targetElement, refs])

  React.useEffect(() => {
    if (isActive) {
      activate()
    } else {
      deactivate()
    }
  }, [isActive, activate, deactivate])

  if (!isActive || !currentStep) return null

  const showNavigation = currentStep.showNavigation ?? true
  const showClose = currentStep.showClose ?? true
  const showProgress = currentStep.showProgress ?? true

  return (
    <TourPortal>
      <div
        ref={(node) => {
          refs.setFloating(node)
          if (containerRef) {
            ;(containerRef as React.MutableRefObject<HTMLElement | null>).current = node
          }
        }}
        style={floatingStyles}
        className={cn(
          'z-50 w-80 rounded-lg border bg-popover p-4 text-popover-foreground shadow-lg',
          currentStep.className,
          className
        )}
        role="dialog"
        aria-modal="true"
        aria-labelledby={`tour-step-title-${currentStep.id}`}
      >
        <TourCardHeader
          title={currentStep.title}
          titleId={`tour-step-title-${currentStep.id}`}
          showClose={showClose}
        />

        <TourCardContent content={currentStep.content} />

        <TourCardFooter
          currentStep={currentStepIndex + 1}
          totalSteps={totalSteps}
          showNavigation={showNavigation}
          showProgress={showProgress}
          isFirstStep={isFirstStep}
          isLastStep={isLastStep}
          onPrev={prev}
          onNext={next}
          onSkip={skip}
        />

        <TourArrow ref={arrowRef} context={context} />
      </div>
    </TourPortal>
  )
}
```

Create `packages/react/src/components/card/index.ts`:

```typescript
export { TourCard } from './tour-card'
export { TourCardHeader } from './tour-card-header'
export { TourCardContent } from './tour-card-content'
export { TourCardFooter } from './tour-card-footer'
```

### Step 9: Create Tour components

Create `packages/react/src/components/tour/tour-step.tsx`:

```typescript
import type { TourStep as TourStepType } from '@tour-kit/core'

export function TourStep(_props: TourStepType): null {
  return null
}

TourStep.displayName = 'TourStep'
```

Create `packages/react/src/components/tour/tour.tsx`:

```typescript
import * as React from 'react'
import { TourProvider, type Tour as TourType, type TourStep as TourStepType } from '@tour-kit/core'
import { TourCard } from '../card/tour-card'
import { TourOverlay } from '../overlay/tour-overlay'
import { TourStep } from './tour-step'

interface TourProps {
  id: string
  autoStart?: boolean
  config?: Omit<TourType, 'id' | 'steps'>
  onStart?: () => void
  onComplete?: () => void
  onSkip?: () => void
  onStepChange?: (step: TourStepType, index: number) => void
  children: React.ReactNode
}

export function Tour({
  id,
  autoStart = false,
  config,
  onStart,
  onComplete,
  onSkip,
  onStepChange,
  children,
}: TourProps) {
  const steps = React.useMemo(() => {
    const stepElements: TourStepType[] = []
    React.Children.forEach(children, (child) => {
      if (React.isValidElement(child) && child.type === TourStep) {
        stepElements.push(child.props as TourStepType)
      }
    })
    return stepElements
  }, [children])

  const tour: TourType = {
    id,
    steps,
    autoStart,
    ...config,
    onStart: onStart ? () => onStart() : undefined,
    onComplete: onComplete ? () => onComplete() : undefined,
    onSkip: onSkip ? () => onSkip() : undefined,
    onStepChange: onStepChange
      ? (step, index) => onStepChange(step, index)
      : undefined,
  }

  return (
    <TourProvider tours={[tour]}>
      <TourOverlay />
      <TourCard />
    </TourProvider>
  )
}
```

Create `packages/react/src/components/tour/index.ts`:

```typescript
export { Tour } from './tour'
export { TourStep } from './tour-step'
```

### Step 10: Create component and package index

Create `packages/react/src/components/index.ts`:

```typescript
export * from './tour'
export * from './card'
export * from './overlay'
export * from './navigation'
export * from './primitives'
```

Create `packages/react/src/index.ts`:

```typescript
// Components
export * from './components'

// Re-export core
export * from '@tour-kit/core'
```

## Key Requirements & Constraints

- [ ] Components use Shadcn styling patterns
- [ ] All components are properly typed
- [ ] Focus management works correctly
- [ ] ARIA attributes are properly set
- [ ] Animations respect reduced motion

## Verification

```bash
cd packages/react
pnpm install
pnpm typecheck
pnpm build
```

## Next Prompt

Continue with hints package or testing.

