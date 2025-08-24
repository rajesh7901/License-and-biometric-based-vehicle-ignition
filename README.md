# License and Biometric-based Vehicle Ignition

🚗🔒 A smart vehicle ignition system that ensures only **authorized and safe drivers** can start the vehicle.  
The system verifies the **driving license**, **biometric fingerprint**, and checks for **alcohol consumption** before allowing ignition.

---

## 🚀 Features
- ✅ Driving license verification (RFID/Smart Card based)  
- ✅ Biometric fingerprint authentication  
- ✅ Alcohol sensor to detect drunk driving  
- ✅ Vehicle ignition only enabled after passing all checks  
- ✅ Arduino-based implementation  

---

## 🛠️ Tech Stack
- **Arduino Uno / Mega** (Microcontroller)  
- **Fingerprint Sensor Module**  
- **RFID / Smart Card Reader**  
- **MQ-3 Alcohol Sensor**  
- **Relay Module** (for ignition control)  
- **C++ / Arduino IDE**  

---


## System Flow
```mermaid
flowchart TD
    A[Driver enters vehicle] --> B[Insert License Card]
    B -->|Valid| C[Fingerprint Scan]
    B -->|Invalid| Z[Access Denied]

    C -->|Matched| D[Alcohol Sensor Test]
    C -->|Not Matched| Z[Access Denied]

    D -->|Safe| E[Ignition ON 🚗]
    D -->|Alcohol Detected| Z[Access Denied]

    Z --> F[System Logs Event]
