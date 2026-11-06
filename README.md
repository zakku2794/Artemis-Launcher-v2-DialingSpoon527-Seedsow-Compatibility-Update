# Artemis Launcher v2 — DialingSpoon527 Seedsow Lua Editor compatibility update

Unofficial community compatibility package for **Seedsow on Nintendo Switch 2** using [DialingSpoon527's `artemis-installer` branch](https://github.com/DialingSpoon527/seedsow-lua-editor/tree/artemis-installer). Based on [alula's Artemis Launcher v2](https://github.com/alula/switch-artemis-launcher/releases/tag/v2).

**Tested on one installation:** the adapted HTTP handler accepted a 100 KB request, and the original Lua Editor still ran. Other installations have not been tested. **Back up your entire Seedsow save before installing.** Do not use this package with a standard Artemis installation, another installer, or another game.

## Contents

- `ax_core2.lua`
- `ax_http2.lua`
- `ax_inject2.lua`
- `SHA256SUMS.txt` — file hashes, generated from the actual files

These filenames are deliberate. Do **not** rename them. The original `ax_core.lua`, `ax_http.lua`, and `ax_inject.lua` should remain untouched by these upload commands.

## Prepare the Switch

1. Connect the Switch 2 and computer to the same local network.
2. On the Switch, find its IP address in **System Settings → Internet**.
3. Open Seedsow, launch the existing Artemis Launcher slot, and start its HTTP server (Y in the tested setup).
4. Leave Artemis running during uploads.

In the examples below, replace `192.168.1.100` with **your Switch's IP address**. Do not use someone else's IP address.

## Windows 11 (PowerShell)

1. Extract the ZIP with **Extract All**.
2. Open the extracted folder in File Explorer, click the address bar, type `powershell`, and press Enter.
3. Verify the three `.lua` files are in this folder.
4. Optionally verify the SHA-256 hashes in `SHA256SUMS.txt` using `Get-FileHash .\ax_core2.lua -Algorithm SHA256` (repeat for the other two files).
5. Run each command separately:

```powershell
$ip = "192.168.1.100"
curl.exe --fail-with-body -X PUT --data-binary "@ax_core2.lua" "http://${ip}:8080/save?path=save:/ax_core2.lua"
curl.exe --fail-with-body -X PUT --data-binary "@ax_http2.lua" "http://${ip}:8080/save?path=save:/ax_http2.lua"
curl.exe --fail-with-body -X PUT --data-binary "@ax_inject2.lua" "http://${ip}:8080/save?path=save:/ax_inject2.lua"
```

Each command should confirm `wrote ... bytes to save:/ax_...`. If any upload fails, stop and investigate before restarting.

## macOS (Terminal)

1. Double-click the ZIP to extract it.
2. Open Terminal, type `cd ` (including the space), drag the extracted folder from Finder into Terminal, and press Return.
3. Verify the three `.lua` files are in this folder.
4. Optionally verify hashes using `shasum -a 256 ax_core2.lua ax_http2.lua ax_inject2.lua` and compare with `SHA256SUMS.txt`.
5. Run:

```bash
IP="192.168.1.100"
curl --fail-with-body -X PUT --data-binary @ax_core2.lua "http://${IP}:8080/save?path=save:/ax_core2.lua"
curl --fail-with-body -X PUT --data-binary @ax_http2.lua "http://${IP}:8080/save?path=save:/ax_http2.lua"
curl --fail-with-body -X PUT --data-binary @ax_inject2.lua "http://${IP}:8080/save?path=save:/ax_inject2.lua"
```

Each upload should confirm the file was written. If any fails, stop and investigate before restarting.

## Restart and verify

1. Completely close Seedsow from the Switch HOME menu.
2. Relaunch the existing Artemis slot **without holding minus**.
3. Start the HTTP server again.
4. Confirm the original Lua Editor slot still runs.
5. Optionally test the v2 larger HTTP limit using a harmless 100 KB Lua comment:

**Windows PowerShell:**

```powershell
$ip = "192.168.1.100"
$payload = "--" + ("A" * 100000)
Invoke-RestMethod -Uri "http://${ip}:8080/eval" -Method Post -ContentType "text/plain" -Body $payload -TimeoutSec 15
```

**macOS Terminal** (Python 3 required):

```bash
python3 - <<'PY'
import urllib.request
url = 'http://192.168.1.100:8080/eval'  # CHANGE IP
req = urllib.request.Request(url, data=b'--' + b'A' * 100000, method='POST')
with urllib.request.urlopen(req, timeout=15) as r:
    print('HTTP status:', r.status)
    print(r.read().decode(errors='replace'))
PY
```

`status: ok` for this request is consistent with v2's increased HTTP limit. `HTTP 413` with `body larger than 28672` suggests the older handler is still active. This test alone does not prove every v2 feature is working.

## Recovery

If the launcher fails, completely close Seedsow and relaunch the Artemis slot **while holding minus (-)** to request the original script path, as supported by the tested sidecar installer. If that fails, restore your verified save backup. Do not overwrite or delete the original `ax_*.lua` files.

## Credits and disclaimer

- [alula — Switch Artemis Launcher](https://github.com/alula/switch-artemis-launcher)
- [DialingSpoon527 — Seedsow Lua Editor / Artemis installer](https://github.com/DialingSpoon527/seedsow-lua-editor/tree/artemis-installer)

Unofficial community adaptation, not endorsed by the original developers. Redistribution should respect the upstream projects' licenses; check those licenses before publishing a modified binary/source bundle.
