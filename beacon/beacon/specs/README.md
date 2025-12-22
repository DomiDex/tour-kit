# TourKit Technical Specifications

## Overview

This directory contains detailed technical specifications for all TourKit packages.

## Specification Files

| File            | Description                      |
| --------------- | -------------------------------- |
| `core-types.md` | All TypeScript type definitions  |
| `hooks.md`      | Hook implementations and APIs    |
| `components.md` | React component specifications   |
| `utilities.md`  | Utility function implementations |

## Type System

### Core Types Hierarchy

```
Types
├── Positioning
│   ├── Side
│   ├── Alignment
│   ├── Placement
│   ├── Position
│   └── Rect
├── Configuration
│   ├── KeyboardConfig
│   ├── SpotlightConfig
│   ├── PersistenceConfig
│   ├── A11yConfig
│   └── ScrollConfig
├── Tour
│   ├── Tour
│   ├── TourStep
│   └── TourOptions
├── State
│   ├── TourState
│   ├── TourContext
│   └── TourActions
└── Hints
    ├── HintConfig
    ├── HintState
    └── BeaconPosition
```

## Hooks Reference

### Core Hooks

| Hook                    | Package | Description               |
| ----------------------- | ------- | ------------------------- |
| `useTour`               | core    | Main tour control hook    |
| `useStep`               | core    | Individual step state     |
| `useSpotlight`          | core    | Spotlight overlay control |
| `useKeyboardNavigation` | core    | Keyboard event handling   |
| `useFocusTrap`          | core    | Focus trapping            |
| `usePersistence`        | core    | State persistence         |
| `useElementPosition`    | core    | Element rect tracking     |
| `useMediaQuery`         | core    | Media query matching      |

### Hints Hooks

| Hook       | Package | Description         |
| ---------- | ------- | ------------------- |
| `useHints` | hints   | Manage all hints    |
| `useHint`  | hints   | Single hint control |

## Components Reference

### @tour-kit/react

| Component        | Description                   |
| ---------------- | ----------------------------- |
| `Tour`           | Main tour wrapper             |
| `TourStep`       | Step definition (declarative) |
| `TourCard`       | Floating tooltip card         |
| `TourOverlay`    | Spotlight overlay             |
| `TourNavigation` | Prev/Next buttons             |
| `TourProgress`   | Progress indicator            |
| `TourClose`      | Close button                  |
| `TourArrow`      | Tooltip arrow                 |
| `TourPortal`     | Portal wrapper                |

### @tour-kit/hints

| Component     | Description           |
| ------------- | --------------------- |
| `Hint`        | Main hint wrapper     |
| `HintBeacon`  | Pulsing dot indicator |
| `HintTooltip` | Hint content tooltip  |

## Utility Functions

### DOM Utilities

- `getElement` - Resolve target to element
- `isElementVisible` - Check viewport visibility
- `waitForElement` - Wait for element existence
- `getFocusableElements` - Get focusable children
- `getScrollParent` - Find scroll container

### Position Utilities

- `getElementRect` - Get element position
- `parsePlacement` - Parse placement string
- `calculatePosition` - Calculate tooltip position
- `wouldOverflow` - Check overflow boundaries

### Scroll Utilities

- `scrollIntoView` - Scroll element into view
- `scrollTo` - Scroll to position
- `lockScroll` - Lock body scroll

### Storage Utilities

- `createStorageAdapter` - Create storage from config
- `createPrefixedStorage` - Add key prefix
- `safeJSONParse` - Safe JSON parsing

### Accessibility Utilities

- `announce` - Screen reader announcement
- `generateId` - Generate unique IDs
- `prefersReducedMotion` - Check motion preference

## Size Budgets

| Package         | Budget | Description    |
| --------------- | ------ | -------------- |
| @tour-kit/core  | < 8KB  | Headless logic |
| @tour-kit/react | < 12KB | Including core |
| @tour-kit/hints | < 5KB  | Hints only     |

## Usage

Reference these specs when:

- Implementing new features
- Debugging issues
- Writing documentation
- Creating tests
