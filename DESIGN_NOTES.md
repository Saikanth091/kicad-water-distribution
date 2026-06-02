# AI-Monitored Water Distribution System - Design Notes

## Project Overview
Industrial-grade PCB for automated agricultural water distribution with AI monitoring capabilities. ESP32-WROOM-32E-based controller with 8 soil moisture sensors and 9 valve outputs.

## Design Considerations

### Power Management
1. **Input Voltage Protection**
   - 12V DC input with solar/battery backup options
   - AO3401A PMOS for reverse polarity protection
   - 2A polyfuse for short-circuit protection
   - SMBJ15A TVS diode for transient suppression

2. **Power Conversion**
   - MP1584EN buck converter: 12V → 5V @ 2A
   - Output LC filter: 10µH inductor + 10µF capacitor
   - AMS1117-3.3: 5V → 3.3V @ 1A for ESP32
   - Linear regulator for low-noise 3.3V to analog circuits

3. **Decoupling Strategy**
   - 100nF ceramic capacitors per IC pin
   - 10µF bulk capacitors at power entry points
   - 22µF electrolytic capacitors for power supply filtering
   - Low-ESR capacitors prioritized for switching noise rejection

### Microcontroller Interface
1. **ESP32-WROOM-32E**
   - Native WiFi (2.4GHz) with keepout zone around antenna
   - Boot and Reset buttons with RC debouncing (10k pullups)
   - Status LEDs: Power (Red), WiFi (Green), Pump (Blue), Valve (Yellow)
   - Crystal oscillator: 40MHz (integral to module)

2. **GPIO Pin Assignments**
   - **Analog Inputs:**
     - GPIO34, 35, 36, 39: High-impedance ADC inputs (attenuation 11dB for 0-3.6V)
     - GPIO32, 33, 25, 26: Standard ADC inputs
   - **Digital Inputs:**
     - GPIO4: DHT22 temperature/humidity sensor
     - GPIO27: Rain sensor detection
     - GPIO14: Flow meter pulse counter
     - GPIO12: Tank level analog input
   - **Digital Outputs:**
     - GPIO23: Pump motor drive (high-current)
     - GPIO16-22, 13, 15: Solenoid valve drives (multiplexed)

### Sensor Interface Design
1. **Capacitive Soil Moisture Sensors**
   - 8x 3-pin JST-XH connectors (VCC, GND, Signal)
   - RC low-pass filters: 1k resistor + 100nF capacitor (fc ≈ 160kHz)
   - Individual 100nF decoupling at each sensor input
   - Pull-down resistors on ADC inputs (1M to ground)
   - 0-3.3V signal range conditioned by firmware

2. **DHT22 Temperature/Humidity**
   - Single-wire digital protocol on GPIO4
   - 10k pullup resistor to 3.3V
   - Timing-critical: requires precise GPIO timing
   - Firmware uses DHT library with error handling

3. **Rain Sensor**
   - Analog output (0-3.3V) on GPIO27
   - 10k external pullup resistor
   - Hysteresis implemented in firmware

4. **Flow Sensor**
   - Pulse output on GPIO14
   - Frequency: 1-8 kHz typical
   - Timer interrupt for pulse counting
   - Calibration: pulses per liter (PPL) in firmware

5. **Tank Level Sensor**
   - 0-10VDC analog sensor stepped down via voltage divider
   - Divider: 2.2k top + 1k bottom = 0-3.3V at ADC
   - 100nF filtering at ADC input

### Output Driver Architecture
1. **MOSFET Driver Stage**
   - AO3400A N-channel MOSFETs (9 total)
   - Gate resistors: 100Ω (limits dI/dt)
   - Pull-down resistors: 10k to GND (prevents floating gate)
   - Flyback diodes: SS14 Schottky (1A, 45V reverse)
   - Output inductance: 5-10µH typical in wiring

2. **Output Specifications**
   - **Pump Output (GPIO23):**
     - Max drain-source voltage: 24V (TVS clamped to +12V + diode drop)
     - Max continuous current: 2A (determined by polyfuse and MOSFET Rds(on))
     - PWM capable for speed control
   
   - **Valve Outputs (GPIO16-22, 13, 15):**
     - Max drain-source voltage: 24V
     - Max continuous current per valve: 0.5A (8 × 0.5A = 4A total with time-multiplexing)
     - Fast switching: trise ≈ 50ns, tfall ≈ 100ns
     - Dead-time insertion in firmware to prevent cross-conduction

3. **Thermal Considerations**
   - All MOSFETs mounted on bottom layer
   - Thermal vias under MOSFET drain pads (minimum 0.3mm drill, grid spacing 1mm)
   - Copper pour on bottom layer for heat spreading
   - Estimated junction temperature @ full load: <85°C with board-level cooling

### Communication Interfaces
1. **RS485 (Modbus RTU)**
   - MAX3485 transceiver IC (half-duplex)
   - Differential pair: A/B lines
   - Terminal block: 3-pin (A, B, GND)
   - Termination resistors: 120Ω (optional, firmware-controlled)
   - Baud rate: 9600-115200 bps configurable
   - Max distance: 1.2km (with proper cable shielding)

