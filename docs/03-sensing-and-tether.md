# 03 — Sensing & Tether

Two separate problems live here, and your original concept fuses them: **how the swing is
sensed** (the IMU clip) and **what physically connects winch to hammock** (the line).
This doc argues they should be *decoupled* — the biggest single design recommendation in
this project.

## 3.1 Sensing: where the swing signal comes from

You proposed an accelerometer/gyro in the clip so the device works on any hammock. Good
instinct — a 6-DoF IMU on the swinging body gives a clean, hammock-independent phase and
amplitude signal. But there are **three** viable sensing sources, and you should use them
in this order:

### Source 1 (free, build first): the winch encoder

If the line is kept under light tension, **line pay-out equals hammock displacement along
the line** — the winch encoder is a hammock position sensor with sub-millimeter resolution
(see [doc 02](02-motor-and-winch.md)). Differentiate for velocity; the sign of line
velocity is exactly the phase signal the pump algorithm needs. No radio, no second battery,
no clip electronics. This is Phase 2 of the [build plan](07-build-plan.md) and it will
already rock the hammock.

Limitations: it senses only the component of motion along the line, it needs tension
established first, and it can't see "a person just climbed in." That's what the IMU adds.

### Source 2 (the refinement): IMU in the clip — make it wireless

For the clip itself, the wired approach has a hidden trap, so here's the direct advice:

**Recommended: a wireless clip pod.**

- **Board:** [Seeed XIAO ESP32-C3](https://wiki.seeedstudio.com/XIAO_ESP32C3_Getting_Started/) (~$5)
  + a 6-DoF IMU breakout (LSM6DS3 or LSM6DSO, ~$6). Or the one-board option: the
  [XIAO nRF52840 **Sense**](https://wiki.seeedstudio.com/XIAO_BLE/), which has the
  **LSM6DS3 IMU and a LiPo charger already on board** (~$16, talks BLE instead of ESP-NOW).
- **Link:** **ESP-NOW** (if XIAO ESP32-C3 ↔ main ESP32): connectionless, ~1–2 ms latency,
  trivially simple API, far lighter than BLE or WiFi. Stream gyro + accel at 25–50 Hz —
  laughably fast relative to a 0.45 Hz swing. With the nRF52840 Sense, use BLE notifications
  to the ESP32 instead (slightly more code, much better clip battery life).
- **Power:** 3.7 V LiPo, 300–500 mAh, charged over the XIAO's own USB-C port — which
  satisfies your "USB rechargeable" requirement for the clip independently. Duty-cycled
  ESP-NOW at 25 Hz averages ~15–25 mA → **~15–30 h of rocking per charge** (the nRF52840
  over BLE: several times that).
- **Housing:** a small printed pod that a **climbing-style carabiner or heavy dog-leash
  clip** passes through, clipping to the hammock edge seam, grommet, or a strap girth-
  hitched around the hammock body. Pod carries the board, IMU, battery, a power switch,
  and exposes the USB-C port through a plugged port.

### Source 3 (optional, v2+): line tension

A small load cell in the line path, or (cheaper) the motor current sensor, gives direct
tension feedback for smoother force control. Motor current is good enough; a load cell is
a refinement you may never need.

### Why not wired I2C, as originally sketched?

An MPU-6050/LSM6DS3 speaks I2C/SPI, which is a *PCB-scale* bus: total capacitance budget
~400 pF ≈ 3–4 m of ordinary cable *at best*, with zero margin, running right alongside a
motor PWM line radiating switching noise, over connectors that flex 1,600 times an hour.
It will "work on the bench, glitch on the balcony."

**If you do want a wired clip** (documented because it's your stated plan, and it does
remove one battery):

- Put a tiny MCU in the clip anyway (the same XIAO) and send framed packets over **UART at
  115200 baud** — robust over 3 m of cable in a way raw I2C never will be. Wire: 5 V,
  GND, TX (clip→main); regulate 3.3 V locally in the clip. Add a simple checksum per
  packet and tolerate drops.
- 4-core, 26–28 AWG, shielded if convenient (USB 2.0 cable is a perfect donor: shielded,
  4 conductors, flexible, cheap).
- Even then: the data cable **must not be the load-bearing member** — see next section.

The wired option's real cost is mechanical (strain relief at two swinging endpoints,
conductor fatigue at 40,000 flex cycles per 10-hour week) — which is why wireless wins.

## 3.2 The tether: what to actually buy

### The rule: conductors never carry load

Copper work-hardens and fatigues; a cable that is also a winch line fails at the crimp/
connector within weeks. Every serious towed/tethered system separates the strength member
from the conductors. So:

**⭐ Pull line: 2 mm Dyneema (UHMWPE) cord** — sold as "2mm Dyneema winch line,"
"throwline," or SK75/SK78 cord (~$8 for 5–10 m).

- Breaking strength ~180–250 kg — a ~25–35× safety factor over the 80 N working load. Buy
  the safety factor; it's nearly free at this diameter.
- Essentially zero stretch → the encoder position signal stays crisp (nylon paracord
  stretches ~10–20%, which turns your position sensing into a spring-mass guessing game).
- Slippery, quiet over the fairlead, UV-tolerant, doesn't absorb water.
- Terminate with a **figure-8 on a bight** or sewn/whipped loop onto the clip carabiner —
  note Dyneema is slippery, so use knots rated for it (figure-8 family is fine at these
  loads) and leave long tails.

Comfort note: zero stretch also means pulls feel crisp. If the rock feels abrupt, splice
**10–15 cm of 4–6 mm bungee in parallel with a Dyneema safety loop** near the clip
(bungee takes the load for the first few cm, Dyneema loop catches it after). Tune feel
mechanically before tuning it in firmware — but try firmware force-ramping first
([doc 05](05-firmware-and-control.md)); you'll probably never need the bungee.

### If wired: how the cable rides along

Two maker-friendly options, in order of preference:

1. **Spiral-tape / clip the data cable along the Dyneema** with ~5% slack loops every
   20–30 cm (small printed C-clips or self-fusing silicone tape). The line takes all load;
   the cable just follows. Service loop + proper strain relief (a printed cord grip, not
   the solder joint) at both ends.
2. **The paracord trick:** gut a length of 550 paracord (remove 3–4 of the 7 inner
   strands), thread two twisted pairs of 28 AWG silicone wire through the sheath alongside
   the remaining strands. Sheath + strands carry load; the wires float inside with built-in
   slack. Neat, tidy, very "handy person with a 3D printer" — but accept the stretch
   penalty of nylon, and it makes the encoder-as-sensor signal mushier.

A wired build also constrains the winch: **you can't wind a data cable onto a spool that
turns** (you'd need a slip ring — real money, real failure mode). The wired variant
therefore wants the pay-out geometry where the spool only ever winds the last meter... in
practice: yet another reason the wireless clip is the recommended path.

### Connectors and hardware

| Item | Recommendation |
|---|---|
| Clip to hammock | Small locking carabiner (real climbing-rated is overkill but cheap and confidence-inspiring) or 25 mm steel trigger snap; girth-hitch a webbing loop around the hammock edge for attachment on hammocks without grommets |
| Line to winch | 3 wraps on spool + stopper knot through spool hole (wraps take the load) |
| In-clip electrical (wired variant) | JST-SH/JST-GH latching connectors; strain-relieve the cable to the pod shell before the connector |
| Mechanical fuse | See [doc 08](08-safety.md) — a deliberate weak link near the clip, ~150–200 N |
