# Smart Room Automation System

This project implements a smart room automation system using an Arduino Uno, featuring predefined password-protected secure door access, automatic lighting based on occupancy, and automatic smoke detection with an alarm. The system prioritizes smoke detection over unauthorized access for safety.

## Project Overview
- **Purpose**: Automate room functions (secure door, occupancy-based lighting, smoke detection) with sensor-based control.
- **Platform**: Designed and simulated in Tinkercad.

## Tinkercad Link
https://www.tinkercad.com/things/3gTcExSN1LN-smart-room 

## Components and Connections
### Hardware Components
- **Arduino Uno**: Microcontroller (central unit).
- **Servo Motor**: Pin 3 (controls door lock, 0° = locked, 90° = unlocked).
- **LEDs**: 
  - Red LED (Pin 5): Indicates incorrect password.
  - Green LED (Pin 6): Indicates access granted.
- **Buzzer**: Pin 4 (audible alarm for smoke detection).
- **Gas Sensor (MQ-2)**: Analog pin A0 (detects smoke, threshold >600 for unsafe level).
- **PIR Sensors**: 
  - PIR1 (Pin A3): Detects entry (outside).
  - PIR2 (Pin A2): Detects exit/inside movement.
- **Light**: Pin A1 (turns on/off based on occupancy).
- **DC Motor (Exhaust Fan)**: Pin 4 (via 2N2222 (npn) transistor with 1kΩ resistor, activates with smoke).
- **Keypad (4x4)**: Pins 7-13 and 2 (for password entry).
- **LCD (16x2 with I2C)** (Type [PCF8574], [39 (0X27)]): Default SDA/SCL (A4/A5) (displays status, occupancy, and messages).

### Connection Details
1. **Servo Motor**: Connect signal to Pin 3, VCC to 5V, GND to GND.
2. **LEDs**: Connect anodes to Pins 5 and 6, cathodes to GND via 220Ω resistors.
3. **Buzzer**: Connect positive to Pin 4, negative to GND.
4. **Gas Sensor**: Connect VCC to 5V, GND to GND, AOUT to A0.
5. **PIR Sensors**: Connect VCC to 5V, GND to GND, OUT to A3 (PIR1) and A2 (PIR2).
6. **Light**: Connect anode to A1, cathode to GND via 220Ω resistor.
7. **DC Motor (Exhaust Fan)**: Connect positive to transistor collector, negative to GND; transistor base to Pin 4 via 1kΩ resistor, emitter to GND; add a diode across motor terminals (cathode to positive) to protect against back EMF.
8. **Keypad**: Connect rows to Pins 7-10, columns to Pins 11-13 and 2.
9. **LCD**: Connect via I2C module to A4 (SDA) and A5 (SCL), VCC to 5V, GND to GND.

## Features and How They Work
### 1. Predefined Password-Protected Secure Door
- **1.1 Success Prompt if Password is Correct**:
  - Enter the default password "0000" (or a new password set via master password "1234") using the keypad.
  - On correct entry (confirmed with 'C'), the LCD displays "Access Granted," the green LED flashes 5 times, and the servo unlocks the door (moves to 90°).
- **1.2 Implement Door Opening Feature**:
  - The `unlockDoor()` function sets the servo to 90° for 10 seconds. Press 'D' on the keypad to lock it manually, or it auto-locks after the timer.
- **1.3 Warning Message if Password is Wrong**:
  - On incorrect password entry, the LCD shows "CODE INCORRECT," and the red LED lights up. After 3 failed attempts, a 30-second wait is enforced; otherwise, a 3-second delay applies. Warnings are deferred if smoke is detected.

### 2. Automatic Turn Light On/Off When Room is Occupied/Unoccupied
- **2.1 Detect People Entering/Exiting a Room**:
  - PIR1 (A3) detects entry when triggered first, followed by PIR2 (A2) within 15 seconds, indicating movement inward.
  - PIR2 triggered first, then PIR1 within 15 seconds (with exit sequence flag), indicates movement outward.
- **2.2 Keep Count of People Occupied**:
  - The `loop()` function updates the `occupancy` variable based on PIR sequences, incrementing for entry and decrementing for exit, with a 2-second debounce.
