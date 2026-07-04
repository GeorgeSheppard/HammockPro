# 03 — Sensing & Tether

Two problems live here: **how the swing is sensed** and **what physically connects the pod
to the anchor**. The pod-on-hammock architecture makes both pleasantly simple — the IMU
lives on the pod's own board stack (your original "sensor on the swinging body" idea,
minus the cable run), and the tether is dumb rope.

## 3.1 Sensing: where the swing signal comes from

Three sources, used in this order:

### Source 1 (free, build first): the winch encoder

If the line is kept under light tension, **line pay-out equals the pod's displacement along
the line** — and the pod is strapped to the hammock, so the winch encoder is a hammock
position sensor with sub-millimeter resolution (see [doc 02](02-motor-and-winch.md)).
Differentiate for velocity; the sign of line velocity is exactly the phase signal the pump
algorithm needs. No IMU required yet. This is Phase 2 of the
[build plan](07-build-plan.md) and it will already rock the hammock.

Limitations: it senses only the component of motion along the line, it needs tension
established first, and it can't see "a person just climbed in." That's what the IMU adds.

### Source 2 (the refinement): the onboard IMU

A **6-DoF IMU breakout (LSM6DS3 / LSM6DSO / MPU-6050 class, ~$6)** mounted inside the pod,
wired to the ESP32 over **I2C across a few centimeters** — which is exactly the PCB-scale
distance I2C is designed for. Because the pod rides on the hammock, the IMU measures the
swing directly: gyro gives phase and rate, accelerometer gives tilt amplitude and the large
transients of someone climbing in or out.

What it contributes on top of the encoder:
- **Startup from dead-still:** any residual micro-swing gives the gyro a phase to lock
  onto, so the controller can skip the frequency-sweep seeding.
- **True amplitude** (angle), independent of line geometry.
- **Occupancy detection** — auto-stop when someone climbs in/out
  ([doc 05](05-firmware-and-control.md)).

Two practical notes:
- **Vibration:** the IMU shares a box with a gearmotor. This is a non-problem in firmware —
  the swing is 0.45 Hz and gear noise is >50 Hz, so a modest low-pass filter separates them
  cleanly — but mount the breakout on a foam pad or TPU standoffs anyway, away from the
  motor bracket.
- Solder it in from day one (it's $6 and two wires), even though the firmware only starts
  using it in Phase 3.

### Source 3 (optional, v2+): line tension

A small load cell in the line path, or (cheaper) the motor current sensor, gives direct
tension feedback for smoother force control. Motor current is good enough; a load cell is
a refinement you may never need.

### The road not taken (for the record)

Earlier iterations of this design put the motor on the railing and the IMU in a separate
hammock clip, which forces a bad choice: I2C over 2–3 m of cable (out of spec — the bus
budget is ~400 pF, and it would run alongside motor PWM noise while flexing 1,600 times an
hour) or a radio link + second battery. Co-locating everything on the hammock deletes that
whole problem space. If you ever re-house the electronics railing-side for noise reasons,
revisit this: the answer there is a wireless clip (XIAO + IMU + small LiPo over ESP-NOW),
never long-run I2C.

## 3.2 The tether: what to actually buy

Nothing on the anchor side needs power or data, so the tether is pure strength member — no
conductors, no connectors, no fatigue-prone copper in a flexing load path.

**⭐ Pull line: 2 mm Dyneema (UHMWPE) cord** — sold as "2mm Dyneema winch line,"
"throwline," or SK75/SK78 cord (~$8 for 5–10 m).

- Breaking strength ~180–250 kg — a ~25–35× safety factor over the 80 N working load. Buy
  the safety factor; it's nearly free at this diameter.
- Essentially zero stretch → the encoder position signal stays crisp (nylon paracord
  stretches ~10–20%, which turns your position sensing into a spring-mass guessing game).
- Slippery, quiet over the fairlead, UV-tolerant, doesn't absorb water.
- Terminate with a **figure-8 on a bight** onto the anchor carabiner — Dyneema is slippery,
  so use knots that hold in it (figure-8 family is fine at these loads) and leave long
  tails.

Comfort note: zero stretch also means pulls feel crisp. If the rock feels abrupt, splice
**10–15 cm of 4–6 mm bungee in parallel with a Dyneema safety loop** near the anchor
(bungee takes the load for the first few cm, Dyneema loop catches it after). Tune feel
mechanically before tuning it in firmware — but try firmware force-ramping first
([doc 05](05-firmware-and-control.md)); you'll probably never need the bungee.

### Anchor-side hardware

| Item | Recommendation |
|---|---|
| Anchor loop | 25 mm webbing sewn/tied loop (or a cam-buckle strap) around a railing baluster, post, tree, or wall hook |
| Connector | Small locking carabiner (climbing-rated is overkill but cheap and confidence-inspiring) |
| Line to spool | 3 wraps on spool + stopper knot through spool hole (wraps take the load) |
| Mechanical fuse | See [doc 08](08-safety.md) — a deliberate weak link at the anchor end, ~150–200 N |

### Pod-side attachment (recap from [doc 02](02-motor-and-winch.md))

25 mm webbing girth-hitched around the hammock edge through two strap slots on the pod.
This is the load path for both the pod's weight (~0.7 kg) and the pull force (~80 N max) —
comfortably within webbing and hammock-fabric limits when spread by the hitch.
