# Movement and upgrade requirements

Updated October 30, 2026. This document records provisional movement, propulsion and expansion targets for the custom Pi 5 / optional Pico quadcopter. It replaces the earlier 3:1 thrust screening target and 1.9 kg gross-mass screening envelope. These are requirements to investigate and validate, not measured aircraft capabilities or an approved payload rating.

## Design cases

Retain 7-inch propellers, a tentative 330–360 mm opposite-motor diagonal, 6S power, native ArduCopter and AI on the Pi 5, optional Picos for defined sensor/I/O tasks, and an independent FPV video path. Exact parts and dimensions remain open.

Use a **2.2 kg future fully loaded design case** to accommodate later projects. The initial [mass allocation](ENDURANCE_AND_MOUNTING_PLAN.md#revised-mass-allocation) remains **875 g dry including 80 g reserve**, or approximately **1.73 kg** with the example 10000 mAh endurance battery. Do not add unnecessary weight simply to reach the future limit.

The difference between 1.73 and 2.2 kg is approximately **470 g of growth budget**, including every added sensor, mount, wire, larger battery and payload. This is not a demonstrated payload capacity. The existing dry allocation already contains reserve; do not count that reserve again as additional hardware capacity. Reconcile the complete mass budget whenever a part changes.

Keep the provisional **1 kg maximum battery class** and [battery-tray allowance](ENDURANCE_AND_MOUNTING_PLAN.md#upgrade-envelope). The 2.2 kg case is a total aircraft requirement, not permission to fit an arbitrarily larger pack. Validate structure, retention, propulsion, balance and cooling at the maximum approved configuration before calling it supported.

## Thrust requirements

Thrust is force. Here **kgf** means kilograms-force, not kilograms of mass; 1 kgf is approximately 9.81 N. In stationary level hover, total upward thrust balances aircraft weight.

```text
hover thrust per motor (kgf) = loaded mass (kg) / 4
thrust-to-weight ratio = total available thrust (N) / aircraft weight (N)
required peak thrust per motor (kgf) = target ratio × loaded mass (kg) / 4
```

| Requirement | Initial endurance configuration | Future loaded design case |
|---|---:|---:|
| Loaded mass used for calculations | 1.73 kg | 2.2 kg |
| Total level-hover thrust | 1.73 kgf / 17.0 N | 2.2 kgf / 21.6 N |
| Level-hover thrust per motor | Approximately 433 gf | 550 gf |
| Proposed available peak thrust per motor | **2.2–2.5 kgf** | **2.2–2.5 kgf** |
| Proposed total peak thrust | **8.8–10 kgf / approximately 86–98 N** | Same propulsion target |
| Corresponding peak thrust-to-weight ratio | Approximately **5.1–5.8:1** | Approximately **4–4.5:1** |

The upgrade requirement is **at least about 4:1 available peak thrust-to-weight at 2.2 kg**, with adequate response and control authority. This is a project design choice, not a universal standard. Four motors producing 2.2 kgf each meet that arithmetic target; aiming toward 2.5 kgf adds margin only if the battery, ESCs, wiring and thermal limits permit it.

Define the voltage, battery state of charge, air conditions and permitted duration for every accepted thrust figure. Verify the chosen motor/prop combination with the intended battery under load, including representative partially discharged conditions. A fresh-battery single-motor bench maximum is not proof of installed four-motor capability. Do not invent a fixed derating percentage without data.

Hover thrust fraction is not throttle-stick fraction. Stick command, ESC command, motor speed, propeller thrust and electrical power have different relationships.

## Manufacturer example and electrical consequences

Hobbywing's [RTF-2807SL 1300 KV specification](https://www.hobbywing.com/en/uploads/file/20260106/3122988f31e535742d09b2b4ebacfe63.pdf) lists these results with an **HQ7x3.5x3 propeller at 24 V**:

| Thrust per motor | Current per motor | Input power per motor |
|---|---:|---:|
| 596 gf | 5.3 A | 128 W |
| 1635 gf | 20.3 A | 487 W |
| 2053 gf | 28.6 A | 688 W |
| 2824 gf | 48.5 A | 1165 W |

The sheet gives a **14-second recommended full-load running limit** for this propeller combination. These are manufacturer bench figures, not continuous aircraft ratings or an independent test. This is a candidate comparison, not a final motor selection; do not substitute another winding or propeller and reuse the figures.

Multiplying the 2053 gf point by four gives approximately **8.21 kgf, 114.4 A and 2.75 kW**, before other electronics. Simultaneous installed output depends on the battery and complete power path. At the published maximum point, four motors would imply about 194 A and 4.66 kW; that calculation does not establish that our aircraft can supply it.

Use the example to screen power delivery and short bursts, not to predict average flight power or twenty-minute endurance. The listed data does not include the proposed aircraft's hover operating point.

## Movement targets

The following are provisional development targets to evaluate after stable flight, not measured performance or ready-to-flash settings:

| Characteristic | Target to evaluate | Evidence required |
|---|---|---|
| Horizontal acceleration | Approximately **0.5 g / 4.9 m/s²** in short maneuvers | Logged velocity/acceleration tracking with acceptable attitude and altitude behavior |
| Roll/pitch angular rate | **300–500 degrees/second** | Commanded versus achieved rate, response time, saturation and vibration logs |
| Yaw angular rate | **150–250 degrees/second** | Measured rate tracking and control margin with the installed payload |
| Maximum forward speed | **Not yet specified** | Aerodynamic and propulsion assessment followed by measured flight behavior |
| Twenty-minute endurance | Retained as a **stretch goal** with AI and FPV running during the agreed frequent-maneuver profile | Demonstrated duration with explicit landing reserve and acceptable temperatures |

At a steady rate of 360 degrees/second, a full rotation takes one second. The time needed to accelerate to that rate is a separate specification. Mass distribution, propeller inertia, motor torque, battery sag and controller timing all affect response.

Ignoring drag and assuming altitude is maintained, horizontal acceleration is approximately `g × tan(tilt angle)` and required total thrust is `weight / cos(tilt angle)`. At 30 degrees, these are about **5.7 m/s²** and **1.15 times hover thrust**. These relationships explain force direction; they do not predict top speed or validate transient maneuvers.

ArduPilot exposes angular-rate, angular-acceleration and input-shaping controls. Follow its [input-shaping guidance](https://ardupilot.org/copter/docs/input-shaping.html), use measured aircraft response, and revisit tuning as payload/inertia changes. Entering a requested rate does not establish that the aircraft can achieve it. Pi/AI scheduling and Pico output timing must also pass the [native flight-computer bench milestone](PI5_FLIGHT_COMPUTER_ARCHITECTURE.md#new-first-milestone).

## Upgrade provisions

- **Motors and props:** compare suitable 2807–2809-class candidates using exact 7-inch propeller and voltage test data. The thrust target takes priority over a stator-size or KV label. Larger motors/props require a new mass, clearance and efficiency assessment.
- **ESCs:** investigate **65–80 A per motor channel**, subject to actual matched peak current, burst duration, cooling, voltage and signal-protocol compatibility. The rating is not the expected current draw. A four-channel ESC's per-channel rating does not establish an equal continuous rating for its shared battery input path.
- **Battery and distribution:** verify continuous and burst capability of cells, pack interconnects, connector, leads, current sensor and PCB together. Capacity and marketing C ratings alone cannot establish the available thrust. Provide rated expansion connectors and a rail/load budget for each peripheral.
- **Frame:** retain replaceable arms and a removable central electronics tray. Check the 2.2 kg case, motor loads up to the accepted peak, arm bending/torsion, fasteners, vibration and maneuver loads. Reserve useful mounting space without fitting heavy unused enclosures.
- **Balance and battery retention:** retain the adjustable tray, non-slip support and two independent straps. Check center of mass, cooler airflow, propeller sweep and camera view for each payload/pack configuration.
- **Interfaces:** retain accessible programming and power connections for Pi, optional Picos and future sensors. Document spare interfaces and the required ArduPilot driver support instead of assuming any added sensor will work automatically.

The present 270 g motor/prop/ESC allocation is a target, not a selected assembly. If higher-headroom parts exceed it, debit the existing reserve or revise the mass budget; do not assume stronger motors add no weight. Increasing peak thrust does not itself improve flight time. The initial approximately 40 W electronics allocation and the twenty-minute battery energy limits remain provisional and require simultaneous-workload measurements.

## Acceptance gates

1. Produce a complete mass and power budget for the initial build and the 2.2 kg future case, including battery, wiring, mounts and reserve accounting.
2. Select exact motor/prop/ESC candidates and obtain matched thrust/current data. Verify peak duration, battery voltage under load, current capability and thermal limits for the complete power path.
3. Review the frame, arm interfaces and retention against the future case before manufacturing. No structural calculation or test has yet established that rating.
4. Complete the propeller-free Pi/ArduCopter/Pico timing and failure-behavior milestone under the intended AI workload. Use a suitable separate fixture for any propulsion thrust testing.
5. Establish controlled flight, inspect vibration/temperature/current logs, then progressively evaluate movement targets. Repeat relevant checks after changing payload or battery; do not jump directly to maximum-rate or maximum-payload tests.
6. Accept endurance only from representative measured flight with AI/FPV running and a stated reserve. Do not infer duration or payload support from peak thrust alone.

Recorded status: arithmetic and document consistency can be checked now. No exact propulsion combination, production frame, timing configuration, flight speed, payload capacity or movement performance has been validated.
