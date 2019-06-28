# MQTT Client Plugin

This plugin provides the ability to publish out any user defined variable to an MQTT Broker.
It is based on the code found here
This is my first plugin so odds are there will be some bugs although so far seems to be working fine.

This plugin is designed for use on systems running UI7.

## Features
- User defined Variables to watch
- User defined Device Alias which makes Logic much more intuitive and makes it easier when replacing devices


## MQTT Message Example

    {"Payload":{"DeviceId":45,"OldOnOff":"1","OnOff":"0","Time":1453209965},"Topic":"Vera/Events/TestSocket"}

## Dependencies

There are a few dependencies in ```Dependencies/usr/lib/lua``` that should be copied to the folder ```/usr/lib/lua```. Upload these using e.g. scp. Reload the LUUP engine afterwards.

## Installation and Configuration

1. Reload LUUP engine.
1. Upload the files in Plugin via the Vera interface under 'Develop Apps'.
1. Create a new device with device_file set to D_SensorMqtt1.xml
1. Set desired variable watches on the Watchdog tab
1. (optional) Set desired Alias on the Alias tab
1. Have fun

## Modifications for subscription

You can subscribe to a topic by creating a child device. This child device will have its value updated upon receiving a message from MQTT.

1. Add a new device and choose a type (e.g. "D_BinaryLight1.xml" or "D_TemperatureSensor1.xml").
1. Reload LUUP engine.
1. Change the attribut "```id_parent```" with the id of the MQTT plugin device.
1. Reload LUUP engine and refresh your browser.
1. You should see variables "```mqttTarget```" and "```mqttTopic```" in your newly created device.
1. Set the topic you want to subcribe to and the target (format: ```service,variable=(formula in LUA)```).
1. Reload LUUP engine.

If the payload of the received message is in JSON, the plugin will try to decode it and put it in the variable "payload" in the context of the LUA formula.

### Examples

```
mqttTopic: Test/+/Sensor
mqttTarget: urn:upnp-org:serviceId:TemperatureSensor1,CurrentTemperature=payload.temperature and (tonumber(payload.temperature) or "0")
```

On a message from topic ```Test/Something/Sensor``` with payload ```{"temperature ":"15.2", "hygrometry":"80"}```, the temperature will be set to 15,2.

## Change a device on publish

Besides receiving a value by subscribing to an MQTT topic, it is also possible to publish a value to a topic. This is currently only support for the ```urn:upnp-org:serviceId:SwitchPower1``` action ```SetTarget```.

1. Add a new device and choose the type: D_BinaryLight1.xml
2. Follow the same steps as for adding a device subscription

When you change the on/off state of the device, the topic will be updated accordingly. To publish the state to a different topic, add a second topic to the "```mqttTopic```" variable and separate it from the first topic by a comma.

### Examples

```
mqttTopic: Test/#,Test/command
mqttTarget: urn:upnp-org:serviceId:SwitchPower1,Status=payload.value and ((payload.value=="alarm") and "1" or "0")
```

On a message from topic ```Test/Something``` with payload ```{"value":"alarm"}```, the switch will be powered on. When the switch is power off, the value 0 is send to ```Test/command```.
