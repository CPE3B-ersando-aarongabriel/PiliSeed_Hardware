# 🌱 PiliSeed Hardware V1 - Smart Crop Recommendation Device

[![Platform](https://img.shields.io/badge/Platform-ESP32-blue.svg)](https://www.espressif.com/en/products/socs/esp32)
[![Framework](https://img.shields.io/badge/Framework-Arduino-00979D.svg)](https://www.arduino.cc/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![API](https://img.shields.io/badge/API-PiliSeed%20Backend-orange.svg)](https://piliseed.onrender.com)

> An IoT-based smart agricultural device that collects environmental sensor data and provides AI-powered crop recommendations through the PiliSeed ecosystem.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [System Architecture](#-system-architecture)
- [Project Structure](#-project-structure)
- [Hardware Components](#-hardware-components)
- [Pin Configuration](#-pin-configuration)
- [Software Requirements](#-software-requirements)
- [Installation & Setup](#-installation--setup)
- [Configuration](#-configuration)
- [Security](#-security)
- [Usage](#-usage)
- [API Integration](#-api-integration)
- [Troubleshooting](#-troubleshooting)
- [Documentation Standards](#-documentation-standards)
- [Contributing](#-contributing)
- [License](#-license)
- [Authors](#-authors)

---

## 🎯 Overview

### What is PiliSeed Hardware V1?

PiliSeed Hardware V1 is an **ESP32-based IoT device** designed for precision agriculture. It serves as the hardware component of the PiliSeed ecosystem, which includes:

- **Hardware Device** (this project) - Collects real-time environmental data
- **Backend Server** - Processes data and runs AI crop recommendation algorithms
- **Mobile Application** - Provides user interface for farmers and agricultural enthusiasts

### Key Features

| Feature                           | Description                                                   |
| --------------------------------- | ------------------------------------------------------------- |
| 🌡️ **Multi-Sensor Integration**   | Temperature, humidity, light, and soil moisture monitoring    |
| 🤖 **AI-Powered Recommendations** | Get top 3 crop suggestions based on environmental conditions  |
| 📡 **WiFi Connectivity**          | Seamless connection to PiliSeed cloud backend                 |
| 🖥️ **OLED Display**               | Real-time sensor data and recommendation visualization        |
| 🔘 **One-Button Interface**       | Simple toggle between sensor data and crop recommendations    |
| ⚡ **Real-Time Processing**       | Instant sensor readings with 120-second AI processing timeout |

### How It Works

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   ESP32 Device  │────▶│  PiliSeed API   │────▶│  Mobile App     │
│   (Sensors)     │◀────│  (AI Backend)   │◀────│  (User View)    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                       │
   Collects Data          Processes &              Displays
   from Environment       Recommends Crops         to Farmer
```

1. **Data Collection**: The device continuously reads environmental data from 4 sensors
2. **Data Transmission**: On button press, sensor data is sent to the PiliSeed backend via HTTP POST
3. **AI Processing**: The backend analyzes the data using machine learning algorithms
4. **Recommendation Display**: Top 3 suitable crops are displayed on the OLED screen

---

## 🏗️ System Architecture

### Hardware Architecture

```
                    ┌──────────────────────────────────────────┐
                    │              ESP32 DevKit                │
                    │                                          │
    ┌───────────┐   │   ┌─────────────────────────────────┐   │
    │  DHT22    │───┼──▶│  GPIO 4 (Digital Input)         │   │
    │ Temp/Hum  │   │   └─────────────────────────────────┘   │
    └───────────┘   │                                          │
                    │   ┌─────────────────────────────────┐   │
    ┌───────────┐   │   │  GPIO 32 (ADC1_CH4)             │   │
    │   LDR     │───┼──▶│  Analog Input - Light           │   │
    │  Sensor   │   │   └─────────────────────────────────┘   │
    └───────────┘   │                                          │
                    │   ┌─────────────────────────────────┐   │
    ┌───────────┐   │   │  GPIO 33 (ADC1_CH5)             │   │
    │   Soil    │───┼──▶│  Analog Input - Moisture        │   │
    │ Moisture  │   │   └─────────────────────────────────┘   │
    └───────────┘   │                                          │
                    │   ┌─────────────────────────────────┐   │
    ┌───────────┐   │   │  GPIO 5 (Digital Input)         │   │
    │  Button   │───┼──▶│  INPUT_PULLUP                   │   │
    └───────────┘   │   └─────────────────────────────────┘   │
                    │                                          │
                    │   ┌─────────────────────────────────┐   │
    ┌───────────┐   │   │  I2C (SDA: GPIO 21, SCL: GPIO 22)│  │
    │  OLED     │◀──┼───│  0x3C Address                   │   │
    │ SSD1306   │   │   └─────────────────────────────────┘   │
    └───────────┘   │                                          │
                    └──────────────────────────────────────────┘
```

### Software Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Main Application                         │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │    Setup     │  │  Main Loop   │  │   State Management   │  │
│  │  - WiFi Init │  │  - Sensor    │  │  - showingPlants     │  │
│  │  - Display   │  │    Reading   │  │  - buttonPressed     │  │
│  │  - Sensors   │  │  - Button    │  │  - recommendedCrops  │  │
│  │  - Pins      │  │    Handling  │  │  - cropCount         │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Sensor Functions                       │  │
│  │  readTemperature() | readHumidity() | readLightLevel()   │  │
│  │  readSoilMoisture()                                       │  │
│  └──────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Display Functions                      │  │
│  │  displayMessage() | displaySensorData() | displayError() │  │
│  │  displayCropRecommendations() | displayProcessing()       │  │
│  └──────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    API Functions                          │  │
│  │  getCropRecommendations() - HTTP POST to PiliSeed API    │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## � Project Structure

```
PiliSeed_Esp32/
├── Main.ino              # Main application code
├── config.h              # ⚠️ Your credentials (GITIGNORED - create from template)
├── config.h.example      # Template for configuration (safe to commit)
├── .gitignore            # Specifies intentionally untracked files
└── README.md             # This documentation file
```

### File Descriptions

| File               | Description                                              |
| ------------------ | -------------------------------------------------------- |
| `Main.ino`         | Main Arduino sketch containing all device logic          |
| `config.h`         | **Your private credentials** - never commit this file    |
| `config.h.example` | Configuration template showing required variables        |
| `.gitignore`       | Lists files that Git should ignore (includes `config.h`) |
| `README.md`        | Project documentation                                    |

---

## �🔧 Hardware Components

### Bill of Materials (BOM)

| #   | Component                     | Model/Specification              | Quantity | Purpose                          |
| --- | ----------------------------- | -------------------------------- | -------- | -------------------------------- |
| 1   | Microcontroller               | ESP32 DevKit V1 (30-pin)         | 1        | Main processing unit with WiFi   |
| 2   | Temperature & Humidity Sensor | DHT22 (AM2302)                   | 1        | Environmental climate monitoring |
| 3   | Light Sensor                  | LDR (Light Dependent Resistor)   | 1        | Ambient light level measurement  |
| 4   | Soil Moisture Sensor          | Capacitive Soil Moisture v1.2    | 1        | Soil water content monitoring    |
| 5   | Display                       | OLED 0.96" SSD1306 (128x64, I2C) | 1        | Data visualization               |
| 6   | Push Button                   | Momentary Tactile Switch         | 1        | User input interface             |
| 7   | Resistor                      | 10kΩ (for LDR voltage divider)   | 1        | LDR circuit                      |
| 8   | Jumper Wires                  | Male-to-Male, Male-to-Female     | ~20      | Connections                      |
| 9   | Breadboard                    | 830-point                        | 1        | Prototyping (optional)           |

### Sensor Specifications

#### 1. DHT22 Temperature & Humidity Sensor

| Parameter            | Specification      |
| -------------------- | ------------------ |
| Temperature Range    | -40°C to 80°C      |
| Temperature Accuracy | ±0.5°C             |
| Humidity Range       | 0-100% RH          |
| Humidity Accuracy    | ±2-5% RH           |
| Operating Voltage    | 3.3V - 5V          |
| Sampling Rate        | 0.5 Hz (2 seconds) |

**Why DHT22?** Provides accurate temperature and humidity readings essential for determining suitable growing conditions for different crops.

#### 2. LDR (Light Dependent Resistor) Sensor

| Parameter         | Specification            |
| ----------------- | ------------------------ |
| Light Range       | 0 - 20,000 lux (mapped)  |
| Response Time     | ~20-30 ms                |
| Operating Voltage | 3.3V - 5V                |
| Output            | Analog (0-4095 on ESP32) |

**Why LDR?** Measures ambient light levels to determine if the environment receives adequate sunlight for specific crops (full sun, partial shade, etc.).

#### 3. Capacitive Soil Moisture Sensor

| Parameter         | Specification                  |
| ----------------- | ------------------------------ |
| Moisture Range    | 0-100% (mapped)                |
| Operating Voltage | 3.3V - 5V                      |
| Output            | Analog (0-4095 on ESP32)       |
| Probe Type        | Corrosion-resistant capacitive |

**Why Capacitive?** Unlike resistive sensors, capacitive sensors don't corrode over time and provide more consistent readings for long-term soil monitoring.

#### 4. OLED Display (SSD1306)

| Parameter         | Specification           |
| ----------------- | ----------------------- |
| Resolution        | 128 x 64 pixels         |
| Color             | Monochrome (White/Blue) |
| Interface         | I2C (Address: 0x3C)     |
| Operating Voltage | 3.3V - 5V               |
| Viewing Angle     | >160°                   |

---

## 📌 Pin Configuration

### Wiring Diagram

```
ESP32 DevKit V1
┌─────────────────────────────────────┐
│                USB                  │
│  ┌─────────────────────────────┐   │
│  │                             │   │
│  │    GPIO 4  ────────────────────── DHT22 Data Pin
│  │    GPIO 32 ────────────────────── LDR Analog Out
│  │    GPIO 33 ────────────────────── Soil Moisture Analog Out
│  │    GPIO 5  ────────────────────── Button (to GND)
│  │    GPIO 21 (SDA) ──────────────── OLED SDA
│  │    GPIO 22 (SCL) ──────────────── OLED SCL
│  │    3.3V ───────────────────────── All Sensor VCC
│  │    GND ────────────────────────── All Sensor GND
│  │                             │   │
│  └─────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

### Pin Mapping Table

| ESP32 Pin | Component     | Connection | Notes                  |
| --------- | ------------- | ---------- | ---------------------- |
| GPIO 4    | DHT22         | Data       | Digital input          |
| GPIO 32   | LDR           | Signal     | ADC1_CH4, Analog input |
| GPIO 33   | Soil Moisture | Signal     | ADC1_CH5, Analog input |
| GPIO 5    | Button        | Signal     | INPUT_PULLUP enabled   |
| GPIO 21   | OLED          | SDA        | I2C Data               |
| GPIO 22   | OLED          | SCL        | I2C Clock              |
| 3.3V      | All Sensors   | VCC        | Power supply           |
| GND       | All Sensors   | GND        | Common ground          |

### Circuit Notes

1. **DHT22**: Connect a 10kΩ pull-up resistor between VCC and Data pin (some modules have built-in)
2. **LDR**: Create a voltage divider with a 10kΩ resistor between LDR and GND
3. **Button**: Uses internal pull-up resistor; connect between GPIO 5 and GND
4. **I2C**: External pull-up resistors (4.7kΩ) may improve communication stability

---

## 💻 Software Requirements

### Development Environment

| Software            | Version            | Purpose                         |
| ------------------- | ------------------ | ------------------------------- |
| Arduino IDE         | 2.0+ (recommended) | Primary development environment |
| ESP32 Board Package | 2.0.0+             | ESP32 compiler and tools        |

### Required Libraries

Install these libraries via Arduino Library Manager (`Sketch > Include Library > Manage Libraries`):

| Library                | Version  | Author          | Purpose                        |
| ---------------------- | -------- | --------------- | ------------------------------ |
| `WiFi`                 | Built-in | Espressif       | WiFi connectivity              |
| `HTTPClient`           | Built-in | Espressif       | HTTP requests to backend       |
| `ArduinoJson`          | 6.x      | Benoit Blanchon | JSON parsing and serialization |
| `Wire`                 | Built-in | Arduino         | I2C communication              |
| `Adafruit GFX Library` | 1.11.x   | Adafruit        | Graphics primitives            |
| `Adafruit SSD1306`     | 2.5.x    | Adafruit        | OLED display driver            |
| `DHT sensor library`   | 1.4.x    | Adafruit        | DHT22 sensor interface         |

### Installing ESP32 Board Package

1. Open Arduino IDE
2. Go to `File > Preferences`
3. Add the following URL to "Additional Board Manager URLs":
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
4. Go to `Tools > Board > Boards Manager`
5. Search for "ESP32" and install "esp32 by Espressif Systems"

---

## 🚀 Installation & Setup

### Step 1: Clone/Download the Project

```bash
git clone https://github.com/your-repo/PiliSeed_Esp32.git
cd PiliSeed_Esp32/PiliSeed_Hardware_V1
```

Or download the ZIP file and extract to your Arduino sketchbook folder.

### Step 2: Install Required Libraries

Open Arduino IDE and install each library:

1. Go to `Sketch > Include Library > Manage Libraries`
2. Search and install:
   - `ArduinoJson` by Benoit Blanchon
   - `Adafruit GFX Library` by Adafruit
   - `Adafruit SSD1306` by Adafruit
   - `DHT sensor library` by Adafruit

### Step 3: Hardware Assembly

1. **Mount the ESP32** on a breadboard (optional for prototyping)
2. **Connect the DHT22**:
   - VCC → 3.3V
   - GND → GND
   - Data → GPIO 4
3. **Connect the LDR**:
   - Create voltage divider (LDR + 10kΩ)
   - Output → GPIO 32
4. **Connect Soil Moisture Sensor**:
   - VCC → 3.3V
   - GND → GND
   - Signal → GPIO 33
5. **Connect OLED Display**:
   - VCC → 3.3V
   - GND → GND
   - SDA → GPIO 21
   - SCL → GPIO 22
6. **Connect Button**:
   - One leg → GPIO 5
   - Other leg → GND

### Step 4: Configure Credentials

> ⚠️ **Security Note**: Never commit sensitive credentials to version control!

This project uses a separate `config.h` file for credentials that is **gitignored** by default.

1. **Copy the example configuration file**:

   ```bash
   cp config.h.example config.h
   ```

2. **Edit `config.h`** with your actual credentials:

   ```cpp
   // WiFi Credentials
   #define WIFI_SSID "your_wifi_ssid"
   #define WIFI_PASSWORD "your_wifi_password"

   // PiliSeed API Configuration
   #define BACKEND_URL "https://piliseed.onrender.com"
   #define SENSOR_ID "your_sensor_id_here"
   ```

3. **Verify the file is gitignored**:
   - Check that `config.h` appears in `.gitignore`
   - The file should **not** appear in `git status` after creation

### Step 5: Upload to ESP32

1. Connect ESP32 to computer via USB
2. Select Board: `Tools > Board > ESP32 Dev Module`
3. Select Port: `Tools > Port > COMx` (your ESP32 port)
4. Configure Upload Settings:
   - Upload Speed: 115200
   - Flash Frequency: 80MHz
   - Flash Mode: QIO
5. Click **Upload** (→ button)

### Step 6: Verify Operation

1. Open Serial Monitor (`Tools > Serial Monitor`)
2. Set baud rate to **115200**
3. You should see:
   ```
   ═══════════════════════════════════════
   🌱 PiliSeed Smart Plant System
   ═══════════════════════════════════════
   🖥️  Initializing OLED display...
   ✅ OLED initialized
   🌡️  Initializing DHT22 sensor...
   ✅ DHT22 initialized
   ...
   ✅ System ready!
   ```

---

## ⚙️ Configuration

### Configuration Files

| File               | Purpose                          | Git Status        |
| ------------------ | -------------------------------- | ----------------- |
| `config.h.example` | Template with placeholder values | ✅ Tracked        |
| `config.h`         | Your actual credentials          | ❌ **Gitignored** |
| `.gitignore`       | Specifies files to exclude       | ✅ Tracked        |

### Setting Up Configuration

```bash
# 1. Copy the example file
cp config.h.example config.h

# 2. Edit config.h with your credentials (use any text editor)
```

### WiFi Configuration

In `config.h`:

```cpp
#define WIFI_SSID "your_network_name"      // Your WiFi network name
#define WIFI_PASSWORD "your_password"       // Your WiFi password
```

### API Configuration

In `config.h`:

```cpp
#define BACKEND_URL "https://piliseed.onrender.com"  // PiliSeed API endpoint
#define SENSOR_ID "your_sensor_id_here"              // Unique sensor identifier
```

> **Note**: The `SENSOR_ID` is obtained from the PiliSeed backend when you register your device. Contact the PiliSeed team or use the mobile app to register a new sensor.

### Pin Reconfiguration

If you need to use different pins, modify these definitions:

```cpp
#define DHTPIN 4              // DHT22 data pin
#define LDR_PIN 32            // Light sensor analog pin
#define SOIL_MOISTURE_PIN 33  // Soil moisture analog pin
#define BUTTON_PIN 5          // Push button pin
```

### Display Settings

```cpp
#define SCREEN_WIDTH 128   // OLED width in pixels
#define SCREEN_HEIGHT 64   // OLED height in pixels
#define OLED_RESET -1      // Reset pin (-1 if sharing Arduino reset)
```

---

## � Security

### Credential Management Best Practices

This project implements secure credential management to protect sensitive information:

#### ✅ What's Protected

| Credential Type | Storage Location | Git Status    |
| --------------- | ---------------- | ------------- |
| WiFi SSID       | `config.h`       | ❌ Gitignored |
| WiFi Password   | `config.h`       | ❌ Gitignored |
| Sensor ID       | `config.h`       | ❌ Gitignored |
| API URL         | `config.h`       | ❌ Gitignored |

#### 🛡️ Security Measures Implemented

1. **Separate Configuration File**: All credentials are stored in `config.h`, isolated from main code
2. **Gitignore Protection**: `config.h` is listed in `.gitignore` to prevent accidental commits
3. **Template File**: `config.h.example` provides a safe template without real credentials
4. **No Hardcoded Secrets**: Main code (`Main.ino`) contains no sensitive information

#### ⚠️ Important Security Guidelines

```
DO:
✅ Keep config.h private and never share it
✅ Use strong, unique WiFi passwords
✅ Regularly rotate credentials if compromised
✅ Verify config.h is gitignored before pushing

DON'T:
❌ Commit config.h to version control
❌ Share screenshots showing credentials
❌ Use the same password across multiple networks
❌ Store credentials in Main.ino or other tracked files
```

#### 🔄 If Credentials Are Accidentally Exposed

1. **Immediately change** your WiFi password
2. **Regenerate** your sensor ID from the PiliSeed backend
3. **Check git history** and remove any commits containing credentials
4. **Consider using** `git filter-branch` or BFG Repo-Cleaner to purge history

---

## �📖 Usage

### Basic Operation

1. **Power On**: Connect ESP32 to power source (USB or external 5V)
2. **Wait for Initialization**: System connects to WiFi (~15 seconds)
3. **View Sensor Data**: Default screen shows real-time sensor readings
4. **Get Recommendations**: Press the button to request AI crop recommendations
5. **Toggle Views**: Press button again to switch back to sensor display

### Display Screens

#### Sensor Data Screen (Default)

```
┌────────────────────────┐
│ Temp: 28.5C            │
│ Humidity: 65.0%        │
│ Light: 15000 lux       │
│ Soil: 45.0%            │
│                        │
│ Press button for       │
│ crop recommendations   │
└────────────────────────┘
```

#### Processing Screen

```
┌────────────────────────┐
│ Processing...          │
│                        │
│ Analyzing sensor       │
│ data with AI...        │
│                        │
│ This may take up       │
│ to 2 minutes...        │
└────────────────────────┘
```

#### Crop Recommendations Screen

```
┌────────────────────────┐
│ Top 3 Crops:           │
│                        │
│ 1. Rice                │
│ 2. Corn                │
│ 3. Soybean             │
│                        │
│ Press btn for sensors  │
└────────────────────────┘
```

### Serial Monitor Output

For debugging, the device outputs detailed logs:

```
═══════════════════════════════════════
📡 Sending sensor data to PiliSeed API
═══════════════════════════════════════
🌐 Endpoint: https://piliseed.onrender.com/recommendations/hardware/xxx/readings

📤 Sending data:
{"soil_moisture_pct":45.0,"temperature_c":28.5,"humidity_pct":65.0,"light_lux":15000}

⏳ Waiting for AI response...
📊 HTTP Status Code: 200

✅ SUCCESS! Recommended crops:
  1. Rice
  2. Corn
  3. Soybean
═══════════════════════════════════════
```

---

## 🌐 API Integration

### Endpoint

```
POST /recommendations/hardware/{sensorId}/readings
```

### Request Format

```json
{
  "soil_moisture_pct": 45.0,
  "temperature_c": 28.5,
  "humidity_pct": 65.0,
  "light_lux": 15000
}
```

### Response Format

```json
{
  "message": "Recommendations generated successfully",
  "top_3_crops": ["Rice", "Corn", "Soybean"],
  "total_crops_generated": 10
}
```

### Timeout Configuration

The device uses a **120-second timeout** for API requests to accommodate AI processing time:

```cpp
http.setTimeout(120000);  // 120 seconds
```

---

## 🔍 Troubleshooting

### Common Issues and Solutions

| Issue                        | Possible Cause                | Solution                                   |
| ---------------------------- | ----------------------------- | ------------------------------------------ |
| OLED not displaying          | Wrong I2C address             | Check address with I2C scanner; try 0x3D   |
| DHT22 reading -999           | Wiring issue or faulty sensor | Check connections; add pull-up resistor    |
| WiFi not connecting          | Wrong credentials             | Verify SSID and password; check WiFi range |
| HTTP Error 404               | Invalid sensor ID             | Verify sensorId from PiliSeed backend      |
| Connection timeout           | Server busy or network issue  | Wait and retry; check internet connection  |
| Soil reading always 0 or 100 | Calibration needed            | Check sensor placement; calibrate in code  |

### Debugging Tips

1. **Enable Serial Monitor**: Always monitor at 115200 baud for detailed logs
2. **Check WiFi Signal**: Signal strength (RSSI) is displayed on startup
3. **Validate Sensor Readings**: Ensure all sensors show valid values before requesting recommendations
4. **API Testing**: Use tools like Postman to test the API endpoint independently

### Error Messages

| Display Error       | Meaning                                       |
| ------------------- | --------------------------------------------- |
| "WiFi Disconnected" | Lost connection to WiFi network               |
| "HTTP Error XXX"    | Server returned an error status code          |
| "Connection Failed" | Could not reach the backend server            |
| "JSON Parse Error"  | Invalid response format from server           |
| "Sensor Error!"     | One or more sensors returned invalid readings |

---

## 📚 Documentation Standards

This README follows industry-standard documentation practices. Here's what makes professional documentation:

### 1. **Structure & Organization**

```
✅ Clear table of contents with anchor links
✅ Logical section ordering (Overview → Setup → Usage → Troubleshooting)
✅ Consistent heading hierarchy (H1 → H2 → H3)
✅ Related information grouped together
```

### 2. **Essential Sections**

| Section                  | Purpose                                 |
| ------------------------ | --------------------------------------- |
| **Title & Badges**       | Quick project identification and status |
| **Overview/Description** | What the project does and why           |
| **Features**             | Key capabilities at a glance            |
| **Requirements**         | Hardware/software prerequisites         |
| **Installation**         | Step-by-step setup guide                |
| **Configuration**        | Customization options                   |
| **Usage**                | How to operate the system               |
| **API Documentation**    | For projects with APIs                  |
| **Troubleshooting**      | Common issues and solutions             |
| **Contributing**         | How others can help                     |
| **License**              | Legal usage terms                       |
| **Authors/Contact**      | Who to reach out to                     |

### 3. **Formatting Best Practices**

```markdown
# Use headers for navigation (H1 for title only)

## Major sections use H2

### Subsections use H3

- Use bullet points for lists
- Use numbered lists for sequential steps
- Use tables for structured data comparison

`Inline code` for commands, file names, and variables
```

```
Code blocks for multi-line code examples
```

> Blockquotes for important notes or warnings

```

### 4. **Visual Elements**

- **Badges**: Show project status, version, license
- **Diagrams**: ASCII art or images for architecture/wiring
- **Tables**: Organize specifications and configurations
- **Icons/Emojis**: Improve scannability (use sparingly)

### 5. **Writing Guidelines**

| Do ✅ | Don't ❌ |
|-------|---------|
| Use clear, concise language | Use jargon without explanation |
| Write in present tense | Mix tenses inconsistently |
| Include examples | Leave steps ambiguous |
| Keep paragraphs short | Write walls of text |
| Update when code changes | Let documentation become stale |

### 6. **README Quality Checklist**

- [ ] Can a new developer set up the project in under 30 minutes?
- [ ] Are all prerequisites clearly listed?
- [ ] Are configuration options documented?
- [ ] Is there a quick start guide?
- [ ] Are common errors addressed?
- [ ] Is the code structure explained?
- [ ] Are contribution guidelines included?

---

## 🤝 Contributing

We welcome contributions to the PiliSeed Hardware project!

### How to Contribute

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/AmazingFeature`)
3. **Commit** your changes (`git commit -m 'Add AmazingFeature'`)
4. **Push** to the branch (`git push origin feature/AmazingFeature`)
5. **Open** a Pull Request

### Code Style Guidelines

- Use meaningful variable and function names
- Comment complex logic
- Follow existing code formatting
- Test thoroughly before submitting

### Reporting Issues

Please include:
- Device/board model
- Arduino IDE version
- Library versions
- Serial monitor output
- Steps to reproduce

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

```

MIT License

Copyright (c) 2025 PiliSeed Team

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.

```

---

## 👥 Authors

**PiliSeed Development Team**

- Hardware Development
- Backend Integration
- Mobile Application

---

## 📞 Support

For support, please:

1. Check the [Troubleshooting](#-troubleshooting) section
2. Review existing GitHub Issues
3. Contact the PiliSeed team through the mobile application

---

<div align="center">

**🌱 PiliSeed - Growing Smarter, Together 🌱**

*Empowering farmers with AI-driven agricultural insights*

</div>
```
