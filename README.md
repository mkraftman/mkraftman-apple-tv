# Apple TV (pyatv 0.18.0 override)

A drop-in **custom-component override** of the built-in Home Assistant `apple_tv`
integration, with the only change being the bundled [`pyatv`](https://github.com/postlund/pyatv)
library bumped to **0.18.0 "Willie"**.

pyatv 0.18.0 includes the Companion fix
[`keep power-state subscription when initial fetch fails`](https://github.com/postlund/pyatv/releases/tag/v0.18.0),
which addresses the `FetchAttentionState` / `exchange_opack` timeout regression seen on
recent tvOS versions, where the Apple TV entity stays "alive" but power state and the
app/source list stop updating and Companion commands time out.

This lets you run the fix ahead of the official Home Assistant release that bundles
pyatv 0.18.0.

## What's in here

- The complete `apple_tv` integration **copied verbatim from Home Assistant Core `2026.6.4`**.
- `manifest.json` modified in exactly two meaningful ways:
  - `requirements`: `pyatv==0.17.0` → `pyatv==0.18.0`
  - added a `version` key (required by HACS for custom integrations)
- Nothing in the Python source has been altered.

## Requirements

- Home Assistant **2026.6.4** (the version this was forked from). See caveats below.
- HACS installed.

## Installation (HACS, custom repository)

1. HACS → ⋮ menu → **Custom repositories**.
2. Add `mkraftman/mkraftman-apple-tv` with category **Integration**.
3. Find **Apple TV (pyatv 0.18.0 override)** in HACS and **Download**.
4. **Restart Home Assistant.** On boot, HA installs `pyatv==0.18.0` into the Core
   container and loads this custom component in place of the built-in one.

You'll see a log line noting a custom integration named `apple_tv` is in use and has
not been tested by Home Assistant — that's expected for any override and is harmless.

You do **not** need to re-add or re-pair the integration; it reuses your existing
config entry and credentials.

## Verifying it took effect

After restart, check Settings → System → Logs, or:

```
grep -i "pyatv" /config/home-assistant.log
```

`pyatv` should report 0.18.0. The Apple TV media_player source/app list and
`power_state` should resume updating.

## Reverting

This is the whole point of keeping it clean:

- **Once HA officially ships pyatv 0.18.0** (or newer), remove this override:
  uninstall it in HACS (or delete `/config/custom_components/apple_tv/`) and restart.
  HA will fall back to the built-in integration.

## Caveats

- **Tied to Core 2026.6.4.** The integration code here is frozen at 2026.6.4. If you
  update Home Assistant to a newer minor (e.g. 2026.7.x) and the built-in `apple_tv`
  integration changed, this frozen copy could lag or break. After any Core update,
  either refresh this override against the new Core tag or remove it.
- **Override precedence.** While installed, this fully replaces the built-in
  `apple_tv`. HA loads exactly one integration per domain.
- **Dependency impact is minimal.** pyatv 0.18.0's runtime pins are lower bounds
  (cryptography ≥44, protobuf ≥6.31, pydantic ≥2.0, zeroconf ≥0.129) already satisfied
  by Core 2026.6; installing it only changes pyatv itself.

## Attribution / License

The `apple_tv` integration is part of [Home Assistant Core](https://github.com/home-assistant/core)
and is licensed under Apache 2.0 (see `LICENSE`). This repository redistributes it
unmodified except for the `manifest.json` changes described above. pyatv is authored by
Pierre Ståhl and licensed MIT.

This is an unofficial, personal override and is not affiliated with or endorsed by
Home Assistant or the pyatv project.
