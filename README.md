# Hello AtomVM Port

A tiny AtomVM port driver (ESP-IDF component) called from an Elixir app on ESP32-S3.

## What’s in this repo

- `ports/`
  ESP-IDF component that registers a port named `sample_app_port`.
  It supports:
  - PING (<<0x01>>) -> OK + "PONG"
  - ECHO (<<0x02, payload>>) -> OK + payload

- `examples/hello_port_elixir/`
  Elixir app that opens the port and calls it via `:port.call/2`.

## How it works

1. Add (or symlink) this repo under AtomVM’s ESP32 components directory:
   `AtomVM/src/platforms/esp32/components/`

2. Build + flash AtomVM firmware with the component included.

3. Build the Elixir app (`mix atomvm.packbeam`) and flash the generated `.avm`.

## Expected output

```text
Starting application...
Ping: :ok
Echo req: "hello from Elixir"
Echo res: {:ok, "hello from Elixir"}
```
