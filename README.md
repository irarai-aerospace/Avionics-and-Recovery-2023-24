# Avionics-and-Recovery-2023-24
Team Abhyuday (Avionics and Recovery)
The codes for GUI simulation of flight computer data and live telemetry sensing will be added here.

---

## File Summaries

### Root-level Files

| File | Summary |
|------|---------|
| `fc.cpp` | Early/rough flight computer (FC) code for an ESP32-based avionics board. Reads BME280 (altitude/temperature), BNO055 (orientation/angular velocity), and GPS data. Detects flight stages (liftoff, target altitude, apogee, reefing, landing) and transmits telemetry over LoRa using CRC-protected packets. Does **not** include SPI flash data logging. |
| `fc_with_flash.cpp` | Full flight computer code with SPI flash data logging and ESP32 NVS (Preferences) for crash-safe state persistence. Adds pyrotechnic channel control (CH1/CH2 for drogue/reefing events), battery and channel voltage monitoring, and saves all sensor data to an external SPI flash chip. State (liftoff, apogee, etc.) is preserved across resets. |
| `collect_data_from_flash.cpp` | Utility sketch for reading back logged data from the SPI flash chip. Iterates through flash memory, parses CRC-framed records, decodes and prints sensor data over serial. Also includes flash erase and ESP NVS clear functionality for resetting the FC between flights. |
| `ground_station.cpp` | LoRa receiver / ground-station code. Receives CRC-framed telemetry packets from the FC over LoRa, parses both data frames (altitude, orientation, GPS, voltage) and status messages (liftoff, apogee, landed, etc.), and prints decoded values to serial. |
| `sensor_integration.ino` | Combined BME280 + BNO055 integration test sketch. Reads altitude with a moving-average filter and angular velocity from the IMU, printing both to serial. Serves as a basic validation sketch for the two primary sensors. |
| `angles2.ino` | MPU-6050 Kalman filter angle estimation sketch. Reads raw accelerometer and gyroscope data from an MPU-6050 via I2C, applies a 1-D Kalman filter to compute Roll and Pitch angles, and prints them over serial at high speed. |
| `bme.ino` | Standalone BME280 test sketch. Initialises the BME280 sensor and continuously reads and prints temperature, pressure, altitude, and humidity to serial every second. |
| `bmp.ino` | Standalone BMP280/BME280 test sketch using the ErriezBMX280 library. Initialises the sensor at I²C address 0x76 with high oversampling and filter settings, then periodically prints temperature, humidity (BME280 only), pressure, and altitude. |
| `bno055.ino` | Standalone BNO055 test sketch. Initialises the BNO055 IMU and prints the 3-axis angular velocity (gyroscope) vector to serial every 100 ms. |
| `gpstest.ino` | Minimal GPS passthrough sketch. Reads raw NMEA bytes from a GPS module connected on a software serial port and forwards them directly to the hardware serial monitor. |
| `led_buzzer.ino` | Early LED and buzzer flight-stage indicator using BMP280. Uses a moving-average altitude filter to detect liftoff, target altitude (3048 m / 10,000 ft), apogee, reefing altitude (304.8 m / 1,000 ft), and landing, toggling LEDs and buzzer at each stage. |
| `led_buzzer_final.ino` | Refined version of `led_buzzer.ino`. Uses a larger moving-average window (25 samples), tracks max altitude for apogee detection, and references relative altitude from initial value rather than absolute thresholds for liftoff detection. |
| `gui_python_black_bg.py` | Python live-telemetry GUI using Matplotlib. Reads comma-separated sensor data from a serial port (COM5), animates real-time plots of altitude, pitch, roll, yaw, linear acceleration, and temperature on a 2×3 subplot grid with a transparent/black background. Each frame is also saved as a PNG for use in the Unity 3D simulation. |
| `FinalSensorIntegration_2.0.ino` | *(Not included in repository listing but referenced.)* Integrated sensor test combining BME280 and BNO055 readings. |
| `BME_BNO.ipynb` | Jupyter notebook for post-flight data analysis. Processes and visualises BME280 and BNO055 data logged to CSV files. |
| `bp_calc.ipynb` | Jupyter notebook for barometric pressure calculations and altitude analysis. |
| `bnobmedata1.csv` | Raw sensor data log (CSV) from a test flight or bench test, containing BNO055 and BME280 readings (first dataset). |
| `bnobmedata2.csv` | Raw sensor data log (CSV) from a test flight or bench test, containing BNO055 and BME280 readings (second dataset). |

---

### `Final/` Folder

Contains the final, production-ready versions of flight computer firmware and ground support tools.

| File | Summary |
|------|---------|
| `Final/fc.cpp` | Final flight computer firmware. Extends `fc_with_flash.cpp` with dual-core ESP32 task scheduling (one core for sensors, one for logging), the SerialFlash library for SPI flash management, BasicLinearAlgebra for attitude computations, and more robust flight-stage handling with persistent state across power cycles. |
| `Final/fc_clear_memory.cpp` | Utility sketch to fully erase both the external SPI flash chip and the ESP32 NVS (Preferences) storage. Used between flights to reset all persisted flight-state variables. |
| `Final/fc_read_data.cpp` | Firmware to read back all recorded flight data from flash and stream it over serial for retrieval. Supports multiple stored flights, outputs per-flight metadata (initial pressure/temperature, apogee altitude) and the full time-series sensor data ready to be captured by `fc_save_data.py`. |
| `Final/fc_save_data.py` | Python script that initiates a data transfer session with `fc_read_data` over serial, receives the structured flight data, and saves each flight as both a CSV file and a binary pickle (`.dat`) file for later analysis and visualisation. |
| `Final/flash_gui.py` | Python GUI (Tkinter + Matplotlib) for visualising post-flight data stored in the binary `.dat` files produced by `fc_save_data.py`. Displays selectable time-series plots of all recorded parameters (temperature, pressure, filtered altitude, gyro, magnetometer, accelerometer, battery voltage). |
| `Final/ground_station.cpp` | Final ground station receiver firmware. Same structure as the root-level `ground_station.cpp` but matches the updated packet format used by `Final/fc.cpp`. |

---

### `Assets/` Folder

Contains a Unity 3D project used to visualise rocket orientation and telemetry in a real-time 3D simulation.

| File | Summary |
|------|---------|
| `Assets/Rotation.cs` | Unity C# script that reads Euler angle data from a serial port and applies them to a 3D rocket GameObject, providing a live 3D orientation visualisation during flight. |
| `Assets/Plots.cs` | Unity C# script for rendering telemetry plots (altitude, orientation, etc.) inside the Unity scene as an in-engine HUD or overlay. |
| `Assets/UI.cs` | Unity C# script that manages the user interface elements in the Unity simulation, displaying live sensor values on screen. |
| `Assets/scripts/getdata.cs` | Unity C# helper script that opens a serial port connection and continuously reads incoming sensor data, making it available to other Unity scripts (e.g., `Rotation.cs`, `Plots.cs`). |
| `Assets/Rocket.blend` | Blender 3D model file of the rocket, imported into Unity for the 3D visualisation. |
| `Assets/frame.png` | Screenshot/frame image exported by `gui_python_black_bg.py` and imported into Unity as a live-updating texture overlay for the telemetry plots. |
| `Assets/sky.mat` | Unity material file used for the sky/HDRI environment background in the 3D simulation scene. |
| `Assets/DaySkyHDRI026A_4K-HDR.exr` | High dynamic range (HDR) sky environment image used as the lighting and background in the Unity 3D rocket simulation.
