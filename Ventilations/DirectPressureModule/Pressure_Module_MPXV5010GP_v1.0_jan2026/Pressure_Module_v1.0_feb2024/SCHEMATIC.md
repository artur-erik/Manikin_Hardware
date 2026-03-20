# Pressure Module v1.0 - Schematic Documentation

## Overview

This module reads pressure from an MPXV5010GP sensor and provides both analog (5V and 3.3V) and digital (I2C) outputs. Designed for the RobotPatient manikin ventilation system.

## Block Diagram

```
                    ┌─────────────────┐
    Pressure ──────►│  MPXV5010GP     │──► A0_PRESSURE_5V
                    │  (0-10 kPa)     │         │
                    └─────────────────┘         │
                                                ▼
                                      ┌─────────────────┐
                                      │ Voltage Divider │──► A0_PRESSURE_3.3V
                                      │  (15K / 10K)    │
                                      └─────────────────┘
                                                │
                                                ▼
                    ┌─────────────────┐    ┌─────────────────┐
                    │   MCP3426       │◄───│  Filter Caps    │
                    │   16-bit ADC    │    │  (470nF, 1µF)   │
                    └────────┬────────┘    └─────────────────┘
                             │
                             ▼ I2C
                    ┌─────────────────┐
                    │   Connector     │──► To RobotPatient
                    │   (J1)          │    Small Module
                    └─────────────────┘
```

## Sensor Section

### MPXV5010GP Pressure Sensor (U$3)

| Pin | Function | Connection |
|-----|----------|------------|
| 1   | NC       | Not connected |
| 2   | VS       | 5V supply |
| 3   | GND      | Ground |
| 4   | VOUT     | A0_PRESSURE_5V |
| 5-8 | NC       | Not connected |

**Specifications:**
- Pressure range: 0 - 10 kPa (0 - 1.45 psi)
- Output: 0.2V to 4.7V (ratiometric to supply)
- Supply voltage: 5V

### Voltage Divider (5V to 3.3V conversion)

For 3.3V MCU compatibility:

```
A0_PRESSURE_5V ──┬── R2 (15K) ──┬── A0_PRESSURE_3.3V
                 │              │
                 └── R1 (10K) ──┴── GND
```

**Calculation:**
- Vout = Vin × R1 / (R1 + R2)
- Vout = Vin × 10K / (10K + 15K) = Vin × 0.4
- Max output: 4.7V × 0.4 = 1.88V (safe for 3.3V ADC)

### Filter Capacitors

| Ref | Value | Purpose |
|-----|-------|---------|
| C3  | 1.0µF | Low-pass filter |
| C4  | 10nF  | High-frequency noise |
| C5  | 470nF | Additional filtering |

## Controller Section

### MCP3426 16-bit ADC (IC2)

| Pin | Function | Connection |
|-----|----------|------------|
| 1   | CH1+     | A0_PRESSURE_5V |
| 2   | CH1-     | GND |
| 3   | VDD      | 3.3V |
| 4   | SDA      | I2C Data |
| 5   | SCL      | I2C Clock |
| 6   | VSS      | GND |
| 7   | CH2+     | NC |
| 8   | CH2-     | GND |

**Features:**
- 16-bit resolution
- I2C interface (address configurable)
- Differential inputs
- Internal reference

### I2C Pull-ups

| Ref | Value | Net |
|-----|-------|-----|
| R29 | 4.7K  | SDA |
| R23 | 4.7K  | SCL |

### Decoupling Capacitors

| Ref | Value | Purpose |
|-----|-------|---------|
| C1  | 0.1µF | High-frequency decoupling |
| C2  | 10µF  | Bulk decoupling |

## Connectors

### J1 - RobotPatient Small Module (5-pin)

| Pin | Signal |
|-----|--------|
| 1   | 5V     |
| 2   | GND    |
| 3   | 3.3V   |
| 4   | SCL    |
| 5   | SDA    |

### AO_PRESSURE Connector (4-pin)

| Pin | Signal |
|-----|--------|
| 1   | 5V     |
| 2   | GND    |
| 3   | 3.3V   |
| 4   | SDA    |

## Bill of Materials

| Ref | Value | Package | Description |
|-----|-------|---------|-------------|
| U$3 | MPXV5010GP | SIP-8 | Pressure sensor 0-10kPa |
| IC2 | MCP3426AD-E | MSOP-8 | 16-bit I2C ADC |
| R1  | 10K | 0603 | Voltage divider |
| R2  | 15K | 0603 | Voltage divider |
| R23 | 4.7K | 0603 | I2C pull-up (SCL) |
| R29 | 4.7K | 0603 | I2C pull-up (SDA) |
| C1  | 0.1µF | 0603 | Decoupling |
| C2  | 10µF | 0805 | Bulk decoupling |
| C3  | 1.0µF | 0603 | Filter |
| C4  | 10nF | 0603 | Filter |
| C5  | 470nF | 0603 | Filter |

## Power Requirements

- **Input voltage:** 5V (from J1)
- **Current consumption:** ~10mA typical
- **3.3V rail:** Derived externally (from host system)
