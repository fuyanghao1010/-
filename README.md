# Custom Ring Countdown Timer — Circuit Technical Documentation

## 1. Overview

This circuit is a **three-digit BCD presettable countdown timer** with an **audible buzzer alarm** that triggers when the count reaches a user-defined target value. It is built entirely from standard TTL and CMOS digital logic ICs. A NE555 timer provides the clock source; three cascaded 74LS192 BCD up/down counters perform down-counting across a 000–999 range; the count is displayed on three common-anode 7-segment displays via 74LS47 BCD-to-7-segment decoder/drivers. Three cascaded 4585 4-bit magnitude comparators form a multi-digit comparison circuit that compares the current count against a preset alarm reference value (set by DIP switches), triggering the buzzer when the count decrements to the target.

- **Design Tool:** NI Multisim 14.3
- **Source File:** `自定义响铃倒计时器.ms14`

---

## 2. Overall Architecture

The circuit comprises the following functional blocks:

```
                         ┌──────────────────┐
   DIP Switches ────────→│  Preset Values    │
  (S7, S9)               │  (Hundreds/Tens/  │
                          │   Units)          │
                         └────────┬───────────┘
                                  │
      ┌───────────────────────────▼──────────────────────────┐
      │      Three 74LS192 Cascaded BCD Down-Counters         │
      │   U12(Units) ──→ U20(Tens) ──→ U24(Hundreds)         │
      │   ~BO Borrow signal propagates between stages          │
      └───────────────────────────┬──────────────────────────┘
                                  │ BCD Outputs
      ┌───────────────────────────▼──────────────────────────┐
      │       Three 74LS47 BCD-to-7-Segment Decoders          │
      │   U11 ──→ U1(Units)      U31 ──→ U32(Tens)           │
      │   U23 ──→ U22(Hundreds)                               │
      └───────────────────────────┬──────────────────────────┘
                                  │
      ┌───────────────────────────▼──────────────────────────┐
      │      Three Common-Anode 7-Segment Displays             │
      │      Display current remaining count value             │
      └──────────────────────────────────────────────────────┘

      ┌──────────────────────────────────────────────────────┐
      │  NE555 Astable Multivibrator (A2) → Clock Source      │
      │  R3=60kΩ, R4=60kΩ, C3=10nF, C4=10nF                  │
      └──────────────────┬───────────────────────────────────┘
                         │ CLK (gated)
      ┌──────────────────▼───────────────────────────────────┐
      │              Control Logic Gates                       │
      │  74LS32(OR) ×4, 74LS08(AND) ×2                        │
      │  74LS00(NAND) ×1, 74LS04(NOT) ×1                      │
      │  Clock gating, borrow propagation, load control        │
      └──────────────────────────────────────────────────────┘

      ┌──────────────────────────────────────────────────────┐
      │    Three 4585 Cascaded 4-Bit Magnitude Comparators     │
      │   74ls85, U2, U3 → Compare count vs. alarm reference   │
      │   Alarm reference set by S5, S6 DIP switches           │
      └──────────────────┬───────────────────────────────────┘
                         │ Equality Signal
      ┌──────────────────▼───────────────────────────────────┐
      │            Buzzer Alarm Driver Circuit                 │
      │   Q4(2N2222) NPN transistor drives LS1(1kHz buzzer)   │
      └──────────────────────────────────────────────────────┘
```

---

## 3. Bill of Materials (BOM)

| # | Qty | Part | RefDes | Package | Function |
|---|-----|------|--------|---------|----------|
| 1 | 3 | **74LS47D** | U11, U23, U31 | DIP-16 | BCD-to-7-segment decoder/driver (OC, common-anode) |
| 2 | 3 | **74LS192D** | U12, U20, U24 | DIP-16 | Synchronous decade up/down counter (dual-clock) |
| 3 | 4 | **74LS32D** | U6, U9, U34, U35 | DIP-14 | Quad 2-input OR gate |
| 4 | 2 | **74LS08D** | U10, U33 | DIP-14 | Quad 2-input AND gate |
| 5 | 2 | **Resistor 60kΩ** | R3, R4 | 0805 SMD | NE555 timing resistors |
| 6 | 2 | **Capacitor 10nF** | C3, C4 | Through-hole | NE555 timing/decoupling capacitors |
| 7 | 1 | **74LS00D** | U17 | DIP-14 | Quad 2-input NAND gate |
| 8 | 1 | **74LS04D** | U18 | DIP-14 | Hex inverter |
| 9 | 1 | **SPDT** | S4 | — | Single-pole double-throw switch (start/load control) |
| 10 | 6 | **DSWPK_4** | S5, S6, S7, S8, S9, S22 | DIP-4 | 4-position DIP switch |
| 11 | 1 | **2N2222** | Q4 | TO-18 | NPN transistor (buzzer driver) |
| 12 | 1 | **BUZZER 1kHz** | LS1 | — | Active buzzer |
| 13 | 3 | **4585BT_5V** | 74ls85, U2, U3 | SO-16 | 4-bit magnitude comparator (CMOS) |
| 14 | 1 | **NE555** | A2 | DIP-8 | Precision timer (astable multivibrator) |
| 15 | 1 | **Resistor** | R2 | — | Transistor base current-limiting resistor |
| 16 | 3 | **7-Seg Display (CA)** | U1, U22, U32 | — | Count display |

