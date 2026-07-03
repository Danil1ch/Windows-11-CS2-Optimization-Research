# Windows 11 + CS2 Optimization Research

Practical research notes from troubleshooting Counter-Strike 2 microstutters on a fresh Windows 11 installation.

This repository is not a generic "FPS boost" pack. It documents what was tested, what helped, what did not help, and which changes should be treated as risky or optional.

> Russian version: [README_RU.md](README_RU.md)

## What this is about

The system already had high average FPS, but CS2 still felt uneven because of:

- frametime spikes;
- poor 1% and 0.2% lows;
- stutters when Chromium/Electron apps were visible on a second monitor;
- DPC latency peaks in LatencyMon;
- unstable wireless mouse polling.

The useful work came from measuring the problem, changing one area at a time, and rolling back tweaks that made the system worse.

## Test system

| Component | Details |
| --- | --- |
| CPU | AMD Ryzen 7 5700X, PBO +200 MHz, Curve Optimizer -30 all-core |
| GPU | NVIDIA GeForce RTX 3070 Ti |
| RAM | 32 GB DDR4 @ 3733 MHz CL17 |
| Motherboard | MSI MPG B550 Gaming Plus |
| OS | Windows 11 Pro 25H2 / build 26200, fresh install |
| Main monitor | Redmi G24, 180 Hz -> 200 Hz through CRU |
| Second monitor | Acer 100 Hz |
| Mouse | ATTACK SHARK X3 wireless |
| Headset | Logitech G435 wireless |
| Recording | NVIDIA Instant Replay / ShadowPlay, recording to HDD |

## Repository map

- [docs/commands_en.md](docs/commands_en.md) - full English command reference with notes and rollback hints.
- [docs/commands_ru.md](docs/commands_ru.md) - full Russian command reference.
- [registry/disable_mpo.reg](registry/disable_mpo.reg) - MPO/DWM registry tweak used in the investigation.
- [registry/restore_mpo.reg](registry/restore_mpo.reg) - rollback file for the MPO/DWM tweak.

## Tools used

- LatencyMon
- CapFrameX
- .NET Desktop Runtime 9.0 for CapFrameX
- CRU / Custom Resolution Utility
- NVIDIA Profile Inspector
- MSI Utility v3
- MouseTester

MSI Utility v3 checksum used in this research:

```text
c08d7ae2fff3052fd801f6bf33831d08
```

DDU, TestMem5, and Process Lasso were not part of this run, so they are not presented as required steps.

## Results

| Metric | Before | After |
| --- | ---: | ---: |
| Average FPS | ~344.7 | ~345 |
| 1% Low Average | ~77.4 | ~137.8 |
| 0.2% Low / P0.2 | ~42.1 | ~143.8 |
| LatencyMon DPC peak | ~2249 us, dxgkrnl.sys | ~1136 us, nvlddmkm.sys |
| Wireless mouse polling | frequent 5-10 ms spikes | much cleaner after moving the receiver |
| Second-monitor stutter | hard freezes | small FPS drop after MPO fix |

Average FPS barely changed. Smoothness improved because frametime stability, lows, DPC latency, and mouse polling became better.

## What helped most

1. MPO fix for a mixed-refresh dual-monitor setup.
2. MSI Utility v3 high interrupt priority for the GPU and the correct USB controller only.
3. Moving the ATTACK SHARK X3 receiver to the desk with cleaner line of sight.
4. Lowering ShadowPlay / Instant Replay load instead of disabling it completely.
5. Defender exclusions for Steam and game libraries instead of fully disabling Defender.
6. Disabling Realtek LAN power saving features while keeping Interrupt Moderation enabled.
7. Measuring with CapFrameX, LatencyMon, and MouseTester instead of relying on feel alone.

## What did not help

- HAGS Off + BIOS Data Link Feature Exchange made this system worse.
- Random Windows service disabling cleaned the system but did not directly fix CS2.
- The idea that a different Steam account was smoother was not supported by inspected CS2 cloud/config files.

## Safer starting point

If you want the least risky path, start with diagnostics and reversible changes:

1. Measure frametimes with CapFrameX.
2. Check DPC behavior with LatencyMon.
3. Check wireless mouse polling with MouseTester.
4. Apply the MPO fix only if you use mixed-refresh monitors and second-monitor windows cause stutters.
5. Add Defender exclusions for Steam/game folders.
6. Move the wireless mouse receiver away from the PC rear I/O and 2.4 GHz interference.

The full command reference is in [docs/commands_en.md](docs/commands_en.md).

## Warnings

- Do not run every command blindly.
- Do not set every MSI Utility device to High.
- Do not disable Windows Hello, VBS, Defender, UAC, or restore points unless you understand the tradeoff.
- Keep CRU `reset-all.exe` available before monitor overclocking.
- Reboot after registry, driver, MPO, HAGS, service, scheduled task, BIOS, or power-plan changes before judging results.

## CS2 settings used

- 1280x1024, 4:3 stretched
- NVIDIA Reflex: On + Boost
- V-Sync: Off
- FPS limit: 0
- MSAA: 2x
- Shadows: Medium
- Models/textures: Low
- HDR: Performance
- FSR: Off
