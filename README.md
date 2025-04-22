# Guition_control_panel config

ESPhome control panel using JC4827W543

This ESPHome configuration powers a control panel built on an ESP32-S3, featuring an LVGL-driven 480x272 display, GT911 touchscreen, WiFi, and Home Assistant integration. The setup includes a dynamic UI with pages for weather, clock, and room controls, plus diagnostic sensors and Easter eggs.

## Components and Their Roles

### Substitutions
- **Purpose**: Defines reusable variables for device naming.
- **Role**: Centralizes `device_name` (`control-panel-1`) and `friendly_name` (`Control Panel 1`) for use in entity names and configurations.
- **Details**: Simplifies maintenance; changes require recompiling.

### ESPHome
- **Purpose**: Configures core ESPHome settings and boot behavior.
- **Role**:
  - Sets device identity for Home Assistant and OTA updates.
  - Enables PSRAM via `build_flags: "-DBOARD_HAS_PSRAM"`.
  - Optimizes memory with `qio_opi` and flash with `dio`.
  - On boot: delays 5s, hides LVGL boot screen, turns on backlight.
- **Details**: Uses PlatformIO options for performance tuning.

### ESP32
- **Purpose**: Specifies ESP32-S3 hardware and framework.
- **Role**:
  - Targets `esp32-s3-devkitc-1` with ESP-IDF framework.
  - Sets CPU to 240 MHz, 64 KB data cache, and PSRAM optimizations.
- **Details**: ESP-IDF ensures robust LVGL support.

### Debug
- **Purpose**: Provides diagnostic sensors.
- **Role**: Monitors heap, PSRAM, and loop time every 15s.
- **Details**: Essential for resource tracking in LVGL projects.

### Logger
- **Purpose**: Enables logging for debugging.
- **Role**: Outputs logs to serial or Home Assistant (e.g., touch events, LVGL states).
- **Details**: Default `DEBUG` level; adjustable for verbosity.

### Home Assistant API
- **Purpose**: Integrates with Home Assistant.
- **Role**:
  - Enables secure communication via encryption key.
  - Shows/hides `lbl_hastatus` widget on Home Assistant connect/disconnect.
- **Details**: Uses `lambda` to filter Home Assistant clients.

### OTA
- **Purpose**: Enables over-the-air firmware updates.
- **Role**: Allows wireless updates with a secure password.
- **Details**: Critical for remote maintenance.

### WiFi
- **Purpose**: Manages WiFi connectivity.
- **Role**:
  - Connects to primary network using credentials.
  - Provides fallback AP (`Control-Panel-1 Fallback Hotspot`).
- **Details**: Supports captive portal for setup.

### Captive Portal
- **Purpose**: Offers a web interface for WiFi configuration.
- **Role**: Accessible in AP mode for network setup.
- **Details**: Simplifies initial configuration.

### Globals
- **Purpose**: Defines global variables for state management.
- **Role**:
  - `wifi_iconstring`: Stores WiFi signal icon.
  - `recent_touch`, `screen_timer`: Track touch and timeouts (partially used).
  - `konami_sequence`, `last_touch_x`, `last_touch_y`: Support Konami code Easter egg.
- **Details**: Reset on reboot; stored in RAM.

### Image
- **Purpose**: Defines images for LVGL.
- **Role**:
  - `esphome_image`: Local RGB image for UI.
  - `boot_logo`: Online 200x200 RGB565 image for boot screen.
- **Details**: `alpha_channel` enables transparency.

### Time
- **Purpose**: Manages time synchronization.
- **Role**:
  - Syncs via Home Assistant (`ha_time`, `time_comp`) and SNTP (`sntp_time`).
  - Updates clock hands, date labels, and schedules anti-burn-in.
- **Details**: Anti-burn-in runs at specific hours.

### Font
- **Purpose**: Defines fonts for LVGL text.
- **Role**: Provides Material Design Icons and Arimo fonts for UI elements.
- **Details**: Limited glyphs and `bpp: 4` for memory efficiency.