> **Note:** The 7-segment displays U1, U22, U32 and resistor R2 are not explicitly listed in the BOM report generated by Multisim, but their presence is confirmed by the netlist connections. R2 connects between Q4's base and the comparator output. U1/U22/U32 are common-anode 7-segment displays (COM pin tied to VCC).

---

## 4. Detailed Functional Block Analysis

### 4.1 Clock Source: NE555 Astable Multivibrator (A2)

The NE555 is configured as an **astable multivibrator** to generate a continuous square-wave clock signal for the counters.

**Pin Connections:**

| NE555 Pin | Function | Connection |
|-----------|----------|------------|
| 1 (GND) | Ground | GND |
| 2 (TRI) | Trigger | Tied to THR(6), junction of R4 and C3 |
| 3 (OUT) | Output | Via AND gate U10A to counter clock input |
| 4 (RST) | Reset | VCC (held high, not reset) |
| 5 (CON) | Control Voltage | Bypassed to GND via C4 (10nF) |
| 6 (THR) | Threshold | Tied to TRI(2) |
| 7 (DIS) | Discharge | Junction of R3 and R4 |
| 8 (VCC) | Supply | VCC (+5V) |

**Oscillation Frequency:**

- R3 = 60kΩ (charging resistor, VCC → DIS)
- R4 = 60kΩ (DIS → THR/TRI resistor)
- C3 = 10nF (timing capacitor)

$$
f = \frac{1.44}{(R_3 + 2R_4) \times C_3} = \frac{1.44}{(60\text{k} + 120\text{k}) \times 10\text{nF}} = \frac{1.44}{180\text{k} \times 10^{-8}} = 800\text{ Hz}
$$

**Duty Cycle:**
$$
D = \frac{R_3 + R_4}{R_3 + 2R_4} = \frac{120\text{k}}{180\text{k}} \approx 66.7\%
$$

> The 800 Hz clock means the counter decrements 800 times per second, corresponding to a countdown range of 0–999 with a maximum duration of approximately 1.25 seconds. For longer countdown intervals, increase R3/R4 or C3 (e.g., switching to µF-range capacitors to achieve 1 Hz → 999 seconds ≈ 16.7 minutes).

### 4.2 Down-Counter: 74LS192 Cascaded Chain (U12, U20, U24)

Three **74LS192 synchronous decade up/down counters** are cascaded to form a three-digit BCD down-counter.

**74LS192 Key Pins:**

| Pin | Name | Function |
|-----|------|----------|
| 4 | DOWN | Count-down clock input (rising-edge triggered) |
| 5 | UP | Count-up clock input (tied to VCC, disabled) |
| 11 | ~LOAD | Asynchronous preset enable (active low) |
| 13 | ~BO | Borrow output (goes low when count = 0 and a DOWN pulse is received) |
| 14 | CLR | Clear (tied to GND, disabled) |
| 15,1,10,9 | A,B,C,D | Preset data inputs |
| 7,6,2,3 | QA,QB,QC,QD | BCD count outputs |

**Cascading Scheme (Borrow Propagation):**

```
 NE555 OUT ──→ [U10A AND] ──→ U20(DOWN)  (gated clock to tens counter)
                      ↑
                 Gate control

 U12(~BO) ──→ U6A(1A) ──→ U6A(1Y) ──→ U9A(1A) ──→ U9A(1Y) ──→ ...
 U24(~BO) ──→ U6A(1B) ──→ ...
```

The counters employ a non-standard cascading scheme via logical combinations of ~BO (borrow) signals. The control logic (74LS32/08/00/04) coordinates the count timing between stages.

**Preset Value Configuration:**

- **S7 (units):** Connected to U12(D,C,B,A) — sets the units initial value (0–9)
- **S9 (tens):** Connected to U20(A,B,C,D) — sets the tens initial value (0–9)
- **S8 (hundreds):** Indirectly controls U24's preset inputs via logic gates U33A, U33C, U34A, U35A

