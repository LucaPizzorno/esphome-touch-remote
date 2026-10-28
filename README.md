# ESPHome Touch Remote for Home Assistant

A battery-powered touch remote for the living room, built on a 2.8" ESP32-S3 display with **ESPHome** and **LVGL**. It goes into deep sleep by itself and wakes up with a touch.

It is **modular**: you choose which pages you want and how many lights, LED zones or appliances go on each one. The navigation bar at the bottom adapts by itself. On-screen texts are available in English and Italian.

<!-- Add a photo here: ![Remote](images/remote.jpg) -->
<img width="180" height="240" alt="20261028_162302-2" src="https://github.com/user-attachments/assets/acd8e7a4-eb03-4a3a-ad5c-5a364ab0eba8" />
<img width="180" height="240" alt="20261028_162320-2" src="https://github.com/user-attachments/assets/3ac3dcfa-8067-47a4-aee4-f1fc239cf196" />
<img width="180" height="240" alt="20261028_162316-2" src="https://github.com/user-attachments/assets/c0cea6c7-a9cb-4017-a2a6-e01fcb6cb811" />
<img width="180" height="240" alt="20261028_162313-2" src="https://github.com/user-attachments/assets/2b3a50b0-b7c4-418d-acb3-19344ec75123" />
<img width="180" height="240" alt="20261028_162309-2" src="https://github.com/user-attachments/assets/bcc7b389-2050-4ad4-b4cb-f7a0a09ba711" />
<img width="180" height="240" alt="20261028_162445-2" src="https://github.com/user-attachments/assets/7c7094e3-9898-43d9-9bf6-9998d3fd2b82" />
<img width="180" height="240" alt="20261028_162438-2" src="https://github.com/user-attachments/assets/f7d4115f-a86d-4ebe-8214-c950ab1a3406" />



> **A note from the author**
> I know very little about programming. This whole project (ESPHome configuration, interface and documentation) was created with the help of **Claude Opus**, Anthropic's AI model, through many conversations in which I explained what I wanted and tried the results on the real device.
> I'm sharing it as is: it works on my setup, but treat it as a hobby project. Suggestions and improvements are very welcome.

## Pages

| Page | Files | What it does |
|---|---|---|
| **Lights** | `lights/page.yaml` + `lights/light.yaml` per light | On/off tiles that follow the real state in HA (even when you use the wall switch), plus "All on" / "All off". With more than 4 lights the grid scrolls |
| **LED** | `led/page.yaml` + `led/zone.yaml` per zone | Selectable zones with on/off, 6 quick colours and brightness (2-4 zones recommended) |
| **TV** | `tv/apple_tv.yaml` | Apple TV remote: power, arrows, OK, volume, play/pause, Menu, Home |
| **Appliances** | `appliances/page.yaml` + `appliances/appliance.yaml` per appliance | Remaining time and pause/resume button for washer, dryer, dishwasher… With more than 3 the list scrolls |
| **Power** | `power/page.yaml` | Instant home power consumption with 4 coloured bars up to your supply limit |
| **Settings** | `settings.yaml` | Opened from the gear at the top: screen brightness and "Stay awake" for OTA updates |

The battery indicator is always shown at the top right, with an estimate of whether it is charging.

## Hardware

- **Freenove ESP32-S3 Display 2.8"** (FNK0104), ILI9341 240×320 with FT6336U capacitive touch
- LiPo battery **102050 1000 mAh** (JST 1.25 connector) on the board's battery connector
- *(optional)* **Qi** wireless charging receiver with USB-C output, connected to the board's USB port. The one I bought is roughly 76x46mm , please consider the dimension and edit the case file accordingly.
- *(optional)* 3D-printed case: files in the [`case/`](case/) folder

## Structure

```
esphome/
├── ha-remote.yaml            ← the only file to edit: pages and entities
├── secrets.yaml.example
└── remote/                   ← the building blocks, no need to touch them
    ├── core.yaml             (board, battery, deep sleep, bottom bar)
    ├── settings.yaml
    ├── lang/       en.yaml, it.yaml
    ├── lights/     page.yaml, light.yaml
    ├── led/        page.yaml, zone.yaml<img width="900" height="1200" alt="20261028_162438" src="https://github.com/user-attachments/assets/069d0c05-af5c-4787-afc7-98c18ca86612" />

    ├── tv/         apple_tv.yaml
    ├── appliances/ page.yaml, appliance.yaml
    └── power/      page.yaml
```

