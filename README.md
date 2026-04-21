# 🔢 Analog Calculator without Digital Processing
### EE 322: Analog & Mixed Signal Circuits — IIT Gandhinagar

---

## 📌 Project Overview

This project implements a **fully analog function calculator** that performs mathematical operations directly in the continuous-time domain — no microprocessors, no digital computation. All arithmetic is carried out using op-amp based circuits built on breadboard and PCB.

The calculator supports **10 mathematical operations**, each implemented as a dedicated analog circuit block using LM358 op-amps and 1N4148 diodes. The system uses a mixed-signal architecture where analog circuits handle computation, and digital ICs (MUX/DEMUX) handle signal routing.

> **Current Implementation Status:** All analog computation circuits and routing logic (Blocks B3–B5) are implemented and tested on breadboard and PCB. The DAC input stage and ADC/display output stage are not yet integrated.

---

## ✨ Supported Operations

| # | Operation | Method | Transfer Function |
|---|---|---|---|
| 1 | Addition | Inverting adder + inverting buffer | Vout = V1 + V2 |
| 2 | Subtraction | Inverting buffer + inverting adder | Vout = V1 − V2 |
| 3 | Multiplication | Log-antilog with reference voltage | Vout = (V1 · V2) / VR |
| 4 | Division | Log-antilog with inverted V2 | Vout = V1 / (V2 · R) |
| 5 | Logarithm | Log conditioning + division circuit | Vout = log_V1(V2) |
| 6 | Square | Log-antilog with gain-of-2 adder | Vout = V² / VR |
| 7 | Differentiator | Practical RC op-amp differentiator | Vo ≈ −Rf·C1·(dVin/dt) |
| 8 | Integrator | Practical RC op-amp integrator | Vo ≈ −(1/R1·Cf)·∫Vin dt |
| 9 | Square Root | Closed-loop log feedback, gain 0.5 | Vout = √(Vi / k) |
| 10 | Anti-logarithm | Diode-resistor antilog circuit | Vo ∝ e^(−Vi/constant) |

---

## 🛠️ Hardware Components

| Component | Part | Purpose |
|---|---|---|
| Op-Amp | LM358 | All analog computation stages |
| Diode | 1N4148 | Log/antilog amplifier nonlinearity |
| Resistors | 4.32 kΩ | Matched resistor network for all stages |
| MUX | 74LS151 | Routes analog output of active block to ADC |
| DEMUX | 74LS138 | Routes input voltages to the correct analog block |
| DC Voltage Source | ±15V | Rail supply for all op-amp circuits |
| Breadboard + Wires | — | Prototyping and testing |
| PCB | Custom (×2) | Final implementation |

---

## 🏗️ System Architecture

The system is divided into **7 functional blocks** across two PCBs:

```
  [User Input]
       │
       ▼
  ┌─────────┐     ┌──────────┐     ┌──────────────────────────────────┐
  │ B1      │     │ B2       │     │ B3 DEMUX (74LS138)               │
  │ Input & │────►│ DAC      │────►│ Routes V1, V2, Vref to the       │
  │ Control │     │ (DAC0808)│     │ selected analog block            │
  └─────────┘     └──────────┘     └──────────┬───────────────────────┘
  [Pending]       [Pending]                    │
                                              ▼
                              ┌───────────────────────────────┐
                              │ B4: Analog Computation (LM358) │
                              │  + Add    + Sub    + Mult      │
                              │  + Div    + Log    + Square    │
                              │  + Diff   + Integ  + Sqrt      │
                              │  + Antilog                     │
                              └───────────────┬───────────────┘
                                              │
                                              ▼
                              ┌───────────────────────────────┐
                              │ B5: MUX (74LS151)             │
                              │ Selects Vout from active block│
                              └───────────────┬───────────────┘
                                              │
                              ┌───────────────▼───────────────┐
                              │ B6: ADC (ADC0808)  [Pending]  │
                              └───────────────┬───────────────┘
                                              │
                              ┌───────────────▼───────────────┐
                              │ B7: Arduino + Display [Pending]│
                              └───────────────────────────────┘
```

---

## 📐 Circuit Details

### 1. Addition
Implements an **inverting adder** followed by an **inverting buffer** to restore sign.
All resistors are matched at 4.32 kΩ for unity gain.
> Vout = V1 + V2

### 2. Subtraction
An **inverting buffer** on V2 feeds into an **inverting adder** with V1.
> Vout = V1 − V2

### 3 & 4. Multiplication / Division
Uses the **log-antilog method**:
- Inverting log amplifiers on input voltages (1N4148 diode in feedback)
- A non-inverting log amp on the reference voltage VR
- An inverting adder sums the log signals (subtraction for division)
- A non-inverting antilog amplifier reconstructs the result

Rail voltages: ±15V. Reference voltage VR set via potentiometer.
> Mult: Vout = (V1 · V2) / VR
> Div:  Vout = V1 / (V2 · R)

### 5. Logarithm
Each input is passed through a **log conditioning circuit** (inverting log amp minus log of VR), then fed into the division circuit.
> Vout = log_V1(V2)

