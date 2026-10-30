# DIY Drone

A custom quadcopter project combining mechanical design, electrical integration, flight control, and onboard AI. The goal is a repairable, remotely controlled aircraft with FPV video and room for future sensors and software features.

The current direction is a custom frame with 7-inch propellers and a Raspberry Pi 5 running native ArduCopter alongside separate AI processes. Optional Raspberry Pi Picos may handle defined sensor and I/O tasks.

## Project status

The project is in the design and feasibility stage. This repository currently contains requirements, architecture decisions, and engineering studies. CAD models, schematics, application code, and Pico firmware have not yet been added. Hardware integration, flight-control timing under AI load, and flight performance remain unvalidated.

## Repository structure

```text
DIY-Drone/
├── README.md
├── design/       # Hardware requirements, architecture, and engineering studies
└── software/     # Reserved for application code and embedded firmware
```

## Hardware

| Area | Current design direction |
| --- | --- |
| Airframe | Custom CAD frame; 7-inch propellers and a tentative 330–360 mm opposite-motor diagonal |
| Flight and AI computer | Raspberry Pi 5 with cooling, storage, and dedicated power regulation |
| Sensors and I/O | Supported flight sensors and motor-output hardware; optional Pico modules with explicitly defined interfaces |
| Propulsion | Four motors and compatible ESCs; final selection depends on measured thrust, current, and installed mass |
| Power | Provisional 6S architecture with a correctly implemented 5 V / 5 A-capable Pi supply |
| Batteries | 3300–4000 mAh high-power LiPo for dynamic-flight development; space for 8000–10000 mAh high-power Li-ion endurance testing |
| Radio and video | Remote-control receiver connected through a supported autopilot input path; separate FPV camera and video link |

Mechanical and electrical work includes component placement, vibration isolation, battery retention, cooling, wiring, connectors, power distribution, and sensor/output interfaces. Final dimensions and parts remain open.

Twenty minutes of flight with AI and FPV running during frequent fast maneuvers is a **stretch goal**, pending propulsion and energy measurements.

## Software

The planned software responsibilities are:

- **Flight control:** native ArduCopter on the Pi 5 for state estimation, stabilization, and motor mixing, using a supported Linux hardware target.
- **AI and camera processing:** separate processes for image capture, inference, and project decisions.
- **Telemetry and logging:** local MAVLink communication with source validation, timestamps, and stale-data handling.
- **Embedded I/O:** optional Pico firmware and corresponding ArduPilot drivers for selected sensor or output interfaces.

The receiver input path must work independently of the AI application. A bare Pi still needs supported sensors and reliable motor-output hardware. Because flight control and AI share the Pi, a Pi power loss or reboot also stops stabilization; timing, resource use, and failure behavior are core integration requirements.

There is no runnable project software or installation procedure yet. Implementation will live in [`software/`](software/).

## Design documents

| Document | Purpose |
| --- | --- |
| [Frame requirements](design/FRAME_REQUIREMENTS.md) | Current airframe direction, packaging, and provisional constraints |
| [Pi 5 flight-computer architecture](design/PI5_FLIGHT_COMPUTER_ARCHITECTURE.md) | Computing responsibilities, sensor/output interfaces, scheduling, and failure behavior |
| [Endurance and mounting plan](design/ENDURANCE_AND_MOUNTING_PLAN.md) | Current battery, mass, power, and mounting study |
| [Project plan](design/PROJECT_PLAN.md) | Broader project scope, implementation gates, and historical hardware research |
| [Power and flight feasibility](design/POWER_AND_FLIGHT_FEASIBILITY.md) | Earlier compact-aircraft calculations retained for historical comparison |

Start with the frame requirements, computing architecture, and endurance study. They supersede earlier sizes, budgets, and separate-flight-controller assumptions retained in the historical research.

## Next milestones

1. Select the Pi-compatible sensor, receiver, and motor-output interfaces.
2. Validate native ArduCopter with actual hardware and the intended AI workload on the bench, with propellers removed.
3. Complete the CAD layout, electrical schematic, wiring map, and measured mass/power budgets.
4. Select and validate matched motors, propellers, ESCs, and batteries.
5. Progress through assembly, bench verification, and controlled flight testing with recorded results.
