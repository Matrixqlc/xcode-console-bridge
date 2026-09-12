# Xcode Console Bridge

[中文说明](README.zh-CN.md)

Turn the text currently visible in Xcode's Debug Console into a local evidence bundle that an AI coding agent can inspect safely.

Xcode is excellent at showing runtime output, but it does not offer a documented public API for reading the populated Debug Console. That leaves a gap in a macOS iOS development loop: an agent can read source code and make a change, while the developer still has to manually export the runtime evidence after reproducing a problem.

Xcode Console Bridge closes that gap without pretending that an arbitrary clipboard payload is trustworthy.

## What it does

1. Activates one explicitly matched Xcode window.
2. Uses Xcode's Activate Console shortcut.
3. Verifies, through macOS Accessibility, that the focused element is `Console`.
4. Copies the Console text, restores the previous clipboard and frontmost app, and rejects HTML or implausible payloads.
5. Writes a local evidence bundle:

```text
xcode-console-evidence/
  <run-label>/<timestamp>/
    xcode-console.raw.log       # full local source of truth
    xcode-console.digest.md     # bounded AI-friendly navigation view
    xcode-console.metrics.tsv   # small comparable counts
    xcode-console.compare.md    # comparison with an optional baseline
  latest -> <run-label>/<timestamp>
```

It does not read a device remotely, start a build, run an app, upload logs, or continuously monitor your Console. It takes an explicit local snapshot of what Xcode is displaying now.

## Requirements

- macOS
- Xcode running with a Debug Console available
- `bash`, `osascript`, `pbcopy`, `pbpaste`, and [ripgrep](https://github.com/BurntSushi/ripgrep)
- Accessibility permission for the terminal or AI-agent host running the command

No network connection is required to capture a log.

## Install

Clone the repository and invoke the executable directly:

```bash
git clone https://github.com/Matrixqlc/xcode-console-bridge.git
cd xcode-console-bridge
./bin/xcode-console-bridge --help
```

## Capture a run

Use a window token specific enough to match exactly one open Xcode window. Add an identity token that is expected to occur in your own app logs when possible; it strengthens the copied-payload check.

```bash
./bin/xcode-console-bridge \
  --window-token MyApp \
  --app-name MyApp \
  --identity-token MyApp \
  --run compression-regression
```

If you are unsure what token to use:

```bash
./bin/xcode-console-bridge --list-windows
```

The tool stops if the token matches zero or multiple windows. It does not guess which project you meant.

### Known-good baseline

Capture an accepted run with `--set-baseline`. Later captures in the same evidence root receive a small metric comparison.

```bash
./bin/xcode-console-bridge --window-token MyApp --run known-good --set-baseline
```

### Manual-copy fallback

If Accessibility permission is unavailable or an Xcode update changes the Console accessibility tree, manually copy the Console text and normalize it into the same evidence format:

```bash
./bin/xcode-console-bridge --import-clipboard --run manual-copy
```

For an already saved file:

```bash
./bin/xcode-console-bridge --raw-log /path/to/xcode-console.raw.log --run imported-log
```

## Give the evidence to an AI agent

Tell your agent to start with the digest and compare report, then use bounded searches in the raw log only when it needs to confirm a claim. Do not ask it to paste or scan a whole raw log into chat.

```text
Read xcode-console-evidence/latest/xcode-console.digest.md first.
Then inspect xcode-console-evidence/latest/xcode-console.compare.md.
Treat xcode-console.raw.log as the source of truth; query it with targeted,
line-bounded searches before making a diagnosis.
```

See [AI integration guidance](docs/AI-INTEGRATION.md) for a reusable instruction block.

## Safety and privacy

- Evidence stays on the local Mac unless you deliberately share it.
- Console output can contain paths, identifiers, request data, or other sensitive material. Review and redact it before attaching it to an issue or sending it to a hosted model.
- The tool restores the clipboard it replaced during an automatic capture.
- Automatic capture is limited by a 12-second process watchdog. A hung AppleScript process is terminated rather than being left behind.
- A successful copy is not enough: the tool requires verified Console focus and a console-shaped or identity-matching payload.
- There is no automatic retention cleanup in this first release. You decide if and when evidence directories are removed.

## Limitations

This is a practical bridge over a missing Xcode Console export API, not a supported Xcode automation API. Xcode or macOS accessibility changes may require an update. It deliberately fails closed when it cannot prove the focused element is `Console`.

`os_log` and the macOS `log` command are complementary system logging tools; they are not a replacement for copying the exact, currently displayed Xcode Debug Console state, which can include debugger and process output relevant to the active reproduction.

## License

[MIT](LICENSE)
