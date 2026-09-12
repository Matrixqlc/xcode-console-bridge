# Release notes

## Initial public release — 2026-09-12

- Published `xcode-console-bridge`, a macOS command-line tool that turns the
  currently displayed Xcode Debug Console into a local evidence bundle for AI
  coding agents.
- Included an English README, a complete Chinese README, AI integration
  guidance, a MIT license, and rules that exclude generated local evidence.
- The capture path verifies Accessibility focus on `Console`, restores the
  prior clipboard and foreground app, applies a 12-second process watchdog,
  and rejects non-console-shaped copied payloads.

### Verification

- `bash -n bin/xcode-console-bridge` passed.
- `git diff --check` passed before each published commit.
- A static privacy review found no personal email, absolute local path,
  historical project/task reference, log content, or credential in tracked
  source files.
- Automated tests and a live Xcode Console capture were not run for this
  release.

### Deliberately not included

- Any existing project profiles, task identifiers, historical logs, or user
  data.
- Automatic evidence-retention deletion.
