# Gas-Leakage-Detection-and-Alerting-System

An embedded safety system that detects gas leakage using an **MQ-2 gas sensor** and **Arduino Uno**, provides a local warning through a **buzzer and 16×2 LCD**, and sends an **SMS alert using a SIM900A GSM module** when the detected gas level exceeds a predefined threshold.

## Project Overview

Gas leakage can create serious safety risks in residential, commercial, and industrial environments. This project was developed as a low-cost embedded system for detecting the presence of combustible gases and providing immediate local and remote alerts.

The **MQ-2 sensor** continuously monitors the gas level. The Arduino Uno reads the sensor value and compares it with a predefined threshold.

When the gas value exceeds the threshold:

* The buzzer is activated.
* The gas value is displayed on the LCD.
* An SMS alert is sent to a predefined mobile number through the SIM900A GSM module.

When the gas level returns below the threshold, the buzzer is turned off.

## Key Features

* Real-time gas monitoring using MQ-2 sensor
* Arduino Uno based embedded control
* Analog gas sensor data acquisition
* 16×2 LCD for displaying gas readings
* Buzzer-based local alarm
* SIM900A GSM-based SMS notification
* Configurable gas detection threshold
* Prevents repeated SMS alerts while the alarm condition remains active
* Simple and low-cost embedded safety prototype

## System Architecture

```text
             ┌─────────────────┐
             │    MQ-2 Sensor  │
             │  Gas Detection  │
             └────────┬────────┘
                      │
                      │ Analog Signal
                      ▼
             ┌─────────────────┐
             │   Arduino UNO   │
             │   ATmega328P    │
             └──────┬───┬───┬──┘
                    │   │   │
             ┌──────┘   │   └───────────┐
             ▼          ▼               ▼
       ┌──────────┐ ┌─────────┐   ┌────────────┐
       │ 16×2 LCD │ │ Buzzer  │   │  SIM900A   │
       │ Display  │ │  Alarm  │   │ GSM Module │
       └──────────┘ └─────────┘   └──────┬─────┘
                                         │
                                         ▼
                                   ┌────────────┐
                                   │ Mobile     │
                                   │ Phone      │
                                   └────────────┘
```

## Hardware Components

| Component                | Quantity | Purpose                      |
| ------------------------ | -------: | ---------------------------- |
| Arduino Uno              |        1 | Main microcontroller         |
| MQ-2 Gas Sensor          |        1 | Detects combustible gases    |
| SIM900A GSM Module       |        1 | Sends SMS alerts             |
| 16×2 LCD                 |        1 | Displays gas readings/status |
| Buzzer                   |        1 | Local alarm indication       |
| 5V Power Supply          |        1 | Power source                 |
| Breadboard               |        1 | Circuit prototyping          |
| Jumper Wires             |        — | Connections                  |
| Resistor / Potentiometer |        — | Circuit/LCD interfacing      |

The original project report lists these components as the hardware used for the prototype.

## Software Used

* Arduino IDE
* Embedded C / Arduino programming
* AT commands for GSM communication

## Pin Configuration

The following pin configuration is used in the project source code:

| Component | Arduino Pin | Purpose                 |
| --------- | ----------- | ----------------------- |
| MQ-2      | A0          | Analog gas sensor input |
| Buzzer    | D6          | Alarm output            |
| LCD RS    | D12         | LCD register select     |
| LCD EN    | D11         | LCD enable              |
| LCD D4    | D5          | LCD data                |
| LCD D5    | D4          | LCD data                |
| LCD D6    | D3          | LCD data                |
| LCD D7    | D2          | LCD data                |
| GSM RX/TX | D8, D7      | Serial communication    |

These assignments are taken from the project's source code.

## Working Principle

### 1. Gas Detection

The MQ-2 sensor continuously monitors the surrounding environment.

The Arduino reads the sensor through its analog input:

```cpp
gasValue = analogRead(MQ2);
```

