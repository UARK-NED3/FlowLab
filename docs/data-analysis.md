# Data Analysis

This document summarizes the core calculations used for FlowLab thermal-fluid
and acoustic-emission analysis. Variable names are intentionally explicit so
they can be mapped to notebook columns or experiment logs.

## Recommended Workflow

1. Load each raw file without modifying it.
2. Validate column names, units, sampling periods, and missing values.
3. Convert units at the data-loading boundary.
4. Build a common timeline, usually the LabVIEW flow-loop timeline.
5. Align heater PSU data, AE data, and accelerometer data to that timeline.
6. Calculate thermal-hydraulic metrics.
7. Calculate spectral or event features for pressure, AE, and acceleration.
8. Save processed data and metadata separately from raw inputs.

## Fluid And Geometry Constants

Record the values used for every analysis run:

- Liquid density, `rho_l`, for deionized water at the relevant temperature.
- Latent heat of vaporization, `h_fg`, for water at the relevant pressure.
- Heat-sink width, `W_hs`.
- Heat-sink length, `L_hs`.
- Heated base area, `A_b = W_hs L_hs`.
- Heat-loss coefficients, `a` and `b`, for the tested heat-sink/sample
  assembly.

The SOP notes that the heat-loss coefficients are empirical and valid for the
aluminum straight-milled microchannel heat-sink sample used in the documented
tests. Do not reuse them for another geometry or material without validation.

## Flow-Rate Calculation

When using flow-meter pulses:

```text
f_pulse = 1 / tau_pulse
Q_L_min = 60 f_pulse / K
```

where:

- `tau_pulse` is the average pulse period in seconds.
- `f_pulse` is the pulse frequency in pulses/s.
- `K = 22000` pulses/liter is the flow-meter K-factor.
- `Q_L_min` is volumetric flow rate in L/min.

Convert to SI volume flow rate and mass flow rate:

```text
Q_m3_s = Q_L_min / 60000
m_dot = rho_l Q_m3_s
```

## Heater PSU Power

For uniformly sampled PSU data:

```text
t_psu[i] = t_psu_0 + i T_psu
P_elec[i] = V[i] I[i]
```

where `i = 0, 1, 2, ...`, `T_psu` is the PSU sampling period, `V` is voltage,
and `I` is current.

If the logged voltage or current uses non-SI units or scaled values, convert to
volts and amperes before calculating power.

## Flow-Loop Timeline And PSU Alignment

For uniformly sampled flow-loop data:

```text
t_fl[j] = t_fl_0 + j T_fl
```

where `j = 0, 1, 2, ...` and `T_fl` is the flow-loop sampling period.

Map PSU power to the flow-loop timeline with nearest-neighbor alignment:

```text
k(j) = argmin_i |t_fl[j] - t_psu[i]|
P_aligned[j] = P_elec[k(j)]
```

Nearest-neighbor alignment is simple and auditable. If the PSU sample period is
large relative to the flow-loop sample period or the power ramp changes quickly,
use interpolation and report the interpolation method.

## Heat-Loss Correction

The SOP models parasitic heat loss to the surroundings as a linear function of
baseplate temperature:

```text
Q_loss = a T_base + b
Q_net = P_aligned - Q_loss
```

where:

- `Q_loss` is empirical heat loss to surroundings.
- `T_base` is baseplate temperature.
- `Q_net` is the electrical input that reaches the coolant/test section after
  losses.

Use the same temperature units used to fit `a` and `b`.

## Thermal Metrics

Base heat flux:

```text
q_base = Q_net / A_b
```

Sensible heat absorbed by the liquid:

```text
Q_sens = m_dot c_p (T_out - T_in)
```

Boiling or latent contribution:

```text
Q_lat = Q_net - Q_sens
```

Overall thermal resistance:

```text
R_th = (T_base - T_in) / Q_net
```

Area-normalized effective resistance:

```text
R_eff = (T_base - T_in) / q_base
```

Outlet vapor quality estimate:

```text
x_out = Q_lat / (m_dot h_fg)
```

Interpret `x_out` only when boiling is physically expected and the assumptions
behind `Q_lat` are valid. Negative `Q_lat` or `x_out` values usually indicate
single-phase sensible heating, heat-loss/model mismatch, transient storage, or
alignment/calibration issues.

## Pressure Drop

Pressure drop across the test section is:

```text
Delta_p = p_in - p_out
```

For steady-state tests, average pressure drop over the stabilized 60 s logging
window and report variability, such as standard deviation or confidence
interval. For transient tests, preserve the time-resolved signal and compare it
against heater power, flow rate, and temperature.

## Pressure-Drop Spectrogram

For a discrete pressure-drop signal `x[n]` sampled with period `T_s`:

```text
f_s = 1 / T_s
X(m, k) = STFT{x[n]} using a selected window length and hop size
S(m, k) = |X(m, k)|^2
S_dB(m, k) = 10 log10(S(m, k) / S_ref)
```

Record the window type, window length, hop size, overlap, detrending, and any
filtering. Use the same spectrogram settings when comparing conditions.

## Acoustic-Emission Feature Definitions

EasyAE exports several event-level parameters. Common definitions used in the
SOP are:

| Feature | Definition |
| --- | --- |
| Amplitude | Highest voltage in the AE waveform, expressed on the dBAE amplitude scale. |
| Energy | Time integral of absolute signal voltage; magnitude depends on the selected Energy Reference Gain and is proportional to signal strength. |
| Counts | Number of threshold crossings. |
| Duration | Time from first to last threshold crossing, in us. |
| RMS | Root mean square voltage during a software-programmable time constant, referred to the input of the signal-processing board. |
| ASL | RMS converted to dBAE, with 0 dBAE = 1 uV at the sensor before amplification. |
| Threshold | Detection threshold on the dBAE scale. Most useful for floating or smart threshold modes; fixed-threshold values do not change during acquisition. |
| Rise time | Time from first threshold crossing to maximum waveform voltage, in us. |
| Counts to peak | Number of threshold crossings from first crossing to maximum waveform voltage. |
| Average frequency | Counts divided by duration and divided by 1000, reported in kHz. This is a time-domain feature, not a spectral-domain calculation. |
| Reverberation frequency | `(Counts - Counts to Peak) / (Duration - Rise Time)`. |
| Initiation frequency | `Counts to Peak / Rise Time`. |
| Signal strength | Time integral of absolute signal voltage, in pV*s, referenced to the sensor before amplification; proportional to Energy. |
| Absolute energy | Time integral of squared sensor voltage before amplification divided by 10 kohm impedance, reported in aJ. |

When analyzing AE data, report threshold settings, preamplifier/gain settings,
sensor mounting condition, acquisition mode, and whether features come from
hit-based, time-based, or waveform-streaming exports.

## Cross-Modal Analysis Ideas

Useful cross-modal comparisons include:

- `Delta_p`, `q_base`, and `x_out` versus AE hit rate or signal strength.
- AE spectral content or waveform energy during pressure-drop oscillations.
- Accelerometer RMS or spectral peaks versus pump operating point and flow
  regime.
- Time-aligned heater-power ramps against thermal response, pressure response,
  and AE event bursts.
- Steady-state 60 s averages grouped by inlet temperature, flow rate, and
  heater power.

Always separate physical interpretation from acquisition artifacts. Pump noise,
valve adjustments, sensor remounting, waveform-trigger settings, and thermal
storage can all create cross-modal changes that are not boiling-regime changes.
