# Epi_Doppler v1*

## Overview
**Epi_Doppler v1** is a JSFX spatial movement tool for REAPER that simulates Doppler and distance behavior using a moving source around a listener. Motion can be **Linear** or **Custom Path**, and the sound is shaped by distance‑based delay, attenuation, panning, optional air absorption, stereo width, and output gain.

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
| 3 | Loop Enable | `Off` stops at the end. `On` loops. |
| 4 | Linear Direction | `Left to Right` or `Right to Left`. |
| 5 | Linear Loop Style | `Wrap` or `PingPong` (only in Linear + Loop On). |
| 6 | Restart | Restarts movement from the start when Loop is Off. |
| 7 | Listener Distance Y (m) | Linear mode listener offset on Y. In Custom Path the Y comes from the path. |
| 8 | Canvas Width (m) | World width in meters. Defines limits and visual scale. |
| 9 | Latency Compensation (ms) | Fixed delay added to the Doppler delay. |
| 10 | Air Lowpass | `Off/On` distance‑based low‑pass. |
| 11 | Stereo Width (%) | 0% mono → 100% full stereo. |
| 12 | Output Gain (dB) | Post‑processing gain. |
| 13 | Clear Path | Clears all Custom Path points. |

## Movement Modes
### Linear Mode
- The source moves on the X axis with constant Y (`Listener Distance Y`).
- **Loop On**: Wrap or PingPong.
- **Loop Off**: Stops at the end position. Use `Restart` to run again.

### Custom Path
- Create and edit control points in the frame. Points are clamped to the frame.
- The path is smoothed (Catmull‑Rom) and baked for stable playback.
- **Loop Off**: Stops at the final point. Use `Restart` to run again.

## Mouse Editing (Custom Path)
- **Left click**: add point or grab an existing point.
- **Drag**: move a grabbed point or draw new points.
- **Right click**: remove last point.
- Maximum points: 100.

## Visualization
- **Edge Frame**: always visible, shows movement limits (±half width).
- **Distance line**: center → source line.
- **Distance label**: live distance in meters (top‑left).
- **Centered scale**: 0 at listener, symmetric tick labels across the frame.

## Technical Notes
- Doppler delay uses $d/340$ seconds and linear interpolation.
- Attenuation uses $1/(1 + 0.5d)$ for a stable, mix‑friendly roll‑off.
- Air lowpass uses $f_c = 20000 \cdot e^{-0.008d}$, clamped to 3–20 kHz.
- Buffer length is 4 seconds; very large distances + latency may exceed this.

## Quick Setup
1. Choose **Movement Mode**.
2. Set **Speed**, **Canvas Width**, and (Linear) **Listener Distance Y**.
3. Enable **Loop** or stop‑at‑end with **Restart**.
4. Dial **Air Lowpass**, **Stereo Width**, and **Output Gain** to taste.

## Troubleshooting
- **Audio seems late**: reduce `Latency Compensation`.
- **Motion feels too wide**: reduce `Canvas Width`.
- **Too bright at distance**: enable `Air Lowpass`.
- **Need mono**: set `Stereo Width` to 0%.

*Code development was assisted by an AI agent.
