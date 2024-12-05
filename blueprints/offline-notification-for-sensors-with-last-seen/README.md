# offline-notification-for-sensors-with-last-seen
Home Assistant Blueprint: Offline detection for Z2M devices with last_seen


## Description

This blueprint checks at configured time for all devices that was last seen x hours (sensors in ‘unavailable’ state are also included) ago and fires action then (like notification). You can include list of devices in notification message. You can also exclude some devices from checking (use their ‘last seen’ sensor entity as exluded entity)

Example of notification bellow:
![notification](/media/offline-notification-for-sensors-with-last-seen/notif.jpg)

```yaml
actions:
  - device_id: a2a0b972c39a402aa8547899f9d15ee7
    domain: mobile_app
    type: notify
    title: Urządzenie Zigbee niedostępne!
    message: '{{sensors}}'
```

## Requirements

This blueprint is intended to work with Zigbee2MQTT devices that has ‘last seen’ sensors with ‘timestamp’ device_class (it is used to filter out sensors to check). It may work with other Zigbee integrations if there are also sensors with ‘last seen’ in name and ‘timestamp’ device_class.

You will need to enable ‘last_seen’ option in zigbee2mqtt configuration:

```yaml
advanced:
  last_seen: ISO_8601_local
```

Remember also to enable ‘last seen’ sensors for all your Zigbee2MQTT devices (it is now disabled by default) in Home Assistant device page! You can also enable all ‘last seen’ sensors by default adding this to your Z2M configuration:

```yaml
device_options:
  homeassistant:
    last_seen:
      enabled_by_default: true
```

## Blueprint

```yaml
blueprint:
  name: Offline detection for Z2M devices with last_seen
  description: Regularly test all sensors with 'last_seen' in name and 'timestamp' device_class
    ('last seen' Z2M sensors) to detect offline and if so execute an action.
  domain: automation
  input:
    hours:
      name: Hours not seen
      description: Sensors not seen this amount of time are assumed to be offline.
      default: 24
      selector:
        number:
          min: 1.0
          max: 168.0
          unit_of_measurement: h
          mode: slider
          step: 1.0
    time:
      name: Time to test on
      description: Test is run at configured time
      default: '10:00:00'
      selector:
        time: {}
    day:
      name: Weekday to test on
      description: 'Test is run at configured time either everyday (0) or on a given
        weekday (1: Monday ... 7: Sunday)'
      default: 0
      selector:
        number:
          min: 0.0
          max: 7.0
          mode: slider
          step: 1.0
    exclude:
      name: Excluded Sensors
      description: "'last seen' sensors (from devices that you want to exclude) to
        exclude from detection. Only entities with 'last seen' in name and 'timestamp'
        in device_class are supported, devices must be expanded!"
      default:
        entity_id: []
      selector:
        target:
          entity:
            domain: sensor
    actions:
      name: Actions
      description: Notifications or similar to be run. {{sensors}} is replaced with
        the names of sensors being offline.
      selector:
        action: {}
  source_url: https://gist.github.com/Mr-Groch/bf073b142b507e3b6f8154223f81803b
variables:
  day: !input 'day'
  hours: !input 'hours'
  exclude: !input 'exclude'
  sensors: "{% set result = namespace(sensors=[]) %}\
    \ {% for state in states.sensor | rejectattr('attributes.device_class', 'undefined') | selectattr('attributes.device_class', '==', 'timestamp') %}\
    \   {% if 'last_seen' in state.entity_id and not state.entity_id in exclude.entity_id and (states(state.entity_id) == 'unavailable' or ((as_timestamp(now()) - as_timestamp(states(state.entity_id))) > ((hours | int) * 60 * 60))) %}\
    \     {% set result.sensors = result.sensors + [state.name | regex_replace(find=' last seen', replace='') ~ ' (' ~ relative_time(strptime(states(state.entity_id), '%Y-%m-%dT%H:%M:%S%z', 'unavailable')) ~ ')'] %}\
    \   {% endif %}\
    \ {% endfor %}\
    \ {{ result.sensors | join(', ') }}"
trigger:
- platform: time
  at: !input 'time'
condition:
- '{{ sensors != '''' and (day | int == 0 or day | int == now().isoweekday()) }}'
action:
- choose: []
  default: !input 'actions'
mode: single
```
