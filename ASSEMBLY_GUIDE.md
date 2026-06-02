# Water Distribution System PCB - Assembly Guide

## Board Specifications
- **Dimensions:** 150mm x 120mm
- **Layers:** 4-layer PCB (FR4, 1.6mm, 1oz copper, ENIG finish)
- **Surface Finish:** ENIG (Electroless Nickel/Immersion Gold)
- **Solder Mask:** Green
- **Silkscreen:** White

## Assembly Sequence

### 1. Bottom Layer Assembly (SMD Components)
- Place MOSFETs (Q2 x9) at positions 110-130mm X, 25-90mm Y
- Place flyback diodes (D2 x9) at positions 115-125mm X, 80-90mm Y
- Place gate resistors (R3 x9) at positions 115-145mm X, 25-35mm Y
- Place decoupling capacitors (C3 x6) at positions 110-130mm X, 45-70mm Y
- Reflow at 260°C peak temperature

### 2. Top Layer Assembly (SMD Components)
- **Power Section (Priority 1):**
  - Place ESP32-WROOM-32E (U1) at 75mm X, 60mm Y
  - Place MP1584EN buck converter (U2) at 40mm X, 80mm Y
  - Place AMS1117-3.3 regulator (U3) at 50mm X, 90mm Y
  - Place reverse polarity PMOS (Q1) at 30mm X, 85mm Y
  - Place 10µF bulk capacitors (C1 x4) at 45-75mm X, 70mm Y
  - Place 22µF capacitors (C4 x2) at 30mm X, 65-75mm Y
  - Place inductor (L1) at 25mm X, 85mm Y
  - Place polyfuse (F1) at 20mm X, 75mm Y
  - Place TVS diode (D1) at 35mm X, 75mm Y

- **Communication Section (Priority 2):**
  - Place RS485 transceiver (U4) at 60mm X, 85mm Y
  - Place 1µF bypass capacitors (C2 x6) at 42-92mm X, 60mm Y
  - Place 100nF decoupling capacitors (C3 x9) at 40-110mm X, 40-70mm Y
  - Place RS485 termination resistors (R5, R6) at 65-75mm X, 95-100mm Y

- **Interface & LED Section (Priority 3):**
  - Place status LEDs (LED1-4) at 15mm X, 40-70mm Y
  - Place LED current-limiting resistors (R4 x4) at 10mm X, 40-70mm Y
  - Place boot and reset buttons (SW1, SW2) at 25mm X, 40-50mm Y
  - Place pull-up resistors (R2 x10) at 45-135mm X, 40mm Y
  - Place 100k bias resistor (R1) at 35mm X, 50mm Y

- **Connectors (Priority 4):**
  - Place USB Micro-B (J1) at 75mm X, 25mm Y
  - Place 12V input terminal blocks (J2 x3) at 10mm X, 20-80mm Y
  - Place RS485 terminal block (J5) at 85mm X, 10mm Y
  - Place soil moisture sensor JST-XH connectors (J6 x8) at 20-65mm X, 10mm Y and 20-65mm X, 115mm Y
  - Place UART expansion header (J7) at 145mm X, 75mm Y

Reflow at 260°C peak temperature.

### 3. Through-Hole Assembly
- Insert and solder power and output terminal blocks from bottom
- Solder mounting holes with M3 standoffs

### 4. Valve Output Terminal Block (J3) & Pump Terminal (J4)
- Mount on bottom layer at 140mm X positions
- Position for easy field access

## PCB Layout Details

### Top Layer (F.Cu)
- Microcontroller and primary components
- Sensor input circuits
- Communication interfaces
- Status indicator LEDs

### Layer 2 (GND)
- Solid ground plane for noise immunity
- Uninterrupted area beneath ESP32
- Via stitching around high-current areas

### Layer 3 (PWR)
- +5V power distribution
- +12V power distribution
- Minimal via placement to reduce noise coupling

### Bottom Layer (B.Cu)
- MOSFET output drivers
- High-current traces (2.5mm for pump, 1.5mm for valves)
- Thermal vias under MOSFETs for heat dissipation

## Trace Width Specifications
- **Signal traces:** 0.25mm minimum
- **Analog sensor traces:** 0.30mm minimum
- **5V power:** 1.0mm minimum
- **12V power:** 1.5mm minimum
- **Pump output:** 2.5mm minimum
- **Valve outputs:** 1.5mm minimum

## Test Points for Verification
- TP1: GND (reference)
- TP2: +3.3V (regulation check)
- TP3: +5V (buck converter output)
- TP4: +12V (input voltage)
- TP5: ESP32 EN pin (reset signal)

## Soldering Profile
- **Preheat:** 160-180°C for 60-120 seconds
- **Thermal Soak:** 180-200°C for 60-120 seconds
- **Peak Temperature:** 245-260°C for 30 seconds
- **Cool Down:** Ramp down at <5°C/second

## Quality Assurance Checklist
- [ ] Visual inspection of all solder joints
- [ ] Continuity testing on all nets
- [ ] Voltage verification at all power rails
- [ ] ESP32 programming test
- [ ] LED functionality test
- [ ] Button functionality test
- [ ] Sensor interface testing
- [ ] RS485 communication test
- [ ] Motor driver output test

## Component Orientation Notes
- **Diodes:** Cathode band toward negative supply
- **Electrolytic Capacitors:** Stripe toward negative
- **MOSFETs:** Pin 1 toward reference marker on silkscreen
- **LEDs:** Flat edge toward negative
- **Terminal Blocks:** Opening faces outward for easy wire insertion

## Manufacturing Notes for LionCircuits
- PCB stackup: L1(35µm Cu) + Dielectric(200µm) + L2(35µm Cu) + Dielectric(200µm) + L3(35µm Cu) + Dielectric(100µm) + L4(35µm Cu)
- Min trace width: 0.25mm
- Min clearance: 0.2mm
- Pad sizes optimized for 0.05mm registration tolerance
- All vias: 0.3mm drill, 0.8mm finished size
- Via tenting on both sides for environmental protection
