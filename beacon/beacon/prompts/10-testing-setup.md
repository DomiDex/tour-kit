# Prompt 10: Testing Setup

## Problem

Set up Vitest testing for all packages with comprehensive test coverage for hooks, utilities, and components.

## Dependencies on Previous Prompts

- All previous prompts complete (packages built)

## Supporting Information

### Target File Structure

```
packages/core/
├── src/
│   └── __tests__/
│       ├── hooks/
│       │   ├── use-tour.test.tsx
│       │   ├── use-step.test.tsx
│       │   └── use-spotlight.test.tsx
│       ├── utils/
│       │   ├── dom.test.ts
│       │   ├── position.test.ts
│       │   └── storage.test.ts
│       └── setup.ts
└── vitest.config.ts
```

## Steps To Complete

### Step 1: Create Vitest config for core

Create `packages/core/vitest.config.ts`:

```typescript
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/__tests__/setup.ts'],
    include: ['src/**/*.{test,spec}.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/__tests__/',
        'dist/',
        '**/*.d.ts',
      ],
      thresholds: {
        statements: 80,
        branches: 80,
        functions: 80,
        lines: 80,
      },
    },
  },
})
```

### Step 2: Create test setup file

Create `packages/core/src/__tests__/setup.ts`:

```typescript
import { afterEach, vi } from 'vitest'
import { cleanup } from '@testing-library/react'
import '@testing-library/jest-dom/vitest'

// Cleanup after each test
afterEach(() => {
  cleanup()
  vi.clearAllMocks()
})

// Mock ResizeObserver
class ResizeObserverMock {
  observe = vi.fn()
  unobserve = vi.fn()
  disconnect = vi.fn()
}

vi.stubGlobal('ResizeObserver', ResizeObserverMock)

// Mock matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation((query) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
})

// Mock scrollTo
window.scrollTo = vi.fn()
Element.prototype.scrollIntoView = vi.fn()
```

### Step 3: Create DOM utility tests

Create `packages/core/src/__tests__/utils/dom.test.ts`:

```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest'
import {
  getElement,
  isElementVisible,
  isElementPartiallyVisible,
  waitForElement,
  getFocusableElements,
} from '../../utils/dom'

describe('DOM Utilities', () => {
  let container: HTMLDivElement

  beforeEach(() => {
    container = document.createElement('div')
    document.body.appendChild(container)
  })

  afterEach(() => {
    document.body.removeChild(container)
  })

  describe('getElement', () => {
    it('returns null for null input', () => {
      expect(getElement(null)).toBeNull()
    })

    it('returns element for valid selector', () => {
      container.innerHTML = '<div id="test">Test</div>'
      const element = getElement('#test')
      expect(element).toBeInstanceOf(HTMLElement)
      expect(element?.textContent).toBe('Test')
    })

    it('returns element directly if passed', () => {
      const div = document.createElement('div')
      expect(getElement(div)).toBe(div)
    })

    it('returns element from ref object', () => {
      const div = document.createElement('div')
      const ref = { current: div }
      expect(getElement(ref)).toBe(div)
    })
  })

  describe('isElementVisible', () => {
    it('returns true for element in viewport', () => {
      const element = document.createElement('div')
      container.appendChild(element)
      
      vi.spyOn(element, 'getBoundingClientRect').mockReturnValue({
        top: 100,
        left: 100,
        bottom: 200,
        right: 200,
        width: 100,
        height: 100,
        x: 100,
        y: 100,
        toJSON: () => {},
      })

      expect(isElementVisible(element)).toBe(true)
    })
  })

  describe('getFocusableElements', () => {
    it('returns focusable elements in container', () => {
      container.innerHTML = `
        <button>Button</button>
        <input type="text" />
        <a href="#">Link</a>
        <div tabindex="0">Focusable div</div>
        <div>Not focusable</div>
      `

      const focusable = getFocusableElements(container)
      expect(focusable).toHaveLength(4)
    })

    it('excludes disabled elements', () => {
      container.innerHTML = `
        <button disabled>Disabled Button</button>
        <button>Enabled Button</button>
      `

      const focusable = getFocusableElements(container)
      expect(focusable).toHaveLength(1)
    })
  })

  describe('waitForElement', () => {
    it('resolves immediately if element exists', async () => {
      container.innerHTML = '<div id="existing">Exists</div>'
      const element = await waitForElement('#existing')
      expect(element.textContent).toBe('Exists')
    })

    it('resolves when element appears', async () => {
      const promise = waitForElement('#dynamic', 1000)
      
      setTimeout(() => {
        container.innerHTML = '<div id="dynamic">Dynamic</div>'
      }, 100)

      const element = await promise
      expect(element.textContent).toBe('Dynamic')
    })

    it('rejects on timeout', async () => {
      await expect(waitForElement('#nonexistent', 100)).rejects.toThrow(
        'Element "#nonexistent" not found within 100ms'
      )
    })
  })
})
```

