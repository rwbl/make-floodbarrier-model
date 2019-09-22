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

#### External River information
Instead using the B4R update timer cycle (with HTTP) to get & update river information, subscribe to a MQTT message holding
the relevant information.
This information could be provided by for example Node-RED.
The Node-RED flow sends in reqular intervals, using the nodes _inject > change > http request > function > change > mqtt out_, the HTTP request, parses the result, built the csv string payload and publishes message (topic _floodbarrier/riverlevel_).
If doing so, the B4R update timer must be stopped, which can be done via a MQTT message as well.
The B4R program method _Mqtt_MessageArrived_ is used with the case item _mqttTopicRiverLevel_, which has already some initial coding.
This needs to be enhanced to align with the HTTP JobDone method.
I addition, a MQTT topic must be defined and subscribed to, i.e.

    Private mqttTopicUseExternalInformation As Byte = "floodbarrier/userexternalinformation" '0=NO,1=YES
and handled in _Mqtt_MessageArrived_
The default would be 0, thus using B4R program HTTP.
