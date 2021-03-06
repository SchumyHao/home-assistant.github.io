---
layout: page
title: "QwikSwitch QSUSB Hub"
description: "Instructions on how to integrate the QwikSwitch QSUSB Hub into Home Assistant."
date: 2016-05-04 00:00
sidebar: true
comments: false
sharing: true
footer: true
logo: qwikswitch.png
ha_category: Hub
featured: false
ha_release: "0.20"
---

The `qwikswitch` component is the main component to integrate various [QwikSwitch](http://www.qwikswitch.co.za/) devices with Home Assistant. The integration requires the QSUSB Modem device and connects to the QS Mobile application.

The `qwikswitch` component discovers all devices from QS Mobile. Currently, Relays and LED dimmers are discovered in Home Assistant. Relay devices are [lights](/components/light.qwikswitch/) by default, and can be configured as [switches](/components/switch.qwikswitch/).

Example configuration:

```yaml
# Example configuration.yaml entry
qwikswitch:
   url: http://127.0.0.1:2020
```

Configuration variables:

- **url** (*Required*): The URL including the port of your QwikSwitch hub.
- **dimmer_adjust** (*Optional*): A decimal value to adjust the brightness of the dimmer exponentially. Increasing this value allows dimmers that reach full brightness with low values in QS Mobile to appear more linear in Home Assistant. Recommended values between 1 and 2 and the default is 1.
- **button_events** (*Optional*): A comma-separated list of button types that will generate events. See [QwikSwitch Events] for detail.
- **switches** (*Optional*): A list of device QS_id's that should be switches, and not lights (i.e. `['@0dev01', '@0dev02']`)
- **sensors** (*Optional*): A dictionary of sensors. In the format of {entity_id: QS_id}. (i.e. `{door_sensor: '@0dev03'}`)

### {% linkable_title QwikSwitch Events %}

QwikSwitch devices (i.e., transmitter buttons) will fire events on the Home Assistant bus. These events can then be used as triggers for any `automation` action, as follows:

```yaml
automation:
  - alias: Action - Respond to A button press
    trigger:
      platform: event
      event_type: qwikswitch.button.@12df34
```

`event_type` names should be in the format **qwikswitch.button.@_QS_id_**. where **@_QS_id_** will be captured in the Home Assistant log when pressing the button. Alternatively, you can also get the device ID from the QS Mobile application or by using the listen API call by browsing to `http://127.0.0.1:2020/&listen` and then pressing the button.

The full packet from the QSUSB API will be passed as `data`

By default events will be fired if the value in the command (cmd) field of the listen packet equals:
- `TOGGLE` - Normal QwikSwitch Transmitter button
- `SCENE EXE` - QwikSwitch Scene Transmitter buttons
- `LEVEL` - QwikSwitch OFF Transmitter buttons

The list of recognized commands can be extended for Keyfobs, door sensors, and PIR transmitters with the **button_events** configuration option. **button_events** can be a list or comma separated list of additional commands that will fire Home Assistant events. By default, it is: TOGGLE,SCENE EXE,LEVEL.

On some QS Mobile servers button events are only generated for switches added to the QS Mobile application, so it might be best to test button presses through the `/&listen` API

### {% linkable_title Qwikswitch Sensors %}

The sensor configuration is a list of sensors. Depending on the type, it will be a sensor or binary_sensor. Each sensor is configured with the following parameters:

- **name** (*Required*): The sensor name from which the entity_id will be derived.
- **id** (*Required*): A QS_Id
- **type** (*Required*): The Qwikswitch sensor type. These could include:
   - imod (binary_sensor, up to 6 channels)
   - door (binary_sensor, single channel)
   - qwikcord (Channel 1 = CTavg, Channel 2 = CTsum)
- **channel** (*Optional, default=1*): The channel of interest. Refer to type above.

The following parameters are only applicabe to binary_sensors (i.e. door/imod).
- **invert** (*Optional, default=False): Invert the open/close state. 
- **class** (*Optional, default=door): The [class](https://www.home-assistant.io/components/binary_sensor) or binary_sensor.

Example sensor configuration:

```yaml
qwikswitch:
  ...
  sensors:
    - name: door sensor
      id: "@id03"
      type: door
    - name: Imod 1 sensor
      id: "@id02"
      channel: 1
      type: imod
    - name: Imod 2 sensor
      id: "@id02"
      channel: 2
      type: imod
```
