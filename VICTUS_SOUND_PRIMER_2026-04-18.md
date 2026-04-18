# VICTUS Sound Primer - 2026-04-18

## Purpose

This note captures the current Sunshine audio configuration on the HP Victus, the exact audio endpoints visible to Sunshine, and the strongest host-side conclusions from the audio investigation.

## Current Live Sunshine Audio Config

Current `sunshine.conf` values on the Victus:

```text
audio_sink = {0.0.0.00000000}.{8d25a018-5b37-4399-995a-dea2a977e4f0}
virtual_sink = {0.0.0.00000000}.{5dca829e-b6f5-40fe-99b2-3f69530af796}
output_name = {f6352b4e-ca82-5c2f-8077-47cd6444caff}
```

Interpretation:

- `audio_sink` is explicitly pinned to the Realtek speaker path.
- `virtual_sink` is explicitly pinned to Steam Streaming Speakers.
- `output_name` is still pinned to the working ultrawide display.

## Sunshine-Visible Audio Devices

Enumerated with `C:\Program Files\Sunshine\tools\audio-info.exe`:

### 1. Steam Streaming Speakers

- Device ID: `{0.0.0.00000000}.{5dca829e-b6f5-40fe-99b2-3f69530af796}`
- Device name: `Speakers (Steam Streaming Speakers)`
- Adapter name: `Steam Streaming Speakers`
- Device description: `Speakers`
- State: `Active`
- Format: `Stereo`

### 2. Realtek Audio

- Device ID: `{0.0.0.00000000}.{8d25a018-5b37-4399-995a-dea2a977e4f0}`
- Device name: `Speaker (Realtek(R) Audio)`
- Adapter name: `Realtek(R) Audio`
- Device description: `Speaker`
- State: `Active`
- Format: `Stereo`

### 3. OMEN Cam & Voice

- Device ID: `{0.0.0.00000000}.{db6da845-07f9-410f-bae0-56baccc8a864}`
- Device name: `Speakers (OMEN Cam & Voice)`
- Adapter name: `OMEN Cam & Voice`
- Device description: `Speakers`
- State: `Active`
- Format: `Stereo`

### 4. LG SMART WQHD (NVIDIA HDMI Audio)

- Device ID: `{0.0.0.00000000}.{f40e75b1-dda4-4f00-b446-96676840b909}`
- Device name: `LG SMART WQHD (NVIDIA High Definition Audio)`
- Adapter name: `NVIDIA High Definition Audio`
- Device description: `LG SMART WQHD`
- State: `Active`
- Format: `Stereo`

## Current Host-Side Audio Evidence

Recent Sunshine log markers show:

```text
config: 'audio_sink' = {0.0.0.00000000}.{8d25a018-5b37-4399-995a-dea2a977e4f0}
config: 'virtual_sink' = {0.0.0.00000000}.{5dca829e-b6f5-40fe-99b2-3f69530af796}
Virtual audio device will use 16-bit to match default device
Audio mixer format is 32-bit, 48000 Hz, no resampling needed
Audio capture format is [F32 48000 2.0]
Resetting sink to [virtual-Stereo{0.0.0.00000000}.{5dca829e-b6f5-40fe-99b2-3f69530af796}] after default changed
```

Interpretation:

- Sunshine is reading the configured Realtek `audio_sink` and Steam `virtual_sink` correctly.
- Sunshine is successfully initializing audio capture.
- Sunshine is successfully re-pointing the sink to the Steam virtual device when Windows default changes.
- The host-side evidence supports the conclusion that Sunshine audio capture itself is working.

## Manual Local Output Sweep Findings

A local Windows output sweep was run across the main endpoints:

1. Realtek
2. Steam Streaming Speakers
3. LG SMART WQHD
4. OMEN Cam & Voice

Observed user feedback:

- `LG SMART WQHD (NVIDIA High Definition Audio)` produced clear local audio when the monitor/TV was switched to the correct physical input.
- That audio was local monitor audio, not Moonlight remote audio.

This establishes an important distinction:

- `Realtek` = local laptop/host playback path
- `LG SMART WQHD` = local HDMI/monitor playback path
- `Steam Streaming Speakers` = virtual remote streaming path intended for Moonlight

## Most Likely Current State

At the time of this primer:

- game/app audio exists on the host
- Sunshine host audio capture appears to be functioning
- the remaining failure is more likely in the remote playback/rendering path than in raw host-side capture

In other words:

**this does not currently look like a simple Sunshine host audio-capture failure.**

## Practical Guidance

### Known current values

If you need to restore the current Sunshine audio fields:

- `audio_sink` -> `{0.0.0.00000000}.{8d25a018-5b37-4399-995a-dea2a977e4f0}`
- `virtual_sink` -> `{0.0.0.00000000}.{5dca829e-b6f5-40fe-99b2-3f69530af796}`

### What not to confuse

- Hearing audio through `LG SMART WQHD` only proves HDMI/monitor audio works locally.
- Hearing audio through `Realtek` only proves local host audio exists.
- Neither of those by itself proves Moonlight remote audio is working.

### Strongest next diagnostic

The cleanest next discriminator is a cross-client test:

- test the same Sunshine host from a second Moonlight client
- if the second client gets audio, the host path is good and the current client is the likely problem
- if no client gets audio, then revisit Sunshine sink behavior with fresh session logs

## Related Files

See also:

- `KNOWN_GOOD_CONFIG.md`
- `VICTUS_FIX_SUMMARY_2026-04-18.md`
- `STATUS.md`
