# 08 — Safety

A motorized winch attached to a device with a human in it deserves ten deliberate minutes.
The forces here are small (a firm handshake, not a machine), but cyclic + unattended +
human-attached means we design the failure modes on purpose.

## Guiding principle

**The device may only ever add a gentle nudge to a system that works fine without it.**
It must never be capable of holding, lifting, or restraining the occupant. Every limit
below enforces that.

## Force & energy limits (defense in depth)

1. **Firmware clamp:** `I_max` caps motor torque → line force ≤ ~80 N, evaluated every
   control tick outside the controller logic. Plus stroke window and speed clamps
   ([doc 05](05-firmware-and-control.md)).
2. **Electrical ceiling:** the power bank's own over-current protection bounds worst-case
   sustained output (~30–65 W) regardless of firmware.
3. **Mechanical fuse:** a deliberate weak link near the clip — a loop of ~20 kg-rated cord
   (e.g. #36 bank line or 2 strands of paracord inner) joining the Dyneema to the
   carabiner. Breaks at ~150–200 N: far above any commanded force, far below anything that
   could injure or drag. If everything else fails, the link pops and the hammock is simply
   a hammock again.
4. **Backdrivable gearbox:** power loss mid-pull = line pays out against a dead motor,
   a soft stop ([doc 02](02-motor-and-winch.md)). This is why worm gears are banned.

## E-stop & watchdog

- **Hardware e-stop:** normally-closed switch in the motor driver's enable line — kills
  torque even with the ESP32 fully hung. Mounted on the unit, reachable from the hammock.
- **Watchdog:** hardware WDT resets a hung loop; the enable pin is pulled down so a
  resetting/crashed MCU releases the motor.
- **FAULT latches:** any trip requires human reset. The device never auto-retries into a
  person.

## Mounting

- The railing clamp sees ~80 N cyclic at 0.45 Hz for hours: use the **steel U-bolt/hose
  clamps as the structural path**, printed parts for positioning. Re-check tightness for
  the first few sessions (printed parts creep).
- Confirm your **railing** is sound and that building rules allow attaching to it. The
  loads are trivial versus a leaning person, but a rusted rail is a rusted rail.
- Orient so a total detachment sends the unit *inward onto the balcony floor* — nothing on
  a balcony should ever be able to fall outward. If the unit sits near/above the rail top,
  add a short safety lanyard to a solid point.

## Line & pinch points

- Keep the spool and fairlead fully enclosed except the line exit — no finger-sized
  openings near moving parts (kids and pets find winches fascinating).
- The line across a balcony is a trip/clothesline hazard at ~0.6–1 m height: make it
  visible (Dyneema comes in bright colors), and make "unclip when not in use" the habit.
- Never wrap the line around a hand to pull it. It's 2 mm cord under motor tension.

## Battery & electrical

- **Option A (power bank)** outsources cell safety to a certified product — the main
  reason it's recommended for v1.
- If you build the **3S pack** (v2): always through a BMS, fuse at the pack, no charging
  unattended on the balcony, no charging below 0 °C, cells from a reputable source
  (counterfeit 18650s are endemic), and a fireproof spot for charging.
- Outdoor use: everything runs at ≤12 V — no shock hazard — but keep connectors facing
  down, board in a splash-shadowed enclosure, and bring it inside when not in use.

## Operating rules (tape a copy inside the enclosure lid)

1. Amplitude and force limits are not user-adjustable above the tested maximum.
2. One occupant; no kids using it unattended; no pets clipped to anything, obviously.
3. Getting in or out: device stopped (Phase 3 auto-stop helps; the rule stands anyway).
4. Inspect the line and the fuse link for fraying at each setup — replace the fuse loop
   whenever it looks tired; it costs pennies.
5. Wind: above a stiff breeze, the controller fights gusts — just don't run it.
