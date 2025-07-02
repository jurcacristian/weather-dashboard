# XIAO 7.5" ePaper Weather Dashboard

A comprehensive weather dashboard for the Seeed Studio XIAO 7.5" ePaper Panel, integrating with Home Assistant and AccuWeather to display real-time weather data and 5-day forecasts.

![Weather Dashboard](images/dashboard-preview.jpg) <!-- Add your actual dashboard photo -->

## Features

- **Real-time Weather Display**
  - Large current temperature display
  - Current weather conditions with detailed description
  - High/Low temperatures for today
  
- **Comprehensive Weather Data**
  - Air Quality index
  - Sun hours for the day
  - Precipitation amount
  - Humidity percentage
  - Atmospheric pressure
  - Wind speed and direction
  - UV Index
  - Cloud coverage and ceiling height

- **5-Day Weather Forecast**
  - Daily high/low temperatures
  - Weather condition icons (sunny, cloudy, rainy, etc.)

- **Smart Power Management**
  - Dynamic refresh schedule:
    - Update every 15 minutes between 6 AM and 10 AM
    - Update every hour from 10 AM to midnight
    - Sleep mode from midnight to 6 AM
  - Optimized for 2000mAh battery (3+ months battery life...this is still to be check, but i would not expect to last more 1 month. But a more granular schedule is possible like having more sleep when nobody is arround.)

## Hardware Requirements

- **Seeed Studio XIAO 7.5" ePaper Panel** (includes):
  - XIAO ESP32-C3 microcontroller
  - 7.5" Waveshare ePaper display (800×480 resolution)
  - 2000mAh battery
  - Pre-wired connections in enclosed case
  - USB Type-C for programming
  - purchase link:  https://www.seeedstudio.com/XIAO-7-5-ePaper-Panel-p-6416.html 

## Software Prerequisites

### Home Assistant
- Home Assistant Core 2024.1 or newer
- ESPHome add-on installed
- AccuWeather integration configured

### AccuWeather Integration
The following AccuWeather sensors must be available in Home Assistant:
- `weather.home` - Main weather entity
- `sensor.home_realfeel_temperature_max_day_0` through `day_4`
- `sensor.home_realfeel_temperature_min_day_0` through `day_4`
- `sensor.home_hours_of_sun_day_0`
- `sensor.home_precipitation`
- `sensor.home_condition_day_0` - Detailed weather description
- `sensor.home_air_quality_day_0`
- `sensor.home_cloud_ceiling_day_0`

## Installation

### 1. Home Assistant Configuration

Add the following to your `configuration.yaml`:

```yaml
sensor:
  - platform: template
    sensors:
      accuweather_sensor:
        friendly_name: "AccuWeather Display Data"
        unique_id: "accuweather_display_sensor_v1"
        value_template: "{{ now().strftime('%Y-%m-%d %H:%M') }}"
        attribute_templates:
          # Current conditions from AccuWeather
          current_temperature: "{{ state_attr('weather.home', 'temperature') | float(0) | round(0) }}"
          current_condition: "{{ states('weather.home') }}"
          current_humidity: "{{ state_attr('weather.home', 'humidity') | float(0) | round(0) }}"
          current_pressure: "{{ state_attr('weather.home', 'pressure') | float(0) | round(0) }}"
          wind_speed: "{{ state_attr('weather.home', 'wind_speed') | float(0) | round(1) }}"
          wind_bearing: "{{ state_attr('weather.home', 'wind_bearing') | int(0) }}"
          uv_index: "{{ state_attr('weather.home', 'uv_index') | float(0) | round(1) }}"
          cloud_coverage: "{{ state_attr('weather.home', 'cloud_coverage') | int(0) }}"
          
          # Additional sensors
          hours_of_sun: "{{ states('sensor.home_hours_of_sun_day_0') | float(0) | round(1) }}"
          precipitation: "{{ states('sensor.home_precipitation') | float(0) | round(1) }}"
          today_condition_text: "{{ states('sensor.home_condition_day_0') }}"
          air_quality: "{{ states('sensor.home_air_quality_day_0') }}"
          cloud_ceiling: "{{ states('sensor.home_cloud_ceiling_day_0') | float(0) | round(1) }}"
          
          # Temperature forecasts for 5 days
          today_high: "{{ states('sensor.home_realfeel_temperature_max_day_0') | float(0) | round(0) }}"
          today_low: "{{ states('sensor.home_realfeel_temperature_min_day_0') | float(0) | round(0) }}"
          # ... (continues for day1 through day4)
```

