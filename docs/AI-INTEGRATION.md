# AI integration guidance

Use this tool after a developer has reproduced a runtime issue in Xcode and explicitly asks an agent to inspect the current Console. It is a snapshot boundary, not a background surveillance mechanism.

## Reusable agent instruction

```text
When I ask you to pull the current Xcode Console, run:

  ./bin/xcode-console-bridge --window-token <project-token> --app-name <app-name> \
    --identity-token <app-token> --run <short-reproduction-label>

Read xcode-console-evidence/latest/xcode-console.digest.md first, then
xcode-console-evidence/latest/xcode-console.compare.md. Treat
xcode-console-evidence/latest/xcode-console.raw.log as local source evidence:
search it with bounded queries and cite relevant line numbers. Do not infer a
runtime fact from the digest alone, and do not upload or paste the full raw log
without my explicit instruction.
```

## Evidence hierarchy

1. `xcode-console.raw.log` is the full captured local evidence.
2. `xcode-console.digest.md` is a lossy navigation aid, not a verdict.
3. `xcode-console.metrics.tsv` is a count summary, useful for comparison but
   insufficient to establish causality.
4. `xcode-console.compare.md` compares those counts with an explicitly saved
   known-good baseline.

## Failure handling

- Multiple windows matched: ask the developer to provide a narrower token.
- No active Console: ask the developer to run from Xcode and show its Debug
  Console before retrying.
- Accessibility permission missing: ask the developer to grant it to the host
  running the command.
- Two verified-focus copies fail: ask the developer to manually copy the
  Console and use `--import-clipboard`.
- Rejected payload: do not analyze it. It may be stale clipboard text, a web
  page, or text from the wrong Xcode focus target.
