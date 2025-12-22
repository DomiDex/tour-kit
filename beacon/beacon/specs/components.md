# Components Specification

## Overview

This document specifies all React components for `@tour-kit/react` and `@tour-kit/hints`.

---

## @tour-kit/react Components

### Tour

Main tour wrapper component that registers a tour and renders the active step.

```tsx
// packages/react/src/components/tour/tour.tsx
import * as React from 'react';
import { TourProvider } from '@tour-kit/core';
import type { Tour as TourType, TourStep } from '@tour-kit/core';
import { TourCard } from '../card/tour-card';
import { TourOverlay } from '../overlay/tour-overlay';

interface TourProps {
  /** Unique tour identifier */
  id: string;
  /** Auto-start tour when mounted */
  autoStart?: boolean;
  /** Tour configuration options */
  config?: Omit<TourType, 'id' | 'steps'>;
  /** Lifecycle callbacks */
  onStart?: () => void;
  onComplete?: () => void;
  onSkip?: () => void;
  onStepChange?: (step: TourStep, index: number) => void;
  /** Tour step definitions (children) */
  children: React.ReactNode;
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
  // Collect TourStep children to build steps array
  const steps = React.useMemo(() => {
    const stepElements: TourStep[] = [];
    React.Children.forEach(children, (child) => {
      if (React.isValidElement(child) && child.type === TourStep) {
        stepElements.push(child.props as TourStep);
      }
    });
    return stepElements;
  }, [children]);

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
  };

  return (
    <TourProvider tours={[tour]}>
      <TourOverlay />
      <TourCard />
    </TourProvider>
  );
}
```

---

### TourStep

Declarative step definition (does not render anything).

```tsx
// packages/react/src/components/tour/tour-step.tsx
import type { TourStep as TourStepType } from '@tour-kit/core';

// This component is used declaratively and parsed by Tour
// It doesn't render anything on its own
export function TourStep(_props: TourStepType): null {
  return null;
}

TourStep.displayName = 'TourStep';
```

**Usage:**
```tsx
<Tour id="onboarding">
  <TourStep
    id="welcome"
    target="#welcome-btn"
    title="Welcome!"
    content="Let's get started."
    placement="bottom"
  />
  <TourStep
    id="dashboard"
    target="#dashboard"
    title="Dashboard"
    content="Your data overview."
    placement="right"
  />
</Tour>
```

---

### TourCard

The main tooltip card that displays step content.

```tsx
// packages/react/src/components/card/tour-card.tsx
import * as React from 'react';
import {
  useFloating,
  autoUpdate,
  offset,
  flip,
  shift,
  arrow,
} from '@floating-ui/react';
import { useTour, useFocusTrap } from '@tour-kit/core';
import { TourCardHeader } from './tour-card-header';
import { TourCardContent } from './tour-card-content';
import { TourCardFooter } from './tour-card-footer';
import { TourArrow } from '../primitives/tour-arrow';
import { TourPortal } from '../primitives/tour-portal';
import { cn } from '../../utils/cn';

interface TourCardProps {
  className?: string;
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
  } = useTour();

  const arrowRef = React.useRef<HTMLDivElement>(null);
  const { containerRef, activate, deactivate } = useFocusTrap(isActive);

  // Get target element
  const targetElement = React.useMemo(() => {
    if (!currentStep?.target) return null;
    if (typeof currentStep.target === 'string') {
      return document.querySelector<HTMLElement>(currentStep.target);
    }
    return currentStep.target.current;
  }, [currentStep?.target]);

  const { refs, floatingStyles, context, middlewareData } = useFloating({
    open: isActive,
    placement: currentStep?.placement ?? 'bottom',
    middleware: [
      offset(currentStep?.offset?.[1] ?? 12),
      flip({ fallbackAxisSideDirection: 'start' }),
      shift({ padding: 8 }),
      arrow({ element: arrowRef }),
    ],
    whileElementsMounted: autoUpdate,
  });

  // Set reference element
  React.useEffect(() => {
    if (targetElement) {
      refs.setReference(targetElement);
    }
  }, [targetElement, refs]);

  // Focus trap
  React.useEffect(() => {
    if (isActive) {
      activate();
    } else {
      deactivate();
    }
  }, [isActive, activate, deactivate]);

  if (!isActive || !currentStep) return null;

  const showNavigation = currentStep.showNavigation ?? true;
  const showClose = currentStep.showClose ?? true;
  const showProgress = currentStep.showProgress ?? true;

  return (
    <TourPortal>
      <div
        ref={(node) => {
          refs.setFloating(node);
          if (node) {
            (containerRef as React.MutableRefObject<HTMLElement | null>).current = node;
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
          onClose={skip}
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

        <TourArrow
          ref={arrowRef}
          context={context}
          middlewareData={middlewareData}
        />
      </div>
    </TourPortal>
  );
}
```

