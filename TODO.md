### Status 20190920

Some further ideas have come to mind = planned for a next version.

#### Display real signal depending gate state
Requires additional LEDs (i.e. gate open: 1 white + 2 red; gate closed: 2 red).
There are no GPIO pins left = requires multiple LED on one pin solution (i.e. Shift Register 74HC595 or MAX7219).

#### REED switch to control gate close position 
(instead of using timer from open to close position).
Requires additional GPIO with pullup There are no GPIO pins left = no solution yet found.

#### Configuration parameter setting EEPROM
using B4J client or Node-RED: WiFi SSID&Key, Gate Close Threshold, MQTT Server IP.

#### MQTT parsing using external library
instead using various ByteConverter methods, i.e. Split, IndexOf and others.

#### Multi push-buttons
to control the gate PF M-Motor open, close, reverse, forward.

#### Request online sea weather forecast
to display on OLED row 6 (wind,temperature).
