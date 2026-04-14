# VTOL Pop-Up Repeaters for Amateur Radio and Environmental — Design Log

**Leader:** Joshua Mendez
**Name:** Bao Nguyen
**Date:** July 4, 2025
**Course:** URMP

---

## Introduction

### Welcome to Project UWU: Unmanned Wide-area Uplink

This project focuses on designing and building a delta wing UAV that can carry a payload of 250 grams or more, including sensors and measurement tools. The drone will be built for long-endurance flights, with a target duration between 1 and 4 hours. All parts, including the frame, ESCs, motors, battery, GPS, and flight controller, will be selected for low cost and high efficiency.

The drone will fly primarily using GPS-based autonomous navigation, with manual override available through remote control. It should be easy for anyone to use. The goal is to create a stable, efficient, and affordable platform for collecting data in the field.

---

## Attributes

| Attribute | Status |
|-----------|--------|
| Dual motor | Yes |
| 1–4 hr flight time | Questionable, maybe 1–2 hours |
| Size limit? | No |
| Delta Wing Style airframe | Yes |
| 250g+ payload | Yes |
| Flight controller | Yes |
| Material | Undetermined, foamboard for gen 1 |
| GPS | Yes |
| Lands and Launches vertically | Yes |
| Battery Size? | Undetermined |

---

## FAA Registration

No one cares we are recreational — ok may need to register with the FAA since this drone will be way heavier than 250g.

---

## Choosing Frame Specs

We are planning a twin motor delta wing design with VTOL capability. Target wingspan is 1000mm, optimized for efficient gliding and long-duration cruise. Materials may include EPP foam or laser-cut foam board with carbon fiber spars for strength.

### Rules of Thumb for VTOL Capability

