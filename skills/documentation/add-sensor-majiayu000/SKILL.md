---
name: add-sensor
description: Use when user wants to add a new sensor to the Enviro+ monitoring system, or asks to monitor a new data point. Guides through importing libraries, initialization, reading sensor values, publishing to Adafruit IO and Home Assistant, updating documentation, testing, and rate limit verification.
---

# Add New Sensor to Enviro+ Logger

This skill guides you through the complete process of adding a new sensor to the project.

## Prerequisites

Before adding a sensor:
- Sensor library must be installed in the virtual environment
- Sensor must be hardware-compatible with Raspberry Pi
- Consider Pi Zero 2W's limited resources

## Step-by-Step Process

### 1. Import Sensor Library
Add the sensor library import at the top of `publish_to_adafruit.py`

```python
from your_sensor_library import SensorClass
```

### 2. Initialize Sensor
Add initialization in the `read_sensors()` function

```python
def read_sensors():
    sensors = {}

    try:
        # ... existing code ...

        # Initialize new sensor
        new_sensor = SensorClass()

        # ... rest of function ...
```

### 3. Read Sensor Value
Get the sensor value and add to `sensors` dict with a clear key name

**Important**: If the sensor uses analog/ADC readings, it may return stale data on first read (like BME280 and MICS6814). Test this and discard first reading if needed:

```python
# If sensor needs first reading discard:
_ = new_sensor.read()
time.sleep(0.1)

# Get actual reading
value = new_sensor.read()
sensors['new_sensor_name'] = round(value, 2)
```

### 4. Publish to Adafruit IO
Add feed mapping in `publish_to_adafruit()` function's `feed_mapping` dict:

```python
feed_mapping = {
    'temperature': 'enviro-temperature',
    'pressure': 'enviro-pressure',
    # ... existing feeds ...
    'new_sensor_name': 'enviro-new-sensor',  # Add this
}
```

### 5. Publish to Home Assistant
Add sensor configuration in `publish_to_homeassistant()` function's `sensor_configs` dict:

```python
sensor_configs = {
    'temperature': {
        'name': 'Enviro+ Temperature',
        'unit': '°C',
        'device_class': 'temperature',
        'icon': 'mdi:thermometer'
    },
    # ... existing sensors ...
    'new_sensor_name': {
        'name': 'Enviro+ New Sensor',
        'unit': 'UNIT',  # e.g., 'Pa', '%', 'lx'
        'device_class': 'CLASS',  # e.g., 'pressure', 'humidity', 'illuminance'
        'icon': 'mdi:ICON_NAME'
    }
}
```

**Home Assistant device_class reference:**
- Temperature: `device_class: 'temperature'`, unit: `'°C'`
- Pressure: `device_class: 'atmospheric_pressure'`, unit: `'hPa'`
- Humidity: `device_class: 'humidity'`, unit: `'%'`
- Light: `device_class: 'illuminance'`, unit: `'lx'` (NOT 'lux'!)
- Custom: Omit device_class, use descriptive icon

**Icon reference**: Browse [Material Design Icons](https://pictogrammers.com/library/mdi/)

### 6. Update Documentation

**README.md**:
- Add sensor to the feeds list
- Explain what it measures and typical values
- Update sensor description section

**CLAUDE.md** (or SKILLS.md if not merged):
- Add sensor details to hardware section
- Document any special considerations (calibration, warm-up, etc.)

### 7. Test Manually
Run at least 3 times to verify stability:

```bash
source ~/.virtualenvs/pimoroni/bin/activate
python3 /home/kleinmatic/Code/enviroplus-logger/publish_to_adafruit.py
```

### 8. Verify in Home Assistant
If Home Assistant publishing is enabled:
1. Check sensor appears in **Settings → Devices & Services → MQTT → Enviro+ Sensor**
2. Verify in **Developer Tools → States** (filter for "enviro")
3. Check unit of measurement displays correctly
4. Verify icon appears as expected

### 9. Rate Limit Check
**Critical**: Ensure total publishing time doesn't exceed Adafruit IO limits

- Free tier: 30 data points/minute maximum
- Each sensor takes 0.5 seconds to publish
- Formula: `(number_of_sensors × 0.5 seconds) < 30 seconds`

Currently: 8 sensors × 0.5s = 4 seconds ✓

With your new sensor: 9 sensors × 0.5s = 4.5 seconds ✓

If you exceed 30 seconds total, you'll get 429 rate limit errors.

### 10. Update Dependencies
If you installed a new library:

```bash
source ~/.virtualenvs/pimoroni/bin/activate
pip freeze > requirements.txt
```

## Important Reminders

### Don't Add Excessive Logging
- microSD card longevity concerns on Pi Zero
- Current logging is acceptable but don't expand it
- Remove temporary debug print statements before committing

### Test First Reading Behavior
Many analog sensors return stale data on first read. Always test:

```bash
source ~/.virtualenvs/pimoroni/bin/activate
python3 -c "
from your_sensor import Sensor
import time

sensor = Sensor()
print('First reading:', sensor.read())
time.sleep(0.1)
print('Second reading:', sensor.read())
time.sleep(10)
print('After 10s idle:', sensor.read())
time.sleep(0.1)
print('Second after idle:', sensor.read())
"
```

If first readings are different, discard them like BME280 and MICS6814.

### Security Check
Before committing:
- Verify `.env` is still in `.gitignore`
- No credentials in code
- Test with `read_sensors.py` first

## Common Issues

**Sensor not appearing in Home Assistant:**
- Check Home Assistant logs: **Settings → System → Logs**
- Look for MQTT discovery errors
- Verify MQTT broker is running
- Check device_class/unit compatibility

**Rate limit 429 errors:**
- Too many sensors or publishing too frequently
- Reduce cron frequency or number of sensors

**Stale readings:**
- Add first reading discard pattern
- Test over 10+ minute period for gas sensors (warm-up time)

## After Adding Sensor

Run the publishing script manually a few times, then monitor the cron job logs:

```bash
tail -f /home/kleinmatic/Code/enviroplus-logger/sensor_log.txt
```

Verify data appears correctly in both Adafruit IO and Home Assistant (if enabled).
