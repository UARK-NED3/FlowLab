# FlowLab

FlowLab is a research toolkit for single-phase and two-phase liquid-cooling
experiments. It combines operating protocols, multimodal data collection
guidance, example files, and analysis notebooks for flow-loop, heater-power,
acoustic-emission, and accelerometer measurements.

The current repository content is organized around the microchannel
two-phase-flow loop with acoustic-emission (AE) sensing documented in the
October 2025 internal SOP, "Microchannel Two-phase Flow Loop with Acoustic
Emission (AE) Sensing."

## Repository Structure

```text
FlowLab/
  README.md
  docs/
    experimental-protocols.md
    multimodal-data-collection.md
    data-analysis.md
  src/
    combined-flow-loop-ae/
      Flow_loop_analysis_notebook2.ipynb
      example-files/
    accelerometer-labview/
      Labview_Accelerometer_CWT_Analysis.ipynb
      example-files/
    accelerometer-raspberry-pi/
      RaspberryPi_Accelerometer_Analysis.ipynb
      RaspberryPi_Accelerometer_Analysis_Flow_loop.ipynb
      example-files/
```

## Experimental System

The facility is a closed-loop deionized-water test loop for controlled
thermal-fluid experiments. A D5 solar pump circulates liquid through a flow
meter, an inline preheater, a metering valve, and a three-layer microchannel
test section. The test section uses a transparent polycarbonate cover plate,
an insulating PEEK section containing a straight-milled aluminum microchannel
heat sink, and a bottom PEEK layer holding a ceramic heater powered by a
separate DC supply.

The loop supports simultaneous measurement of:

- Thermal-hydraulic signals from NI cDAQ/LabVIEW: flow rate, inlet/outlet
  temperatures, pressure transducers, and control-state variables.
- Heater electrical signals from the PSCS-controlled BK Precision 1685B power
  supply.
- Hit-based, time-based, and waveform-streaming AE signals from the EasyAE DAQ.
- Accelerometer signals from PCB Piezotronics sensors acquired through a
  Raspberry Pi 4 and MCC 172 IEPE DAQ HAT, or through LabVIEW workflows.

## Documentation

- [Experimental protocols](docs/experimental-protocols.md) covers safety,
  hardware layout, setup checks, startup, steady-state tests, transient tests,
  and shutdown.
- [Multimodal data collection](docs/multimodal-data-collection.md) describes
  the LabVIEW, PSCS, EasyAE, and accelerometer data streams, file products,
  naming guidance, and synchronization order.
- [Data analysis](docs/data-analysis.md) summarizes flow-rate conversion,
  heater-power alignment, heat-transfer metrics, pressure-drop spectrograms,
  and acoustic-emission feature definitions.

## Existing Analysis Notebooks

- `src/combined-flow-loop-ae/Flow_loop_analysis_notebook2.ipynb`:
  combined flow-loop, heater-power, and AE analysis using the example files in
  `src/combined-flow-loop-ae/example-files/`.
- `src/accelerometer-labview/Labview_Accelerometer_CWT_Analysis.ipynb`:
  continuous wavelet transform analysis for LabVIEW accelerometer data.
- `src/accelerometer-raspberry-pi/RaspberryPi_Accelerometer_Analysis.ipynb`:
  accelerometer analysis for Raspberry Pi/MCC 172 data.
- `src/accelerometer-raspberry-pi/RaspberryPi_Accelerometer_Analysis_Flow_loop.ipynb`:
  accelerometer workflow paired with flow-loop data.

## Data Handling Notes

Raw experimental files can be large and may contain facility-specific metadata.
Keep raw files outside git unless they are intentionally small, anonymized
examples. Preserve raw inputs as read-only data, write cleaned or synchronized
outputs separately, and record the test ID, operating condition, data source,
sampling period, and processing settings used for each derived result.

## Safety Note

The flow loop integrates liquid circulation, electrical heating, energized
power supplies, and hot surfaces. Users must follow the laboratory SOP,
required PPE, startup/shutdown sequence, emergency shutoff procedure, and
equipment-specific manuals before operating the facility.
