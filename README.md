# peyara-actions

Preset button repo for Peyara Shortcut Deck (`action` + `active-app`).

## Fetch URLs (client)

Base: `https://raw.githubusercontent.com/ayonshafiul/peyara-actions/refs/heads/main`

App index (cache 24h):

```
GET https://raw.githubusercontent.com/ayonshafiul/peyara-actions/refs/heads/main/apps.json
```

Per-app actions:

```
GET https://raw.githubusercontent.com/ayonshafiul/peyara-actions/refs/heads/main/apps/<slug>_<os>.json
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
| `google-chrome` | `apps/google-chrome_windows.json`, `apps/google-chrome_macos.json`, `apps/google-chrome_linux.json` (71 / 63 / 71 actions) |
| `brave` | `apps/brave_windows.json`, `apps/brave_macos.json`, `apps/brave_linux.json` (71 / 63 / 71 actions) |
| `firefox` | `apps/firefox_windows.json`, `apps/firefox_macos.json`, `apps/firefox_linux.json` (75 / 66 / 75 actions) |
| `photoshop` | `apps/photoshop_windows.json`, `apps/photoshop_macos.json`, `apps/photoshop_linux.json` (128 / 128 / 128 actions) |
| `illustrator` | `apps/illustrator_windows.json`, `apps/illustrator_macos.json`, `apps/illustrator_linux.json` (118 / 118 / 118 actions) |
| `premiere` | `apps/premiere_windows.json`, `apps/premiere_macos.json`, `apps/premiere_linux.json` (98 / 98 / 98 actions) |
| `vscode` | `apps/vscode_windows.json`, `apps/vscode_macos.json`, `apps/vscode_linux.json` (108 / 108 / 108 actions) |
| `zed` | `apps/zed_windows.json`, `apps/zed_macos.json`, `apps/zed_linux.json` (67 / 67 / 67 actions) |
| `pycharm` | `apps/pycharm_windows.json`, `apps/pycharm_macos.json`, `apps/pycharm_linux.json` (123 / 123 / 123 actions) |
| `intellij-idea` | `apps/intellij-idea_windows.json`, `apps/intellij-idea_macos.json`, `apps/intellij-idea_linux.json` (122 / 122 / 122 actions) |
| `android-studio` | `apps/android-studio_windows.json`, `apps/android-studio_macos.json`, `apps/android-studio_linux.json` (123 / 123 / 123 actions) |
| `xcode` | `apps/xcode_macos.json` (macOS only, 87 actions) |
| `word` | `apps/word_windows.json`, `apps/word_macos.json`, `apps/word_linux.json` (101 / 101 / 101 actions) |
| `excel` | `apps/excel_windows.json`, `apps/excel_macos.json`, `apps/excel_linux.json` (101 / 101 / 101 actions) |
| `powerpoint` | `apps/powerpoint_windows.json`, `apps/powerpoint_macos.json`, `apps/powerpoint_linux.json` (88 / 87 / 88 actions) |
| `ableton-live` | `apps/ableton-live_windows.json`, `apps/ableton-live_macos.json`, `apps/ableton-live_linux.json` (154 / 154 / 154 actions) |

Every deck ships at least 60 shortcuts.

## Action schema

```json
[{"id":"chrome-new-tab","label":"New Tab","key":"t","modifiers":["ctrl"],"pressDurationMs":50,"pressCount":1}]
```

- `id` required, unique per file, `^[a-z0-9-]+$`, sent back as `actionId`.
- `label` required. `icon` optional (emoji/asset, phone-rendered).
- `key` required (§4 key names). `modifiers` default `[]`.
- `pressDurationMs` default `0` (`0..=5000`). `pressCount` default `1` (`1..=10`).
- `code` reserved for v2, ignored by v1 server.
