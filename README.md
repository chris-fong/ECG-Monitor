# ECG Recording Amplifier with Custom Active Filters

## Overview
This repository documents the design, mathematical analysis, and hardware implementation of an amplifier system capable of recording human electrocardiograms (ECGs). Originally designed for the EE 2245 Microelectronics Labs, this project integrates several custom-designed active filters to amplify micro-volt physiological signals while aggressively filtering out environmental and common-mode noise.

## System Architecture
The system consists of primary analog signal processing stages designed to interface directly with wet electrodes:

### 1. Instrumentation Amplifier (INA)
An Instrumentation Amplifier (INA) is implemented at the front end to remove common-mode interferences.
* **Op-Amps:** TLC2264.
* **Function:** Provides high input impedance and extracts the differential voltage between the two arm electrodes ($V_{IN}^+$ and $V_{IN}^-$).


[Image of an instrumentation amplifier schematic]


### 2. Active High-Pass Filter
* **Cutoff Frequency:** Designed for 1 Hz.
* **Input Impedance:** Designed to be $\ge 1 \text{ M}\Omega$ at 1 Hz.
* **Function:** Removes baseline wandering and low-frequency motion artifacts caused by human movement, which can severely distort the signal.

### 3. Sallen-Key Low-Pass Filter
* **Natural Frequency ($f_n$):** Designed to fall between 20 Hz and 60 Hz.
* **Flat-band Voltage Gain:** $\ge 3 \text{ V/V}$.
* **Function:** Attenuates high-frequency environmental noise before the final output.


### 4. Biquad Band-Pass Filter
* **Band-pass Frequency:** 5 kHz (with a $\pm 2\%$ variation).
* **Quality Factor:** $Q \ge 8$.

---

## Schematics, Calculations, & Design Math

### Instrumentation Amplifier (INA) Gain
The theoretical differential gain of the Instrumentation Amplifier is governed by the external resistor $R_G$ using the following relationship:
$$V_{O1} = (V_{in}^+ - V_{in}^-)(1 + \frac{50 \times 10^3}{R_G})$$

* **Theoretical:** By setting $R_G = 7 \text{ k}\Omega$, the theoretical calculation yields $A_v \cong 8.14 \text{ V/V}$.
* **Measured:** The actual measured voltage gain was 8.8 V/V. This represents a 7.5% error margin, likely stemming from resistor tolerances or loading effects from the oscilloscope and function generator.

### Active High-Pass Filter Math


**Transfer Function & Gain:**
The transfer function of the filter is derived as:
$$A_v(s) = \frac{[C_2(R_2+R_3) + \frac{1}{s}]R_1 C_1}{[R_2 C_2 + \frac{1}{s}][R_1 C_1 + \frac{1}{s}]}$$

To find the flat-band voltage gain, we evaluate the limit as frequency approaches infinity:
$$\lim_{s \to \infty} A_v(s) = 1 + \frac{R_3}{R_2}$$

Our target flat-band gain was 100 V/V. This requires the resistor ratio to satisfy:
$$90 < 1 + \frac{R_3}{R_2} < 110$$
$$89 < \frac{R_3}{R_2} < 109$$
We selected $R_3 = 1.543 \text{ M}\Omega$ and $R_2 = 15.89 \text{ k}\Omega$.

**Input Impedance Constraint:**
To prevent loading the delicate ECG signal, the input impedance at 1 Hz must be $\ge 1 \text{ M}\Omega$. The impedance magnitude is:
$$|Z_{in}| = \sqrt{R_1^2 + \left(\frac{1}{\omega C_1}\right)^2}$$

To satisfy this constraint at 1 Hz ($\omega = 2\pi$), the input capacitor must be carefully sized:
$$C_1 \le \frac{1}{\sqrt{2}\pi 10^6} \le 225.08 \text{ nF}$$

### Sallen-Key Low-Pass Filter Math
**Transfer Function:**
The generalized input-output relationship for this specific implementation is:
$$\frac{V_{LP}}{V_i} = \frac{\frac{R_3+R_4}{R_3}}{s^2(C_1 C_2 R_1 R_2) + s(R_2 C_2 + R_1 C_2 - \frac{C_1 R_1 R_4}{R_3}) + 1}$$

