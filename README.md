# Demonstration-of-STDP-Learning-Rule-using-RRAM-in-Cadence-
# Neuromorphic RRAM: 10-Bit Quantized STDP Synapse

This repository contains a Verilog-A implementation of a Resistive Random Access Memory (RRAM) cell designed for Spike-Timing-Dependent Plasticity (STDP). The model bridges the gap between biological neural behavior and digital hardware by utilizing a 1024-level (10-bit) quantization architecture.

## Overview
Biological synapses learn through the relative timing of spikes. This project replicates Long-Term Potentiation (LTP) and Long-Term Depression (LTD) using a memristive device model. By simulating the interaction of bio-inspired pulses, we generate the characteristic "Butterfly" STDP curve used in neuromorphic computing.

### Key Features
* **10-Bit Precision:** 1024 discrete conductance states to balance biological smoothness with digital storage requirements.
* **Bio-Inspired Pulses:** Custom VPWL pulse design featuring an "Active Head" for thresholding and an "Exponential Tail" for timing overlap.
* **Automated Analysis:** Python-based post-processing for baseline zeroing and coordinate transformation (Delta t centering).
* **Verilog-A Core:** A physics-based filament growth model with hard bounds and digital "dead-zone" management.

---

## Project Architecture

### 1. The RRAM Model (Verilog-A)
The core of the synapse is the RRAM_Cell module. It calculates the filament growth (w) based on the differential voltage across the Top (TE) and Bottom (BE) electrodes.

**Primary Parameters:**
| Parameter | Value | Description |
| :--- | :--- | :--- |
| rl / rh | 40 Ohm / 13 kOhm | LRS and HRS resistance limits |
| vh / vl | 1.5 V / -1.5 V | Set/Reset thresholds |
| levels | 1024 | Number of discrete states (10-bit) |
| alpha_p/n | 5e6 | Learning rate (Potentiation/Depression) |

### 2. Stimulus Design (VPWL)
Learning is triggered by the overlap of Pre- and Post-synaptic spikes. The pulse mimics a biological neuron:
* **Magnitude:** 1.2 V peak.
* **Timing:** 1.5 us total duration.
* **Phase:** Includes a negative phase to enable Symmetric or Hebbian behavior depending on the overlap.

---

## Results and Visualization

### The STDP "Butterfly" Curve
The final result is obtained through a 1001-point parametric sweep in Cadence Virtuoso. The data is then processed in Python to center the trigger point at 0 us.

* **LTP (Potentiation):** Occurs when Delta t < 0 (Pre fires before Post).
* **LTD (Depression):** Occurs when Delta t > 0 (Post fires before Pre).

### Quantization Comparison
Increasing the resolution from 8-bit (256 levels) to 10-bit (1024 levels) significantly reduces quantization noise, providing a near-analog transition curve while remaining compatible with digital memory arrays.

---

## Usage

### Requirements
* **Circuit Simulator:** Cadence Virtuoso (Spectre/APS)
* **Analysis:** Python 3.x (Pandas, Matplotlib)

### Running the Analysis
To center and zero your simulation data, use the following Python logic:
```python
import pandas as pd
import matplotlib.pyplot as plt

# Load and Zero Data
df = pd.read_csv('1024levels_quantized_1001points.csv')

# Clean column names (removes hidden spaces)
df.columns = [c.strip() for c in df.columns]

# Subtract the first value to ensure baseline starts at 0
df['delG_zeroed'] = df['delG'] - df['delG'].iloc[0]

# Coordinate Transform: Shift by 2us to set trigger point to 0
df['delta_t_us'] = (df['tdelay'] - 2e-6) * 1e6 

# Plot
plt.figure(figsize=(10, 6))
plt.plot(df['delta_t_us'], df['delG_zeroed'] * 1e3, color='red')
plt.axhline(0, color='black', linewidth=1)
plt.axvline(0, color='black', linestyle='--')
plt.xlabel('Delta t (us)')
plt.ylabel('Delta G (mS)')
plt.title('Final 1024-Level STDP Curve')
plt.show()
