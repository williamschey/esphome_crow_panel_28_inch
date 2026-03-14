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

### 1. Configure secrets

Create or update your `secrets.yaml` in the ESPHome configuration directory:

```yaml
ota_password: "<your_ota_password>"
wifi_ssid: "<your_wifi_ssid>"
wifi_password: "<your_wifi_password>"
```

### 2. (Optional) Enable API encryption

The default configuration runs **without** API encryption. If you want encrypted communication between Home Assistant and the device, generate a random base64-encoded 32-byte key:

```bash
# Using OpenSSL
openssl rand -base64 32

# Or using Python
python3 -c "import secrets, base64; print(base64.b64encode(secrets.token_bytes(32)).decode())"
```

Then add the key to your `secrets.yaml`:

```yaml
api_key: "<paste_generated_key_here>"
```

And update the `api:` section in the YAML config:

```yaml
api:
  encryption:
    key: !secret api_key
```

> **Important:** If you add or change the encryption key after the device has already been adopted in Home Assistant, you will need to re-add the device with the new key.

### 3. Verify the Home Assistant sensor

Make sure `sensor.gaming_pc_2_on_time` exists in Home Assistant (or change the `entity_id` in the YAML to match your own duration sensor). You can verify this under **Developer Tools → States**.

### 4. Compile and flash

```bash
esphome run "Countdown Timer Crowpanel 28inch.yaml"
```

### 5. Add the device to Home Assistant

After flashing, the device needs to be **adopted into the ESPHome integration** in Home Assistant before it can receive sensor states. Without this step, the display will show `--:--:--` because the API connection is never established.

1. In Home Assistant, go to **Settings → Devices & Services**.
2. The device should appear automatically under **Discovered**. Click **Configure**, then **Submit**.
3. If it does not auto-discover, click **+ Add Integration → ESPHome** and enter the device's IP address or hostname (`countdown_timer_crowpanel.local`).
4. If you configured API encryption, you will be prompted to enter the encryption key.
5. Once connected, you should see `Home Assistant connected!` in the ESPHome device logs, and the display will begin showing the timer value.

### Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Display shows `--:--:--` | Home Assistant is not connected to the device | Follow step 5 above to add the device |
| Device not discovered in HA | Network/mDNS issue | Add manually by IP address |
| HA fails to connect | API encryption mismatch | Ensure the key in `secrets.yaml` matches what HA expects, or remove encryption from both sides |
| Timer value doesn't update | Entity ID mismatch | Verify `sensor.gaming_pc_2_on_time` exists in **Developer Tools → States** |