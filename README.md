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

Note: If GPIO2 is high then the WiFi status LED will be red. If GPIO2 is low then pulling GPIO1 high will light the WiFi status LED as blue. This is a consequence of the way the board was manufactured. It is not possible to light both red and blue simultaneously. However, using a PWM output you might be able to get it to appear purple.
