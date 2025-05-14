## Vehicle Access Control System

### Overview
This project implements a **vehicle access control system** for a parking lot using an **8051 microcontroller** programmed in assembly language. The system manages vehicle entry by verifying IDs entered via an RFID card reader or keypad, categorizes vehicles into five types (A, B, C, D, E), tracks available parking slots, and controls a gate. It includes user feedback through indicator lights and voice prompts, and communicates with an exit microcontroller to update slot counts when vehicles leave.

### Features
- **Vehicle Detection**: Detects arriving vehicles using a sensor.
- **ID Verification**: Accepts 3-character IDs (e.g., "A01") via RFID or keypad and verifies them against a stored database.
- **Vehicle Categories**: Supports five vehicle types (A, B, C, D, E) with predefined slot limits:
  - Type A: 40 slots
  - Type B: 20 slots
  - Type C: 60 slots
  - Type D: 20 slots
  - Type E: 20 slots
- **Gate Control**: Opens/closes the gate for valid IDs with available slots.
- **User Feedback**: Displays green/red lights and plays voice messages (e.g., "Welcome, tap your card") or warning tones.
- **Timeouts**: Waits 5 seconds for RFID input and 10 seconds for keypad input.
- **Serial Communication**: Updates slot counts when vehicles exit via communication with an exit microcontroller at 9600 baud.

### How It Works
1. **Initialization**: Sets up slot counts for each vehicle type and configures serial communication.
2. **Vehicle Detection**: A sensor (`P1.0`) detects an arriving vehicle.
3. **ID Input**:
   - Plays "Welcome, please tap your card" and turns on a green light (`P1.4`).
   - Waits for an RFID card (5 seconds) or keypad input (10 seconds).
   - Stores the 3-character ID (e.g., "A01") in RAM (addresses 50–52).
4. **ID Verification**:
   - Checks the first character (vehicle type: A, B, C, D, or E) against stored types in ROM (`0x1000`).
   - Verifies the next two characters against valid IDs for that type (e.g., "01" for A in `0x1010`).
   - Confirms slot availability (e.g., A’s count < 40).
5. **Gate Operation**:
   - If valid, opens the gate (`P2.6`, `P2.7`), decrements the slot count, and waits for the vehicle to pass.
   - If invalid or no slots, shows a red light and plays a warning tone until the vehicle leaves.
6. **Exit Handling**: Receives vehicle type (e.g., 'A') from the exit microcontroller via serial communication and increments the slot count.

### Code Structure
- **Interrupt Handlers**:
  - `0x0003`: Processes keypad input (3 characters).
  - `0x0023`: Handles RFID card input (3 characters).
  - `0x0213`: Manages serial communication for exit updates.
- **Main Program** (`0x0500`): Runs the detection loop, plays prompts, and initiates ID verification.
- **Subroutines**:
  - `VEHICLE_DETECT_1`: Monitors the vehicle sensor.
  - `CARD_READER_OR_KEYPAD`: Manages RFID/keypad input with timeouts.
  - `VERIFY_ACCESS`: Compares IDs and checks slots.
  - `OPEN_GATE_A/B/C/D/E`: Controls the gate for each vehicle type.
  - `NO_ACCESS_SUBROUTINE`: Handles invalid access with red light and warning tone.
  - `DELAY_*`: Implements delays (e.g., 5s, 10s, gate opening times).
  - `SERIAL_TIMER_START`: Configures 9600 baud serial communication.
- **Data Storage**:
  - `0x1000`: Vehicle types (A, B, C, D, E).
  - `0x1010`–`0x1800`: Valid IDs for each type (e.g., "01" to "50" for A).

### Hardware Requirements
- **8051 Microcontroller**: Core processor.
- **RFID Reader**: Connected via serial interface.
- **Keypad**: Inputs IDs via Port 2 (`P2`).
- **Sensors**: Vehicle detection (`P1.0`), gate position (`P3.6`, `P3.7`).
- **Indicators**: Green/red lights (`P1.4`), speaker for voice/warning (`P1.1`, `P1.2`, `P1.3`).
- **Gate Control**: Relays or motors on `P2.6`, `P2.7`.
- **Exit Microcontroller**: Sends vehicle type via serial communication.

### Usage
1. **Assemble the Code**: Use an 8051 assembler (e.g., Keil µVision or A51).
2. **Upload to Microcontroller**: Flash the hex file to the 8051.
3. **Connect Hardware**: Wire the RFID reader, keypad, sensors, lights, speaker, and gate control as per pin assignments.
4. **Power On**: The system will start detecting vehicles and processing IDs.
5. **Test**:
   - Place a vehicle at the sensor (`P1.0`).
   - Tap an RFID card or enter an ID (e.g., "A01").
   - Verify gate operation and slot count updates.

### Example Scenario
- A car arrives, triggering `P1.0`.
- System plays "Welcome, tap your card" and lights green.
- Driver enters "A01" via keypad.
- System verifies "A01" is valid for type A and a slot is available (e.g., 39/40).
- Gate opens, slot count drops to 38, and the car enters.
- If "A99" is entered (invalid), a red light and warning tone activate until the car leaves.

### Notes
- **Calibration**: Ensure sensors and RFID reader are calibrated for reliable input.
- **Timeouts**: Adjust delay loops (`DELAY_*`) for different hardware response times.
- **Slot Limits**: Modify `INITIALIZE_NO_SLOTS` to change slot counts.
- **ID Database**: Update ROM data (`0x1010`–`0x1800`) to add/remove valid IDs.

