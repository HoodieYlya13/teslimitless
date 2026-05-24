# TesLimitless ⚡

> **Tesla Model 3 Highland Hardware & Software Integration**  
> _A 4th-Year Engineering Project focusing on protocol reverse engineering, real-time telemetry HUD, and low-level steering/LIN button customization._

---

## 📌 Project Overview

**TesLimitless** is an advanced vehicle-integration platform designed to intercept, decode, and customize the communication networks of a **Tesla Model 3 Highland**. By tapping into the car’s internal network (strictly reading and writing CAN-bus frames non-invasively through the **Tesla diagnostic port** and acting as an inline physical **LIN-bus Man-in-the-Middle** gateway on the steering wheel control harness), this project delivers a custom-designed real-time Heads-Up Display (HUD) and physical customization controls (such as turn signal and scroll wheel remapping) without interfering with safety-critical powertrain systems.

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

### 📐 CAD & Rapid Prototyping

- Designed, iterated, and manufactured a fully functional, **3D-printed mechanical turn signal stalk integration**.
- Custom mount housing for transceivers, voltage regulators, and the ESP32 module, fitting seamlessly behind the steering column.

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
