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

Here, you can start, pause, and stop the charging, or choose to charge with one phase or use slow charging. 
Slow charging is implemented to avoid high capacity costs, which are currently in effect in Flanders. 
This is also the section where you can set the charging station as available or unavailable


## Charging Mode
<img width="445" height="64" alt="image" src="https://github.com/user-attachments/assets/f8e492e5-9dd8-4042-aa46-27dfcd8b065d" />

This part shows on the left part the charging mode, with the slider one can change this 


## Charging speed

<img width="445" height="64" alt="image" src="https://github.com/user-attachments/assets/ce99fd71-a8bc-40b8-977b-c54ea328740a" />

On the left, you can see the maximum charging speed (11 kW). This value can be adjusted in the Smappee app.   
In the Smappee settings, it is referred to as "Fail-safe maximum current 15A."   
Together with the car, this setting determines the maximum charging speed.   
For a single-phase installation, the calculation is 230V × 15A = 3.5 kW.   
For a three-phase installation, it is 230V × 15A × 3, which is the maximum most cars can handle.  
In the Flemish part of Belgium, there is a "capaciteitstarief" (capacity tariff) of €4.50 per kWh. People who don’t drive much should be aware that charging at full speed could cost them roughly €600 extra for little to no added convenience.  
Below this, you can see the actual charging speed, and underneath, it indicates whether the installation is single-phase or three-phase.  

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
  # Background image (upload to /config/www/)
  - type: picture
    image: /local/torres-evx.png  # Replace with your image
    view_layout:
      grid-column: 1 / span 5
      grid-row: 1

  # START
  - type: custom:button-card
    icon: mdi:play-circle-outline
    name: Start
    tap_action:
      action: call-service
      service: button.press
      service_data:
        entity_id: button.smappee_ev_5130104006_start_charging_1
    styles:
      card:
        - "--ha-card-border-width": 0px
        - background: >
            [[[ if (states['switch.smappee_ev_5130104006_station_available'].state == 'off') return 'var(--error-color)'; return null; ]]]
        - color: var(--primary-text-color)
        - border-radius: 5px
        - height: 100%
        - padding: 2px 3px
    view_layout:
      grid-column: 1
      grid-row: 2
      align-self: center
      justify-self: center

  # SLOW (6A low peak for capacity tariff)
  - type: custom:button-card
    icon: mdi:play-speed
    name: Slow
    tap_action:
      action: call-service
      service: number.set_value
      service_data:
        value: 6
        entity_id: number.smappee_ev_5130104006_max_charging_speed_1
    styles:
      card:
        - "--ha-card-border-width": 0px
        - background: >
            [[[ if (states['switch.smappee_ev_5130104006_station_available'].state == 'off') return 'var(--error-color)'; return null; ]]]
        - color: var(--primary-text-color)
        - border-radius: 10px
        - height: 100%
        - padding: 2px 5px
    view_layout:
      grid-column: 2
      grid-row: 2
      align-self: center
      justify-self: center

  # PAUSE
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
        - background: >
            [[[ if (states['switch.smappee_ev_5130104006_station_available'].state == 'off') return 'var(--error-color)'; return null; ]]]
        - color: var(--primary-text-color)
        - border-radius: 10px
        - height: 100%
        - padding: 2px 5px
    view_layout:
      grid-column: 3
      grid-row: 2
      align-self: end
      justify-self: center

  # STOP
  - type: custom:button-card
    icon: mdi:stop-circle-outline
    name: Stop
    tap_action:
      action: call-service
      service: button.press
      service_data:
        entity_id: button.smappee_ev_5130104006_stop_charging_1
    styles:
      card:
        - "--ha-card-border-width": 0px
        - background: >
            [[[ if (states['switch.smappee_ev_5130104006_station_available'].state == 'off') return 'var(--error-color)'; return null; ]]]
        - color: var(--primary-text-color)
        - border-radius: 10px
        - height: 100%
        - padding: 2px 5px
    view_layout:
      grid-column: 4
      grid-row: 2
      align-self: center
      justify-self: center

  # AVAILABILITY (add if button exists, else use switch)
  - type: custom:button-card
    entity: switch.smappee_ev_5130104006_station_available  # Or button
    show_state: false
    icon: >
      [[[ return entity.state == 'on' ? 'mdi:power-plug' : 'mdi:power-plug-off'; ]]]
    name: >
      [[[ return entity.state == 'on' ? 'Available' : 'Unavailable'; ]]]
    tap_action:
      action: toggle  # Or button.press if button
    styles:
      card:
        - "--ha-card-border-width": 0px
        - height: 100%
        - padding: 2px 5px
    view_layout:
      grid-column: 5
      grid-row: 2
      align-self: center
      justify-self: center

  # SPEED SLIDER (row 3)
  - type: entities
    entities:
      - entity: number.smappee_ev_5130104006_max_charging_speed_1
        name: Charging Speed (A)
    view_layout:
      grid-column: 1 / span 5
      grid-row: 3


```

scripts.yaml
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

pause_charging_1:
  alias: "Pause charging"
  sequence:
    - service: button.press
      target:
        entity_id: button.smappee_ev_5130104006_pause_charging_1

slow_charging:
  alias: "Slow charging (6A)"
  sequence:
    - service: number.set_value
      target:
        entity_id: number.smappee_ev_5130104006_max_charging_speed_1
      data:
        value: 6
```
