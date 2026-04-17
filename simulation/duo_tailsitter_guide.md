# Duo Tailsitter VTOL — Simulation Setup Guide

**Author:** Bao Nguyen
**Project:** URMP — Autonomous Duo Tail-Sitter VTOL
**Date:** 04/17/26

---

## Overview

This guide documents how the `duo_tailsitter` Gazebo simulation model was created and how to run it from scratch. The goal is a functional PX4 SITL simulation of a duo tail-sitter VTOL that can:

- Spawn nose-up (sitting on its tail like a real tailsitter)
- Hover using two wing-tip motors with differential thrust
- Control pitch/roll using two elevon surfaces
- Transition from hover to forward flight via PX4's VTOL tailsitter mode

The model is based on the `standard_vtol` airframe (X8 flying wing) with all quad/pusher motors removed and replaced with two wing-tip motors.

---

## File Structure

All files live inside your PX4-Autopilot clone:

```
PX4-Autopilot/
├── Tools/simulation/gz/models/duo_tailsitter/
│   ├── model.config          ← Gazebo model metadata
│   └── model.sdf             ← Full model definition (physics, visuals, plugins)
│
└── ROMFS/px4fmu_common/init.d-posix/airframes/
    └── 4022_gz_duo_tailsitter  ← PX4 airframe config (control allocation, params)
```

---

## What Was Modified and Why

### 1. Based on `standard_vtol`, not `quadtailsitter`

The `standard_vtol` model uses the X8 flying wing mesh (`x8_wing.dae`) which visually matches the real duo tail-sitter airframe. The `quadtailsitter` uses a compact box body that doesn't match.

### 2. Removed motors: rotors 0–3 and rotor_puller

The `standard_vtol` has 4 quad lift motors + 1 pusher motor for forward flight. All 5 were removed. A real duo tailsitter uses only 2 motors (one per wing) for both hover and forward flight.

### 3. Added 2 wing-tip motors

Two new motors were added at the leading edge of each wing tip:
- **rotor_0** (right wing, CCW): body pose `0.15 -0.9 0.04`
- **rotor_1** (left wing, CW): body pose `0.15 0.9 0.04`

Both use joint axis `1 0 0` (body X = nose direction). When the model spawns nose-up, body X = world Z (up), so the motors thrust upward. Counter-rotating props cancel gyroscopic torque.

### 4. Spawn pose rotated nose-up

The model pose is `0 0 0.4 0 -1.5707963 0` — a -90° pitch rotation that makes body X (nose) point upward in the world. This is how a real tailsitter sits on the ground before takeoff.

### 5. Thickened collision box

The `standard_vtol` collision box was `0.55 × 2.144 × 0.05` (very thin). When standing upright, the thin Z dimension becomes the contact footprint, making the model fall over. Changed to `0.55 × 2.144 × 0.3` so it has a stable base.

### 6. Prop visuals rotated to match motor axis

The iris prop mesh disc is in the XY plane by default (normal = Z). Since the motor axis is X, the prop mesh was rotated `-90°` around Y (`0 0 0 0 -1.5707963 0`) so the disc faces the X direction and looks correct when the model is upright.

### 7. Motor mount visuals added

Two small dark cylinders were added to the base_link to show where the motors are and confirm their orientation is vertical when the model stands upright.

### 8. Kept both elevons from standard_vtol

`servo_0` (left elevon) and `servo_1` (right elevon) were kept exactly as in the standard_vtol. They provide pitch and roll control in both hover and forward flight.

### 9. PX4 airframe configured as duo tailsitter

A new airframe file `4022_gz_duo_tailsitter` was created with:
- `CA_AIRFRAME 4` — tailsitter control allocation
- `CA_ROTOR_COUNT 2` — two motors only
- `VT_TYPE 0` — tailsitter VTOL type
- Disabled failure detector for tailsitter angles (`FD_FAIL_P 180`, `FD_FAIL_R 180`)
- Disabled airspeed check (`CBRK_AIRSPD_CHK 162128`) — no pitot tube in sim
- Disabled RC requirement (`COM_RC_IN_MODE 1`) — allows arming via QGC without a transmitter
- Disabled mag requirement at startup (`EKF2_MAG_TYPE 0`)

---

## Step-by-Step: Reproduce From Scratch

### Prerequisites

- Ubuntu 24.04 Noble
- PX4-Autopilot cloned and built (see `SITL_setup_guide.md`)
- Gazebo Harmonic installed via PX4's `ubuntu.sh`
- QGroundControl AppImage downloaded

---

### Step 1 — Create the model folder

```bash
mkdir -p ~/PX4-Autopilot/Tools/simulation/gz/models/duo_tailsitter
```

---

### Step 2 — Create `model.config`

```bash
nano ~/PX4-Autopilot/Tools/simulation/gz/models/duo_tailsitter/model.config
```

Paste:

```xml
<?xml version="1.0"?>
<model>
  <name>duo_tailsitter</name>
  <version>1.0</version>
  <sdf version="1.10">model.sdf</sdf>
  <description>
    Duo tailsitter VTOL based on standard_vtol X8 flying wing.
    Two wing-tip motors, two elevon control surfaces.
  </description>
</model>
```

---

### Step 3 — Create `model.sdf`

