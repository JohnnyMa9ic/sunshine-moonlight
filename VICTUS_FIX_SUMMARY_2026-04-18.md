# VICTUS Fix Summary - 2026-04-18

## Outcome

Sunshine streaming from the HP Victus to Moonlight was restored, including a successful connection after explicitly targeting the ultrawide display.

## Final Working State

- One clean Sunshine install only
- Installed path: `C:\Program Files\Sunshine`
- Stable version used: `v2025.924.154138`
- Sunshine launched successfully using the installed shortcut path (`--shortcut` behavior)
- Sunshine web UI login succeeded
- Moonlight connection succeeded after setting the ultrawide as the capture target

## Final Ultrawide Setting

In the Sunshine web UI, `config.output_name_windows` was set to:

```text
{f6352b4e-ca82-5c2f-8077-47cd6444caff}
```

That device ID corresponded to:

- `display_name`: `\\.\\DISPLAY6`
- `friendly_name`: `LG SMART WQHD`
- resolution: `3440x1440`

## What Was Wrong

The host had a bad Sunshine state with multiple conflicting installs and unstable runtime behavior.

Confirmed on-host during investigation:

- Sunshine existed in two locations:
  - `E:\Sunshine`
  - `C:\Program Files\Sunshine`
- Windows showed two registered Sunshine uninstall entries
- `winget list` also showed Sunshine twice
- Earlier debugging was being performed against the `E:\Sunshine` copy while a second full install still existed in `Program Files`
- The old host repeatedly failed with symptoms such as:
  - `Failed to locate an output device`
  - `Failed to sync desktop to thread [0x000000AA]`
  - session starts with audio but no video ever arriving

## Failed/Explored Paths Before Final Fix

The following were tested on the broken host state and did not produce a stable fix:

- explicit `output_name = \\.\\DISPLAY6`
- explicit `output_name = \\.\\DISPLAY1`
- removing `output_name`
- `capture = wgc`
- `dd_configuration_option = disabled`
- Windows high-performance GPU preference for Sunshine executables
- targeting the ultrawide by stable device ID on the broken install

These steps were useful diagnostically, but the root issue was not resolved until the install state itself was cleaned up.

## Clean Reinstall Path That Worked

1. Identified the duplicate installs and uninstall registrations.
2. Unstuck the hanging uninstallers by killing their orphaned helper `cmd`/`powershell` processes.
3. Removed both conflicting Sunshine installs.
4. Downloaded the latest stable Windows AMD64 installer directly from the official GitHub release.
5. Reinstalled a single clean copy.
6. Launched Sunshine the supported way.
7. Confirmed clean host health (`SUNSHINE_SERVER_FREE`).
8. Paired/logged in again through the Sunshine web UI.
9. Set `config.output_name_windows` to the ultrawide device ID.
10. Re-tested Moonlight successfully.

## Important Operational Notes

- The reinstall reset pairing/host identity, so re-login/re-pairing was expected.
- On the clean stable install, Sunshine came back as a Windows service (`SunshineService`) running as `LocalSystem`.
- The clean baseline initially worked on the default `1920x1080` AMD-side display path.
- The ultrawide only became the active working target after explicitly setting `config.output_name_windows` in the web UI.

## Best Current Recommendation

Keep the current working state as the baseline.

Do not reapply old troubleshooting-era tweaks unless a new problem appears. In particular, avoid blindly restoring prior manual config from the broken `E:\Sunshine` install.

If future tuning is needed, change one variable at a time from this known-good baseline.
