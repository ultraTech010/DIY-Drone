# Power and flight feasibility: custom Pi 5 quadcopter

October 30, 2026. This is a first-pass engineering study with explicit assumptions. It is not a measured performance result or validated motor selection.

**Current sizing/computing update:** The team proposes **7-inch propellers**, a twenty-minute target with AI/FPV running and frequent fast maneuvers/freestyle. **Pi 5 will run native ArduCopter and AI**, probably without another flight-control computer; optional Picos may manage sensors/I/O, subject to change. Use the [current architecture](PI5_FLIGHT_COMPUTER_ARCHITECTURE.md), [endurance/mounting study](ENDURANCE_AND_MOUNTING_PLAN.md) and [requirements](FRAME_REQUIREMENTS.md). They set a provisional **875 g dry allocation including reserve**, approximately 1.7 kg gross with example large packs, and a 330–360 mm diagonal layout study. No dimensions/performance are validated. Sensor/output hardware remains necessary, so no FC-removal power saving is assumed.

**Upgrade target:** The [movement and upgrade requirements](MOVEMENT_AND_UPGRADE_SPEC.md) now govern thrust and future loaded mass: **2.2 kg future design case**, **2.2–2.5 kgf available peak per motor**, approximately **4–4.5:1 at that mass**. This replaces earlier 1.9 kg / 3:1 screening; the compact-aircraft examples below remain historical teaching calculations. Initial dry mass and battery-energy allocations are unchanged until actual component selection and measurement.

**Reading this older study:** The arithmetic below is retained for teaching and historical comparison. Its 177.8/250/300 mm layouts, 500–700 g mass cases, 4S 1500 mAh example and 20–25 W electronics allowances do not describe the current aircraft. The new study provisionally uses 6S, about 40 W average electronics allocation including FPV, separate 5 V/5 A-capable Pi power, and swappable dynamic-flight/endurance pack types. Twenty-minute freestyle is not established by either study. No propulsion average-power figure has yet been measured.

## Historical compact-aircraft assessment

A Pi 5, battery, flight controller, additional small microcontrollers and modest sensors are not inherently too heavy or power-hungry for a quadcopter. On this particular 177.8 mm diagonal aircraft, feasibility depends primarily on the complete installed mass, actual 3.5–4-inch propulsion performance, and how the electronics fit without obstructing the rotors. Flight capability and high maneuverability are separate requirements.

That earlier compact direction was physically plausible as an experiment, but was not demonstrated. Keeping a full Pi 5, low cost, high maneuverability and substantial future payload capacity simultaneously requires compromises. No selected motor/propeller test established the necessary thrust/current for that layout.

## Historical smaller-layout comparison

| Candidate | Total disk area | Ideal induced hover-power ratio at identical total mass |
|---|---:|---:|
| Original 177.8 mm diagonal, four 4-inch props | 0.03243 m² | 1.00 |
| 250 mm diagonal, four 5-inch props | 0.05067 m² | 0.80 |
| 300 mm diagonal, four 6-inch props | 0.07297 m² | 0.67 |

The last column follows ideal momentum theory with identical mass, air density and non-overlapping rotors; it is not total measured electrical power or a flight-time prediction. For example, if the move from a 4-inch to a 5-inch system also increases gross mass from 600 g to 650 g, the ideal induced-power ratio becomes `(650/600)^1.5 × 4/5 = 0.902`, approximately 10% lower rather than 20% lower.

The earlier study preferred 250 mm/5-inch and considered 300 mm/6-inch for packaging margin, under a provisional 700 g gross ceiling. These are now historical comparisons, not the current sizing recommendation or mass constraint.

## Compact reference case: geometry constrains rotor area

For a square X motor layout, the 177.8 mm diagonal gives 125.724 mm between adjacent motor centers. Four-inch propellers are 101.6 mm across, leaving an ideal 24.124 mm tip-circle gap between neighbors. Five-inch propellers overlap, so earlier large-prop recommendations do not apply.

Four non-overlapping 4-inch disks provide approximately 0.03243 m² total rotor area. Four 3.5-inch disks provide approximately 0.02483 m². For the same weight, the larger total rotor area lowers the ideal induced power needed to hover. Actual efficiency also depends on the blade, motor, operating point and installation.

## Illustrative mass budget

