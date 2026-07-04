# HammockPro

A USB-rechargeable, railing-mounted winch that keeps a hammock rocking indefinitely by
pulling on a tether in resonance with the hammock's natural swing — like a robot that
pushes you on a swing, forever.

📚 **Start here: [docs/README.md](docs/README.md)** — full research, design decisions,
motor/cable recommendations, bill of materials, and a phased build plan.

## The one-paragraph version

A brushed DC gearmotor drives a 3D-printed spool clamped to the balcony railing. A thin
Dyneema line runs from the spool to a clip on the hammock. Firmware on an ESP32 senses the
swing phase (from the winch's own encoder, plus an optional IMU in the clip) and winds the
line in with a gentle tug each time the hammock swings *toward* the railing, then pays it
back out under light tension as it swings away. Pumping energy in at the resonant frequency
— a few joules per cycle — is all it takes to sustain the rock.
