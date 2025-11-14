# FormSync: A Bimanual Motion Analysis System

> **Project Confidentiality Note**
>
> This is an ongoing capstone project for the University of Calgary, sponsored by Garmin Canada. This project is governed by a Non-Disclosure Agreement (NDA).
>
> This README describes the project's goals, high-level architecture, and the technical challenges being solved. It **does not** contain any proprietary information, sponsor-specific IP, or confidential data from the project.

### 1. Project Overview

FormSync is a wearable sensor system designed to analyze and provide detailed feedback on barbell bench press form. The system uses two independent, IMU-equipped wristbands to capture bimanual (both hands) motion data. This data is streamed to a mobile application, which reconstructs the barbell's 3D path, analyzes it for asymmetries, and provides the user with actionable insights to improve their form and prevent injury.

### 2. The Problem

During compound lifts like the bench press, form errors (such as an uneven barbell path, imbalanced arm extension, or bar drift) are common, especially for beginners or those lifting heavy weight. These errors are difficult for a lifter to self-diagnose and can lead to inefficient lifts, performance plateaus, or serious injury.

### 3. The Solution

Our system is a full-stack solution comprised of two main components:

1.  **Dual Embedded Sensors:** Two custom wristbands, each built on an **nRF52840** SoC with a 9-axis IMU. These run a real-time operating system **(Zephyr RTOS)** and are responsible for high-frequency data sampling, local storage, and BLE communication.
2.  **Mobile Application:** A **React Native** mobile app (for Android/iOS) that serves as the central hub. It manages device connection, runs a per-set calibration, ingests data from both sensors, and presents a post-set analysis to the user.

### 4. High-Level System Flow

1.  **Connect:** The user connects both wristbands to the mobile app via Bluetooth Low Energy (BLE).
2.  **Calibrate:** The app runs a per-set calibration routine, streaming live, low-frequency data to establish a stable "neutral" position before the lift.
3.  **Log:** Once calibrated, the user starts their set. The wristbands log high-frequency IMU data (accel, gyro, mag) locally to on-board SPI flash. This **"log-then-transfer"** model ensures 100% data integrity and is robust against
    BLE connection drops.
4.  **Transfer:** At the end of the set, the app triggers a batch data transfer, receiving the complete, high-integrity data files from both wristbands.
5.  **Analyze:** The app's **Analytic Engine** processes the data. It time-aligns the two streams (using the calibration phase as a reference), runs sensor fusion algorithms to determine orientation, and reconstructs the 3D barbell path.
6.  **Visualize:** A summary dashboard displays the barbell trajectory and key metrics (e.g., rep count, bar path deviation, left/right imbalances) to the user.

### 5. Core Technical Features

#### Embedded System (Firmware)

* **RTOS-Based:** Built on **Zephyr RTOS** for robust real-time performance, thread management, and a built-in BLE stack.
* **Data Integrity (Log-then-Transfer):** We deliberately avoided real-time streaming for set data. By logging to local flash first and transferring as a complete batch, we guarantee that the analysis engine receives a perfect, gap-free dataset, which is critical for accurate path reconstruction.
* **Custom BLE GATT Profile:** A custom GATT service manages the device's complex state machine (e.g., `Idle`, `Calibrating`, `Logging`, `Transferring`) and handles the streaming of calibration data and the batch transfer of set data.
* **Time Synchronization:** The "dual-sensor" sync problem is solved on the mobile app. By recording the calibration data from both devices simultaneously, the app can calculate the precise time offset between the two streams *after* collection and apply it during analysis.

#### Application (Mobile App)

* **Cross-Platform:** Built with **React Native** to target both Android and iOS from a single TypeScript codebase.
* **Modular Data Pipeline:** A clear separation of concerns:
    * **IMU Engine:** Manages all BLE communication, state control, and data ingestion.
    * **Analytic Engine:** An ETL-like pipeline that takes raw sensor files, cleans, transforms (time-aligns, fuses), and processes them into the final metrics.
* **Data Visualization:** Renders a 2D/3D visualization of the barbell's trajectory for each set, allowing users to visually inspect their lift for drift or asymmetry.
* **Offline-First Design:** All user profile and session history is stored in a local on-device database (e.g., SQLite/Realm). This ensures the app is 100% functional in gym basements or areas with poor network connectivity.

### 6. Tech Stack

* **Firmware:** C, Zephyr RTOS, nRF52 SDK, BLE
* **Application:** React Native, TypeScript
* **Hardware:** nRF52840, 9-axis IMU (BMI270 + BMM150), SPI Flash

### 7. My Role (Wristband Development Lead)

As the Wristband Development Lead, my primary responsibilities are on the embedded firmware. This includes:

* Designing and implementing the Zephyr RTOS application.
* Defining the custom BLE GATT profile for the data-transfer state machine.
* Managing the "log-then-transfer" data pipeline by writing/reading from external SPI flash.
* Developing and debugging the device-side logic for calibration and synchronization.
* My app-side work involved implementing the BLE connection management and developing the core data-processing algorithms for aligning the multi-sensor data streams (time-sync) and reconstructing the barbell path.

### 8. Project Status

**Ongoing.** (Expected completion: April 2026)
