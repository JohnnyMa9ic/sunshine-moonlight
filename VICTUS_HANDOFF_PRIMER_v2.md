# VICTUS HANDOFF PRIMER - Sunshine/Moonlight `-1` Investigation

**Revised:** 2026-04-18 by Codex
**For:** Claude Code running on Victus (Windows, 192.168.5.85)
**Context:** John has already spent significant time on this. Current evidence points strongly to the Windows host path, so this handoff is designed to avoid re-running low-value Mac/network diagnostics and to produce the first truly useful Sunshine-side evidence.

---

## TL;DR

Sunshine on Victus accepts `/launch`, audio starts, then **no video traffic ever arrives** and the session dies with `Connection terminated: -1` on the Mac.

Treat this as a **Victus-side capture or encode path failure unless new host logs prove otherwise**. Do not spend time on firewall, pairing, mDNS, or generic Mac-side theories unless new Windows-side evidence contradicts the current picture.

The first priority is to capture a **real launch-window Sunshine log**. The repo logs committed so far are consistent with polling windows and do not yet show a useful launch failure path.

---

## What Is Already Established

These are already strong enough to de-prioritize:

- Mac Application Firewall was disabled and ruled out.
- Mac could reach `192.168.5.85` with healthy LAN latency.
- Sunshine TCP ports were reachable from the Mac.
- `/serverinfo` returned valid responses from the Mac.
- Moonlight version on Mac was current.
- mDNS discovery worked.
- RTSP session setup succeeded far enough for audio packets to begin.

Interpretation:

- The session is not failing at discovery.
- The session is not failing at pairing.
- The session is not failing at generic host reachability.
- The failure happens after launch negotiation, at or near host-side video initialization.

---

## Exact Failure Signature

Load-bearing Moonlight lines:

```text
Executing request: "https://192.168.5.85:47984/launch?...&appid=881448767&mode=3440x1440x100&..."
Launch response: <?xml ...><sessionUrl0>rtspenc://192.168.5.85:48010</sessionUrl0><gamesession>1</gamesession></root>
Starting RTSP handshake...
Audio port: 48000 / Video port: 47998 / Control port: 47999
Starting video stream... Video stream is 3440x1440x100 (format 0x100) done
Starting audio stream... done
Received first audio packet after 500 ms
IDR frame request sent
Control stream received unexpected disconnect event
SDL Error (0): Connection terminated: -1
SDL Info (0): No video traffic was ever received from the host!
```

Interpretation:

1. `/launch` worked.
2. RTSP negotiation worked.
3. Audio started.
4. The client explicitly requested a keyframe.
5. Sunshine never produced even the first video frame.
6. The control stream then dropped.

That points most strongly to one of these host-side buckets:

- Display capture failed to initialize.
- Sunshine is capturing the wrong output or no active output.
- The encode backend failed before first frame emission.
- Hybrid-GPU routing on the laptop is putting capture and encode on mismatched devices.

---

## Important Repo Context

The commits currently visible in `JohnnyMa9ic/sunshine-moonlight` show Sunshine log snapshots dominated by repeated `/serverinfo` and `/applist` polling. That matches the concern that the repo still lacks a true launch-window Sunshine log with:

- `/launch`
- session startup
- capture selection
- encoder selection
- first warning/error

So the repo already supports this conclusion:

**P0 is still the right first move.**

---

## P0 - Capture A Real Sunshine Launch-Window Log

Current repo logs are not enough. Fix that before deeper theorizing.

Success criteria for the next captured Sunshine log:

- contains `/launch`
- contains session start
- contains capture/output selection
- contains encoder selection
- contains the first warning or error after launch begins

Procedure:

```powershell
# 1. Stop Sunshine completely
Stop-Service SunshineService -ErrorAction SilentlyContinue
Stop-Process -Name sunshine -Force -ErrorAction SilentlyContinue
Start-Sleep -Seconds 2

# 2. Locate and rotate the active Sunshine log
$logPath = "$env:WINDIR\Temp\sunshine.log"
if (-not (Test-Path $logPath)) { $logPath = "$env:ProgramData\Sunshine\sunshine.log" }
if (Test-Path $logPath) {
  Move-Item $logPath "${logPath}.$(Get-Date -Format 'yyyyMMdd-HHmmss').bak"
}

# 3. Ensure Sunshine is set to Debug logging in the Web UI
# Configuration > Advanced > Log Level = Debug

# 4. Start Sunshine again
Start-Service SunshineService -ErrorAction SilentlyContinue

# 5. Confirm idle state before retry
curl.exe -s http://127.0.0.1:47989/serverinfo | Select-String "state|currentgame"
```

