<!-- refreshed: 2026-06-29 -->

# Architecture

**Analysis Date:** 2026-06-29

## System Overview

```text
┌─────────────────────────────────────────────────────────────┐
│              VS Code Extension Host                          │
│  `src/extension.ts` - Timer State & Command Handler          │
│                                                               │
│  State: remainingSeconds, currentSessionType, isPaused      │
│  Timer: globalTimerInterval (setInterval loop)              │
│  UI: updateStatusBar(), updateUI()                          │
└─────────────────────┬───────────────────────────────────────┘
         │
         │ Commands & State Updates
         ▼
┌─────────────────────────────────────────────────────────────┐
│        Webview Provider (Presentation Controller)            │
│  `src/pomodoroWebviewProvider.ts`                            │
│                                                               │
│  - Implements vscode.WebviewViewProvider                     │
│  - Renders HTML + CSS + JS to webview                        │
│  - Receives postMessage() from webview                       │
│  - Routes button clicks to extension commands               │
└─────────────────────┬───────────────────────────────────────┘
         │
         │ postMessage() → updateTimer event
         ▼
┌─────────────────────────────────────────────────────────────┐
│              Webview (VS Code Sidebar Panel)                 │
│  `media/main.js` + `media/styles.css` + HTML                │
│                                                               │
│  - Message listener: receives timer updates                  │
│  - Button handlers: Start, Pause, Reset, Switch             │
│  - DOM updates: timer display, session type, button state    │
└─────────────────────────────────────────────────────────────┘
```

## Component Responsibilities

| Component        | Responsibility                                                                   | File                                       |
| ---------------- | -------------------------------------------------------------------------------- | ------------------------------------------ |
| Extension Host   | Timer state management, interval control, status bar rendering, command dispatch | `src/extension.ts`                         |
| Webview Provider | HTML/CSS/JS assembly, message routing, state translation to webview format       | `src/pomodoroWebviewProvider.ts`           |
| Webview          | User input capture, timer display rendering, button state toggling               | `media/main.js`, `media/styles.css`        |
| Status Bar       | Quick-access timer display and play/pause toggle                                 | Generated in `src/extension.ts` line 46-77 |

## Pattern Overview

**Overall:** Unidirectional command-based messaging with centralized state in the extension host.

**Key Characteristics:**

- Extension host owns all timer state — no state duplication in webview
- Communication via VS Code `postMessage()` and `executeCommand()` APIs
- Webview is purely presentational — stateless renderer
- Status bar provides quick toggle without opening webview
- Timer managed by global `setInterval()` on main thread

## Layers

**Extension Host (Control/Logic):**

- Purpose: Timer state machine, interval management, VS Code API integration
- Location: `src/extension.ts`
- Contains: Timer functions, command handlers, status bar management, configuration loading
- Depends on: VS Code API (`vscode`), PomodoroWebviewProvider
- Used by: VS Code runtime, commands palette, status bar clicks

**Webview Provider (Translator/Router):**

- Purpose: Convert extension state to webview messages, route webview events to commands
- Location: `src/pomodoroWebviewProvider.ts`
- Contains: WebviewViewProvider implementation, HTML/CSS/JS assembly, message handlers
- Depends on: VS Code API (`vscode`), extension host for command execution
- Used by: VS Code webview system, extension host

**Webview UI (Presentation):**

- Purpose: Render timer display and controls, capture user input
- Location: `media/main.js`, `media/styles.css`, HTML in `pomodoroWebviewProvider.ts` lines 97-130
- Contains: Event listeners, DOM manipulation, icon buttons
- Depends on: VS Code Webview API (`acquireVsCodeApi()`)
- Used by: User interaction

## Data Flow

### Primary Request Path: Timer Tick

1. **Interval fires** (`src/extension.ts` line 111: `setInterval()`)
   - `remainingSeconds--`
   - Check if session complete (< 0)

2. **If session active:**
   - Call `updateUI()` (line 140)
   - `updateStatusBar()` formats time, updates status bar text
   - `provider.updateTimer()` converts state to webview format

3. **Webview receives message:**
   - `media/main.js` line 1-29: message listener
   - Updates `#timer-display` text (minutes:seconds)
   - Shows/hides start/pause buttons based on `isActive`
   - Displays session type (Work/Break)

### User Action Flow: Click Start Button

1. **Webview button click** (`media/main.js` line 34-35)
   - `vscode.postMessage({ type: "startTimer" })`

2. **Provider receives message** (`src/pomodoroWebviewProvider.ts` line 44-59)
   - `case "startTimer"` matches
   - `vscode.commands.executeCommand("paulie-pomodoro.startTimer")`

3. **Extension handler executes** (`src/extension.ts` line 91-142)
   - Sets `isPaused = false`
   - Initializes `remainingSeconds` if at 0
   - Clears existing interval if running
   - Starts new `setInterval()` loop
   - Calls `updateUI()` to sync status bar and webview

4. **Status bar updates immediately** (same `updateUI()` call)
   - Emoji changes from play (▶️) to pause (⏸️)
   - Ready for quick toggle

### Session Complete Flow

1. **Timer reaches 0** (`src/extension.ts` line 113)
   - Interval clears
   - Check `enableNotifications` setting
   - Switch `currentSessionType` Work ↔ Break
   - Set `remainingSeconds = 0`

