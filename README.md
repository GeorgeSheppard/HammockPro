# HammockPro

A self-contained, USB-rechargeable pod that clips onto any hammock and keeps it rocking
indefinitely by winching a tether against a fixed anchor, in resonance with the hammock's
natural swing — like a robot that pushes you on a swing, forever.

📚 **Start here: [docs/README.md](docs/README.md)** — full research, design decisions,
motor/line recommendations, bill of materials, and a phased build plan.

## The one-paragraph version

One 3D-printed pod straps to the hammock's edge; inside are a brushed DC gearmotor driving
a spool, an ESP32, an IMU, and a USB-C-charged battery. A thin Dyneema line runs from the
spool to a passive anchor (a webbing loop and carabiner on the balcony railing — no
electronics on that side). Firmware senses the swing phase from the winch's own encoder
and the onboard IMU, winds the line in with a gentle tug each time the hammock swings
*toward* the anchor, then pays it back out under light tension as it swings away. Pumping
energy in at the resonant frequency — a few joules per cycle — is all it takes to sustain
the rock.