## Installation

1. Copy the contents of the `esphome/` folder into your ESPHome configuration (for the Home Assistant add-on it is `/config/esphome/`), keeping the `remote/` subfolder.
2. Create `secrets.yaml` from [`secrets.yaml.example`](esphome/secrets.yaml.example). If you already have a `secrets.yaml` for other devices, just add the missing keys.
3. Customise `ha-remote.yaml` (see below).
4. Flash the first time over USB; after that, updates go over the air (OTA).
5. In Home Assistant, add the ESPHome device when it is discovered and enter the API key.
6. **Important:** in the ESPHome device options in HA, enable *"Allow the device to perform Home Assistant actions"*, otherwise the buttons won't control anything.

The first compile downloads the fonts (Montserrat from Google Fonts and Material Design Icons from GitHub), so an internet connection is needed.

## Customisation

Everything happens in the `packages:` section of `ha-remote.yaml`. The included file uses my own entities as an example.

**Language:** keep one of the two `language:` lines (`en.yaml` or `it.yaml`). To add a language, copy `remote/lang/en.yaml`, translate it and include your copy.

**Removing a page:** comment out or delete the page line and the lines of its entities. For example, without an Apple TV you delete the `tv:` block.

**Adding an item:** copy an existing block, give it a unique package name and change the variables. For example, a fifth light:

```yaml
  light_bathroom: !include
    file: remote/lights/light.yaml
    vars:
      key: bathroom              # short unique name (lowercase, digits, _)
      name: Bathroom             # text on the tile
      entity: light.bathroom
      icon: "\U000F0335"         # optional
```

**Order:** the order of the packages is the order of the buttons in the bottom bar, and the first page is the one shown at power-on. Keep `language` and `core` first and `settings` last.

The variables of each building block (required and optional, with defaults) are described at the top of its file in `remote/`. A few special cases:

- **LED zones with several strips:** `entity` is the light whose state and brightness are shown; with `extra_entities` you add more lights controlled together.
- **Appliances:** states depend on the integration. By default the machine is "stopped" when the state is `idle` and "paused" when it is `pause`. If your integration has many different states, use `active_states` to list the ones in which it is running and `state_labels` to give a text to the others. The dishwasher in the example file shows how.
- **Without pause/resume buttons** the appliance card is information only.

### Icons

Icons are [Material Design Icons](https://pictogrammers.com/library/mdi/). Find the icon, copy its code (e.g. `F0335`) and write it like this: `"\U000F0335"`, i.e. `\U000` followed by the code.

A few icons used in the example: `F0335` light bulb, `F0769` ceiling light, `F04B9` sofa, `F072A` washing machine, `F0917` tumble dryer, `F0502` television, `F0241` lightning bolt.

## Deep sleep and OTA updates

After 20 s without touches the board goes into deep sleep; a touch wakes it up in about 5 s (the first touch only wakes it). While asleep it can't receive OTA updates, so before updating:

1. wake the remote;
2. open the gear and turn on **Stay awake** (or tap the battery indicator: a dot appears);
3. start the update. The option turns itself off on restart.

## How it works (for those who want to modify it)

Each building block is an [ESPHome package](https://esphome.io/components/packages/) with variables. The files append their own actions to a few shared scripts defined in `core.yaml`, using `id: !extend`:

- `build_ui`: on boot, moves buttons, tiles and cards into their containers (bottom bar, lights grid, zone row, appliance list);
- `refresh_all`: on connection to HA, syncs everything with the real state;
- `nav_refresh`: on page change, highlights the right button in the bar.

To create a new page, start from `power/page.yaml`, which is the simplest.

## Notes

- If the touch responds mirrored, see the comments in the `touchscreen` section of `core.yaml`.
- The board does not route the charging signal to the processor: the "charging" state is estimated from the battery voltage trend.
- GPIO45 and GPIO46 are strapping pins and ESPHome shows a warning: this is normal for this board.

## License

Code released under the [MIT](LICENSE) license.
