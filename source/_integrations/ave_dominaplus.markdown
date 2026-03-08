---
title: AVE DominaPlus
description: Instructions on how to integrate your AVE DominaPlus home automation system with Home Assistant.
ha_category:
  - Button
  - Climate
  - Cover
  - Light
  - Sensor
ha_config_flow: true
ha_release: 2026.4
ha_iot_class: Local Push
ha_codeowners:
  - '@Klaas-'
ha_domain: ave_dominaplus
ha_platforms:
  - button
  - climate
  - cover
  - diagnostics
  - light
  - sensor
ha_integration_type: hub
works_with:
  - local
ha_quality_scale: platinum
---

The **AVE DominaPlus** {% term integration %} connects Home Assistant directly to an [AVE DominaPlus](https://www.ave.it/) home automation server over a local WebSocket connection. AVE DominaPlus is a wired home automation system that controls lights, dimmers, shutters, thermostats, and scenarios from a central server. Once connected, all your AVE devices appear in Home Assistant and any state change from a physical wall switch is reflected instantly — no polling, no delay.

You can use this integration to control your AVE lights, dimmers, and shutters from Home Assistant dashboards and automations, monitor and schedule thermostat setpoints centrally, trigger pre-programmed AVE scenarios from automations or voice assistants, and react to physical switch events in real-time.

## Supported devices

The integration supports all device types managed by an AVE DominaPlus server:

- Lights — single-channel on/off relay outputs
- Dimmers — brightness-controllable light outputs
- Shutters — motorized roller shutters and blinds
- Thermostats — zone thermostats with temperature measurement and setpoint control
- Scenarios — pre-configured automation scenes stored on the server

## Prerequisites

1. Your AVE DominaPlus server must be connected to the same local network as Home Assistant.
2. Note the IP address of the AVE DominaPlus server. Assign it a static IP address or a <abbr title="Dynamic Host Configuration Protocol">DHCP</abbr> reservation on your router so that it does not change over time.
3. The server must be reachable on its WebSocket port (default: `14001`).

{% include integrations/config_flow.md %}

{% configuration_basic %}
Host:
  description: The IP address of your AVE DominaPlus server.
Port:
  description: The WebSocket port of your AVE DominaPlus server. The default is `14001` and does not normally need to be changed.
{% endconfiguration_basic %}

## Supported functionality

### Lights

Each light output on the AVE DominaPlus system appears as a Home Assistant light {% term entity %}. There are two types:

- On/off lights — simple relay outputs that you can turn on and off.
- Dimmers — support brightness control from 0% to 100%, mapped to the 31-step range used by the AVE protocol.

### Covers

Each shutter output appears as a cover {% term entity %} with device class shutter. You can open, close, and stop each shutter, and also set a target position.

{% note %}
The AVE DominaPlus protocol does not report exact shutter positions. Home Assistant synthesizes three discrete position values: 0 (closed), 50 (stopped or partially open), and 100 (fully open). Setting a position between 11 and 89 stops the shutter at its current position. Positions of 10 or lower close the shutter, and positions of 90 or higher open it fully.
{% endnote %}

### Climate

Each thermostat appears as a climate {% term entity %}. You can:

- Switch between heat (winter season), cool (summer season), and off modes.
- Switch between auto (follows the programmed schedule) and manual (holds a fixed setpoint) preset modes.
- Adjust the temperature setpoint in 0.5 °C steps, between 5 °C and 40 °C.
- Monitor the current temperature as measured by the thermostat's built-in sensor.

### Buttons

Each scenario stored on the AVE DominaPlus server appears as a button {% term entity %}. Pressing the button activates the scenario on the server, which can trigger any sequence of actions programmed in the AVE system, such as turning off all lights, lowering all shutters, or setting thermostats to night mode.

### Sensors

Thermostats that are equipped with a humidity probe expose a humidity sensor {% term entity %}.

{% note %}
The humidity sensor is disabled by default because not all thermostats include a humidity probe. To enable it, go to {% my integrations title="**Settings** > **Devices & services**" %}, select the **AVE DominaPlus** integration, find the thermostat device, and enable the humidity sensor from its entity list.
{% endnote %}

## Examples

### Turn off all lights when you leave home

```yaml
automation:
  - alias: "Turn off AVE lights when leaving home"
    triggers:
      - trigger: state
        entity_id: person.your_name
        to: "not_home"
    actions:
      - action: light.turn_off
        target:
          entity_id: all
```

### Activate a goodnight scenario at bedtime

```yaml
automation:
  - alias: "Goodnight routine"
    triggers:
      - trigger: time
        at: "22:30:00"
    actions:
      - action: button.press
        target:
          entity_id: button.goodnight_activate
```

### Set the thermostat to an away temperature when no one is home

```yaml
automation:
  - alias: "Set thermostat to away temperature"
    triggers:
      - trigger: state
        entity_id: group.family
        to: "not_home"
    actions:
      - action: climate.set_temperature
        target:
          entity_id: climate.living_room_thermostat
        data:
          temperature: 15
```

## Data updates

This integration uses {% term "local push" %}. The AVE DominaPlus server sends state updates to Home Assistant over a persistent WebSocket connection the moment a device changes state. Pressing a physical wall switch appears in Home Assistant immediately, with no polling delay.

If the connection is lost, the integration marks all {% term entities %} as unavailable and reconnects to the server automatically. Once reconnected, all entity states are refreshed.

## Known limitations

- Exact intermediate shutter positions are not available. The AVE DominaPlus protocol only reports fully open, fully closed, and stopped states.
- AVE DominaPlus energy meter devices are not yet exposed as entities.
- Fan speed and fan-coil configuration are available in the AVE system but are not yet exposed by this integration.
- The humidity sensor is disabled by default and must be manually enabled per thermostat device.

## Troubleshooting

### The integration cannot connect to the server

#### Symptom: "Failed to connect" error during setup

When you try to add the integration, the setup form shows a "Failed to connect" error.

##### Description

Home Assistant cannot reach the AVE DominaPlus server on the IP address and port you entered.

##### Resolution

1. Confirm the AVE DominaPlus server is powered on.
2. Check that the IP address and port (`14001` by default) you entered are correct.
3. Make sure both the server and the machine running Home Assistant are on the same local network.
4. Check whether any firewall rule on your router or network is blocking port `14001`.

### Entities show as unavailable

#### Symptom: All AVE entities show as unavailable

After the integration was working, all entities suddenly become unavailable.

##### Description

The WebSocket connection to the AVE DominaPlus server was lost.

##### Resolution

1. Check that the AVE DominaPlus server is powered on and reachable on your network.
2. The integration reconnects automatically — wait a minute and check if the entities recover.
3. If the entities do not recover, reload the integration from {% my integrations title="**Settings** > **Devices & services**" %}, select **AVE DominaPlus**, and then select **Reload**.
4. Check the Home Assistant logs for error messages from the `pyavedominaplus` logger for more detail.

### A device is missing from Home Assistant

#### Symptom: A device that exists on the server does not appear in Home Assistant

##### Description

The integration discovers devices when it starts up. Devices added to the AVE DominaPlus server after the integration loaded are not picked up automatically.

##### Resolution

Reload the integration from {% my integrations title="**Settings** > **Devices & services**" %}, select **AVE DominaPlus**, and then select **Reload**. All devices currently registered on the server will be loaded, and any devices that were removed from the server will be cleaned up automatically.

## Removing the integration

This integration follows standard integration removal, no extra steps are required.

{% include integrations/remove_device_service.md %}
