# Minecraft Server Status — iCUE Widget

A Corsair iCUE custom widget for the XENEON Edge that shows live
status for a Crafty-managed Minecraft server and lets you
start/stop/restart/backup it without leaving the panel.

This widget is display-only — all the actual polling, protocol
handling, and Crafty API calls happen in a separate local relay
process. See that project's README for setup; this one covers the
widget itself.

## What it shows

- **Status** — online/offline, with a small block-style indicator
  and an uptime counter ("Up 2h 14m")
- **Players** — current/max count, a sparkline of players-online over
  the last hour, and the names of who's online
- **Version and latency**, plus an update-available badge (vanilla
  version checks only — see Limitations)
- **CPU / RAM / world size**, if Crafty is configured
- **Mods** — count and full list (ID + version) for Forge servers,
  pulled from the server's own ping response
- **Recent console line** — the latest line from Crafty's server log
- **Idle nudge** — flags how long the server's been running with zero
  players, so you know it's safe to stop
- **Controls** — Start / Stop / Restart / Backup buttons. Stop and
  Restart require a second tap within 4 seconds to confirm, so a
  stray touch on the panel doesn't take the server down.

## Layout

Built and sized for the **Small** XENEON Edge slot (840×344px):
two columns — server info and controls on the left, players and mod
list on the right — since that slot is wide but short. If you're
using a taller slot (Medium/Large/XL) instead, the fixed sizing here
will need adjusting; it isn't currently responsive across slot sizes.

## Files

```
mc-widget/
├── manifest.json   # widget metadata, device targeting
├── index.html      # all UI + logic (CSS and JS inlined)
└── icon.svg        # preview icon shown in iCUE's widget picker
```

## How it gets its data

The widget polls `http://127.0.0.1:8787/mc-stats` every 10 seconds —
that's the local relay, not Crafty or the Minecraft server directly.
It never talks to Crafty's API itself; button presses just `POST` to
the relay's `/mc-action` endpoint (`{"action": "start"}`, etc.), and
the relay handles authentication and the actual Crafty call.

If the relay isn't running, the widget shows a "Relay Down" status
rather than failing silently.

## Packaging

From inside this folder:
```
icuewidget validate
icuewidget package
```
`validate` checks the manifest and files against iCUE's schema;
`package` produces the installable `.icuewidget` file.

## Limitations (by design, not oversight)

- **Update-available badge** only compares against Mojang's public
  *vanilla* release list. For a modpack, this tells you a new base
  Minecraft version exists — not that new mod versions are available,
  which nothing here can detect.
- **"Backup requested Xm ago"** is this widget's own memory of the
  last time you pressed the button, not Crafty's actual backup
  history (no API exists for the latter). It resets if the relay
  restarts.
- Font sizes and layout are tuned specifically for the Small slot's
  physical size on the XENEON Edge panel, which is a high-DPI but
  physically compact display — sizes that look reasonable in a
  browser can still read as tiny on the actual hardware.