**Natural Frequency ($\omega_n$):**
The natural frequency of the filter determines where the attenuation begins:
$$\omega_n = \frac{1}{\sqrt{C_1 C_2 R_1 R_2}} = 2\pi f_n$$

The design constraint required $f_n$ to fall between 20 Hz and 60 Hz. We targeted a natural frequency of 40 Hz, which gives:
$$C_1 C_2 R_1 R_2 = 15.83\mu$$
Given the secondary constraint that $R_1 \ge 10 \text{ k}\Omega$, we selected $R_1 = 22 \text{ k}\Omega$ and $C_1, C_2 = 100 \text{ nF}$. This dictated an $R_2$ value of $72 \text{ k}\Omega$.

**Flat-band Gain:**
By evaluating the transfer function at DC ($s = 0$), the flat-band voltage gain is established:
$$\frac{V_{LP}}{V_i}\Big|_{s=0} = 1 + \frac{R_4}{R_3} \ge 3$$
Setting $R_3 = 20 \text{ k}\Omega$ yields a required $R_4 \ge 2 R_3$ to satisfy the minimum gain constraint, leading to an empirical $R_4 = 43 \text{ k}\Omega$.

### Heart Rate Extraction
Using the oscilloscope capture of the live ECG, the heart rate was calculated based on the period between R-wave peaks:
* **Calculation:** $\Delta t = 1.86 \text{ s}$ and $\Delta f = 400 \text{ ms}$, yielding $\frac{32.26}{400 \times 10^{-3}} = 80.65 \text{ bpm}$.

---

## Filter Component Selection & Tolerances

### 1. Biquad Band-Pass Filter 
| Component | Calculated Value | Measured Value |
| :--- | :--- | :--- |
| **C1** | 1 nF | 0.982 nF |
| **C2** | 20 nF | 20.80 nF |
| **R1** | 300 kΩ | 298.57 kΩ |
| **R2** | 400 kΩ | 395.98 kΩ |
| **R3** | 100 kΩ | 97.92 kΩ & 99.52 kΩ |
| **R4** | 5.1 kΩ | 5.098 kΩ |
| **R5** | 10 kΩ | 9.992 kΩ |

### 2. Active High-Pass Filter 
| Component | Calculated Value | Measured Value |
| :--- | :--- | :--- |
| **C1** | 100 nF | 103.6 nF |
| **C2** | 10 µF | 11.23 µF |
| **R1** | 1.5 MΩ | 1.549 MΩ |
| **R2** | 16 kΩ | 15.89 kΩ |
| **R3** | 1.5 MΩ | 1.543 MΩ |

### 3. Sallen-Key Low-Pass Filter 
| Component | Calculated Value | Measured Value |
| :--- | :--- | :--- |
| **C1** | 100 nF | 97.8 nF |
| **C2** | 100 nF | 98.6 nF |
| **R1** | 22 kΩ | 21.84 kΩ |
| **R2** | 72 kΩ | 73.8 kΩ |
| **R3** | 20 kΩ | 20.1 kΩ |
| **R4** | 43 kΩ | 42.1 kΩ |

---

## Frequency Response Characterization

### 1. Biquad Band-Pass Filter Response 
| Frequency (kHz) | $V_{in(p-p)}$ (mV) | $V_{out(p-p)}$ (mV) | Gain (V/V) |
| :--- | :--- | :--- | :--- |
| 1 | 2000 | 90 | 0.045 |
| 1.5 | 2000 | 120 | 0.06 |
| 2 | 2000 | 160 | 0.08 |
| 2.5 | 2000 | 210 | 0.105 |
| 3 | 2000 | 270 | 0.135 |
| 3.5 | 2000 | 340 | 0.17 |
| 4 | 2000 | 450 | 0.225 |
| 4.5 | 2000 | 1020 | 0.51 |
| 4.6 | 2000 | 1080 | 0.54 |
| 4.65 | 2000 | 1190 | 0.595 |
| 4.7 | 2000 | 1380 | 0.69 |
| 4.8 | 2000 | 1670 | 0.835 |
| 4.9 | 2000 | 2090 | 1.045 |
| 5 | 2000 | 2520 | 1.26 |
| 5.1 | 2000 | 2540 | 1.27 |
| 5.2 | 2000 | 2200 | 1.1 |
| 5.3 | 2000 | 1800 | 0.9 |
| 5.5 | 2000 | 1300 | 0.65 |
| 5.8 | 2000 | 800 | 0.4 |
| 6 | 2000 | 620 | 0.31 |
| 6.2 | 2000 | 580 | 0.29 |
| 6.9 | 2000 | 410 | 0.205 |
| 7 | 2000 | 360 | 0.18 |

