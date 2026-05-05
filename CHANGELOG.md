# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.4] - 2026-05-04

Two new behaviors. CLI flags, bundle identifier, and state schema
unchanged.

### Added
- **Wave overlay** in the menu bar app. While engaged, optionally
  render a translucent stacked-ridge animation across each display
  via a click-through borderless panel per `NSScreen` (drawn with
  `Canvas` + `TimelineView`, ~48 stroked sine paths per screen,
  multi-octave). Mutually exclusive with **Dim displays** since the
  overlay needs the screen lit. New menu toggle, persisted in
  `UserDefaults` under `halfcaf.overlay`. Lifecycle is driven from
  the bar app: opens on engage success, closes on session end (CLI
  exit, hotkey, manual disengage), and on app termination.

### Changed
- **Display power yields to the OS while the screen is locked.**
  When `com.apple.screenIsLocked` fires (remote screen sharing,
  manual lock, sleep with password, screensaver with password),
  halfcaf now stops fighting display power: WakeWatcher pauses,
  brightness restores to the captured value, and any held
  `PreventUserIdleDisplaySleep` assertion is released. The lock
  screen behaves normally — visible on input, idle-sleeps after the
  OS display-sleep timeout, wakes on key/trackpad. The
  `PreventUserIdleSystemSleep` assertion stays held the whole time
  so background work keeps running. On unlock, halfcaf re-dims to 0
  and re-arms the WakeWatcher regardless of whether the original
  session was started with `--no-dim`, since being locked-out means
  the user was actually away.

## [1.0.3] - 2026-05-02

Bug fix.

### Fixed
- With `--no-dim` (or **Dim displays** unchecked in the bar app),
  the display would still go dark on macOS's idle timer because
  halfcaf only held a system-sleep assertion. halfcaf now also
  holds `PreventUserIdleDisplaySleep` (the same assertion
  `caffeinate -d` uses) whenever dimming is opted out, so the
  display stays lit while halfcaf is engaged. Bundle identifier,
  CLI flags, and state schema unchanged.

## [1.0.2] - 2026-05-01

Cosmetic patch. New app icon: a moody coffee cup with glowing
orange lid against a deep teal painterly background. Same bundle
identifier and same name field as v1.0.1; nothing else moved.

### Changed
- App icon (CFBundleIconFile resource AppIcon.icns).

## [1.0.1] - 2026-05-01

Cosmetic patch.

### Changed
- Bundle name is now "Halfcaf" (was "halfcaf"). Visible in Cmd+Tab,
  Dock, Finder, right-click app menu. Bundle identifier
  `com.rasterstate.halfcaf-bar` is unchanged so login items,
  persisted settings, and TCC permissions carry over from v1.0.0
  without re-prompting.

## [1.0.0] - 2026-05-01

First public release.

### Added
- **Settings window** in the menu bar app, accessible via "Settings..."
  in the menu (or the standard ⌘, shortcut). Currently exposes the
  disengage hotkey configuration.
- **Configurable global disengage hotkey** with a SwiftUI key
  recorder. Press the recorder, hit any modifier-prefixed key combo,
  and the new binding is persisted to UserDefaults and re-registered
  with Carbon. Reset-to-default button included. Default remains
  ⌘⇧⎋. Bare-Esc cancels recording; bare keys (no modifier) are
  rejected with a beep.
- **`--verbose` actually emits log lines now.** Each phase of capture,
  mutate, restore, and shutdown writes a timestamped line to stderr.
- **`--json`** flag for `--status`: machine-readable output for
  scripts. `{"status": "active|stale|none", "session": {...}}`.

### Changed
- API stability commitment: the CLI flag set is now considered stable
  for the v1.x series.

## [0.4.0] - 2026-05-01

### Added
- **Menu bar app** (`HalfcafBar.app`): SwiftUI `MenuBarExtra` agent
  that wraps the `halfcaf` CLI as a subprocess. One-click engage and
  disengage, persisted toggle states (Dim, Mute, Toggle Focus),
  Launch at Login via `SMAppService`, and status reflection from the
  shared `state.json`.
