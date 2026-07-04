# 08 — Safety

A motorized winch riding on a device with a human in it deserves ten deliberate minutes.
The forces here are small (a firm handshake, not a machine), but cyclic + unattended +
human-attached — and now sharing the hammock with the occupant — means we design the
failure modes on purpose.

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
3. **Mechanical fuse:** a deliberate weak link at the anchor end — a loop of ~20 kg-rated
   cord (e.g. #36 bank line or 2 strands of paracord inner) joining the Dyneema to the
   anchor carabiner. Breaks at ~150–200 N: far above any commanded force, far below
   anything that could injure or drag. If everything else fails, the link pops and the
   hammock is simply a hammock again.
4. **Backdrivable gearbox:** power loss mid-pull = the swing pays line out against a dead
   motor, a soft stop ([doc 02](02-motor-and-winch.md)). This is why worm gears are banned.

## Kill switch & watchdog

- **Hardware kill switch:** normally-closed switch in the motor driver's enable line —
  kills torque even with the ESP32 fully hung. Because the pod rides on the hammock, the
  switch is naturally **within the occupant's reach at all times** — one of the quiet
  safety advantages of this layout. Mount it on the pod face, big and obvious.
- **Watchdog:** hardware WDT resets a hung loop; the enable pin is pulled down so a
  resetting/crashed MCU releases the motor.
- **FAULT latches:** any trip requires human reset. The device never auto-retries into a
  person.

## Pinch points — now the top mechanical hazard

The spool spins next to a person in loose clothing, possibly with long hair, possibly with
a curious kid or pet nearby. Non-negotiable pod design rules:

- Spool and fairlead **fully enclosed**; the only opening is a line-exit slot too small
  for a finger (≤8 mm).
- No exposed shaft, hub screws, or gear faces anywhere on the pod exterior.
- The line exit points *away* from the occupant, toward the anchor.
- Never touch the line while the device is powered; never wrap it around a hand. It's
  2 mm cord under motor tension.

## Anchoring & mounting

- **Anchor side:** a webbing loop + locking carabiner around a sound railing baluster or
  post sees only ~80 N — trivial — but confirm the railing member isn't rusted or loose,
  and that building rules allow attaching to it. Loop around a structural member, not a
  decorative infill panel.
- **Pod side:** the button anchors / strap carry the pod (~0.7 kg) plus the pull force.
  The fabric attachment must be a **geometric interlock, never a friction/magnet grip** —
  friction grips loaded in shear fail suddenly and silently under cyclic tugs, dropping a
  live winch onto the occupant ([doc 02](02-motor-and-winch.md)). Inspect the fabric at
  the buttons/hitch during the first sessions; pad on ultralight nylon.
- Nothing on a balcony should be able to fall outward: the pod hangs inboard on the
  hammock and the anchor is a closed loop — keep it that way (no hooking the line over the
  rail top).
- The line across the balcony is a trip/clothesline hazard at ~0.6–1 m height: bright
  Dyneema, and make "unclip when not in use" the habit.

## Battery & electrical

- **Option A (power bank)** outsources cell safety to a certified product — the main
  reason it's recommended, doubly so now that the battery rides next to a person.
- If you build the **3S pack** (v2): always through a BMS, fuse at the pack, no charging
  unattended, no charging below 0 °C, cells from a reputable source (counterfeit 18650s
  are endemic), and a fireproof spot for charging — off the hammock.
- Outdoor use: everything runs at ≤12 V — no shock hazard — but keep the USB-C port and
  switch facing down, boards in a splash-shadowed enclosure, and bring the pod inside when
  not in use.

## Operating rules (tape a copy inside the enclosure lid)

1. Amplitude and force limits are not user-adjustable above the tested maximum.
2. One occupant; no kids using it unattended; no pets clipped to anything, obviously.
3. Getting in or out: device stopped (Phase 3 auto-stop helps; the rule stands anyway).
4. Inspect the line, the strap, and the fuse link for fraying at each setup — replace the
   fuse loop whenever it looks tired; it costs pennies.
5. Wind: above a stiff breeze, the controller fights gusts — just don't run it.
