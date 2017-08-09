# Paradox Multi-MQTT
Python-based 'middleware' that aims to use any method to connect to a Paradox Alarm, exposing the interface for monitoring and control via an MQTT Broker.
Currently access through the Serial port is implemented. As this a fork of the [ParadoxIP150v2](https://github.com/Tertiush/ParadoxIP150v2) project, access through the IP150 can be easily (re-)added (volunteers wanted). As I do not have access to the module, it is impossible for me to test it.

While I would welcome a merge with the origin project, the changes are substantial and I do not guarantee future compatibility.

Tested with the following environment:

* Python 2.7.13 
* [Mosquitto MQTT Broker v1.4.14](http://mosquitto.org)
* [OrangePi 2G-IOT](http://www.orangepi.org/OrangePi2GIOT/) through its internal Serial Port
* Ubuntu Server 16.04.3 LTS

## Steps to use it:
1.  Download the files in this repository and place it in some directory
2.  Edit the config.ini file to match your setup
3.  Run the script: Python Multi-MQTTv2.py


## What to expect:

The behaviour is similar to the one obtained with [ParadoxIP150v2](https://github.com/Tertiush/ParadoxIP150v2), __but there are some RELEVANT changes__. Please check this project documentation.

## Changes from the original project
* Using a python logging module, instead of print
* Polling the serial port is now always done. This may help to discover connectivity losses and will allow to obtain some stats (battery, and eventually zones)
* Changed the logic so that it handles Paradox messages without the headers required by the connectivity module (Serial or IP150).
* Abstracted the channel to a separate class.
* A new config file option was added (SERIAL). See the example.
* Changes on how to deal with Arming and events. 
* Because my scenario is not bandwidth limited, MQTT topics are now more verbose.
* Voltage values are processed
* New KeepAlive mechanism now maintains a constant connection with the module.
* Many small fixes
* __PLANED__: Passthrough or IP150 mode, so that an IP150 is emulated and babyware/winload can connect remotelly. Passthrough already tested successfully and will be integrated soon.

## What happens in the background:
The main script will connect to a panel (currently only through Serial) and login with your password (usually 0000). It will then use two seperate classes (containing dictionaries) referenced in from the ParadoxMap.py file to extract different info from the alarm and translate events into meaningfull text. The dictionaries currently supports the MG5050 V4 and V5 firmware but could evolve over time if the community adds more alarm types to the dictionaries.

Once the info (mostly labels) has been extracted from the alarm, the MQTT broker is used as middleware between your application and the alarm panel.

Seeing as not all alarm variants are initially supported, you can use the config.ini file to switch off different portions of the script. Not all event types may be implemented, but most shoud be!

## What to expect:

### Labels
If successfully connected to your panel and MQTT broker, the app will optionally start off by publishing (once-off) all the detected labels (zone, partition, output, etc. names) to your broker. Both the reading of label names and publishing thereof can be independantly controlled through the config.ini file. If you do read the labels, events such as "Zone open - Zone number 3" will translate to "Zone open - Garage door" or "Low battery on zone - Zone 1" to "Low battery on zone - Alley Beam" (assuming this is named/configured as such in your alarm).

Note: on a Spectre SP5500 these labels do not seem to read.  To this end, I've updated to pull the zone name from the event message and will publish an MQTT topic for the zone name.

* Zone Labels:
  * Topic: <b>Paradox/Labels/Zones</b>
  * Payload (example): <b>1:Front PIR;2:Back PIR;3:Garage Door;... </b>
* Output Labels:
  * Topic: <b>Paradox/Labels/Outputs</b>
  * Payload (example): <b>1:Status LED;2:ToggleGarage;3:UnlockFront;4:MakeCoffee;5:Output 05;6:Output 06;...</b>
* Partition Labels:
  * Topic: <b>Paradox/Labels/Partitions</b>
  * Payload (example): <b>1:Main Alarm;2:Front Beams</b>
* User Labels:
  * Topic: <b>Paradox/Labels/Users</b>
  * Payload (example): <b>1:System Master;2:Master 1;3:Master 2;4:Bob;5:Alice;6:User 06;...</b>
* Bus Module Labels:
  * Topic: <b>Paradox/Labels/BusModules</b>
  * Payload (example): <b>1:ZX8-1;2:Bus Module 02;3:Bus Module 03;4:Bus Module 04;5:Bus Module 05;...</b>
* Wireless Repeater Labels:
  * Topic: <b>Paradox/Labels/WirelessRepeaters</b>
  * Payload (example): <b>1:Front Rpt;2:Repeater 2</b>
* Wireless Keypad Labels:
  * Topic: <b>Paradox/Labels/WirelessKeypads</b>
  * Payload (example): <b>1:Main Keyp;2:Garage Keyp;3:Wireless Keyp 3;...</b>
* Wireless Siren Labels:
  * Topic: <b>Paradox/Labels/WirelessSirens</b>
  * Payload (example): <b>1:Wireless Siren 1;2:Wireless Siren 2;3:Wireless Siren 3;4:Wireless Siren 4</b>
* Site Name Labels:
  * Topic: <b>Paradox/Labels/SiteNames</b>
  * Payload (example): <b>1:Your Alarm Site</b>

### Events
Once the script has settled to listen for events, the following topics are available (and their names are also configurable in the config.ini file):
* Events:
  * Topic: <b>Paradox/Events</b>
    * JSON Payload example: ```{"event": "Zone OK", "subevent": "Braaikamer PIR"}```
    * JSON Payload example: ```{"event": "Zone open", "subevent": "Braaikamer PIR"}```
    * JSON Payload example: ```{"event": "Partition status", "subevent": "Disarm partition"}```
    * JSON Payload example: ```{"event": "Zone forced", "subevent": "Back Door"}```
    * JSON Payload example: ```{"event": "Disarming with user", "subevent": "User 06"}```
    * JSON Payload example: ```{"event": "Non-reportable event", "subevent": "Arm in sleep mode"}```
    * etc.....

    It should be noticed the use of JSON for these events!

  * Topic <b>Paradox/Status/Zones/<i>zone label</i></b>
  Sets a status for each zone (Open, Closed).  Allows Openhab and Homeassistant to easily configure an item for each zone

  * Topic <b>Paradox/Status/Partitions</b>
  Shows the current partition status and events.  Can't be determines on startup though, only while running. 

  * Topic <b>Paradox/Status/Bell</b>
  Shows the current Bell status

  * Topic <b>Paradox/Status/System</b>
  Shows the current System status

  * Topic <b>Paradox/Status/Voltage</b>
  Shows the current voltage status
  * Payload example: ```{"battery": 13.6, "vdc": 16.8, "dc": 13.9} ```


### Controls
* Controlling the alarm or outputs
  * Publish the following topic to control the <b>alarm</b>:
    * <b>Paradox/Control/Partitions/number or "ALL"</b>
    * Payload contains the action = Disarm / Arm / Sleep / Stay
    * Is ALL is sent (e.g, Paradox/Control/Partitions/ALL), all partitions are set
  * Publish the following topic to control (Force/Pulse) an <b>output</b> (PGM):
    * <b>Paradox/Control/Output/number</b>
    * Payload contains On or Off
    * Pulse outputs are configured for approx. 0.5sec.

<b>Note: If you modified the subscription topic for <b>controls</b> in the config.ini file ensure it ends with a '/'.</b>

### Script State
* The current state of the script is linked to a topic found in the config file. Examples:
 * Topic <b>Paradox/State</b>
  * Payload (example): State Machine 1, Connected to MQTT Broker
  * Payload (example): State Machine 2, Connecting to IP Module...
  * Payload (example): State Machine 2, Connected to IP Module, unlocking...
  * Payload (example): State Machine 2, Logged into IP Module successfully
  * Payload (example): State Machine 3, Reading labels from alarm
  * Payload (example): State Machine 4, Listening for events...
  * Payload (example): Output: Forcing PGM 1 to state: On


## Running as a service / daemon

If you want to run this as a daemon on Linux, 
 1. Copy the paradoxip.service file to /usr/lib/systemd/system (where mine is)
 2. Run sudo systemctl daemon-reload
 3. Then you should be able to start the service with sudi service paradoxip start

