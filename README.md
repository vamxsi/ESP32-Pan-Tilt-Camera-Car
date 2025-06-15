# ESP32 WiFi Rover with Pan-Tilt Camera

This repository contains the full source code and documentation for a 4WD WiFi-controlled rover. The rover is built around the ESP32-CAM module and features a web-based interface for remote control and live FPV (First-Person View) video streaming from a pan-tilt camera.


## Features

- **Live Video Streaming:** Real-time FPV video is streamed directly to a web browser over a WebSocket connection.
- **Full Pan-Tilt Control:** The camera can be moved 0-180 degrees horizontally (pan) and vertically (tilt) for a wide field of view.
- **4WD Mobility:** The rover has full directional control (forward, backward, left, right) for excellent maneuverability.
- **Web-Based Control Panel:** The ESP32 hosts a responsive, mobile-friendly web application for all controls and video display. No separate app is needed.
- **Onboard Flashlight:** The ESP32-CAM's powerful built-in LED can be toggled remotely for low-light conditions.
- **Variable Speed Control:** Adjust the rover's speed using a slider in the web interface for precise movements.

## Hardware Components

| Component | Quantity | Purpose |
| --- | --- | --- |
| ESP32-CAM Module | 1 | The main controller for processing, camera functions, and WiFi. |
| Pan-Tilt Servo Kit | 1 | The mechanical assembly for camera movement. |
| SG90 Micro Servos | 2 | Drives the pan and tilt actions. |
| 4WD Rover Chassis Kit | 1 | The robot's body, motors, and wheels. |
| L298N Motor Driver | 1 | Controls the four DC motors of the chassis. |
| 5V UBEC or Buck Converter | 1 | Provides a stable, high-current 5V supply for the servos. |
| 2S LiPo Battery (7.4V) | 1 | The primary power source for the entire rover. |
| FTDI Programmer (or Arduino Uno) | 1 | Required to upload firmware to the ESP32-CAM. |
| Jumper Wires & Mounting Tape | - | For wiring and securing components. |

## Circuit & Wiring

A stable power distribution is critical for this project. The servos require a dedicated 5V power source from the UBEC to prevent voltage drops that would crash the ESP32-CAM.

*You can add a Fritzing diagram or a hand-drawn schematic to the `docs/` folder and link it here:*
`![Circuit Diagram](docs/circuit_diagram.png)`

**Pinout Connections:**

| From Component | To Component | Pin |
| :--- | :--- | :--- |
| **L298N Motor Driver** | ESP32-CAM | `IN1` -> `GPIO 12` |
| | | `IN2` -> `GPIO 13` |
| | | `IN3` -> `GPIO 15` |
| | | `IN4` -> `GPIO 14` |
| | | `ENA` (Speed A) -> `GPIO 2` |
| | | `ENB` (Speed B) -> `GPIO 2` |
| | 5V Output Pin | ESP32-CAM `5V` Pin (for logic power) |
| **Servos** | | |
| Pan Servo Signal | ESP32-CAM | `GPIO 16` |
| Tilt Servo Signal | ESP32-CAM | `GPIO 4` |
| Both Servos VCC (Red) | UBEC/Buck Converter | 5V Output |
| Both Servos GND (Brown) | UBEC/Buck Converter | GND Output |
| **ESP32-CAM Flash LED** | ESP32-CAM | `GPIO 4` |

*(Note: The `Flash LED` shares a pin with the `Tilt Servo`. When the flashlight is on, the tilt servo may twitch. This is a hardware limitation of the default pin usage.)*

## Software Setup

### 1. Arduino IDE Configuration
- Install the latest Arduino IDE.
- Open **File > Preferences** and add the following URL to "Additional Board Manager URLs":
  `https://dl.espressif.com/dl/package_esp32_index.json`
- Open **Tools > Board > Boards Manager**, search for "esp32", and install the package by Espressif Systems.

### 2. Required Libraries
Install the following libraries in the Arduino IDE:
- `ESP32Servo` (Install from **Tools > Manage Libraries...**)
- `AsyncTCP` ([Download .ZIP](https://github.com/me-no-dev/AsyncTCP/archive/refs/heads/master.zip))
- `ESPAsyncWebServer` ([Download .ZIP](https://github.com/me-no-dev/ESPAsyncWebServer/archive/refs/heads/master.zip))

### 3. Critical Performance Optimization
For smooth, real-time video streaming, you must modify one of the web server library files.
1.  Navigate to your Arduino libraries folder (usually `Documents/Arduino/libraries/`).
2.  Open the `ESPAsyncWebServer` folder, then the `src` folder.
3.  Edit the file `AsyncWebSocket.h`.
4.  Find the line `#define WS_MAX_QUEUED_MESSAGES 16` and change the value to `1`.
    ```
    // This optimization prevents video lag by only sending the most recent frame.
    #define WS_MAX_QUEUED_MESSAGES 1
    ```
5.  Save the file.

## Usage Guide

### 1. Uploading the Firmware
The ESP32-CAM requires an external programmer.
1.  Connect the ESP32-CAM to your FTDI programmer or Arduino Uno.
2.  **Important:** Ground the `IO0` pin on the ESP32-CAM to enable flashing mode.
3.  In the Arduino IDE, select the **"AI Thinker ESP32-CAM"** board.
4.  Select the correct COM port.
5.  Click **Upload**. After the upload begins, you can disconnect `IO0` from ground.

### 2. Operating the Rover
1.  Power the rover with the LiPo battery.
2.  On your phone or computer, search for WiFi networks.
3.  Connect to the SSID: **`My_WiFi_Car`** with the password: **`12345678`**.
4.  Open a web browser and go to the IP address: **`http://192.168.4.1`**
5.  The control interface will load, and the video stream will start automatically.

## Troubleshooting

| Problem | Solution |
| :--- | :--- |
| **Video is lagging or stuttering** | Ensure you have completed the **Critical Performance Optimization** step and set `WS_MAX_QUEUED_MESSAGES` to `1`. |
| **No video stream appears** | Refresh your browser page. Disconnect and reconnect to the rover's WiFi network. |
| **Firmware upload fails** | Check all wiring. Confirm that `IO0` is securely connected to `GND` before you press upload. |
| **Servos are jittery or reset the ESP32** | The servos are not getting enough power. Verify they are powered by a dedicated UBEC providing at least 5V/2A. Do not power them from the L298N or ESP32. |

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.
