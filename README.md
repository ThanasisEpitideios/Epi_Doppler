# Epi_Doppler v1.1

## Overview
**Epi_Doppler v1.1** is a JSFX spatial movement tool for REAPER that simulates Doppler and distance behavior using a moving source around a listener. Movement can be **Linear** or **Custom Path**, and the sound is shaped by distance‑based delay, attenuation, panning, optional air absorption, stereo width, and output gain.

## Changelog
### v1.1
**Behavior changes / workflow**
- Replaced `Loop Enable` with a single `Mode` selector:
	- `Speed (One-Shot)`: run start → end and stop.
	- `Speed (Loop)`: continuous looping using `Style` (`Wrap` / `PingPong`).
	- `Automation (Open)`: `Position` maps 0% → 100% on an open path.
	- `Automation (Loop)`: `Position` maps cyclically (includes implicit last→first segment for seamless looping).
- `Position (%)` drives movement only in Automation modes (Speed modes ignore Position for movement).
- Soft-disables (locks) controls when not applicable to reduce confusion (e.g. `Direction` in Custom Path; `Style` when not in `Speed (Loop)`).
- UI label cleanup: `Direction`, `Style`.
- Expanded `Output Gain (dB)` range (v1 had -12..12 dB; v1.1 uses -24..24 dB).

**Bug fixes**
- Seamless cyclic automation: `Position` envelopes wrapping 100% → 0% no longer cause a teleport/glitch (phase unwrapping).
- Custom Path PingPong: fixed backward traversal so it follows the curve correctly when reversing direction.
- Restart/transport stability:
	- Snap internal state on play/resume to prevent “starting from wrong position then gliding”.
	- Temporary delay snapping after restart/resume to reduce pitch-glide artifacts.
- Restart correctness:
	- Linear restart returns to the correct start based on `Direction`.
	- Custom Path restart snaps to the path start.
- Trigger reliability:
	- Triggers behave like buttons in the UI (auto-reset).
	- Triggers are edge-triggered (0→1) so they do not continuously re-fire if an envelope holds them high.
- Multi-instance safety: removed reliance on shared `gmem` for path storage (prevents cross-instance/path corruption).
- Visualization: improved stopped/paused behavior and display refresh (including a more robust “paused-like” detection).

**Additions / improvements**
- Added `Doppler Amount (%)` to scale Doppler intensity.
- Added `Position (%)` for manual scrub + automation-driven movement in Automation modes.
- Added parameter smoothing where needed to reduce zipper noise and audible artifacts (Position smoothing and delay smoothing).
- Automation mapping quality:
	- `Automation (Open)` uses open mapping (no implicit closing segment).
	- `Automation (Loop)` uses cyclic mapping (includes implicit closing segment for seamless looping).
- GUI/editor improvements:
	- Larger/easier point hit target and clearer point feedback.
	- When stopped/paused, the visual cursor reflects `Position` only in Automation modes (Speed modes show the actual source position).
- Persistence: Custom Path control points are stored in project state and restored on reopen.

## Installation
### Windows
1. In REAPER, go to Options → Show REAPER resource path in explorer/finder.
2. Open the **Effects** folder.
3. Copy `custom_path_doppler.jsfx` (or your JSFX file name) into **Effects**.
4. In REAPER, open the FX browser and click **Re-scan** (or restart REAPER).

### macOS
1. In REAPER, go to Options → Show REAPER resource path in explorer/finder.
2. Open the **Effects** folder.
3. Copy `custom_path_doppler.jsfx` (or your JSFX file name) into **Effects**.
4. In REAPER, open the FX browser and click **Re-scan** (or restart REAPER).

## Signal Flow (Conceptual)
1. **Position** (Linear or Custom Path) → source $(x,y)$
2. **Distance** $d = \sqrt{x^2 + y^2}$
3. **Doppler delay** $\Delta t = d / 340$ seconds
4. **Distance attenuation**
5. **Air Lowpass** (optional)
6. **Angle‑based panning**
7. **Stereo width**
8. **Output gain**