- **2.3 Turn On/Off Light Based on Count**:
  - The light (A1) turns on if `occupancy > 0` and off if `occupancy = 0`, managed in `loop()`.

### 3. Automatic Smoke Detection
- **3.1 Turn On Alarm if Smoke Detected**:
  - The gas sensor (A0) monitors smoke levels. If >600, the exhaust fan (Pin 1) activates via a transistor, and the buzzer (Pin 4) beeps (500ms) as an alarm. This has the highest priority.
- **3.2 Turn Off Alarm When Smoke Level is Below Unsafe Level**:
  - When the gas sensor value drops to <=600, the exhaust fan and buzzer turn off, resuming deferred unauthorized access warnings if any.

## How Each Function Works
### 1. `setup()`
- Initializes serial communication (9600 baud) for debugging.
- Configures pin modes (OUTPUT for LEDs, buzzer, fan, light; INPUT for sensors).
- Initializes LCD, sets servo to locked (0°), and displays "Smart Room Ready".

### 2. `loop()`
- **Smoke Detection**: Prioritizes checking A0; if >600, activates fan and buzzer, defers unauthorized warnings. If <=600, turns off fan/buzzer and processes pending warnings.
- **Door Unlock Countdown**: Manages a 10-second unlock timer.
- **Keypad Input**: Handles password entry, master check, or setup.
- **Occupancy Counting**: Processes PIR sequences for entry/exit.
- **Light Control**: Adjusts light based on occupancy.
- Uses a 50ms delay for responsiveness.

### 3. `updateDisplay()`
- Updates LCD with occupancy count (second line) if not in password/unlock mode.
- Logs to Serial Monitor.

### 4. `updateLcdMainScreen()`
- Clears LCD, shows "Enter Password:" with occupancy.

### 5. `lockDoor()`
- Locks door (servo to 0°), updates LCD, resets exit flag.

### 6. `unlockDoor()`
- Unlocks door (servo to 90°), flashes green LED, sets 10-second timer.

### 7. `handlePasswordSetup(char key)`
- Sets new password (0-9, 'B' backspace, 'C' confirm).

### 8. `handleMasterPasswordCheck(char key)`
- Verifies master password ("1234") for setup.

### 9. `handlePasswordEntry(char key)`
- Processes password input; defers incorrect warning if smoke detected.

### 10. `incorrectPassword()`
- Displays warning, lights red LED, enforces delays after failures.

### 11. `resetPIRTriggers()`
- Resets PIR flags and timers.

## Priority Logic
- **Smoke Detection Priority**: If smoke is detected, the system activates the exhaust fan and buzzer immediately, deferring unauthorized access warnings until smoke levels drop below 600.

## Images
- Circuit Overview
<img width="1043" height="720" alt="image" src="https://github.com/user-attachments/assets/950fd7ef-152d-4b52-aa6a-19065cc9b68f" />

- Smoke Detection Active *(fan and buzzer on)*
<img width="1296" height="476" alt="image" src="https://github.com/user-attachments/assets/9d613da1-ac7f-4047-99c5-d8ee07cf1fb2" />

- Door Unlocked *(servo at 90°, green led blinking)*
<img width="1200" height="778" alt="image" src="https://github.com/user-attachments/assets/7f1c8f5f-2e59-4464-ab57-9b403e41b84d" />

- Occupancy Display *(LCD showing people count, count > 0 turn on blue led)*
<img width="1040" height="642" alt="image" src="https://github.com/user-attachments/assets/af5c2d9c-63b2-484d-acb2-b75b4274e86b" />


## How to Use
1. Open the Tinkercad link and start the simulation.
2. Enter "0000" to unlock the door (green LED flashes).
3. Use PIR sensors to simulate entry/exit and watch occupancy/light changes.
4. Set gas sensor >600 to trigger smoke alarm and fan.
5. Test wrong password during smoke to see deferred warning.

## Contributions
- Developed by Fahith K.R.M. on Monday, October 02, 2025.
- Shared on GitHub for portfolio and resume enhancement.
