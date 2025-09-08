# Smappee-card
This card will give you control over the charging process with Smappee
You'll need the Smappee EV integration from
[myny-git]([url)](https://github.com/myny-git/smappee_ev):

[![Add to my Home Assistant](https://my.home-assistant.io/badges/config_flow_start.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=myny-git&repository=smappee_ev&category=integration)

You need in your configuration.yaml:

```script: !include scripts.yaml```

<img width="445" height="422" alt="image" src="https://github.com/user-attachments/assets/2bd97dfc-c7c0-4af0-bcbc-e45d054d24d7" />

This card has 3 sections:
## Actions
<img width="445" height="64" alt="image" src="https://github.com/user-attachments/assets/daf0892c-1454-430f-8dcc-97bd57888f22" />

Here you can start, pauze and stop the charging or start with one phase or slow charging.
Slow charging is implemented to avoid hight capacity cost which we have now in Flanders.
This is also the section where we can set the charging station available or unavailable


## Charging Mode
<img width="445" height="64" alt="image" src="https://github.com/user-attachments/assets/f8e492e5-9dd8-4042-aa46-27dfcd8b065d" />

This part shows on the left part the charging mode, with the slider one can change this 


## Charging speed

<img width="445" height="64" alt="image" src="https://github.com/user-attachments/assets/ce99fd71-a8bc-40b8-977b-c54ea328740a" />

On the left one can see the maximum charging speed(11kWh), this is a value that can be changed in the Smappee.
In the Smappee settings it's called "Fail-safe maximum current 15A" for example, this togheter with the car decides the max charging speed.
In a one phase installation it's 230 x 15 which is 3,5 kWh
In a 3 phase installation it's 230x15x3 which is the maximum most cars can handle.
In the Flemish part of Belgium we have a "capaciteits tarief" which is 4,5 Euro/kWh.
People who don't drive a lot should be aware that full speed charging will cost them to roughly **600 EURO** extra for litle to no extra comfort.  

Below this we see the actual charging speed
Under this we see if it's a 3 phase installation or a 1 phase

Here is the code for the card itself:

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
    image: /local/torres-evx.png
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
              if (states['switch.smappee_ev_5130088735_station_available'].state == 'off')
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
      service: homeassistant.turn_on
      service_data:
        entity_id: script.stop_charging
    styles:
      card:
        - "--ha-card-border-width": 0px
        - background: |
            [[[
              if (states['switch.smappee_ev_5130088735_station_available'].state == 'off')
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
      service: input_boolean.turn_on
      service_data:
        entity_id: input_boolean.pause_charging_1
    styles:
      card:
        - "--ha-card-border-width": 0px
        - background: |
            [[[
              if (states['switch.smappee_ev_5130088735_station_available'].state == 'off')
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
      service: homeassistant.turn_on
      service_data:
        entity_id: script.stop_charging
    styles:
      card:
        - "--ha-card-border-width": 0px
        - background: |
            [[[
              if (states['switch.smappee_ev_5130088735_station_available'].state == 'off')
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
    entity: switch.smappee_ev_5130088735_station_available
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
      - select.smappee_ev_5130088735_charging_mode_1
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
              {{ states('select.smappee_ev_5130088735_charging_mode_1') }}
            </div>
          </div>
          <div class="relative flex flex-col items-center w-full h-[40px]">
            <div class="flex justify-between w-full -mx-1  mb-1 mt-[-6px]">
              <ha-icon
                icon="mdi:transmission-tower"
                class="text-[22px] cursor-pointer {% if is_state('select.smappee_ev_5130088735_charging_mode_1', 'NORMAL') %}text-red-500{% else %}text-gray-500{% endif %}"
                onClick="hass.callService('select', 'select_option', {entity_id: 'select.smappee_ev_5130088735_charging_mode_1', option: 'NORMAL'})"
              ></ha-icon>
              <ha-icon
                icon="mdi:white-balance-sunny"
                class="text-[22px] cursor-pointer {% if is_state('select.smappee_ev_5130088735_charging_mode_1', 'SOLAR') %}text-yellow-500{% else %}text-gray-500{% endif %}"
                onClick="hass.callService('select', 'select_option', {entity_id: 'select.smappee_ev_5130088735_charging_mode_1', option: 'SOLAR'})"
              ></ha-icon>
              <ha-icon
                icon="mdi:car-clock"
                class="text-[22px] cursor-pointer {% if is_state('select.smappee_ev_5130088735_charging_mode_1', 'SMART') %}text-blue-500{% else %}text-gray-500{% endif %}"
                onClick="hass.callService('select', 'select_option', {entity_id: 'select.smappee_ev_5130088735_charging_mode_1', option: 'SMART'})"
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
                hass.callService('select', 'select_option', {entity_id: 'select.smappee_ev_5130088735_charging_mode_1', option: modeMap[value]});
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
          border: 5px solid
            {% if is_state('select.smappee_ev_5130088735_charging_mode_1', 'NORMAL') %}red
            {% elif is_state('select.smappee_ev_5130088735_charging_mode_1', 'SOLAR') %}#f4b400
            {% elif is_state('select.smappee_ev_5130088735_charging_mode_1', 'SMART') %}blue
            {% else %}red{% endif %};
          border-radius: 30%;
          background: white;
          cursor: pointer;
          margin-top: -8px;
        }
        .slider::-moz-range-thumb {
          width: 8px;
          height: 20px;
          border: 5px solid
            {% if is_state('select.smappee_ev_5130088735_charging_mode_1', 'NORMAL') %}red
            {% elif is_state('select.smappee_ev_5130088735_charging_mode_1', 'SOLAR') %}#f4b400
            {% elif is_state('select.smappee_ev_5130088735_charging_mode_1', 'SMART') %}blue
            {% else %}red{% endif %};
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
      - sensor.smappee_ev_5130088735_connector_1_power
      - sensor.smappee_ev_5130088735_connector_1_current_l3
      - input_number.laadsnelheid_slider_waarde
    content: |
      <ha-card class="w-full p-1">
        <div class="grid grid-cols-[1fr_4fr] items-center gap-2.5 font-sans">
          <div class="flex flex-col items-center text-xs">
            <div class="text-gray-400 text-xs mb-1">
              {% set l3_state = states('sensor.smappee_ev_5130088735_connector_1_current_l3') %}
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
              {{ states('sensor.smappee_ev_5130088735_connector_1_power') | float | round(0) }} W Now
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
                {% if phase_count == 1 %} 7.4 kW {% else %} 11 kW {% endif %}
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
          border: 5px solid;
          border-radius: 30%;
          color: #60a5fa;
          cursor: pointer;
        
        }
        .slider::-moz-range-thumb {
          width: 8px;
          height: 20px;
          border: 5px solid
          border-radius: 30%;
          color: #60a5fa;
          cursor: pointer;
        }
      </style>

```

For the scripts I have the following:

<pre>Work in progres</pre>
