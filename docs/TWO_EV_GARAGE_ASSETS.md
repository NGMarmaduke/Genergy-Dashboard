# Two-EV Garage — Asset Specification

This document specifies the image assets needed to show a **second garage + car** on the
house card when two EV chargers are configured. The code to consume these assets already
ships (gated behind the **Settings → Features → "Two-Garage House Scene (EV 2)"** toggle,
default off). Once the images below are added to the integration's `frontend/images/`
folder and the toggle is enabled, the second garage becomes live — no code changes needed.

---

## Why new artwork is required

The house scene (house + garage + gate + car) is a **pre-painted raster PNG**, not vector
code. The "gate opening" is a swap between whole-scene images. The current art contains
**one** garage, so a second EV can only be shown by supplying **new composite scenes** that
include a second garage. These must be produced by a designer / image tool — they can't be
generated from code.

---

## Required images (day)

All four are **required**. Place them in `custom_components/genergy_dashboard/frontend/images/`.

| Filename | Scene |
|----------|-------|
| `home_2ev_none.png` | House with **two** garages, **both gates closed**, no cars. Both wall AC chargers visible. |
| `home_2ev_ev1.png`  | Garage **1** gate open + car present; garage 2 closed, no car. |
| `home_2ev_ev2.png`  | Garage **2** gate open + car present; garage 1 closed, no car. |
| `home_2ev_both.png` | **Both** gates open, **both** cars present. |

The code picks one of these four based on which EV(s) are currently connected/charging:

```
              EV2 not shown          EV2 shown
EV1 not shown  home_2ev_none.png     home_2ev_ev2.png
EV1 shown      home_2ev_ev1.png      home_2ev_both.png
```

- **EV 1** = the *existing* garage position (same spot as today's single-garage car), driven by the unprefixed `ev_charger_power` / `ev_charger_state` entities.
- **EV 2** = the *new* second garage, driven by `ev2_charger_power` / `ev2_charger_state`.

---

## Hard requirements (so it drops in without code changes)

1. **Dimensions: exactly `1170 × 1013 px`** — identical to the current `home_has_solar_has_car.png`. The card's SVG flow-line coordinates, label positions, and clickable zones are all mapped to this canvas; any change breaks alignment.
2. **Format: PNG, RGBA (8-bit), transparent background** — match the transparency of the existing scene PNGs so the card background/theme shows through the same way.
3. **Pixel-perfect registration** — the house body, roof, battery (SigenStor), ammeter, and everything shared with the current art must sit at the **same pixels** as `home_has_solar_no_car.png`. Only the garage area changes. Easiest path: start from the existing scene and paint the second garage in.
4. **Both wall AC chargers baked in** — in two-garage mode the card does **not** draw the separate `ac_charger_bg.png` overlay, so each garage's wall charger must be part of these scenes.
5. **Keep art style / perspective / lighting consistent** with the current illustration.

---

## Optional: night variants (future)

Night support is **not wired yet** — two-garage mode currently uses the day scenes at all
times. If/when you want night versions, provide `dark_home_2ev_none.png`,
`dark_home_2ev_ev1.png`, `dark_home_2ev_ev2.png`, `dark_home_2ev_both.png` (same specs) and
open an issue; a one-line code change enables night selection (`this._isNight`).

---

## What the code already does with these

In `sigenergy-house-card.js`:
- `_twoEvGarage` — true when `features.two_ev_garage` is on (set only when a 2nd EV is enabled).
- `_showEvVehicle` / `_showEv2Vehicle` — per-EV "is the car shown" using each EV's own
  power threshold + connection-state detection (auto mode), or the manual "Always Show EV
  Vehicle" toggle.
- `_baseImage` — returns `home_2ev_{none|ev1|ev2|both}.png` when `_twoEvGarage` is on.

## Not yet included (nice-to-have follow-ups)

These are **not** part of this asset drop and would be separate small tasks once garage 2's
coordinates are known from the finished art:
- A second animated **energy-flow cable/dots** line to garage 2.
- A second EV **label / live power value** overlay near garage 2.
- Night scenes (see above).

---

## How to enable (after adding the images)

1. Drop the four PNGs into `custom_components/genergy_dashboard/frontend/images/`.
2. Settings → **Features** → set **Number of EV Chargers** to **2**.
3. Settings → **Features** → enable **Two-Garage House Scene (EV 2)**.
4. Configure `ev2_charger_power` / `ev2_charger_state` (Settings → Entities) so EV 2's gate
   opens on connect/charge. With **Auto EV** on, each garage opens independently based on
   its own charger.