| Category | Rule of Thumb | Explanation |
|----------|---------------|-------------|
| [Thrust-to-weight](https://discuss.ardupilot.org/t/thrust-to-weight-ratio/55253/1) | ≥2:1 total thrust | Total vertical motor thrust must be at least 2× total weight to handle safe takeoff, hover, and basic maneuvering with margin |
| [Motor sizing](https://www.reddit.com/r/fpv/comments/12buu69/what_consumes_more_power_hovering_or_chill_fly/) | 1 motor per 500–1000g (for lift motors) | Balance motor weight and thrust. For endurance, use fewer, larger motors instead of many small ones |
| Prop size | Larger, slower props = more efficient | 10–13" props on 4S–6S systems are common; low pitch helps for stable VTOL and efficient hover |
| Hover power draw | ~60–80% of full throttle current | Plan for 30–40A total current draw during hover for ~2kg drones; don't size ESCs too tight |
| Hover time vs. cruise | Hover burns 4–6× more power | Minimize hover time — just long enough for takeoff/landing. Plan to spend 90%+ of flight in fixed-wing mode. **We are gliding in a circle.** |
| [Battery sizing](https://www.grepow.com/blog/li-ion-vs-lipo-battery-for-long-range-flights.html) | Use Li-ion for endurance, LiPo for power | Li-ion packs offer higher energy density, but lower current tolerance |
| Center of gravity (CG) | At or slightly ahead of wing's aerodynamic center (for delta planes) | Crucial for smooth transition between hover and forward flight; balance battery, motors, payloads carefully |
| Flight controller | Must support fixed wing aircraft and hovering | ArduPilot and PX4 are top choices; support vertical-to-fixed transitions, auto missions, and fallback modes |
| ESCs | 30–60A with braking, BLHeli_S or fixed-wing ESCs | Pick reliable, smooth throttle ESCs that support brake-on-stop for folding props (reduces drag in cruise) |
| [Airframe weight budget](https://www.dronevibes.com/forums/threads/payload-as-a-of-auw.19033/) | Frame + motors + ESCs should not exceed 35% of total drone | Save weight for battery and payload |

> **Airfoil note:** Need to calculate the appropriate airfoil for mostly horizontal flight with climb to offset gravity. Should consult a specialist (Peter).

> **Reference builds:** Eflite X-VERT VTOL and the XK/WLTOYS X520

**Chosen wingspan:** 1200mm (to properly size for heavy payload + VTOL)

---

## Choosing a Motor and Propeller

On my Carl Goldberg Super Cub I'm using a 4120 710KV motor — feels safe but slightly underpowered on 4S. So we should choose a higher KV to ensure enough RPM for more power. That aircraft is also incredibly heavy (~8 lbs).

The Delta Wing Drone will probably weigh less than 2kg (4.4 lbs) so we can use lighter materials.

- **Motors:** Dual 2814 900KV–1000KV motors (or similar) for efficient hover and cruise
  - "Mitoot Style" motors: orange/gold top and bottom, silver can — very strong for the price
- **Props:** 10×5, 11×5, or 10×6 slow-fly props — folding props maybe?
- Each motor must produce at least **1.2 kg of thrust** for vertical takeoff at ~1.6–1.8 kg total weight

### Motor Comparison Table

| Vendor | Motor | KV | Battery / Max Thrust | Weight (g) | Total Cost (2x, USD) |
|--------|-------|----|----------------------|------------|----------------------|
| [Amazon](https://www.amazon.com/Brushless-Thrust-Electric-Controller-1000KV/dp/B09YRP5RDW) | 2814 | 1000 KV | 2–4S / unknown | 52g | $42 |
| [Amazon](https://www.amazon.com/FlashHobby-Brushless-Aircraft-Helicopter-Outrunner/dp/B089Y9KYY4) | FLASH HOBBY D2830 | 1000 KV | 2–4S / 890g | 52g | $38 |
| [Amazon](https://www.amazon.com/FLASH-HOBBY-Brushless-Multicopter-Fixed-Wing/dp/B08B1FMYCF) | FLASH HOBBY D3542 | 1000 KV | 2–5S / 1260g | 130g | $46 |
| [Amazon](https://www.amazon.com/FLASH-HOBBY-Brushless-Multicopter-Fixed-Wing/dp/B08B1FMYCF) | FLASH HOBBY D3542 | 1450 KV | 2–5S / 1260g | 130g | $46 |
| [FlashHobby](https://www.flashhobby.com/2830-evo-fixed-wing-motor.html) | FLASH HOBBY D2830 EVO | 1000 KV | 2–4S / 930g | 68.1g | $40 |
| — | FlashHobby D4260 EVO | 600 KV | 5–6S / 2000g | — | — |
| [Amazon](https://www.amazon.com/FlashHobby-D3548-Brushless-Multicopters-Helicopter/dp/B08B1D6GJD) | D3548 | 790 KV | 3–5S / 1650g | 156g | $40 |
| [Amazon](https://www.amazon.com/FlashHobby-Brushless-Aircraft-Helicopter-Outrunner/dp/B089YJ52JC) | D2836 | 1100 KV | 2–4S / 1130g | 72g | $38 |
| **[Amazon](https://www.amazon.com/FlashHobby-Brushless-Aircraft-Helicopter-Outrunner/dp/B089YN3DBR)** | **D2836 850KV** | **850 KV** | **2–4S / 850g** | **72g** | **$38** |

---

### Deep Dive into Motor Research

Full test data for each motor series is in separate reference files:

| Motor Series | File |
|---|---|
| FlashHobby D2830 EVO (750/850/1000/1300 KV) | [D2830_EVO.md](motors/D2830_EVO.md) |
| FlashHobby D2836 EVO (750/850/1100/1450 KV) — **Selected** | [D2836_EVO.md](motors/D2836_EVO.md) |
| FlashHobby D2826 EVO (930/1000/1450/2200 KV) | [D2826_EVO.md](motors/D2826_EVO.md) |
| FlashHobby D3530 EVO (1100/1400/1700 KV) | [D3530_EVO.md](motors/D3530_EVO.md) |
| FlashHobby 3536 EVO (910/1000/1250/1450 KV) | [3536_EVO.md](motors/3536_EVO.md) |
| FlashHobby D3542 EVO (1000/1250/1400 KV) | [D3542_EVO.md](motors/D3542_EVO.md) |


## Choosing ESC

We need a 50A ESC with advanced programming features to tune for endurance. Any BLHeli-based ESC will work.

**Options:**
- [BLHeli RC Brushless ESC (12A–80A range)](https://www.amazon.com/Brushless-Electric-Controller-Quadcopter-Multi-axis/dp/B0DDKK8CR7) — budget option
- [HGLRC Specter 90A 8S BLHeli32](https://www.amazon.com/HGLRC-90A-ESC-High-Performance-Professional/dp/B0DQKS5Z63) — high-performance option

---

## Choosing a Flight Controller

We want a PX4-based flight controller.

| Flight Controller | Notes |
|-------------------|-------|
| [SpeedyBee F405 WING APP](https://www.speedybee.com/speedybee-f405-wing-app-fixed-wing-flight-controller/) | Limited — non-native VTOL support |
| **[Pixhawk 6C](https://www.amazon.com/gp/product/B07NRMFTXL)** | **Good option. Newer, info harder to find. ~$300** |
| Pixhawk 4 | Good option. Well documented for VTOL. Expensive. |
| Pixhawk 2.4.8 | ~$209 |
| [T1 VTOL FX-405 (with GPS & PMU)](https://www.heewing.com/products/fx-405-fc-gps-pmu) | Affordable, that's about it |

Required sensors: GPS, barometer

**Setup Videos:**
- [(1/5) PixHawk Video Series — Simple initial setup, config and calibration](https://www.youtube.com/watch?v=uH2iCRA9G7k&list=PLYsWjANuAm4r4idFZY24pP6s1K6ABMU0p)
- [PixHawk/ArduCopter for Beginners (2023): Flashing the PixHawk 6C and basic setup steps](https://www.youtube.com/watch?v=_ketmb8u2UI)

---

## Choosing Controller Electronics

Already have a **RadioMaster Boxer** radio controller. Just need an ELRS receiver.

- **Receiver:** [RadioMaster RP3](https://www.amazon.com/SoloGood-RadioMaster-RP3-ExpressLRS-Quadcopter/dp/B0BGBKG635) — longest range ELRS nano RX
- **Module:** [RadioMaster Nomad](https://radiomasterrc.com/collections/nomad/products/nomad-dual-1-watt-gemini-xrossband-expresslrs-module) — supports both 2.4GHz and 900MHz

### Frequency Plan

| Band | Use |
|------|-----|
| 915 MHz | ELRS RX (control link) |
| 433 MHz | Telemetry (avoids clash) |
| 5.8 GHz | Video |

---

## Choosing a Power Source

For long-range fixed-wing drones, **lithium-ion (Li-ion)** batteries are preferred over LiPo. Li-ion offers higher energy density for longer flight times at the cost of lower peak current — ideal for endurance-focused cruisers.

**Planned pack:** 4S4P Li-ion (14.8V, ~16.8Ah) built from **Molicel P42A** or **Sony VTC6** cells
- ~248 Wh usable energy
- Supports 2+ hour flight times depending on throttle and glide efficiency
- Lighter per Wh than LiPo

> Ed was asked to buy Sony VTC6 18650 cells.

LiPo may still be used for short missions, high-power VTOL phases, or test flights.

---

## GPS / Navigation Module

Since we plan to operate 5–10 UAVs, accurate GPS is critical for safe autonomous landings.

### What is GNSS?
GNSS (Global Navigation Satellite System) is the collective term for satellite positioning systems: GPS (USA), GLONASS (Russia), Galileo (Europe), BeiDou (China). Multi-GNSS modules listen to all simultaneously for faster, more accurate fixes.

### What is Update Rate?
Update rate (Hz) = how often the GPS sends a new position to the flight controller.

| Model | GNSS Systems | Update Rate | Sensitivity | Compass | Power Draw | Compatibility | Pros | Cons | Cost |
|-------|-------------|-------------|-------------|---------|------------|---------------|------|------|------|
| M8 | GPS, GLONASS | Up to 10 Hz | Low in poor weather | Usually included | ~50 mW | Fully supported | Cheap, widely available | Poor in cloudy/urban | $20–$35 |
| M9N | GPS, GLONASS, Galileo, BeiDou | Up to 25 Hz | Good | Usually included (IST8310) | ~48 mW | Fully supported | High update rate, reliable | Slightly higher power | $50–$70 |
| **M10** | **GPS, GLONASS, Galileo, BeiDou** | **Max 10 Hz** | **Excellent** | **Varies** | **~25 mW** | **PX4 v1.14+** | **Better signal, lower power** | **Lower update rate** | **$60–$90** |

**Chosen: M10** — half the power draw of M9N, sufficient 10 Hz update rate for a fixed-wing cruiser.

---

## Tail Cone

### Why does the tail cone matter?

1. **Aerodynamic Stability** — smooths airflow around the tail, reduces turbulence near control surfaces
2. **Shifts Center of Pressure** — longer rear fuselage increases tail moment arm, improving elevator leverage and pitch stability
3. **Protects Control Surfaces** — shields elevator from ground strikes during takeoff, landing, or crashes
4. **Structural / Aesthetic** — replicates full-scale design for aerodynamic continuity
5. **Improves Efficiency** — smoother airflow reduces drag, improves glide ratio, can improve battery life up to ~10%

---

## Open Questions / To-Dos

- [ ] Consult Peter on airfoil selection and sizing calculations
- [ ] Ask Ed about flight controller final choice (ArduPilot vs PX4)
- [ ] Ask Ed: are we hovering or gliding in a circle during data collection? *(Answer: gliding in a circle)*
- [ ] Determine final battery size and weight budget
- [ ] FAA registration (drone will exceed 250g)
- [ ] Solar panel feasibility study (currently: expensive and heavy — CN3791 charging circuit noted)
