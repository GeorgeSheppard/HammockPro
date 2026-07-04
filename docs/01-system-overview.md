# 01 — System Overview

## The physics: you are building a swing-pusher, not a crane

A hammock with a person in it is a pendulum. Everything about this project follows from
three numbers, so let's establish them first.

### Natural period

For a pendulum of effective length `L` (suspension point to combined center of mass):

```
T = 2π √(L / g)
```

A hammock typically hangs with its center of mass 1.0–1.5 m below the anchor line, so:

| Effective length L | Period T | Frequency |
|---|---|---|
| 1.0 m | 2.0 s | 0.50 Hz |
| 1.2 m | 2.2 s | 0.45 Hz |
| 1.5 m | 2.5 s | 0.40 Hz |

The period is **independent of the occupant's weight** — a lovely property, because it
means the controller doesn't need to know who's in the hammock. It only needs to find the
period, and it can measure that directly.

### Energy required

Take a 90 kg person + hammock rocking to θ₀ = 10° (a relaxed, pleasant amplitude), L = 1.2 m:

- Height rise at the end of the swing: `h = L(1 − cos θ₀)` ≈ 18 mm
- Total swing energy: `E = mgh` ≈ 90 × 9.81 × 0.018 ≈ **16 J**
- A person-in-hammock swing decays slowly (air drag + suspension friction); the energy lost
  per cycle is only a fraction of the total — realistically **2–5 J per cycle** must be
  replaced to sustain the motion.

### Force and stroke

The hammock's horizontal excursion at ±10° is `x = L sin θ₀` ≈ ±0.21 m, so the tether pays
in and out by roughly **0.3–0.4 m each cycle** (depending on attachment geometry). To
inject ~4 J while the hammock swings toward the device, pulling over ~0.15–0.20 m of that
stroke:

```
F ≈ 4 J / 0.17 m ≈ 24 N   →   design for 50 N continuous, 80 N peak (comfortable margin)
```

Peak line speed (at the bottom of the swing) is `v ≈ θ₀ √(gL)` ≈ **0.6 m/s**.

**These are the design drivers: ~50 N, ~0.6 m/s, ~0.4 m of stroke, once every ~2.2 s.**
That's a few watts of mechanical power. Everything in [doc 02](02-motor-and-winch.md) is
sized from these numbers.

## How the pumping works

To add energy to a pendulum, apply force **in the direction of motion**. A tether can only
pull, so the winch:

1. **Pulls** (winds in, with the pump force) while the hammock moves *toward* the railing.
2. **Pays out** (unwinds, keeping just enough tension that the line never goes slack) while
   the hammock moves *away*.

```
hammock position ──►  away ────────── toward ─────────── away ──────
                       ╱‾‾‾‾‾╲                 ╱‾‾‾‾‾╲
                      ╱       ╲               ╱       ╲
                     ╱         ╲_____________╱         ╲____
line tension:        [ light ]  [ PULL 30N ]  [ light ]
motor:               pay out     wind in      pay out
```

This is exactly how you push a child on a swing: a well-timed nudge once per cycle, in
phase with the velocity. The controller's whole job is (a) knowing the phase and (b)
modulating the pull strength to hold a target amplitude. See
[doc 05](05-firmware-and-control.md).

### The key control insight: tension control is self-synchronizing

If the winch runs in *tension mode* — "always keep ~2 N on the line; when the line starts
coming back toward you, pull with 30 N instead" — the hammock itself dictates the timing.
There is no clock to synchronize, no phase-locked loop to tune. The encoder on the motor
tells you which way the line is moving, and that *is* the phase signal. The IMU in the clip
then becomes a **refinement** (better amplitude estimation, works before tension is
established, detects a person climbing in/out), not a prerequisite. This ordering is what
makes the [build plan](07-build-plan.md) low-risk.

## System architecture

```mermaid
flowchart TB
    subgraph CLIP["Hammock clip pod (3D printed, carabiner-mounted)"]
        IMU["6-DoF IMU<br/>(accel + gyro)"] --> MCU2["XIAO ESP32-C3"]
        LIPO["LiPo 500mAh<br/>USB-C charged"] --> MCU2
    end

    subgraph MAIN["Railing unit (3D-printed clamp + enclosure)"]
        MCU1["ESP32 devkit"]
        DRV["Motor driver"]
        MOT["Brushed DC gearmotor<br/>+ quadrature encoder"]
        SPOOL["Spool + fairlead"]
        PWR["USB-C power bank +<br/>12V PD trigger"]
        SENSE["Current sensor"]

        PWR --> DRV --> MOT --> SPOOL
        PWR -->|buck 5V| MCU1
        MCU1 -->|PWM + DIR| DRV
        MOT -->|encoder A/B| MCU1
        SENSE --> MCU1
    end

    MCU2 -.->|"ESP-NOW, 25 Hz<br/>quaternion/gyro packets"| MCU1
    SPOOL ===|"2mm Dyneema, 2–3 m"| CLIP
    MCU1 <-->|"WiFi AP — web dashboard<br/>for tuning & live plots"| PHONE["Your phone"]
```

## Placement geometry (matters more than it looks)

- **Swing direction:** a hammock rocks *side-to-side*, perpendicular to its long axis. The
  railing unit must sit on the side of the hammock, and the clip should attach at the
  hammock's edge (or to a ridgeline point above the occupant's hip), so the line pulls
  along the direction of swing.
- **Height:** pull as close to horizontal as you can at the hammock's resting height. A
  balcony railing (~1.0–1.1 m) pulling on a hammock bed at 0.5–0.7 m gives a shallow
  downward angle — fine. Efficiency scales with the cosine of the angle between line and
  motion; keep it under ~30°.
- **Distance:** 1.5–3 m of line is the sweet spot. Too close and the line angle changes
  a lot over the stroke (nonlinear, harder on control); too far and sag/whip in the line
  gets annoying.

## Design decisions at a glance

| Decision | Choice | Rejected alternatives & why |
|---|---|---|
| Actuation | Winch (spool + line) | Crank arm: fixed stroke, can't adapt to hammocks/positions. Linear actuator: far too slow (~0.05 m/s). Fan/propeller: comically inefficient at these forces. |
| Motor | Brushed DC gearmotor + encoder | Stepper: poor torque at 0.6 m/s line speed, loud, wastes hold current. BLDC + FOC: lovely but overkill complexity. Sail-winch servo: viable for a quick hack but limited travel and no torque control. |
| Gear ratio | Moderate (~30:1), backdrivable spur/helical | Worm gear: **not backdrivable** — the away-swing would jerk to a stop; also fails dangerous instead of failing soft. |
| Phase sensing | Winch encoder (primary), clip IMU (refinement) | IMU-only: works, but you get the encoder for free and it derisks the build. |
| Clip link | Wireless (ESP-NOW) | Wired I2C: out of spec over 2–3 m and noisy next to motor PWM. Wired UART: workable fallback, documented in [doc 03](03-sensing-and-tether.md). |
| Power | USB-C PD power bank + 12V trigger | Integrated 3S pack: better v2, but battery management is a whole subproject; do it after the fun part works. |
