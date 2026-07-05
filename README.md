## Project 5: Tank Mixing Control System
## Siemens S7-1200 | TIA Portal V17 | WinCC Advanced HMI

**Author:** Adekunle Francis Adeniyi
**Platform:** SIMATIC S7-1200 CPU 1214C DC/DC/DC
**Software:** TIA Portal V17 | S7-PLCSIM V17 | WinCC Advanced (KTP700 Basic PN)
**Status:** Complete — fully validated in PLCSIM with live HMI runtime

## Overview
A complete industrial batch process control system for tank filling, 
mixing, and discharging — the classic process found in chemical, 
pharmaceutical, and food and beverage plants.

Built as a **finite state machine** with modular block architecture:
**IDLE → FILLING → MIXING → DRAINING → IDLE**

The system features a retentive mix timer that preserves product 
quality through fault interruptions, full alarm handling with 
operator acknowledgement, state-aware fault recovery, and 
Auto/Manual batch modes — all operated from a WinCC Advanced HMI.

## Control Narrative
1. System waits in IDLE
2. Operator presses START → Fill Valve opens (FILLING)
3. Hi Level reached → Fill Valve closes, Mixer starts, 60s timer runs (MIXING)
4. Mix time complete → Mixer stops, Discharge Valve opens (DRAINING)
5. Low Level reached → Discharge closes
6. MANUAL mode → return to IDLE | AUTO mode → loop back to FILLING
7. E-Stop at any time → outputs de-energise, system ALARMED, interrupted state remembered
8. Operator clears fault → RESUME (continue where stopped) or ABORT (full clear to IDLE)

## Program Architecture
| Block | Function |
|---|---|
| OB1 Main | Calls all function blocks every scan |
| FC3 State Machine | All state transitions and Current_State stamping |
| FC1 Process Control | Output control — valves, mixer, lamps |
| FC2 Alarm Handler | TONR mix timer and state-aware resume logic |
| DB1 Process Data | Mix setpoint, elapsed time, cycle counter |

## I/O Schedule
| Address | Tag | Type | Description |
|---|---|---|---|
| %I0.0 | Start_Button | DI | Physical start pushbutton NO |
| %I0.1 | Stop_Button | DI | Physical stop pushbutton |
| %I0.2 | Hi_Level_Sensor | DI | Tank full |
| %I0.3 | Low_Level_Sensor | DI | Tank empty |
| %I0.4 | Emergency_Stop | DI | Fail-safe E-Stop NC |
| %Q0.0 | Fill_Valve | DO | Gravity fill valve |
| %Q0.1 | Discharge_Valve | DO | Gravity discharge valve |
| %Q0.2 | Mixer_Motor | DO | Agitator motor |
| %Q0.3 | Alarm_Lamp | DO | Alarm beacon |
| %Q0.4 | Running_Lamp | DO | Running indicator |
| %M0.0 | State_Idle | Memory | IDLE state bit |
| %M0.1 | State_Filling | Memory | FILLING state bit |
| %M0.2 | State_Mixing | Memory | MIXING state bit |
| %M0.3 | State_Draining | Memory | DRAINING state bit |
| %M0.4 | State_Alarmed | Memory | ALARMED state bit |
| %M1.1 | Alarm_Active | Memory | Latched alarm flag |
| %M1.2 | Resume_Filling | Memory | Remembers interrupted FILLING |
| %M1.3 | Resume_Mixing | Memory | Remembers interrupted MIXING |
| %M1.4 | Resume_Draining | Memory | Remembers interrupted DRAINING |
| %M2.0 | HMI_Start | Memory | HMI start and resume command |
| %M2.1 | HMI_Stop | Memory | HMI stop and abort command |
| %M2.2 | Auto_Mode | Memory | Continuous batch mode selector |
| %MW20 | Current_State | Int | 0 Idle, 1 Filling, 2 Mixing, 3 Draining, 4 Alarmed |

## Key Engineering Features

### TONR Retentive Mix Timer
A standard TON resets on any interruption — ruining batch consistency.
The TONR retains elapsed mix time through an E-Stop and continues on
resume. It resets only on deliberate cycle boundaries. This protects
product quality exactly as required in real batch plants.

### State-Aware Fault Recovery
When E-Stop fires, the active state is captured in a Resume flag
before states clear. On RESUME the system returns to exactly the
interrupted step — mixing continues mixing with retained timer.

### Alarm Acknowledgement — EN ISO 13850
Releasing the E-Stop does NOT clear the alarm or restart the machine.
The alarm persists until a deliberate operator action acknowledges
it. This is the manual reset requirement of the E-Stop standard.

### Auto and Manual Batch Modes
HMI switch selects single-batch or continuous batching with a live
cycle counter incrementing every completed batch.

### WinCC Advanced HMI
- Animated tank with colour per state
- Valve, mixer, and sensor status indicators
- Dynamic state text via text list on Current_State
- Live mix timer elapsed display from TONR through DB to HMI
- Cycle counter, flashing alarm banner
- START, STOP, RESUME, ABORT buttons and AUTO/MANUAL switch

## FAT Test Summary
| Test | Description | Result |
|---|---|---|
| 1 | Power-up defaults to IDLE, outputs off | PASS |
| 2 | Full normal cycle IDLE-FILL-MIX-DRAIN-IDLE | PASS |
| 3 | Mix timer 60s from DB setpoint | PASS |
| 4 | E-Stop in each state — ALARMED, outputs drop | PASS |
| 5 | Resume returns to interrupted state | PASS |
| 6 | TONR retains elapsed time through E-Stop | PASS |
| 7 | ABORT full clear from any condition | PASS |
| 8 | Alarm persists after E-Stop release until acknowledged | PASS |
| 9 | AUTO mode continuous cycling with cycle count | PASS |
| 10 | HMI buttons, animations, displays live against PLCSIM | PASS |

## Lessons Learned
- HMIs cannot write to %I inputs — the input image overwrites them
every scan. HMI commands belong on %M bits merged in parallel with
physical buttons.
- Every state transition must stamp the state register — a single
missed MOVE leaves the display lying while the plant moves on.
- SIM tables and HMIs fight over the same address — one owner per
signal.
- An alarm that clears itself when the fault clears is a safety
failure, not a convenience.

## Industrial Equivalent
| Simulation Component | Industrial Equivalent |
|---|---|
| PLCSIM virtual CPU | S7-1200 PLC in control panel |
| SIM table sensors | Float switches / level transmitters |
| WinCC RT Simulator | KTP700 panel on plant floor |
| HMI Start/Stop bits | Panel-mounted pushbuttons in parallel |
| TONR mix timer | Batch recipe timer in DCS |

## Media
HMI runtime screenshots of all five states, TIA Portal program
structure, and live demo video — see the media folder.

## Author
**Adekunle Adeniyi**
MIET | MNSE | BS 7671
Pursuing CEng & COREN
[LinkedIn Profile](https://www.linkedin.com/in/adekunle-francis-adeniyi-miet-mnse-86a196198)