The sensor value is then displayed on the 16×2 LCD.

```text
Gas: <sensor value>
```

The MQ-2 module used in the project can detect gases such as LPG, methane, propane, hydrogen, carbon monoxide and smoke.

### 2. Threshold Comparison

A predefined threshold is used to determine whether the detected gas level should trigger an alarm.

```cpp
int gasThreshold = 100;
```

The basic decision is:

```text
Gas Value > Threshold
        │
        ▼
   Gas Detected
        │
   ┌────┴─────┐
   ▼          ▼
Buzzer ON   Send SMS
```

The threshold value can be adjusted according to the sensor/module calibration and testing conditions.

### 3. Buzzer Alert

When the sensor value exceeds the threshold, the Arduino activates the buzzer.

```cpp
digitalWrite(BUZZER, HIGH);
```

When the value falls below the threshold:

```cpp
digitalWrite(BUZZER, LOW);
```

This provides an immediate local warning.

### 4. GSM SMS Alert

The SIM900A GSM module is used to send an SMS to a predefined mobile number.

The Arduino communicates with the GSM module using serial communication and AT commands.

The SMS process is:

```text
Arduino
   │
   │ AT+CMGF=1
   ▼
SIM900A
   │
   │ AT+CMGS
   ▼
Mobile Network
   │
   ▼
Registered Mobile Number
```

The project uses:

```cpp
gsm.println("AT+CMGF=1");
```

to configure text mode.

The message is then sent using:

```cpp
gsm.println("AT+CMGS=\"<PHONE_NUMBER>\"");
gsm.println("Gas leak detected!");
gsm.println((char)26);
```

The original report's source code contains a phone number; it is intentionally replaced here with `<PHONE_NUMBER>` so you do not publish a personal phone number on GitHub.

### 5. Preventing Repeated SMS

The project uses an `alarmTriggered` flag:

```cpp
bool alarmTriggered = false;
```

When gas is detected:

```cpp
if (gasValue > gasThreshold && !alarmTriggered)
```

the system sends the SMS and sets:

```cpp
alarmTriggered = true;
```

This prevents the system from continuously sending SMS messages during the same gas-leak event.

When the gas level falls below the threshold:

```cpp
else if (gasValue <= gasThreshold)
```

the buzzer is turned off and:

```cpp
alarmTriggered = false;
```

This allows a new SMS to be sent if another gas-leak event occurs later.

## Firmware Flow

```text
             START
               │
               ▼
       Initialize LCD
       Initialize GSM
       Configure Buzzer
               │
               ▼
       Read MQ-2 Sensor
               │
               ▼
       Display Gas Value
          on LCD
               │
               ▼
     Gas Value > Threshold?
          /           \
        YES            NO
         │              │
         ▼              ▼
    Buzzer ON       Buzzer OFF
         │              │
         ▼              │
    Alarm Already       │
     Triggered?         │
       /     \           │
     YES      NO         │
      │        │         │
      │        ▼         │
      │    Send SMS      │
      │        │         │
      │        ▼         │
      │  Set Alarm Flag  │
      │        │         │
      └────────┴─────────┘
               │
               ▼
           Wait 1 sec
               │
               ▼
             LOOP
```

The report describes the same overall flow: sensor detection → Arduino processing → LCD/buzzer indication → GSM SMS alert.

## Complete Source Code

The project source code is based on the Arduino implementation used in the project report:

