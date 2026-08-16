---
title: Hydro-Québec Peak Events
description: Instructions on how to integrate Hydro-Québec peak events into Home Assistant.
ha_category:
  - Binary sensor
  - Energy
  - Sensor
ha_release: 2026.10
ha_iot_class: Cloud Polling
ha_config_flow: true
ha_codeowners:
  - '@Beat-YT'
ha_domain: hydroquebec_peak
ha_platforms:
  - binary_sensor
  - sensor
ha_integration_type: service
ha_quality_scale: bronze
related:
  - url: https://donnees.hydroquebec.com/explore/dataset/evenements-pointe/information/
    title: Hydro-Québec peak events open data
---

The **Hydro-Québec Peak Events** {% term integration %} tracks winter peak demand events (_événements de pointe_) published by [Hydro-Québec](https://www.hydroquebec.com/), the electricity utility of Québec, Canada.

During Québec winters (December 1 to March 31), Hydro-Québec announces peak events when electricity demand is high due to cold weather. Customers enrolled in a peak savings offer — such as the Winter Credit Option or Rate Flex D — reduce their consumption during these events and receive bill credits or lower rates in return. This integration exposes the event schedule so your automations can prepare for and react to peak events, for example by preheating your home before an event and lowering thermostats while one is in progress.

The integration uses [Hydro-Québec's public open data](https://donnees.hydroquebec.com/explore/dataset/evenements-pointe/information/). It does not connect to your Hydro-Québec account and requires no credentials.

{% include integrations/config_flow.md %}

{% configuration_basic %}
Offer:
  description: The Hydro-Québec peak savings offer you are enrolled in. The list is retrieved from Hydro-Québec's open data for the current season and includes residential and business offers.
{% endconfiguration_basic %}

To track more than one offer, add the integration again and select another offer. Each configured offer appears as its own service with its own set of entities.

## Supported functionality

For each configured offer, the integration provides the following entities. Events are published by Hydro-Québec a few hours before they occur, typically the day before.

### Sensors

- **Event begins**
  - **Description**: Start time of the peak event in progress, or of the next upcoming event. Unknown when no event is scheduled.
- **Event ends**
  - **Description**: End time of the peak event in progress, or of the next upcoming event. Unknown when no event is scheduled.

### Binary sensors

- **Peak event in progress**
  - **Description**: On while a peak event is currently active. Turns on and off at the exact event boundaries.
- **Peak event today AM**
  - **Description**: On if a morning peak event is scheduled for today.
- **Peak event today PM**
  - **Description**: On if an evening peak event is scheduled for today.
- **Peak event tomorrow AM**
  - **Description**: On if a morning peak event is scheduled for tomorrow.
- **Peak event tomorrow PM**
  - **Description**: On if an evening peak event is scheduled for tomorrow.

## Example automations

Preheat the home before a peak event by using the **Event begins** sensor with a time offset:

{% raw %}

```yaml
automation:
  - alias: "Preheat before peak event"
    triggers:
      - trigger: time
        at:
          entity_id: sensor.credit_hivernal_residentiel_cpc_d_event_begins
          offset: "-03:00:00"
    actions:
      - action: climate.set_temperature
        target:
          entity_id: climate.living_room
        data:
          temperature: 22
```

{% endraw %}

Lower the thermostats while a peak event is in progress:

{% raw %}

```yaml
automation:
  - alias: "Reduce heating during peak event"
    triggers:
      - trigger: state
        entity_id: binary_sensor.credit_hivernal_residentiel_cpc_d_peak_event_in_progress
        to: "on"
    actions:
      - action: climate.set_temperature
        target:
          entity_id: climate.living_room
        data:
          temperature: 18
```

{% endraw %}

## Data updates

The integration {% term polling "polls" %} Hydro-Québec's open data every 15 minutes. The feed is served from a content delivery network and the integration uses conditional requests, so polling is lightweight. Entity states also update at event boundaries and at midnight, independently of polling.

## Known limitations

- Events are only published for the current season. Outside the winter season, sensors report no upcoming events for most offers.
- The integration provides the event schedule only. It does not report your consumption or the credits you earn; those are available in your Hydro-Québec account.

## Removing the integration

This integration follows standard integration removal. No extra steps are required.

{% include integrations/remove_device_service.md %}
