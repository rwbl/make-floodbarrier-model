### Status 20190920

Ideas come to mind whilst making ...

#### Display real signal depending gate state
Requires additional LEDs (i.e. gate open: 1 white + 2 red; gate closed: 2 red).
There are no GPIO pins left = requires a multiple LEDs on one pin solution (i.e. Shift Register 74HC595 or MAX7219).

#### REED switch to control gate close position 
(instead of using timer from open to close position).
Requires additional GPIO with pullup There are no GPIO pins left = no solution yet found.

#### Configuration parameter setting EEPROM
Using B4J client or Node-RED: WiFi SSID&Key, Gate Close Threshold, MQTT Server IP.

#### MQTT parsing using external library
Instead using various ByteConverter methods, i.e. Split, IndexOf and others.

#### Multiple push-buttons
To control the gate PF M-Motor open, close, reverse, forward.

#### Request online sea weather forecast
To display on OLED row 6 (wind,temperature).
