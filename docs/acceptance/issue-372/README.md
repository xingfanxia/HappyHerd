# Issue 372: Opus 5.5 on lockfile builds

Owner: [#372](https://github.com/NickGuAI/HappyHerd/issues/372).

## Cause

Turns started from the app run through `@anthropic-ai/claude-agent-sdk`, which
launches the Claude Code binary bundled in its platform package. The lockfile
pinned the SDK at 0.3.260, whose bundled Claude Code is 2.1.260. The API
rejects `claude-opus-5-5` for that version. A session started in a terminal
with `happyherd claude` is not affected, because local mode runs the Claude
Code installed on the host.

## Change

`server/pnpm-lock.yaml` only: the nine `@anthropic-ai/claude-agent-sdk` entries
move from 0.3.260 to 0.3.289 (bundled Claude Code 2.1.289) with their registry
integrity values. 0.3.289 is inside the declared range `^0.3.259`, and its peer
dependencies are the same as 0.3.260, so no `package.json` changes.

## Evidence

Linux x64 host, Node 24.20.0, pnpm 10.11.0, 2026-10-05. One self-hosted
account and machine; no shared service was touched.

Direct call to each Claude Code binary with the same sign-in, prompt and
`--model claude-opus-5-5`:

| Binary | Result |
|---|---|
| SDK 0.3.260 bundle (2.1.260) | `API Error: 400 Claude Code 2.1.260 does not support this model; version 2.1.280 or newer is required.` |
| SDK 0.3.289 bundle (2.1.289) | `ok`, model usage `claude-opus-5-5` |

Checks on this branch:

- `pnpm install --frozen-lockfile`: passed.
- `pnpm --filter @happyherd/cli typecheck`: passed.
- `vitest run --project unit src/claude/sdk src/claude/claudeRemote.test.ts src/claude/claudeRemote.rotation.test.ts src/claude/claudeRemote.plugins.test.ts src/capabilities`: 7 files, 106 tests passed.
- `scripts/build-native-installer-asset.sh --target linux-x64`: built; the
  asset was installed with `install.sh --asset`.

Runtime, on the installed asset with the bundled self-host server:

- Session created through the daemon with `--provider claude --model
  claude-opus-5-5`, first message sent with `happyherd session send`: the
  Claude transcript records Claude Code 2.1.289, entrypoint `remote_mobile`,
  and model `claude-opus-5-5` on every assistant record.
- A message sent to that session from the Web app in a clean browser profile
  was answered on `claude-opus-5-5`.
- A session started in a terminal with `happyherd claude --model
  claude-opus-5-5` answered on `claude-opus-5-5`; a Web message then switched
  it to remote mode and the next answer was also on `claude-opus-5-5`.

Not covered: macOS, Linux arm64, the mobile app, and the full contract suite.
The changelog has an October 5 entry for this fix. The golden screenshots that
show the latest changelog entries were not regenerated here.