### Step 4: Create position utility tests

Create `packages/core/src/__tests__/utils/position.test.ts`:

```typescript
import { describe, it, expect } from 'vitest'
import {
  parsePlacement,
  getOppositeSide,
  calculatePosition,
  wouldOverflow,
} from '../../utils/position'

describe('Position Utilities', () => {
  describe('parsePlacement', () => {
    it('parses simple side', () => {
      expect(parsePlacement('top')).toEqual({ side: 'top', alignment: 'center' })
      expect(parsePlacement('bottom')).toEqual({ side: 'bottom', alignment: 'center' })
    })

    it('parses side with alignment', () => {
      expect(parsePlacement('top-start')).toEqual({ side: 'top', alignment: 'start' })
      expect(parsePlacement('bottom-end')).toEqual({ side: 'bottom', alignment: 'end' })
    })
  })

  describe('getOppositeSide', () => {
    it('returns opposite sides', () => {
      expect(getOppositeSide('top')).toBe('bottom')
      expect(getOppositeSide('bottom')).toBe('top')
      expect(getOppositeSide('left')).toBe('right')
      expect(getOppositeSide('right')).toBe('left')
    })
  })

  describe('calculatePosition', () => {
    const targetRect = { x: 100, y: 100, width: 100, height: 50 }
    const tooltipSize = { width: 200, height: 100 }

    it('calculates bottom center position', () => {
      const position = calculatePosition(targetRect, tooltipSize, 'bottom')
      expect(position.y).toBe(150) // 100 + 50
      expect(position.x).toBe(50) // 100 + (100 - 200) / 2
    })

    it('calculates top center position', () => {
      const position = calculatePosition(targetRect, tooltipSize, 'top')
      expect(position.y).toBe(0) // 100 - 100
      expect(position.x).toBe(50)
    })
  })

  describe('wouldOverflow', () => {
    const viewport = { width: 1000, height: 800 }
    const dimensions = { width: 200, height: 100 }

    it('detects no overflow', () => {
      const overflow = wouldOverflow({ x: 100, y: 100 }, dimensions, viewport)
      expect(overflow).toEqual({
        top: false,
        right: false,
        bottom: false,
        left: false,
      })
    })

    it('detects left overflow', () => {
      const overflow = wouldOverflow({ x: -50, y: 100 }, dimensions, viewport)
      expect(overflow.left).toBe(true)
    })

    it('detects right overflow', () => {
      const overflow = wouldOverflow({ x: 900, y: 100 }, dimensions, viewport)
      expect(overflow.right).toBe(true)
    })
  })
})
```

### Step 5: Create storage utility tests

Create `packages/core/src/__tests__/utils/storage.test.ts`:

```typescript
import { describe, it, expect, beforeEach, vi } from 'vitest'
import {
  createStorageAdapter,
  createNoopStorage,
  createCookieStorage,
  safeJSONParse,
  createPrefixedStorage,
} from '../../utils/storage'

describe('Storage Utilities', () => {
  describe('createNoopStorage', () => {
    it('returns null for getItem', () => {
      const storage = createNoopStorage()
      expect(storage.getItem('key')).toBeNull()
    })

    it('setItem does nothing', () => {
      const storage = createNoopStorage()
      expect(() => storage.setItem('key', 'value')).not.toThrow()
    })
  })

  describe('createStorageAdapter', () => {
    it('returns localStorage by default', () => {
      const storage = createStorageAdapter('localStorage')
      expect(storage).toBe(window.localStorage)
    })

    it('returns sessionStorage when specified', () => {
      const storage = createStorageAdapter('sessionStorage')
      expect(storage).toBe(window.sessionStorage)
    })

    it('returns custom storage when passed', () => {
      const customStorage = {
        getItem: vi.fn(),
        setItem: vi.fn(),
        removeItem: vi.fn(),
      }
      const storage = createStorageAdapter(customStorage)
      expect(storage).toBe(customStorage)
    })
  })

  describe('safeJSONParse', () => {
    it('parses valid JSON', () => {
      expect(safeJSONParse('{"a":1}', {})).toEqual({ a: 1 })
    })

    it('returns fallback for null', () => {
      expect(safeJSONParse(null, { default: true })).toEqual({ default: true })
    })

    it('returns fallback for invalid JSON', () => {
      expect(safeJSONParse('invalid', [])).toEqual([])
    })
  })

  describe('createPrefixedStorage', () => {
    it('prefixes keys', () => {
      const mockStorage = {
        getItem: vi.fn(),
        setItem: vi.fn(),
        removeItem: vi.fn(),
      }

      const prefixed = createPrefixedStorage(mockStorage, 'test')
      
      prefixed.getItem('key')
      expect(mockStorage.getItem).toHaveBeenCalledWith('test:key')

      prefixed.setItem('key', 'value')
      expect(mockStorage.setItem).toHaveBeenCalledWith('test:key', 'value')

      prefixed.removeItem('key')
      expect(mockStorage.removeItem).toHaveBeenCalledWith('test:key')
    })
  })
})
```

