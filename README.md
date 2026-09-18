# RC Controller

A DIY remote-control transmitter built around an Arduino Pro Mini and a repurposed USB game-pad's joysticks and buttons, sending its readings wirelessly over an nRF24L01 radio. Built as the controller half of the [Quadruped-On-Arduino](https://github.com/KozhaAkhmet/Quadruped-On-Arduino) robot.

## Hardware

- Arduino Pro Mini
- nRF24L01 2.4GHz radio module
- Two analog joysticks (4 potentiometer axes total)
- 5 momentary buttons (D-pad/arrow input + 4 shape buttons, salvaged from an old USB controller)

## How It Works

`Pro_mini/Pro_mini.ino` polls the joystick axes and buttons every loop and packs them into a single `Data_Package` struct:

| Field | Source | Description |
| ----- | ------ | ------------ |
| `j1PotX`, `j1PotY` | `A5`, `A3` | Left joystick, mapped from the raw `0–1023` analog reading down to a `0–255` byte |
| `j2PotX`, `j2PotY` | `A4`, `A1` | Right joystick, same mapping |
| `Arrows` | `A0` | D-pad, read as a single analog voltage (each direction sits in its own voltage band) |
| `Circle`, `Square`, `Triangle`, `Ex` | `D8`, `D5`, `D6`, `D7` | Shape buttons, read as plain digital input |

That struct is broadcast over the air with `radio.write()` on a fixed pipe address (`"10101"`), at `RF24_250KBPS` / low power, with auto-ack disabled — a receiver (like the one on the quadruped) listens on the same address and pipe to decode it. A `Variables()` helper also prints all the raw readings to `Serial` for debugging on the bench.

## Build & Upload

1. Open `Pro_mini/Pro_mini.ino` in the Arduino IDE.
2. Install the [`RF24`](https://github.com/nRF24/RF24) library via the Library Manager (it pulls in `nRF24L01.h` support).
3. Wire the nRF24L01's CE/CSN pins to Arduino pins 9/10 (as set by `RF24 radio(9, 10);`), and the joystick/button pins as listed in `setup()` (`D2`–`D8`, `A0`–`A5`).
4. Select "Arduino Pro or Pro Mini" as the board and flash.
