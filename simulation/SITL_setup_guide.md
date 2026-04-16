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
4. [Creating a Custom PX4 Gazebo Drone Model](#creating-a-custom-px4-gazebo-drone-model)
5. [Running Your Custom PX4 Drone Model](#running-your-custom-px4-drone-model)

---

## Purpose of this Guide

This guide explains how to set up and run a drone simulation using PX4, Gazebo Harmonic, and QGroundControl on Ubuntu 24.04. It walks through the installation process and shows how the programs connect and work together.

This guide is meant for beginners who want to test and control a drone in a virtual environment without using real hardware.

**Assumed setup:**
- Ubuntu 24.04 (Noble)
- Gazebo Harmonic (gz-harmonic, installed automatically by PX4's setup script)
- PX4 in Software-In-The-Loop (SITL) mode
- QGroundControl via official AppImage
- RadioMaster Boxer transmitter connected by USB (joystick mode)
- PX4 and QGC communicating locally over UDP on port 14550
- No physical flight controller connected

> **Note:** PX4's `ubuntu.sh` script installs **Gazebo Harmonic** (`gz-harmonic`), not Jetty. Do not manually install `gz-jetty` before running PX4 setup — mixing Harmonic and Jetty libraries causes CMake link errors. If both are installed, set `GZ_DISTRO=harmonic` in your build command.

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

#### Step 4 — Launch the simulation (X500 quadcopter test)

```bash
cd ~/PX4-Autopilot
GZ_DISTRO=harmonic make px4_sitl gz_x500
```

> **Why `GZ_DISTRO=harmonic`?** If your system has both `gz-harmonic` and `gz-jetty` installed (which can happen if you previously installed Gazebo manually), CMake picks up Jetty's unversioned libraries and fails with a link error. Setting `GZ_DISTRO=harmonic` forces all Gazebo targets to use the Harmonic (version 8) libraries that PX4 expects.
>
> If you only have `gz-harmonic` installed, this variable is still safe to set — it does no harm.

This command will build PX4, launch Gazebo Harmonic, and spawn the X500 quadcopter. The first build takes several minutes.

#### Step 5 — Open QGroundControl

In a new terminal:

```bash
./QGroundControl-x86_64.AppImage
```

QGC should auto-connect to PX4 over UDP port 14550.

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

### Step 7 — Start PX4 with a Working Model

```bash
cd ~/PX4-Autopilot
GZ_DISTRO=harmonic make px4_sitl gz_x500
```

This starts PX4, Gazebo, and a default quadcopter. Leave this terminal running.

### Step 8 — Open a Second Terminal and Spawn Your Model

Press `Ctrl + Alt + T`, then run:

```bash
gz service -s /world/default/create \
  --reqtype gz.msgs.EntityFactory \
  --reptype gz.msgs.Boolean \
  --timeout 1000 \
  --req 'sdf_filename: "/home/bao/PX4-Autopilot/Tools/simulation/gz/models/my_vtol_drone/model.sdf"'
```

Your drone should appear inside Gazebo.

### Why the `gz_my_vtol_drone` Error Happens

If you see:
```
ninja: error: unknown target 'gz_my_vtol_drone'
```

This is because PX4 launch commands only work for **registered airframe targets**, such as:
```bash
GZ_DISTRO=harmonic make px4_sitl gz_x500
GZ_DISTRO=harmonic make px4_sitl gz_standard_vtol
GZ_DISTRO=harmonic make px4_sitl gz_tiltrotor
```

Your model is currently only a Gazebo object — PX4 doesn't know how to launch it automatically yet. Later we can create a custom PX4 airframe, which will allow:
```bash
GZ_DISTRO=harmonic make px4_sitl gz_my_vtol_drone
```

### Quick Check

Verify your model folder exists:
```bash
ls ~/PX4-Autopilot/Tools/simulation/gz/models/my_vtol_drone
```

You should see:
```
model.config
model.sdf
meshes/
```

---

## Running Your Custom PX4 Drone Model

**Goal:** Run Gazebo and PX4 so that only `my_vtol_drone` is controlled, QGC connects normally, and the drone can be flown.

### Step 1 — Start PX4 Using Your Model

```bash
cd ~/PX4-Autopilot
PX4_GZ_MODEL_NAME=my_vtol_drone GZ_DISTRO=harmonic make px4_sitl gz_standard_vtol
```

This tells PX4 to control the model named `my_vtol_drone`. If no drone appears immediately, that is normal — proceed to step 2.

### Step 2 — Spawn Your Model

Open a second terminal:

```bash
gz service -s /world/default/create \
  --reqtype gz.msgs.EntityFactory \
  --reptype gz.msgs.Boolean \
  --timeout 3000 \
  --req 'sdf_filename: "/home/bao/PX4-Autopilot/Tools/simulation/gz/models/my_vtol_drone/model.sdf", name: "my_vtol_drone"'
```

Your drone should now appear in Gazebo.

### Step 3 — Open QGroundControl

Launch it normally (double-click the AppImage or run from terminal):

```bash
./QGroundControl-x86_64.AppImage
```

### Step 4 — Fly the Drone

In QGroundControl:
1. Click **Arm**
2. Raise throttle
3. The drone should lift off
