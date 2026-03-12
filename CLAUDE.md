# ClaudeGlance Agent Guide

## Scope

This repository is a native macOS menu bar app that shows a floating HUD for multiple Claude Code sessions. It is written in Swift with SwiftUI and AppKit, and uses shell hook scripts to forward Claude Code events into the app.

Use this file as the execution guide when making changes with Claude Code or another coding agent.

## Project Facts

- App entry point: `ClaudeGlance/ClaudeGlanceApp.swift`
- Core runtime services:
  - `ClaudeGlance/Services/SessionManager.swift`
  - `ClaudeGlance/Services/IPCServer.swift`
- Hook payload models: `ClaudeGlance/Models/SessionState.swift`
- Main HUD views live under `ClaudeGlance/Views/`
- Shell scripts exist in two places:
  - root `Scripts/`
  - app bundle resources `ClaudeGlance/Scripts/`

## How The App Works

1. The app launches from the menu bar and creates the HUD window from `AppDelegate`.
2. On startup it auto-installs or repairs the Claude hook script in `~/.claude/hooks/claude-glance-reporter.sh`.
3. On startup it also verifies `~/.claude/settings.json` and inserts missing hook entries.
4. Claude hook events are sent through:
   - Unix socket: `/tmp/claude-glance.sock`
   - HTTP fallback: `http://localhost:19847/api/status`
5. `IPCServer` receives payloads and forwards them to `SessionManager`.
6. `SessionManager` maintains session state, statistics, timeouts, and filtering for speculative or stale events.

## Working Rules

- Preserve existing menu bar and floating HUD behavior unless the task explicitly changes UX.
- Do not break the hook auto-install and auto-repair path in `ClaudeGlanceApp.swift`.
- Treat the Unix socket as the primary transport and HTTP as compatibility fallback.
- Keep status transitions aligned with `SessionManager` logic. This file contains non-trivial behavior around:
  - silent period after `Stop`
  - delayed display for new sessions
  - timeout-based cleanup
  - unique-session statistics
- When changing hook payload shape, update both the shell script and Swift decoding models together.
- When changing reporter/install/uninstall/build scripts, inspect both `Scripts/` and `ClaudeGlance/Scripts/` and keep intent aligned. The bundled copy is what the app installs at runtime.
- Do not assume there are tests. Verify with targeted builds and manual reasoning.

## Preferred Commands

Build:

```bash
xcodebuild -scheme ClaudeGlance -configuration Release
```

Package DMG from repo root:

```bash
./Scripts/build-dmg.sh
```

Search code:

```bash
rg "symbol_or_text" ClaudeGlance Scripts
```

## Change Guidance

### UI changes

- Prefer small, intentional edits over large rewrites.
- Keep the existing macOS-native style.
- Check both menu bar state and HUD behavior when changing session presentation.

### Hook or IPC changes

- Validate end-to-end flow: shell script -> socket/HTTP -> decode -> `SessionManager`.
- Preserve compatibility with existing Claude hook names:
  - `PreToolUse`
  - `PostToolUse`
  - `Notification`
  - `Stop`
- Be careful with port assumptions. `IPCServer` can fall back from `19847` to `19857`.

### Settings and diagnostics

- The app includes hook diagnostics and shadowed project detection in the settings window.
- Project-local `.claude/settings.json` files can shadow global hooks. Do not remove this behavior accidentally.

## Verification

For code changes, prefer some combination of:

- `xcodebuild -scheme ClaudeGlance -configuration Release`
- manual review of changed Swift compile paths
- manual review of script syntax if shell files changed

If you cannot run the full app, state that clearly and explain what was verified instead.

## Repository Notes

- `README.md` is user-facing and should stay concise.
- There is currently no dedicated automated test suite in the repository.
- The worktree may contain unrelated user files such as media artifacts; ignore them unless the task is about packaging or assets.
