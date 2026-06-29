# Testing Patterns

**Analysis Date:** 2026-06-29

## Test Framework

**Runner:**

- @vscode/test-cli (0.0.10) - Orchestrates test execution
- @vscode/test-electron (2.5.2) - Runs tests in real VS Code/Electron instance
- Mocha - Built-in test runner (via test-cli)

**Assertion Library:**

- Node.js built-in `assert` module (CommonJS import in tests)
- Methods: `strictEqual(actual, expected)`

**Config:**

- `.vscode-test.mjs` - Defines test entry points and configuration
- Uses `defineConfig()` from @vscode/test-cli

**Run Commands:**

```bash
npm test              # Run all tests (runs compile-tests, compile, and lint first)
npm run compile-tests # Compile TypeScript test files to out/test/
npm run watch-tests   # Watch mode - recompile tests on changes
npm run pretest       # Runs: compile-tests, compile, lint (pre-test hook)
```

## Test File Organization

**Location:**

- Source: `src/test/` directory
- Compiled output: `out/test/` directory
- Glob pattern: `out/test/**/*.test.js` (compiled JavaScript)

**Naming:**

- Pattern: `*.test.ts` extension
- Single test file currently: `extension.test.ts`
- Compiled to: `out/extension.test.js`

**Structure:**

```
src/test/
└── extension.test.ts
```

## Test Structure

**Suite Organization:**

```typescript
import * as assert from "assert";
import * as vscode from "vscode";

suite("Extension Test Suite", () => {
  vscode.window.showInformationMessage("Start all tests.");

  test("Sample test", () => {
    assert.strictEqual(-1, [1, 2, 3].indexOf(5));
    assert.strictEqual(-1, [1, 2, 3].indexOf(0));
  });
});
```

**Patterns:**

- `suite()` wraps test cases (Mocha convention)
- Initial `vscode.window.showInformationMessage()` runs when suite loads (not inside test)
- Individual test cases with `test(description, callback)`
- Assertions inside test callbacks

## Mocking

**Framework:** None configured

**What to Mock:**

- VS Code API calls (e.g., `vscode.window.*`, `vscode.commands.*`)
- Timer functions (`setInterval`, `clearInterval`) for deterministic tests
- Configuration lookups (`.getConfiguration()`)

**What NOT to Mock:**

- Internal timer state variables (test these through public API)
- Command registration flow

**Example Pattern (from sample test):**

```typescript
test("Sample test", () => {
  assert.strictEqual(-1, [1, 2, 3].indexOf(5));
  assert.strictEqual(-1, [1, 2, 3].indexOf(0));
});
```

## Fixtures and Factories

**Test Data:**

- Currently: None defined
- Tests use inline literals (arrays, numbers)

**Location:**

- Would go in `src/test/` directory alongside test files
- Consider: `src/test/fixtures/` subdirectory for complex data

## Coverage

**Requirements:** None enforced

**View Coverage:**

- No coverage command configured in package.json
- To enable: would require Istanbul/nyc integration with vscode-test

## Test Types

**Unit Tests:**

- Scope: Individual functions and API calls
- Approach: Direct assertion on return values or side effects
- Current example: `indexOf()` tests (sample test)
- Location: `src/test/extension.test.ts`

**Integration Tests:**

- Scope: Extension activation, command registration, state management
- Currently: Minimal coverage
- Recommended: Test timer state changes, session switching, configuration reading
- Would use: VS Code Extension API directly (vscode.commands.executeCommand, etc.)

**E2E Tests:**

- Framework: Not used
- The vscode-test runner IS the E2E environment (runs in real VS Code)
- Could add: Webview interaction tests by posting messages through `webview.postMessage()`

## Common Patterns

**Async Testing:**

```typescript
test("async operation", async () => {
  const result = await vscode.commands.executeCommand(
    "paulie-pomodoro.startTimer",
  );
  assert.strictEqual(result, expectedValue);
});
```

**Mocha Hooks:**

```typescript
suite("Extension Test Suite", () => {
  suiteSetup(() => {
    // Runs once before all tests
  });

  setup(() => {
    // Runs before each test
  });

  teardown(() => {
    // Runs after each test
  });

  suiteTeardown(() => {
    // Runs once after all tests
  });
});
```

**Error Testing Pattern:**

```typescript
test("error case", () => {
  assert.throws(() => {
    // Code that should throw
  }, Error);
});
```

## Test Execution Flow

1. **Pretest Hook:**
   - Compiles test files: `tsc -p . --outDir out`
   - Compiles source: `node esbuild.js`
   - Runs linting: `eslint src`

2. **Test Run:**
   - vscode-test CLI loads `.vscode-test.mjs`
   - Launches VS Code instance with extension
   - Loads compiled tests from `out/test/**/*.test.js`
   - Mocha executes suites and tests

3. **Extension Context:**
   - Real VS Code/Electron environment
   - Full VS Code API available
   - Can test UI interactions, commands, configuration
   - Timer system uses real `setInterval` (not mocked)

## Writing New Tests

**Location:** `src/test/` directory

**File Pattern:** `{feature}.test.ts`

**Template:**

```typescript
import * as assert from "assert";
import * as vscode from "vscode";

suite("[Feature Name] Tests", () => {
  test("should [expected behavior]", () => {
    // Arrange
    // Act
    // Assert
  });
});
```

**Compilation:** Automatic with `npm test` (via pretest hook)

---

_Testing analysis: 2026-06-29_
