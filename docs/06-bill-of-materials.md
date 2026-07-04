# 06 — Bill of Materials

Prices are approximate (USD, mid-2026). Links are to reference products; equivalents are
fine where noted. Phase numbers refer to the [build plan](07-build-plan.md) — Phases 1–2
make a working rocker.

## The pod (Phases 1–3)

| Item | Part | ~Price | Notes |
|---|---|---|---|
| Motor | [Pololu 37D 30:1 12V, 64 CPR encoder, helical pinion](https://www.pololu.com/category/116/37d-metal-gearmotors) | $55 | The heart. Budget alt: JGB37-520 12V ~330RPM w/ encoder, ~$15 (louder — it rides next to you) |
| Motor bracket | Pololu 37D stamped bracket | $8 | Don't print this |
| Shaft hub | Pololu 6mm universal aluminum hub | $8 | Bolts to printed spool |
| Motor driver | [Cytron MD13S](https://www.cytron.io/p-13amp-6v-30v-dc-motor-driver) | $12 | Alt: BTS7960 module, ~$8 |
| Controller | ESP32-WROOM-32 devkit | $8 | ESP32-S3 also fine; not C3 |
| IMU | LSM6DS3 / LSM6DSO breakout | $6 | I2C, centimeters from the ESP32; MPU-6050 also fine |
| Current sensor | ACS712-20A or ACS723-10A module | $4 | |
| Buck converter | Mini560 12V→5V 3A | $3 | |
| PD trigger | USB-C PD decoy board, 12V | $4 | Pairs with the power bank |
| Power bank | Compact 30W+ USB-C PD, 5,000–10,000mAh | $0–30 | You may already own one; 5,000mAh saves ~90g |
| Bulk caps | 2× 2200µF 25V low-ESR | $3 | |
| Kill switch | NC toggle/mushroom, panel-mount | $5 | Wired to driver enable, reachable from the hammock |
| Pull line | 2mm Dyneema/SK78 cord, 5–10m | $8 | "Throwline" listings work |
| Fairlead bearing | 608ZZ bearing | $1 | Printed sheave around it |
| Pod strap | 1m of 25mm webbing (+ tri-glide or sewn loop) | $4 | Girth-hitches around hammock edge |
| Anchor | 25mm webbing loop + small locking carabiner | $8 | The entire railing-side hardware |
| Wire, JST connectors, heat-shrink, TPU/PETG filament, foam pad for IMU | — | $10 | From stock, mostly |

## Printed parts (your filament)

Spool, fairlead sheave + exit guide, pod enclosure (two-slot strap interface, TPU motor
isolation grommets, splash shield, down-facing USB-C port), small line-fuse-link jig if
you want tidy fuse loops. PETG for structure (better creep/UV than PLA); TPU for grommets
and pads. **No railing clamp to print** — the anchor side is webbing and a carabiner.

## Grand total

- **Recommended build: ~$120–150** (≈$115 of parts + power bank if needed)
- **Budget build: ~$65–85** (budget motor/driver, owned power bank)

Compared to the earlier split-unit concept, this drops the second MCU, LiPo, radio link,
and the engineered railing clamp — about $25 and a subsystem's worth of complexity.
