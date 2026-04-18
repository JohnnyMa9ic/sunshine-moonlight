# Known Good Config

## Host

- Machine: HP Victus
- Sunshine install path: `C:\Program Files\Sunshine`
- Sunshine version: `v2025.924.154138`
- Launch mode: installed shortcut / `--shortcut`
- Service state after clean reinstall: `SunshineService` running

## Known Good Display Target

Set `config.output_name_windows` in the Sunshine web UI to:

```text
{f6352b4e-ca82-5c2f-8077-47cd6444caff}
```

This corresponds to:

- `display_name`: `\\.\\DISPLAY6`
- `friendly_name`: `LG SMART WQHD`
- resolution: `3440x1440`

## Important Working Assumptions

- Leave `Adapter Name` blank
- Leave advanced display device configuration at the clean default unless a new problem appears
- Do not restore old config from the previous broken `E:\Sunshine` install
- Do not reintroduce troubleshooting-era tweaks unless needed

## Recommended First Test Profile

When validating from a fresh state, start with:

- `1080p60`
- `H.264`
- `SDR`

After baseline success, move to the ultrawide target.

## What To Preserve

If Sunshine is working:

- avoid uninstalling or reinstalling unnecessarily
- avoid changing multiple settings at once
- record any new tweak in `STATUS.md`

## Related Audit Notes

See also:

- `VICTUS_FIX_SUMMARY_2026-04-18.md`
- `STATUS.md`
