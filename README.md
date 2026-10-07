# ESPHome + LVGL wall panel for Home Assistant

A swipeable touch panel for the living room: clock, thermostat with live boiler
status, weather, wind and rain compass, 7-day forecast, energy, awning, blinds
and a light. Written in ESPHome with LVGL, 800x480, labels in Catalan.

It was built to sit next to a boiler controlled from Home Assistant (see
[ha-vaillant-ebus-heating](https://github.com/gabychan/ha-vaillant-ebus-heating)),
but every page is independent and everything that belongs to one house lives in
a `substitutions` block at the top of `panel.yaml`.

## Status

This is a sanitized copy of the author's running panel. It was checked with
`esphome config` (ESPHome 2026.9.1) after replacing the Google fonts with a local
font, because the check environment could not download them; it was **not**
compiled in this exact form. The original, with the author's entity names, runs
on the author's panel.

Do not flash this file over a working panel of your own without filling in the
substitutions first: the defaults are placeholders.

## Hardware

An ESP32-S3 board with an 800x480 RGB display, a CH422G I/O expander (backlight
and resets), a GT911 touch controller, and an SHT3x temperature sensor on I2C
(address 0x44). The pins in `panel.yaml` are the author's; compare them with your
board's pinout. ESPHome warns about the strapping pins GPIO46 and GPIO3 (they are
the display's sync pins); the author's panel runs with them.

## Pages

Swipe left or right to move between pages. The panel returns to the clock after
60 seconds without touch, and the screen goes black when the presence sensor has
been off for 3 minutes (touch or presence wakes it).

| Page | What it shows | What it needs from Home Assistant |
| --- | --- | --- |
| Clock | Time, date, outside temperature, weather alert banner | A Meteocat station (`meteocat_station`) |
| Thermostat | Room temperature from the panel's own sensor, boiler status, setpoint with - and + | A `climate` entity (`thermostat_entity`); a sensor with the boiler's S.xx code (`boiler_state_entity`) |
| Weather | Temperature, condition, min and max, wind, humidity | A weather entity from the Meteocat integration |
| Sun and rain | Sunrise, sunset, chance of rain | `sunrise_entity`, `sunset_entity` (text "HH:MM"), `rain_probability_entity` |
| Wind and rain | Compass needle, direction, speed, gusts, rain | Meteocat station sensors |
| 7-day forecast | Bars with minimum and maximum | `week_forecast_entity` with attributes `maxs` and `mins` (comma-separated integers) |
| Energy | Solar, house, grid, and an appliance traffic light | Power sensors in W (`solar_power_entity`, `house_power_entity`, `grid_power_entity`) and a text sensor with `verd`, `ambre`, `vermell` or `nit` |
| Awning, Blinds, Light | Up, stop, down; right or left blind; toggle | `awning_entity`, `blind_right_entity`, `blind_left_entity`, `lamp_entity` |

If an entity does not exist in your Home Assistant, ESPHome reports it as
unavailable and the page shows `--`; nothing breaks. To remove a page, delete its
block under `lvgl.pages` and the sensors that update it.

The compass images in `images/` are simple placeholders generated for this
repository; replace them with your own 300x300 PNGs with transparency (needle
pointing up, pivot at the centre).

## The thermostat page

- The room temperature comes from the panel's own SHT3x. It sits next to the
  display and reads high, so calibrate `temp_offset`: put a reference thermometer
  about 30 cm to one side, compare after 15 to 20 minutes, and compare again at
  another time of day. The author needed -6.12 and, with it, the panel and the
  thermometer differed by 0.0 to 0.1 °C at two times of day. Do not trust a single
  measurement, and do not place the reference thermometer on top of the panel.
- The - and + buttons call `climate.set_temperature` in steps of 0.5 °C and
  ignore a touch that was part of a swipe, so brushing a button while paging does
  not change the setpoint. The panel cannot switch the thermostat between heat
  and off; do that from Home Assistant.
- The line under the temperature reads the boiler's status code and shows, for
  Vaillant ecoTEC plus codes: S.4 "escalfant", S.0 to S.3 "arrencant", S.5 to S.7
  "aturant", S.8 "en espera", S.10 to S.17 "aigua calenta", S.20 to S.28
  "arrencada en calent", S.30 "sense demanda", S.31 "en repòs", S.53 and S.54
  "espera (S.54)", S.97 "comprovant sensors", and "sense dades" when the sensor
  is unavailable. Any other code is shown as "Caldera: S.xx". Edit the
  `update_caldera` script for other boilers.

## Install

1. In the ESPHome builder, create a new device and paste `panel.yaml`. Put the
   `images/` folder next to it (in the ESPHome config folder).
2. Copy `secrets.yaml.example` to `secrets.yaml` and fill in your Wi-Fi.
3. Edit the `substitutions` block: names, timezone, town, entities and
   `temp_offset`.
4. Check that your board's pins match, validate, and install. The first build
   downloads Roboto from Google Fonts, so it needs internet access.

## Notes

- The interface is in Catalan. Texts are plain strings in the YAML; search and
  replace to translate them.
- Weather alerts use the Meteocat integration's `alerta_*` sensors, which report
  `opened` while an alert is active.

## Feedback

Issues and pull requests are welcome, especially pinouts for other boards.

## License

MIT, see [LICENSE](LICENSE). No warranty of any kind.