```cpp
#include <LiquidCrystal.h>
#include <SoftwareSerial.h>

LiquidCrystal lcd(12, 11, 5, 4, 3, 2);
SoftwareSerial gsm(8, 7);

#define MQ2 A0
#define BUZZER 6

int gasThreshold = 100;
int gasValue;
bool alarmTriggered = false;

void setup()
{
    lcd.begin(16, 2);

    gsm.begin(9600);

    pinMode(BUZZER, OUTPUT);
}

void loop()
{
    gasValue = analogRead(MQ2);

    lcd.clear();
    lcd.setCursor(0, 0);

    lcd.print("Gas: ");
    lcd.print(gasValue);

    if (gasValue > gasThreshold && !alarmTriggered)
    {
        digitalWrite(BUZZER, HIGH);

        sendSMS();

        alarmTriggered = true;
    }
    else if (gasValue <= gasThreshold)
    {
        digitalWrite(BUZZER, LOW);

        alarmTriggered = false;
    }

    delay(1000);
}

void sendSMS()
{
    gsm.println("AT+CMGF=1");

    delay(1000);

    gsm.println("AT+CMGS=\"<PHONE_NUMBER>\"");

    delay(1000);

    gsm.println("Gas leak detected!");

    delay(1000);

    gsm.println((char)26);

    delay(1000);
}
```

The source code above follows the actual implementation included in the project report.

## Important Implementation Detail

The project uses:

```cpp
SoftwareSerial gsm(8, 7);
```

for communication between the Arduino and GSM module.

The GSM module is initialized at:

```cpp
gsm.begin(9600);
```

The sensor is read through:

```cpp
analogRead(A0);
```

and the buzzer is controlled using digital pin D6.

## Testing

The system was tested by introducing a small amount of LPG near the gas sensor and observing the system response. The project report states that the hardware and software were developed and tested for the desired detection and alert behavior.

### Expected Behavior

| Condition                   | LCD                 | Buzzer | SMS                  |
| --------------------------- | ------------------- | ------ | -------------------- |
| Gas below threshold         | Gas value displayed | OFF    | No SMS               |
| Gas above threshold         | Gas value displayed | ON     | Alert SMS            |
| Gas returns below threshold | Updated value       | OFF    | Ready for next event |

## Applications

Potential applications include:

* Domestic gas leakage detection
* Kitchens
* Gas storage areas
* Industrial environments
* Hotels and commercial buildings
* Laboratories
* Combustible gas monitoring

The project report identifies domestic and industrial gas detection as major application areas.

## Limitations

This project is an educational prototype and has some limitations:

* MQ-2 is a general-purpose gas sensor and is not a precision gas concentration measurement instrument.
* The threshold value requires appropriate calibration and testing for the intended environment.
* GSM communication depends on network availability.
* The prototype provides detection and alerting but does not automatically shut off the gas supply.
* The system should not be treated as a certified safety device for real-world hazardous environments without appropriate safety validation.

## Future Improvements

Possible improvements include:

* Automatic gas supply shut-off using a suitable valve/relay system
* IoT-based remote monitoring
* Mobile application for monitoring
* Web dashboard
* Multiple gas sensors
* Improved sensor calibration
* Data logging
* Emergency-service notification
* Smart-home integration

These directions are also identified in the original project report's future-scope section.

## Key Concepts Learned

Through this project, I worked with:

* Arduino Uno / ATmega328P
* Embedded C / Arduino programming
* Analog sensor interfacing
* ADC-based sensor reading
* Threshold-based decision making
* Digital GPIO control
* LCD interfacing
* UART / serial communication
* SoftwareSerial
* GSM communication
* AT commands
* SMS-based alerting
* Embedded system debugging and testing

## Project Structure

Recommended GitHub repository structure:

```text
Gas-Leakage-Detection/
│
├── README.md
│
├── Code/
│   └── Gas_Leakage_Detection.ino
│
├── Circuit/
│   └── circuit_diagram.png
│
├── Images/
│   ├── hardware_setup.jpg
│   ├── lcd_output.jpg
│   └── gsm_alert.jpg
│
└── Documentation/
    └── project_report.pdf
```

## Team

This was a group capstone project completed during the 2023–24 academic year at **New Polytechnic, Kolhapur**, Department of Electronics and Telecommunication.

### Team Members

* Swapnali Rathod
* Akash Dhavale
* Snehal Onkar
* Jui Kasar
* Dinesh Mohite

## License

This project was developed as an academic/educational embedded systems project.