2. **Check `autoSwitchSessions` setting** (line 132)
   - If true: call `startTimer()` again (automatic next session)
   - If false: set `isPaused = true`, call `updateUI()` (manual start)

3. **UI updates reflect new session:**
   - Status bar shows new emoji (😴 for break, 👷 for work)
   - Webview shows new session type
   - Timer display updates to new duration

**State Management:**

- All state lives in `extension.ts`: `remainingSeconds`, `currentSessionType`, `isPaused`, `globalTimerInterval`
- Webview receives computed values only: minutes, seconds, sessionType lowercase, isActive boolean
- No reverse state flow — webview never updates extension state directly
- Configuration read fresh from workspace on each action (not cached)

## Key Abstractions

**SessionType Enum:**

- Purpose: Strongly type Work vs Break sessions
- File: `src/extension.ts` line 6-9
- Values: `Work`, `Break`
- Used to determine session duration, emoji, webview display

**updateUI() Function:**

- Purpose: Synchronization point — ensures status bar and webview stay in sync
- Called after every state change (start, pause, reset, switch, tick)
- Invokes both `updateStatusBar()` and `provider.updateTimer()`

**Timer Conversion:**

- Extension stores `remainingSeconds` (integer)
- Webview displays `minutes:seconds` format
- Conversion: `Math.floor(totalSeconds / 60)` and `totalSeconds % 60`
- Applied in both `formatTime()` (status bar) and webview provider (line 75-76)

## Entry Points

**Extension Activation:**

- Location: `src/extension.ts` line 11-203
- Triggers: VS Code startup (specified in `package.json` activationEvents: `onStartupFinished`)
- Responsibilities:
  - Register webview provider
  - Initialize state variables
  - Register all commands
  - Create and show status bar
  - Establish message handlers

**Webview Resolution:**

- Location: `src/pomodoroWebviewProvider.ts` line 27-68
- Triggers: User opens Paulie Pomodoro sidebar view
- Responsibilities:
  - Store reference to WebviewView
  - Generate HTML with styles and scripts
  - Attach message listener for button clicks
  - Request initial UI update from extension

**Command Handlers:**

- `paulie-pomodoro.startTimer`: line 91-142
- `paulie-pomodoro.pauseTimer`: line 145-151
- `paulie-pomodoro.resetTimer`: line 154-163
- `paulie-pomodoro.switchSessionType`: line 166-174
- `paulie-pomodoro.statusBarClick`: line 177-185
- `paulie-pomodoro.updateUI`: line 200

## Architectural Constraints

- **Threading:** Single-threaded JavaScript event loop. Timer uses `setInterval()` on main thread — long-running computations could pause timer.
- **Global state:** Extension host stores all state in function scope variables (lines 26-28). No module-level state except `globalTimerInterval` (line 206).
- **Circular imports:** None — provider depends on extension, extension depends on provider, but no circular require/import.
- **Webview isolation:** Webview has no direct access to extension state. All communication via message passing (`postMessage()`). CSP (Content Security Policy) applied in line 101.
- **Configuration access:** Configuration read at action time (not cached). Allows settings changes without extension restart, but incurs lookup cost on each timer tick.
- **Status bar persistence:** Status bar item created once (line 46-54) and persisted. Updated on every tick or state change.

## Anti-Patterns

### No State Duplication (Correct)

**What happens:** Extension owns all timer state. Webview receives computed display values only.

**Why it's right:** Single source of truth prevents desync bugs. Webview can be destroyed/recreated without losing timer state.

### Command-Based Communication (Correct)

**What happens:** Webview button clicks route through `vscode.commands.executeCommand()` rather than direct function calls.

**Why it's right:** Decouples webview from extension implementation. Commands work from palette, keybindings, or other sources.

### No Promise-Based Timer (Correct)

**What happens:** Uses `setInterval()` with manual state tracking rather than async/await generator.

**Why it's right:** Simpler control flow for pause/resume. Avoids promise state management complexity.

### Configuration Read on Each Tick (Potential Issue)

**What happens:** `vscode.workspace.getConfiguration()` called in `startTimer()` and timer loop.

**Why it could be improved:** Performance cost on every tick. Better to cache config at session start and only update when settings change event fires.

**Do this instead:** Implement `vscode.workspace.onDidChangeConfiguration()` listener that caches workDuration and breakDuration. Update cache only when setting changes.

## Error Handling

**Strategy:** Minimal explicit error handling. Relies on VS Code's built-in safeguards.

**Patterns:**

- No try/catch blocks in timer code — setInterval() errors would exit loop
- Configuration reads include defaults: `.get<number>("workingSessionLength", 24)` — falls back to 24 if missing
- Webview message handler switches on message type — unknown types silently ignored
- Interval clear is idempotent: `clearInterval()` on undefined interval is safe

## Cross-Cutting Concerns

**Logging:** `console.log()` on activation (line 12-14). No logging in timer loop or message handlers. Consider adding debug logging for timer ticks and state changes.

**Validation:** No explicit input validation. Configuration assumed to be numeric (relies on type guards). Webview messages assumed valid (switch statement fails gracefully for unknown types).

**Authentication:** Not applicable (no external services).

---

_Architecture analysis: 2026-06-29_
