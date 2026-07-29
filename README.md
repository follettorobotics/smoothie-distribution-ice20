# smoothie-distribution-ice20

Public OTA channel for **ICE20** smoothie machines (J-Group).

The rev0/gen2 fleet uses a different repo — [`smoothie-distribution`](https://github.com/follettorobotics/smoothie-distribution).
The two are deliberately separate: `latest.json` is a single file polled by
every kiosk on a channel, and `minVersionCode` in it can pause sales fleet-wide.
Sharing one manifest between two machine families would mean an ICE20 release
offering itself to rev0 kiosks — and an urgent ICE20 fix locking them.

## What's in here

| File | Purpose |
|------|---------|
| `latest.json` | Manifest the ICE20 kiosk app polls. Overwritten by every release. |
| `changelog.json` | Full release history, feeds the patch-notes page. |
| `index.html` | The patch-notes page itself (GitHub Pages). |
| Releases | The signed APKs (`app-release.apk`) the manifest points at. |

Published page: https://follettorobotics.github.io/smoothie-distribution-ice20/

## Do not edit by hand

Everything here is written by `.github/workflows/release.yml` in the private
`follettorobotics/smoothie-agent` repo. Push an `ice20-v*.*.*` tag there and
this repo updates itself:

```bash
git tag -a ice20-v1.0.0 -m "$(cat <<'EOF'
---ko---
- 첫 ICE20 릴리스
---en---
- First ICE20 release
EOF
)"
git push origin ice20-v1.0.0
```

A hand-edit to `latest.json` reaches every ICE20 kiosk within two hours. The
one legitimate manual edit is lowering `minVersionCode` back to `1` to retract
a mistaken critical release — see below.

## latest.json

```json
{
  "versionCode": 1010000,
  "versionName": "1.0.0",
  "apkUrl": "https://github.com/follettorobotics/smoothie-distribution-ice20/releases/download/ice20-v1.0.0/app-release.apk",
  "sha256": "…",
  "minVersionCode": 1,
  "releaseNotes": "…",
  "releasedAt": "2026-07-29T00:00:00Z"
}
```

`versionCode` carries a **+1000000 channel offset** so ICE20 codes can never
collide with rev0's, even if a manifest were ever crossed by accident.
`ice20-v1.0.0` → `1*10000 + 0*100 + 0 + 1000000` = `1010000`.

`minVersionCode` is the **critical-update floor**. It normally stays `1`.
Raising it to a build's own `versionCode` makes every kiosk below that version
stage the APK and pause sales at idle until it is installed. Lowering it back
to `1` clears that lock fleet-wide on the next poll — that is the undo.

## Verifying an APK

```bash
curl -sL -o app-release.apk \
  https://github.com/follettorobotics/smoothie-distribution-ice20/releases/download/ice20-vX.Y.Z/app-release.apk
shasum -a 256 app-release.apk   # must equal .sha256 in latest.json
```

The kiosk does this itself before installing; a mismatch aborts the update.
