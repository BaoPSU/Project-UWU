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
| Thrust-to-weight | ≥2:1 total thrust | Total vertical motor thrust must be at least 2× total weight to handle safe takeoff, hover, and basic maneuvering with margin |
| Motor sizing | 1 motor per 500–1000g (for lift motors) | Balance motor weight and thrust. For endurance, use fewer, larger motors instead of many small ones |
| Prop size | Larger, slower props = more efficient | 10–13" props on 4S–6S systems are common; low pitch helps for stable VTOL and efficient hover |
| Hover power draw | ~60–80% of full throttle current | Plan for 30–40A total current draw during hover for ~2kg drones; don't size ESCs too tight |
| Hover time vs. cruise | Hover burns 4–6× more power | Minimize hover time — just long enough for takeoff/landing. Plan to spend 90%+ of flight in fixed-wing mode. **We are gliding in a circle.** |
| Battery sizing | Use Li-ion for endurance, LiPo for power | Li-ion packs offer higher energy density, but lower current tolerance |
| Center of gravity (CG) | At or slightly ahead of wing's aerodynamic center (for delta planes) | Crucial for smooth transition between hover and forward flight; balance battery, motors, payloads carefully |
| Flight controller | Must support fixed wing aircraft and hovering | ArduPilot and PX4 are top choices; support vertical-to-fixed transitions, auto missions, and fallback modes |
| ESCs | 30–60A with braking, BLHeli_S or fixed-wing ESCs | Pick reliable, smooth throttle ESCs that support brake-on-stop for folding props (reduces drag in cruise) |
| Airframe weight budget | Frame + motors + ESCs should not exceed 35% of total drone | Save weight for battery and payload |

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
| Amazon | 2814 | 1000 KV | 2–4S / unknown | 52g | $42 |
| Amazon | FLASH HOBBY D2830 | 1000 KV | 2–4S / 890g | 52g | $38 |
| Amazon | FLASH HOBBY D3542 | 1000 KV | 2–5S / 1260g | 130g | $46 |
| Amazon | FLASH HOBBY D3542 | 1450 KV | 2–5S / 1260g | 130g | $46 |
| Amazon | FLASH HOBBY D2830 EVO | 1000 KV | 2–4S / 930g | 68.1g | $40 |
| — | FlashHobby D4260 EVO | 600 KV | 5–6S / 2000g | — | — |
| Amazon | D3548 | 790 KV | 3–5S / 1650g | 156g | $40 |
| Amazon | D2836 | 1100 KV | 2–4S / 1130g | 72g | $38 |
| **Amazon** | **D2836** | **850 KV** | **2–4S / 850g** | **72g** | **$38** |

### Motor Test Data

<details>
<summary>D2830 EVO Motor Test Data</summary>

| Item | Volts (V) | Prop | Throttle (%) | Current (A) | Power (W) | Thrust (g) | RPM | Efficiency (g/W) | Temp (°C) |
|------|-----------|------|--------------|-------------|-----------|------------|-----|------------------|-----------|
| D2830-750KV(EVO) | 12.65 | APC 9×6 | 40% | 1.89 | 23.9 | 231 | 4501 | 9.6 | 36.7 |
| | 12.63 | | 50% | 2.51 | 31.7 | 285 | 5014 | 9.0 | |
| | 12.61 | | 60% | 3.12 | 39.3 | 328 | 5463 | 8.3 | |
| | 12.59 | | 70% | 3.71 | 46.7 | 369 | 5836 | 7.9 | |
| | 12.53 | | 80% | 4.80 | 60.1 | 402 | 6358 | 6.7 | |
| | 12.47 | | 90% | 6.23 | 77.7 | 525 | 7020 | 6.8 | |
| | 12.42 | | 100% | 7.08 | 87.9 | 580 | 7278 | 6.6 | |
| D2830-850KV(EVO) | 12.56 | 10×5 | 40% | 2.61 | 32.8 | 272 | 4400 | 8.3 | 38.3 |
| | 12.54 | | 50% | 3.54 | 44.4 | 346 | 4925 | 7.8 | |
| | 12.52 | | 60% | 4.44 | 55.6 | 412 | 5380 | 7.4 | |
| | 12.49 | | 70% | 5.70 | 71.2 | 494 | 5881 | 6.9 | |
| | 12.45 | | 80% | 7.57 | 94.2 | 604 | 6546 | 6.4 | |
| | 12.40 | | 90% | 10.09 | 125.1 | 741 | 7155 | 5.9 | |
| | 12.37 | | 100% | 11.27 | 139.4 | 795 | 7436 | 5.7 | |
| D2830-1000KV(EVO) | 12.31 | 10×5 | 40% | 3.93 | 48.4 | 356 | 4997 | 7.4 | 40.4 |
| | 12.22 | | 50% | 5.30 | 64.8 | 446 | 5585 | 6.9 | |
| | 12.10 | | 60% | 6.69 | 80.9 | 526 | 6056 | 6.5 | |
| | 11.91 | | 70% | 8.74 | 104.1 | 632 | 6611 | 6.1 | |
| | 11.84 | | 80% | 11.38 | 134.7 | 761 | 7272 | 5.6 | |
| | 11.68 | | 90% | 14.85 | 173.4 | 902 | 7876 | 5.2 | |
| | 11.70 | | 100% | 16.80 | 196.6 | 953 | 8199 | 4.8 | |

