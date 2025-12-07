# SiLabs-bgm220-bhi360-bmm350-IMU
Firmware and hardware reference for a BGM220 Explorer Kit + custom SPI board interfacing Bosch BHI360 IMU and BMM350 magnetometer.

## Overview

This repository contains firmware and hardware resources for a **Silicon Labs BGM220 Explorer Kit** used together with a **custom SPI expansion board** hosting a **Bosch BHI360 smart IMU** and an on-board **BMM350 magnetometer**.

The goal of the project is to provide a clean reference design for:
- Bringing up the BHI360 over SPI from the SiLabs BGM220
- Reading fused IMU + magnetometer data (accelerometer, gyroscope, orientation, quaternion)
- Handling the BHI360 FIFO via **polling** (no external interrupt line)
- Exposing sensor data over **VCOM UART (CDC J-Link COM port, 115200 baud)** and optionally over **Bluetooth Low Energy (BLE)** for logging, visualization, or real-time control

The repo includes:
- BGM220 firmware project (SPI + sensor configuration + FIFO polling)
- Pinout and wiring notes for the custom BHI360/BMM350 carrier board
- Example of printing sensor data over the on-board VCOM UART from `app.c`

---

## Enabled BHI360 sensors

The firmware configures the BHI360 over SPI and enables the following virtual sensors via FIFO parse callbacks:

- **Orientation (ORI)** – Fused 3D orientation reported as Euler angles  
- **Accelerometer passthrough (ACC_PASS)** – 3-axis accelerometer data (`int16`)  
- **Gyroscope passthrough (GYRO_PASS)** – 3-axis gyroscope data (`int16`)  
- **Game rotation vector (GEORV)** – Quaternion-based rotation vector  

The BHI360 FIFO is **polled in the main loop**, and the corresponding parse callbacks are invoked when new data is available. Orientation and rotation-vector outputs are produced by the BHI360 sensor fusion engine, which can utilize the external **BMM350 magnetometer** on the custom board (depending on the firmware image and configuration).

---

## Bosch BHI360 firmware & SDK

This project uses the latest official **Bosch Sensortec BHI360 SensorAPI** and firmware image:

- Bosch repo: [BHI360_SensorAPI (master branch)](https://github.com/boschsensortec/BHI360_SensorAPI/tree/master)

We integrate:
- The **latest BHI360 firmware binary** from the repository  
- The matching **SensorAPI / driver sources** into the Silicon Labs BGM220 project  

This keeps the project aligned with Bosch’s reference implementation and simplifies updates to newer firmware/API versions.

---

## Data output (VCOM UART)

Sensor samples are formatted and printed over the **VCOM UART**, which appears on the host PC as a **CDC J-Link COM port**.

- Default UART configuration:
  - **Baudrate:** 115200  
  - **Data bits:** 8  
  - **Parity:** None  
  - **Stop bits:** 1  
- The `printf` path in `app.c` that sends sensor data to the virtual COM port is **enabled by default** (not commented out).
- Any serial terminal (PuTTY, TeraTerm, minicom, etc.) can be used to:
  - Open the J-Link CDC COM port
  - View live IMU/magnetometer data
  - Log the stream to a file for post-processing in Python, MATLAB, etc.
