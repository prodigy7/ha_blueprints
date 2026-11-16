# Window & Door Reminder

Reminds you **per window/door** when it has been left open too long. Every contact that carries a chosen **label** is monitored, so you can create one automation per room (or per group of people) from the same blueprint.

[![Open your Home Assistant instance and show the blueprint import dialog with this blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fprodigy7%2Fha_blueprints%2Fblob%2Fmain%2Fautomation%2Fwindow_door_reminder.yaml)

## Features

- **Fixed wait time** before the first reminder, then a fixed **repeat interval**. No temperature-dependent stretching.
- **No reminders above a limit temperature** (outside temperature from a weather entity), unless rain is forecast within the next two hours.
- **Voice output** on the Alexa that sits in the same *area* as the sensor, plus optional extra Alexa devices.
- **Push notifications** to chosen Home Assistant app devices with optional buttons: *snooze*, *pause this window*, *pause all*. The notification is cleared automatically when the window is closed.
- **Escalation** after N reminders (urgent channel, additional phones/Alexas).
- **Quiet period** driven by a `schedule` helper: Alexa speaks at a low volume (and restores it afterwards), push notifications are silent.
- **Immediate alerts** (each one optional) for rain/storm while something is open, for leaving home with something open, and for sensors that have been unreachable for a while.
- **Survives restarts**: after a Home Assistant restart, reminders resume for everything that is still open.
- **Fully translatable**: all spoken and pushed texts, button captions and Android channel names are blueprint inputs.

## Requirements

- Home Assistant **2026.7** or newer (the blueprint uses the purpose-specific `window.opened` / `door.opened` / `window.closed` / `door.closed` triggers with a label target).
- Window/door contacts as `binary_sensor` entities with the device class `window`, `door`, `opening` or `garage_door`.
- A **label** for each group of sensors (see [Setup](#setup)).
- At least one of: a mobile app device (push) or an Alexa notify entity from the `alexa_devices` integration (voice).
- A `weather` entity (outside temperature, wind speed, hourly forecast).

## Setup

1. **Import the blueprint** with the button above (or copy `automation/window_door_reminder.yaml` to `config/blueprints/automation/<your-folder>/`).
2. **Create a label** per sensor group, e.g. `Window warning living room`.
3. **Assign the label** to the window/door contacts, either on the **entity** or on its **device**. Both work.
4. *(Optional)* Assign Alexa devices to the same **area** as the sensors so the announcement comes from the right room.
5. *(Optional)* Create helpers for muting and the quiet period:
   - an `input_boolean` for "mute until everything is closed",
   - a `schedule` for the quiet period (e.g. nightly).
6. **Create an automation** from the blueprint: choose the label, the recipients, the weather entity, and adjust times and texts.

One automation per label means one set of recipients per room, e.g. the bedroom windows notify the parents, a child's room notifies the child and the parents.

## How it works

For each window/door that opens, the automation starts an independent run (`mode: parallel`) that is bound to that particular opening (identified by the sensor's `last_changed` timestamp):

1. Wait **first wait** minutes. If the window is closed in the meantime, the run ends.
2. Loop while it is still open:
   1. Skip this round if the outside temperature is at or above the **limit temperature** (unless rain is forecast) or the **mute switch** is on.
   2. Send push + voice output (with escalation and quiet-period handling).
   3. Wait for the **repeat interval**, or for a button press (snooze or pause).

Closing the window clears the push notification (via a notification tag per sensor).

### Buttons

| Button | Effect |
|---|---|
| Snooze (always shown) | No reminder for this window for *snooze minutes*, then reminders continue. |
| Pause (optional) | No more reminders for this window until it is closed. |
| Pause all (optional) | Turns on the **mute switch**. Never shown when no mute switch is configured. |

### Resetting the mute switch

The blueprint **never turns the mute switch off**: several automations created from this blueprint can share one switch, and each of them would otherwise reset it as soon as *its own* sensors are closed. Use one separate automation that switches the mute off when **everything** is closed. Example with a binary template helper `binary_sensor.any_window_open` (on if any monitored contact is open):

```yaml
alias: Window warning - end mute when everything is closed
triggers:
  - trigger: state
    entity_id: binary_sensor.any_window_open
    to: "off"
actions:
  - action: input_boolean.turn_off
    target:
      entity_id: input_boolean.window_warning_mute
```

## Inputs

### Sensors

| Input | Default | Description |
|---|---|---|
| `sensor_label` | – | Label whose window/door contacts are monitored (on the entity or its device). |

### Recipients

| Input | Default | Description |
|---|---|---|
| `mobile_devices` | `[]` | Mobile app devices that receive push notifications. |
| `alexa_in_room` | `true` | Announce on the Alexa assigned to the same area as the sensor. |
| `alexa_mode` | `announce` | `announce` (with chime) or `speak` (no chime). |
| `extra_outputs` | `[]` | Additional Alexa notify entities that always speak. |

### Wait times & temperature

| Input | Default | Description |
|---|---|---|
| `weather` | – | Weather entity (temperature, wind, forecast). |
| `limit_temperature` | `16 °C` | At or above this outside temperature there are no reminders (unless rain is forecast). |
| `first_wait` | `10 min` | Fixed wait before the first reminder. |
| `repeat_interval` | `5 min` | Time between reminders. |
| `snooze_minutes` | `15 min` | Length of the snooze button. |

### Escalation

| Input | Default | Description |
|---|---|---|
| `escalate_from` | `3` | Escalate from reminder number N. `0` disables escalation. |
| `escalation_devices` | `[]` | Additional phones on escalation. |
| `escalation_outputs` | `[]` | Additional Alexa notify entities on escalation. |

### Quiet period & muting

| Input | Default | Description |
|---|---|---|
| `quiet_schedule` | – | `schedule` helper. While on: low Alexa volume, silent push. |
| `quiet_volume` | `20 %` | Alexa volume during the quiet period (restored afterwards). |
| `mute_switch` | – | `input_boolean`. While on there are no reminders. |
| `pause_button` | `false` | Show the *Pause* button. |
| `pause_all_button` | `false` | Show the *Pause all* button (needs a mute switch). |

### Immediate alerts & misc

| Input | Default | Description |
|---|---|---|
| `weather_warning` | `true` | Alert when rain/storm starts while something is open. |
| `wind_limit` | `50 km/h` | Wind speed that counts as a storm. |
| `rain_probability` | `60 %` | Rain counts as forecast from this probability within 2 hours. Without a probability from the weather service, forecast precipitation or rainy conditions count. |
| `leaving_warning` | `true` | Alert when the home zone becomes empty while something is open. |
| `home_zone` | `zone.home` | Zone used for the leaving check. |
| `offline_warning` | `true` | Alert for unreachable sensors. |
| `offline_minutes` | `30 min` | How long a sensor must be unavailable. |
| `dashboard_url` | – | Where a tap on the notification leads, e.g. `/dashboard-home/start#open`. |

### Message texts

All texts are plain text with placeholders. **Do not use Jinja (`{{ }}`)** in these fields. The placeholders are replaced by the blueprint.

| Input | Placeholders | Default |
|---|---|---|
| `title_open` | `{name}` `{minutes}` | `{name}: Open` |
| `text_open` | `{name}` `{minutes}` | `{name} has been open for {minutes} minutes. Please close it.` |
| `text_rain_suffix` | – | ` Rain is forecast.` |
| `voice_open` | `{name}` `{minutes}` | `{name} is open. Please close it.` |
| `button_snooze` | `{minutes}` | `Remind me in {minutes} min` |
| `button_pause` | – | `Pause` |
| `button_pause_all` | – | `Pause all` |
| `word_rain` / `word_storm` | – | `Rain` / `Storm` (used for `{reason}`) |
| `title_weather` | `{reason}` `{open}` | `{reason} - window open` |
| `text_weather` | `{reason}` `{open}` | `Still open: {open}` |
| `voice_weather` | `{reason}` `{open}` | `Attention, {reason}. Still open: {open}.` |
| `title_leaving` | `{open}` | `Nobody home - still open` |
| `text_leaving` | `{open}` | `{open}` |
| `title_offline` | `{unreachable}` | `Window sensor unreachable` |
| `text_offline` | `{unreachable}` | `No signal from: {unreachable}. Whether open or closed is unknown.` |
| `channel_normal` / `channel_quiet` / `channel_urgent` | – | `Window warning` / `Window warning (quiet)` / `Window warning (urgent)` |

Placeholders: `{name}` friendly name of the window/door, `{minutes}` minutes it has been open (or the snooze time for the snooze button), `{open}` comma-separated names of everything currently open, `{reason}` the word for rain or storm, `{unreachable}` names of unreachable sensors.

### German texts

To get German notifications, set these values in the automation (the placeholders stay English):

| Input | Value |
|---|---|
| `title_open` | `{name}: Offen` |
| `text_open` | `{name} ist seit {minutes} Minuten offen. Bitte schließen.` |
| `text_rain_suffix` | ` Regen ist angekündigt.` |
| `voice_open` | `{name} ist offen. Bitte schließen.` |
| `button_snooze` | `In {minutes} Min erinnern` |
| `button_pause` | `Aussetzen` |
| `button_pause_all` | `Alle aussetzen` |
| `word_rain` / `word_storm` | `Regen` / `Sturm` |
| `title_weather` | `{reason} – Fenster offen` |
| `text_weather` | `Noch offen: {open}` |
| `voice_weather` | `Achtung {reason}. Noch offen: {open}.` |
| `title_leaving` | `Niemand zuhause – noch offen` |
| `text_leaving` | `{open}` |
| `title_offline` | `Fenstersensor nicht erreichbar` |
| `text_offline` | `Kein Signal von: {unreachable}. Ob offen oder zu, ist unbekannt.` |
| `channel_normal` / `channel_quiet` / `channel_urgent` | `Fensterwarnung` / `Fensterwarnung leise` / `Fensterwarnung dringend` |

## Notes and limitations

- **Android** push uses `priority: high`, `ttl: 0` and a separate notification channel per mode (normal, quiet, urgent). Channel settings (sound, vibration) can be changed per channel in the Android app settings.
- **Alexa** announcements use `notify.send_message` on `alexa_devices` notify entities. The room Alexa is found through the **area** of the sensor. Entities whose id contains `sprech` or `speak` are treated as the *speak* variant, all others as *announce*.
- **Push targets** are derived from the app device name (`notify.mobile_app_<slugified device name>`). If you rename the device in the app, the service name follows the name set **in the app**.
- Changing blueprint **input names** breaks existing automations until their inputs are updated. This blueprint was renamed to English once (see the changelog).
- Not covered by the blueprint on purpose: resetting the mute switch (see above).

## Changelog

- **1.0.0** – First public version: English inputs and configurable texts, label on entity or device, per-window reminders with fixed intervals, optional snooze/pause/pause-all buttons, escalation, quiet period, rain/storm, leaving-home and offline alerts.
