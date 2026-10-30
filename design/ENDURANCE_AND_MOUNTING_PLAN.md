# Seven-inch FPV / Pi 5: battery and mounting study

Updated October 30, 2026. This is the current sizing study. It supersedes the previous 250 mm / 5-inch / 700 g candidate. It is not production CAD or a tested flight-time prediction.

**Computing update:** The team now plans to run native ArduCopter and AI on the **same Pi 5**, probably without a separate flight-control computer. Optional Picos may manage sensors and I/O; their roles remain open. The [Pi flight-computer architecture](PI5_FLIGHT_COMPUTER_ARCHITECTURE.md) governs this change, including hardware support, scheduling, sensor interfaces and failure behavior. A bare Pi does not replace the sensor/output functions of an FC board.

## Requirements and the main tradeoff

- Native ArduCopter and AI run as separate processes on Pi 5; optional Picos handle defined peripheral/sensor/output tasks. No separate flight-control computer is assumed.
- Custom frame and electrical integration, remote control, FPV, low cost and upgrade space.
- Tentatively use **7-inch propellers**, the usual meaning of a “7-inch FPV drone.” This is different from the original 7-inch motor diagonal. Confirm this interpretation before manufacturing.
- Desired endurance: **20 minutes**, with AI and FPV running.
- The team explicitly selected **frequent fast maneuvers or freestyle** as the flight style.

Twenty minutes of frequent aggressive maneuvers is a stretch requirement for a low-cost 7-inch aircraft carrying a Pi. Do not promise it from a battery label. More capacity can increase mass, motor demand and inertia. A sensible first aircraft accepts two interchangeable pack types: a high-power LiPo for dynamic flying and a larger high-power Li-ion pack for endurance experiments. Twenty-minute smooth-flight performance would not validate twenty-minute freestyle performance.

