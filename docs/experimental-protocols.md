# Experimental Protocols

This document summarizes the operating protocol for the microchannel
two-phase-flow loop with acoustic-emission sensing. It is based on the October
2025 internal SOP for the ENRC 3414 flow-loop facility.

## Safety Requirements

All users must complete local training before operating the loop. At minimum:

- Wear required PPE: safety glasses, lab coat, and heat-resistant gloves when
  handling hot components.
- Know the master power switch location and emergency shutoff procedure.
- Keep the inline heater disconnected until liquid flow and temperature are
  stable.
- Follow the startup and shutdown sequences.
- Handle pumps and valves gradually to avoid leaks, sudden pressure changes,
  dry heating, or overheating.
- Handle the working fluid according to laboratory environmental, health, and
  safety guidance.

## Test Loop Overview

The loop circulates deionized water through a closed circuit. Flow is driven by
a D5 solar pump powered independently by a BK Precision 1550 programmable DC
power supply. The liquid passes through a BV1000TRN025B flow-rate sensor, a
Watlow FLC-178 inline heater, a Swagelok SS-SS4-VH low-flow metering valve, and
then the microchannel test section.

The inline heater is controlled through a LabVIEW VI using an Arduino-based PID
interface. The metering valve is located just upstream of the test-section
inlet for fine flow control; the flow sensor is upstream of the valve because
the liquid-phase volumetric flow rate is uniform around the closed loop. Three
needle valves provide modular isolation of sections of the piping network.

After the test section, heated liquid returns to a jacketed glass reservoir
cooled by a TEYU S&A CW-5200 chiller. The reservoir discharges back to the
pump. A filtration loop upstream of the pump removes particulates and helps
preserve fluid purity. The full facility is mounted on a vibration-damped
optical table to reduce mechanical disturbances during acoustic and vibration
measurements.

The piping network uses smooth-bore seamless 304 stainless-steel tubing
(1/4 in. OD, 0.02 in. wall), Swagelok elbows and tees, brass/steel fittings,
and PTFE thread tape where appropriate.

## Test Section

The test section is a three-layer assembly:

- Top layer: transparent polycarbonate cover plate with inlet/outlet ports,
  thermocouple feedthroughs, and pressure-transducer ports.
- Middle layer: insulating PEEK section containing the straight-milled
  aluminum microchannel heat sink.
- Bottom layer: PEEK section holding a ceramic heater directly beneath the heat
  sink.

The ceramic heater is powered by a BK Precision 1685B DC power supply through
PSCS software. This architecture enables simultaneous fluid delivery, heating,
thermal isolation, electrical isolation, optical access, and instrumentation
access.

## Sensing System

Thermal-hydraulic measurements are collected with an NI cDAQ-9178 chassis using
NI 9219 and NI 9239 cards. The loop measures upstream/downstream pressure with
PX2300-1BDI pressure transducers and inlet/outlet fluid temperature with
T-type thermocouples (TJ36-CPSS-116G-3-SB).

Vibration and acoustic measurements are collected with:

- PCB Piezotronics TLD352A56 accelerometer, nominal 10 kHz range.
- PCB Piezotronics 621C40 accelerometer, nominal 30 kHz range.
- Physical Acoustics R3a low-frequency AE sensor, nominal 30 kHz class.
- Physical Acoustics EasyAE DAQ, model 1288-5015.
- Raspberry Pi 4 Model B with Measurement Computing MCC 172 IEPE DAQ HAT for
  accelerometer acquisition.

The AE sensor is mounted on a flat wall of the test-section housing parallel to
the heat sink. A magnetic sticker and MHR15A magnetic hold-down provide the
mounting interface. A compressed sponge-like layer supplies light spring force
to hold the AE sensor against the ferromagnetic surface.

## Pre-Test Setup Checks

Before running the experiment:

1. Confirm the loop is filled, sealed, filtered, and free of visible leaks.
2. Confirm the pump, inline heater, test-section heater, chiller, cDAQ, EasyAE
   DAQ, and accelerometer system are connected.
3. Open NI MAX and verify that `cDAQ1` reports `Present`.
4. Verify the NI cDAQ-9178 chassis and the NI cards in Mod3 and Mod4 are
   detected.
5. Open NI Package Manager and confirm that LabVIEW Run-Time Engine and
   NI-DAQmx are installed, active, and version-compatible.
6. Check Device Manager for the Arduino Uno COM port used by the inline heater
   PID interface.
7. Select the same Arduino COM port in the LabVIEW VI front panel and the
   LabVIEW MakerHub interface.
8. Check Device Manager or NI MAX for the heater PSU communication port used
   by PSCS.
9. Confirm the EasyAE software reports `Hardware OK`.
10. Confirm the chiller is on and reservoir cooling is active before applying
    sustained heater power.

## Inline Heater Operation

The inline heater preheats the inlet liquid to the desired temperature setpoint.
After the Arduino PID interface and LabVIEW VI are configured:

1. Start pump circulation and verify stable flow.
2. Plug the green inline-heater cable into the power source.
3. Enable the inline heater in LabVIEW.
4. Set the inlet temperature setpoint.
5. Increase the setpoint in small increments toward the target temperature.
6. Keep the inline heater duty cycle below 70 percent during approach to the
   target temperature.
7. Wait until the inlet temperature and key loop readings are nearly steady.

## Test-Section Heater Operation

