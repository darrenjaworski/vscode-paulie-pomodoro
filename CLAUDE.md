# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A VS Code extension: a Pomodoro timer ("Paulie Pomodoro") that lives in the status bar and an Activity Bar webview panel. Published to the VS Code Marketplace as `darrenjaworski.paulie-pomodoro`.

## Commands

Node 24 LTS is required (`.nvmrc`, `engines.node: ">=24"`). Run `nvm use` first.

- `npm run compile` — full local build: `check-types` → `lint` → esbuild bundle. Use this to verify a change.
- `npm run watch` — parallel esbuild + `tsc --noEmit` watchers for active development (launch the extension with F5 / `.vscode/launch.json`).
- `npm run check-types` — `tsc --noEmit` only (esbuild does NOT type-check; this is the only type gate).
- `npm run lint` — `eslint src`.
- `npm run package` — production bundle (minified, no sourcemap); run by `vscode:prepublish`.
- `npm test` — runs `vscode-test`, which downloads a real VS Code/Electron instance and executes `out/test/**/*.test.js`. `pretest` compiles tests + the extension + lints first. This is slow and needs a display — avoid running it casually; prefer `npm run check-types` + `npm run lint` for fast feedback.
- Single test: tests run under Mocha via `@vscode/test-cli`; filter with `npx vscode-test --grep "<pattern>"` (after `npm run pretest`).
- `npm run package-vsix` — produces the `.vsix` release artifact via `@vscode/vsce`.

## Architecture

The extension host owns all state; the webview is purely presentational. Understanding the data flow across these files is the key to working here:

- **`src/extension.ts`** — the single source of truth. Timer state (`remainingSeconds`, `currentSessionType`, `isPaused`) lives in closure variables inside `activate()`, plus a module-level `globalTimerInterval` (the `setInterval` handle, cleared in `deactivate()`). The countdown, session-end logic, auto-switch, and notifications all live here. `updateUI()` is the central sync point — it pushes state to **both** render targets (status bar and webview) on every tick.

- **`src/pomodoroWebviewProvider.ts`** — `WebviewViewProvider` that renders the panel and bridges messages. Its `viewType` string must match the view `id` in `package.json`. It builds the webview HTML inline (with a CSP + per-load nonce; loads Font Awesome from cdnjs). It holds NO timer state — `updateTimer()` just formats incoming state and `postMessage`s it to the webview.

- **`media/main.js` + `media/styles.css`** — the webview front end. `main.js` receives `updateTimer` messages and renders; button clicks `postMessage` back to the provider.

**Message flow (both directions go through commands):**

- Webview → extension: button click → `postMessage` → provider's `onDidReceiveMessage` → `executeCommand("paulie-pomodoro.startTimer" | pauseTimer | resetTimer | switchSessionType)`.
- Extension → webview: state change → `updateUI()` → `provider.updateTimer()` → `postMessage({type: "updateTimer", ...})` → `main.js`.

**Commands:** four user-facing commands are declared in `package.json` `contributes.commands`. Two more are registered in code but intentionally NOT in the palette: `paulie-pomodoro.statusBarClick` (toggles start/pause and reveals the panel) and `paulie-pomodoro.updateUI` (internal glue the provider calls to request initial state on view resolve).

**Settings** (`paulie-pomodoro.*`: `workingSessionLength` 24, `breakSessionLength` 6, `enableNotifications`, `autoSwitchSessions`) are read fresh from `vscode.workspace.getConfiguration` at each use rather than cached — defaults in code must stay in sync with `package.json` `contributes.configuration`.

## Build specifics

- esbuild (`esbuild.js`) bundles `src/extension.ts` → `dist/extension.js` (CJS, `vscode` externalized). It is bundle-only; type errors surface only through `tsc --noEmit`.
- Tests use a separate `tsc -p . --outDir out` compile (`compile-tests`) — `out/` is the test build, `dist/` is the shipped bundle.

## Release process

When cutting a release, **`CHANGELOG.md` and the README's "Release Notes" section must be updated together** — they duplicate the same per-version notes and drift easily (this is a common miss). Typical flow:

1. `npm version patch --no-git-tag-version` (bumps `package.json` + lockfile).
2. Add the matching entry to both `CHANGELOG.md` and `README.md`.
3. Commit as `chore: release x.y.z` (repo uses Conventional Commits).
4. `npm run package-vsix`, then tag `vX.Y.Z` and create a GitHub release with the `.vsix` attached.

Note: the git `origin` still points at the old `paulie-pomodoro` repo name; GitHub redirects to `vscode-paulie-pomodoro`.
