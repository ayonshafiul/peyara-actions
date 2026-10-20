# peyara-actions

Preset button repo for Peyara Shortcut Deck (`action` + `active-app`).

## Fetch URLs (client)

Base: `https://raw.githubusercontent.com/ayonshafiul/peyara-actions/main`

App index (cache 24h):

```
GET https://raw.githubusercontent.com/ayonshafiul/peyara-actions/main/apps.json
```

Per-app actions:

```
GET https://raw.githubusercontent.com/ayonshafiul/peyara-actions/main/apps/<slug>_<os>.json
```

- `<slug>` comes from `apps.json` (`apps[].slug`).
- `<os>` is `windows` | `macos` | `linux` (matches `active-app.os`).

## Client flow

1. Pair (`hello`) → receive `active-app` (`{app, slug, os, ...}`).
2. `GET apps.json` (cache 24h).
3. Map `active-app.app` → `slug` via `apps[].match` (case-insensitive). Fallback: slugify locally (`lowercase, [^a-z0-9]+ → "-"`).
4. `GET apps/<slug>_<os>.json`, render `label` buttons.
5. Tap → send `{"type":"action","actionId":...,"key":...,"modifiers":[...],"pressDurationMs":...,"pressCount":...}` over `wss://<host>:1313/ws`.
6. Missing file/slug → generic deck or empty state, never crash.

## Available files

| slug | file |
|---|---|
| `google-chrome` | `apps/google-chrome_windows.json`, `apps/google-chrome_macos.json`, `apps/google-chrome_linux.json` |
| `brave` | `apps/brave_windows.json`, `apps/brave_macos.json`, `apps/brave_linux.json` |
| `firefox` | `apps/firefox_windows.json`, `apps/firefox_macos.json`, `apps/firefox_linux.json` |

`apps.json` also reserves slugs for `photoshop`, `illustrator`, `premiere`, `vscode`, `word`, `excel`, `powerpoint` (files TBD).

## Action schema

```json
[{"id":"chrome-new-tab","label":"New Tab","key":"t","modifiers":["ctrl"],"pressDurationMs":50,"pressCount":1}]
```

- `id` required, unique per file, `^[a-z0-9-]+$`, sent back as `actionId`.
- `label` required. `icon` optional (emoji/asset, phone-rendered).
- `key` required (§4 key names). `modifiers` default `[]`.
- `pressDurationMs` default `0` (`0..=5000`). `pressCount` default `1` (`1..=10`).
- `code` reserved for v2, ignored by v1 server.
