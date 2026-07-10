# High Performance HMI Rework (ISA-101)

**Video demonstration (after):** https://www.youtube.com/watch?v=7RuPBUB_dJ0
**Original version (before):** https://youtu.be/88LiqQwhCOU

## Objectives

Following peer feedback on the original HMI — colour used decoratively rather than by exception — the Main_Overview screen was reworked to High Performance HMI principles per ISA-101, and a dedicated alarm support screen was added. The PLC program was not modified; all changes are HMI-layer only, preserving the validated control logic and FAT results.

## Design Philosophy Applied

- **Colour by exception:** on a healthy screen, the only colour present is bold dark blue live data. Red appears exclusively for alarm conditions.
- **Grey-by-default:** screen background set to light grey; equipment rendered in neutral greys.
- **2D vessels:** tank drawn with a thin black outline and uniform interior — no gradients, no animated internals.
- **Hollow/solid state grammar:** valves, sensors, and mixer are hollow (white fill, grey border) when inactive and solid mid-grey when active. One consistent visual language across all field devices.
- **Text discipline:** static labels in plain dark grey on transparent backgrounds; mixed case for readability with UPPERCASE reserved for equipment tags (HI/LO); live data in bold dark blue; no leading zeros.

## Before / After Change Map

| Element | Before | After |
|---|---|---|
| Screen background | Default template grey | Gray 3 (222,223,222) |
| Tank level | Blue shades per state, red on alarm | Grey all healthy states, red + flash on ALARMED only |
| Fill/Discharge valves | Grey → green when open | Hollow → solid mid-grey when open |
| HI/LO sensors | Grey → yellow when made | Hollow → solid mid-grey when made |
| Mixer | Grey → blue when running | Hollow → solid mid-grey when running |
| Buttons (Start/Stop/Resume/Abort) | Green/red/cyan/orange | Uniform grey, black text; red reserved for alarms |
| RUNNING lamp | Permanent green lamp | Deleted — redundant with state text |
| Static text | White on dark blue boxes | Dark grey, transparent background |
| Live data (State/Mix Time/Cycles) | Plain black | Bold dark blue on white field |
| Alarm banner | Red flashing banner | Unchanged — now the only colour on a healthy screen |

## Display Hierarchy

The reworked HMI implements Level 2 (Process Unit Control) and Level 4 (Process Unit Support) of the ISA-101 display hierarchy. Levels 1 and 3 are not applicable to a single-unit system at this scale; a mixer detail screen would be the natural L3 candidate.

| Screen | ISA-101 Level | Function |
|---|---|---|
| Level 2 (Process Unit Control) | L2 | Operator home: full unit awareness plus Start/Stop/Resume/Abort and Auto/Manual control |
| Level 4 (Process Unit Support) | L4 | Alarm history via Alarm view on the alarm buffer, newest first, with time and date |

Navigation is by dedicated grey DIAG/OVERVIEW buttons (single Click → ActivateScreen), positioned bottom-right on both screens, clear of the process control cluster.

## Alarm System

- **Discrete alarm:** "EMERGENCY STOP ACTIVATED - Clear E-Stop and press RESUME", class Errors.
- **Trigger workaround:** WinCC discrete alarms cannot trigger on a Bool tag. A Word tag (`Alarm_Trigger_Word`, %MW0, read-only overlay) windows the memory area so `Alarm_Active` (%M1.1) maps to trigger bit 1. Single-owner signal discipline is preserved — the word is never written.
- **Annunciation:** the Global screen system alarm window pops on alarm arrival with the full operator instruction; the alarm indicator is configured to appear by exception and disappears at zero active alarms.
- **History:** the L4 Alarm view reads the panel alarm buffer (FIFO), showing incoming/outgoing events with timestamps, newest on top.

## Testing

The full eight-point acceptance sequence was re-run after the rework: IDLE grey check, Start→Filling, Hi→Mixing, Draining, cycle count increment, E-Stop trip with resume and retained mix time (TONR), Abort action, and Auto loop — all passed. Alarm buffer verified with repeated E-Stop trips producing a stacked, timestamped history.

## Known Limitations and Next Steps

- **Volatile alarm history:** the KTP700 Basic alarm buffer (256 entries, FIFO) does not survive a power cycle. Persistent alarm logging requires Comfort-class hardware or SCADA.
- **Panel colour palette:** Basic Panels round custom RGB values to a fixed palette; specified values are documented as "nearest palette value".
- **Abort semantics (roadmap):** ABORT currently shares the stop signal and behaves as a second STOP. Proper ISA-88 abort semantics require a dedicated abort path — forced transition to DRAINING, cleared resume flags and mix timer, and a gated cycle counter so aborted batches are not counted as completed. Planned as the next PLC-side revision with re-FAT.

## Lessons Learned

- A stale incremental compile can report phantom faults ("process tag is missing" on an intact object). Force **Compile → Software (rebuild all)** before diagnosing further.
- Never edit or delete tag table rows without checking **Cross-references** first — deleting a navigation tag silently broke the screen template and two event functions.
- Tahoma on Basic Panels does not embed all glyphs (e.g. ⚠); alarm text should use plain characters.
- Basic Panel colour handling rounds RGB input to the device palette; specify targets, record actuals.

## Acknowledgement

This rework was prompted by direct, constructive feedback from a practising controls engineer on the original published version — a reminder that publishing work invites the review that improves it.
