# Pi 5 running ArduPilot and AI, with optional Pico I/O

Updated October 30, 2026. This is the current computing architecture and supersedes the separate-flight-controller assumption in earlier project research. It is a design study, not a working flight-controller implementation.

## Current decision

The team probably will not use a separate board running flight stabilization. **Raspberry Pi 5 will run both native ArduCopter and the AI application.** One or more Raspberry Pi Picos may manage sensors and other I/O; their exact number and responsibilities remain open. Do not silently move attitude estimation or stabilization to a Pico: that would be another architecture decision.

“Flight controller” describes a function, not necessarily a separate product. In this plan the Pi becomes the flight controller. The aircraft still needs the sensors, reliable motor-output hardware, power monitoring and failure handling normally supplied by a conventional FC board.

The current airframe study remains a custom 7-inch-propeller quad, tentatively 330–360 mm motor diagonal, 6S power, FPV and a desired twenty minutes of frequent fast maneuvers. The latter remains a stretch requirement. See the [battery and mounting study](ENDURANCE_AND_MOUNTING_PLAN.md).

The [movement and upgrade requirements](MOVEMENT_AND_UPGRADE_SPEC.md) now define a **2.2 kg future loaded case** and approximately **4:1 or better peak thrust-to-weight at that mass**, replacing earlier 1.9 kg / 3:1 screening. The Pi/Pico timing and output path must be validated for the selected propulsion; additional thrust does not compensate for stale sensor data or missed output updates.

## What is supported, and what is not established

