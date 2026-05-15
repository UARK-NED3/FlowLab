# Multimodal Data Collection

FlowLab experiments combine thermal-hydraulic, electrical, acoustic-emission,
and vibration data streams. Each system writes its own native files, so every
test should use a shared test ID and a written acquisition sequence.

## Data Streams

| Stream | Hardware/software | Typical output |
| --- | --- | --- |
| Flow loop | LabVIEW VI with NI cDAQ-9178, NI 9219, NI 9239 | `.lvm` |
| Heater power | BK Precision 1685B through PSCS | `.csv` |
| AE hit data | EasyAE / AEwin64 | `.DTA` converted to hit-data `.TXT` |
| AE time data | EasyAE / AEwin64 | `.DTA` converted to time-data `.TXT` |
| AE waveforms | EasyAE waveform streaming | `.WFS` converted to per-hit `.csv` files |
| Accelerometers | Raspberry Pi 4 + MCC 172, or LabVIEW workflow | `.csv`, `.lvm`, or notebook-specific input |

Use one test identifier across all files. Existing example names follow this
pattern:

```text
FL-71-flow-loop.lvm
FL-71-PSCS-power-supply.csv
FL-71_mistras-ae-primary-data.TXT
FL-71-mistras-ae-time-driven-data.TXT
FL-102-raspberryPi-accelerometer.csv
```

## Acquisition Order

The SOP uses the following start order:

```text
Flow Loop LabVIEW -> EasyAE DAQ -> Heater PSU (PSCS)
```

This order ensures that baseline flow-loop and AE measurements are already
recording when the heater-power program begins. For transient power ramps,
include a short pre-ramp baseline and post-ramp cooldown in every data stream
when possible.

## LabVIEW Flow-Loop Data

The LabVIEW VI records loop-level thermal-hydraulic data to `.lvm` files. It is
the primary source for:

- Flow-meter pulse period or flow rate.
- Inlet and outlet fluid temperatures.
- Upstream and downstream pressures.
- Pressure drop across the test section.
- Inline-heater state, inlet setpoint, and relevant control variables when
  included in the VI output.

Before a test, verify cDAQ presence in NI MAX and confirm LabVIEW Run-Time
Engine and NI-DAQmx compatibility. Start logging only after the loop is near
the intended condition unless the pre-transient approach period is part of the
experiment.

## Heater PSU Data

The BK Precision 1685B heater PSU is operated through PSCS. The PSCS data log
should be saved as CSV after every test. Record or preserve:

- Sampling period used by PSCS.
- Voltage and current channels.
- Programmed ramp or step sequence.
- Start time relative to the manual acquisition sequence.

The analysis workflow converts logged voltage and current to instantaneous
heater input power and aligns it to the LabVIEW flow-loop timeline.

## EasyAE Acquisition

Use `AEwin64 for EasyAE` for AE data acquisition.

Acquisition sequence:

1. Open the pre-designed `.lay64` layout.
2. Confirm the software reports `Hardware OK`.
3. Press `Acquire`.
4. Choose the `.DTA` data-storage file name.
5. Press `Start` to begin logging. `Test Active` indicates that `.DTA` logging
   has started.
6. With manual trigger enabled, press `Trigger Wave Stream` to create and start
   the `.WFS` waveform-streaming file.
7. At the end of the test, press `Stop/Abort`.
8. Confirm that one `.DTA` and one `.WFS` file were saved for the test.

### Hit And Time Data Export

For hit-based and time-based ASCII outputs:

1. Navigate to `Post Analysis > ASCII Output > Line Display Setup > Select
   Messages to Display`.
2. Disable the other message categories and enable `Hit Data Display`.
3. Select the input `.DTA` file.
4. Save the output `.TXT` file using a descriptive name such as `Hit`.
5. Repeat the export with `Time Data Display` enabled to create the time-data
   `.TXT` file.

### Waveform Streaming Export

For waveform-streaming ASCII output:

1. Navigate to `Post Analysis > ASCII Output > Waveform Streaming ASCII Output`.
2. Select the `.WFS` file corresponding to the test.
3. Choose the folder where waveform CSV files will be written.
4. Select `Time of sample relative to trigger (T=0)` as the output time option.
5. Wait for EasyAE to finish generating all waveform CSV files.

## Accelerometer Acquisition

Accelerometer data may be collected with PCB Piezotronics sensors through the
Raspberry Pi 4/MCC 172 IEPE workflow or through the LabVIEW accelerometer
workflow. Record the following metadata for each accelerometer file:

- Sensor model and mounting location.
- Sampling rate.
- Channel mapping.
- Units and sensitivity/calibration.
- Start/stop relationship to LabVIEW, EasyAE, and PSCS.
- Any anti-aliasing, filtering, or gain settings applied during acquisition.

## Synchronization Metadata

Each test should include a lightweight run log with:

- Test ID.
- Date and operator.
- Inlet temperature target.
- Flow-rate target.
- Heater power or voltage program.
- LabVIEW file name and logging start/stop notes.
- PSCS file name and sampling period.
- EasyAE `.DTA`, `.WFS`, hit-data, time-data, and waveform-output folder names.
- Accelerometer file name and sampling rate.
- Any observed disturbances, leaks, sensor remounting, or acoustic contact
  changes.

## Raw Data Preservation

Keep raw files unchanged. Write processed outputs to a separate folder and
record processing settings, including alignment method, time offsets, filters,
window lengths, and thresholds. For repository examples, prefer small,
representative, non-sensitive files.
