# Smappee EV Card

A custom Lovelace card for Home Assistant to control your Smappee EV charging station.

## Features

- **Start** charging (with 30 second delay)
- **Slow** charging (6A, to avoid capacity tariff costs in Flanders)
- **Pause** charging
- **Stop** charging
- **Available/Unavailable** toggle with color feedback
- **Charging mode** selector (NORMAL / SOLAR / SMART) with slider
- **Charging speed** slider with live kW calculation
- Automatic single-phase / three-phase detection

---

## Requirements

### HACS Integrations
- [smappee_ev](https://github.com/myny-git/smappee_ev) by myny-git

### HACS Frontend
- [layout-card](https://github.com/thomasloven/lovelace-layout-card) by thomasloven
- [button-card](https://github.com/custom-cards/button-card) by RomRider
- [tailwindcss-template-card](https://github.com/usernein/tailwindcss-template-card)

---

## Installation

### 1. Install Dependencies
Install all HACS integrations and frontend cards listed above.

### 2. configuration.yaml
Add this line to your `/config/configuration.yaml`:
```yaml
script: !include scripts.yaml
```

### 3. Scripts
Add the following to `/config/scripts.yaml`:
```yaml
start_charging_with_delay:
  alias: "Start charging (delay)"
  mode: single
  sequence:
    - delay: "00:00:30"
    - service: button.press
      target:
        entity_id: button.smappee_ev_5130104006_start_charging_1

stop_charging:
  alias: "Stop charging"
  sequence:
    - service: button.press
      target:
        entity_id: button.smappee_ev_5130104006_stop_charging_1
```

### 4. Helpers
Create the following helpers in **Settings → Devices & Services → Helpers → + Create Helper → Number**:

| Name | Entity ID | Min | Max | Step |
|------|-----------|-----|-----|------|
| charging_mode_number | `input_number.charging_mode_number` | 0 | 2 | 1 |
| laadsnelheid_slider_waarde | `input_number.laadsnelheid_slider_waarde` | 6 | 32 | 1 |

### 5. Background Image
Upload `KIAEV9.png` to `/config/www/KIAEV9.png` on your Home Assistant instance.  
This makes it accessible via the URL `/local/KIAEV9.png` which the card uses.

> 💡 You can upload files to `/config/www/` using the **File Editor** add-on, **Samba share**, or via **SSH**.  
> If the `www` folder does not exist yet, create it manually inside `/config/`.

### 6. Add the Card
1. Go to your Dashboard
2. Click the pencil icon (edit mode)
3. Click **+ Add Card**
4. Scroll down and select **Manual**
5. Delete the placeholder YAML
6. Paste the full card YAML (see below)
7. Click **Save**

---

## Entity IDs
All entity IDs use the serial number of your Smappee device (`5130104006`).  
If your serial number is different, replace all occurrences of `5130104006` with your own.

You can find your serial number in **Settings → Devices**.

---

## Card Sections

### Actions
Start, Slow, Pause, Stop, and Available/Unavailable buttons.  
Buttons turn red when the station is set to unavailable.

### Charging Mode
Displays the current charging mode (NORMAL / SOLAR / SMART).  
Use the slider or the icons to switch between modes.

### Charging Speed
Shows maximum and current charging speed in kW.  
Automatically detects single-phase (max 7.4 kW) or three-phase (max 11 kW).  
Use the slider to adjust the charging speed.

> ⚠️ **Note for Flanders:** A capacity tariff (*capaciteitstarief*) of €4.50/kWh is in effect.  
> Use **Slow** charging (6A) to avoid high peak costs.

---

## Full Card YAML

```yaml
type: custom:layout-card
layout_type: grid
layout:
  grid-template-columns: 1fr 1fr 1fr 1fr 1fr
  grid-template-rows: auto 14% auto auto
  padding: 2px 5px 20px 5px
  margin: 0px 0px 0px 0px
  height: 100%
cards:
  - type: picture
    image: /local/KIAEV9.png
    view_layout:
      grid-column: 1 / span 5
      grid-row: 1
  - type: custom:button-card
    styles:
      card:
        - background: null
        - box-shadow: none
        - border: none
    view_layout:
      grid-column: 1 / span 5
      grid-row: 1
  - type: custom:button-card
    icon: mdi:play-circle-outline
    name: Start
    tap_action:
      action: call-service
      service: script.start_charging_with_delay
    styles:
      card:
        - "--ha-card-border-width": 0px
        - background: |
            [[[
              if (states['switch.smappee_ev_5130104006_station_available'].state == 'off')
                return 'var(--error-color)';
              return null;
            ]]]
        - color: var(--primary-text-color)
        - border-radius: 5px
        - height: 100%
        - padding: 2px 3px
    view_layout:
      grid-column: 1
      grid-row: 2
      align-self: center
      justify-self: center
  - type: custom:button-card
    icon: mdi:play-speed
    name: Slow
    tap_action:
      action: call-service
      service: number.set_value
      service_data:
        entity_id: number.smappee_ev_5130104006_max_charging_speed_1
        value: 6
    styles:
      card:
        - "--ha-card-border-width": 0px
        - background: |
            [[[
              if (states['switch.smappee_ev_5130104006_station_available'].state == 'off')
                return 'var(--error-color)';
              return null;
            ]]]
        - color: var(--primary-text-color)
        - border-radius: 10px
        - height: 100%
        - padding: 2px 5px
    view_layout:
      grid-column: 2
      grid-row: 2
      align-self: center
      justify-self: center
  - type: custom:button-card
    icon: mdi:pause-circle-outline
    name: Pause
    tap_action:
      action: call-service
      service: button.press
      service_data:
        entity_id: button.smappee_ev_5130104006_pause_charging_1
    styles:
      card:
        - "--ha-card-border-width": 0px
        - background: |
            [[[
              if (states['switch.smappee_ev_5130104006_station_available'].state == 'off')
                return 'var(--error-color)';
              return null;
            ]]]
        - color: var(--primary-text-color)
        - border-radius: 10px
        - height: 100%
        - padding: 2px 5px
    view_layout:
      grid-column: 3
      grid-row: 2
      align-self: end
      justify-self: center
  - type: custom:button-card
    icon: mdi:stop-circle-outline
    name: Stop
    tap_action:
      action: call-service
      service: script.turn_on
      service_data:
        entity_id: script.stop_charging
    styles:
      card:
        - "--ha-card-border-width": 0px
        - background: |
            [[[
              if (states['switch.smappee_ev_5130104006_station_available'].state == 'off')
                return 'var(--error-color)';
              return null;
            ]]]
        - color: var(--primary-text-color)
        - border-radius: 10px
        - height: 100%
        - padding: 2px 5px
    view_layout:
      grid-column: 4
      grid-row: 2
      align-self: center
      justify-self: center
  - type: custom:button-card
    entity: switch.smappee_ev_5130104006_station_available
    show_state: false
    icon: |
      [[[
        return entity.state == 'on' ? 'mdi:power-plug' : 'mdi:power-plug-off';
      ]]]
    name: |
      [[[
        return entity.state == 'on' ? 'Available' : 'Unavailable';
      ]]]
    tap_action:
      action: toggle
    styles:
      card:
        - "--ha-card-border-width": 0px
        - background: |
            [[[
              if (entity.state == 'on')
                return 'var(--success-color)';
              else
                return 'var(--error-color)';
            ]]]
        - color: var(--primary-text-color)
        - border-radius: 10px
        - height: 100%
        - padding: 2px 5px
    view_layout:
      grid-column: 5
      grid-row: 2
      align-self: center
      justify-self: center
  - type: custom:tailwindcss-template-card
    ignore_line_breaks: true
    always_update: false
    parse_jinja: true
    entities:
      - select.smappee_ev_5130104006_charging_mode_1
      - input_number.charging_mode_number
    view_layout:
      grid-column: 1 / span 5
      grid-row: 3
      align-self: center
      justify-self: center
    entity: ""
    content: |
      <ha-card class="w-full p-1">
        <div class="grid grid-cols-[1fr_4fr] items-center gap-2.5 font-sans">
          <div class="flex flex-col items-center text-xs">
            <ha-icon icon="mdi:ev-station" class="text-[28px] mb-1"></ha-icon>
            <div class="text-gray-500 text-xs">
              {{ states('select.smappee_ev_5130104006_charging_mode_1') }}
            </div>
          </div>
          <div class="relative flex flex-col items-center w-full h-[40px]">
            <div class="flex justify-between w-full -mx-1 mb-1 mt-[-6px]">
              <ha-icon
                icon="mdi:transmission-tower"
                class="text-[22px] cursor-pointer {% if is_state('select.smappee_ev_5130104006_charging_mode_1', 'NORMAL') %}text-red-500{% else %}text-gray-500{% endif %}"
                onClick="hass.callService('select', 'select_option', {entity_id: 'select.smappee_ev_5130104006_charging_mode_1', option: 'NORMAL'})"
              ></ha-icon>
              <ha-icon
                icon="mdi:white-balance-sunny"
                class="text-[22px] cursor-pointer {% if is_state('select.smappee_ev_5130104006_charging_mode_1', 'SOLAR') %}text-yellow-500{% else %}text-gray-500{% endif %}"
                onClick="hass.callService('select', 'select_option', {entity_id: 'select.smappee_ev_5130104006_charging_mode_1', option: 'SOLAR'})"
              ></ha-icon>
              <ha-icon
                icon="mdi:car-clock"
                class="text-[22px] cursor-pointer {% if is_state('select.smappee_ev_5130104006_charging_mode_1', 'SMART') %}text-blue-500{% else %}text-gray-500{% endif %}"
                onClick="hass.callService('select', 'select_option', {entity_id: 'select.smappee_ev_5130104006_charging_mode_1', option: 'SMART'})"
              ></ha-icon>
            </div>
            <div class="absolute bottom-[1px] w-[97%] h-[5px] bg-gray-300 rounded-sm left-1/2 -translate-x-1/2"></div>
            <div class="absolute bottom-[1px] w-[6px] h-[12px] bg-gray-500 transform -translate-x-1/2" style="left: 2.5%;"></div>
            <div class="absolute bottom-[1px] w-[6px] h-[12px] bg-gray-500 transform -translate-x-1/2" style="left: 50%;"></div>
            <div class="absolute bottom-[1px] w-[6px] h-[12px] bg-gray-500 transform -translate-x-1/2" style="left: 97.5%;"></div>
            <input
              type="range"
              min="0"
              max="2"
              step="1"
              class="absolute bottom-[-5px] w-[97%] bg-transparent appearance-none z-10 m-0 slider left-1/2 -translate-x-1/2"
              value="{{ iif(is_number(states('input_number.charging_mode_number') | float), states('input_number.charging_mode_number') | int, 0) }}"
              onInput="{
                const value = parseInt(this.value);
                hass.callService('input_number', 'set_value', {entity_id: 'input_number.charging_mode_number', value: value});
                const modeMap = {0: 'NORMAL', 1: 'SOLAR', 2: 'SMART'};
                hass.callService('select', 'select_option', {entity_id: 'select.smappee_ev_5130104006_charging_mode_1', option: modeMap[value]});
              }"
            >
          </div>
        </div>
      </ha-card>
      <style>
        .slider::-webkit-slider-thumb {
          -webkit-appearance: none;
          width: 8px;
          height: 20px;
          border: 5px solid {% if is_state('select.smappee_ev_5130104006_charging_mode_1', 'NORMAL') %}red{% elif is_state('select.smappee_ev_5130104006_charging_mode_1', 'SOLAR') %}#f4b400{% elif is_state('select.smappee_ev_5130104006_charging_mode_1', 'SMART') %}blue{% else %}red{% endif %};
          border-radius: 30%;
          background: white;
          cursor: pointer;
          margin-top: -8px;
        }
        .slider::-moz-range-thumb {
          width: 8px;
          height: 20px;
          border: 5px solid {% if is_state('select.smappee_ev_5130104006_charging_mode_1', 'NORMAL') %}red{% elif is_state('select.smappee_ev_5130104006_charging_mode_1', 'SOLAR') %}#f4b400{% elif is_state('select.smappee_ev_5130104006_charging_mode_1', 'SMART') %}blue{% else %}red{% endif %};
          border-radius: 30%;
          background: white;
          cursor: pointer;
        }
      </style>
  - type: custom:tailwindcss-template-card
    ignore_line_breaks: true
    view_layout:
      grid-column: 1 / span 5
      grid-row: 4
      align-self: end
      justify-self: center
    entities:
      - sensor.smappee_ev_5130104006_connector_1_power
      - sensor.smappee_ev_5130104006_connector_1_current_l3
      - input_number.laadsnelheid_slider_waarde
    content: |
      <ha-card class="w-full p-1">
        <div class="grid grid-cols-[1fr_4fr] items-center gap-2.5 font-sans">
          <div class="flex flex-col items-center text-xs">
            <div class="text-gray-400 text-xs mb-1">
              {% set l3_state = states('sensor.smappee_ev_5130104006_connector_1_current_l3') %}
              {% set l3_exists = l3_state not in ['unavailable', 'unknown', 'none', 'undefined'] %}
              {% set phase_count = 3 if l3_exists else 1 %}
              {% set slider_value = states('input_number.laadsnelheid_slider_waarde') | int %}
              {% if phase_count == 1 %}
                {% set max_wattage = 7.4 %}
              {% else %}
                {% set max_wattage = 11 %}
              {% endif %}
              {{ max_wattage }} kW Max
            </div>
            <div class="text-gray-400 text-xs">
              {{ states('sensor.smappee_ev_5130104006_connector_1_power') | float | round(0) }} W Now
            </div>
            <div class="flex flex-row justify-center gap-2 w-full mt-1">
              <div class="text-[12px] {{ 'text-blue-400' if phase_count == 1 else 'text-gray-400' }}">1~</div>
              <div class="text-[12px] {{ 'text-blue-400' if phase_count == 3 else 'text-gray-400' }}">3~</div>
            </div>
          </div>
          <div class="relative flex flex-col items-center w-full h-[70px]">
            <div id="roundedValue" class="absolute bottom-[38px] text-[25px] text-gray-400">
              {{ (slider_value * 230 * phase_count / 1000) | round(1) }} kW
            </div>
            <div class="absolute bottom-[22px] w-[97%] h-[5px] bg-gray-300 rounded-sm z-10 left-1/2 -translate-x-1/2"></div>
            <div class="absolute bottom-[16px] w-[6px] h-[12px] bg-gray-500 transform -translate-x-1/2 -translate-y-1/2 z-20" style="left: 2.5%;"></div>
            <div class="absolute bottom-[16px] w-[6px] h-[12px] bg-gray-500 transform -translate-x-1/2 -translate-y-1/2 z-20" style="left: 97.5%;"></div>
            <div class="absolute bottom-[50px] w-[97%] flex justify-between left-1/2 -translate-x-1/2">
              <div class="text-[10px] text-gray-400">1.4 kW</div>
              <div class="text-[10px] text-gray-400">
                {% if phase_count == 1 %}7.4 kW{% else %}11 kW{% endif %}
              </div>
            </div>
            <input
              type="range"
              min="6"
              max="{% if phase_count == 1 %}32{% else %}16{% endif %}"
              step="1"
              class="absolute bottom-[20px] w-[97%] bg-transparent appearance-none z-30 m-0 slider left-1/2 -translate-x-1/2"
              value="{{ slider_value }}"
              oninput="
                const value = parseInt(this.value);
                const phaseCount = {{ phase_count }};
                document.getElementById('roundedValue').textContent = (value * 230 * phaseCount / 1000).toFixed(1) + ' kW';
              "
              onchange="
                const value = parseInt(this.value);
                hass.callService('input_number', 'set_value', {
                  entity_id: 'input_number.laadsnelheid_slider_waarde',
                  value: value
                });
              "
            >
          </div>
        </div>
      </ha-card>
      <style>
        .slider::-webkit-slider-thumb {
          -webkit-appearance: none;
          width: 8px;
          height: 20px;
          border: 5px solid #60a5fa;
          border-radius: 30%;
          background: white;
          cursor: pointer;
        }
        .slider::-moz-range-thumb {
          width: 8px;
          height: 20px;
          border: 5px solid #60a5fa;
          border-radius: 30%;
          background: white;
          cursor: pointer;
        }
      </style>
```
