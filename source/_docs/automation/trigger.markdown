---
layout: page
title: "Automation Trigger"
description: "All the different ways how automations can be triggered."
date: 2016-04-24 08:30 +0100
sidebar: true
comments: false
sharing: true
footer: true
redirect_from: /getting-started/automation-trigger/
---

Triggers are what starts the processing of an automation rule. It is possible to specify [multiple triggers](/docs/automation/trigger/#multiple-triggers) for the same rule - when _any_ of the triggers becomes true then the automation will start. Once a trigger starts, Home Assistant will validate the conditions, if any, and call the action.

### {% linkable_title Event trigger %}

Triggers when an event is being processed. Events are the raw building blocks of Home Assistant. You can match events on just the event name or also require specific event data to be present.

Events can be fired by components or via the API. There is no limitation to the types. A list of built-in events can be found [here](/docs/configuration/events/).

```yaml
automation:
  trigger:
    platform: event
    event_type: MY_CUSTOM_EVENT
    # optional
    event_data:
      mood: happy
```

<p class='note warning'>
  Starting 0.42, it is no longer possible to listen for event `homeassistant_start`. Use the 'homeassistant' platform below instead.
</p>

### {% linkable_title Home Assistant trigger %}

Triggers when Home Assistant starts up or shuts down.

```yaml
automation:
  trigger:
    platform: homeassistant
    # Event can also be 'shutdown'
    event: start
```

### {% linkable_title MQTT trigger %}

Triggers when a specific message is received on given topic. Optionally can match on the payload being sent over the topic. The default payload encoding is 'utf-8'. For images and other byte payloads use `encoding: ''` to disable payload decoding completely.

```yaml
automation:
  trigger:
    platform: mqtt
    topic: living_room/switch/ac
    # Optional
    payload: 'on'
    encoding: 'utf-8'
```

### {% linkable_title Numeric state trigger %}

Triggers when numeric value of an entity's state crosses a given threshold. On state change of a specified entity, attempts to parse the state as a number and triggers once if value is changing from above to below or from below to above the given threshold.

{% raw %}
```yaml
automation:
  trigger:
    platform: numeric_state
    entity_id: sensor.temperature
    # Optional
    value_template: '{{ state.attributes.battery }}'
    # At least one of the following required
    above: 17
    below: 25

    # If given, will trigger when condition has been for X time, can also use days and milliseconds.
    for:
      hours: 1
      minutes: 10
      seconds: 5
```
{% endraw %}

<p class='note'>
Listing above and below together means the numeric_state has to be between the two values.
In the example above, a numeric_state that goes to 17.1-24.9 (from 17 or below, or 25 or above) would fire this trigger.
</p>

The `for:` can also be specified as `HH:MM:SS` like this:

{% raw %}
```yaml
automation:
  trigger:
    platform: numeric_state
    entity_id: sensor.temperature
    # Optional
    value_template: '{{ state.attributes.battery }}'
    # At least one of the following required
    above: 17
    below: 25

    # If given, will trigger when condition has been for X time.
    for: '01:10:05'
```
{% endraw %}

### {% linkable_title State trigger %}

Triggers when the state of a given entity changes. If only `entity_id` is given trigger will activate for all state changes, even if only state attributes change.

```yaml
automation:
  trigger:
    platform: state
    entity_id: device_tracker.paulus, device_tracker.anne_therese
    # Optional
    from: 'not_home'
    # Optional
    to: 'home'

    # If given, will trigger when state has been the to state for X time.
    for: '01:10:05'
```

<p class='note warning'>
  Use quotes around your values for `from` and `to` to avoid the YAML parser interpreting values as booleans.
</p>

### {% linkable_title Sun trigger %}

Triggers when the sun is setting or rising. An optional time offset can be given to have it trigger a set time before or after the sun event (i.e. 45 minutes before sunset, when dusk is setting in).

Sunrise as a trigger may need special attention as explained in time triggers below. This is due to the date changing at midnight and sunrise is at an earlier time on the following day.

```yaml
automation:
  trigger:
    platform: sun
    # Possible values: sunset, sunrise
    event: sunset
    # Optional time offset. This example is 45 minutes.
    offset: '-00:45:00'
```

Sometimes you may want more granular control over an automation based on the elevation of the sun. This can be used to layer automations to occur as the sun lowers on the horizon or even after it is below the horizon. This is also useful when the "sunset" event is not dark enough outside and you would like the automation to run later at a precise solar angle instead of the time offset such as turning on exterior lighting. For most things, a general number like -4 degrees is suitable and is used in this example:

{% raw %}
```yaml
automation:
  alias: "Exterior Lighting on when dark outside"
  trigger:
    platform: numeric_state
    entity_id: sun.sun
    value_template: "{{ state.attributes.elevation }}"
    # Can be a positive or negative number
    below: -4.0
  action:
    service: switch.turn_on
    entity_id: switch.exterior_lighting
```
{% endraw %}

If you want to get more precise, start with the US Naval Observatory [tool](http://aa.usno.navy.mil/data/docs/AltAz.php) that will help you estimate what the solar angle will be at any specific time. Then from this, you can select from the defined twilight numbers. Although the actual amount of light depends on weather, topography and land cover, they are defined as:

- Civil twilight: Solar angle > -6°
- Nautical twilight: Solar angle > -12°
- Astronomical twilight: Solar angle > -18°
    
A very thorough explanation of this is available in the Wikipedia article about the [Twilight](https://en.wikipedia.org/wiki/Twilight).

### {% linkable_title Template trigger %}

Template triggers work by evaluating a [template](/docs/configuration/templating/) on every state change for all of the recognized entities. The trigger will fire if the state change caused the template to render 'true'. This is achieved by having the template result in a true boolean expression (`{% raw %}{{ is_state('device_tracker.paulus', 'home') }}{% endraw %}`) or by having the template render 'true' (example below). Being a boolean expression the template must evaluate to false before it will fire again.
With template triggers you can also evaluate attribute changes by using is_state_attr (`{% raw %}{{ is_state_attr('climate.living_room', 'away_mode', 'off') }}{% endraw %}`)

{% raw %}
```yaml
automation:
  trigger:
    platform: template
    value_template: "{% if is_state('device_tracker.paulus', 'home') %}true{% endif %}"
```
{% endraw %}

<p class='note warning'>
Rendering templates with time (`now()`) is dangerous as trigger templates only update based on entity state changes.
</p>

### {% linkable_title Time trigger %}

The time trigger is configured to run once at a specific point in time each day.

```yaml
automation:
  trigger:
    platform: time
    # Military time format. This trigger will fire at 3:32 PM
    at: '15:32:00'
```

### {% linkable_title Time pattern trigger %}

With the time pattern trigger, you can match if the hour, minute or second of the current time matches a specific value. You can prefix the value with a `/` to match whenever the value is divisible by that number. You can specify `*` to match any value.

```yaml
automation:
  trigger:
    platform: time_pattern
    # Matches every hour at 5 minutes past whole
    minutes: 5

automation 2:
  trigger:
    platform: time_pattern
    # Trigger once per minute during the hour of 3
    hours: '3'
    minutes: '*'

automation 3:
  trigger:
    platform: time_pattern
    # You can also match on interval. This will match every 5 minutes
    minutes: '/5'

automation 4:
  trigger:
    platform: time_pattern
    # This will match every 3 minutes and 15 seconds
    minutes: '/3'
    seconds: 15
```

<p class='note warning'>
  Do not prefix numbers with a zero - using `'00'` instead of '0' for example will result in errors. Also: maximum allowed value for minutes and seconds is 59. So don't try 120 seconds but use 2 minutes.
</p>

### {% linkable_title Webhook trigger %}

Webhook triggers are triggered by web requests made to the webhook endpoint: `/api/webhook/<webhook_id>`. This endpoint does not require authentication besides knowing the webhook id. You can either send encoded form or JSON data, available in the template as either `trigger.json` or `trigger.data`.

```yaml
automation:
  trigger:
    platform: webhook
    webhook_id: some_hook_id
```

You could test triggering the above automation by sending a POST HTTP request to `http://your-home-assistant:8123/api/webhook/some_hook_id`. An example with no data sent to a SSL/TLS secured installation and using the command-line curl program is `curl -d "" https://your-home-assistant:8123/api/webhook/some_hook_id`.


### {% linkable_title Zone trigger %}

Zone triggers can trigger when an entity is entering or leaving the zone. For zone automation to work, you need to have setup a device tracker platform that supports reporting GPS coordinates. This includes [GPS Logger](/components/device_tracker.gpslogger/), the [OwnTracks platform](/components/device_tracker.owntracks/) and the [iCloud platform](/components/device_tracker.icloud/).

```yaml
automation:
  trigger:
    platform: zone
    entity_id: device_tracker.paulus
    zone: zone.home
    # Event is either enter or leave
    event: enter  # or "leave"
```

### {% linkable_title Geolocation trigger %}

Geolocation triggers can trigger when an entity is appearing in or disappearing from a zone. Entities that are created by a [Geolocation](/components/geo_location/) platform support reporting GPS coordinates.
Because entities are generated and removed by these platforms automatically, the entity id normally cannot be predicted. Instead, this trigger requires the definition of a `source` which is directly linked to one of the Geolocation platforms.

```yaml
automation:
  trigger:
    platform: geo_location
    source: nsw_rural_fire_service_feed
    zone: zone.bushfire_alert_zone
    # Event is either enter or leave
    event: enter  # or "leave"
```

### {% linkable_title Multiple triggers %}

When your want your automation rule to have multiple triggers, just prefix the first line of each trigger with a dash (-) and indent the next lines accordingly. Whenever one of the triggers fires, your rule is executed.

```yaml
automation:
  trigger:
      # first trigger
    - platform: time_pattern
      minutes: 5
      # our second trigger is the sunset
    - platform: sun
      event: sunset
```
