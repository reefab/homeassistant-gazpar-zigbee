# Homeassistant Gazpar Zigbee

## How to get (almost) real time gas usage metric from a Gazpar meter using a zigbee button and Home Assistant

![meter](meter.jpg)

Based on [^1].

### Disclaimer

As with any DIY project there is a chance of damage or injury and we are dealing with gas. Follow this guide at your own risk.

### What you need

 * Home assistant
 * A Gazpar meter or compatible
 * a Zigbee button, I recommend an Ikea one because they have a magnet on the back and the meter has a steel casing
 * a JAE connector. Either [buy one](https://www.google.com/search?q=gazpar+JAE+cable) or print one, instruction below

### How it works

The Gazpar has a [dry contact](https://en.wikipedia.org/wiki/Dry_contact) that closes for 300ms every 10L of gaz.

We can use this dry contact to close a zigbee switch that triggers a Home Assistant automations which in turn increment a counter.

### Hardware

![assembly](assembly.jpg)

If you do not have a pre-made cable, you can use the `Gazpar-MX44-Connector.3mf` file for prusaslicer with pre-configured supports, originally from[^2]. Just add Dupont female connectors and some hot glue to secure them.

Just solder the cable across a switch of your Zibgee button. Plug the connector into the side of your meter.

### Software

I'm using Zigbee2Mqtt but you can use other systems, you'll need to adapt the trigger.


#### Input counter

This is the variable that stores the amount of Liters that have been used.

```yaml
input_number:
  gas_counter:
    min: 0
    max: 100000000000
    name: gas_counter
    icon: "mdi:gas-station"
    mode: box
```

#### Automation

This increment the counter above by 10 every time the zigbee button is "pressed" (read: the dry contact is closed).

```yaml
automation:
  - alias: ⛽️ Increment gas counter
    trigger:
      - platform: mqtt
        topic: zigbee2mqtt/<name of the button>/action
        payload: '<state when it's pressed>'
    action:
      - service: input_number.set_value
        data:
          value: "{{ states('input_number.gas_counter') | int + 10 | int }}"
        entity_id: input_number.gas_counter
    mode: single
```

#### Volume sensor

This will create a sensor in cubic meters as Home assistant will require it for the gas utility meter and the dashboard.

```yaml
template:
  - sensor:
    - name: "Gas Volume"
      unit_of_measurement: 'm³'
      state: "{{ states('input_number.gas_counter') | int / 1000 | round(3) }}"
      device_class: gas
      state_class: total
```

You can now add it to your energy dashboard!

#### Utility meter

Used for stats:

```yaml
utility_meter:
  gas_total_usage_daily:
    source: sensor.gas_volume
    cycle: daily
  gas_total_usage_weekly:
    source: sensor.gas_volume
    cycle: weekly
  gas_total_usage_monthly:
    source: sensor.gas_volume
    cycle: monthly
  gas_total_usage_quarterly:
    source: sensor.gas_volume
    cycle: quarterly
  gas_total_usage_yearly:
    source: sensor.gas_volume
    cycle: yearly
```

## Sources

[^1]: [Home Assistant: récupération Gazpar](https://www.auto-domo.fr/home-assistant-recuperation-gazpar/)
[^2]: [Gazpar connector](https://www.thingiverse.com/thing:4922264)
