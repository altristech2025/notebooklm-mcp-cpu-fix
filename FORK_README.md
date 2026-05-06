# This is a fork — please use upstream when possible

This repository is a **fork** of [PleasePrompto/notebooklm-mcp](https://github.com/PleasePrompto/notebooklm-mcp)
based on tag `v1.1.2`, with two surgical patches that fix sustained high CPU
and orphan processes on macOS.

## Why this fork exists

While using `notebooklm-mcp@1.2.1` on macOS (M-series), one of our machines
accumulated 8+ orphan processes consuming up to **99% CPU each**, causing
sustained overheating and fan noise. The root causes were already documented
in two open upstream issues:

- [#29 — orphan/stuck processes on macOS, sustained 600%+ CPU](https://github.com/PleasePrompto/notebooklm-mcp/issues/29)
- [#16 — 100% CPU infinite loop when browser context becomes unresponsive](https://github.com/PleasePrompto/notebooklm-mcp/issues/16)

A pull request with both fixes has been opened upstream. **If/when that PR is
merged, this fork becomes redundant** and you should switch back to upstream.

## What changed (vs. upstream v1.1.2)

Two patches, both small and targeted:

### Patch 1 — Detect stdin EOF (fix #29)

`src/index.ts`: the MCP server only listened for `SIGINT` / `SIGTERM`, but
Claude Desktop and Claude Code signal disconnect by closing the stdio pipe,
not by sending a signal. Patchright/Chromium then keeps the event loop alive
indefinitely, leaving the process zombie.

Fix: add `process.stdin.on('end' | 'close')` handlers that trigger graceful
shutdown when the parent MCP client disconnects. Plus a defensive 30-minute
idle watchdog (`.unref()`'d, so it never itself blocks exit) for edge cases
where stdin events don't fire.

### Patch 2 — Replace busy-wait `page.waitForTimeout` (fix #16)

`src/utils/page-utils.ts`: when the browser context becomes unresponsive,
`page.waitForTimeout()` returns immediately, causing the polling `while` loop
in `waitForLatestAnswer` to busy-spin at 100% CPU.

Fix: use the existing `sleep()` helper from `stealth-utils.ts` (a plain
`setTimeout`-based promise, immune to browser disconnects) plus a
`page.isClosed()` early-exit at the top of each iteration.

## Installation

This fork is **not published to npm** by design — you should prefer upstream.

If you want to use this fork directly:

```bash
git clone https://github.com/altristech2025/notebooklm-mcp-cpu-fix.git
cd notebooklm-mcp-cpu-fix
git checkout altris-patches
npm install
```

Then point your MCP client (Claude Desktop / Claude Code) at the local
`dist/index.js` instead of `npx notebooklm-mcp@latest`.

## Credits

All credit for the original project goes to **Gérôme Dexheimer ([@PleasePrompto](https://github.com/PleasePrompto))**.
This fork only adds two bug fixes; it is not a new project, and the LICENSE
(MIT) is preserved unchanged.

If you find this useful, please [star the upstream
repository](https://github.com/PleasePrompto/notebooklm-mcp) — that is where
the real work lives.