Copy the full SDF from the repo at:
`PX4-Autopilot/Tools/simulation/gz/models/duo_tailsitter/model.sdf`

Key things to understand in the SDF:

**Model spawns nose-up:**
```xml
<model name='duo_tailsitter'>
  <pose>0 0 0.4 0 -1.5707963 0</pose>
```
The `-1.5707963` pitch (-90°) rotates body X (nose) to point up in the world.

**Body collision box is thick so it doesn't fall over:**
```xml
<size>0.55 2.144 0.3</size>
```

**Motor joint axis is body X (thrust = upward when nose-up):**
```xml
<joint name='rotor_0_joint' type='revolute'>
  <axis>
    <xyz>1 0 0</xyz>   <!-- body X = world Z when standing -->
  </axis>
</joint>
```

**Prop visuals rotated to face forward:**
```xml
<visual name='rotor_0_visual'>
  <pose>0 0 0 0 -1.5707963 0</pose>   <!-- rotate disc to face body X -->
```

**Servo topics connect PX4 to elevon joints:**
```xml
<plugin filename="gz-sim-joint-position-controller-system" ...>
  <joint_name>servo_0</joint_name>
  <sub_topic>servo_0</sub_topic>
</plugin>
```

---

### Step 4 — Create the PX4 airframe

```bash
nano ~/PX4-Autopilot/ROMFS/px4fmu_common/init.d-posix/airframes/4022_gz_duo_tailsitter
```

Copy the full airframe from the repo. Make it executable:

```bash
chmod +x ~/PX4-Autopilot/ROMFS/px4fmu_common/init.d-posix/airframes/4022_gz_duo_tailsitter
```

---

### Step 5 — Register the airframe in CMakeLists

```bash
nano ~/PX4-Autopilot/ROMFS/px4fmu_common/init.d-posix/airframes/CMakeLists.txt
```

Find the line `4021_gz_x500_flow` and add the new airframe after it:

```
    4021_gz_x500_flow
    4022_gz_duo_tailsitter
```

---

### Step 6 — Build PX4

```bash
cd ~/PX4-Autopilot
GZ_DISTRO=harmonic make px4_sitl
```

This only rebuilds what changed (fast rebuild since only ROMFS files changed).

---

### Step 7 — Run the simulation

```bash
cd ~/PX4-Autopilot/build/px4_sitl_default/src/modules/simulation/gz_bridge
GZ_DISTRO=harmonic GZ_IP=127.0.0.1 PX4_SIM_MODEL=gz_duo_tailsitter PX4_GZ_SIM_RENDER_ENGINE=ogre ~/PX4-Autopilot/build/px4_sitl_default/bin/px4
```

Wait for:
```
INFO  [gz_bridge] world: default, model: duo_tailsitter_0
INFO  [px4] Startup script returned successfully
```

Gazebo will open automatically showing the duo tailsitter standing nose-up.

---

### Step 8 — Open QGroundControl

```bash
~/Applications/QGroundControl_e39cc090356a6b6f41df88eff8cfbfe9.AppImage &
```

QGC auto-connects over UDP port 14550.

---

### Step 9 — Arm and hover (no RC transmitter)

Since `COM_RC_IN_MODE 1` is set, you can arm without a physical transmitter:

1. In QGC, click the **Arm** button
2. Switch to **Takeoff** mode
3. The drone should spin up motors and hover nose-up

To fly with your RadioMaster Boxer when available:
1. Plug in USB
2. QGC → Q logo → Application Settings → Joystick → Enable and calibrate

---

## Clearing Stale Parameters

If you switch between airframes (e.g., ran `gz_quadtailsitter` before `gz_duo_tailsitter`), PX4 may load cached parameters that conflict. Always wipe the param cache before running a new airframe:

```bash
rm -f ~/PX4-Autopilot/build/px4_sitl_default/src/modules/simulation/gz_bridge/parameters.bson
rm -f ~/PX4-Autopilot/build/px4_sitl_default/src/modules/simulation/gz_bridge/parameters_backup.bson
```

---

## Restarting Cleanly

If Gazebo or PX4 crashes or you need a fresh start:

```bash
# Kill everything
ps aux | grep -E "gz sim|bin/px4|QGround" | grep -v grep | awk '{print $2}' | xargs kill -9 2>/dev/null

# Remove lock files
rm -f /tmp/px4_lock-0 /tmp/px4-sock-0

# Relaunch
cd ~/PX4-Autopilot/build/px4_sitl_default/src/modules/simulation/gz_bridge
GZ_DISTRO=harmonic GZ_IP=127.0.0.1 PX4_SIM_MODEL=gz_duo_tailsitter PX4_GZ_SIM_RENDER_ENGINE=ogre ~/PX4-Autopilot/build/px4_sitl_default/bin/px4
```

---

## Control Allocation Summary

| Output | Function | Gazebo topic |
|--------|----------|--------------|
| Motor 1 | Right wing motor (CCW) | `command/motor_speed` index 0 |
| Motor 2 | Left wing motor (CW) | `command/motor_speed` index 1 |
| Servo 1 | Left elevon | `servo_0` |
| Servo 2 | Right elevon | `servo_1` |

In hover: differential motor thrust controls yaw. Elevons control pitch and roll.
In forward flight: both motors at equal throttle, elevons act as ailerons/elevator.
