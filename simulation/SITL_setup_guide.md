# Setting Up a PX4 SITL Simulation Environment with QGC and Gazebo

**Mentor:** Joshua Mendez
**Name:** Bao Nguyen
**Date:** 02/19/26
**Course:** URMP

---

## Table of Contents

1. [Purpose of this Guide](#purpose-of-this-guide)
2. [What's MAVLink, PX4, Gazebo, QGC?](#whats-mavlink-px4-gazebo-qgc)
3. [Installation](#installation)
   - [QGroundControl](#qgroundcontrol-installation)
   - [PX4 (includes Gazebo Harmonic)](#installing-px4)
4. [Building PX4 (First Time Only)](#building-px4-first-time-only)
5. [Running the Simulation](#running-the-simulation)
6. [Creating a Custom PX4 Gazebo Drone Model](#creating-a-custom-px4-gazebo-drone-model)
7. [Troubleshooting](#troubleshooting)

---

## Purpose of this Guide

This guide explains how to set up and run a drone simulation using PX4, Gazebo Harmonic, and QGroundControl on Ubuntu 24.04 Noble. It walks through the installation process and shows how the programs connect and work together.

This guide is meant for beginners who want to test and control a drone in a virtual environment without using real hardware.

**Assumed setup:**
- Ubuntu 24.04 (Noble)
- Gazebo Harmonic (gz-harmonic 8.x, installed automatically by PX4's setup script)
- PX4 in Software-In-The-Loop (SITL) mode
- QGroundControl via official AppImage
- RadioMaster Boxer transmitter connected by USB (joystick mode)
- PX4 and QGC communicating locally over UDP on port 14550
- No physical flight controller connected

> **Critical:** PX4's `ubuntu.sh` script installs **Gazebo Harmonic** (`gz-harmonic`), not Jetty. **Do NOT manually install `gz-jetty` or `gz-sim10` before or after running PX4 setup.** Mixing Harmonic and Jetty causes protobuf conflicts and CMake link errors that are very difficult to debug. If both are installed, see [Removing Gazebo Jetty](#removing-gazebo-jetty) below.

---

## What's MAVLink, PX4, Gazebo, QGC?

### PX4

PX4 is open-source flight control software that acts as the brain of the drone. It processes sensor data, runs control algorithms, and sends commands to the motors to keep the drone stable and responsive. PX4 manages flight modes, stabilization, navigation, and safety features.

In simulation, PX4 runs in **Software-In-The-Loop (SITL)** mode, where it controls a virtual drone inside Gazebo instead of real hardware. In real life, the same PX4 software runs on a flight controller such as the Pixhawk 6C — meaning the logic and control behavior in simulation are nearly identical to what runs on the physical drone.

In simple terms:
- **Gazebo** simulates the environment and physics
- **PX4** makes the flight decisions
- **QGroundControl** lets you monitor and configure the system

### QGroundControl

QGroundControl (QGC) is software that lets you both configure and control a drone. It works like a control panel and setup tool combined. You can use it to:
- Set up the drone (calibrate sensors, set parameters, configure flight modes)
- Monitor live data (position, altitude, battery, attitude)
- Plan and upload autonomous missions
- Arm, disarm, and change flight modes
- Control the drone manually using on-screen controls or a connected transmitter

In simulation, QGC connects to PX4 and lets you see everything the drone is doing while also letting you send commands. It is both a configuration tool and a real-time control/monitoring interface — that's why it's called a ground control station.

### Gazebo

Gazebo Harmonic is a robotics simulation software that creates a realistic virtual environment for testing robots and drones. It simulates physics such as gravity, collisions, inertia, and aerodynamics, allowing a drone to behave as if it were flying in the real world.

In a PX4 simulation setup, Gazebo acts as the virtual world — it models the drone's body, motors, sensors, and environment. When PX4 sends motor commands, Gazebo calculates how the drone should move based on physics and updates its position accordingly.

Gazebo is the **virtual test field** where the drone "flies," PX4 acts as the **flight controller**, and QGC acts as the **control interface**.

### MAVLink

MAVLink (Micro Air Vehicle Link) is a lightweight communication protocol used in drones and robotic vehicles to exchange data between a flight controller and a ground control station. It transmits telemetry information such as position, attitude, battery status, and sensor data, as well as control commands like arming, mode changes, and mission uploads.

---

## Installation

### QGroundControl Installation

**Supported OS:** Ubuntu 22.04 and 24.04

#### Step 1 — Enable Serial Port Access

Add your user to the `dialout` group so you can access USB devices without root privileges:

```bash
sudo usermod -aG dialout "$(id -un)"
```

> **Important:** Log out and log back in after running this. Group permissions only apply at login.

#### Step 2 — (Optional) Disable ModemManager

On some Ubuntu systems, ModemManager automatically claims serial devices needed by QGC.

Recommended method (safer — masks but doesn't remove):
```bash
sudo systemctl mask --now ModemManager.service
```

Or completely remove it:
```bash
sudo apt remove --purge modemmanager
```

#### Step 3 — Install Required Dependencies

QGC requires GStreamer for video streaming and several additional libraries:

```bash
sudo apt install gstreamer1.0-plugins-bad gstreamer1.0-libav gstreamer1.0-gl -y
sudo apt install libfuse2 -y
sudo apt install libxcb-xinerama0 libxkbcommon-x11-0 libxcb-cursor-dev -y
```

These provide:
- Video decoding and streaming support
- AppImage runtime support
- Required graphical libraries

#### Step 4 — Download and Run QGroundControl

1. Download `QGroundControl-x86_64.AppImage` from the official QGroundControl website
2. Make it executable:
   ```bash
   chmod +x QGroundControl-x86_64.AppImage
   ```
3. Run it:
   ```bash
   ./QGroundControl-x86_64.AppImage
   ```

QGC should now launch and be ready to connect to PX4 (SITL or hardware such as a Pixhawk 6C).

---

### Installing PX4

PX4's setup script (`ubuntu.sh`) automatically installs **Gazebo Harmonic** and all simulation dependencies. You do not need to install Gazebo separately.

#### Step 1 — Install required tools

```bash
sudo apt update
sudo apt install git python3-pip python3-jinja2 python3-empy \
  python3-toml python3-numpy python3-dev python3-yaml \
  build-essential ninja-build exiftool cmake -y
```

#### Step 2 — Clone PX4

```bash
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
cd PX4-Autopilot
```

#### Step 3 — Run the PX4 setup script

```bash
bash ./Tools/setup/ubuntu.sh
```

This installs all dependencies including **Gazebo Harmonic** (`gz-harmonic`). It may install extra or older packages — this is expected. Close and reopen your terminal when done so group and environment changes take effect.

> **Important:** After this script runs, do **not** install `gz-jetty`, `gz-sim10`, or any version-10+ Gazebo packages. They conflict at runtime with Harmonic even if the build succeeds.

---

## Building PX4 (First Time Only)

The first build compiles all of PX4 and takes several minutes. You only need to do this once (or after source code changes).

```bash
cd ~/PX4-Autopilot
GZ_DISTRO=harmonic make px4_sitl gz_x500
```

> **Why `GZ_DISTRO=harmonic`?** If your system has both `gz-harmonic` and `gz-jetty` installed (which can happen if you previously installed Gazebo manually), CMake picks up Jetty's unversioned libraries and fails with a link error like `gz-sensors::gz-sensors target not found`. Setting `GZ_DISTRO=harmonic` forces all Gazebo targets to use the Harmonic (version 8) libraries that PX4 expects.
>
> If you only have `gz-harmonic` installed, this variable is still safe to set — it does no harm.

After the build completes, you will see `bin/px4` linked. The `make` command also attempts to launch the simulation — you can let it run or kill it with `Ctrl+C`. The important thing is the build completed successfully.

---

## Running the Simulation

After the one-time build, use this method to start the simulation each time. It runs the PX4 binary directly, which is more reliable than `make`.

### Step 1 — Clean up any stale PX4 state (if restarting)

If PX4 exited abnormally in a previous session, remove its lock files first:

```bash
pkill -9 -f "bin/px4" 2>/dev/null; rm -f /tmp/px4_lock-0 /tmp/px4-sock-0
```

### Step 2 — Launch PX4 with Gazebo

Open a terminal and run:

```bash
cd ~/PX4-Autopilot/build/px4_sitl_default/src/modules/simulation/gz_bridge
GZ_DISTRO=harmonic \
  GZ_IP=127.0.0.1 \
  PX4_SIM_MODEL=gz_x500 \
  PX4_GZ_SIM_RENDER_ENGINE=ogre \
  ~/PX4-Autopilot/build/px4_sitl_default/bin/px4
```

Wait for output like:
```
INFO  [init] Gazebo simulator 8.11.0
INFO  [init] Setting Gazebo render engine to 'ogre'!
INFO  [gz_bridge] world: default, model: x500_0
INFO  [mavlink] mode: Normal, data rate: 4000000 B/s on udp port 18570 remote port 14550
INFO  [px4] Startup script returned successfully
```

A Gazebo window should open showing the X500 quadcopter on the ground.

> **Why `PX4_GZ_SIM_RENDER_ENGINE=ogre`?** Without this flag, Gazebo uses the OGRE2 renderer by default. On many systems (especially those without a dedicated GPU or with certain GPU drivers), OGRE2 crashes the Gazebo GUI with a segmentation fault (SIGSEGV). Setting this flag forces OGRE1, which is more compatible. If your Gazebo window opens successfully without this flag, you don't need it.

> **Why run the binary directly instead of `make px4_sitl gz_x500`?** Using `make` re-triggers cmake checks and rebuilds, which is slow. Running the binary directly is faster and avoids lock conflicts from partially-started simulations.

### Step 3 — Launch QGroundControl

In a new terminal:

```bash
~/Applications/QGroundControl-x86_64.AppImage
```

(Adjust the path to wherever you saved the AppImage.)

QGC should auto-connect to PX4 over UDP port 14550 within a few seconds. You will see the X500 drone appear on the map.

### Step 4 — Fly the Drone

In QGroundControl:
1. Check that the drone shows as connected (green heartbeat indicator)
2. Click **Arm** (or use your RC transmitter)
3. Raise throttle to take off

---

## Creating a Custom PX4 Gazebo Drone Model

The goal is to:
1. Open the PX4 models folder
2. Copy an existing model
3. Rename it
4. Edit it
5. Spawn it in Gazebo

### Step 1 — Open a Terminal

Press `Ctrl + Alt + T`

### Step 2 — Navigate to the PX4 Models Folder

```bash
cd ~/PX4-Autopilot/Tools/simulation/gz/models
ls
```

You should see models like `x500`, `standard_vtol`, `tiltrotor`, `quadrotor`, etc.

### Step 3 — Copy an Existing Model

Copy `standard_vtol` as a starting point (already close to what we need):

```bash
cp -r standard_vtol my_vtol_drone
ls
```

You should now see `my_vtol_drone` alongside the original models.

### Step 4 — Enter Your Model Folder

```bash
cd my_vtol_drone
ls
```

You should see:
```
model.config
model.sdf
meshes/
```

### Step 5 — Rename the Model

```bash
nano model.config
```

Find this line:
```xml
<name>standard_vtol</name>
```

Change it to:
```xml
<name>my_vtol_drone</name>
```

### Step 6 — Save and Exit

- Save: `Ctrl + O`, then `Enter`
- Exit: `Ctrl + X`

### Step 7 — Start PX4 Using Your Model

```bash
cd ~/PX4-Autopilot/build/px4_sitl_default/src/modules/simulation/gz_bridge
GZ_DISTRO=harmonic \
  GZ_IP=127.0.0.1 \
  PX4_SIM_MODEL=gz_standard_vtol \
  PX4_GZ_MODEL_NAME=my_vtol_drone \
  PX4_GZ_SIM_RENDER_ENGINE=ogre \
  ~/PX4-Autopilot/build/px4_sitl_default/bin/px4
```

This tells PX4 to use the standard VTOL airframe config but control the model named `my_vtol_drone`.

### Step 8 — Spawn Your Model in Gazebo (if not auto-spawned)

If the drone does not appear automatically, open a second terminal and run:

```bash
gz service -s /world/default/create \
  --reqtype gz.msgs.EntityFactory \
  --reptype gz.msgs.Boolean \
  --timeout 3000 \
  --req 'sdf_filename: "/home/bao/PX4-Autopilot/Tools/simulation/gz/models/my_vtol_drone/model.sdf", name: "my_vtol_drone"'
```

Your drone should now appear in Gazebo.

### Why the `gz_my_vtol_drone` Error Happens

If you try:
```bash
make px4_sitl gz_my_vtol_drone
```

You will see:
```
ninja: error: unknown target 'gz_my_vtol_drone'
```

This is because PX4 launch targets only exist for **registered airframes**. Your model is a Gazebo object but not yet a registered PX4 airframe. The workaround above (using `PX4_GZ_MODEL_NAME`) lets PX4 control a custom model without registering a full airframe.

---

## Troubleshooting

### Gazebo crashes with SIGSEGV / segmentation fault

This is almost always the OGRE2 renderer. Make sure you include `PX4_GZ_SIM_RENDER_ENGINE=ogre` in your run command.

### "PX4 server already running for instance 0"

A previous PX4 run left a lock file. Fix:

```bash
pkill -9 -f "bin/px4"
rm -f /tmp/px4_lock-0 /tmp/px4-sock-0
```

Then retry.

### CMake error: `gz-sensors::gz-sensors` target not found

You have both gz-harmonic and gz-jetty installed. Either:
1. Always set `GZ_DISTRO=harmonic` in all build commands, **and** remove Jetty (see below)
2. Or remove Jetty entirely

### Removing Gazebo Jetty

If you accidentally installed Jetty packages, remove them and restore Harmonic:

```bash
sudo apt remove gz-sim10-cli gz-sim10-server gz-transport15-cli \
  libgz-sim10 libgz-transport15 libgz-msgs12 \
  libgz-physics9 libgz-plugin4 libgz-math9 libgz-fuel-tools9

sudo apt install gz-harmonic
```

Verify the correct version is running:

```bash
gz sim --version
```

Output should show `8.x.x` (Harmonic), not `10.x.x` (Jetty).

### Gazebo opens but drone is invisible / window is blank

Try bringing the window to the front:

```bash
xdotool search --name "Gazebo" windowraise
```

If Gazebo is running but the window never appears, kill it and restart:

```bash
pkill -f "gz sim"; pkill -f "ruby.*gz"
```

Then restart PX4 from Step 2 above.

### QGC doesn't connect to PX4

- Make sure PX4 started successfully (look for the `INFO [mavlink]` line in terminal output)
- PX4 listens on UDP port 18570 and sends to QGC on port 14550
- QGC should auto-detect the connection — no manual configuration needed on localhost
- If QGC was open before PX4 started, it may need a moment to detect the new connection

### Build fails partway through

If the build stops mid-way with a link error, try a clean build:

```bash
cd ~/PX4-Autopilot
rm -rf build/
GZ_DISTRO=harmonic make px4_sitl gz_x500
```

Note: a clean build takes significantly longer than an incremental one.
