# 🚗 ThermoGuard AI: Smart Automotive Climate Control System

![Python](https://img.shields.io/badge/Python-3.9+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)
![C/C++](https://img.shields.io/badge/C%2FC%2B%2B-00599C?style=for-the-badge&logo=cplusplus&logoColor=white)
![Arduino](https://img.shields.io/badge/Arduino-00979D?style=for-the-badge&logo=arduino&logoColor=white)
![AVR](https://img.shields.io/badge/AVR-ATmega328P-red?style=for-the-badge)

An intelligent, multimodal automotive climate control platform that dynamically optimizes cabin temperature, fan speed, and air airflow distribution based on real-time vision occupant detection, hands-free voice commands, and closed-loop hardware thermal sensing.

---

## 📌 Features

- **Occupant-Aware Metabolic Fan Boosting**: Detects cabin occupants using an onboard camera pipeline and automatically applies a **+15% PWM metabolic boost per person** (+38 PWM) to the HVAC fan motor.
- **Dynamic Servo Zone Targeting**: Directs airflow toward occupied seating zones (Left, Center, Right, or multi-zone sweeps) based on passenger spatial positioning.
- **Hands-Free Speech Control**: Processes natural voice commands for target temperature set-points (18°C–40°C), Turbo Mode toggles, and manual louver swing controls via speech recognition.
- **Native C AVR Firmware**: Runs bare-metal C control firmware on an ATmega328P microcontroller utilizing direct register access for Timer2 PWM, ADC sampling, and hardware UART.
- **Emergency Overheat Protection**: Automatically triggers system lockdown, visual/auditory alarms, and motor shutdown if cabin temperatures exceed 50°C.
- **Power-Saving ECO Mode**: Dims the 16x2 LCD backlight and lowers system power consumption after prolonged user inactivity.

---

## 🧠 System Components & Control Modes

### 1. Computer Vision Pipeline (`ThermoGuard_AI_Vision.py`)
- **Dual Detection Engine**: Combines HOG + Linear SVM body detection with Haar Cascade face detection to ensure high accuracy for close-up seated passengers.
- **Non-Maximum Suppression (NMS)**: Eliminates redundant bounding box detections around passengers.
- **Temporal Voting Buffer**: Smooths frame-to-frame occupant counts across a 5-frame rolling window to prevent control flickering.
- **Zone Multi-Sweep**: Divides the video frame into Left, Center, and Right zones, sending range sweep coordinates (`LL`, `LC`, `LR`, `CC`, `RR`) over serial.

### 2. Speech Recognition Module (`ThermoGuard_Voice.py`)
- **Natural Language Parsing**: Maps spoken phrases to discrete hardware state updates.
- **Temperature Range Commands**: Accepts direct temperature requests from `temperature 18` to `temperature 40`.
- **Mode Control**: Toggles `turbo mode on/off` and `swing on/off` dynamically.

### 3. AVR ATmega328P Microcontroller Core (`firmware/`)
- **Closed-Loop Thermal Regulation**: Calculates temperature error ($T_{current} - T_{target}$) via an LM35 sensor and adjusts motor PWM across COOL, HEAT, IDLE, and TURBO states.
- **Analog Keypad Interface**: Reads a 5-button voltage ladder on ADC Channel 0 for manual overrides.
- **Custom 16x2 LCD Driver**: Displays real-time telemetry, animated fan status graphics, set-points, and AI occupant count indicators in 4-bit mode.

---

## 📊 Serial Communication Protocol Reference

The laptop and microcontroller communicate over UART (`COM7` @ 9600 Baud) using a lightweight byte sequence:

| Stream Source | Byte Sequence | Purpose / Command Action | Microcontroller Reaction |
|---|---|---|---|
| **Camera Vision** | `[Count] [ZoneStart] [ZoneEnd]` | 3-Byte Occupant Data (e.g., `'2'`, `'L'`, `'R'`) | Applies metabolic PWM boost and sweeps servo between `ZoneStart` & `ZoneEnd` |
| **Voice Module** | `'T'` / `'t'` | Turbo Mode ON / OFF | Sets target temp to 16°C @ 100% PWM / Restores 24°C |
| **Voice Module** | `'S'` / `'s'` | Louver Swing ON / OFF | Toggles full manual 180° servo sweep |
| **Voice Module** | `'W'` `[18–40]` | Set Target Temperature | Overrides target set-point to specified integer value |
| **System Heartbeat** | `'H'` | Serial Health Check | Resets AI link watchdog timer |

---

## 🛠️ System Architecture & Workflow

```text
┌───────────────────────────┐     ┌──────────────────────────────────┐
│   Webcam Video Stream     │ ──> │ OpenCV / Haar + HOG Detector     │ ──┐
└───────────────────────────┘     └──────────────────────────────────┘   │
                                                                         │
┌───────────────────────────┐     ┌──────────────────────────────────┐   |   ┌──────────────────────────────────────────┐
│   Microphone Audio Input  │ ──> │ Python Speech Recognition        │ ──┤─> │ Native AVR UART Driver (`Serial_AI`)     │
└───────────────────────────┘     └──────────────────────────────────┘   │   └──────────────────────────────────────────┘
                                                                         │                        │
┌───────────────────────────┐                                            │                        ▼
│ LM35 Sensor & Keypad      │ ───────────────────────────────────────────┘   ┌──────────────────────────────────────────┐
│ (Analog ADC Inputs)       │                                                │ ATmega328P Core Logic (`ThermoGuard_AC`) │
└───────────────────────────┘                                                └──────────────────────────────────────────┘
                                                                                                  │
                                                                                                  ▼
                                                                             ┌──────────────────────────────────────────┐
                                                                             │ Actuators: DC Fan PWM, Servo Steering,   │
                                                                             │ 16x2 LCD Display, Alert Buzzers / LED    │
                                                                             └──────────────────────────────────────────┘
```

---

## ⚙️ Tech Stack & Hardware Components

- **Programming Languages:** Python 3.9+, Native C / C++ (AVR-GCC)
- **Computer Vision & Speech:** OpenCV, NumPy, PySerial, SpeechRecognition, PyAudio
- **Microcontroller Hardware:** Arduino Uno (Microchip AVR ATmega328P core)
- **Driver Shields & Displays:** Arduino Motor Shield (L298N/Timer2 PWM), 16x2 Character LCD Keypad Shield
- **Sensors & Actuators:** LM35 Precision Centigrade Temperature Sensor, SG90 Micro Servo Motor, 12V DC HVAC Fan Motor, Piezo Buzzers

---

## 🚀 How to Run the System

### 1. Hardware Assembly Setup
1. Stack the **Arduino Motor Shield** onto the **Arduino Uno**.
2. Mount the **16x2 LCD Keypad Shield** directly on top of the motor shield.
3. Connect the **LM35 Temperature Sensor** to Analog Pin `A2`, the **SG90 Servo** to Pin `A1`, and the **DC Fan Motor** to the motor driver terminal outputs.

### 2. Microcontroller Firmware Flash
Compile and upload the firmware files inside `firmware/` using Arduino IDE or Microchip Studio:
- Open `firmware/ThermoGuard_AC.ino`
- Select **Arduino Uno (ATmega328P)** target board and upload.

### 3. Python Host Services Setup
Clone the repository and install required laptop dependencies:

```bash
git clone https://github.com/El-adghamF90/ThermoGuard_AI.git
cd ThermoGuard_AI
pip install opencv-python numpy pyserial speechrecognition pyaudio
```

### 4. Launching the Control Modules

#### Run Computer Vision Occupant Tracking:
```bash
python python/ThermoGuard_AI_Vision.py
```

#### Run Speech Recognition Voice Interface:
```bash
python python/ThermoGuard_Voice.py
```

---

## 📂 Project Structure

```text
ThermoGuard_AI/
│
├── README.md                    # Project Documentation
│
├── python/
│   ├── ThermoGuard_AI_Vision.py # Real-time occupant vision detection & UART messaging
│   └── ThermoGuard_Voice.py     # Speech recognition voice command module
│
└── firmware/
    ├── ThermoGuard_AC.ino       # Main application entry point & super-loop logic
    ├── Hardware_Defs.h          # Hardware pin mappings, timer constants & thresholds
    ├── Serial_AI.h              # UART communication driver header
    ├── Serial_AI.ino            # Non-blocking UART parser implementation
    ├── Adc.h                    # ADC peripheral driver header
    ├── Adc.ino                  # ADC sampling & LM35 temperature translation
    ├── Keypad.h                 # Analog keypad voltage ladder header
    ├── Keypad.ino               # Keypad button decode implementation
    ├── Lcd.h                    # 16x2 LCD display driver header
    ├── Lcd.ino                  # LCD 4-bit mode communication & custom characters
    ├── Motor.h                  # DC motor PWM & bit-banging servo driver header
    └── Motor.ino                # Timer2 Fast PWM & servo pulse generation
```

---

## 🔄 System Execution Workflow

```text
Power On & Initialization (Adc_Init, Motor_Init, Lcd_Init, Serial_AI_Init)
   ↓
Read Analog Sensors (LM35 Temperature & Keypad State)
   ↓
Parse Serial Data (Camera Person Count / Zones & Voice Commands)
   ↓
Evaluate Safety Interlocks (Emergency Lockdown if Temp > 50°C)
   ↓
Compute Thermal Speed State (COOL / HEAT / IDLE / TURBO)
   ↓
Apply AI Metabolic Boost (+15% PWM per detected occupant)
   ↓
Update Actuators (Timer2 DC Motor PWM & Servo Zone Target Steering)
   ↓
Refresh LCD Telemetry & Fan Graphics
```

---

## 👥 Authors & Team

- **Mahmoud El-Adgham** — System Integration, Microcontroller Control Logic & AI Link Driver
- **Ziad Khalil** — Embedded Drivers & Hardware Circuit Integration
- **Ali Akram Ali** — Computer Vision & Speech Recognition Modules

---

## 📜 Certification & Context

Project developed as an engineering capstone demonstrating multimodal AI system integration across real-time computer vision, voice automation, and bare-metal embedded C firmware.

---

## 👨‍💻 Primary Author

**Mahmoud El-Adgham**  
Electrical & Electronic Engineering | AI & Machine Learning  

* **GitHub:** [El-adghamF90](https://github.com/El-adghamF90)
* **LinkedIn:** [Mahmoud EL-Adgham](https://www.linkedin.com/in/mahmoud-el-adgham/)
