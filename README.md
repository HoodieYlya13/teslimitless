# TesLimitless ⚡

> **Tesla Model 3 Highland Hardware & Software Integration**  
> _A 4th-Year Engineering Project restoring physical control stalks and enabling low-level steering wheel button remapping via an ESP32 LIN-bus gateway and React Native companion app._

---

## 📌 Project Overview

The **Tesla Model 3 Highland** famously removed all physical steering column stalks (turn signals, high beams, and wipers), replacing them with controversial capacitive touch buttons on the steering wheel. 

**TesLimitless** is a premium, non-invasive hardware-software retrofitting platform designed to:
1. **Restore Physical Control Stalks:** Add custom-engineered physical turn-signal and high-beam stalks back onto the steering column.
2. **Repurpose Steering Wheel Buttons:** Intercept and remap the now-redundant factory capacitive buttons to perform custom vehicle macros (e.g., pressing the original capacitive high-beam button to toggle Comfort/Sport acceleration modes or passenger controls).
3. **Dynamic Customization & Telemetry HUD:** Manage and configure these button mappings on the fly via a landscape React Native companion app over Bluetooth (BLE) that doubles as a real-time cockpit Heads-Up Display (HUD).

By acting as an inline physical **LIN-bus Man-in-the-Middle (MITM) gateway** on the steering wheel harness and injecting custom commands via the **CAN-bus diagnostic port**, TesLimitless achieves this high-impact customization safely and seamlessly.

![System Schema](https://hy13dev.com/img/teslimitless-schema.png)

---

## 🏗️ System Architecture

The project is split across two core domains to ensure high performance, stability, and absolute safety:

### 1. Dual-Core Embedded Firmware (`/esp32`)

Powered by an ESP32 microcontroller running **FreeRTOS** with asymmetric dual-core processing:

- **Core 1 (Safety-Critical & LIN-bus):** Dedicated exclusively to the active steering wheel LIN-bus MITM transceiver gateway. When a capacitive touch button or scroll wheel event is polled, Core 1 intercepts the frame in sub-1.5ms, sends a mock "idle" response to the vehicle's body controller to silent standard actions, and queues custom commands for injection.
- **Core 2 (Networking & CAN-bus):** Handles high-throughput vehicle telemetry reading and custom frame injection over CAN-bus via the vehicle's diagnostic port. Simultaneously manages the low-overhead BLE GATT server to stream dashboard telemetry and receive custom configuration remapping tables from the companion app.

### 2. Custom HUD & Companion App (`/mobile`)

A high-performance **React Native** application built to serve as a minimalist, dark-themed landscape HUD and customization controller:

- **Real-time Telemetry Dashboard:** Connects over BLE to display real-time speed, battery percentage, range status, gear selector, turn signals, and a dynamic SVG power/regen visualization arc.
- **Customization Mapping Hub:** Provides a steering wheel customization interface to dynamically bind custom event macros directly to vehicle hardware inputs (for example, remapping a capacitive left turn signal button touched/pressed event to toggle the vehicle's comfort mode profile). Custom mappings are serialized and streamed over BLE to the ESP32.

---

## 🔒 Repository Submodules

To protect the hard-won reverse-engineering schemas, raw protocol DBC files, and proprietary bus intercept coordinates, this repository organizes the code into **two private git submodules**:

- 📂 **[`esp32`](https://github.com/HoodieYlya13/teslimitless-esp32)**: Asymmetric dual-core FreeRTOS C++ firmware, transceiver configurations, and LIN/CAN intercept drivers.
- 📂 **[`mobile`](https://github.com/HoodieYlya13/teslimitless-native)**: React Native BLE ecosystem, state management for telemetry, and the premium vector-based landscape dashboard HUD.

---

## 🏎️ Key Features

### 📡 Reverse Engineering & Protocols

- **CAN-bus Sniffing:** Real-time decoding of high-speed vehicle state indicators (speedometer, state of charge, odometer, gear selector, turn signal indicators).
- **LIN-bus Interception:** Master-slave sniffing and active command injection to capture button presses from the steering wheel and inject custom commands back to the system.

### ⚙️ Asymmetric Dual-Core Embedded System

- Utilizes **FreeRTOS** tasks bound to specific ESP32 cores.
- A strict hardware watchdog architecture ensures that even if wireless networking tasks on Core 2 stall, Core 1's safety-critical LIN loop continues running continuously with microsecond response times.

### 📐 CAD & Mechanical Retrofitting

- **Physical Stalk Restoration:** Custom-designed, prototyped, and manufactured a highly functional mechanical turn-signal and high-beam stalk retrofitted directly onto the Highland steering column.
- **OEM-Grade Integration:** Designed a custom 3D-printed enclosure fitting seamlessly behind the steering wheel to house the ESP32, dual transceivers, and voltage regulators with factory-level aesthetics.

### 📱 Wireless Telemetry Ecosystem

- Ultra-low latency connection using Bluetooth Low Energy (BLE).
- Optimized data payloads utilizing compressed byte structures to maximize throughput and minimize HUD display latency (< 16ms update loop).

---

## 📱 HUD Interface in Action

The custom React Native application is engineered with a gorgeous, high-contrast HUD design suitable for vehicle cockpits, preventing eye strain during night driving while offering maximum legibility.

<p align="center">
  <img src="https://hy13dev.com/img/teslimitless-app.png" alt="HUD Application Screen" width="600px" style="border-radius: 12px; box-shadow: 0 4px 20px rgba(0,0,0,0.5);" />
</p>

---

## 🛠️ Hardware Stack & Components

- **Microcontroller:** ESP32-WROOM-32D (Dual-Core)
- **CAN-bus Transceiver:** MCP2515 CAN Bus Module TJA1050 receiver
- **LIN-bus Transceiver:** 2x TJA1021 (Master and Slave Node Interceptors)
- **Power Management:** 12V-to-5V step-down regulator tapped from the vehicle's accessory rail.
