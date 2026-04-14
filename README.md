# Project UWU — Unmanned Wide-area Uplink

**An open-source delta wing VTOL drone platform for autonomous wide-area sensing.**

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Platform: PX4](https://img.shields.io/badge/Platform-PX4-blue.svg)
![Status: In Development](https://img.shields.io/badge/Status-In%20Development-yellow.svg)

---

## Overview

Project UWU is an open-source duo tail-sitter VTOL (Vertical Take-Off and Landing) drone platform designed for autonomous field deployment. It combines the endurance of fixed-wing flight with the convenience of vertical takeoff and landing — no runway required.

| Spec | Value |
|------|-------|
| Airframe | Delta wing foam, ~1200mm wingspan |
| Configuration | Duo tail-sitter (twin motor, nose-up hover) |
| Motors | One per leading edge of each wing |
| Control Surfaces | 2 (elevons) |
| Autopilot | Pixhawk 6C running PX4 |
| Target Flight Time | 1–2 hours |
| Payload Capacity | 250g+ sensor payload |
| Navigation | Autonomous GPS waypoint (QGroundControl) |

---

## Features

- Vertical takeoff and landing — no launcher or catch system needed
- Hover in place (nose-up) with full 6-DOF control: translate left/right/forward/back, rotate
- Autonomous GPS waypoint missions via QGroundControl click-to-fly
- Manual override for safe testing and development
- Designed for multi-drone expansion (fleet autonomy)

---

## Repository Structure

```
Project-UWU/
├── docs/               # Research notes, wiring diagrams, setup guides
├── firmware/
│   ├── px4-config/     # Airframe configs, mixer files, PX4 customizations
│   ├── params/         # PX4 parameter files (.params)
│   └── mixers/         # Custom mixer definitions
├── hardware/
│   ├── airframe/       # Physical build notes, foam cutting templates
│   ├── electronics/    # Wiring schematics, component specs, BOM
│   └── CAD/            # CAD files for mounts, payload bay, etc.
├── simulation/
│   ├── gazebo/         # Gazebo world/model files for SITL
│   └── scripts/        # Launch scripts for PX4 SITL
├── gcs/                # QGroundControl mission files, config exports
├── media/              # Photos, videos, flight logs
├── LICENSE
└── README.md
```

---

## Getting Started

### Prerequisites

- [PX4 Autopilot](https://docs.px4.io/main/en/dev_setup/dev_env.html) (v1.14+)
- [QGroundControl](http://qgroundcontrol.com/) (daily build recommended for VTOL)
- [Gazebo](https://gazebosim.org/) (for SITL simulation)
- Pixhawk 6C flight controller

### Hardware Setup

1. See [`hardware/electronics/`](hardware/electronics/) for wiring diagram and BOM.
2. Flash PX4 firmware to the Pixhawk 6C.
3. Load the airframe config from [`firmware/px4-config/`](firmware/px4-config/).
4. Load parameters from [`firmware/params/`](firmware/params/).
5. Calibrate sensors, radio, and actuators in QGroundControl.

### Simulation (SITL)

```bash
# Clone PX4-Autopilot
git clone https://github.com/PX4/PX4-Autopilot.git --recursive

# Launch SITL with Gazebo (tail-sitter model)
cd PX4-Autopilot
make px4_sitl gazebo-classic_tiltrotor  # replace with UWU model once configured
```

See [`simulation/`](simulation/) for UWU-specific Gazebo models and launch scripts.

---

## Flight Modes

| Mode | Description |
|------|-------------|
| MANUAL | Direct RC control |
| STABILIZED | Attitude stabilized, manual throttle |
| ALTCTL | Altitude hold |
| POSCTL | GPS position hold |
| MISSION | Autonomous waypoint flight |
| TAKEOFF | Automated vertical takeoff |
| LAND | Automated vertical landing |

---

## Contributing

Contributions are welcome. Please open an issue before submitting a pull request for major changes.

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

## Project Status

This project is under active development as part of a URMP (Undergraduate Research and Mentoring Program) research initiative. Expect frequent updates.