---

### TourCardHeader

Header section of the tour card.

```tsx
// packages/react/src/components/card/tour-card-header.tsx
import * as React from 'react';
import { X } from 'lucide-react';
import { cn } from '../../utils/cn';

interface TourCardHeaderProps {
  title?: React.ReactNode;
  titleId: string;
  showClose?: boolean;
  onClose?: () => void;
  className?: string;
}

export function TourCardHeader({
  title,
  titleId,
  showClose = true,
  onClose,
  className,
}: TourCardHeaderProps) {
  if (!title && !showClose) return null;

  return (
    <div className={cn('flex items-start justify-between gap-2', className)}>
      {title && (
        <h3
          id={titleId}
          className="font-semibold leading-none tracking-tight"
        >
          {title}
        </h3>
      )}
      {showClose && (
        <button
          type="button"
          onClick={onClose}
          className="rounded-sm opacity-70 ring-offset-background transition-opacity hover:opacity-100 focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2"
          aria-label="Close tour"
        >
          <X className="h-4 w-4" />
        </button>
      )}
    </div>
  );
}
```

---

### TourCardContent

Content section of the tour card.

```tsx
// packages/react/src/components/card/tour-card-content.tsx
import * as React from 'react';
import { cn } from '../../utils/cn';

interface TourCardContentProps {
  content: React.ReactNode;
  className?: string;
}

export function TourCardContent({ content, className }: TourCardContentProps) {
  return (
    <div className={cn('py-3 text-sm text-muted-foreground', className)}>
      {content}
    </div>
  );
}
```

---

### TourCardFooter

Footer with navigation and progress.

```tsx
// packages/react/src/components/card/tour-card-footer.tsx
import * as React from 'react';
import { TourNavigation } from '../navigation/tour-navigation';
import { TourProgress } from '../navigation/tour-progress';
import { cn } from '../../utils/cn';

interface TourCardFooterProps {
  currentStep: number;
  totalSteps: number;
  showNavigation?: boolean;
  showProgress?: boolean;
  isFirstStep: boolean;
  isLastStep: boolean;
  onPrev: () => void;
  onNext: () => void;
  onSkip: () => void;
  className?: string;
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
      {showProgress && (
        <TourProgress current={currentStep} total={totalSteps} />
      )}
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
  );
}
```

---

### TourNavigation

Navigation buttons (prev, next, skip).

```tsx
// packages/react/src/components/navigation/tour-navigation.tsx
import * as React from 'react';
import { cn } from '../../utils/cn';

interface TourNavigationProps {
  isFirstStep: boolean;
  isLastStep: boolean;
  onPrev: () => void;
  onNext: () => void;
  onSkip?: () => void;
  className?: string;
  prevLabel?: string;
  nextLabel?: string;
  finishLabel?: string;
  skipLabel?: string;
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
          className="text-sm text-muted-foreground hover:text-foreground"
        >
          {skipLabel}
        </button>
      )}
      {!isFirstStep && (
        <button
          type="button"
          onClick={onPrev}
          className="inline-flex items-center justify-center rounded-md border border-input bg-background px-3 py-1.5 text-sm font-medium hover:bg-accent hover:text-accent-foreground"
        >
          {prevLabel}
        </button>
      )}
      <button
        type="button"
        onClick={onNext}
        className="inline-flex items-center justify-center rounded-md bg-primary px-3 py-1.5 text-sm font-medium text-primary-foreground hover:bg-primary/90"
      >
        {isLastStep ? finishLabel : nextLabel}
      </button>
    </div>
  );
}
```

---

### TourProgress

Progress indicator (dots or bar).

```tsx
// packages/react/src/components/navigation/tour-progress.tsx
import * as React from 'react';
import { cn } from '../../utils/cn';

interface TourProgressProps {
  current: number;
  total: number;
  variant?: 'dots' | 'bar' | 'text';
  className?: string;
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
    );
  }

  if (variant === 'bar') {
    const percentage = (current / total) * 100;
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
    );
  }

  // Dots variant (default)
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
  );
}
```

---

### TourOverlay

Spotlight overlay with cutout.