Restart Home Assistant after adding the configuration.

### 2. ESPHome Configuration

1. In ESPHome, create a new device configuration
2. Copy the `xiao-seed.yaml` configuration from this repository
3. Update the following in the configuration:
   - WiFi credentials (or use secrets.yaml)
   - API encryption key
   - OTA password

### 3. Initial Setup

1. Connect the XIAO ePaper Panel via USB-C
2. Open the stand to access the programming buttons
3. Enter bootloader mode:
   - Hold BOOT button
   - Press RESET once while holding BOOT
   - Release RESET, then BOOT
   - Reconnect USB
4. In ESPHome dashboard, click INSTALL and choose "Plug into this computer"
5. Select the appropriate COM port
6. Wait for the initial flash to complete (approximately 5 minutes)

## Configuration Details

### Display Layout

The 800×480 display is divided into three main sections:

1. **Left Section (x: 0-400)**
   - Large current temperature
   - Today's high/low
   - Detailed weather description (word-wrapped, max 3 lines)

2. **Right Section (x: 400-800)**
   - Two columns of weather metrics
   - 10 different weather parameters

3. **Bottom Section (y: 340-480)**
   - 5-day forecast with icons
   - Update timestamp

### Weather Icons

The display draws custom weather icons based on conditions:
- **Sunny**: Circle with radiating lines
- **Cloudy**: Overlapping filled circles
- **Partly Cloudy**: Sun partially behind cloud
- **Rainy**: Cloud with rain drops
- **Snowy**: Snowflake pattern

### Power Management

The display uses smart scheduling to maximize battery life:
- More frequent updates during morning hours when weather changes rapidly
- Reduced updates in afternoon/evening
- Complete sleep during night hours
- Expected battery life: 3+ months on 2000mAh battery

## Customization

### Adjusting Refresh Schedule

Modify the `time` component in the ESPHome configuration:
### Example bellow updates the data every 10 minutes from 6 AM to 3PM . Everything else outside of this is considered to be sleep.
```yaml
time:
  - platform: homeassistant
    id: homeassistant_time
    on_time:
      - seconds: 0
        minutes: /10        # Update frequency
        hours: 6-14       # Active hours
        then:
          - component.update: epaper_display
```

### Adding/Removing Weather Metrics

1. Add the sensor to Home Assistant template sensor attributes
2. Add corresponding sensor in ESPHome configuration
3. Add display code in the lambda section

### Changing Fonts

The display uses Google Fonts (Roboto) in various sizes:
- `font_tiny`: 16px - timestamps and small labels
- `font_small`: 24px - weather data and labels
- `font_medium`: 30px - section headers
- `font_xlarge`: 185px - main temperature display

## Troubleshooting

### Display Not Updating
- Check if `sensor.accuweather_sensor` exists in Home Assistant Developer Tools
- Verify all AccuWeather sensors are available
- Check ESPHome logs for connection issues

### Wrong Data Displayed
- Ensure AccuWeather integration is properly configured
- Verify sensor names match your AccuWeather entity names
- Check Home Assistant template sensor for errors

### Display Flickering
- This is normal for ePaper displays during refresh
- Full refresh occurs every 30 partial updates

### Battery Life Issues
- Verify the refresh schedule isn't too frequent
- Check WiFi signal strength (weak signal drains battery)
- Ensure deep sleep is working during night hours

## Contributing

Feel free to submit issues and enhancement requests!

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Seeed Studio for the XIAO 7.5" ePaper Panel hardware
- ESPHome community for the Waveshare ePaper driver
- Home Assistant community for the platform