*Test condition: motor surface temperature at 100% throttle after 15s. Ambient: 26°C*
</details>

<details>
<summary>D2836 EVO Motor Test Data</summary>

| Item | Volts (V) | Prop | Throttle (%) | Current (A) | Power (W) | Thrust (g) | RPM | Efficiency (g/W) | Temp (°C) |
|------|-----------|------|--------------|-------------|-----------|------------|-----|------------------|-----------|
| D2836-750KV(EVO) | 12.46 | APC 9×6 | 40% | 1.95 | 24.3 | 220 | 5441 | 9.1 | 30.8 |
| | 12.45 | | 50% | 2.44 | 30.4 | 260 | 5932 | 8.6 | |
| | 12.44 | | 60% | 2.83 | 35.2 | 288 | 6341 | 8.2 | |
| | 12.43 | | 70% | 3.21 | 39.9 | 318 | 6639 | 8.0 | |
| | 12.41 | | 80% | 3.61 | 44.8 | 346 | 6929 | 7.7 | |
| | 12.39 | | 90% | 4.77 | 59.1 | 424 | 7666 | 7.2 | |
| | 12.38 | | 100% | 4.88 | 60.4 | 425 | 7766 | 7.0 | |
| **D2836-850KV(EVO)** | 16.59 | 10×5 | 40% | 3.46 | 57.4 | 387 | 5102 | 6.7 | 39.3 |
| | 16.56 | | 50% | 4.98 | 82.5 | 494 | 5745 | 6.0 | |
| | 16.52 | | 60% | 6.47 | 106.9 | 611 | 6429 | 5.7 | |
| | 16.44 | | 70% | 8.51 | 139.9 | 736 | 7077 | 5.3 | |
| | 16.34 | | 80% | 11.12 | 181.7 | 875 | 7760 | 4.8 | |
| | 16.19 | | 90% | 14.66 | 237.3 | 1026 | 8409 | 4.3 | |
| | 16.18 | | 100% | 15.11 | 244.5 | 1028 | 8466 | 4.2 | |
| D2836-1100KV(EVO) | 16.26 | APC 9×6 | 40% | 7.72 | 125.5 | 667 | 7637 | 5.3 | 42.4 |
| | 16.17 | | 50% | 10.41 | 168.3 | 794 | 8453 | 4.7 | |
| | 16.07 | | 60% | 13.03 | 209.4 | 912 | 9119 | 4.4 | |
| | 15.84 | | 70% | 15.92 | 252.2 | 956 | 9719 | 3.8 | |
| | 15.63 | | 80% | 20.92 | 327.0 | 1210 | 10617 | 3.7 | |
| | 15.35 | | 90% | 27.22 | 417.8 | 1456 | 11466 | 3.5 | |
| | 15.27 | | 100% | 30.30 | 462.7 | 1623 | 11804 | 3.5 | |

*Test condition: motor surface temperature at 100% throttle after 15s. Ambient: 26°C*
</details>

<details>
<summary>D3542 EVO Motor Test Data</summary>

