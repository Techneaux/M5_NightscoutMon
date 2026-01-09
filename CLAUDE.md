# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

M5Stack Nightscout Monitor - An ESP32-based blood glucose monitoring display for the M5Stack platform. Fetches and displays real-time glucose data from Nightscout diabetes data platform with visual/audio alarms.

**Target Hardware**: M5Stack Core and Core2 (ESP32-based)

## Build System

### PlatformIO (Recommended)
```bash
# Extract the PlatformIO project template first
unzip PlatformIO/M5_NightscoutMon_PlatformIO.zip

# Copy latest sources to src/ folder and rename .ino to .cpp
cp *.cpp *.h *.c M5_NightscoutMon_PlatformIO/src/
mv M5_NightscoutMon_PlatformIO/src/M5_NightscoutMon.ino M5_NightscoutMon_PlatformIO/src/M5_NightscoutMon.cpp

# Build
pio run

# Upload to connected M5Stack
pio run --target upload
```

### Arduino IDE
- Install ESP32 board support and M5Stack libraries
- Install ArduinoJson library
- Open M5_NightscoutMon.ino and build

### Pre-built Binaries
Use M5Burner from releases for easy flashing without compilation.

## Architecture

### Core Source Files

| File | Purpose |
|------|---------|
| `M5_NightscoutMon.ino` | Main application (~2900 lines): setup, main loop, display rendering, Nightscout API communication, alarm handling |
| `M5NSconfig.cpp/h` | Configuration management - reads M5NS.INI from SD card, handles Preferences flash storage |
| `M5NSWebConfig.cpp/h` | Built-in web server for configuration (port 80, accessible at m5ns.local) |
| `IniFile.cpp/h` | INI file parser library |
| `DHT12.cpp/h` | DHT12 temperature/humidity sensor driver |
| `SHT3X.cpp/h` | SHT30 temperature/humidity sensor driver |
| `microdot.cpp/h` | Pimoroni Micro Dot pHAT I2C display support |

### Key Data Structures

```cpp
tConfig cfg;              // Configuration (thresholds, WiFi, display settings)
NSinfo ns;                // Nightscout data (glucose, IOB, COB, delta, loop status)
DynamicJsonDocument JSONdoc(16384);  // JSON parsing buffer
```

### Main Loop Flow

1. Check WiFi connection
2. Fetch Nightscout data every ~305 seconds via `readNightscout()`
3. Parse JSON, calculate delta, extract IOB/COB/loop status
4. Check alarm conditions, play sounds if needed
5. Render current display page via `draw_page()`
6. Handle button input via `buttons_test()`

### Display Pages (cycled via right button)

- **Page 0**: Detailed view - name, IOB/COB, BG with trend arrow, mini graph
- **Page 1**: Large BG value for distance viewing
- **Page 2**: Analog clock with temperature/humidity (requires ENV unit)
- **Error Log**: Last 10 errors with timestamps

### Nightscout API Support

- Nightscout v1/v2 API
- Sugarmate (Dexcom follower)
- Railway platform

### Configuration

Configuration via `M5NS.INI` on SD card or internal flash (via web interface):
- Nightscout URL and security token
- Up to 9 WiFi networks
- Alarm thresholds (yellow/red low/high)
- Display settings (brightness, rotation, units)
- Optional hardware (RGB LED strip, vibration motor, external sensors)

## Hardware Connections

- **I2C (Port A, pins 21/22)**: DHT12, SHT30, Micro Dot pHAT
- **Port B (pin 26)**: RGB LED strip, vibration motor
- **Port C (pin 17)**: Alternative LED/vibration pin
- **Pin 15**: M5Stack Fire internal RGB LEDs

## Dependencies

- M5Stack (official library)
- ArduinoJson v6+
- Adafruit_NeoPixel (optional, for RGB LED support)
