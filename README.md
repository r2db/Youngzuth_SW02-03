# Youngzuth_SW02-03
## Hardware hacking of Youngzuth SW02-03 for Tasmota/ ESPHome use

Any modifications you do to your device are solely your responsibility and knowing that you may be placing yourself in imminent danger by doing so. This document comes without any warranties or guarantees. These devices use line voltage.
## _You can kill yourself or someone else by playing with line voltage._

## Tasmota Configuration:

![Tasmota Configuration](https://github.com/r2db/Youngzuth_SW02-03/blob/main/Tasmota.png)
Buttons 1, 2, and 3 are self-explanatory, as are relays 1, 2, and 3.

Relays 4, 5, and 6 are the LEDs for buttons 1, 2, and 3, respectively.

Relays 7 and 8 control the WiFi status LED. See below as this requires hardware modification beyond just a swap from the Tuya module to an ESP8266.

## Post-operative Photo

![Post-operative photo](https://github.com/r2db/Youngzuth_SW02-03/blob/main/Post-operative.png)
Notice the bodge wire from ADC0 to GPIO2. First, the solder mask was removed. Then, the trace was cut to ADC0. Finally, the trace was re-routed to GPIO2. When desoldering the old module it is very easy to lift the pad for GPIO2 as it is not connected to anything.

|            | GPIO2 Low | GPIO2 High |
| :---       | :----:    | :----:     |
| GPIO1 Low  | LED Off   | LED Red    |
| GPIO1 High | LED Blue  | LED Red    |

Note: If GPIO2 is high then the WiFi status LED will be red. If GPIO2 is low then pulling GPIO1 high will light the WiFi status LED as blue. This is a consequence of the way the board was manufactured. It is not possible to light both red and blue simultaneously. However, using a PWM output on GPIO2 you are able to get it to appear purple. Due to the specific RGB LED used in this device, it does appear to be a funky purple where you can see a halo of the component colors.

## ESPHome Configuration

```yaml
substitutions:
  # Youngzuth SW02-03
  device_name: sw02-03 # hostname & entity_id
  friendly_name: Youngzuth 3 Button Switch # Displayed in HA frontend
#  ip_address: 192.168.1.34

esphome:
  name: ${device_name}
  name_add_mac_suffix: true
  
esp8266:
  board: esp01_1m
  restore_from_flash: false

# Enable logging
logger:

# Enable Home Assistant API
api:

ota:
  password: !secret ota_password

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: ${friendly_name} Fallback Hotspot
    password: !secret hotspot_password
  # Optional manual IP
#  manual_ip:
#    static_ip: ${ip_address}
#    gateway: 192.168.1.1
#    subnet: 255.255.255.0

captive_portal:
   
web_server:
  port: 80
  auth:
    username: "root"
    password: !secret hotspot_password
  ota: true

binary_sensor:
  - platform: gpio
    pin:
      number: GPIO12
      mode:
        input: true
        pullup: true
      inverted: true
    name: "Top Button"
    id: top_button
  - platform: gpio
    pin:
      number: GPIO5
      mode:
        input: true
        pullup: true
      inverted: true
    name: "Middle Button"
    id: middle_button
  - platform: gpio
    pin:
      number: GPIO3
      mode:
        input: true
        pullup: true
      inverted: true
    name: "Bottom Button"
    id: bottom_button
  - platform: status
    name: ${friendly_name} Switch Status
    id: sensor_status
    # Turn on the blue LED if the switch is connected to Home Assistant
    on_state:
      if:
        condition:
          api.connected
        then:
          light.turn_on: wifi_status_led
        else:
          light.turn_off: wifi_status_led

status_led:
  # Turn on the red LED if there is an error/ warning
  pin:
    number: GPIO2
    inverted: false
  id: red_status_led

output:
  - platform: esp8266_pwm
    pin: GPIO0
    id: top_led
  - platform: esp8266_pwm
    pin: GPIO14
    id: middle_led
  - platform: esp8266_pwm
    pin: GPIO16
    id: bottom_led
  - platform: esp8266_pwm
    pin: GPIO1
    id: wifi_blue_led

switch:
  - platform: gpio
    pin: GPIO13
    name: "Top Relay"
    id: top_relay
  - platform: gpio
    pin: GPIO15
    name: "Middle Relay"
    id: middle_relay
  - platform: gpio
    pin: GPIO4
    name: "Bottom Relay"
    id: bottom_relay

light:
  - platform: monochromatic
    name: "Top LED"
    output: top_led
  - platform: monochromatic
    name: "Middle LED"
    output: middle_led
  - platform: monochromatic
    name: "Bottom LED"
    output: bottom_led
  - platform: monochromatic
    name: "WiFi Blue"
    output: wifi_blue_led
    id: wifi_status_led
    internal: true

button:
  - platform: restart
    id: restart_button
    name: $friendly_name Restart
    entity_category: diagnostic

text_sensor:
  - platform: version
    name: $friendly_name ESPHome Version
    id: esphome_version
    hide_timestamp: True
  - platform: wifi_info
    ip_address:
      id: ip_address
      name: $friendly_name IP Address
    mac_address:
      name: $friendly_name Mac
      id: mac_address

sensor:
  - platform: uptime
    name: $friendly_name Uptime Sensor
  - platform: wifi_signal
    name: $friendly_name wifi signal
```