| Item | Volts (V) | Prop | Throttle (%) | Current (A) | Power (W) | Thrust (g) | RPM | Efficiency (g/W) | Temp (°C) |
|------|-----------|------|--------------|-------------|-----------|------------|-----|------------------|-----------|
| D3542-1000KV(EVO) | 12.47 | 10×5 | 40% | 7.29 | 90.9 | 591 | 6204 | 5.6 | 41.5 |
| | 12.43 | | 50% | 9.69 | 120.4 | 723 | 6871 | 5.4 | |
| | 12.39 | | 60% | 11.97 | 148.3 | 828 | 7409 | 5.2 | |
| | 12.35 | | 70% | 14.16 | 174.9 | 947 | 7865 | 4.8 | |
| | 12.29 | | 80% | 17.53 | 215.4 | 1088 | 8495 | 4.8 | |
| | 12.18 | | 90% | 23.58 | 287.2 | 1346 | 9342 | 4.6 | |
| | 12.13 | | 100% | 26.84 | 325.6 | 1451 | 9703 | 4.4 | |
| D3542-1250KV(EVO) | 12.48 | APC 9×6 | 40% | 10.18 | 127.0 | 639 | 7631 | 5.0 | 44.2 |
| | 12.42 | | 50% | 13.52 | 167.9 | 784 | 8457 | 4.7 | |
| | 12.37 | | 60% | 16.59 | 205.2 | 909 | 9072 | 4.4 | |
| | 12.31 | | 70% | 19.45 | 239.4 | 983 | 9589 | 4.1 | |
| | 12.25 | | 80% | 23.55 | 288.5 | 1097 | 10238 | 3.8 | |
| | 12.10 | | 90% | 30.98 | 374.9 | 1395 | 11177 | 3.7 | |
| | 12.04 | | 100% | 34.89 | 420.1 | 1530 | 11619 | 3.6 | |
| D3542-1400KV(EVO) | 12.33 | APC 9×6 | 40% | 12.92 | 159.3 | 757 | 8159 | 4.8 | 47.5 |
| | 12.26 | | 50% | 17.26 | 211.6 | 926 | 9036 | 4.4 | |
| | 12.19 | | 60% | 21.29 | 259.5 | 1085 | 9721 | 4.2 | |
| | 12.12 | | 70% | 25.24 | 305.9 | 1233 | 10239 | 4.0 | |
| | 12.02 | | 80% | 30.55 | 367.2 | 1408 | 10920 | 3.8 | |
| | 11.85 | | 90% | 39.43 | 467.2 | 1670 | 11878 | 3.6 | |
| | 11.74 | | 100% | 44.78 | 525.7 | 1815 | 12293 | 3.5 | |

*Test condition: motor surface temperature at 100% throttle after 15s. Ambient: 26°C*
</details>

---

## Choosing ESC

We need a 50A ESC with advanced programming features to tune for endurance. Any BLHeli-based ESC will work.

**Options:**
- [BLHeli 50A ESC on Amazon](https://www.amazon.com) — budget option
- [HGLRC Specter 90A 8S BLHeli32](https://www.amazon.com) — high-performance option

---

## Choosing a Flight Controller

We want a PX4-based flight controller.

| Flight Controller | Notes |
|-------------------|-------|
| SpeedyBee F405 WING APP | Limited — non-native VTOL support |
| **Pixhawk 6C** | **Good option. Newer, info harder to find. ~$300** |
| Pixhawk 4 | Good option. Well documented for VTOL. Expensive. |
| Pixhawk 2.4.8 | ~$209 |
| T1 VTOL FX-405 (with GPS & PMU) | Affordable, that's about it |

Required sensors: GPS, barometer

**Setup Videos:**
- [(1/5) PixHawk Video Series - Simple initial setup, config and calibration](https://www.youtube.com)
- [PixHawk/ArduCopter for Beginners (2023): Flashing Pixhawk 6C and basic setup](https://www.youtube.com)

---

## Choosing Controller Electronics

Already have a **RadioMaster Boxer** radio controller. Just need an ELRS receiver.

- **Receiver:** RadioMaster RP3 — longest range ELRS nano RX
- **Module:** RadioMaster Nomad — supports both 2.4GHz and 900MHz

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