## Controls (UI Order)
| # | Control | Description |
|---|---|---|
| 1 | Speed (km/h) | Speed along the current path. |
| 2 | Movement Mode | `Linear Loop` or `Custom Path`. |
| 3 | Mode | `Speed (One-Shot)`, `Speed (Loop)`, `Automation (Open)`, `Automation (Loop)`. |
| 4 | Direction | Linear direction (`Left to Right` / `Right to Left`). (Linear only) |
| 5 | Style | Loop style for `Speed (Loop)`: `Wrap` / `PingPong`. |
| 6 | Restart | Restarts from the start (Speed modes). Trigger/button (edge-triggered). |
| 7 | Listener Distance Y (m) | Linear mode listener offset on Y. In Custom Path the Y comes from the path. |
| 8 | Canvas Width (m) | World width in meters. Defines limits and visual scale. |
| 9 | Latency Compensation (ms) | Fixed delay added to the Doppler delay. |
| 10 | Air Lowpass | `Off/On` distance‑based low‑pass. |
| 11 | Stereo Width (%) | 0% mono → 100% full stereo. |
| 12 | Output Gain (dB) | Post‑processing gain. |
| 13 | Clear Path | Clears all Custom Path points. |
| 14 | Doppler Amount (%) | Scales Doppler delay amount (0% disables Doppler delay movement effect). |
| 15 | Position (%) | Manual/automatable position along the movement. Drives movement only in Automation modes. |

## Movement Modes
### Movement Mode (geometry)
- **Linear Loop**: the source moves on the X axis with constant Y (`Listener Distance Y`).
- **Custom Path**: draw/edit control points; the curve is smoothed (Catmull‑Rom) and baked for stable playback.

### Mode (how motion is driven)
- **Speed (One-Shot)**: moves from start → end and stops.
- **Speed (Loop)**: moves continuously using `Style` (`Wrap` / `PingPong`).
- **Automation (Open)**: `Position` maps 0% → 100% across the open path (no implicit last→first).
- **Automation (Loop)**: `Position` maps cyclically (includes implicit last→first segment for seamless looping).

## Mouse Editing (Custom Path)
- **Left click**: add point or grab an existing point.
- **Drag**: move a grabbed point or draw new points.
- **Right click**: remove last point.
- Maximum points: 100.
- **Persistence**: path points are saved with the REAPER project and restored on reopen.

## Visualization
- **Edge Frame**: always visible, shows movement limits (±half width).
- **Distance line**: center → source line.
- **Distance label**: live distance in meters (top‑left).
- **Centered scale**: 0 at listener, symmetric tick labels across the frame.

## Technical Notes
- Doppler delay uses $d/340$ seconds and linear interpolation.
- Doppler delay is smoothed to reduce pitch zippering during automation and transport changes.
- Attenuation uses $1/(1 + 0.5d)$ for a stable, mix‑friendly roll‑off.
- Air lowpass uses $f_c = 20000 \cdot e^{-0.008d}$, clamped to 3–20 kHz.
- Buffer length is 4 seconds; very large distances + latency may exceed this.

## Quick Setup
1. Choose **Movement Mode**.
2. Choose **Mode** (Speed or Automation).
3. Set **Speed**, **Canvas Width**, and (Linear) **Listener Distance Y**.
4. For looping, use **Mode = Speed (Loop)** and pick **Style**.
5. For automation, use **Mode = Automation (Open/Loop)** and automate **Position (%)**.
6. Dial **Air Lowpass**, **Stereo Width**, and **Output Gain** to taste.

## Troubleshooting
- **Audio seems late**: reduce `Latency Compensation`.
- **Motion feels too wide**: reduce `Canvas Width`.
- **Too bright at distance**: enable `Air Lowpass`.
- **Need mono**: set `Stereo Width` to 0%.

*Code development was assisted by an AI agent.