### Text
- **Purpose**: Exposes LVGL text widgets as entities.
- **Role**: Updates weather forecast data (e.g., condition, temperature).
- **Details**: Tied to specific LVGL widgets.

### Color
- **Purpose**: Defines reusable colors for LVGL.
- **Role**: Ensures consistent UI styling (e.g., `ha_gold`, `blue_color`).
- **Details**: Supports hex and RGB formats.

### Web Server
- **Purpose**: Hosts a web interface.
- **Role**: Provides browser access to status and logs.
- **Details**: Uses version 3 for modern features.

### I2C
- **Purpose**: Configures I2C for touchscreen.
- **Role**: Connects GT911 touchscreen to pins 8 (SDA) and 4 (SCL) at 400 kHz.
- **Details**: Fast communication for touch events.

### Light and Output
- **Purpose**: Controls display backlight with dimming.
- **Role**:
  - `output`: Uses `ledc` (PWM) on GPIO1 for brightness control.
  - `light`: Exposes `display_backlight` as a monochromatic entity with smooth transitions.
- **Details**: Turns on at boot and on touch; supports dimming.

### SPI
- **Purpose**: Configures Quad SPI for display.
- **Role**: Drives NV3041A display with pins 47 (CLK) and 21, 48, 40, 39 (data).
- **Details**: Quad SPI maximizes graphics throughput.

### Display
- **Purpose**: Configures NV3041A display.
- **Role**: Renders LVGL UI at 480x272, rotated 180°.
- **Details**: `update_interval: never` and `auto_clear_enabled: false` optimize performance.

### LVGL
- **Purpose**: Manages graphical interface.
- **Role**:
  - Configures display and touchscreen.
  - Defines UI themes, styles, and pages (e.g., clock, weather).
  - Enters deep sleep on idle; supports anti-burn-in snow effect.
- **Details**: Includes navigation and Easter egg widgets.

### PSRAM
- **Purpose**: Configures external PSRAM.
- **Role**: Provides memory for LVGL buffers at 80 MHz in octal mode.
- **Details**: Essential for graphics performance.

### Touchscreen
- **Purpose**: Configures GT911 touchscreen.
- **Role**:
  - Detects touch/release events on pins 3 (interrupt) and 38 (reset).
  - Resumes LVGL, turns on backlight, and supports Konami code via coordinate-based swipes.
  - Wakes from deep sleep via interrupt pin.
- **Details**: Logs touch coordinates; includes Easter egg for long press.

### Deep Sleep
- **Purpose**: Reduces power consumption.
- **Role**: Enters deep sleep after idle timeout; wakes on touch via GPIO3.
- **Details**: Configured for 30-minute sleep; active-low interrupt.

### Number
- **Purpose**: Exposes screen timeout setting.
- **Role**: Adjusts LVGL idle timeout (10–180s) in Home Assistant.
- **Details**: Persists across reboots.

### Text Sensor and Sensor
- **Purpose**: Provides diagnostics and environmental data.
- **Role**:
  - `text_sensor`: Reports device info and reset reasons.
  - `sensor`: Monitors heap, PSRAM, loop time, and WiFi signal.
- **Details**: WiFi sensors drive UI icon updates.

### Script
- **Purpose**: Defines reusable UI update actions.
- **Role**:
  - `update_ui_values`: Updates WiFi signal icons.
  - `time_update`: Updates clock and date.
- **Details**: Runs on schedule or time sync.

### Interval
- **Purpose**: Schedules periodic tasks.
- **Role**: Executes `update_ui_values` every 10s.
- **Details**: Keeps UI current without overloading CPU.

### Binary Sensor
- **Purpose**: Reports connectivity status.
- **Role**: Indicates device online status in Home Assistant.
- **Details**: Useful for monitoring.

### Switch
- **Purpose**: Controls anti-burn-in feature.
- **Role**: Activates snow effect to prevent burn-in; resumes UI when off.
- **Details**: Scheduled or manually controlled.

## Notes
- **Memory Context**: Past interactions involved LED effects and arcade designs, influencing the Konami code Easter egg.
- **Optimizations**: Consider reducing font glyphs, adjusting WiFi intervals, or adding sensors for new use cases.
