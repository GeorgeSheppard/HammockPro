# 06 — Bill of Materials

Prices are approximate (USD, mid-2026). Links are to reference products; equivalents are
fine where noted. Phase numbers refer to the [build plan](07-build-plan.md) — you don't
need to order everything at once, and Phases 1–2 make a working rocker.

## Railing unit (Phases 1–2)

| Item | Part | ~Price | Notes |
|---|---|---|---|
| Motor | [Pololu 37D 30:1 12V, 64 CPR encoder, helical pinion](https://www.pololu.com/category/116/37d-metal-gearmotors) | $55 | The heart. Budget alt: JGB37-520 12V ~330RPM w/ encoder, ~$15 |
| Motor bracket | Pololu 37D stamped bracket | $8 | Don't print this |
| Shaft hub | Pololu 6mm universal aluminum hub | $8 | Bolts to printed spool |
| Motor driver | [Cytron MD13S](https://www.cytron.io/p-13amp-6v-30v-dc-motor-driver) | $12 | Alt: BTS7960 module, ~$8 |
| Controller | ESP32-WROOM-32 devkit | $8 | ESP32-S3 also fine |
| Current sensor | ACS712-20A or ACS723-10A module | $4 | |
| Buck converter | Mini560 12V→5V 3A | $3 | |
| PD trigger | USB-C PD decoy board, 12V | $4 | Pairs with your power bank |
| Power bank | Any 30W+ USB-C PD, 10,000mAh+ | $0–30 | You may already own one |
| Bulk caps | 2× 2200µF 25V low-ESR | $3 | |
| E-stop | NC mushroom button or chunky toggle | $5 | Wired to driver enable |
| Pull line | 2mm Dyneema/SK78 cord, 5–10m | $8 | "Throwline" listings work |
| Fairlead bearing | 608ZZ bearing | $1 | Printed sheave around it |
| Clamp hardware | 2× M6×60 bolts + wing nuts, steel U-bolt or 2 hose clamps | $6 | U-bolt is the structural path |
| Clip | Small locking carabiner + 25mm webbing loop | $6 | |
| Wire, JST connectors, heat-shrink, TPU/PETG filament | — | $10 | From stock, mostly |

**Railing unit subtotal: ~$110–140** (≈$70 with budget motor/driver and an owned power bank)

## Clip pod (Phase 3 — optional but recommended)

| Item | Part | ~Price | Notes |
|---|---|---|---|
| MCU | [Seeed XIAO ESP32-C3](https://wiki.seeedstudio.com/XIAO_ESP32C3_Getting_Started/) | $5 | Onboard LiPo charging + USB-C |
| IMU | LSM6DS3 / LSM6DSO breakout | $6 | I2C, centimeters from MCU — fine |
| Battery | 3.7V 500mAh LiPo, JST-PH | $8 | ~15–30h per charge |
| Switch | Slide switch | $1 | |
| **One-board alternative** | [XIAO nRF52840 **Sense**](https://wiki.seeedstudio.com/XIAO_BLE/) (IMU + charger onboard) | $16 | BLE link instead of ESP-NOW; much longer battery life |

**Clip pod subtotal: ~$20**

## Wired-clip variant (only if you reject wireless — see [doc 03](03-sensing-and-tether.md))

| Item | Part | ~Price |
|---|---|---|
| Cable | 3m USB 2.0 cable (donor: 4 conductors + shield) or 4-core 26AWG | $4 |
| Clip MCU | Same XIAO (UART framing) | $5 |
| C-clips / silicone tape to lash cable to Dyneema | — | $4 |

## Printed parts (your filament)

Spool, fairlead sheave + guide, railing clamp jaws (+ TPU pads), motor/electronics
enclosure with drip shield, clip pod shell, cable C-clips (wired variant). PETG for
structure and anything living outside (better creep/UV than PLA); TPU for pads.

## Grand total

- **Recommended build (Phases 1–3): ~$130–160**
- **Budget build: ~$75–95**
