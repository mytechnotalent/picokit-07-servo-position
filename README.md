![picokit-07-servo-position](https://raw.githubusercontent.com/mytechnotalent/picokit-07-servo-position/main/picokit-07-servo-position.png)

<br>

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/reverse-engineering)
## FREE Embedded Hacking Course [HERE](https://github.com/mytechnotalent/Embedded-Hacking)

<br>

# PICOKIT-07 SERVO POSITION

### SG90 Servo Position and Authenticated Heartbeat
#### Lesson 7 of the Picokit Series

<br>

***
**LEGAL DISCLAIMER:**
The information, tools, and code provided in this repository and course are strictly for educational, research, and defensive purposes only.

You are explicitly prohibited from using any materials contained herein to access, test, modify, or exploit any device, network, or system that you do not own 100% or for which you do not have explicit, documented, and legally binding authorization to interact with.

By using this repository and course, you acknowledge and agree that:

1. Any illegal, unauthorized, or malicious use of this information is solely your responsibility.
2. The author(s) and contributor(s) of this repository and course shall not be held liable for any damages, legal repercussions, criminal charges, or unauthorized actions resulting from the use, misuse, or abuse of the contents herein.
3. You will comply with all applicable local, state, national, and international laws regarding cybersecurity and computer fraud.

**IF YOU DO NOT AGREE WITH THESE TERMS, DO NOT USE THIS REPOSITORY AND COURSE.**
***

<br>
<br>

## Overview

The seventh Picokit lesson. The node drives an SG90 servo on GP14
with the hardware PWM peripheral and moves the horn to a position
commanded from a VS1838B infrared receiver or from the serial
console, and every five seconds it transmits an authenticated
heartbeat over LoRa to a Python gateway that logs and displays it.
It reuses the standard node shape and adds an interactive actuator
commanded over two separate input surfaces.

<br>

## What it teaches

- Driving an SG90 servo with a 50 Hz hardware PWM pulse train.
- Decoding a NEC infrared remote frame and parsing a console angle.
- Clamping an operator requested angle onto the servo range.
- Sealing a tiny JSON body with Argon2id and XChaCha20-Poly1305 and
  sending it with an AT+SEND over the RYLR998.
- The gateway side: receive, authenticate, reject, log, and display.

<br>

## Hardware

| Peripheral | Pico 2 pin | Role |
| --- | --- | --- |
| SG90 servo | GP14 | 50 Hz PWM position |
| VS1838B receiver | GP5 | NEC remote position input |
| Serial console | USB / UART0 | typed angle input |
| Onboard LED | GP25 | heartbeat, one blink per report |
| RYLR998 | GP8 TX / GP9 RX | LoRa heartbeat |
| Debug Probe | SWCLK/SWDIO/GND, GP0/GP1 | SWD and the console |

<br>

## How it works

The node runs `monitor_step` in a loop. It polls the infrared eye
and drains the console, applies any requested angle to the SG90
after clamping it onto the zero to one hundred eighty degree range,
blinks GP25 once per report, and every 5 seconds it seals
`{"n":7,"s":<seq>,"a":<angle>}` with the field key and sends it
over LoRa. The gateway authenticates each frame and only then
parses it.

<br>

## Build and flash

```bash
cd firmware
cmake -S . -B build -G Ninja -DPICO_BOARD=pico2 -DPICO_PLATFORM=rp2350-arm-s
cmake --build build
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg \
  -c "program build/picokit_07_servo_position.elf verify reset exit"
```

<br>

## Watch the node

Open the console at 115200 and reset, then type an angle and press
enter:

```text
BOOT
SET angle=90
POS angle=90 seq=0
IR 0x5A
POS angle=90 seq=0
RX from 0x0001, N bytes
```

<br>

## The gateway

```bash
cd gateway
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python3 listen.py --port /dev/cu.usbserial-A50285BI --hub 0001 --network 18 --db gateway.db
```

It prints `OK node=7 rssi=...` per authenticated heartbeat.
The terminal dashboard `python3 tui.py --db gateway.db` and the web dashboard
`python3 web/app.py --db gateway.db` show the same rows.

<br>

## Verify

```bash
python3 .opencode/skill/embedded-c-standard/audit_c_standard.py
python3 .opencode/skill/embedded-python-standard/audit_python_standard.py
python3 .opencode/skill/iot-readme-standard/validate_readme.py
python3 .opencode/skill/iot-banner-standard/validate_banner.py
python3 scripts/run_tests.py
python3 scripts/check_coverage.py
```

<br>

# Next
[picokit-08-servo-gauge](https://github.com/mytechnotalent/picokit-08-servo-gauge)

<br>

# License
[MIT License](https://github.com/mytechnotalent/picokit-07-servo-position/blob/main/LICENSE)
