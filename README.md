# Countdown Timer – CrowPanel 2.8″ ESP32 Display

An [ESPHome](https://esphome.io/) configuration that turns an Elecrow CrowPanel 2.8-inch ESP32 display into a live "On Time" countdown/timer panel for Home Assistant.

## Hardware

| Component | Details |
|---|---|
| **Board** | Elecrow CrowPanel 2.8″ (ESP32-based, `esp32dev`) |
| **Display** | 320 × 240 ILI9341 TFT (SPI), oriented in landscape mode |
| **Touchscreen** | XPT2046 resistive touch controller (SPI, shared bus) |
| **Backlight** | LEDC PWM on GPIO27 (exposed as a dimmable light in HA) |
| **Framework** | ESP-IDF |

## What It Does

The panel connects to Home Assistant and subscribes to the sensor **`sensor.gaming_pc_2_on_time`** (a duration sensor whose state is measured in **hours**). It then:

1. **Displays the elapsed time** as `H:MM:SS` on a large centered label using the LVGL graphics library.
2. **Fills an arc gauge** proportionally — 0 hours maps to 0 % and 2 hours maps to 100 %.
3. **Turns the timer text red** (`#FF4444`) when the value exceeds 2 hours, giving a clear visual alert that the time limit has been passed. Otherwise the text stays the default light blue (`#E0E0FF`).

### UI Layout

```
┌──────────────────────────┐
│                          │
│      ╭── arc gauge ──╮   │
│      │   H:MM:SS     │   │
│      │   On Time     │   │
│      ╰───────────────╯   │
│                          │
└──────────────────────────┘
```

- **Arc gauge** – teal (`#00D4AA`) indicator on a dark track (`#2A2A4A`), 220 × 220 px.
- **Timer label** – Montserrat 36 pt, light blue (`#E0E0FF`) by default, red (`#FF4444`) when > 2 h.
- **Subtitle** – "On Time", Montserrat 14 pt, muted purple (`#5555AA`).
- **Background** – dark navy (`#1A1A2E`).

## Home Assistant Sensor

The configuration expects a sensor with the following attributes:

| Attribute | Value |
|---|---|
| Entity ID | `sensor.gaming_pc_2_on_time` |
| State class | Measurement |
| Unit of measurement | `h` |
| Device class | `duration` |

The state is a **decimal number of hours** (e.g. `0.0`, `0.5`, `1.75`).

## Pin Mapping

| Function | GPIO |
|---|---|
| SPI CLK | 14 |
| SPI MOSI | 13 |
| SPI MISO | 12 |
| Display CS | 15 |
| Display DC | 2 |
| Touch CS | 33 |
| Touch IRQ | 36 |
| Backlight | 27 |

## Getting Started

1. Copy the YAML file into your ESPHome configuration directory.
2. Create or update your `secrets.yaml` with the required values:
   ```yaml
   api_key: "<your_encryption_key>"
   ota_password: "<your_ota_password>"
   wifi_ssid: "<your_wifi_ssid>"
   wifi_password: "<your_wifi_password>"
   ```
3. Make sure `sensor.gaming_pc_2_on_time` exists in Home Assistant (or change the `entity_id` in the YAML to match your own duration sensor).
4. Compile and flash:
   ```bash
   esphome run "Countdown Timer Crowpanel 28inch.yaml"
   ```

## License

See [LICENSE](LICENSE) for details.
