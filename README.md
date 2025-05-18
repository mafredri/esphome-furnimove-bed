# esphome-furnimove-bed

Sample ESPHome configuration for controlling [DewertOkin FurniMove](https://www.dewertokin.com/news/news/furnimove-bedding-app-with-a-new-name/) beds via Bluetooth and an ESP32.

Notably, this repository implements the FurniMove BLE control commands (remote [RF-FLASHLINE](https://www.dewertokin.com/products/bedding/handsets/rf-flashline/), 94238) for the [OKIMAT 4 RF BT MEMORY](https://www.dewertokin.com/products/bedding/double-drives-2/okimat-4-rf-systems/okimat-4-rf-bt-memory/) motor. If you have a different bed, you may need to adjust the commands and durations.

## Features

- State aware control of under bed lighting (0x020000)
- Interruptable motor control (as covers)
  - Head up/down (0x01, 0x02)
  - Legs up/down (0x04, 0x08)
  - Both up/down (0x05, 0x0A)
  - Memory 1/2 (0x1000, 0x2000)
- Memory save (0x010000, then memory 1/2)
