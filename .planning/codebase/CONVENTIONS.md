# Coding Conventions

**Analysis Date:** 2026-06-29

## Naming Patterns

**Files:**

- Class implementations: PascalCase (e.g., `pomodoroWebviewProvider.ts`)
- Entry points and utilities: lowercase (e.g., `extension.ts`)

**Functions:**

- camelCase for all functions (e.g., `updateStatusBar()`, `formatTime()`, `startTimer()`)
- Named functions inside activation scope follow same convention
- Handler functions: camelCase with "Handler" suffix (e.g., `statusBarClickHandler()`)

**Variables:**

- camelCase for all variables (e.g., `remainingSeconds`, `currentSessionType`, `isPaused`)
- Private class fields: prefixed with underscore + camelCase (e.g., `_view`, `_extensionUri`)

**Types:**

- PascalCase for enums and types (e.g., `SessionType`)
- PascalCase for classes (e.g., `PomodoroWebviewProvider`)

**Constants:**

- camelCase with semantic meaning (e.g., `viewType = "paulie-pomodoro-webview"`)
- Static readonly class properties: camelCase (e.g., `viewType`)

## Code Style

**Formatting:**

- No dedicated formatter (Prettier not configured)
- 2-space indentation (observed in code)
- Double quotes for strings (both JS and JSDoc)
- Single quotes in one test file but double quotes primary

**Linting:**

- Tool: eslint with flat config (`eslint.config.mjs`)
- Plugin: @typescript-eslint/eslint-plugin and @typescript-eslint/parser
- Target: TypeScript files only (`files: ["**/*.ts"]`)

**Linting Rules:**

- `@typescript-eslint/naming-convention` (warn): imports must be camelCase or PascalCase
- `curly` (warn): Enforce curly braces
- `eqeqeq` (warn): Enforce strict equality (=== instead of ==)
- `no-throw-literal` (warn): Don't throw plain strings/objects
- `semi` (warn): Enforce semicolons

## Import Organization

**Order:**

1. External packages: `import * as vscode from "vscode"`
2. Internal modules: `import { PomodoroWebviewProvider } from "./pomodoroWebviewProvider"`
3. Node.js built-ins: `import * as assert from 'assert'`

**Path Aliases:**

- Relative paths used (no alias configuration)
- Namespace imports preferred for large modules (`import * as vscode`)
- Named imports for specific exports (`import { PomodoroWebviewProvider }`)

**Quote Style:**

- Double quotes primary in source files
- Single quotes observed in test files (inconsistency tolerated)

## Error Handling

**Patterns:**

- No custom error classes defined
- Using console.log for extension diagnostics (e.g., activation message)
- VS Code API for user-facing errors: `vscode.window.showInformationMessage()`
- Conditional notifications based on configuration
- Global state cleared on error/deactivation (clearInterval pattern)

**Guidelines:**

- Validate configuration before use with `.get<Type>(key, defaultValue)` pattern
- Always clear timers on deactivation (see `deactivate()`)
- Use strict equality in condition checks (eqeqeq rule enforced)

## Logging

**Framework:** console object

**Patterns:**

- Initial activation logging: `console.log()` on extension startup
- No structured logging or debug levels
- User notifications via `vscode.window.showInformationMessage()`
- Status bar used as primary UI display (real-time state)

## Comments

**When to Comment:**

- Explain non-obvious logic (e.g., time formatting with different units)
- Document configuration fallbacks and defaults
- Clarify intentional parameter ordering (e.g., "// Changed to Left alignment")
- Mark sections for clarity (e.g., "// Initial state or after reset/session switch")

**JSDoc/TSDoc:**

- Not used in this codebase
- Type hints via TypeScript instead (all parameters typed)

## Function Design

**Size:** Functions are small to medium (5-25 lines)

**Parameters:**

- Always typed (strict mode enforced in tsconfig)
- Command handlers take required vscode extension context as parameters
- Timer functions capture state from closure (activate function scope)

**Return Values:**

- Typed explicitly (e.g., `: string`, `: void`)
- No implicit returns (strict TypeScript)
- Promise returns used where needed for async operations

**Example - Function Definition Pattern:**

```typescript
const formatTime = (totalSeconds: number): string => {
  const minutes = Math.floor(totalSeconds / 60);
  const seconds = totalSeconds % 60;

  if (totalSeconds >= 60) {
    return `${minutes}m`;
  } else {
    return `${seconds}s`;
  }
};
```

## Module Design

**Exports:**

- `activate()` and `deactivate()` functions as required by VS Code extension API
- Class exports for providers: `export class PomodoroWebviewProvider`
- Named exports only (no default exports)

**Barrel Files:**

- Not used (simple module structure)

## TypeScript Configuration

**Strict Mode:**

- Enabled: `strict: true` in `tsconfig.json`
- Target: ES2022
- Module: Node16
- Source maps enabled for debugging

**Compilation:**

- Root directory: `src/`
- Output: Default (dist or out depending on context)
- Watch mode available: `npm run watch:tsc`

---

_Convention analysis: 2026-06-29_
