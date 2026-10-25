# esphome-furnimove-bed

ESPHome packages for controlling [DewertOkin FurniMove](https://www.dewertokin.com/news/news/furnimove-bedding-app-with-a-new-name/) beds via Bluetooth and an ESP32.

Notably, this repository implements the FurniMove BLE control commands (remote [RF-FLASHLINE](https://www.dewertokin.com/products/bedding/handsets/rf-flashline/), 94238) for the [OKIMAT 4 RF BT MEMORY](https://www.dewertokin.com/products/bedding/double-drives-2/okimat-4-rf-systems/okimat-4-rf-bt-memory/) motor. If you have a different bed, you may need to adjust the commands and durations.

## Features

- State aware control of under bed lighting (0x020000)
- Interruptable motor control (as covers)
  - Head up/down (0x01, 0x02)
  - Legs up/down (0x04, 0x08)
  - Both up/down (0x05, 0x0A)
  - Memory 1/2 (0x1000, 0x2000)
- Memory save (0x010000, then memory 1/2)

## Usage

The packages are pulled straight from this repository by ESPHome, so the only file you keep in your ESPHome configuration directory is a small device configuration:

1. Copy [`examples/bedroom-bed.yaml`](examples/bedroom-bed.yaml) into your ESPHome configuration directory.
2. Add the values from [`examples/secrets.example.yaml`](examples/secrets.example.yaml) to your ESPHome `secrets.yaml` (Wi-Fi credentials, API encryption key and the bed MAC addresses).
3. Pick a board package and optional packages, and set `ref` to a release tag (or `main` to follow the latest changes). ESPHome re-fetches the packages according to `refresh`.

### Packages

| Package                         | Description                                                          |
| ------------------------------- | -------------------------------------------------------------------- |
| `packages/base.yaml`            | Required. Wi-Fi, API, OTA, web server, BLE, diagnostics.             |
| `packages/bed.yaml`             | Required, once per bed (`device_name`, `device_id`, `device_mac`).   |
| `packages/boards/*.yaml`        | Board/variant: `esp32`, `esp32-c3`, `esp32-s3`, `esp32-c6`.          |
| `packages/bluetooth_proxy.yaml` | Optional. Also act as a Home Assistant Bluetooth proxy.              |
| `packages/debug.yaml`           | Optional. Verbose logs and a runtime "Logger Level" select.          |

For other boards, replace the board package with your own `esp32:` block.

### Device-specific options

These go in your device configuration, next to `wifi:`:

```yaml
wifi:
  # ...
  # Fallback hotspot for changing Wi-Fi credentials when the configured
  # network is unreachable (needs captive_portal). Without a password the
  # hotspot is open and exposes the web server.
  ap:
    password: !secret bedroom_bed_fallback_ap_password

captive_portal:

esp32:
  framework:
    advanced:
      # Only if the boot log suggests it ("Set flash_chip: generic ... to
      # save IRAM"); the value depends on the flash chip of your board.
      flash_chip: generic
```

### Logging

By default only connection changes, commands and warnings are logged. When troubleshooting, add `packages/debug.yaml` to log every BLE write and advertisement, and use the "Logger Level" select to lower the level again without reflashing.

## Pairing the bed

1. Configure the device as described above and enter your bed MAC addresses in `secrets.yaml`.
2. Flash the ESP32 with the ESPHome firmware and this configuration.
3. Pull the power plug from the bed for 30 seconds or press a remote button to deplete the residual charge. (You may or may not need to unplug the beds backup battery.)
4. Power the bed back on (it should be blinking indicating it's in pairing mode) and wait for the ESP32 to connect to the bed.
5. Verify by viewing the ESP32 logs in the ESPHome dashboard.

## Development

Tooling is managed with [mise](https://mise.jdx.dev/):

```sh
mise run setup     # install tools and the git pre-commit hook
mise run lint      # yamllint, jactionlint, gitleaks
mise run validate  # esphome config for tests/*.yaml
mise run build     # esphome compile for tests/*.yaml
```

`tests/` contains configurations equivalent to the example, using the local packages and dummy secrets. CI runs lint, validation and a full compile on every push and pull request, and weekly against the latest ESPHome beta.

Never commit real MAC addresses or keys: put them in `secrets.yaml` (gitignored). gitleaks blocks MAC addresses other than documentation placeholders (`00:11:22:33:44:xx`). The gitignored `local/` folder is for private configuration variants, BLE packet captures and device logs.