These entries are design allowances, not verified catalog weights or a completed bill of materials. Replace them with measured weights before selecting propulsion.

| Installed item | Assumed mass |
|---|---:|
| Frame and structural hardware | 60 g |
| Four motors | 80 g |
| Four propellers | 8 g |
| Four-channel ESC hardware | 15 g |
| Pi 5, cooler and storage | 80 g |
| Camera and ribbon cable | 8 g |
| Flight controller, receiver, small MCU/sensor additions | 24 g |
| Custom power board, regulators and wiring | 35 g |
| Additional mounts and retention | 20 g |
| Example flight battery | 170 g |
| **Illustrative subtotal** | **500 g** |
| Additional design/growth contingency | 100 g |
| **Gross-mass screening case** | **600 g** |

The contingency is an allowance, not a physical component. It helps prevent a design that only works if every early estimate is exact. Extra sensor modules, an AI accelerator, large connectors, heavier structural material or a protective enclosure can consume it quickly.

Do not count only bare chips: installed boards also contain connectors, regulators, wire and mounting hardware. A few small MCU/sensor boards generally deserve less attention than the battery, complete computer installation, motors and frame, but their actual power and mass still need to be recorded. Radios, lidar, heaters and high-power peripherals are not covered by a blanket assumption that all sensors are negligible.

## Thrust requirement

In level stationary hover, the total upward thrust approximately equals weight. Expressing thrust in gram-force for convenient comparison with hobby motor data:

```text
600 g gross aircraft -> approximately 600 gf total hover thrust
600 gf / 4 motors -> approximately 150 gf per motor in level hover
```

For a preliminary 3:1 maximum-thrust-to-weight design target:

```text
600 g × 3 = 1800 gf total maximum thrust
1800 gf / 4 = 450 gf maximum thrust per motor
```

Three-to-one is a design target for useful control margin, not a universal certification threshold or guarantee of agility. Verify current, temperatures, voltage sag, motor response and control margin at the chosen propeller and battery, including a partially discharged pack. A short maximum-thrust result does not establish continuous thermal capability. For a 700 g version, the same screening target rises to 525 gf per motor.

Avoid choosing a motor from a 5-inch-propeller test and assuming it produces the same thrust on a 4-inch propeller. Obtain data for the actual propeller diameter, pitch, blade count, voltage and motor winding, then verify the installed system.

## Electrical power budget

