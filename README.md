# Paradox Multi-MQTT
Python-based 'middle-ware' that aims to use any method to connect to a Paradox Alarm, exposing the interface for monitoring and control via an MQTT Broker.
Currently access through the Serial port is implemented. As this a fork of the [ParadoxIP150v2](https://github.com/Tertiush/ParadoxIP150v2) project, access through the IP150 can be easily (re-)added (volunteers?)

## Origin

This is a fork of the [ParadoxIP150v2](https://github.com/Tertiush/ParadoxIP150v2) project, and changed to implement serial support. The changes were made in such a way that the communication method can be abstracted. I hope that the fork can be merged.

As i do not have an IP150 module, help is welcome (and needed).

## Steps to use it:
1.  Tested with Python 2.7.13 & [Mosquitto MQTT Broker v1.4.14](http://mosquitto.org)
2.  Download the files in this repository and place it in some directory
3.  Edit the config.ini file to match your setup
4.  Run the script: Python Multi-MQTTv2.py


## What to expect:

The behaviour is similar to the one obtained with [ParadoxIP150v2](https://github.com/Tertiush/ParadoxIP150v2). Please check the project documentation.

## Changes from the original project

* Using a logging module, instead of print
* Polling the serial port is now always done. This may help to discover connectivity losses.
* Changed the logic so that it handles Paradox messages without the headers required by the connectivity module (Serial or IP150).
* Abstracted the channel to a separate class.
* A new config file option was added (SERIAL). See the example.