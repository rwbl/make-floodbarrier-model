# make-project-floodbarrier-model
A LEGO Model to control the gate of a flood barrier model by a microcontroller with sensors, actuators and LEGO Power Functions. 

# Objectives
* To create a model of a flood barrier with a gate, in the style of the "Sperrwerk Wedeler Au" (barrier between the German rivers "Elbe" and "Wedeler Au").
* To obtain, in regular intervals, real tide data from the "Elbe".
* To control the gate depending high tide (flood) level = close the gate at Abs NHN river level threshold reached else open the gate.
* To display in a large number display the "Elbe" level Abs NHN = Absolute Standard Elevation Zero Level (cm).
* To display information in a small display, i.e. Abs NHN (cm), Rel PNP = Relative Tide Gauge Zero (cm), PNP=Tide Gauge Zero (m), Tide Ebb or Flood, Gate State Open or Close.
* To remote control the gate.
* To publish tide data.

_Note_: This make project is for private use only.

![flood-barrier-p-fv](https://user-images.githubusercontent.com/47274144/65248648-cc894600-daf2-11e9-989e-08f7e56f9e6e.PNG)

![flood-barrier-t](https://user-images.githubusercontent.com/47274144/65248662-d14dfa00-daf2-11e9-9f9a-8541ccdbc99a.PNG)

### Creation Rules
* Use as microcontroller an ESP8266 microcontroller
* Use standard LEGO components (LEGO® is a trademark of the LEGO Group)
* Minimize LEGO brick modifications
* LEGO Case with modular components
* Develop the microcontroller code with B4R
* Develop client solutions in Node-RED and B4J

## Hardware Parts
* 1x NodeMCU (ESP-12E) with Development Board
* 1x Infrared Transmitter (KY-005)
* 1x 4-Digit Display (Catalex 4-Digit Display)
* 1x OLED Display
* 1x REED Switch (KY-021)
* 1x Push-Button
* 1x LED Red
* 1x LED Green
* LEGO Power Functions Light (8870)
* LEGO Power Functions IR Speed Remote Control (8879)
* LEGO Power Functions Battery Box (8881)
* LEGO Power Functions M-Motor (8883)
* LEGO Power Functions IR Receiver (8884) 
* LEGO Bricks, LEGO Technic

## Software
* Arduino IDE 1.8.9 or higher
* B4R 2.8.0 or higher
* Node-RED 0.20.7 with Node-RED Dashboard 2.16.3

## Brief Solution
The model is built of LEGO® Bricks, LEGO® Technic™, LEGO® Power Functions controlled by a microcontroller with sensors & actuators attached.
The microcontroller connects to the WiFi network (hardcoded SSID & Key for now).
The river "Elbe" level information is obtained every 5 minutes, via HTTP from the German webservice PEGELONLINE.de.

The HTTP JSON response is parsed to get the river level current measurement "Relative PNP" and the reference value "PNP" which are used to calculate the "Abs NHN".
The "Abs NHN" value is compared to the previous "Abs NHN" value to determine the tide Ebb or Flood.
In addition, the gate state open or closed is set according river level threshold reached (=gate is closed) or below threshold (=gate is open).
The threshold is initially hardcoded, but can be set via client (i.e. Node-RED) using MQTT. If the threshold is reached, the gate closed if open else the gate opens if closed.

On every 5 minute update cycle, controlled via timer, the 4-Digit is display is set with the "Abs NHN" cm and the OLED display with the river levels "Abs NHN", "Rel PNP", "PNP", Tide Ebb or Flood, Gate State Open or Closed.
LED signals are used to indicate i the gate is moving (two white lights, the motor is running), LED red if the gate is closed, LED green if the gate is open.

The initial position of the gate is closed and is set via LEGO Power Functions IR Remote Control.
A push-button enabled to open or close the gate manually.

MQTT messages are published with the river level information (can be used to display a Node-RED Dashboard with gauges and charts), but also to control the gate, move the motor or set the river level threshold.

### Outline
See also TODO.md.
Some ideas come to mind planned for a next version.
* Display real signal depending gate state.
	* Requires additional LEDs (i.e. gate open: 1 white + 2 red; gate closed: 2 red). There are no GPIO pins left = requires multiple LED on one pin solution (i.e. Shift Register 74HC595 or MAX7219)
* REED switch to control gate close position (instead of using timer from open to close position).
	* Requires additional GPIO with pullup There are no GPIO pins left = no solution yet found.
* Configuration parameter setting EEPROM using B4J client or Node-RED: WiFi SSID&Key, Gate Close Threshold, MQTT Server IP
* MQTT parsing using extenal library instead of using ByteConverter with Split & IndexOf methods.
* Multi push-buttons to control the gate PF M-Motor open, close, reverse, forward.
* Request online sea weather forecast to display on OLED row 6 (wind,temperature).

## Wiring
![floodbarrier-c](https://user-images.githubusercontent.com/47274144/65248619-bed3c080-daf2-11e9-99fe-e938e12ad2e4.png)

```
IR Sender=NodeMCU
Signal=D2 (GPIO4) yellow
GND=GND black

TM1637=NodeMCU
CLK=D6 (GPIO12)
DIO=D7 (GPIO13)
VCC=3V3
GND=GND

OLED=NodeMCU
SDA=D4 (GPIO2)
SCL=D5 (GPIO14)
VCC=3V3
GND=GND

REED Switch=NodeMMCU
S=D1 (GPIO5)
VCC=3V3
GND=GND

Push-Button Gate Switch=NodeMCU
S=D3 (GPIO0)
GND=GND

LED GATE SIGNAL=NodeMCU
Gate open=green=RX (GPIO03)
Gate closed=green=D8 (GPIO15)
```

### HTTP
In regular intervals, a HTTP download request (url hardcoded) is used to get the river level information from a German webservice PEGELONLINE.de.
The HTTP response is a JSON string which is parsed (bit quick and dirty using various ByteConverter methods, i.e. Split,IndexOf and others with StringFromBytes to get the various river "Elbe" values.
Extract from the HTTP JSOn Response:
```
{...
  "timeseries": [
	{
      "currentMeasurement": {
        "timestamp": "2019-09-15T11:50:00+02:00",
        "value": 393.0,
        "trend": 1,
        "stateMnwMhw": "unknown",
        "stateNswHsw": "unknown"
      },
      "gaugeZero": {
        "unit": "m. ü. NHN",
        "value": -5.006,
        "validFrom": "2001-11-01"
      }
    }
  ]
...}
```
The keys used are

    timerseries[0].currentMeasurement.value for Abs PNP.
    timerseries[0].gaugeZero.value for PNP.

In method HTTP JobDone, the values are extracted (snippet):

    Dim bc As ByteConverter
    For Each value() As Byte In bc.Split(jobresponse, """value"": ")
      If bc.IndexOf(value, "trend".GetBytes) > 0 Then
        riverlevelRELPNP = bc.StringFromBytes(bc.SubString2(value, 0, bc.IndexOf(value, ",".GetBytes)))
      End If
      If bc.IndexOf(value, "valid".GetBytes) > 0 Then
        riverlevelPNP = bc.StringFromBytes(bc.SubString2(value, 0, bc.IndexOf(value, ",".GetBytes)))
	riverlevelPNPS = bc.StringFromBytes(bc.SubString2(value, 0, bc.IndexOf(value, ",".GetBytes)))
      End If
     Next
     riverlevelABSNHN = riverlevelRELPNP + (riverlevelPNP * 100)

### MQTT
A connection is made to a MQTT Broker (IP address hardcoded) running on a Raspberry Pi 4.
The B4R program publishes:
```
topic floodbarrier/riverlevelabsnhn ; payload NN cm
topic floodbarrier/riverlevelrelpnp ; payload NN cm
topic floodbarrier/riverlevelpnp ; payload NN m
topic floodbarrier/riverleveltide ; payload Flut or Ebbe
topic floodbarrier/gatestate ; payload 0 (closed), 1 (open)
```

The B4R program subscribes to:
```
topic floodbarrier/gateopen ; payload n/a - the topic is used to trigger action.
topic floodbarrier/gateclose ; payload n/a - the topic is used to trigger action.
topic floodbarrier/gatemotorfwd ; payload n/a - the topic is used to trigger action = move the PF M-Motor forward with lowest speed.
topic floodbarrier/gatemotorrev ; payload n/a - the topic is used to trigger action = move the PF M-Motor reverse with lowest speed.
topic floodbarrier/gatemotorbrk ; payload n/a - the topic is used to trigger action = break (=stop) the PF M-Motor.

topic floodbarrier/gatestate ; payload 0=closed,1=open
topic floodbarrier/gatestatethreshold ; payload NNN in cm

topic floodbarrier/riverlevel ; payload csv string: absnhn,relpnp,pnp,tide = this can be used to set the River "Elbe" levels via MQTT (by f.e. Node-RED) instead of HTTP. 
```

### LEGO
The model is built using mix of classic & technic bricks. No modifications made nor soldering of components.

### LEGO Power Functions (abbreviated as PF)

#### Wiring
PF Battery -> IR Receiver Green Output -> M-Motor | Lights

#### Gate Open / Close Speed
The PF M-Motor controls opening or closing the gate and is used to set the initial position of the gate (this via IR Speed Remote Control).
To avoid moving the gate too fast, a gear box has been built between the M-Motor and the gate.
The gear box has three times gear ratio of 3. This has been accomplished by using three times 8 and 24 teeth gears.
The M-Motor has a speed of ~220RPM. Speed reduction calculation (numbers rounded):
S1=220; S2=(220x8)/24=73; S3=(73x8)/24=24; S4=(24x8)/24=8 RPM. 

#### Gate Open / Close Indicator
When the PF M-Motor is running to either open or close the gate, the PF Light is on (two white lights).

#### Gate Open / Close Movement
The initial state of the gate must be set to closed. The REED switch has state 1 (true).

When the gate opens, the PF M-Motor is running Forward (FWD2) for 4 seconds (with speed FWD2 defined in library rPowerFunctions).
When the gate closes, the PF M-Motor is running Reverse (REV2) till the REED switch gets state 0 (false) as the ecognised by the magnet on the back of the gate.
The 4 seconds are controlled by a timer, i.e. 4 cycles a 1000 ms.

The gate state are using global vars:

    'open (gateState=1),closed (gateState=0)
    Private gateState As Byte = 0
    'ABSNHN >= 210 close the gate else open
    Private gateStateThreshold As Int = 210

__Note__
for next version considering to use a second REED switch (but not GPIO pins left anymore).

### 4-Digit Display (Catalex 4-Digit Display)
The display shows the river "Elbe" level in "Abs NHN" in cm.
This is a 3 digit positive or negative value.
The library rTM1637 is used to set the value with the minus sign.
If the value is negative, then the method SetSegments displays the minus sign.
The minus sign is defined as

    Private displaySegments(1) As Byte
    
To display the minus sign = the segment G bit set, all other segents are not set.
ParseInt with radix 2 for the string with length 8 holding the bits for the segment

    displaySegments(0) = Bit.ParseInt("01000000",2)

The 4 digits used: most left set by SetSegments, the other 3 are set most right starting at 2 pos (=index 1)
[0][1][2][3] > [minus or empty][N][N][N]

    If riverLevel < 0 Then
	    'Display 3 digits starting at pos 1
	    tmRiverLevel.ShowNumberDec2(Abs(Round(riverLevel)), False, 3, 1)
	    'Display segment starting leftmost - only one segment is used to show the - sign
	    tmRiverLevel.SetSegments(displaySegments,0)
    Else
	    tmRiverLevel.ShowNumberDec2(Abs(Round(riverLevel)), False, 4, 0)
    End If

### OLED Display
The display shows river "Elbe" various levels + Tide (Ebb or Flood; German text used Ebbe, Flut) + Gate State (Open or Closed; German text used AUF, ZU).
The B4R library rESPOLED1608 is used to display 8 rows (0-7) with up-to 16 columns (0-15).
Each time the display is updated, the rows are cleared (tmDisplay.Clear) prior printing new rows.

    Row 0: Not used
    Row 1: Abs NHN in cm
    Row 2: Relative PNP in cm
    Row 3: PNP in m
    Row 4: Tide: Ebbe or Tide: Flut
    Row 5: Tor: AUF or Tor: ZU
    Row 6: Not used
    Row 7: Not used - but can be used to display Version

In case of an error, the error message is displayed on row 1.

_Note_: The columns start at position 1 and not 0.

## Node-RED
![flood-barrier-nr-f](https://user-images.githubusercontent.com/47274144/65248639-c72bfb80-daf2-11e9-919f-35ba0d2f5841.PNG)

![flood-barrier-nr-d](https://user-images.githubusercontent.com/47274144/65248627-c3987480-daf2-11e9-8daf-1a082f56a4b5.PNG)

## Soure Code
The source code is well documented.
See files b4r-source.zip, b4r-additional-libraries.zip, node-red.flow.

## Development Notes

### B4R IDE
Board NodeMCU 1.0 (ESP-12E Module) with 4MB.
IDE Options > Configure Process Timeout set to 120.

### B4R IDE Compile & Upload NodeMCU
1. Turn on the LEGO Power Functions Battery Box
2. Set Gate to CLOSE position using the LEGO Power Functions IR Remote Control or Push-Button
3. Turn the LEGO Power Functions M-Motor OFF
4. Compile & Upload
5. Turn the LEGO Power Functions M-Motor ON

### Switch the Model ON
1. Turn the LEGO Power Functions M-Motor ON
2. Set the Gate to CLOSE position using the LEGO Power Functions IR Remote Control
3. Power ON the NodeMCU

### Switch the Model OFF
1. Set the Gate to CLOSE position using the LGO Power Functions IR Remote Control or Push-Button or Node-RED Client
2. Turn the LEGO Power Functions Motor OFF
3. Power OFF the NodeMCU

### Credits
* Anywhere Software for providing B4R (and of course the whole B4X suite) [Info](https://www.b4x.com/)
* Friends-of-Fritzing foundation for fritzing [Info](https://fritzing.org)
* jurriaan for the LEGO Power Functions library [Info](https://github.com/jurriaan/Arduino-PowerFunctions)
* remoteme for the ESP8266 OLED library [Info](https://github.com/remoteme/esp8266-OLED)

### Licence
GNU GENERAL PUBLIC LICENSE Version 3, 29 June 2007

