# Prompt 02: Configure Tooling

## Problem

Set up shared TypeScript configurations and Biome linting/formatting for the monorepo.

## Dependencies on Previous Prompts

- Prompt 01 complete (monorepo structure exists)

## Supporting Information

### Target File Structure

```
tour-kit/
├── tooling/
│   ├── tsconfig/
│   │   ├── base.json
│   │   ├── react-library.json
│   │   └── nextjs.json
│   └── biome/
│       └── biome.json
├── biome.json
└── tsconfig.json
```

## Steps To Complete

### Step 1: Create base TypeScript config

Create `tooling/tsconfig/base.json`:

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "allowSyntheticDefaultImports": true,
    "verbatimModuleSyntax": true
  },
  "exclude": ["node_modules", "dist"]
}
```

### Step 2: Create React library TypeScript config

Create `tooling/tsconfig/react-library.json`:

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "./base.json",
  "compilerOptions": {
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "declaration": true,
    "declarationMap": true,
    "noEmit": false
  }
}
```

### Step 3: Create Next.js TypeScript config

Create `tooling/tsconfig/nextjs.json`:

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "./base.json",
  "compilerOptions": {
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "preserve",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowJs": true,
    "noEmit": true,
    "incremental": true,
    "plugins": [{ "name": "next" }]
  }
}
```

### Step 4: Create shared Biome config

Create `tooling/biome/biome.json`:

```json
{
  "$schema": "https://biomejs.dev/schemas/1.8.0/schema.json",
  "organizeImports": {
    "enabled": true
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "complexity": {
        "noExcessiveCognitiveComplexity": "warn"
      },
      "correctness": {
        "noUnusedImports": "error",
        "noUnusedVariables": "error",
        "useExhaustiveDependencies": "warn"
      },
      "style": {
        "noNonNullAssertion": "warn",
        "useConst": "error",
        "useImportType": "error"
      },
      "suspicious": {
        "noExplicitAny": "warn"
      },
      "a11y": {
        "recommended": true
      }
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "trailingCommas": "es5",
      "semicolons": "asNeeded"
    }
  },
  "json": {
    "formatter": {
      "trailingCommas": "none"
    }
  }
}
```

### Step 5: Create root Biome config

Create `biome.json` (root):

```json
{
  "$schema": "https://biomejs.dev/schemas/1.8.0/schema.json",
  "extends": ["./tooling/biome/biome.json"],
  "files": {
    "ignore": [
      "**/dist/**",
      "**/node_modules/**",
      "**/.next/**",
      "**/.turbo/**",
      "**/coverage/**"
    ]
  }
}
```

### Step 6: Create root TypeScript config

Create `tsconfig.json` (root):

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "./tooling/tsconfig/base.json",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@tour-kit/core": ["packages/core/src"],
      "@tour-kit/react": ["packages/react/src"],
      "@tour-kit/hints": ["packages/hints/src"]
    }
  },
  "include": [],
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/react" },
    { "path": "./packages/hints" }
  ]
}
```

### Step 7: Create tooling package.json files

Create `tooling/tsconfig/package.json`:

```json
{
  "name": "@tour-kit/tsconfig",
  "private": true,
  "version": "0.0.0",
  "files": ["*.json"]
}
```

Create `tooling/biome/package.json`:

```json
{
  "name": "@tour-kit/biome-config",
  "private": true,
  "version": "0.0.0",
  "files": ["biome.json"]
}
```

## Key Requirements & Constraints

- [ ] TypeScript strict mode enabled
- [ ] React JSX runtime used (not classic)
- [ ] Biome handles both linting and formatting
- [ ] All configs use proper $schema references
- [ ] Path aliases configured for internal packages
- [ ] Configs are properly composable/extendable

## Verification

```bash
# Check Biome config
pnpm biome check --max-diagnostics=0

# Should work without errors
pnpm typecheck
```

## Next Prompt

Continue with `03-ci-cd-setup.md` to configure GitHub Actions.
