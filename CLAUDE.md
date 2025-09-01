# Claude Code / LLM Instructions

This file contains important context and instructions for AI assistants (Claude, GitHub Copilot, etc.) working on this ESPHome E-Paper transit display project.

## 🎯 Project Context

This is an ESPHome-based project that displays public transit departure information on a Waveshare 2.9" e-paper display using an ESP32 microcontroller. The display mimics Austrian (Graz) public transit departure boards.

### Current Implementation Status
- ✅ Static departure data display
- ✅ Formatted as transit departure board
- ✅ WiFi connectivity configured
- ✅ OTA updates enabled
- ❌ No real-time data fetching (yet)
- ❌ No API integration (yet)

## 📁 Key Files

### Main Configuration Files
- `busbahnbim_display.yaml` - **PRIMARY FILE** - Active transit display configuration
- `epaper_test.yaml` - Test configuration (DO NOT MODIFY - kept for reference)

### File Purposes
- **busbahnbim_display.yaml**: Production configuration showing transit departures
- **epaper_test.yaml**: Original test file with graphical elements (preserved as backup)
- **README.md**: User documentation
- **CLAUDE.md**: This file - AI assistant instructions

## ⚙️ Technical Details

### Hardware Configuration
```yaml
# E-Paper Display Model
model: 2.90inv2-r2  # Waveshare 2.9" V2 display
board: esp32dev     # ESP32 development board

# Pin Connections (CRITICAL - DO NOT CHANGE)
clk_pin: GPIO18     # SPI Clock
mosi_pin: GPIO23    # Data line (DIN on display)
cs_pin: GPIO21      # Chip Select
dc_pin: GPIO17      # Data/Command
reset_pin: GPIO16   # Reset
busy_pin: GPIO4     # Busy signal

# Power: 3.3V (NOT 5V!)
```

### Display Specifications
- **Resolution**: 296x128 pixels
- **Colors**: Monochrome (black/white)
- **Refresh**: Full refresh every 30 seconds
- **Type**: E-ink/E-paper (retains image without power)

### Font Configuration
```yaml
# Line numbers: Bold, size 13, weight 800
font_line_number: Roboto, weight 800, size 13

# Destinations: Regular, size 12
font_normal: Roboto, weight 400, size 12  

# Minutes: Monospace bold, size 16
font_time: Roboto Mono, weight 700, size 16
```

## 🔧 Common Modifications

### Changing Departure Entries
When modifying departure information, edit the lambda section in `busbahnbim_display.yaml`:

```cpp
// Pattern for each entry:
// 1. Draw black rectangle for line number
it.filled_rectangle(0, y_pos - 8, WIDTH, 16, COLOR_ON);
// 2. Print line number (white on black)
it.print(X, y_pos, id(font_line_number), COLOR_OFF, TextAlign::CENTER, "LINE");
// 3. Print destination
it.print(X, y_pos, id(font_normal), COLOR_ON, TextAlign::CENTER_LEFT, "DESTINATION");
// 4. Print minutes
it.print(it.get_width() - 5, y_pos, id(font_time), COLOR_ON, TextAlign::CENTER_RIGHT, "MIN");
```

### Important Rules for Text Display
1. **Line break threshold**: Destinations > 12 characters need two lines
2. **Line number box sizing**:
   - Single digit: 20px wide
   - Double digit: 20px wide
   - With slash (e.g., "5/6"): 24px wide
3. **Y-position spacing**:
   - Single line entries: y_pos += 26
   - Two-line entries: y_pos += 34

### Current Display Data
```
Entry 1: Line 64 → St. Leonhard → 2 min
Entry 2: Line 63 → Merangasse → 5 min
Entry 3: Line 64 → St. Leonhard → 9 min
Entry 4: Line 64 → Merangasse → 14 min
Entry 5: Line 5/6 → Jakominip. → 21 min
```

## 🚀 Working with the Code