For context, [GEPRC's MOZ7 V2](https://geprc.com/product/geprc-moz7-v2-analog-long-range-fpv-drone/) advertises 25 minutes **based on slow cruising tests** and recommends a 6S 3300 mAh LiPo or 8000 mAh Li-ion. It has 7.5-inch propellers and is not our custom Pi-equipped aircraft. [iFlight's battery guidance](https://iflighteurope.com/products/chimera7-pro-v2-o4-gps) likewise favors LiPo for dynamic flying and 6S2P Li-ion for endurance cruising. These establish useful design comparisons, not a performance guarantee.

## Battery recommendation

Use **6S as the provisional common voltage**: 22.2 V nominal and 25.2 V fully charged for the conventional 4.2 V/cell packs considered here. All connected equipment must tolerate its actual input rail, including full charge and transients. Keep cell count constant when changing capacity; changing voltage is a propulsion/electrical redesign decision.

1. **Initial flight/tuning pack: 6S 3300–4000 mAh high-power LiPo.** This prioritizes throttle response and the team's maneuverability requirement. It is not a twenty-minute freestyle promise. Select the exact pack from verified current capability, mass, dimensions, connector and charger compatibility.
2. **Endurance test pack: 6S2P 8000–10000 mAh high-power Li-ion.** Reserve room for this pack class in CAD from the beginning. Use it to investigate the twenty-minute target after measuring the complete aircraft. Do not assume an endurance pack is suitable for repeated maximum-power maneuvers.
3. **6S1P 5000 mAh Li-ion is a lighter alternative,** but its single parallel string and pack-specific discharge limits must be checked. It is not the preferred default for this freestyle requirement. A high-energy laptop-style battery is not an appropriate substitute for a verified high-current flight pack.

“6S2P” means two parallel strings of six series-connected cells: twelve cells total. Series determines voltage; parallel strings increase charge capacity and can increase available current. Actual current limits depend on the cells, interconnects, leads, connectors, temperature and state of charge, not just this notation or an advertised C rating.

## Capacity calculation and its limits

Use energy, not mAh alone. For a preliminary calculation:

```text
nominal energy (Wh) = nominal pack voltage × capacity (Ah)
estimated flight hours = nominal energy × usable fraction / average battery-side power
required capacity (Ah) = average power × flight hours / (nominal voltage × usable fraction)
```

Here use 22.2 V and an **assumed usable fraction of 0.80**, reserving 20% of nominal energy for planning. This is not a measured discharge curve or a universal safe landing threshold. Real usable energy depends on load, voltage sag, temperature, cell condition and the chosen landing reserve. Average power includes motors, ESC losses, Pi, FPV, sensors and regulator losses.

| 6S capacity | Nominal energy | Maximum average total power for 20 minutes, at assumed 80% usable energy |
|---|---:|---:|
| 3300 mAh | 73.3 Wh | 176 W |
| 4000 mAh | 88.8 Wh | 213 W |
| 5000 mAh | 111.0 Wh | 266 W |
| 6000 mAh | 133.2 Wh | 320 W |
| 8000 mAh | 177.6 Wh | 426 W |
| 10000 mAh | 222.0 Wh | 533 W |

These are **power ceilings for an energy calculation**, not predicted aircraft power or guaranteed durations. Each pack changes aircraft mass, so its actual average power must be determined separately.

For illustration, a measured 500 W average would require about **9400 mAh** for twenty minutes under these assumptions; 800 W would require about **15000 mAh**. These powers are scenarios, not claims about typical freestyle consumption. A 10000 mAh pack at 800 W would provide only about **13.3 minutes** in this simplified calculation. Installing a still larger pack can itself raise the power demand, so repeatedly increasing capacity is not a complete solution.

Capacity and maximum power are separate checks. A pack may have enough Wh but sag or overheat during a throttle burst. Select motors/props from matched bench data, then check the whole battery-to-motor power path at peak demand and low state of charge. Do not multiply a marketing C rating by capacity and treat that result as a verified aircraft current limit.

## What the Pi contributes

The Pi matters, but propulsion is likely to dominate total flight power. For an initial electronics allocation, use:

| Electronics group | Provisional average allowance |
|---|---:|
| Pi 5 running ArduCopter and AI, AI camera and cooler | 15–25 W |
| Separate FPV camera and video transmitter | 5–15 W |
| Sensor/I/O hardware, optional Picos, receiver, GPS and modest sensors | 2–5 W |
| DC conversion losses | 2–4 W |

The combined range is 24–49 W; use about **40 W as a first allocation** and measure the actual simultaneous flight-control/AI configuration. These are engineering allowances, not manufacturer measurements. Removing the separate FC does not remove the sensor/I/O hardware or establish a power saving. An accelerator, lidar or other substantial payload requires a new allowance. The video transmitter's advertised RF output power is not its DC input consumption.

A Pi subsystem drawing an assumed 20 W uses **6.7 Wh over twenty minutes**, equivalent to approximately **375 mAh of additional nominal 6S capacity** under the same 80% usable-energy assumption. Its installed weight also makes the motors work harder; this calculation accounts only for its direct electrical draw.

Provide a dedicated, appropriately cooled **5 V / 5 A-capable supply path** for the Pi, following the [Pi 5 power specification](https://www.raspberrypi.com/products/raspberry-pi-5/) and the selected connection method. Five-amp capability does not mean constant 25 W consumption. USB-C current negotiation/advertisement, cable drop and startup behavior must be implemented correctly; an arbitrary USB-C connector does not establish a suitable supply.

## Real battery examples for CAD

Manufacturer specifications checked October 30, 2026. Dimensions below are normalized to length × width × height; verify against the actual delivered pack and lead exits.

| Example | Chemistry/configuration | Nominal energy | Listed mass | Listed pack body, L × W × H |
|---|---|---:|---:|---:|
| [Tattu 4000 mAh](https://www.tattuworld.com/products/tattu-4000mah-6s1p-22-2v-95c-fpv-lipo-battery-with-xt60-plug.html) | 6S LiPo | 88.8 Wh | 575 g | 164 × 47 × 38.5 mm |
| [Fullsend 5000 mAh](https://shop.iflight.com/Fullsend-6S-5000mAh-Li-Ion-Battery-Pro2362) | 6S1P Li-ion | 111 Wh | 436 ± 5 g | 75 × 63 × 43 mm |
| [Fullsend 8000 mAh](https://shop.iflight.com/Fullsend-6S-8000mAh-Li-Ion-Battery-Pro1914) | 6S2P Li-ion | 177.6 Wh | 840 ± 10 g | 147 × 64 × 42 mm |
| [Fullsend 10000 mAh](https://shop.iflight.com/Fullsend-6S-10000mAh-Li-Ion-Battery-Pro2363) | 6S2P Li-ion | 222 Wh | 855 ± 10 g | 148 × 63 × 43 mm |

The two large Li-ion examples illustrate an upgrade with nearly unchanged packaging: the larger capacity uses different-capacity cells, rather than adding another parallel string. Their listed masses differ by only 15 g. This is manufacturer data, not independent capacity/current testing, and is not a general rule that larger-capacity packs weigh the same. Check the actual cell revision and usable energy at your load. At checking, the 8000 mAh page listed $83.99; the 10000 mAh page listed $95.99 and was sold out. Tax, shipping and availability vary.

## Revised mass allocation

The former 700 g complete-aircraft ceiling is retired. A listed 8000 mAh battery alone weighs about 840 g.

| Installed group, excluding flight battery | Initial allocation |
|---|---:|
| Frame, fasteners, battery retention, project mounts | 230 g |
| Four motors, props and ESC | 270 g |
| Pi 5, cooler, AI camera, storage and computer cables | 115 g |
| FPV camera, transmitter and antennas | 50 g |
| Sensor/output interface, optional Picos, receiver, GPS and modest sensors | 60 g |
| Regulators, power distribution and remaining wire | 70 g |
| Unallocated development reserve | 80 g |
| **Dry design allocation including reserve** | **875 g** |

This is a target to reconcile with real components, not a completed BOM. The Pi-based sensor/I/O/Pico allowance replaces the earlier 35 g FC group; it does not claim a measured 25 g increase. A heavier frame, larger motor, enclosure or peripheral can exceed it. With the listed examples, design-budget gross masses would be approximately **1.450 kg with the 4000 mAh LiPo**, **1.715 kg with the 8000 mAh Li-ion**, or **1.730 kg with the 10000 mAh Li-ion**. The reserve is budget headroom, not a physical installed part.

Use approximately **1 kg maximum battery mass / 1.9 kg gross mass as the initial structural and propulsion screening envelope**, subject to engineering validation. This is not an established payload rating. At 1.73 kg gross mass, each motor supplies about 433 gf in level hover. An illustrative 3:1 total-thrust screening target requires about 1300 gf per motor, rising to 1425 gf at 1.9 kg; freestyle may need more margin and good response. Verify the chosen motor/prop/battery combination rather than treating this ratio as proof of agility. Check maneuver loads, arm stiffness, fasteners and battery restraint at the maximum pack, not just static weight.

## Frame and mounting layout

Start the CAD comparison around **330–360 mm opposite-motor diagonal with 7-inch props**. Final dimensions depend on all equipment envelopes and structural choices. A symmetric X is a useful initial geometry for maneuvers; change to a camera-clearance layout only after comparing its rotor positions, mass balance and control configuration.

For a square X with 340 mm diagonal, adjacent centers are approximately 240.4 mm apart. Seven-inch propellers are 177.8 mm in diameter, giving an ideal adjacent disk gap of 62.6 mm. This says nothing by itself about central electronics clearance: model swept blades, flex, wiring and elevations in three dimensions. Larger props in the future may need new arms/motors/ESCs even if the center body is retained.

Use a long central body and replaceable arms. Put heavy items close to the rotor center, keep the battery close to the motor plane, and avoid a tall stack of battery + Pi + accelerator. Keep mounts/boards out of the swept volume and minimize obstruction of rotor inflow. Reserve airflow and service access before adding a cosmetic shell.

Conceptual side view — positions and deck heights must be solved in CAD:

```text
FRONT                                               REAR
          [sliding battery, lengthwise, two straps]
      ========= structural battery deck =========
 FPV cam   [Pi/cooler + flight IMU/I/O]       [VTX]
 AI cam    [optional Pico modules] [ESC]      antenna
      ========= stiff lower frame ===============
                 motor/arm plane nearby
```

This is not a requirement to stack the Pi beneath the battery. Solve Pi/cooler, flight IMU/I/O, optional Picos and battery placement together near the rotor center, keeping mass compact and balanced. Keep ground/impact clearance for all boards. There is no separate FC location required by the current plan.

| Component | Mounting requirement |
|---|---|
| Battery | Removable top-center tray with fore/aft adjustment, non-slip pad, two independent straps, protected lead exits and non-sharp locating stops. Provide strap positions that retain both short and long packs. |
| Pi 5 flight computer | Removable central module near the center of mass, on electrically insulated mounts. Provide cooler airflow, impact protection and strain relief. If its expansion board carries the primary IMU, assess vibration isolation for the complete assembly, not the Pi board alone. |
| Primary flight IMU / sensor-I/O module | Stable known orientation near the center, short buses, appropriate vibration isolation and protection from electrical noise. A separate IMU mount and a complete Pi/IMU module require different isolation designs. |
| Optional Picos | Output controller near ESC signals, peripheral controllers near their sensors where useful; retained wiring and access for programming. Their number/roles remain open. |
| ESC / power board | Near the center to limit high-current cable length; provide electrical insulation, thermal space and capacitor retention. Keep heat, noise and mechanical contact away from the flight IMU/module. |
| FPV camera | Protected front bracket with adjustable viewing angle. Keep the pilot's low-latency video path independent of the experimental AI program. |
| AI camera | Rigid, replaceable bracket with an unobstructed task-appropriate view. Recalibrate its relationship to the aircraft after changing placement. Sharing an FPV camera with the Pi needs a verified video interface, not just a spare wire. |
| Video transmitter | Ventilated rear location, with antenna/cables retained outside the propeller volume. Ground testing also needs its specified cooling. |
| GPS / compass | Sky view and separation from motor/power wiring and transmitting antennas; inspect compass interference under power. Use a short stiff mount, not a tall fragile tower. |
| RC antennas | Follow the receiver's specified orientation and clearances; keep active elements away from conductive carbon, battery shielding and props. |

ArduPilot recommends locating the flight controller near the center of gravity and addressing vibration: see [mounting guidance](https://ardupilot.org/copter/docs/common-mounting-the-flight-controller.html). Here the Pi performs flight control; the flight IMU's position and isolation are central to applying that guidance. Do not blindly reuse tiny-FC damping hardware for a much heavier Pi/sensor assembly. Check logged vibration after assembly.

### Upgrade envelope

For the examples above, start with **180 × 75 mm available tray area and approximately 65 mm adjustable pack-height allowance**, plus separately modeled space for lead bends, connector handling, straps and access. This is a design allowance, not a standardized battery size or a reason to add a heavy enclosed box. Verify the maximum pack envelope in CAD and with a physical mockup.

Retain the main frame across upgrades only when all these conditions remain satisfied:

- Same approved voltage class and compatible connector/charger.
- Pack body and lead exits fit, with proper restraint for that pack size.
- Battery mass and total aircraft mass stay inside the validated envelope.
- Pack current capability, propulsion thrust/response, heat and wiring limits remain adequate.
- Battery can slide to restore center of mass near the rotor center.
- Propeller clearance, camera views, cooling and antenna placement remain adequate.

A larger pack changes inertia and may require control tuning and filter review. Unlimited battery growth without structural or propulsion changes is not a realistic requirement. A replaceable battery plate and replaceable arms offer useful modularity without burdening the first aircraft with every future possibility.

## Power wiring architecture

```text
6S battery -> appropriately rated current/voltage monitoring and distribution
          +-> motor power path -> ESC -> motors
          +-> quiet I/O/sensor supply -> receiver + sensor/I/O hardware + optional Picos
          +-> suitable Pi regulator -> Pi running ArduCopter + AI + AI camera
          +-> suitable FPV supply -> camera + video transmitter

RC receiver -> supported input path -> ArduCopter process on Pi
AI process <-> local MAVLink endpoint <-> ArduCopter process on the same Pi
ArduCopter -> supported output hardware / optional I/O Pico -> ESCs
```

The Pi supply now powers the flight-control computer as well as AI. A Pi restart stops stabilization; the old separate-FC survival assumption no longer applies. Provide a correctly rated Pi supply, quiet sensor/I/O regulation as needed, appropriate protection, common signal reference, short power paths and strain relief. Check that any I/O board's through-power limit is sufficient before feeding the Pi through it. An independent Pico output watchdog can apply a defined action on stale commands, but cannot keep the vehicle level or land it. See the [architecture and failure behavior](PI5_FLIGHT_COMPUTER_ARCHITECTURE.md#failure-behavior-changes).

Keep high-current loops away from the compass and sensitive signals. Model total loop current and conductor heating; final wire/connector sizes cannot be established from average flight power alone. Insulate all exposed conductors from conductive frame material. Exact regulator, capacitor and PCB design awaits component selection and load testing.

## Validation sequence and acceptance

1. First complete the [native Pi/ArduCopter, sensor and output bench milestone](PI5_FLIGHT_COMPUTER_ARCHITECTURE.md#new-first-milestone), with props removed. Run flight control, the intended AI, camera and FPV equipment together. Record timing, sensor freshness, output gaps, Pi-rail and whole-electronics power, startup behavior and temperatures for at least the intended duration plus a test margin. Cooling must represent the actual installation.
2. Weigh the actual dry aircraft. Obtain matched motor/propeller test data and screen both the lighter flight pack and the heaviest allowed upgrade pack for thrust and current. Do this before cutting the final frame.
3. Complete CAD envelopes, battery balance/retention checks and structural review. Produce a schematic and pin-to-pin interface table. No production hardware has been designed by this document.
4. Configure and verify electronics with props removed. Calibrate power monitoring and set battery-specific warnings/failsafes using pack guidance and observed sag; an assumed 80% energy fraction is not a voltage setting.
5. Establish short, controlled flight and inspect logs, vibration, temperatures and power. Increase maneuver intensity progressively. The first flight is not an endurance test.
6. Compare average power and usable energy for each pack during representative flying with AI and FPV running. Use measured Wh where possible rather than nominal voltage times mAh. Evaluate peak current and sag separately.
7. Accept the twenty-minute requirement only after demonstrating **twenty minutes of the agreed maneuver profile**, with an explicit landing reserve, adequate control margin and acceptable temperatures. A smooth cruise, a no-payload result or a flight to battery exhaustion does not satisfy it.

If the maneuver profile exceeds the energy/power budget, choose explicitly among shorter sorties with pack swaps, gentler twenty-minute flights, a lighter AI installation, or a larger propulsion platform with different agility. An 8–10-inch design may improve endurance operating points, but is not an automatic solution to twenty minutes of freestyle.
