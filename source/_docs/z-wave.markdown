---
layout: page
title: "Z-Wave"
description: "Instructions how to integrate your existing Z-Wave within Home Assistant."
date: 2016-02-27 19:59
sidebar: true
comments: false
sharing: true
footer: true
redirect_from: /getting-started/z-wave/
---

[Z-Wave](http://www.z-wave.com/) integration for Home Assistant allows you to observe and control connected Z-Wave devices. Z-Wave support requires a [supported Z-Wave USB stick or module](https://github.com/OpenZWave/open-zwave/wiki/Controller-Compatibility-List) to be plugged into the host.

There is currently support for climate, covers, lights, locks, sensors, switches, and thermostats. All will be picked up automatically after configuring this platform.

Before configuring the Z-Wave setup, please take a moment and read [this article](https://drzwave.blog/2017/01/20/seven-habits-of-highly-effective-z-wave-networks-for-consumers/) to understand the most common pitfalls of Z-Wave networks. 

### {% linkable_title Installation %}

As of version 0.45, Home Assistant automatically installs python-openzwave from PyPI as needed.

There is one dependency you will need to have installed ahead of time (included in `systemd-devel` on Fedora/RHEL systems):

```bash
$ sudo apt-get install libudev-dev
```

### {% linkable_title Configuration %}

```yaml
# Example configuration.yaml entry
zwave:
  usb_path: /dev/ttyUSB0
```

Configuration variables:

- **usb_path** (*Optional*): The port where your device is connected to your Home Assistant host.
- **network_key** (*Optional*): The 16-byte network key in the form `"0x01,0x02..."` used in order to connect securely to compatible devices.
- **config_path** (*Optional*): The path to the Python OpenZWave configuration files. Defaults to the 'config' that is installed by python-openzwave
- **autoheal** (*Optional*): Allows disabling auto Z-Wave heal at midnight. Defaults to True.
- **polling_interval** (*Optional*): The time period in milliseconds between polls of a nodes value. Be careful about using polling values below 30000 (30 seconds) as polling can flood the zwave network and cause problems.
- **device_config** (*Optional*): This attribute contains node-specific override values. (For releases prior to 0.39 this variable is called **customize**) See [Customizing devices and services](https://home-assistant.io/getting-started/customizing-devices/) for format:
  - **polling_intensity** (*Optional*): Enables polling of a value and sets the frequency of polling (0=none, 1=every time through the list, 2=every other time, etc). If not specified then your device will not be polled.
  - **ignored** (*Optional*): Ignore this entity completely. It won't be shown in the Web Interface and no events are generated for it.
  - **refresh_value** (*Optional*): Enable refreshing of the node value. Only the light component uses this. Defaults to False.
  - **delay** (*Optional*): Specify the delay for refreshing of node value. Only the light component uses this. Defaults to 2 seconds.
  - **invert_openclose_buttons** (*Optional*): Inverts function of the open and close buttons for the cover domain. Defaults to `False`.
- **debug** (*Optional*): Print verbose z-wave info to log. Defaults to `False`.
- **new_entity_ids** (*Optional*): Switch to new entity_id generation. Defaults to `True`.

To find the path of your Z-Wave USB stick or module, run:

```bash
$ ls /dev/ttyUSB*
```

Or, if there is no result, try to find detailed USB connection info with:
```bash
$ dmesg | grep USB
```

Or, on some other systems (such as Raspberry Pi), use:

```bash
$ ls /dev/ttyACM*

# If Home Assistant (`hass`) runs with another user (e.g. *homeassistant* on Hassbian) give access to the stick with:
$ sudo usermod -a -G dialout homeassistant
```

Or, on some other systems (such as Pine 64), use:

```bash
$ ls /dev/ttyS*
```

Or, on macOS, use:

```bash
$ ls /dev/cu.usbmodem*
```

<p class='note'>
Depending on what's plugged into your USB ports, the name found above may change. You can lock in a name, such as `/dev/zwave`, by following [these instructions](http://hintshop.ludvig.co.nz/show/persistent-names-usb-serial-devices/).
</p>

<p class='note'>
After installing and configuring Z-Wave, Home Assistant will try to install required Python Z-Wave libraries during startup (if they are not already installed). This process might take anywhere between 5 to 25 minutes depending upon various factors. Please be patient, and let it run! This happens only once and the subsequent restarts will have no impact in performance.
</p>

### {% linkable_title Adding Devices %}

To add a Z-Wave device to your system, go to the Z-Wave panel in the Home Assistant frontend and click the Add Node button in the Z-Wave Network Management card. This will place the controller in inclusion mode, after which you should activate your device to be included by following the instructions provided with the device.

<p class='note'>
Some Z-Wave controllers, like Aeotec ZW090 Z-Stick Gen5, have ability to add devices to the network using their own contol buttons. This method should be avoided as it is prone to errors. Devices added to the Z-Wave network using this method may not function well.
</p>

### {% linkable_title Adding Security Devices %}

Security Z-Wave devices require a network key before being added to the network using the Add Secure Node button in the Z-Wave Network Management card. You must set the *network_key* configuration variable to use a network key before adding these devices.

An easy script to generate a random key:
```bash
cat /dev/urandom | tr -dc '0-9A-F' | fold -w 32 | head -n 1 | sed -e 's/\(..\)/0x\1, /g'
```

### {% linkable_title Battery Powered Devices %}

Battery powered devices need to be awake before you can use the Z-Wave control panel to update their settings. How to wake your device is device specific, and some devices will stay awake for only a couple of seconds. Please refer to the manual of your device for more details.

### {% linkable_title Events %}

#### {% linkable_title zwave.network_complete %}

Home Assistant will trigger an event when the Z-Wave network is complete, meaning all of the nodes on the network have been queried. This can take quite some time, depending on wakeup intervals on the battery-powered devices on the network.

```yaml
 - alias: Z-Wave network is complete
   trigger:
     platform: event
     event_type: zwave.network_complete
```

#### {% linkable_title zwave.network_ready %}

Home Assistant will trigger an event when the Z-Wave network is ready for use. Between `zwave.network_start` and `zwave.network_ready` Home Assistant will feel sluggish when trying to send commands to Z-Wave nodes. This is because the controller is requesting information from all of the nodes on the network. When this is triggered, all awake nodes have been queried and sleeping nodes will be queried when they awake.

```yaml
 - alias: Z-Wave network is ready
   trigger:
     platform: event
     event_type: zwave.network_ready
```

#### {% linkable_title zwave.network_start %}

Home Assistant will trigger an event when the Z-Wave network is set up to be started.

```yaml
 - alias: Z-Wave network is starting
   trigger:
     platform: event
     event_type: zwave.network_start
```

#### {% linkable_title zwave.network_stop %}

Home Assistant will trigger an event when the Z-Wave network is stopping.

```yaml
 - alias: Z-Wave network is stopping
   trigger:
     platform: event
     event_type: zwave.network_stop
```

#### {% linkable_title zwave.node_event %}
Home Assistant will trigger an event when command_class_basic changes value on a node. This can be virtually anything, so tests have to be made to determine what value equals what. You can use this for automations.

Example:

```yaml
 - alias: Minimote Button Pressed
   trigger:
     platform: event
     event_type: zwave.node_event
     event_data:
       entity_id: zwave.aeon_labs_minimote_1
       basic_level: 255
```

The *object_id* and *basic_level* of all triggered events can be seen in the console output.

#### {% linkable_title zwave.scene_activated %}

Some devices can also trigger scene activation events, which can be used in automation scripts (for example, the press of a button on a wall switch):

```yaml
# Example configuration.yaml automation entry
automation:
  - alias: Turn on Desk light
    trigger:
      platform: event
      event_type: zwave.scene_activated
      event_data:
        entity_id: zwave.zwaveme_zme_wallcs_secure_wall_controller_8
        scene_id: 11
```

Some devices (like the HomeSeer wall switches) allow you to do things like double, and triple click the up and down buttons and fire an event.  These devices will also send `scene_data` to differentiate the events.  This is an example of double clicking the on/up button:

```yaml
# Example configuration.yaml automation entry
automation
  - alias: 'Dining room dimmer - double tap up'
    trigger:
      - event_type: zwave.scene_activated
        platform: event
        event_data:
          entity_id: zwave.dining_room_cans
          scene_id: 1
          scene_data: 3
```

The *object_id* and *scene_id* of all triggered events can be seen in the console output.

For more information on HomeSeer devices and similar devices, please see the [device specific page](https://home-assistant.io/docs/z-wave/device-specific/#homeseer-switches).

### {% linkable_title Services %}

The `zwave` component exposes multiple services to help maintain the network.

| Service | Description |
| ------- | ----------- |
| add_node | Put the Z-Wave controller in inclusion mode. Allows one to add a new device to the Z-Wave network.|
| add_node_secure | Put the Z-Wave controller in secure inclusion mode. Allows one to add a new device with secure communications to the Z-Wave network. |
| cancel_command | Cancels a running Z-Wave command. If you have started a add_node or remove_node command, and decide you are not going to do it, then this must be used to stop the inclusion/exclusion command. |
| change_association | Add or remove an association in the Z-Wave network |
| heal_network | Tells the controller to "heal" the Z-Wave network. Basically asks the nodes to tell the controller all of their neighbors so the controller can refigure out optimal routing. |
| print_config_parameter | Prints Z-Wave node's config parameter value to the log. |
| print_node | Print all state of Z-Wave node. |
| refresh_entity| Refresh Z-Wave entity by refreshing dependent values.|
| refresh_node| Refresh Z-Wave node. |
| remove_node | Put the Z-Wave controller in exclusion mode. Allows one to remove a device from the Z-Wave network.|
| rename_node | Sets a node's name. Requires a `node_id` and `name` field. |
| rename_value | Sets a value's name. Requires a `node_id`, `value_id`, and `name` field. |
| remove_failed_node | Remove a failed node from the network. The Node should be on the controller's Failed Node List, otherwise this command will fail.|
| replace_failed_node | Replace a failed device with another. If the node is not in the controller's Failed Node List, or the node responds, this command will fail.|
| reset_node_meters | Reset a node's meter values. Only works if the node supports this. |
| set_config_parameter | Lets the user set a config parameter to a node. NOTE: Use the parameter option's `label` string as the `value` for list parameters (e.g. `"value": "Off"`). For all other parameters use the relevant integer `value` (e.g. `"value": 1`). |
| set_poll_intensity | Lets the user set the polling intensity of a value. Changes the polling intensity without the need of a restart. This does not persist over restarts. To keep the setting over restarts, use the Z-Wave entity-card to set the config also.
| soft_reset | Tells the controller to do a "soft reset." This is not supposed to lose any data, but different controllers can behave differently to a "soft reset" command.|
| start_network | Starts the Z-Wave network.|
| stop_network | Stops the Z-Wave network.|
| test_network | Tells the controller to send no-op commands to each node and measure the time for a response. In theory, this can also bring back nodes which have been marked "presumed dead."|

The `soft_reset` and `heal_network` commands can be used as part of an automation script to help keep a Z-Wave network running reliably as shown in the example below. By default, Home Assistant will run a `heal_network` at midnight. This is a configuration option for the `zwave` component. The option defaults to `true` but can be disabled by setting `autoheal` to false. Using the `soft_reset` function with some Z-Wave controllers can cause the Z-Wave network to hang. If you're having issues with your Z-Wave network, try disabling this automation.

```yaml
# Example configuration.yaml automation entry
automation:
  - alias: soft reset at 2:30am
    trigger:
      platform: time
      at: '2:30:00'
    action:
      service: zwave.soft_reset

  - alias: heal at 2:31am
    trigger:
      platform: time
      at: '2:31:00'
    action:
      service: zwave.heal_network
```
