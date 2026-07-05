Tank Mixing Control System

Siemens S7-1200 | TIA Portal V17 | WinCC Advanced HMI | Finite State Machine

Author: Adekunle Francis Adeniyi
Date: July 2026
Platform: SIMATIC S7-1200 CPU 1214C DC/DC/DC
Software: TIA Portal V17 | S7-PLCSIM V17 | WinCC Advanced (KTP700 Basic PN)
Status: ✅ Complete — fully validated in PLCSIM with live HMI runtime


Project Overview

A complete industrial batch process control system for tank filling, mixing, and discharging — the classic process found in chemical, pharmaceutical, and food & beverage plants. Built as a finite state machine with a modular block architecture, a retentive mix timer that preserves product quality through fault interruptions, full alarm handling with operator acknowledgement, state-aware fault recovery (resume) logic, and Auto/Manual batch modes — all operated from a WinCC Advanced HMI.


Control Narrative


System waits in IDLE
Operator presses START → Fill Valve opens (FILLING)
Hi Level reached → Fill Valve closes, Mixer starts, 60 s mix timer runs (MIXING)
Mix time complete → Mixer stops, Discharge Valve opens (DRAINING)
Low Level reached → Discharge closes:

MANUAL mode → return to IDLE, await operator
AUTO mode → loop directly back to FILLING (continuous batching, cycle counter increments)



E-Stop at any time → all outputs de-energise, system enters ALARMED, the interrupted state is remembered
Operator clears the fault, then RESUME (continue interrupted state — mix timer continues from where it stopped) or ABORT (full system clear to IDLE)



Architecture

OB1 (Main)
 ├── FC3_State_Machine   — all state transitions (IDLE/FILLING/MIXING/DRAINING/ALARMED)
 ├── FC1_Process_Control — output control (valves, mixer, lamps) driven by state bits
 └── FC2_Alarm_Handler   — TONR mix timer + state-aware resume logic
DB1_Process_Data          — mix setpoint, elapsed time, cycle counter

Design principles applied:


State bits (%M) drive outputs — outputs are never set directly by transitions
SET/RESET latching for state transitions; each transition stamps an Int Current_State (via MOVE) consumed by the HMI text list
HMI writes only to %M memory bits (HMI_Start, HMI_Stop) — never to %I physical inputs — merged in parallel with physical pushbuttons
E-Stop is a physical input only, never operable from the HMI (EN ISO 13850)



I/O Schedule

AddressTagTypeDescription%I0.0Start_ButtonDIPhysical start pushbutton (NO)%I0.1Stop_ButtonDIPhysical stop pushbutton%I0.2Hi_Level_SensorDITank full%I0.3Low_Level_SensorDITank empty%I0.4Emergency_StopDIFail-safe E-Stop (NC, de-energise to safe)%Q0.0Fill_ValveDOGravity fill valve%Q0.1Discharge_ValveDOGravity discharge valve%Q0.2Mixer_MotorDOAgitator motor%Q0.3Alarm_LampDOAlarm beacon%Q0.4Running_LampDORunning indicator%M0.0–0.4State bitsMIDLE / FILLING / MIXING / DRAINING / ALARMED%M1.1Alarm_ActiveMLatched alarm — cleared only by operator action%M1.2–1.4Resume flagsMRemember interrupted state for correct resume%M2.0/2.1HMI_Start / HMI_StopMHMI pushbutton commands%M2.2Auto_ModeMContinuous batch mode selector%MW20Current_StateM (Int)0=Idle 1=Filling 2=Mixing 3=Draining 4=Alarmed


Key Engineering Features

1. TONR retentive mix timer — product quality protection

A standard TON would reset on any interruption, ruining batch consistency. The TONR retains elapsed mix time through an E-Stop and continues on resume. It resets only on deliberate cycle boundaries (IDLE or new FILLING) — a genuine process-quality design decision.

2. State-aware fault recovery

When E-Stop fires, the active state is captured in a Resume flag before states clear. On RESUME, the system returns to exactly the interrupted step — filling continues filling, mixing continues mixing (with retained timer).

3. Alarm acknowledgement per EN ISO 13850 / IEC 62682

Releasing the E-Stop does not clear the alarm or restart the machine. The alarm persists until a deliberate operator action (RESUME/ABORT) acknowledges it — manual reset requirement as per the E-Stop standard.

4. Auto / Manual batch modes

HMI switch selects single-batch (return to IDLE) or continuous batching (loop to FILLING) with a live cycle counter.

5. HMI (WinCC Advanced, KTP700 Basic PN)


Animated tank (colour per state), valve/mixer/sensor indicators
Dynamic state text via text list on Current_State
Live mix-timer elapsed display from TONR.ET → DB → HMI
Cycle counter, alarm banner (flashing), START/STOP/RESUME/ABORT + AUTO/MANUAL switch



FAT Test Summary — all passed ✅

#TestResult1Power-up defaults to IDLE, outputs off✅2Full normal cycle IDLE→FILL→MIX→DRAIN→IDLE✅3Mix timer 60 s from DB setpoint✅4E-Stop in each state → ALARMED, outputs drop✅5Resume returns to interrupted state✅6TONR retains elapsed time through E-Stop✅7ABORT full clear from any condition✅8Alarm persists after E-Stop release until acknowledged✅9AUTO mode continuous cycling + cycle count✅10HMI buttons, animations, displays live against PLCSIM✅


Screenshots & Demo


HMI runtime screenshots (all five states), TIA Portal program structure, and a live demo video — see /media folder.



Lessons Learned


HMIs cannot write to %I inputs — the input image overwrites them every scan. HMI commands belong on %M bits merged in parallel with physical buttons.
Every state transition must stamp the state register — a single missed MOVE leaves the display lying while the plant moves on.
SIM tables and HMIs fight over the same address — one owner per signal.
An alarm that clears itself when the fault clears is a safety failure, not a convenience.



Part of a progressive EC&I portfolio: Arduino embedded projects → LOGO! 8 hardware commissioning → S7-1200 TIA Portal (this project) → Factory I/O 3D simulation (next phase).

Adekunle Francis Adeniyi — MIET | MSc Control & Instrumentation Engineering | BS 7671