### Step 6: Create useTour hook tests

Create `packages/core/src/__tests__/hooks/use-tour.test.tsx`:

```typescript
import { describe, it, expect, vi } from 'vitest'
import { renderHook, act } from '@testing-library/react'
import * as React from 'react'
import { useTour } from '../../hooks/use-tour'
import { TourProvider } from '../../context/tour-provider'
import type { Tour } from '../../types'

const testTour: Tour = {
  id: 'test-tour',
  steps: [
    { id: 'step-1', target: '#step1', content: 'Step 1' },
    { id: 'step-2', target: '#step2', content: 'Step 2' },
    { id: 'step-3', target: '#step3', content: 'Step 3' },
  ],
}

const wrapper = ({ children }: { children: React.ReactNode }) => (
  <TourProvider tours={[testTour]}>{children}</TourProvider>
)

describe('useTour', () => {
  it('throws error when used outside provider', () => {
    expect(() => {
      renderHook(() => useTour())
    }).toThrow('useTour must be used within a TourProvider')
  })

  it('returns inactive state initially', () => {
    const { result } = renderHook(() => useTour(), { wrapper })

    expect(result.current.isActive).toBe(false)
    expect(result.current.currentStep).toBeNull()
    expect(result.current.currentStepIndex).toBe(0)
    expect(result.current.totalSteps).toBe(0)
  })

  it('starts tour when start is called', () => {
    const { result } = renderHook(() => useTour(), { wrapper })

    act(() => {
      result.current.start()
    })

    expect(result.current.isActive).toBe(true)
    expect(result.current.currentStep?.id).toBe('step-1')
    expect(result.current.currentStepIndex).toBe(0)
    expect(result.current.totalSteps).toBe(3)
  })

  it('advances to next step', () => {
    const { result } = renderHook(() => useTour(), { wrapper })

    act(() => {
      result.current.start()
    })

    act(() => {
      result.current.next()
    })

    expect(result.current.currentStepIndex).toBe(1)
    expect(result.current.currentStep?.id).toBe('step-2')
  })

  it('goes to previous step', () => {
    const { result } = renderHook(() => useTour(), { wrapper })

    act(() => {
      result.current.start()
    })

    act(() => {
      result.current.next()
    })

    act(() => {
      result.current.prev()
    })

    expect(result.current.currentStepIndex).toBe(0)
  })

  it('completes tour on last step next', () => {
    const { result } = renderHook(() => useTour(), { wrapper })

    act(() => {
      result.current.start()
    })

    // Go to last step
    act(() => {
      result.current.goTo(2)
    })

    act(() => {
      result.current.next()
    })

    expect(result.current.isActive).toBe(false)
  })

  it('skips tour', () => {
    const { result } = renderHook(() => useTour(), { wrapper })

    act(() => {
      result.current.start()
    })

    act(() => {
      result.current.skip()
    })

    expect(result.current.isActive).toBe(false)
  })

  it('calculates progress correctly', () => {
    const { result } = renderHook(() => useTour(), { wrapper })

    act(() => {
      result.current.start()
    })

    expect(result.current.progress).toBeCloseTo(1 / 3)
    expect(result.current.isFirstStep).toBe(true)
    expect(result.current.isLastStep).toBe(false)

    act(() => {
      result.current.goTo(2)
    })

    expect(result.current.progress).toBe(1)
    expect(result.current.isLastStep).toBe(true)
  })
})
```

### Step 7: Add test scripts to packages

Update `packages/core/package.json` scripts:

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  }
}
```

### Step 8: Create test command in root

The root `package.json` already has `"test": "turbo test"`. Ensure each package has matching test scripts.

## Key Requirements & Constraints

- [ ] Test coverage > 80%
- [ ] All hooks tested with React Testing Library
- [ ] Utilities have unit tests
- [ ] Mocks are properly configured
- [ ] Tests run in CI

## Verification

```bash
# Run all tests
pnpm test

# Run with coverage
pnpm test:coverage
```

## Next Prompt

Continue with documentation site setup.

