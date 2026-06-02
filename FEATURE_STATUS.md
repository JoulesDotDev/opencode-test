# Fork Catch-Up — Feature Status

Our fork (Klaus Scheiböck's 86 commits) replayed onto **latest upstream opencode** (`origin/dev` @ `d5a0ddb`, 2026-06-02) on branch **`klaus-changes`**. Fork point was 2026-04-27; upstream was ~1763 commits ahead, so this was a consolidated 3-way merge, not a commit-by-commit replay.

## Build & Test

| Check | Result |
|---|---|
| `packages/app` typecheck | ✅ 0 errors |
| `packages/opencode` typecheck | ✅ 0 errors |
| `packages/desktop-electron` typecheck | ✅ 0 errors |
| opencode tests (config/provider/tool/mcp) | 872+ pass; only 8 **environmental** failures remain (sandbox blocks the tests' localhost fixture servers — these fail identically on clean upstream) |

## Feature status

| # | Fork feature | Status | Notes |
|---|---|---|---|
| 1 | AI Factory (RRZ) provider — custom provider, host override, model visibility/priority/rollout, branding, model-selection prompt, legacy config migration | ✅ Adapted | `provider.ts` heavily merged (AIFACTORY_ID, proxy, overrides kept). **Runtime needs the AI Factory backend to verify.** |
| 2 | Corporate / Windows proxy for provider + webfetch/websearch | ✅ Adapted | `webfetch.ts`, `websearch.ts`, `mcp-websearch.ts` proxy paths kept. **Runtime needs a corporate proxy / Windows to verify.** |
| 3 | Managed MCP — config + PAT auth + popup | ✅ Kept | `/mcp/managed` endpoint wired to `MCP.managed()`; app `settings-mcp.tsx` adapted to `server-sdk`/`server-sync`. |
| 4 | Custom updater / update-server (beta rollout, TLS bypass, custom server) | ✅ Kept | Lives in `desktop-electron`; typechecks. **Windows packaging path not runtime-verified here.** |
| 5 | `desktop-electron` app shell | ✅ Restored | Merge initially deleted it (upstream removed the package); **all 44 source files restored from fork** and typecheck clean. |
| 6 | Webfetch / websearch improvements (format, decoding, redirects, consent, grounding) | ✅ Adapted | Tests are environmental (localhost). |
| 7 | UI — startup splash MOTD + branding, resumable todo dock, changelog tab, boot/session-restore fixes | ✅ Kept | Build green; composer/todo-dock wiring intact. |
| 8 | Permissions — remember-accepted + **full access** | ✅ Intact | `decide("full-access")` → `enableAutoAcceptDirectory` wired; auto-respond module preserved. (Dead `"never"` response type value removed.) |
| 9 | Auto-compact on model switch (smaller target) | ✅ Intact | `model-switch-compaction.ts` preserved; caller adapted to upstream providers-Map. |
| 10 | Message queue + steering (composer followup queue) | ✅ Wired | Reconciled to upstream composer props (`onSteer`/`onDelete`/`onMove`). |
| 11 | Built-in Playwright tool | ✅ Restored | Merge dropped it from the registry; **re-wired** (`registry.ts`). |
| 12 | chokidar watcher fallback; bundled watcher/ripgrep for Windows | ✅ Kept | In upstream's current `startSubscription`; desktop bundling in `desktop-electron`. |
| 13 | **Background bash tools** (`bash_read`, `bash_stop`, `run_in_background`) | ✅ Restored | Initially dropped (the `BashProcess` Effect service collided with the refactor). **Re-ported**: service adapted to upstream's moved helpers, layer-provision fixed, `run_in_background` re-added to the shell tool, all three re-wired into the registry. Note: upstream opencode has **no** equivalent — these are fork-only. Typecheck 0; tool tests pass. |

## Resolution policy used

- Infra/CI/lockfiles/generated → upstream
- Old Tauri `packages/desktop` → upstream's Electron version (we ship `desktop-electron`)
- Fork-only files → kept
- Files upstream intentionally deleted (refactored-away `provider/schema.ts`, `provider/models.ts`, `global-sdk.tsx`, old server routes) → **not resurrected** (would break the build); fork logic on top of them dropped/adapted
- Shared code → merged (upstream structure + fork feature intent)
- `global-sdk`/`global-sync` → `server-sdk`/`server-sync`; providers list → Map

## Still needs human / real-environment verification

These can't be fully verified in this sandbox and should be checked on a real setup before release:

1. **AI Factory provider** end-to-end (needs the AI Factory/RRZ backend)
2. **Corporate proxy + Windows proxy** for webfetch/websearch (needs proxy + Windows)
3. **Windows-only**: updater helper / deferred install, PowerShell ripgrep/webfetch, installer/packaging
4. **`desktop-electron` runtime** (typechecks, but boot + the app shell against upstream's new `app`/`ui` should be smoke-tested)
5. **Managed-MCP** end-to-end against a managed server

## Background-bash tools — re-ported (done)

`bash_read` / `bash_stop` / `run_in_background` are back and working on the current base. Upstream opencode has **no** equivalent (its shell tool is synchronous-only), so these remain a fork-only capability. The `BashProcess` service was adapted to upstream's moved helpers and its layer is now provided in the registry + test layer sets.