```tsx
// packages/react/src/components/overlay/tour-overlay.tsx
import * as React from 'react';
import { useTour, useSpotlight, usePrefersReducedMotion } from '@tour-kit/core';
import { TourPortal } from '../primitives/tour-portal';
import { cn } from '../../utils/cn';

interface TourOverlayProps {
  className?: string;
  onClick?: () => void;
}

export function TourOverlay({ className, onClick }: TourOverlayProps) {
  const { isActive, currentStep, skip } = useTour();
  const { overlayStyle, cutoutStyle, show, hide, targetRect } = useSpotlight();
  const prefersReducedMotion = usePrefersReducedMotion();

  // Get target element
  const targetElement = React.useMemo(() => {
    if (!currentStep?.target) return null;
    if (typeof currentStep.target === 'string') {
      return document.querySelector<HTMLElement>(currentStep.target);
    }
    return currentStep.target.current;
  }, [currentStep?.target]);

  React.useEffect(() => {
    if (isActive && targetElement) {
      show(targetElement, {
        padding: currentStep?.spotlightPadding,
        borderRadius: currentStep?.spotlightRadius,
        animate: !prefersReducedMotion,
      });
    } else {
      hide();
    }
  }, [isActive, targetElement, currentStep, show, hide, prefersReducedMotion]);

  if (!isActive) return null;

  const handleClick = () => {
    onClick?.();
    // If clickToExit is enabled in config, call skip
  };

  return (
    <TourPortal>
      <div
        className={cn('fixed inset-0 z-40', className)}
        style={overlayStyle}
        onClick={handleClick}
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
  );
}
```

---

### TourArrow

Arrow element for tooltip.

```tsx
// packages/react/src/components/primitives/tour-arrow.tsx
import * as React from 'react';
import { FloatingArrow } from '@floating-ui/react';
import type { FloatingContext, MiddlewareData } from '@floating-ui/react';

interface TourArrowProps {
  context: FloatingContext;
  middlewareData: MiddlewareData;
  className?: string;
}

export const TourArrow = React.forwardRef<HTMLDivElement, TourArrowProps>(
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
    );
  }
);

TourArrow.displayName = 'TourArrow';
```

---

### TourPortal

Portal wrapper for rendering outside DOM hierarchy.

```tsx
// packages/react/src/components/primitives/tour-portal.tsx
import * as React from 'react';
import { createPortal } from 'react-dom';

interface TourPortalProps {
  children: React.ReactNode;
  container?: HTMLElement;
}

export function TourPortal({ children, container }: TourPortalProps) {
  const [mounted, setMounted] = React.useState(false);

  React.useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) return null;

  const portalContainer = container ?? document.body;

  return createPortal(children, portalContainer);
}
```

---

### TourClose

Standalone close button.

```tsx
// packages/react/src/components/navigation/tour-close.tsx
import * as React from 'react';
import { X } from 'lucide-react';
import { useTour } from '@tour-kit/core';
import { cn } from '../../utils/cn';

interface TourCloseProps {
  className?: string;
  'aria-label'?: string;
}

export function TourClose({
  className,
  'aria-label': ariaLabel = 'Close tour',
}: TourCloseProps) {
  const { skip } = useTour();

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
      <X className="h-4 w-4" />
    </button>
  );
}
```

---

## @tour-kit/hints Components

### Hint

Main hint component combining beacon and tooltip.

```tsx
// packages/hints/src/components/hint.tsx
import * as React from 'react';
import { useHint, useElementPosition } from '@tour-kit/core';
import type { HintConfig } from '../types';
import { HintBeacon } from './hint-beacon';
import { HintTooltip } from './hint-tooltip';

type HintProps = HintConfig & {
  children?: React.ReactNode;
  className?: string;
};

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
  const { isOpen, isDismissed, show, hide, dismiss } = useHint(id);
  const { element: targetElement, rect: targetRect } = useElementPosition(
    typeof target === 'string' ? target : target?.current ?? null
  );

  // Auto show on mount
  React.useEffect(() => {
    if (autoShow && !isDismissed) {
      show();
      onShow?.();
    }
  }, [autoShow, isDismissed, show, onShow]);

  const handleBeaconClick = () => {
    onClick?.();
    if (isOpen) {
      hide();
    } else {
      show();
      onShow?.();
    }
  };

  const handleDismiss = () => {
    if (persist) {
      dismiss();
    } else {
      hide();
    }
    onDismiss?.();
  };

  if (isDismissed || !targetElement || !targetRect) return null;

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
  );
}
```

---

### HintBeacon

The pulsing dot indicator.

