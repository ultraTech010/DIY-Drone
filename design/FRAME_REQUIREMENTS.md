# Current design: custom Pi 5 AI quadcopter

Updated October 30, 2026. This document supersedes the earlier aircraft-size recommendations in the project plan.

## Confirmed direction

- Use ArduPilot for flight control.
- Design the frame in CAD rather than selecting a completed commercial frame.
- Design the electrical system and its integration.
- The original 7-inch diagonal means **177.8 mm between opposite motor centers**; the user has now authorized sizing changes where necessary.
- Preserve the goals of remote control, maneuverability, low cost and future improvements.
- Raspberry Pi 5 is the intended **flight computer and AI computer**, running native ArduCopter and separate project processes. A separate flight-control computer is no longer assumed.
- Optional Picos may manage sensors and other I/O; their number, interfaces and responsibilities remain subject to change. Stabilization remains on the Pi in this plan.
- Current tentative size: **7-inch propellers**, rather than the original 7-inch motor diagonal; start a 330–360 mm diagonal CAD comparison.
- Desired flight time: **20 minutes with AI and FPV running**; the team selected **frequent fast maneuvers or freestyle**. This is a stretch requirement awaiting measured energy and propulsion feasibility.
- Design for future projects using a **2.2 kg fully loaded case**, with a provisional **2.2–2.5 kgf peak thrust per motor** target. See [movement and upgrade requirements](MOVEMENT_AND_UPGRADE_SPEC.md); no payload or structural rating is established yet.

The current [endurance and mounting study](ENDURANCE_AND_MOUNTING_PLAN.md) governs battery, mass, mounting and upgrade-envelope decisions. It retires the earlier 250 mm / 5-inch / 700 g study. Provisionally use a common 6S electrical architecture, a 3300–4000 mAh high-power LiPo for initial dynamic flight and room for an 8000–10000 mAh high-power Li-ion endurance pack. Neither pack choice establishes twenty-minute freestyle performance.

The electrical-system requirement includes a block diagram, schematic, power and signal interfaces, connectors, wiring and potentially a custom sensor/I/O PCB. The [current Pi 5 flight-computer architecture](PI5_FLIGHT_COMPUTER_ARCHITECTURE.md) governs hardware support, Pico roles, Linux scheduling and failure behavior. The Pi still needs supported flight sensors, battery monitoring and reliable motor outputs; deleting a conventional FC board does not delete those functions.

## Current size study

Start with a **330–360 mm opposite-motor diagonal and 7-inch propellers**. Resolve actual motor locations, battery envelope and computer placement in three dimensions. At a 340 mm square-X diagonal, adjacent motor centers are about 240.4 mm apart, leaving an ideal 62.6 mm gap between neighboring 7-inch propeller disks. This does not establish clearance from the central body or cables. Preserve maneuverability by minimizing installed mass and keeping it close to the center.

The smaller alternatives below are retained as historical geometry comparisons, not the current recommended layout:

| Square X layout candidate | Adjacent motor spacing | Nominal neighboring prop-circle gap | Total rotor disk area |
|---|---:|---:|---:|
| Original 177.8 mm diagonal, 4-inch props | 125.7 mm | 24.1 mm | 0.03243 m² |
| 250 mm diagonal, 5-inch props | 176.8 mm | 49.8 mm | 0.05067 m² |
| 300 mm diagonal, 6-inch props | 212.1 mm | 59.7 mm | 0.07297 m² |

These candidate dimensions establish ideal spacing only. Verify actual board, battery, cable, cooler, arm, motor and propeller clearance in three dimensions before manufacturing. A larger frame can increase rotor area but also increases structural/propulsion mass; compare complete designs.

## Provisional weight and power constraints

