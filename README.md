# ha_blueprints

A collection of [Home Assistant](https://www.home-assistant.io/) blueprints.

## Blueprints

| Blueprint | Type | Description | Import |
|---|---|---|---|
| [Window & Door Reminder](docs/window_door_reminder.md) | Automation | Per-window/door reminders with push, Alexa, escalation, quiet period, rain/storm, leaving-home and offline alerts. Configurable texts. | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fprodigy7%2Fha_blueprints%2Fblob%2Fmain%2Fautomation%2Fwindow_door_reminder.yaml) |

## Installation

Use the **Import** button next to a blueprint (requires a public repository), or paste the blueprint's GitHub URL under *Settings → Automations & Scenes → Blueprints → Import Blueprint*. Alternatively copy the YAML file into `config/blueprints/<automation|script>/<your-folder>/` and reload.

## Repository layout

```
automation/   blueprints for automations
script/       blueprints for scripts
docs/         one documentation page per blueprint
```

## Conventions

- Inputs are English `snake_case`; every user-facing text is a configurable input.
- Every blueprint carries a `source_url` so Home Assistant can offer updates.
- Each blueprint has its own page in `docs/` (features, requirements, inputs, notes) and a changelog.

## License

[MIT](LICENSE)