- ArduPilot officially supports [Linux-based autopilots](https://ardupilot.org/copter/docs/common-autopilots.html#linux-based-autopilots). Linux flight control is possible; a normal desktop configuration under arbitrary AI load is not thereby validated.
- Upstream [Pi 5 GPIO/RP1 support was merged](https://github.com/ArduPilot/ardupilot/pull/27026) in May 2024. This does not establish compatibility with every older Pi expansion board, sensor or RC input method.
- The official firmware service publishes an [ArduCopter navigator64 target](https://firmware.ardupilot.org/Copter/stable/navigator64/). A target exists for particular hardware; it is not a generic image that automatically discovers arbitrary sensors and custom Pico firmware.
- Emlid's [Navio2 supported-hardware list](https://docs.emlid.com/navio2/hardware-setup/) names Pi 2/3/4, not Pi 5. Do not buy Navio2 on the assumption that its old Pi image is compatible. Community Pi 5 Rover ports are not proof of quadcopter suitability.
- **SITL on a Pi is a simulator**, not a connection to real IMU/ESC hardware. Use the correct native Linux hardware target for flight development.

No selected Pi 5 + Pico hardware interface has yet been integrated with ArduCopter, timing-tested under AI load or flown as part of this project.

## Proposed responsibilities

```text
Remote -> receiver -> supported serial input -> ArduCopter on Pi 5

Primary IMU -> short SPI link ----------------> ArduCopter on Pi 5
Other sensors -> Pi directly OR sensor Pico -> supported driver/bridge -> ArduCopter

AI camera -> AI process on Pi 5
                    |
              local MAVLink link
                    |
             ArduCopter on Pi 5
                    |
          supported output driver/link
                    |
       output hardware / optional I/O Pico -> ESC -> motors

FPV camera -> video transmitter -> goggles
```

The serial receiver may connect directly to a Pi UART, through a compatible expansion board, or through a deliberately designed I/O Pico. Start with the simplest supported receiver path; do not route pilot commands through the AI program. Verify receiver protocol, inversion, voltage levels and driver support.

| Function | Initial owner | Integration requirement |
|---|---|---|
| State estimation, attitude/rate control and motor mixing | ArduCopter on Pi | Native hardware build, correct sensor drivers and measured timing |
| Camera capture, inference and project decisions | Separate Pi processes | Bounded resource use, timestamps and stale-data handling |
| Primary gyro/accelerometer | Prefer a supported SPI IMU directly connected to Pi | Short wiring, stable orientation, supported sampling/timestamps |
| Barometer, GPS/compass, battery voltage/current | Pi-compatible interfaces or supported sensor/I/O module | Actual data must reach the corresponding ArduPilot drivers |
| Extra project sensors, lights, status or simple peripheral control | Optional Pico(s) | Explicit protocol, rates, units, timestamps and failure behavior |
| Precise ESC output and stale-command handling | Supported output hardware, optionally an I/O Pico | ArduPilot output backend, tested protocol and independent timeout |
| Flight logging and ground-station telemetry | ArduCopter / bounded logging services | Avoid storage contention and distinguish logging time from sensor time |

A Pico can acquire an IMU, but then delivery latency, jitter, sample timestamps, clock synchronization and missing samples become part of the flight-control path. Prefer a direct supported primary IMU initially; use a sensor Pico first for slower or non-flight-critical measurements. Arbitrary USB/serial lines containing sensor readings are not automatically ArduPilot sensor inputs.

If a Pico is used for outputs, it receives motor commands already calculated by ArduCopter. It must not independently reinterpret AI detections into motor speeds. A basic implementation needs startup outputs held inactive, explicit arming/disarming state, message integrity, command sequence/age checking and a defined timeout. These details require an actual interface specification and firmware; no such firmware is supplied here.

## Hardware options

### Existing Linux sensor/I/O expansion board

The [Blue Robotics Navigator](https://bluerobotics.com/store/comm-control-power/control/navigator/) is an example of the required functions: IMU, barometer, compass, ADC and hardware PWM outputs. It is an I/O/sensor board accessed by ArduPilot running on the Pi, rather than an independent MCU running the flight-control loop.

At checking, its kit started at **$220**, the PCB was listed at **26 g**, and its included heatsink at **30 g**. Its revision history mentions Pi 5 compatibility changes, while the main specification still lists Pi 4. Manufacturer discussion also identifies Pi 5 cooling/power integration work: see the [Pi 5 discussion](https://discuss.bluerobotics.com/t/pi-5-with-navigator/15903?page=2). Treat it as a bench-evaluation candidate, not an unconditional plug-and-play recommendation for this airborne build. Verify actual board revision, Pi 5 cooling, power path and native Copter image before buying. In particular, do not assume the listed 4 A continuous through-board power path satisfies a 5 A Pi supply requirement.

This route demonstrates why removing a small FC board does not necessarily reduce cost or installed weight. It may nevertheless reduce custom driver development compared with an entirely new interface.

### Custom sensor/I/O module with Pico(s)

This fits the team's custom electrical-design goal, but adds embedded software work:

1. Choose supported IMU/barometer and receiver interfaces; document exact buses, pins and voltage levels.
2. Decide whether the output Pico connects over SPI, UART or another supported link. Select for measured latency and reliability, not connector convenience alone.
3. Implement the corresponding Linux ArduPilot driver/backend and Pico firmware where an existing compatible implementation is unavailable.
4. Implement/verify battery sensing and telemetry. Pi GPIO is not a battery-voltage ADC; never connect 6S voltage directly to a logic pin.
5. Establish arming, boot ordering, stale-command behavior and failure tests before applying motor power.

An RP2040/RP2350 can be an I/O processor, but merely purchasing a Pico does not make it an ArduPilot-compatible motor controller or sensor bridge. Do not assume stock AP_Periph firmware supports the proposed board and protocol. Small sensor boards can be economical; one-off PCB fabrication, debugging and development time must be budgeted separately.

### ESC protocol decision

The current ArduPilot [Linux-autopilot documentation](https://ardupilot.org/copter/docs/common-autopilots.html#linux-based-autopilots) lists lack of DShot/bidirectional DShot among its limitations. Start by verifying a supported hardware-PWM path and compatible ESCs. A Pico could generate other protocols in a custom implementation, but waveform generation alone does not provide a working ArduPilot output driver, telemetry support or timing validation. Do not carry the earlier H743/DShot setup into this architecture unchanged.

## Software structure

Run **ArduCopter as its own native process**, using the selected Linux hardware target. Keep camera capture, AI, mission logic and logging outside its core control loop. The first project application can use Python/pymavlink and a local UDP endpoint; the autopilot is on the same computer, so a Pi-to-FC serial cable is no longer required. Physical UARTs remain useful for receiver/GPS/peripherals.

The planned telemetry service will use a distinct local MAVLink output from the native autopilot/router and verify source IDs and timestamps. See the [software scope](../README.md#software). Timing monitoring and output watchdogs require separate implementations and validation.

ArduPilot itself is C++; Pico firmware will normally be C/C++ with the corresponding SDK. Use Python for AI and telemetry applications. Sensor and output integration requires the appropriate embedded interfaces, interrupt/timer behavior, binary protocols and Linux HAL interfaces. Avoid rewriting ArduPilot's stabilizer to accommodate a sensor bridge.

Linux scheduling work is a first-class project task. ArduPilot uses [threads with realtime priorities](https://ardupilot.org/dev/docs/learning-ardupilot-threading.html). Evaluate a suitable realtime-capable kernel/configuration, permissions, priorities, CPU affinity/IRQ placement and bounded memory/storage use for the chosen target. CPU-core separation is useful to investigate, but does not isolate shared memory bandwidth, kernel drivers, power or heat. Do not assume “one core for flight, three for AI” proves deadlines are met.

Test with the actual inference model, camera rate/resolution, logging and radio traffic, including worst observed load. Record control-loop overruns, sensor age, output-update gaps, CPU scheduling behavior, throttling and undervoltage. Reduce or suspend AI work before it compromises control timing. Exact acceptable timing limits depend on the configured loop, sensor and output path; they cannot be set here without those choices.

## Failure behavior changes

- An AI **process** crash need not stop the separate ArduCopter process, provided resource and error handling are properly designed.
- A Pi power loss, kernel hang, severe resource problem or reboot can stop **both** AI and stabilization. Moving processes into containers does not supply an independent flight computer.
- Pilot override can leave an AI-guided mode only while the autopilot/input path is healthy. The receiver cannot restore attitude control when the Pi is no longer running it.
- An independently executing I/O Pico/watchdog can detect absent or stale motor commands and apply a defined output action. It cannot hold attitude, land or return home without independently implementing flight-control functions. Cutting motors in flight causes loss of lift; it is a defined failure response, not continued flight.
- A watchdog must monitor fresh progress in the command path, not merely any Pi process that is still sending heartbeats. A stalled control task must not be masked by a healthy unrelated heartbeat task. Behavior of any output hardware on host loss must be tested; some generators can retain the last output.

The project must decide which failures it intends to tolerate and what response is acceptable. There is no requirement to add a second flight computer, but the documentation must not retain the old claim that stabilization survives a Pi reboot.

## Weight, power and battery consequences

Keep the earlier battery classes: **6S 3300–4000 mAh high-power LiPo for dynamic-flight development**, and a frame envelope for **6S2P 8000–10000 mAh high-power Li-ion endurance testing**. Do not reduce capacity simply because there is no separate FC: sensor/output hardware remains, and the Pi now runs the flight software too.

Until actual parts are chosen, reserve **60 g** for sensor/output interface hardware, optional Pico(s), receiver, GPS and modest sensors, replacing the earlier 35 g FC/receiver/sensor allowance. This makes the provisional dry allocation **875 g including the existing 80 g growth reserve**. It is an allocation, not a measured BOM. If a large commercial I/O kit or several modules exceed it, revise the full budget.

Keep an initial **about 40 W average battery-side electronics allocation** for the combined Pi/autopilot/AI, camera, modest Pico/sensor hardware, FPV and conversion losses. Retain a correctly implemented 5 V / 5 A-capable Pi branch and a separately budgeted, quiet I/O/sensor supply where appropriate. Measure actual average/peak draw and startup behavior under the simultaneous workload; the power is not validated by this allocation. High-power sensors or accelerators require an explicit increase.

As an illustration, even a genuine 2 W electronics saving would save only 0.67 Wh over twenty minutes, equivalent to about **38 mAh of nominal 6S capacity** at the earlier 80% usable-energy assumption. No 2 W saving is claimed here. Propulsion, mass and maneuver profile remain the primary endurance-sizing questions.

At the revised dry allocation, the listed 4000/8000/10000 mAh examples give gross budgets of approximately **1.450 / 1.715 / 1.730 kg** respectively. Retain the provisional **1 kg maximum battery class**, but use the new **2.2 kg future fully loaded design case** for structural and propulsion assessment instead of the previous 1.9 kg screening envelope. Initial dry allocations are unchanged until parts are selected; any larger ESC, motor, connector or interface consumes reserve/growth budget. Battery dimensions and tray allowance remain unchanged, while the central Pi/sensor module still requires a packaging study. No payload rating is established.

## Mounting changes

1. Place the Pi and primary flight IMU near the aircraft's center of mass. Keep the IMU rigidly oriented relative to the airframe and its bus short. Protect the barometer from light/propwash as appropriate without sealing its pressure reference.
2. If the IMU is on a Pi expansion board, assess vibration isolation for the **whole Pi + sensor assembly**. Its mass and wiring differ substantially from a tiny isolated FC. If the IMU is separate, design and verify that mount instead; do not let cables bypass the intended isolation.
3. Use a removable central electronics tray, accommodating Pi cooler, sensor/I/O hardware, optional Picos, connectors and accessible programming ports. A loosely suspended computer is not a substitute for a designed vibration mount.
4. Keep the cooler's magnetic and electrical effects away from the compass; verify interference under load. Preserve airflow and test sustained control + AI with the actual upper battery deck installed.
5. Place an output Pico near the ESC signal connectors and peripheral Picos near their sensors where useful. Route short, secured interconnects outside rotor sweep. Keep power/ESC noise away from IMU and receiver wiring.
6. Retain the adjustable battery tray, two independent straps, separate FPV path and replaceable arms. Reserve an optional small adapter location for future architecture changes without requiring that an FC be fitted now.

## New first milestone

Before final propulsion purchases or final-frame fabrication, demonstrate a **propeller-free Pi 5 native-ArduCopter bench system**:

1. Pin the OS/kernel, native hardware target, ArduPilot version, sensor/I/O revision and Pico firmware/interface versions.
2. Verify real sensor data, orientation, battery measurement and receiver input in the ground station.
3. Measure actual ESC command waveforms/update timing and boot/arming states with a logic analyzer or oscilloscope; verify the chosen ESC protocol independently of AI.
4. Run the intended camera/AI/logging workload for at least the twenty-minute mission duration and a sustained margin; observe scheduling, sensor freshness, power and temperature. A short low-load demonstration is insufficient.
5. With props removed, test AI exit, camera disconnect, receiver loss, ArduPilot stop/hang, Pi reboot/power loss and Pico-link loss. Confirm defined output behavior and that each case is distinguishable in logs.
6. Complete the revised measured mass and rail budgets, then finalize mechanical layout and matched motor/prop/ESC/battery screening.

SITL and bench success are prerequisites, not proof of airborne freestyle capability. No kernel configuration, custom HAL driver, Pico control firmware, production PCB or flight test has been implemented by preparing this document.