The Pi's supply capacity is different from its measured consumption. Raspberry Pi's [launch engineering description](https://www.raspberrypi.com/news/introducing-raspberry-pi-5/) reports around 12 W under its intensive test conditions. That number is not a guarantee for your full AI/camera/peripheral setup. The [current product specification](https://www.raspberrypi.com/products/raspberry-pi-5/) recommends a suitable 5 V/5 A supply. Measure the actual configuration, including startup and transient loads.

For an initial workload study, use these **provisional allocations**:

| Load | Illustrative allowance |
|---|---:|
| Pi 5, camera and cooling during the intended workload | 12–20 W |
| Flight electronics, radio, modest extra MCU/sensors | 1–4 W |
| Regulator losses, depending on load and efficiency | 1–3 W |

These are not published measured ranges for this aircraft. In the simple energy example below, assume **20 W total battery-side electronics power**, including conversion losses, and independently assume **200 W average battery-side propulsion power**. Those assumptions give 220 W total. The motor power is not derived from the mass budget; it is a scenario to show the calculation until a matched propulsion test is available.

At a nominal 4S voltage of 14.8 V, 220 W corresponds to approximately 14.9 A average battery current. At 14.0 V it is approximately 15.7 A for the same power. Maneuver peaks can be much higher; size the battery, ESCs, wiring and connectors using the actual peak requirements, not this average.

Use a high-current motor power branch and dedicated regulated electronics branches. The Pi should have an appropriately designed regulator separate from the FC's small peripheral supply. Both regulators still share the flight battery, so this alone does not create full power redundancy.

For a regulator delivering 15 W at 90% efficiency:

```text
Battery-side power = 15 / 0.90 = 16.67 W
Heat lost in regulator = 16.67 - 15 = 1.67 W
```

Check full-charge battery voltage, transient headroom, ripple, connector/cable voltage drop and cooling. The [Pi 5 cooling guidance](https://www.raspberrypi.com/news/heating-and-cooling-raspberry-pi-5/) explains thermal throttling under sustained workloads. Cooling must also work during bench operation and low-speed flight, not only in forward airflow.

## Battery energy and duration

An illustrative 4S 1500 mAh pack has nominal energy:

```text
14.8 V × 1.5 Ah = 22.2 Wh
Assuming 80% usable for this example: 17.76 Wh
Arithmetic duration at 220 W: 60 × 17.76 / 220 = 4.84 minutes
```

The 80% factor is a modeling assumption, not a sufficient landing rule. Actual limits depend on cell voltage under load, battery health, current and the configured reserve policy.

Sensitivity to assumed **total battery-side power**, holding nominal usable energy fixed:

| Total average power | Arithmetic duration |
|---|---:|
| 160 W | 6.66 minutes |
| 220 W | 4.84 minutes |
| 300 W | 3.55 minutes |

These are energy arithmetic scenarios, not predicted or promised flight times. Actual propulsion data, battery sag and flight measurements are still missing. Never substitute the Pi's 25 W supply rating for the whole aircraft's power budget.

Increasing battery capacity adds energy but also mass. Recalculate thrust and propulsion power when changing packs. Doubling capacity does not generally double flight duration on an otherwise similar airframe.

## Why installed mass can matter more than compute power

Ideal momentum theory relates hover induced power to thrust and rotor area:

```text
P_ideal = T^(3/2) / sqrt(2 × air_density × total_rotor_area)
```

For an ideal equal-thrust, non-overlapping rotor system, with weight/air density/area held consistently, this gives an induced-power scaling proportional to mass^(3/2). See [NASA's rotor hover performance treatment](https://ntrs.nasa.gov/api/citations/20220015637/downloads/NASA-TM-20220015637final.pdf) and the [Penn State multirotor sizing derivation](https://www.aero.psu.edu/avia/pubs/Volpe_MS.pdf).

Adding a 100 g installed module to a hypothetical 500 g aircraft increases mass to 600 g. Its ideal induced hover-power component rises by:

```text
(600 / 500)^1.5 = 1.315, approximately a 31% increase
```

This is **not** a prediction that total measured electrical power rises by exactly 31%. It excludes blade profile losses, motor/ESC losses and installation effects. It explains why a computer consuming a modest number of watts can still impose a substantial flight cost through its mass.

## Aerodynamic and mechanical layout

At hover and low speeds, weight, rotor area and installation interference are central. In faster flight, the body, wiring, cooler and mounts also produce drag. A streamlined enclosure may reduce drag but can add weight and trap heat; test the complete design rather than assuming an enclosure is an improvement.

Keep the Pi, battery and other heavy parts near the rotor layout's center. Avoid heavy components far along arms or on tall towers: this increases rotational inertia and can reduce responsiveness. Place the center of mass near the intended thrust center. Frame stiffness, vibration control and adequate motor response also matter for maneuverability.

Keep propeller swept volumes free of wires and components. Wide body plates near rotor flow can cause interference; projected overlap alone is not a quantitative loss estimate. Analyze the full 3D arrangement and measure installation effects. Do not remove necessary structural members simply to increase open area.

The Pi 5 outline is approximately 85 × 56 mm, but a useful CAD envelope includes connectors, cooler, cable bends, restraints and service clearance. Board rotation and vertical placement can change whether it conflicts with the rotors. A large protective box or full-size stacked AI board can make this compact layout much harder to package.

## Evidence needed before calling the aircraft viable

1. Weigh or obtain reliable installed masses and build a conservative gross-mass estimate.
2. Complete top/side CAD checks for battery, Pi, cooling and rotor clearance.
3. Obtain matched motor/propeller/battery thrust and current data for the candidate configuration.
4. Verify useful thrust margin at representative battery voltage and acceptable motor/ESC temperatures.
5. Test Pi power stability with the actual AI workload and peripherals, including startup and shared-battery transients.
6. Test structure, retention, vibration and manual flight progressively before enabling experimental flight commands.

A 500–600 g screening range is useful for the compact reference calculation, not an approved target envelope. Compare the complete larger layouts using actual installed masses and matched propulsion data. The original 177.8 mm dimension is now flexible, and the selected geometry will be documented before manufacturing.
