# Mkraftman Apple TV (extended)

A custom-component fork of the built-in Home Assistant `apple_tv` integration that
adds **programmatic user-account switching** alongside an updated pyatv bundle.

Two things on top of the stock integration:

1. **pyatv bumped to 0.18.0 "Willie"**, which resolves the Companion
   `FetchAttentionState` / `exchange_opack` timeout regression seen on recent tvOS
   versions, where the entity stays "alive" but power state and the source/app list
   stop updating and Companion commands time out.
2. **Two new services** that expose pyatv's `user_accounts` interface:
   - `apple_tv.list_user_accounts` — returns the device's user accounts (name + identifier).
   - `apple_tv.switch_user_account` — switches the active user by identifier.

The three keyboard services from the upstream integration
(`set_keyboard_text`, `append_keyboard_text`, `clear_keyboard_text`) are preserved
unchanged.

## Services

### `apple_tv.list_user_accounts`

Returns the list of user accounts the Apple TV supports switching between.

| Field             | Required | Description                                   |
| ----------------- | -------- | --------------------------------------------- |
| `config_entry_id` | yes      | The Apple TV config entry to query.           |

Returns a response (`supports_response: only`):

```yaml
accounts:
  - name: "Media Kraftman"
    identifier: "F43114AD-342D-4EF2-A97C-8C2F95D323E9"
  - name: "Michael Kraftman"
    identifier: "E04B0E83-0536-4832-BBCA-FD40160A221D"
```

Example call:

```yaml
action: apple_tv.list_user_accounts
data:
  config_entry_id: "01KS8JZDGF09SNRW7H6DRPVARC"
response_variable: accounts_response
```

### `apple_tv.switch_user_account`

Switches the active user account on the Apple TV.

| Field             | Required | Description                                                  |
| ----------------- | -------- | ------------------------------------------------------------ |
| `config_entry_id` | yes      | The Apple TV config entry to act on.                         |
| `account_id`      | yes      | The account identifier from `list_user_accounts`.            |

Example call:

```yaml
action: apple_tv.switch_user_account
data:
  config_entry_id: "01KS8JZDGF09SNRW7H6DRPVARC"
  account_id: "F43114AD-342D-4EF2-A97C-8C2F95D323E9"
```

Both services raise `user_accounts_not_available` if the device or tvOS version
doesn't support account switching, and `user_accounts_error` if the underlying
Companion command fails.

## What's in here

- The complete `apple_tv` integration copied from Home Assistant Core `2026.6.4`.
- `manifest.json`: `pyatv` requirement bumped from `0.17.0` → `0.18.0`; HACS
  `version` set.
- `services.py`: two additional async handlers wired to
  `atv.user_accounts.account_list()` and `atv.user_accounts.switch_account()`,
  with `NotSupportedError` / `ProtocolError` mapped to the appropriate HA exception
  classes.
- `services.yaml`: schemas for the two new services.
- `const.py`: `ATTR_ACCOUNT_ID` constant added.
- `strings.json`: descriptions and translation keys for the new services and their
  errors.

Nothing else in the upstream Python source has been altered.

## Requirements

- Home Assistant **2026.6.4** or a compatible release (see *Caveats*).
- HACS installed.

## Installation (HACS, custom repository)

1. HACS → ⋮ menu → **Custom repositories**.
2. Add `mkraftman/mkraftman-apple-tv` with category **Integration**.
3. Find **Mkraftman Apple TV (extended)** in HACS and **Download**.
4. **Restart Home Assistant.** On boot, HA installs `pyatv==0.18.0` into the Core
   container and loads this custom component in place of the built-in one.

You'll see a log line noting that a custom integration named `apple_tv` is in use
and has not been tested by Home Assistant — that's expected for any override and is
harmless.

You do **not** need to re-add or re-pair the integration; it reuses your existing
config entry and credentials.

## Verifying it took effect

After restart, check Settings → System → Logs, or:

```
grep -i "pyatv" /config/home-assistant.log
```

`pyatv` should report 0.18.0. The Apple TV `media_player` `source_list` and
`power_state` should resume updating, and the two new services should appear under
**Developer Tools → Actions** in the `apple_tv` domain.

To do a quick functional check, call `apple_tv.list_user_accounts` (with response
enabled) from Developer Tools → Actions — you should see the accounts returned
inline.

## Caveats

- **Tied to Core 2026.6.4.** The base integration code is frozen at 2026.6.4. If
  Home Assistant updates and the upstream `apple_tv` integration changes, this fork
  could lag or break. After any Core upgrade, refresh this fork against the new
  Core tag, preserving the user-account additions.
- **Override precedence.** While installed, this fully replaces the built-in
  `apple_tv` integration. Home Assistant loads exactly one integration per domain.
- **Removal trade-off.** Removing this fork (for example once Core eventually ships
  pyatv 0.18.0 itself) means losing the `list_user_accounts` and
  `switch_user_account` services, since those are not in the stock integration. The
  pyatv bump becomes redundant in that case; the user-account services do not.
- **Dependency impact is minimal.** pyatv 0.18.0's runtime pins are lower bounds
  (cryptography ≥44, protobuf ≥6.31, pydantic ≥2.0, zeroconf ≥0.129) already
  satisfied by Core 2026.6; installing it only changes pyatv itself.

## Attribution / License

The `apple_tv` integration is part of
[Home Assistant Core](https://github.com/home-assistant/core) and is licensed
under Apache 2.0 (see `LICENSE`). This repository redistributes it with the
modifications described above. pyatv is authored by Pierre Ståhl and licensed MIT.

This is an unofficial, personal fork and is not affiliated with or endorsed by
Home Assistant or the pyatv project.