After the Mac retries, immediately preserve the fresh log covering the failed launch and commit that window, not a later polling snapshot.

---

## P1 - Cheapest High-Value Fixes In Order

### P1a. Force a conservative stream profile

Use these together for the next retry:

- Windows HDR: Off
- Moonlight/Sunshine codec target: H.264
- Resolution/frame rate: 1080p60
- SDR / 8-bit desktop path

Reason:

The failed request was for an aggressive combination: `3440x1440 @ 100 fps`, `HEVC Main 10`, HDR. That stacks multiple stressors at once. First isolate whether the break is tied to HDR/10-bit/HEVC/high-refresh rather than generic streaming failure.

Important distinction:

- `H.264` vs `HEVC` is the **codec**
- `NVENC` / `AMF` / `software` is the **encoder backend**

Do not conflate those in testing.

### P1b. Check Victus hybrid-GPU routing early

Because this is a laptop, move this near the top:

- Windows Settings > System > Display > Graphics
- Set `sunshine.exe` to **High performance**
- If NVIDIA is present, make sure Sunshine is not stuck on the integrated GPU while trying to encode or capture from another device path

If Sunshine config exposes capture backend:

- Prefer trying `capture = wgc` if current config is `ddx`

This is one of the most plausible missing checks for a Victus-class machine.

### P1c. Check active output / login state / primary display

Verify all of the following:

- Victus is unlocked and at an active desktop
- display is awake
- primary display is real and active
- Sunshine `output_name` is not pointing at a stale or removed output

### P1d. Check encoder backend availability

Close likely encoder consumers before trusting tooling:

- OBS / Streamlabs
- Discord screen share
- NVIDIA Overlay / ShadowPlay
- AMD ReLive equivalents

`nvidia-smi` can be supportive, but it is **not strong proof** that NVENC is free. Do not over-trust `--query-compute-apps` as a definitive encoder-session test.

### P1e. Try software encoder only if conservative H.264 SDR still fails

If the stream still dies after `1080p60 + H.264 + SDR + HDR off`, then try switching encoder backend to software temporarily.

That test answers a different question:

- If software works and hardware does not, the problem is likely encoder-backend or GPU-path specific.

### P1f. Update driver if still stuck

Check GPU status and update to the latest stable vendor driver if needed:

```powershell
Get-CimInstance Win32_VideoController | Select-Object Name, DriverVersion, DriverDate, Status
```

---

## P2 - Audio Regression Check

Do not ignore this entirely. The repo includes a Sunshine crash-log commit explicitly described as:

`audio output device failure during Moonlight session`

That means audio was a real observed failure at least once. It is probably **not the current primary theory** because the Mac later received audio packets, but it should be treated as a prior failure mode that may have regressed or coexisted with the video issue.

Check whether the configured audio sink still exists and is stable.

Persistent fix preference:

- Steam Streaming Speakers
- or VB-Cable

as a sink that remains present even when physical devices change.

---

## P3 - Coordination Protocol

Use the repo as a bridge, but keep it simple:

- `logs/moonlight_YYYY-MM-DD_HHMM.log` for Mac-side stderr
- `logs/sunshine_YYYY-MM-DD_HHMM_launch_window.log` for the fresh host log
- `READY_FOR_MAC_RETRY.txt` as the retry signal
- `STATUS.md` as append-only event notes

If a new Sunshine log does not include launch-window lines, it is not useful enough yet.

---

## Pass Criteria

1. Desktop launch succeeds and holds for at least 60 seconds.
2. A second consecutive launch also succeeds.
3. Quit, wait 30 seconds, relaunch, and confirm it still works.
4. Sunshine returns to idle cleanly and does not remain wedged in `SUNSHINE_SERVER_BUSY`.

---

## Starting Order

1. Capture a true launch-window Sunshine log.
2. Retry with `1080p60 + H.264 + SDR + HDR off`.
3. Check hybrid GPU routing and capture backend.
4. Check active display/output targeting.
5. Only then branch into hardware encoder contention, software encoder fallback, and driver refresh.

Everything above is aimed at turning this from a vague `-1` disconnect into a concrete host-side failure mode with evidence attached.