### 6. Square
Same log-antilog architecture as multiplication, but with the inverting adder gain set to **2** (doubles the log of V1 before antilog).
> Vout = V² / VR

### 7. Differentiator (Practical)
A practical design with series R–C1 at the inverting input and Rf ∥ Cf in feedback. The series resistor limits high-frequency gain; Cf provides roll-off for stability.
Compensation: Rcomp = Rf ∥ R1
> Vo ≈ −Rf·C1·(dVin/dt)  [at low frequencies]

### 8. Integrator (Practical)
R1 at the inverting input charges feedback capacitor Cf. Rf in parallel with Cf prevents DC saturation.
Compensation: Rcomp = R1 ∥ Rf
> Vo ≈ −(1/R1·Cf)·∫Vin dt  [at mid frequencies]

### 9. Square Root
A **closed-loop log-feedback** configuration with three stages:
- Left: inverting log amp (Vlog ∝ ln(Vi))
- Middle: inverting amplifier with gain **0.5** (halves the log signal)
- Right: antilog converter

The feedback forces: exp(0.5·ln(Vi)) = Vi^0.5 = √Vi
> Vout = √(Vi / k)

*Note: Both diodes must be thermally matched. Input Vi must be positive.*

### 10. Anti-logarithm
Diode placed at the input (instead of feedback as in the log amp), resistor Rf in the feedback path. The input current flows through the diode whose V–I relationship is exponential.
> Vo ∝ e^(−Vi/constant)

---

## 📡 Signal Routing

### DEMUX — 74LS138 (3-to-8 decoder)
- 3 select lines (A0, A1, A2) from control logic
- Routes V1, V2, and Vref to the selected analog block
- Active-LOW outputs

### MUX — 74LS151 (8-to-1 selector)
- Same 3 select lines as DEMUX (synchronised)
- Selects Vout from the active computation block
- Routes result to the ADC input

---

## 📦 PCB Design

Two dedicated PCBs for modularity and noise isolation:

**Control/Display PCB** *(B1, B2, B6, B7 — Pending)*
- Arduino UNO, keypad input, DAC0808, ADC0808, display
- Handles all digital signalling and power supply conditioning for the analog board

**Analog Computation PCB** *(B3, B4, B5 — Implemented)*
- 74LS138 DEMUX, all op-amp circuits (LM358 + 1N4148), 74LS151 MUX
- Designed for low noise, precise grounding, and matched component placement to maximise accuracy

---

## ✅ Implementation Status

| Block | Component | Status |
|---|---|---|
| B1 — Input & Control | Arduino + Keypad | ⏳ Pending |
| B2 — DAC | DAC0808 | ⏳ Pending |
| B3 — DEMUX | 74LS138 | ✅ Implemented & Tested |
| B4 — Analog Computation | LM358 + 1N4148 (all 10 ops) | ✅ Implemented & Tested |
| B5 — MUX | 74LS151 | ✅ Implemented & Tested |
| B6 — ADC | ADC0808 | ⏳ Pending |
| B7 — Display | Arduino + Display | ⏳ Pending |

---

## 🧪 Testing Approach

Each block was tested independently before integration:

1. **Individual analog blocks** — DC bench supply used as inputs; output verified against theoretical transfer function using multimeter/oscilloscope
2. **DEMUX routing** — correct signal routing to each analog block verified by toggling select lines
3. **MUX output selection** — confirmed correct Vout is selected from the active block
4. **Log/antilog circuits** — reference voltage VR calibrated via potentiometer until output matched expected values; diodes thermally matched for multiplication, division, square, and square root circuits

---

## ⚠️ Known Limitations

- All log/antilog operations (multiply, divide, log, square, square root) require **positive input voltages only** due to diode polarity constraints
- Reference voltage VR must be manually calibrated via potentiometer for each log-domain operation
- LM358 op-amps have a limited slew rate (~0.6 V/µs); the differentiator and integrator are accurate only within their designed frequency range
- 74LS138 and 74LS151 are digital logic ICs routing analog signals — care is needed to avoid signal degradation at the switching boundary

---

## 📚 References

1. MIT 6.101 Analog Circuits Project — [gopalan_Project_Final_Report.pdf](https://web.mit.edu/6.101/www/s2018/projects/gopalan_Project_Final_Report.pdf)
2. Texas Instruments Analog Engineer's Calculator — [ti.com](https://www.ti.com/tool/ANALOG-ENGINEER-CALC)
3. IIT Roorkee Virtual Lab — Log-Antilog Amplifier Theory — [ae-iitr.vlabs.ac.in](https://ae-iitr.vlabs.ac.in/exp/log-antilog-amplifier/theory.html)
4. TutorialsPoint — Log and Anti-Log Amplifiers — [tutorialspoint.com](https://www.tutorialspoint.com/linear_integrated_circuits_applications/linear_integrated_circuits_applications_log_and_anti_log_amplifiers.htm)

---

*EE 322: Analog & Mixed Signal Circuits — Indian Institute of Technology Gandhinagar*