- **Global hotkey ⌘⇧⎋** for blind-disengage when the screen is dark,
  registered via Carbon `RegisterEventHotKey`.
- **Engaged-state visual change**: menu bar icon swaps from `moon` to
  `moon.zzz.fill` when a session is active.
- **Failure surfacing**: stderr from the spawned `halfcaf` is
  captured and shown as an `NSAlert` if the process exits non-zero
  for any reason other than user interrupt.
- **`.app` bundle assembly** (`scripts/build-app.sh`): generates
  `Info.plist` (LSUIElement, NSAppleEventsUsageDescription),
  programmatic `AppIcon.icns` from `scripts/make-icon.swift`, and
  bundles the CLI inside `Contents/MacOS/` so a single Cask install
  provides both the GUI and the binary on PATH.
- **Release pipeline**: `scripts/release.sh` orchestrates build,
  codesign, notarize, package CLI tarball, package DMG, optional R2
  upload. Mirrors the Pier project's structure;
  `scripts/lib/notarize-common.sh` and `scripts/upload-to-r2.sh` are
  reused verbatim from Pier.
- **CLI entitlements**: `automation.apple-events` for `osascript` and
  `shortcuts run`. GUI gets a no-op entitlements file for hardened
  runtime.
- **Homebrew Cask draft**: `Casks/halfcaf.rb` for `brew install
  --cask halfcaf` once the tap is published; includes a `binary`
  stanza so the bundled CLI is symlinked onto PATH.
- **3 new tests**: `RestoreCLITests` covers `--status` and `--restore`
  CLI paths.

### Changed
- **Toggle Focus defaults to OFF in the menu bar.** macOS Focus
  syncs across iCloud-connected devices by default; defaulting on
  surprised users by also engaging DND on iPhone/iPad. CLI behavior
  unchanged: `--no-focus` is still the explicit opt-out.
- README rewritten to cover both the CLI and the bar app, with a
  dedicated section on Focus / DND iOS sync trade-offs.

## [0.3.0] - 2026-05-01

### Added
- Initial CLI release.
- IOPMAssertion (`PreventUserIdleSystemSleep`) to keep the system
  awake without preventing display sleep.
- Multi-display brightness dimming via `DisplayServicesSetBrightness`
  (private framework, dlopen'd at runtime).
- 500ms `WakeWatcher` polling that re-asserts brightness 0 on display
  wake, after a probe spike showed
  `CGDisplayRegisterReconfigurationCallback`,
  `NSWorkspace.screensDidWake`, and `IORegisterForSystemPower` all
  fail to fire on display-only sleep on macOS 26.
- Audio mute via `osascript`, capturing both volume and mute state
  independently.
- Optional Focus / DND toggling via `shortcuts run` with capability
  detection: prints clear setup instructions if the
  `halfcaf-focus-on` / `halfcaf-focus-off` shortcuts are missing,
  continues without Focus toggling rather than failing.
- Atomic `state.json` write under
  `~/Library/Application Support/halfcaf/` *before* any system
  mutation. Auto-restores on next `halfcaf` launch if the recorded
  pid is no longer alive.
- Wait conditions: `--timeout`, `--until-process`, `--until-command`,
  trailing `--` for direct command execution.
- Opt-out flags: `--no-dim`, `--no-mute`, `--no-focus`.
- `--restore`, `--status`, `--version`, `--help`.
- `DispatchSource` signal handlers for SIGINT, SIGTERM, SIGHUP.
- Homebrew tap formula draft (source-build).
- 18 unit tests for duration parsing, state JSON round-trip, and
  atomic state-file persistence.
- README with full walkthrough including the optional Shortcuts.app
  setup.

[Unreleased]: https://github.com/rasterandstate/halfcaf-app/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/rasterandstate/halfcaf-app/compare/v0.4.0...v1.0.0
[0.4.0]: https://github.com/rasterandstate/halfcaf-app/compare/v0.3.0...v0.4.0
[0.3.0]: https://github.com/rasterandstate/halfcaf-app/releases/tag/v0.3.0
