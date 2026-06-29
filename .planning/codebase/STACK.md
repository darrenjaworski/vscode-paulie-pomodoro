# Technology Stack

**Analysis Date:** 2026-06-29

## Languages

**Primary:**

- TypeScript 5.8.3 - All source code in `src/`

**Secondary:**

- JavaScript (Node.js) - Build scripts and test configuration

## Runtime

**Environment:**

- Node.js 24 (specified in `.nvmrc`)

**Package Manager:**

- npm - Lockfile: `package-lock.json` present

## Frameworks

**Core:**

- VS Code Extension API 1.100.0 - Extension host API in `src/extension.ts`, webview API in `src/pomodoroWebviewProvider.ts`

**Testing:**

- Mocha 10.0.10 (@types/mocha) - Test runner via @vscode/test-cli
- @vscode/test-cli 0.0.10 - Official VS Code test CLI runner
- @vscode/test-electron 2.5.2 - VS Code extension testing infrastructure

**Build/Dev:**

- esbuild 0.25.3 - Bundles extension via `esbuild.js` (CJS format, vscode externalized)
- TypeScript 5.8.3 - Language compiler (ES2022 target)
- ESLint 9.25.1 - Static code analysis via `eslint.config.mjs`
- @typescript-eslint/parser 8.31.1 - TypeScript parsing for ESLint
- @typescript-eslint/eslint-plugin 8.31.1 - TypeScript ESLint rules
- npm-run-all 4.1.5 - Task orchestration (parallel watch mode)

## Key Dependencies

**Critical:**

- @vscode/vsce (npm script dependency) - Package VS Code extensions into .vsix files

**Infrastructure:**

- @types/vscode 1.100.0 - TypeScript type definitions for VS Code API
- @types/node 24.x - Node.js type definitions
- @types/mocha 10.0.10 - Mocha test framework types

## Configuration

**Environment:**

- Configuration via VS Code settings (`paulie-pomodoro.*` namespace)
  - `paulie-pomodoro.enableNotifications` (boolean, default: false)
  - `paulie-pomodoro.workingSessionLength` (number minutes, default: 24)
  - `paulie-pomodoro.breakSessionLength` (number minutes, default: 6)
  - `paulie-pomodoro.autoSwitchSessions` (boolean, default: false)

**Build:**

- TypeScript: `tsconfig.json` - Module: Node16, Target: ES2022, Strict: true, sourceMap: true
- ESLint: `eslint.config.mjs` - ESM config with TypeScript support
- esbuild: `esbuild.js` - Entry: `src/extension.ts` → Output: `dist/extension.js`

## Platform Requirements

**Development:**

- Node.js 24+
- VS Code 1.100.0+

**Production:**

- VS Code Extension host (Linux, macOS, Windows)
- Published via Visual Studio Code Marketplace

---

_Stack analysis: 2026-06-29_