### 2. Active High-Pass Filter Response
| Frequency (Hz) | $V_{in(p-p)}$ (mV) | $V_{out(p-p)}$ (mV) | Gain (V/V) | Gain (dB) |
| :--- | :--- | :--- | :--- | :--- |
| 0.1 | 100 | 136 | 1.36 | 2.67 |
| 0.5 | 100 | 301 | 3.01 | 9.57 |
| 1 | 100 | 930 | 9.30 | 19.37 |
| 2 | 100 | 2630 | 26.30 | 28.40 |
| 3 | 100 | 4060 | 40.60 | 32.17 |
| 4 | 100 | 5380 | 53.80 | 34.62 |
| 5 | 100 | 6500 | 65.00 | 36.26 |
| 6 | 100 | 7410 | 74.10 | 37.40 |
| 7 | 100 | 7910 | 79.10 | 37.98 |
| 8 | 100 | 8300 | 83.00 | 38.38 |
| 9 | 100 | 8910 | 89.10 | 39.00 |
| 10 | 100 | 9190 | 91.90 | 39.27 |
| 20 | 100 | 10860 | 108.60 | 40.72 |
| 50 | 100 | 11440 | 114.40 | 41.17 |
| 100 | 100 | 11500 | 115.00 | 41.21 |

### 3. Sallen-Key Low-Pass Filter Response 
| Frequency (Hz) | $V_{in(p-p)}$ (mV) | $V_{out(p-p)}$ (mV) | Gain (V/V) | Gain (dB) |
| :--- | :--- | :--- | :--- | :--- |
| 0.1 | 100 | 345 | 3.45 | 10.76 |
| 0.5 | 100 | 345 | 3.45 | 10.76 |
| 1 | 100 | 345 | 3.45 | 10.76 |
| 5 | 100 | 345 | 3.45 | 10.76 |
| 10 | 100 | 345 | 3.45 | 10.76 |
| 25 | 100 | 345 | 3.45 | 10.76 |
| 30 | 100 | 362 | 3.62 | 11.17 |
| 33 | 100 | 358 | 3.58 | 11.08 |
| 35 | 100 | 349 | 3.49 | 10.86 |
| 40 | 100 | 315 | 3.15 | 9.97 |
| 60 | 100 | 180 | 1.80 | 5.11 |
| 80 | 100 | 131 | 1.31 | 2.35 |
| 100 | 100 | 93 | 0.93 | -0.63 |

---

## Results (Images) & Hardware Troubleshooting

### Live ECG Capture

*(Placeholder: Upload the photo showing the periodic ECG waveform on the Tektronix oscilloscope.)*

### Experimental Setup

*(Placeholder: Upload the photo of the wet electrodes attached to the body.)*

### Hardware Troubleshooting & Observations
Transitioning from theoretical models to a physical breadboard introduced several real-world challenges that required troubleshooting:

* **DC Offsets:** The amplifier chain exhibited a DC offset of approximately -50mV. The intermediate amplifier stage (A4) failed to completely reject this offset, which subsequently passed to the A5 amplifier and was amplified to approximately -45mV at the output.
* **Electromagnetic Interference (EMI):** The raw signal experienced noticeable interference. This was primarily caused by electromagnetic coupling from the nearby lab power supply at around 50 Hz. Additionally, human movement introduced low-frequency interference that distorted the signal.
* **Mitigation Strategy:** To improve signal integrity, interference was reduced by minimizing RF emissions (moving mobile phones away from the circuit) and ensuring the wet electrodes were securely and correctly placed.
