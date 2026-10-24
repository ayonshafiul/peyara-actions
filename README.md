# peyara-actions

Preset button repo for Peyara Shortcut Deck (`action` + `active-app`).

## Fetch URLs (client)

Base: `https://raw.githubusercontent.com/ayonshafiul/peyara-actions/refs/heads/main`

App index:

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
2. `GET apps.json`.
3. Map `active-app.app` → `slug` via `apps[].match` (case-insensitive). Fallback: slugify locally (`lowercase, [^a-z0-9]+ → "-"`).
4. `GET apps/<slug>_<os>.json`, render `label` buttons.
5. Tap → send `{"type":"action","actionId":...,"key":...,"modifiers":[...],"pressDurationMs":...,"pressCount":...}` over `wss://<host>:1313/ws`.
6. Missing file/slug → generic deck or empty state, never crash.

## Apps

`apps.json` is the source of truth for the app list, labels, and `active-app.app`
matching aliases (`apps[].match`). Each app has one actions file per supported OS:

```
apps/<slug>_<os>.json
```

Not every app ships all three OS files — platform-exclusive apps only include the OS
they run on (for example `xcode` is macOS-only, `cakewalk` is Windows-only).

## Action schema

```json
[{"id":"chrome-new-tab","label":"New Tab","key":"t","modifiers":["ctrl"],"pressDurationMs":50,"pressCount":1}]
```

- `id` required, unique per file, `^[a-z0-9-]+$`, sent back as `actionId`.
- `label` required. `icon` optional (emoji/asset, phone-rendered).
- `key` required (§4 key names). `modifiers` default `[]`.
- `pressDurationMs` default `0` (`0..=5000`). `pressCount` default `1` (`1..=10`).
- `code` reserved for v2, ignored by v1 server.

## Contributing: add a new app

1. **Pick a slug.** Lowercase, hyphenated, matching the filename: `sample-app`.
2. **Create one file per supported OS** in `apps/`:
   - `apps/sample-app_windows.json`
   - `apps/sample-app_macos.json`
   - `apps/sample-app_linux.json`

   Apps that only run on one platform need only that file (see `xcode`, `cakewalk`).
3. **Register the app in `apps.json`:**

   ```json
   {
     "slug": "sample-app",
     "label": "Sample App",
     "match": ["Sample App", "sample-app.exe", "com.example.sample-app"]
   }
   ```

   `match` is case-insensitive and should contain what the OS reports as
   `active-app.app`: display names, process/executable names, and macOS bundle IDs.
   Add versioned names (for example `"Sample App 2"`) when the app reports them.
4. **Add actions** using the schema above. Minimum example:

   ```json
   [
     {"id": "sample-record", "label": "Start Recording", "key": "r", "modifiers": ["ctrl"], "pressDurationMs": 50, "pressCount": 1},
     {"id": "sample-export", "label": "Export", "key": "e", "modifiers": ["ctrl"], "pressDurationMs": 50, "pressCount": 1},
     {"id": "sample-play", "label": "Play", "key": "space", "modifiers": [], "pressDurationMs": 50, "pressCount": 1}
   ]
   ```

   - Prefix `id` with the app (for example `sample-`) and keep IDs stable; clients send them back as `actionId`.
   - Keep the same IDs and labels across OS files, and change only `key` / `modifiers`.
   - Aim for at least 30 useful shortcuts per file.
5. **Map platform keys.** Windows/Linux use `ctrl` + `alt`; macOS uses `cmd` + `alt` (Option).
   Single-key shortcuts are usually identical everywhere. Avoid keys the server cannot emit on macOS:
   `+`, `{`, `}`, `&`, `~`, `^`, `"` and `insert`/numpad keys.
6. **Smoke test** the published URL after merging:

   ```bash
   curl -s https://raw.githubusercontent.com/ayonshafiul/peyara-actions/refs/heads/main/apps/sample-app_macos.json | python3 -m json.tool | head
   ```