### Before Making Changes
1. **NEVER modify `epaper_test.yaml`** - it's kept as reference
2. **Always work with `busbahnbim_display.yaml`** for transit display
3. Check USB connection: `ls /dev/cu.usb*`
4. Verify current configuration compiles: `esphome compile busbahnbim_display.yaml`

### Testing Changes
```bash
# 1. Compile first (faster, catches errors)
esphome compile busbahnbim_display.yaml

# 2. Flash to device
esphome run busbahnbim_display.yaml --device /dev/cu.usbserial-0001

# 3. Monitor logs
esphome logs busbahnbim_display.yaml --device /dev/cu.usbserial-0001
```

### Common Tasks

#### Add More Departure Entries
- Maximum practical limit: 8-9 entries (display height constraint)
- Each entry needs ~26px height (single line) or ~34px (two lines)
- Start position: y_pos = 15 (no header)

#### Change Font Sizes
- Edit the `size:` parameter in font definitions
- Smaller fonts = more entries fit
- Test readability on actual display

#### Modify Update Interval
- Change `update_interval: 30s` to desired value
- E-paper displays have limited write cycles
- Recommended: 30s minimum, 5min for battery operation

## ⚠️ Critical Warnings

### DO NOT:
- ❌ Change SPI pin assignments without hardware rewiring
- ❌ Set update_interval < 10s (display needs time to refresh)
- ❌ Use `platform: ESP32` in esphome section (deprecated)
- ❌ Forget to update WiFi credentials before flashing

### ALWAYS:
- ✅ Test compile before flashing
- ✅ Check USB device exists before flashing
- ✅ Use CENTER_RIGHT alignment for minutes
- ✅ Use COLOR_OFF for text on black backgrounds
- ✅ Use COLOR_ON for black text/lines on white
- ✅ Keep separator lines between entries

## 🐛 Debugging Tips

### If Display Doesn't Update
1. Check BUSY pin (GPIO4) - might be stuck
2. Verify 3.3V power (not 5V!)
3. Check all SPI connections
4. Try full power cycle
5. Increase update_interval

### If Compilation Fails
1. Check YAML indentation (2 spaces, no tabs)
2. Verify font IDs match between definition and usage
3. Ensure no duplicate component IDs
4. Check ESPHome version: `esphome version`

### If Flashing Fails
1. Check USB cable (data cable, not charge-only)
2. Verify port: `ls /dev/cu.usb*`
3. Try different USB port
4. Hold BOOT button on ESP32 while flashing starts

## 📊 Performance Considerations

- **Compile time**: ~30-60 seconds
- **Flash time**: ~60-90 seconds
- **Display refresh**: ~2-3 seconds (full refresh)
- **WiFi connection**: ~5-10 seconds on boot
- **Power consumption**: ~50mA active, ~20mA idle

## 🔄 Future Implementation Notes

### For Real-Time Data Integration
Consider:
- HTTP requests to transit APIs
- JSON parsing capabilities
- Error handling for API failures  
- Fallback to static data
- Caching mechanism

### For Battery Operation
Implement:
- Deep sleep between updates
- Longer update intervals (5-15 min)
- WiFi disconnect after update
- Power consumption optimization

## 📝 Code Style Guidelines

1. **Comments**: Keep inline comments for complex logic
2. **Naming**: Use descriptive names (e.g., `font_line_number` not `font1`)
3. **Spacing**: Maintain consistent y_pos increments
4. **Organization**: Group related display elements together
5. **Constants**: Define magic numbers at the top when possible

## 🎯 Quick Command Reference

```bash
# Most used commands for this project:
esphome compile busbahnbim_display.yaml                      # Test compilation
esphome run busbahnbim_display.yaml --device /dev/cu.usbserial-0001  # Flash
esphome logs busbahnbim_display.yaml --device /dev/cu.usbserial-0001 # Monitor
ls /dev/cu.usb*                                             # Find USB device
```

---

*This file is specifically for AI assistants and developers. For general usage, see README.md*