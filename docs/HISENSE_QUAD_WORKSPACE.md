# Hisense Quad Workspace — Radagast Edition

Status: discovery complete; implementation and Windows validation pending.
Branch: `feature/hisense-quad-workspace`. Base: `master`.

## Goal
Use a single 4K display as four usable 2×2 work areas, each the equivalent *in pixel area* of 1920×1080 when the physical display is 3840×2160. This is window-zone emulation, NOT four independent Windows display devices.

## Confirmed existing functionality (source inspection)
- `ZoneLayoutEditorView.xaml.cs` already supplies a **Quadrants** preset with four (0, 0), (0.5, 0), (0, 0.5), and (0.5, 0.5) zones of size 0.5×0.5 each.
- `ZoneLayoutEditorView.xaml` includes the **Quadrants ⊞** preset button and **Show per-zone taskbar** checkbox.
- `ZoneService` currently calls `SetWindowPos` to snap dragged windows to zone bounds on move-size end. Snapping depends on the modifier setting.
- `ZoneTaskbarWindow` enumerates visible windows and lists a window when its center lies in a zone, updating every 1.5 seconds.
- Zones are fractions of monitor bounds and are saved per monitor ID.

## Limitations to address
1. Maximizing an application through Windows' ordinary maximize button still targets the physical monitor work area, not the saved zone. Need an opt-in, nonintrusive maximize-to-zone behavior with correct restoration.
2. Restoring the user's named workspace/apps after login requires integration with existing window-layout persistence; avoid launching untrusted executable paths.
3. Zone taskbars currently occupy visual space over the bottom of zones. Account for their size when fitting window bounds; ensure correct DPI scaling.
4. Test multi-monitor setups and display reconnect/resolution changes. Avoid affecting monitors without an explicitly enabled quad profile.
5. Fullscreen video, games, protected windows, UAC, elevated apps, and Windows display APIs may bypass window-zone behavior. Do not misrepresent these as genuine display devices.
6. Ensure that any event hooks do not cause loops or override user-initiated resizing; provide a simple opt-out and diagnostic logging.

## First-run validation checklist (without code changes)
1. Install and run an upstream release, or build the fork with .NET 8 SDK on Windows 11.
2. Select the Hisense in Zone Layouts → Monitor.
3. Enable Zone Layouts, optionally Show per-zone taskbar.
4. Select **Quadrants ⊞**, then **Save Layout**.
5. Drag four ordinary app windows into the four zones; verify snapping and zone labels.
6. Test maximize, restore, DPI scale, taskbar visibility, and behavior after restart.
7. Record the observed differences from four true virtual monitors.

## Implementation order
- Phase A: make a named, saved Hisense profile and document baseline behavior.
- Phase B: opt-in maximize-to-zone with recovery and edge-case guards.
- Phase C: named launch/restore workspace presets.
- Phase D: assess whether actual Windows virtual displays are warranted; this requires a separate driver/system architecture and cannot be delivered by ordinary WPF zone snapping.

## Safety
Preserve existing MIT attribution. Keep development changes isolated from `master`, compile and test before merging, and do not install experimental display drivers during the baseline phase.
