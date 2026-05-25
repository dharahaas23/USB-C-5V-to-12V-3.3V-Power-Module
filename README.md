# USB-C Power Delivery Board

A compact multi-rail power supply PCB powered by USB-C, providing three
regulated output voltages: +5V, +3.3V, and +12V. Designed from scratch in KiCad.

![3D Render Front](https://github.com/user-attachments/assets/401b7bf6-4510-472d-8035-8b7a724674e9)

## Features

- **Input:** USB-C (5V from any standard charger)
- **Output 1:** +5V — direct from USB-C, filtered
- **Output 2:** +3.3V — AMS1117-3.3V LDO regulator
- **Output 3:** +12V — MT3608 boost converter
- **Indicators:** LED per output rail (power status)
- **Outputs:** Screw terminals for all 3 rails + GND
- **USB-C CC pins:** 5.1kΩ pull-downs (proper power negotiation)
- **Layers:** 2-layer PCB
- **ERC:** 0 violations
- **DRC:** 0 errors, 0 warnings
- **Unrouted:** 0

## PCB Screenshots

| Schematic | PCB Front | PCB Back | 3D Front | 3D Back |
|---|---|---|---|---|
| ![](https://github.com/user-attachments/assets/7de8c966-7a17-4c1d-ace3-6070accc6bd8) | ![](https://github.com/user-attachments/assets/886c4d0a-2905-4fe6-8c00-ac657cee22c4) | ![](https://github.com/user-attachments/assets/eb5241c5-332c-444a-ba89-7ebfceddac5e) | ![](https://github.com/user-attachments/assets/fbf68e3d-edcc-40d2-ae9c-ce6d2a467052) | ![](https://github.com/user-attachments/assets/e4406ef6-888d-4722-8d64-119b3d111ca1) |

## Schematic Blocks

### Block 1 — USB-C Input
```
USB-C VBUS ──► 100µF bulk cap ──► +5V rail
USB-C CC1  ──► 5.1kΩ ──► GND
USB-C CC2  ──► 5.1kΩ ──► GND
+5V ──► LED (D2) + 470Ω ──► GND  (5V indicator)
```

### Block 2 — MT3608 Boost Converter (+5V → +12V)
```
+5V ──► MT3608 IN (pin5) + EN (pin4)
     ──► 100µF cap ──► GND
     ──► 100nF cap ──► GND

MT3608 SW (pin1) ──► 22µH inductor ──► SS34 cathode ──► +12V
SS34 anode ──► GND

+12V ──► 22µF cap ──► GND
+12V ──► RV1 (100kΩ) ──► FB (pin3) ──► R6 (2.2kΩ) ──► GND
+12V ──► LED (D4) + 470Ω ──► GND  (12V indicator)

MT3608 GND (pin2) ──► GND
MT3608 NC  (pin6) ──► no connect
```

### Block 3 — AMS1117-3.3V (+5V → +3.3V)
```
+5V ──► AMS1117 VI (pin3)
AMS1117 VO (pin2) ──► 10µF cap ──► GND ──► +3.3V rail
AMS1117 GND (pin1) ──► GND
+3.3V ──► LED (D1) + 1kΩ ──► GND  (3.3V indicator)
```

### Block 4 — Output Connectors
```
J2 ──► +5V / GND
J3 ──► +3.3V / GND
J4 ──► +12V / GND
```

## Output Voltage Calculation (MT3608)

```
Vout = Vref × (1 + R1/R2)
Vref = 0.6V (MT3608 internal reference)

For 12V:
12 = 0.6 × (1 + R1/R2)
R1/R2 = 19

R1 = 100kΩ (RV1 trimmer)
R2 = 2.2kΩ (R6)
Vout = 0.6 × (1 + 100k/2.2k) = 0.6 × 46.45 = ~27V (max)

Adjust RV1 trimmer pot down to set exactly 12V output.
```

> **Note:** RV1 is a trimmer potentiometer — adjust it to dial in exactly 12V output.
> This allows fine-tuning without changing resistors.

## Critical Layout Rules Followed

| Rule | Implementation |
|---|---|
| Switching loop tight | MT3608 → inductor → SS34 → output cap placed close together |
| Input cap near VIN | 100µF cap next to MT3608 VIN pin |
| CC pull-downs on USB-C | 5.1kΩ on CC1 + CC2 (required for USB-C power draw) |
| Power trace width | 0.7mm on power rails |
| GND fill both layers | Noise reduction + current return path |
| LED indicators per rail | Visual confirmation each rail is live |

## What I Learned

- **USB-C CC pins are mandatory** — without 5.1kΩ pull-downs, the charger won't provide current
- **MT3608 switching loop must be tight** — inductor, diode, and output cap placed as close as possible to minimize EMI
- **Boost converters can oscillate** without proper input/output capacitors — always add bulk caps
- **Trimmer pot for feedback** — much more flexible than fixed resistors for setting output voltage
- **Switching regulators are a completely different layout skill** from linear regulators like AMS1117

## Specifications

| Rail | Type | Output | Max Current |
|---|---|---|---|
| +5V | Direct (filtered) | 5V | 1.5A (USB-C limited) |
| +3.3V | LDO (AMS1117) | 3.3V | 800mA |
| +12V | Boost (MT3608) | 12V (adjustable) | 1.2A |

## Files

| File/Folder | Description |
|---|---|
| [USB-C 5V to 12V(or)3.3V Power Module.zip](https://github.com/user-attachments/files/28216456/USB-C.5V.to.12V.or.3.3V.Power.Module.zip) | Production-ready Gerber files,schematic file,PCB file |
| https://github.com/user-attachments/assets/7de8c966-7a17-4c1d-ace3-6070accc6bd8 | Full KiCad schematic export |
| [USB-C Power Delivery Board.csv](https://github.com/user-attachments/files/28215278/USB-C.Power.Delivery.Board.csv) | Bill of materials |
| https://github.com/user-attachments/assets/2622ae87-a2a4-46a8-87d8-6a400186b871 |PCB layout|

## Bill of Materials

| Reference | Component | Value | Package |
|---|---|---|---|
| U1 | MT3608 | Boost converter | SOT-23-6 |
| U2 | AMS1117-3.3 | LDO regulator | SOT-223 |
| J1 | USB-C Receptacle | PowerOnly_6P | — |
| J2 | Screw Terminal | +5V output | 2-pin |
| J3 | Screw Terminal | +3.3V output | 2-pin |
| J4 | Screw Terminal | +12V output | 2-pin |
| L1 | Inductor | 22µH / 2A | SMD |
| D5 | Schottky Diode | SS34 | SMA |
| D1 | LED | Green (3.3V) | 3mm |
| D2 | LED | Green (5V) | 3mm |
| D4 | LED | Green (12V) | 3mm |
| RV1 | Trimmer pot | 100kΩ | Through-hole |
| R1 | Resistor | 1kΩ | 0402 |
| R2,R3 | Resistor | 5.1kΩ (CC pins) | 0402 |
| R4 | Resistor | 470Ω | 0402 |
| R6 | Resistor | 2.2kΩ (FB) | 0402 |
| C1 | Capacitor | 10µF | 0603 |
| C2 | Capacitor | 100µF | Through-hole |
| C3 | Capacitor | 22µF | Through-hole |

## Fabrication Specs

- **Layers:** 2
- **Min trace:** 0.7mm (power)
- **Via drill:** 0.8mm
- **Board size:** ~55 × 35mm

## Part of #30DaysOfPCB Series

This is Day 10 of my daily PCB design challenge.
Every day: one new PCB, one new skill.
Follow on [LinkedIn](https://www.linkedin.com/in/dharahaas-simhadri-b17b4b358?utm_source=share_via&utm_content=profile&utm_medium=member_android)

## Author

**Simhadri Dharahaas** — ECE Undergraduate | PCB Design & Embedded Systems
[LinkedIn](https://www.linkedin.com/in/dharahaas-simhadri-b17b4b358?utm_source=share_via&utm_content=profile&utm_medium=member_android) | [GitHub](https://github.com/dharahaas23)