### 4.3 Display Circuit: 74LS47 + 7-Segment Displays

| Counter | Decoder | Display | Digit Position |
|---------|---------|---------|----------------|
| U12 (Units) | U11 (74LS47) | U1 | Units |
| U20 (Tens) | U31 (74LS47) | U32 | Tens |
| U24 (Hundreds) | U23 (74LS47) | U22 | Hundreds |

**74LS47 Key Characteristics:**
- Open-collector outputs, active low (suited for common-anode displays)
- BCD input → seven-segment output (OA–OG)
- ~LT (lamp test), ~RBI (ripple blanking input), ~BI/RBO (blanking input / ripple blanking output)

The common terminals (COM) of displays U1/U22/U32 are tied to VCC, confirming a **common-anode** configuration. Each segment illuminates when the corresponding 74LS47 output sinks current (low).

### 4.4 Magnitude Comparison Circuit: 4585 ×3 Cascaded (74ls85, U2, U3)

Three **4585 4-bit magnitude comparators** are cascaded to form a multi-digit comparison system that determines whether the current count has reached the preset alarm reference value.

**Comparator Cascading:**

```
 74ls85 (Low-order comparator)
   ├── A[3:0] ← U24(QD,QC,QB,QA) (hundreds count)
   ├── B[3:0] ← S22 DIP switch (alarm reference, low digit)
   ├── OAGTB, OAEQB, OALTB → cascade to U2
   │
 U2 (Middle comparator)
   ├── A[3:0] ← U24(QD,QC,QB,QA) (hundreds count)
   ├── B[3:0] ← S5 DIP switch (alarm reference, middle digit)
   ├── Cascade inputs ← 74ls85 comparison results
   ├── OAGTB, OAEQB, OALTB → cascade to U3
   │
 U3 (High-order comparator)
   ├── A[3:0] ← U20(QD,QC,QB,QA) (tens count)
   ├── B[3:0] ← S6 DIP switch (alarm reference, high digit)
   ├── Cascade inputs ← U2 comparison results
   └── OAEQB → Alarm trigger signal
```

**Comparison Logic:** When the three cascaded comparators determine that all digit positions of the count value equal the preset reference value, U3's OAEQB (Output A Equals B) pin goes high, triggering the alarm.

DIP switches S5, S6, and S22 together set the alarm target value in BCD format.

### 4.5 Control Logic

The control logic is built from the following gate ICs, implementing clock gating, borrow handling, and load control:

| IC | RefDes | Gates Used | Function |
|----|--------|-----------|----------|
| 74LS32 | U6, U9, U34, U35 | 4 of 4 OR gates | Borrow signal combining, load logic |
| 74LS08 | U10, U33 | 2 of 4 AND gates | Clock gating, preset enable |
| 74LS00 | U17 | 1 of 4 NAND gates | Control signal combination |
| 74LS04 | U18 | 1 of 6 inverters | Signal polarity inversion |

**Key Control Signals:**

- **Clock Gating (U10A):** The NE555 output is gated through AND gate U10A before reaching the counter. Counting can be paused/resumed via the control signal on U10A(1B).
- **Borrow Propagation:** OR gates U6A and U9A combine the ~BO signals from each counter stage to implement proper cascaded borrow propagation.
- **Load Control:** SPDT switch S4 controls the ~LOAD pins of U12 and U20, allowing manual loading of preset values.
- **U24 Load:** The AND output of U33B drives U24(~LOAD), controlled by S8 and the logic combination.

### 4.6 Buzzer Alarm Circuit

When the comparator detects that the count has reached the preset value, the alarm is triggered:

```
 U3(OAEQB) ──→ R2 ──→ Q4(B) Base
                        │
 Q4(C) Collector ──→ LS1(2) (buzzer low side)
                        │
 LS1(1) ──→ VCC
                        │
 Q4(E) ──→ GND
```

- **Q4 (2N2222):** NPN transistor operating in **switching mode**. When the comparator OAEQB output goes high, the transistor saturates and energizes the buzzer.
- **R2:** Base current-limiting resistor, protecting the transistor. Value chosen to provide sufficient base current while staying within the comparator's output drive capability.
- **LS1:** 1kHz active buzzer, rated for 5V operation.

---

## 5. Power Distribution

The circuit employs two supply rails:

