# Log

## 2026-09-03

**Initial bundle.**

Created the OKF bundle for herderweb from the codebase and `docs/ARCHITECTURE_RECOMMENDATIONS.md`. Documents the Go bridge (`cmd/herdr-bridge`, `internal/{herdr,server,protocol,config,webui}`), the SvelteKit frontend (`web/src`), the HTTP/WS service surface, the session-snapshot model, and configuration.

Reflects the post-`bridge-ipc-hardening` backend: `herdr.Client` uses a persistent, id-multiplexed socket connection and `server.Hub` bounds in-flight WebSocket RPC goroutines per connection.

## 2026-09-08

**Direct-control swipe + theme/chrome-sync fixes.**

Documented the pane view's `◎ direct control` toggle (agent panes: swipe the transcript to send one arrow key per swipe via `agent.send_keys`, replacing native scroll; gated off if the pane stops being an agent) and the `theme-color`/`apple-mobile-web-app-status-bar-style` meta sync that now tracks the active theme pre-paint and — for `theme-color` — on every live switch, fixing a stuck-dark-chrome bug on light themes (the status-bar-style meta is kept correct too, but iOS only applies it on an installed PWA's next cold launch). Corrected a stale theme list (`herdr-dark`/`ash`/`gruvbox`/`solarized-light`) across `themes.md`, `settings.md`, and `navigation.md` to the actual four themes (`herdr-dark`, `gruvbox`, `solarized-light`, `paper`) — `ash` never shipped; `paper` was missing. Fixed the same list in `README.md` and `AGENTS.md`. Corrected `terminal-view.md`'s stale "soft-wrap under 880px" claim to the actual `use:fitToWidth` font-shrink behaviour.

## 2026-09-28

**Windows support.**

Documented the Windows transport (Herdr's named pipe behind `herdr.ConfigDir()`/`Listen`), the `-session` flag, and the Task Scheduler `-service` backend.

**Browser hardening.**

Documented the bridge's `hostGuard` (Host/Origin checks against DNS rebinding and cross-site WebSocket hijacking), the `-allow-host` flag, and the `browserMethods` allowlist on the `/ws` pass-through.

**UI settings move out of Herdr's config.toml.**

Herdr now reports unknown tables in `config.toml` (`unknown config section [web]`), and the bridge's `server.reload_config` after every settings save surfaced that warning in the Herdr TUI. Settings now live in `herdrweb-settings.json` beside `config.toml`; `config.Migrate` moves an existing `[web]` table there at startup and deletes only its lines. `/api/config` no longer reloads Herdr. Updated `config/settings.md`, `packages/config.md`, `packages/server.md`, `packages/cmd-herdr-bridge.md`, `services/bridge-http.md`, `concepts/themes.md` and the indexes.

**Collapsible desktop sidebar; clean pane switches.**

On `/pane/*` the desktop sidebar can be hidden so the terminal gets the full width (for tablets in landscape), shown over the pane from the header, and pinned back. Switching panes used to reuse the terminal scroller: the new pane kept the previous one's horizontal offset, and iOS Safari left tiles of the old pane's text painted beside the new text. The scroller is now keyed per pane, and lines are tagged with the pane they were read from. Updated `frontend/navigation.md` and `concepts/terminal-view.md`.

**Terminal fits its view at every width.**

`fitToWidth` applied only below the 880px breakpoint, so an iPad in landscape showed a 152-column pane a few columns short even with the sidebar hidden. It now fits at every width and measures the text with a `Range` instead of `scrollWidth`, which stayed at the box width once fitted and so kept the font shrunk after the view widened. Updated `concepts/terminal-view.md`.

**One Herdr connection per call; prompts without `wait`.**

After a prompt was sent from the web UI, the pane stopped updating and its status stayed stale until the agent finished. The bridge multiplexed every call over one persistent Herdr connection, but Herdr answers one request per connection and then closes it: measured, a second request after the first reply finds the pipe closing, and requests written while one is in progress wait behind it and then fail with EOF. The Composer's `agent.prompt` carried `wait: {until: [idle, blocked]}`, so each prompt held that connection until the agent went idle. `herdr.Client.Call` now dials per call, and the Composer sends prompts without `wait`, whose result nothing read. Removed `mux_test.go`; the server test fake now serves one request per connection. Updated `packages/herdr.md`, `packages/server.md`, `packages/index.md`, `architecture/overview.md`, `architecture/data-flow.md` and `frontend/composer.md`.
