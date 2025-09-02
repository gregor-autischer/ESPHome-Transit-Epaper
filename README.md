# ESPHome E-Paper Transit Display

A project to display public transit departure information on a Waveshare 2.9" e-paper display using ESP32 and ESPHome.

## 📋 Project Overview

This project creates a transit departure board similar to those found at bus/tram stops in Graz, Austria. The display shows:
- Line numbers (with bold white text on black background)
- Destination names
- Minutes until departure

The display updates every minute and uses an e-paper display for low power consumption and excellent readability.

## 🛠 Hardware Requirements

- **ESP32 Development Board**
- **Waveshare 2.9" E-Paper Display** (Model: 2.90inv2-r2)
- **USB cable** for programming and power

### Pinout Configuration

| E-Paper Display | ESP32 Pin | Description |
|----------------|-----------|-------------|
| VCC | 3V3 | Power supply (3.3V) |
| GND | GND | Ground |
| DIN | GPIO23 | MOSI/Data line |
| CLK | GPIO18 | SPI Clock |
| CS | GPIO21 | Chip Select |
| DC | GPIO17 | Data/Command control |
| RST | GPIO16 | Reset |
| BUSY | GPIO4 | Busy signal |

## 📁 Project Files

- **`transit_display_live.yaml`** - **PRODUCTION VERSION** - Dynamic transit display with real-time Home Assistant integration
- `transit_display_dummy_data.yaml` - Test configuration with static dummy data for development
- `README.md` - This documentation file
- `CLAUDE.md` - AI assistant instructions and technical documentation

## 🚀 Installation & Setup

### Prerequisites

1. **Install ESPHome**:
   ```bash
   pip install esphome
   ```

2. **Install drivers** for your USB-to-Serial adapter (if needed)
   - macOS usually has drivers built-in for most ESP32 boards
   - For CP2102/CP2104 chips, you may need Silicon Labs drivers

### WiFi Configuration

⚠️ **IMPORTANT**: Before flashing, you **MUST** edit the WiFi credentials in `transit_display_live.yaml`:

```yaml
wifi:
  ssid: "YourWiFiSSID"      # Replace with your actual WiFi name
  password: "YourWiFiPassword"  # Replace with your actual WiFi password
```

**Security Note**: Never commit your actual WiFi credentials to version control. Consider using environment variables or ESPHome secrets for sensitive data.

### Home Assistant Integration

#### 🚌 Live Transit Data for Steiermark, Austria

For **plug-and-play** live departure data in the region of Steiermark (Austria), use the companion HACS integration:

**[PH_Steiermark_Oeffi](https://github.com/gregor-autischer/PH_Steiermark_Oeffi)** - A Home Assistant integration that provides real-time public transit departure data with the exact entities required by this display. Simply install via HACS, configure your stops, and the display will automatically show live departures!

#### Manual Entity Configuration

If setting up entities manually, the dynamic version requires Home Assistant entities with the following structure:

**Required entities:**
- `sensor.transit_departure_1` to `sensor.transit_departure_7` (minutes with indicators like "3!", "15*")

**Required attributes for each entity:**
- `line` - Line number (e.g., "5/6", "64", "63")
- `destination` - Destination name (e.g., "Hauptbahnhof", "St. Leonhard")

**Example Home Assistant entity:**
```yaml
sensor:
  - name: "Transit Departure 1"
    state: "3*"
    attributes:
      line: "5/6"
      destination: "Puntigam"
```

## 💾 Flashing Instructions

### 1. Check Connected USB Devices

```bash
# List USB serial devices on macOS
ls /dev/cu.usb* 2>/dev/null || ls /dev/tty.usb* 2>/dev/null

# Alternative: Check if specific device exists
test -e /dev/cu.usbserial-0001 && echo "Device connected" || echo "Device not found"
```

### 2. Compile the Configuration

```bash
# Compile without flashing (to check for errors)
esphome compile transit_display_live.yaml
```

### 3. Flash to ESP32

```bash
# Flash using specific USB port
esphome run transit_display_live.yaml --device /dev/cu.usbserial-0001

# Or let ESPHome auto-detect the port
esphome run transit_display_live.yaml
```

### 4. Monitor Device Logs

```bash
# View logs from USB-connected device
esphome logs transit_display_live.yaml --device /dev/cu.usbserial-0001

# View logs from WiFi-connected device
esphome logs transit_display_live.yaml
```

## 📝 Useful Commands

### ESPHome Commands

```bash
# Validate configuration
esphome config transit_display_live.yaml

# Clean build files
esphome clean transit_display_live.yaml

# Create initial configuration wizard
esphome wizard new_device.yaml

# Start web dashboard (access at http://localhost:6052)
esphome dashboard .
```

### Device Information

```bash
# Get detailed USB device info on macOS
system_profiler SPUSBDataType | grep -A 5 -B 5 "Serial"

# Monitor serial output directly
screen /dev/cu.usbserial-0001 115200
# (Press Ctrl+A, then K to exit screen)
```

## 🎨 Dynamic Display Features

The **production version** (`transit_display_live.yaml`) displays real-time transit data with:

### 🧪 Development & Testing

For development and testing without Home Assistant, use `transit_display_dummy_data.yaml` which contains static test data.

### ✨ Smart Features

- **Real-time updates** from Home Assistant entities
- **Up to 7 departures** displayed simultaneously
- **Dynamic line box sizing** (20px for regular lines, 24px for "5/6" style)
- **Automatic text truncation** (destinations >12 chars → 11 chars + ".")
- **Special abbreviations**: "Hauptbahnhof" → "Hauptbhf."
- **Status indicators**: 
  - solid line = Delayed departure
  - dashed line = Scheduled departure
- **Fallback display**: Shows "XX"/"xx" when data unavailable

### 📊 Display Format

Each departure shows:
```
[LINE]  Destination Name    MIN
 64     St. Leonhard         3
 5/6    Hauptbhf.           15
 63     Jakominipla.        21
```

### 🔄 Data Flow

1. **Home Assistant** provides transit entities with attributes
2. **ESPHome** reads entity states and attributes via API
3. **Display updates** automatically when data changes

### Font Configuration

The display uses Google Fonts (Roboto family):
- **Line numbers**: Roboto weight 800, size 13px
- **Destinations**: Roboto weight 400, size 12px  
- **Minutes**: Roboto Mono weight 700, size 16px

## 🔧 Troubleshooting

### Device Not Found

1. Check USB cable (use a data cable, not charge-only)
2. Try different USB port
3. Install/update USB drivers
4. Check device permissions:
   ```bash
   sudo chmod 666 /dev/cu.usbserial-0001
   ```

### Compilation Errors

1. Ensure ESPHome is up to date:
   ```bash
   pip install --upgrade esphome
   ```

2. Check YAML syntax (indentation matters!)

3. Verify board configuration matches your ESP32 model

### Display Not Updating

1. Check wiring connections
2. Verify power supply (3.3V,)
3. Check BUSY pin - display won't update if stuck busy
4. Try increasing update interval if refreshing too frequently

### WiFi Connection Issues

1. Verify SSID and password (case-sensitive)
2. Ensure 2.4GHz network (ESP32 doesn't support 5GHz)
3. Check router allows new device connections

## 📊 Project Status

**🟢 PRODUCTION READY** - The dynamic version is fully functional with Home Assistant integration.

### Current Implementation
- ✅ **Real-time transit data** from Home Assistant
- ✅ **Dynamic updates** with automatic refresh
- ✅ **Smart text formatting** and truncation
- ✅ **Status indicators** for delays and schedules
- ✅ **Robust error handling** with fallback displays
- ✅ **7 departure display** with proper spacing

### Technical Details
- **Update frequency**: automatically on entity change
- **Display technology**: E-ink for low power consumption
- **Communication**: ESPHome API with Home Assistant
- **Text processing**: Automatic truncation and special cases

## 📄 License

This project uses ESPHome, which is licensed under the MIT License.

## 🤝 Contributing

Feel free to submit issues and enhancement requests!

---

*Last updated: September 2025*