| Rail | Voltage | Supplies |
|------|---------|----------|
| **VCC** | +5V | 74LS TTL ICs, NE555, 7-segment displays, buzzer |
| **VDD** | +5V (CMOS) | 4585 comparators (74ls85, U2, U3), DIP switches S22, S5, S6 |
| **VSS** | GND | 4585 comparator negative supply |
| **GND** | 0V | System common ground |

> Note: VCC and VDD are the TTL and CMOS supply networks respectively. Although both are +5V, they are represented with distinct net labels in the schematic.

---

## 6. Key Netlist Connections

The following table summarizes the core signal paths (representative nets):

| Net | Connections |
|-----|-------------|
| Net 17 | NE555 OUT → AND gate U10A input |
| Net 21 | OR gate U9A output → AND gate U10A other input (gate control) |
| Net 30 | Switch S4 → U12(~LOAD), U20(~LOAD) (load control) |
| Net 25 | AND gate U33B output → U24(~LOAD) (hundreds digit load) |
| Net 27 | 2N2222 collector → Buzzer LS1 |
| Net 47 | Comparator OAEQB → R2 → 2N2222 base |
| Net 58–61 | U12(QA-QD) → 74LS47(U11) → Display U1 |
| Net 97 | Comparator cascade: 74ls85 → U2 → U3 |

---

## 7. Operating Instructions

1. **Set the countdown initial value:** Use DIP switches S7 (units), S9 (tens), and S8 (hundreds) to configure the three-digit BCD starting value (000–999).
2. **Set the alarm reference value:** Use DIP switches S22, S5, and S6 to configure the three-digit BCD alarm target value.
3. **Load the preset value:** Toggle switch S4 (SPDT) to load the DIP-switch-defined initial value into the counters.
4. **Start countdown:** Upon releasing the load, the NE555 clock drives the counters to decrement sequentially through the gating circuit.
5. **Alarm:** When the count decrements to the preset alarm value, the 4585 comparator outputs the equality signal, turning on the transistor and sounding the buzzer.
6. **Display:** The three 7-segment displays show the current remaining count value in real time.

---

## 8. Technical Characteristics and Improvement Suggestions

### 8.1 Technical Characteristics

- **All-hardware implementation:** No microcontroller required — pure digital logic, offering high reliability and deterministic behavior.
- **Modular design:** Counting, display, comparison, and alarm functions are independent modules, facilitating debugging and expansion.
- **TTL/CMOS hybrid:** 74LS series provides strong drive capability; 4585 CMOS comparators offer low power consumption.
- **Flexible presetting:** Initial countdown value and alarm reference value are independently configurable via DIP switches.

### 8.2 Improvement Suggestions

1. **Clock frequency adjustment:** The current 800 Hz supports only ~1.25 seconds of countdown. Increase R3/R4 or use µF-range capacitors for C3 to achieve longer intervals (e.g., 1 Hz → 999 s ≈ 16.7 min).
2. **Add pause/resume:** Utilize the 74LS192 UP pin or expand the gating circuit to add a pause/continue button.
3. **Larger display digits:** Replace with larger 7-segment displays or LED dot-matrix modules for improved visibility.
4. **Battery operation:** Add a battery power circuit and migrate to low-power CMOS equivalents (e.g., 74HC series).
5. **Adjustable alarm volume:** Insert a potentiometer in series with the buzzer for volume control.
6. **Visual alarm indicator:** Add a flashing LED circuit synchronized with the buzzer.

---

## 9. Appendix

### A. File Manifest

| File | Description |
|------|-------------|
| `自定义响铃倒计时器.ms14` | Original Multisim 14 circuit design file |
| `circuit_image.png` | Schematic capture screenshot |
| `bom_report.txt` | Auto-generated Bill of Materials |
| `netlist_report.txt` | Auto-generated netlist report |

### B. Key Component Datasheet References

| Part | Type |
|------|------|
| NE555 | Precision Timer |
| 74LS192 | Synchronous Decade Up/Down Counter (TTL) |
| 74LS47 | BCD-to-7-Segment Decoder/Driver (TTL, OC) |
| 74LS32 | Quad 2-Input OR Gate (TTL) |
| 74LS08 | Quad 2-Input AND Gate (TTL) |
| 74LS00 | Quad 2-Input NAND Gate (TTL) |
| 74LS04 | Hex Inverter (TTL) |
| 4585 (CD4585B) | 4-Bit Magnitude Comparator (CMOS) |
| 2N2222 | NPN Switching Transistor |

---

*Document generated: June 28, 2026*  
*Circuit design tool: NI Multisim 14.3*
<img width="1462" height="1118" alt="image" src="https://github.com/user-attachments/assets/2277f0bb-0683-4c89-aac5-74eb73b436af" />