```tsx
// packages/hints/src/components/hint-beacon.tsx
import * as React from 'react';
import type { BeaconPosition } from '../types';
import { cn } from '../utils/cn';

interface HintBeaconProps {
  targetRect: DOMRect;
  position: BeaconPosition;
  pulse?: boolean;
  isOpen?: boolean;
  onClick: () => void;
  className?: string;
}

const positionStyles: Record<BeaconPosition, { top: string; left: string }> = {
  'top-left': { top: '-4px', left: '-4px' },
  'top-right': { top: '-4px', left: 'calc(100% - 8px)' },
  'bottom-left': { top: 'calc(100% - 8px)', left: '-4px' },
  'bottom-right': { top: 'calc(100% - 8px)', left: 'calc(100% - 8px)' },
  center: { top: 'calc(50% - 6px)', left: 'calc(50% - 6px)' },
};

export function HintBeacon({
  targetRect,
  position,
  pulse = true,
  isOpen = false,
  onClick,
  className,
}: HintBeaconProps) {
  const pos = positionStyles[position];

  return (
    <button
      type="button"
      onClick={onClick}
      className={cn(
        'absolute z-50 h-3 w-3 rounded-full bg-primary',
        pulse && !isOpen && 'animate-beacon-pulse',
        className
      )}
      style={{
        position: 'fixed',
        top: targetRect.top + parseFloat(pos.top.replace('calc(', '').replace(')', '').replace('100%', String(targetRect.height)).replace('50%', String(targetRect.height / 2)).replace(' - 8px', '') || 0) - 4,
        left: targetRect.left + parseFloat(pos.left.replace('calc(', '').replace(')', '').replace('100%', String(targetRect.width)).replace('50%', String(targetRect.width / 2)).replace(' - 8px', '') || 0) - 4,
      }}
      aria-label="Show hint"
      aria-expanded={isOpen}
    >
      <span className="sr-only">Show hint</span>
    </button>
  );
}

// CSS animation (add to globals.css)
// @keyframes beacon-pulse {
//   0%, 100% { transform: scale(1); opacity: 1; }
//   50% { transform: scale(1.5); opacity: 0.5; }
// }
// .animate-beacon-pulse {
//   animation: beacon-pulse 2s ease-in-out infinite;
// }
```

---

### HintTooltip

The tooltip that appears when beacon is clicked.

```tsx
// packages/hints/src/components/hint-tooltip.tsx
import * as React from 'react';
import {
  useFloating,
  autoUpdate,
  offset,
  flip,
  shift,
  useClick,
  useDismiss,
  useRole,
  useInteractions,
  FloatingPortal,
} from '@floating-ui/react';
import type { Placement } from '@tour-kit/core';
import { X } from 'lucide-react';
import { cn } from '../utils/cn';

interface HintTooltipProps {
  target: HTMLElement;
  placement?: Placement;
  children: React.ReactNode;
  onClose: () => void;
  className?: string;
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
  });

  const dismiss = useDismiss(context);
  const role = useRole(context, { role: 'tooltip' });
  const { getFloatingProps } = useInteractions([dismiss, role]);

  React.useEffect(() => {
    refs.setReference(target);
  }, [target, refs]);

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
          className="absolute right-2 top-2 rounded-sm opacity-70 hover:opacity-100"
          aria-label="Dismiss hint"
        >
          <X className="h-3 w-3" />
        </button>
        {children}
      </div>
    </FloatingPortal>
  );
}
```

---

## Component Index

### @tour-kit/react exports

```typescript
// packages/react/src/index.ts
// Components
export { Tour } from './components/tour/tour';
export { TourStep } from './components/tour/tour-step';
export { TourCard } from './components/card/tour-card';
export { TourCardHeader } from './components/card/tour-card-header';
export { TourCardContent } from './components/card/tour-card-content';
export { TourCardFooter } from './components/card/tour-card-footer';
export { TourOverlay } from './components/overlay/tour-overlay';
export { TourNavigation } from './components/navigation/tour-navigation';
export { TourProgress } from './components/navigation/tour-progress';
export { TourClose } from './components/navigation/tour-close';
export { TourArrow } from './components/primitives/tour-arrow';
export { TourPortal } from './components/primitives/tour-portal';

// Re-export core
export * from '@tour-kit/core';
```

### @tour-kit/hints exports

```typescript
// packages/hints/src/index.ts
// Context
export { HintsProvider } from './context/hints-provider';

// Components
export { Hint } from './components/hint';
export { HintBeacon } from './components/hint-beacon';
export { HintTooltip } from './components/hint-tooltip';

// Hooks
export { useHints } from './hooks/use-hints';
export { useHint } from './hooks/use-hint';

// Types
export type { HintConfig, HintState, BeaconPosition } from './types';
```