Weight and power remain explicit design gates. The current [allocation](ENDURANCE_AND_MOUNTING_PLAN.md#revised-mass-allocation) remains **875 g dry including development reserve**, approximately **1.7 kg gross** with the example large Li-ion packs. The [upgrade requirement](MOVEMENT_AND_UPGRADE_SPEC.md) raises the future fully loaded design case to **2.2 kg**, replacing the former 1.9 kg screening envelope. Keep the provisional **1 kg maximum battery class**. These are targets to validate, not measured masses or established payload ratings. The former 700 g complete-aircraft ceiling is retired.

For electrical planning, use about **40 W average battery-side electronics allocation**, including Pi/ArduCopter/AI camera/cooling, FPV, receiver, sensor/I/O hardware, optional Picos and conversion losses. The study's illustrative allowances span 24–49 W. Measure the simultaneous flight-control/AI workload and its timing. Provision a correctly implemented **5 V/5 A-capable Pi rail** and suitable quiet I/O/sensor regulation; supply capacity and average consumption are different. No power saving from removing an FC is assumed.

Propulsion power is not validated. At 1.73 kg gross, level hover requires about **433 gf per motor**, increasing to **550 gf at 2.2 kg**. Target **2.2–2.5 kgf available peak thrust per motor**, or **8.8–10 kgf total**, to obtain approximately **4–4.5:1 at the future mass**. This replaces the earlier 3:1 screening target. Validate matched propulsion and installed power delivery under declared voltage, state-of-charge, duration and thermal conditions. Investigate **65–80 A-per-channel ESCs**, but select ratings and the shared input path from actual demand and cooling. Static thrust alone does not establish agility, endurance or structural capacity.

## Original compact geometry — reference case

Use a square X motor arrangement for initial calculations. A different shape requires checking every actual motor-to-motor spacing and the chosen control geometry.

For diagonal D = 177.8 mm:

```text
Adjacent motor spacing = D / sqrt(2) = 125.724 mm
Motor coordinate magnitude = D / (2 × sqrt(2)) = 62.862 mm
```

Place the origin at the center of the square. An initial top-view CAD sketch uses these centers:

| Motor location | X (mm) | Y (mm) |
|---|---:|---:|
| Front right | +62.862 | +62.862 |
| Front left | -62.862 | +62.862 |
| Rear left | -62.862 | -62.862 |
| Rear right | +62.862 | -62.862 |

Here X points right and Y points forward for this CAD sketch. These names and coordinates are **not** an ArduPilot motor-output assignment or flight-coordinate convention. Determine output mapping and motor direction from the selected firmware/frame configuration later.

Draw a circle centered on each motor to represent its propeller's swept area.

| Propeller diameter | Diameter (mm) | Ideal adjacent tip-to-tip gap (mm) |
|---|---:|---:|
| 3.5 inches | 88.9 | 36.8 |
| 4.0 inches | 101.6 | 24.1 |
| 4.5 inches | 114.3 | 11.4 |
| 5.0 inches | 127.0 | -1.3: overlap |
| 7.0 inches | 177.8 | -52.1: overlap |

These are nominal mathematical gaps for adjacent coplanar circles. They exclude blade flex, tolerances, damaged blades and any central body/arm obstruction. They do not establish flight suitability.

**Initial propeller candidates: 3.5 and 4 inches.** Compare these in the layout and mass study. A 4.5-inch propeller is geometrically possible in the ideal square, but has smaller clearance and still requires appropriate structure/propulsion testing. Do not order motors based only on circle clearance.

With 4-inch props, the top-view axis-aligned outer envelope is approximately 227.3 × 227.3 mm before other protrusions. A 177.8 mm motor diagonal does not mean that the aircraft, including propellers, fits inside a 177.8 mm square.

## Raspberry Pi 5 packaging study

Use the [official Pi 5 mechanical drawing](https://pip.raspberrypi.com/documents/RP-008347-DS-1-raspberry-pi-5-mechanical-drawing.pdf) as a reference, then verify against the actual board. Its PCB outline is approximately 85 × 56 mm; connector protrusions, cooler, wires and restraint require additional space. The manufacturer labels drawing dimensions approximate and unsuitable as production data without verification.

Model these separately:

1. Pi board and protruding components/connectors.
2. Cooler, fan airflow and service access.
3. Camera, ribbon bend radius and camera field of view.
4. Primary flight IMU, sensor/output interface and optional Pico modules, with appropriate isolation and programming access. If the IMU is on a Pi expansion board, assess the whole Pi/sensor assembly's mounting.
5. ESC, heat dissipation, power leads and capacitor.
6. Battery with retention and connector clearance.
7. Receiver, antenna and any GPS/compass.
8. Project regulator, wiring and custom board.

Test placement and rotation in top view and verify elevations in side view. Where a component overlaps a propeller's projection, determine whether the actual 3D arrangement provides adequate clearance throughout blade motion and frame deflection. Do not infer clearance from a top view alone. Keep wires and mounts outside the actual swept volume.

Keep the center of mass near the center of the rotor layout. Put heavy components close to the center, and leave access for battery replacement. A removable computer module can support later experiments, but it cannot provide unlimited payload capacity on this compact frame.

The Pi is a major packaging decision in the compact reference case. Confirm an actual mass budget including cooler, camera, regulator, storage, mounts and wiring before choosing battery/motors. Sizing changes are now authorized; document the chosen dimensions and consequences rather than carrying the original 177.8 mm assumption into the new design.

See the [first-pass power and flight feasibility study](POWER_AND_FLIGHT_FEASIBILITY.md) for an illustrative installed-mass budget, thrust screening requirements, energy calculations and the limits of those assumptions. These are not measured aircraft performance figures.

## Preliminary electrical architecture

### Pi flight computer with supported sensor/I/O hardware

The previous Kakute H7 Mini recommendation is superseded by the team's Pi-based direction. ArduPilot has [Linux autopilot support](https://ardupilot.org/copter/docs/common-autopilots.html#linux-based-autopilots) and [merged Pi 5 support](https://github.com/ArduPilot/ardupilot/pull/27026); exact sensors, receiver and outputs still require matching drivers. Select/evaluate a supported expansion board or a custom interface with Pico(s), following the [architecture study](PI5_FLIGHT_COMPUTER_ARCHITECTURE.md). No final I/O board is selected.

Reserve interfaces for the primary IMU, RC receiver, GPS, battery monitoring, output hardware and future sensors. The AI/ArduCopter MAVLink connection can be local on the Pi instead of consuming a Pi-to-FC UART. Verify Linux-device mappings and Pi 5 electrical/bus compatibility. Do not assume DShot support from the earlier MCU-based setup.

```text
Battery
  +-- appropriately sized power path --> four ESC channels --> four motors
  +-- quiet I/O/sensor supply --> receiver + sensor/I/O hardware + optional Picos
  +-- suitable regulated Pi supply --> Pi 5 running ArduCopter + AI + camera

Remote --> receiver --> supported input path --> ArduCopter process on Pi
AI process <--> local MAVLink <--> ArduCopter process on the same Pi
ArduCopter --> supported output hardware / optional I/O Pico --> ESCs
```

ArduCopter on the Pi handles estimation and stabilization; separate Pi processes handle images/project logic. A Pi power loss or reboot now stops both. The [Pi 5 specifications](https://www.raspberrypi.com/products/raspberry-pi-5/) recommend suitable 5 V/5 A power and sustained-work cooling. Verify delivery, startup peaks, voltage drop, heat and the connection method. Independently define/test output behavior on stale commands; an I/O Pico watchdog is not a backup stabilizer.

Custom electrical-design outputs should include:

- Block diagram identifying every power rail and device.
- Load budget: continuous and peak current on each rail.
- Battery full-charge voltage and transient allowance for connected components.
- Regulator selection, schematic, layout guidance and thermal assessment.
- Pin-to-pin connector table with power, ground and signal voltage levels.
- Pi/sensor/Pico/output/ESC signal mapping and current/voltage measurement interfaces.
- Native ArduPilot hardware target and driver support for every flight-critical interface.
- Pico protocol/firmware, timing, boot/arming and command-timeout definitions where used.
- Power-distribution and return-current routing sized for actual motor demand.
- Test points and a power-up procedure with appropriate current limiting.
- Mounting and electrical insulation from conductive frame material, if used.
- Access for programming/debugging and replacing experimental components.

A custom power/interconnect PCB can connect the Pi, sensors, Pico(s) and purchased ESC module. Any new sensor/output interface needs compatible Linux ArduPilot drivers or explicit development of those drivers. Pico waveform generation and a working Pi application alone do not establish native autopilot integration. Record the software/backend requirements alongside the schematic.

## What is not yet selected

Propeller model/pitch, motor size/KV, exact battery, ESC current rating/protocol, frame material/thickness, regulator circuitry, sensor/I/O hardware, Pico roles, Linux configuration and flight-ready mass remain open. Six series cells and 7-inch props are provisional choices. The previous 1700 KV motor suggestion is **not** a validated propulsion specification.

No flight-time, payload or high-maneuverability claim is established by these dimensions. After estimating mass, select candidate propulsion using compatible manufacturer thrust/current data, then validate the complete design in stages.

## First design milestone

Produce a geometry and mass feasibility package before manufacturing structural or power electronics:

First establish the [propeller-free native Pi flight-control bench milestone](PI5_FLIGHT_COMPUTER_ARCHITECTURE.md#new-first-milestone), including real sensors, receiver, outputs and timing under AI load. The CAD package then reflects a demonstrated interface architecture.

1. CAD skeletons around 330–360 mm diagonal with 7-inch props; retain smaller layouts only as historical comparisons. Include the approximately 180 × 75 mm battery-tray allowance, adjustable pack height and separate cable/connector space from the current mounting study.
2. Pi, battery and electronics keep-out envelopes in top/side views.
3. Mass budget with measured/catalog masses and an allowance for wiring/mounts.
4. Electrical block diagram and initial rail/load budget.
5. Comparison of matched propulsion candidates at the initial gross mass and the **2.2 kg future design case**, including frame, motor, battery and power-path changes. Check maneuver/peak motor loads and retention against that case before treating the frame as upgrade-ready.

This is a requirements and feasibility document, not production CAD, a completed circuit schematic, or a validated bill of materials. Its geometry calculations have been checked; hardware fit, structural performance and electrical operation have not yet been tested.
