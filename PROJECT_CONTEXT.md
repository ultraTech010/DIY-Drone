# User-stated project context

Updated October 30, 2026. This file preserves the user's goals and current preferences. It is not a design, a bill of materials, a calculated specification or a project plan.

## Learning and ownership

- This is a team learning project. The user has basic C experience and is learning the other subjects involved.
- The user and their team will build the drone and create the project plan themselves. The assistant teaches concepts, guides calculations, helps develop ideas and performs focused research when asked.
- Apply the teaching instructions in [AGENTS.md](AGENTS.md) only within this project.

## Goals and preferences

- Build a DIY quadcopter with remote control, FPV and high maneuverability.
- Support onboard AI tasks and leave room for future projects, sensors and other improvements.
- Keep the budget low. Explain worthwhile higher-cost possibilities so the team can decide; no exact spending limit has been set.
- Weight and power consumption are important constraints.
- The current desired flight duration is **20 minutes**, with the selected flying style of **frequent fast maneuvers or freestyle**. This is a goal, not an achieved or validated capability.
- Allow battery-capacity upgrades without major frame changes where practical. The team has requested substantial upgrade headroom; the amount must be investigated and decided by the team.

## Current technical direction, subject to change

- The team will CAD/design the frame and electrical system themselves.
- Use **ArduPilot / ArduCopter**.
- Use a **Raspberry Pi 5** for onboard AI and probably also flight control, rather than assuming a separate flight-controller computer. Exact hardware/software support has not been established by the team's build.
- Some **Raspberry Pi Picos** may manage sensors and other pieces. Their number and roles are explicitly subject to change.
- The team is currently considering a **7-inch FPV drone**, with sizing flexible. Earlier in the conversation, “7 inches” meant the opposite-motor diagonal; the later FPV discussion used it as the tentative propeller-size class. Clarify which dimension is intended when the distinction affects a calculation or drawing.
- The remote controller remains part of the intended system.

## What is not established

- There is no finalized motor, propeller, ESC, battery capacity/cell count, sensor interface, frame geometry, mass limit, thrust target, speed or payload rating recorded as a team-derived specification here.
- Previous detailed numerical targets and component selections were assistant proposals. The user requested deletion of those plans; they are not retained as confirmed requirements.
- No hardware integration, control timing under AI load, power budget, structural capacity or flight performance has been validated through this repository.

## Repository

- GitHub: [ultraTech010/DIY-Drone](https://github.com/ultraTech010/DIY-Drone).
- The current local checkout is `/Users/avibindlish/DIY-Drone`; paths may change.
- Future plans, calculations and implementation should reflect work and decisions owned by the user and their team.