The heater beneath the heat sink is controlled remotely with PSCS software:

1. Open PSCS and use `Add` to establish a remote connection to the BK Precision
   1685B.
2. Enter the COM port and port description identified through NI MAX or Device
   Manager.
3. Open the digital output/control panel.
4. Enter the programmed heater operation in the external timed program panel.
5. Check the `Data Log` tab and confirm the data-log sampling time before the
   test.
6. If the data-log tab must be reset, change or confirm the sampling period
   before starting the test.
7. At the end of heater operation, save the PSCS data log as a CSV file.

For the documented steady-state tests, the PSU is operated in voltage-control
mode with the current limit set to 4.5 A. The voltage may be varied from 0 to
48 V. At 48 V, the corresponding current is about 3.75 A, giving roughly
179.6 W electrical heater input.

## Steady-State Test Procedure

The steady-state procedure is intended to characterize pressure drop and
thermal response over flow-rate and heating conditions.

Target operating envelope from the SOP:

- Inlet temperature: 60 degC for single-phase liquid tests.
- Inlet temperature: 95 degC for tests that may include single-phase and
  two-phase regimes.
- Flow rate: 0.3 to 0.6 L/min for facility safety.
- Heater PSU voltage: 0 to 48 V with current limit set to 4.5 A.
- Pressure-drop recording window: 60 s after stabilization.

Recommended sequence:

1. Start loop circulation and set the desired flow rate with the metering
   valve.
2. Bring the inlet temperature to the target setpoint using the inline heater.
3. Set the test-section heater to the target power or voltage step.
4. Wait for the flow rate, inlet temperature, outlet temperature, pressure
   drop, and base/heater readings to stabilize.
5. Start flow-loop logging in LabVIEW.
6. Start EasyAE logging and waveform streaming.
7. Start or confirm PSCS heater-power logging.
8. Record at least 60 s of steady data at the condition.
9. Increase heater power sequentially and repeat the stabilization and logging
   process.

## Transient Test Procedure

The transient procedure uses automated PSCS power ramping after the inlet
temperature has stabilized.

1. Stabilize the loop at the selected inlet temperature and flow rate.
2. Configure the PSCS external timed program for the desired ramp-up and
   ramp-down power profile.
3. Start LabVIEW logging.
4. Start EasyAE acquisition and trigger waveform streaming.
5. Start the PSCS program and data log.
6. Continue recording through the ramp and return period.
7. Stop the PSCS program, EasyAE logging, and LabVIEW logging after the test is
   complete.

## LabVIEW Logging Procedure

After the inline heater PID setup is complete:

1. Press `Run` in the LabVIEW VI front panel.
2. Enable inline heating only after stable circulation is confirmed.
3. Set the inlet setpoint and approach it gradually.
4. Monitor all displayed readings until the loop is near steady state.
5. Enable `Start/Stop Logging` to record flow-loop data to an `.lvm` file.
6. Press `Start/Stop Logging` again to end the file at the conclusion of the
   test.

## Shutdown

At the end of a test:

1. Stop or ramp down the test-section heater in PSCS.
2. Disable the inline heater and allow the loop to circulate while cooling.
3. Stop EasyAE acquisition and confirm output files are written.
4. Stop LabVIEW logging and save any remaining files.
5. Let the chiller and pump continue until temperatures are safe.
6. Turn off power supplies, DAQ systems, and the chiller according to the lab
   shutdown checklist.
7. Inspect for leaks, unusual deposits, loose mounts, or sensor movement.

## Equipment List

| Component | Manufacturer | Part number/model |
| --- | --- | --- |
| T-type thermocouple | DwyerOmega | TJ36-CPSS-116G-3-SB |
| Pressure transducer | DwyerOmega | PX2300-1BDI |
| NI cDAQ chassis | National Instruments | cDAQ-9178 |
| NI DAQ cards | National Instruments | NI 9219, NI 9239 |
| Flow-rate sensor | DwyerOmega | BV1000TRN025B |
| Pump | US Solar Pumps | D5 Solar Pump |
| Heater power supply | BK Precision | 1685B |
| Pump power supply | BK Precision | 1550 |
| AE DAQ | Physical Acoustics | EasyAE 1288-5015 |
| Acoustic sensor | Physical Acoustics | R3a low-frequency AE sensor |
| Magnetic hold-down | Physical Acoustics | MHR15A |
| Magnetic sticker | SALEX | X002VQXSR9 |
| Single-board computer | Raspberry Pi Foundation | Raspberry Pi 4 Model B |
| DAQ HAT | Measurement Computing | MCC 172 |
| Accelerometer | PCB Piezotronics | TLD352A56 |
| Accelerometer | PCB Piezotronics | 621C40 |
| Inline heater | Watlow | FLC-178 |
| Heat sink | Custom machined | Straight-milled aluminum microchannel |
| External chiller | TEYU S&A | CW-5200 |
| Tubing | McMaster | 89895K722 |
| 90-degree tube elbow | Swagelok | S-400-9 |
| Pneumatic flow-control valve | Zoro Select | 5TUL2 |
| Metering valve | Swagelok | SS-SS4-VH |
| Male NPT connector | Swagelok | S-400-1-4 |
| Tee connector | Swagelok | SS-400-3-4-2 |
| Organic material filter | Pentair | GS-10 |
| Reservoir | Wilmad-LabGlass | LG-8079C-104, 2 L jacketed vessel |