2. **UART Expansion Header**
   - 2×5 pin header for modular connectivity
   - Pins: TX, RX, 3.3V, GND (+ 1 spare)
   - Compatible with:
     - GSM modules (SIM800H, SIM7600)
     - LoRa modules (SX1276-based)
     - Raspberry Pi GPIO header (with voltage level shifting)
   - Software serial (2nd UART) implementation option

3. **WiFi**
   - Onboard antenna on ESP32-WROOM-32E
   - Keepout zone: 20mm × 20mm around antenna
   - No ground plane under antenna area
   - No high-impedance traces near antenna

### PCB Stack-Up & Layer Arrangement
```
Layer 1 (F.Cu - Top):    Signals, components, micro-vias
Dielectric 1:             200µm FR4 (Tg 140°C)
Layer 2 (In1.Cu):         Ground plane (continuous)
Dielectric 2:             200µm FR4
Layer 3 (In2.Cu):         Power distribution (+5V, +12V)
Dielectric 3:             100µm FR4
Layer 4 (B.Cu - Bottom):  Signals, high-current traces
```

### Trace Routing Strategy
1. **High-Current Paths**
   - 12V input to buck converter: 1.5mm traces, minimal length
   - 5V from buck to regulator: 1.0mm traces
   - 3.3V to ESP32: ≥0.25mm traces with multiple vias
   - Pump drive: 2.5mm traces on bottom layer
   - Return paths: Direct to ground plane via stitching vias

2. **Analog Signal Integrity**
   - Sensor input traces: 0.30mm minimum width
   - 100nF capacitors within 5mm of ADC inputs
   - Separate analog and digital grounds (star-point at input)
   - ADC input traces routed on inner layers when possible
   - Shield traces with ground to minimize capacitive coupling

3. **Signal Integrity**
   - RS485 differential pair: 0.25mm traces, length-matched ±10mm
   - SPI clock rate: 1MHz max (8MHz internal divider)
   - Impedance: 50-100Ω for controlled propagation

### EMI/RFI Mitigation
1. **Conducted Emissions**
   - Ferrite bead on 12V input: Z=1kΩ @ 100MHz
   - RC snubber networks on MOSFET outputs (optional)
   - Bulk capacitors positioned near noise sources

2. **Radiated Emissions**
   - Ground planes on layers 2 & 3 for shielding
   - Component placement: sensitive circuits away from MOSFETs
   - WiFi antenna in corner to minimize board coupling
   - Shield can on RS485 IC (optional)

3. **ESD Protection**
   - TVS diodes on external connectors
   - 100nF capacitors on all IC power pins
   - Slow edge rates on digital outputs

## Manufacturing Specifications

### Tolerances & Capabilities
- **Minimum trace width:** 0.25mm (6mil)
- **Minimum clearance:** 0.2mm (8mil)
- **Via diameter:** 0.3mm drill, 0.8mm finished pad
- **Annular ring:** 0.15mm minimum
- **Pad-to-mask clearance:** 0.05mm nominal
- **Solder mask registration:** ±0.1mm

### Supplier Recommendations
- **PCB:** LionCircuits (FR4, 1.6mm, ENIG, 4-layer)
- **Assembly:** JLC PCB or NextPCB (PCBA service)
- **Component sourcing:** Digikey, Mouser, LCSC

## Testing & Verification

### Pre-Power Verification
1. Resistance measurements:
   - 12V to GND: >10MΩ (with Q1 reverse-biased)
   - 5V to GND: >1MΩ
   - 3.3V to GND: >100kΩ (after bulk capacitor charging)

2. Visual inspection:
   - No solder bridges on BGA or fine-pitch components
   - All capacitors oriented correctly
   - Test points accessible and labeled

### Functional Testing
1. Power-up sequence:
   - 12V input applied → 5V output should reach 4.75-5.25V within 500ms
   - 3.3V should reach 3.135-3.465V within 1s
   - ESP32 EN pin high after 100ms

2. ESP32 verification:
   - Serial bootloader detection over USB @ 115200 bps
   - GPIO continuity testing
   - ADC voltage measurements

3. Peripheral testing:
   - Soil moisture sensor readings: 0-1024 ADC counts
   - DHT22 data format validation
   - Valve driver output pulse verification
   - RS485 communication loopback

## Firmware Architecture (Reference)
- **Framework:** Arduino IDE for ESP32
- **Key libraries:** DHT22, Modbus, WiFi, SPIFFS (file system)
- **Task scheduling:** FreeRTOS kernel (built into Arduino core)
- **ADC sampling:** DMA-assisted multi-channel sampling @ 1kHz
- **Cloud integration:** MQTT over WiFi for remote monitoring

## BOM Cost Estimate
- **Microcontroller & IO:** ~$12
- **Power management:** ~$5
- **Sensors & connectors:** ~$8
- **Output drivers:** ~$4
- **Passives & misc:** ~$3
- **PCB manufacturing:** ~$20-50 (qty 1-10 units)
- **Total component cost (qty 100):** ~$20-25 per unit

## Revision History
- v1.0 (2026-06-02): Initial design with 8 soil sensors, 9 outputs, RS485 communication

## Future Enhancements
- LoRa module integration for long-range connectivity
- Secondary LiPo battery backup with charging circuit
- CAN bus interface for fleet management
- SD card slot for local data logging
- 4G modem support for remote areas
