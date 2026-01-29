# Hello AtomVM Port

A tiny “hello world” AtomVM port driver (ESP-IDF component) called from an
Elixir app on ESP32.

This repo includes:

- A native port driver written in C (`sample_app_port`)
- A minimal Elixir example that calls the driver via `:port.call/2`

Tested on ESP32-S3, but the example is intended to work on ESP32 targets
supported by AtomVM + ESP-IDF.

![](https://github.com/user-attachments/assets/8840b0e2-68bd-408f-b7c3-1d27bc9e5310)

## What’s in this repo

- `ports/`
  - ESP-IDF component that registers a port named `sample_app_port`
  - Protocol (binary):
    - Request: `<<opcode::8, payload::binary>>`
    - Reply (success): `<<0x00, payload::binary>>`
    - Reply (error): `<<0x01, code::8>>`
  - Opcodes:
    - `0x01` Ping → returns `"PONG"`
    - `0x02` Echo → returns payload unchanged

- `examples/elixir/`
  - Elixir app that opens the port and calls it via `:port.call/2`

## How it works

Message flow:

```text
Elixir process -> :port.call/2 -> native driver (C) -> reply -> Elixir
```

## Build and run

This section assumes you want a fresh clone setup. If you already have AtomVM
checked out, skip the clone step and set `ATOMVM_REPO_PATH` accordingly.

### Clone repos (fresh start)

```sh
# Paths (adjust if you want)
ATOMVM_REPO_PATH="$HOME/atomvm/AtomVM"
ATOMVM_ESP32_PATH="$ATOMVM_REPO_PATH/src/platforms/esp32"
PORT_COMPONENT_PATH="$ATOMVM_ESP32_PATH/components/hello_atomvm_port"

# Clone AtomVM
git clone https://github.com/atomvm/AtomVM.git "$ATOMVM_REPO_PATH"

# Clone this example directly into AtomVM's ESP32 components directory
git clone https://github.com/piyopiyoex/hello_atomvm_port.git "$PORT_COMPONENT_PATH"
```

### Configure partition table for the Elixir app

AtomVM needs a partition table that includes the Elixir partition (`main.avm`).

Edit `"$ATOMVM_ESP32_PATH/sdkconfig.defaults"` and ensure it contains:

```ini
CONFIG_PARTITION_TABLE_CUSTOM_FILENAME="partitions-elixir.csv"
```

> You may already need to edit `sdkconfig.defaults` for other settings (flash size, Wi-Fi, etc).
> The only required line for this example is the partition table filename.

### Build and flash AtomVM firmware

```sh
cd "$ATOMVM_ESP32_PATH"

source "$HOME/esp/esp-idf/export.sh"

idf.py fullclean
idf.py set-target esp32s3
idf.py build
idf.py -p /dev/ttyACM0 flash
```

> If you’re using a different ESP32 target, change the `set-target` value accordingly.

### Build and flash the Elixir app

```sh
cd "$PORT_COMPONENT_PATH/examples/elixir"

mix deps.get
mix do clean + atomvm.esp32.flash --port /dev/ttyACM0
```

### Monitor serial output

```sh
cd "$ATOMVM_ESP32_PATH"
idf.py -p /dev/ttyACM0 monitor
```

## Expected output

```text
Starting application...
Ping: :ok
Echo request: "hello from Elixir: 1700000000"
Echo reply: "hello from Elixir: 1700000000"
```
