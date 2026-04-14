# 🤖 ProtoBot

<div align="center">

![ProtoBot Banner](https://img.shields.io/badge/ProtoBot-Modular%20Research%20Robot-blue?style=for-the-badge&logo=ros&logoColor=white)

[![ROS2](https://img.shields.io/badge/ROS2-Jazzy-22314E?style=flat-square&logo=ros&logoColor=white)](https://docs.ros.org/en/jazzy/)
[![STM32](https://img.shields.io/badge/STM32-N657X0--Q-03234B?style=flat-square&logo=stmicroelectronics&logoColor=white)](https://www.st.com/)
[![Raspberry Pi](https://img.shields.io/badge/Raspberry%20Pi-5-A22846?style=flat-square&logo=raspberrypi&logoColor=white)](https://www.raspberrypi.com/)
[![micro-ROS](https://img.shields.io/badge/micro--ROS-Enabled-6D28D9?style=flat-square)](https://micro.ros.org/)
[![License](https://img.shields.io/badge/License-ProtoBot%20Custom-orange?style=flat-square)](./LICENSE)
[![Status](https://img.shields.io/badge/Status-Active%20Development-brightgreen?style=flat-square)]()

**An open modular robotics platform for research, education, and experimentation — powered by ROS 2 and real-time embedded control.**

[Overview](#-overview) · [Architecture](#-system-architecture) · [Hardware](#-hardware) · [Software Stack](#-software-stack) · [Getting Started](#-getting-started) · [Applications](#-protobot-applications) · [Modules](#-modular-system) · [Contributing](#-contributing) · [License](#-license)

</div>

---

## Overview

**ProtoBot** is an open, modular robotic platform designed to bridge the gap between expensive commercial research robots and accessible, hackable hardware. Built around a **dual-brain architecture** — a **Raspberry Pi 5** running ROS 2 as the high-level cognitive layer, and an **STM32 Nucleo N657X0-Q** running an RTOS as the real-time embedded controller — ProtoBot is engineered to be reconfigured, extended, and repurposed with minimal effort.

Whether you are exploring SLAM, navigation, computer vision, manipulator control, multi-robot coordination, or embedded systems research, ProtoBot gives you a unified, documented, and community-supported platform to do it on.

### Key Goals

- **Modularity first** — swap sensors, actuators, and compute modules without redesigning the base platform
- **Research-grade** — suitable for academic research, lab environments, and serious experimentation
- **Accessible** — significantly lower cost barrier than commercial equivalents (e.g., TurtleBot 4, Spot, Stretch)
- **Full-stack** — from bare-metal firmware to ROS 2 nodes to desktop/mobile applications
- **Open recreation** — free to build for personal or educational use, with attribution required

---

## System Architecture

ProtoBot follows a **hierarchical dual-processor architecture** inspired by modern collaborative robotics systems:

```
┌─────────────────────────────────────────────────────────┐
│                    HIGH-LEVEL LAYER                     │
│                   Raspberry Pi 5                        │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │
│  │  ROS 2   │  │  Nav2    │  │  OpenCV  │  │  Apps  │  │
│  │  Jazzy   │  │  Stack   │  │  Vision  │  │  Layer │  │
│  └──────────┘  └──────────┘  └──────────┘  └────────┘  │
│         │                                               │
│   micro-ROS Agent (UART / USB / SPI)                    │
└─────────────────────┬───────────────────────────────────┘
                      │  Serial / USB CDC / SPI
┌─────────────────────▼───────────────────────────────────┐
│                 REAL-TIME LAYER                         │
│             STM32 Nucleo N657X0-Q                       │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │
│  │ micro-ROS│  │ FreeRTOS │  │ Motor    │  │ Sensor │  │
│  │  Client  │  │  / Zephyr│  │ Control  │  │  HAL   │  │
│  └──────────┘  └──────────┘  └──────────┘  └────────┘  │
│                                                         │
│  PWM · Encoders · IMU · LIDAR · Servo · CAN · I2C      │
└─────────────────────────────────────────────────────────┘
```

### Communication Protocol Stack

| Layer | Protocol | Purpose | Typical Speed |
|-------|----------|---------|---------------|
| ROS 2 ↔ micro-ROS | UART / USB CDC | Topic & service bridging | 4 Mbps |
| STM32 ↔ IMU | SPI / I2C | Inertial sensor data | 10 MHz SPI |
| STM32 ↔ Motor Driver | PWM + GPIO | Speed & direction control | ~20 kHz PWM |
| STM32 ↔ Encoders | Quadrature / Timer | Position feedback | Hardware timer |
| STM32 ↔ LIDAR | UART / SPI | Ranging data | 115200 baud |
| RPi5 ↔ Camera | CSI / USB | Vision input | 60 fps / 4K |
| RPi5 ↔ PC/Mobile | WiFi / Ethernet | App communication | 100+ Mbps |

---

##  Hardware

### Core Compute

| Component | Model | Role |
|-----------|-------|------|
| High-Level SBC | Raspberry Pi 5 (8 GB) | ROS 2, Navigation, Vision, App Server |
| Real-Time MCU | STM32 Nucleo N657X0-Q | Motor control, sensor fusion, RTOS tasks |

### STM32 N657X0-Q Highlights

- **ARM Cortex-M55** with Helium DSP/ML extensions
- Up to **800 MHz** core clock
- **4 MB Flash**, **2.9 MB SRAM**
- Hardware FPU — ideal for PID loops and sensor fusion
- Multiple UARTs, SPI, I2C, CAN FD, USB FS/HS
- ETM/SWV trace support for real-time debugging

### Base Hardware Bill of Materials (BOM)

| # | Component | Qty | Notes |
|---|-----------|-----|-------|
| 1 | Raspberry Pi 5 (8 GB) | 1 | Main compute |
| 2 | STM32 Nucleo N657X0-Q | 1 | Embedded controller |
| 3 | DC Gearmotor with encoder | 2–4 | Differential / mecanum drive |
| 4 | L298N or DRV8833 Motor Driver | 1–2 | PWM motor control |
| 5 | IMU (MPU-6050 / ICM-42688) | 1 | Inertial sensing |
| 6 | LIDAR (RPLIDAR A1/A2/C1) | 1 (optional) | 2D/3D mapping |
| 7 | Depth Camera (Intel RealSense / OAK-D) | 1 (optional) | 3D perception |
| 8 | LiPo Battery 11.1V / 5000 mAh | 1 | Main power |
| 9 | 5V / 5A Buck Converter | 1 | RPi5 power rail |
| 10 | 3.3V LDO | 1 | STM32 auxiliary |
| 11 | Chassis (custom / 3D printed) | 1 | Modular frame |
| 12 | OLED Display 128x64 (optional) | 1 | Local status display |

>  A full Mouser/DigiKey BOM with part numbers is available in `/hardware/BOM.xlsx`

### Expansion Modules

ProtoBot supports hot-swappable physical modules attached to a standardized mounting rail and connector interface:

| Module | Description |
|--------|-------------|
| `MOD-ARM` | 4 or 6 DOF robotic arm attachment |
| `MOD-VISION` | Stereo camera + depth sensor boom |
| `MOD-LIDAR` | LIDAR tower with servo pan |
| `MOD-GRIPPER` | Servo-actuated parallel gripper |
| `MOD-IMU-EXT` | High-precision 9-DOF IMU expansion |
| `MOD-AUDIO` | Microphone array + speaker for HRI |
| `MOD-MECANUM` | Mecanum wheel kit for omnidirectional drive |
| `MOD-TRAIL` | Differential trail-following sensor array |

---

## Software Stack

### High-Level (Raspberry Pi 5)

```
OS:       Ubuntu Server 24.04 LTS (64-bit ARM)
ROS:      ROS 2 Jazzy Jalisco
Nav:      Nav2 (navigation2)
SLAM:     slam_toolbox / Cartographer
Vision:   OpenCV 4.x, MediaPipe, Intel RealSense SDK
Bridge:   micro-ROS Agent (serial transport)
Apps:     ProtoBot Desktop App (Electron/React)
          ProtoBot Mobile App (Flutter)
```

### Real-Time (STM32 N657X0-Q)

```
RTOS:     FreeRTOS (primary) / Zephyr RTOS (optional)
Framework:micro-ROS client (via STM32CubeIDE)
HAL:      STM32 HAL + LL drivers
IDE:      STM32CubeIDE + STM32CubeMX
Debug:    SWD + ST-LINK V3 / OpenOCD
```

### ROS 2 Package Structure

```
protobot_ws/
├── src/
│   ├── protobot_bringup/        # Launch files & system startup
│   ├── protobot_description/    # URDF / Xacro robot model
│   ├── protobot_hardware/       # Hardware interface (ros2_control)
│   ├── protobot_navigation/     # Nav2 configuration & maps
│   ├── protobot_slam/           # SLAM configuration
│   ├── protobot_vision/         # Computer vision nodes
│   ├── protobot_teleop/         # Teleoperation nodes
│   ├── protobot_diagnostics/    # System health monitoring
│   └── protobot_msgs/           # Custom ROS 2 messages & services
├── firmware/
│   ├── stm32_protobot/          # STM32CubeIDE project
│   │   ├── Core/
│   │   ├── Middlewares/         # FreeRTOS, micro-ROS
│   │   └── Drivers/
├── hardware/
│   ├── schematics/              # KiCad schematics
│   ├── pcb/                     # Custom PCB layouts
│   ├── models/                  # 3D printable STL/STEP files
│   └── BOM.xlsx
├── apps/
│   ├── desktop/                 # ProtoBot Desktop App
│   └── mobile/                  # ProtoBot Mobile App
└── docs/
    ├── guides/
    └── api/
```

---

## Getting Started

### Prerequisites

- Raspberry Pi 5 with Ubuntu 24.04 flashed (via Raspberry Pi Imager)
- STM32 Nucleo N657X0-Q board + ST-LINK V3
- STM32CubeIDE installed on your development machine
- Git, Docker (optional, for micro-ROS agent)

### 1. Set Up the Raspberry Pi 5

```bash
# Flash Ubuntu 24.04 Server to SD card / NVMe
# Then SSH into the Pi and run:

# Install ROS 2 Jazzy
sudo apt update && sudo apt install -y software-properties-common
sudo add-apt-repository universe
sudo apt update
curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
  http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" \
  | sudo tee /etc/apt/sources.list.d/ros2.list
sudo apt update
sudo apt install -y ros-jazzy-desktop ros-jazzy-nav2-bringup \
  ros-jazzy-slam-toolbox ros-jazzy-ros2-control \
  ros-jazzy-ros2-controllers
source /opt/ros/jazzy/setup.bash
```

### 2. Clone ProtoBot Workspace

```bash
mkdir -p ~/protobot_ws/src && cd ~/protobot_ws/src
git clone https://github.com/YOUR_USERNAME/protobot.git .
cd ~/protobot_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
```

### 3. Start the micro-ROS Agent

```bash
# Via Docker (easiest)
docker run -it --rm -v /dev:/dev --privileged --net=host \
  microros/micro-ros-agent:jazzy serial \
  --dev /dev/ttyAMA0 -b 4000000

# Or natively
sudo apt install ros-jazzy-micro-ros-agent
ros2 run micro_ros_agent micro_ros_agent serial \
  --dev /dev/ttyAMA0 --baud 4000000
```

### 4. Flash the STM32 Firmware

```bash
# Open firmware/stm32_protobot/ in STM32CubeIDE
# Connect ST-LINK V3 to the Nucleo board
# Build and Flash via CubeIDE or:
openocd -f interface/stlink.cfg \
        -f target/stm32n6x.cfg \
        -c "program build/stm32_protobot.elf verify reset exit"
```

### 5. Launch ProtoBot

```bash
# Full system bringup
ros2 launch protobot_bringup protobot.launch.py

# With SLAM
ros2 launch protobot_bringup protobot.launch.py \
  slam:=true use_rviz:=true

# Teleoperation only
ros2 launch protobot_teleop keyboard_teleop.launch.py
```

---

## ProtoBot Applications

ProtoBot ships with a first-party application ecosystem designed to make robot programming and monitoring accessible to all skill levels.

### ProtoBot Desktop App

> Built with **Electron + React + TypeScript**

| Feature | Description |
|---------|-------------|
| Real-time Dashboard | Live topic data, sensor feeds, battery, diagnostics |
| Node Graph Viewer | Visual ROS 2 computation graph |
| Map Editor | Interactive occupancy grid editing |
| Mission Planner | Drag-and-drop waypoint navigation |
| Code IDE | Browser-based Python/C++ editor with robot context |
| Log Viewer | Filtered, searchable ROS 2 log stream |
| Parameter Tuner | Live PID and Nav2 parameter adjustment |
| Module Manager | Detect and configure attached hardware modules |

```bash
cd apps/desktop
npm install
npm run dev        # Development
npm run build      # Production build
```

### ProtoBot Mobile App

> Built with **Flutter (iOS + Android)**

| Feature | Description |
|---------|-------------|
| Joystick Control | Virtual joystick for manual teleoperation |
| Camera Stream | Live video feed from robot cameras |
| Robot Status | Battery, pose, active nodes at a glance |
| Mission Monitor | Track autonomous navigation missions |
| Push Notifications | Alerts for task completion or hardware faults |
| Bluetooth Pairing | Initial setup without network configuration |

```bash
cd apps/mobile
flutter pub get
flutter run         # Run on connected device
flutter build apk   # Android release build
flutter build ios   # iOS release build
```

### ProtoBot API

ProtoBot exposes a REST + WebSocket API from the RPi5 for third-party integrations:

```
Base URL: http://protobot.local:8080/api/v1

GET  /status              → System health, battery, uptime
GET  /topics              → Active ROS 2 topics list
POST /cmd_vel             → Send velocity commands
GET  /map                 → Current occupancy grid (PNG/PGM)
POST /navigate            → Send navigation goal (x, y, theta)
WS   /stream/camera       → Live MJPEG camera stream
WS   /stream/telemetry    → Real-time sensor telemetry
```

---

##  Modular System

ProtoBot's modularity operates at three levels:

### Level 1 — Physical Modules
Standard mounting slots on the chassis frame accept bolt-on hardware modules. Each module exposes a standard 12-pin Molex connector carrying power (5V/12V), UART, I2C, and GPIO.

### Level 2 — Firmware Modules
The STM32 firmware uses a task-based RTOS architecture. Each hardware module corresponds to a FreeRTOS task that can be enabled or disabled via compile flags or dynamic configuration:

```c
// protobot_config.h — enable/disable modules
#define PROTOBOT_MOD_ENCODER    1
#define PROTOBOT_MOD_IMU        1
#define PROTOBOT_MOD_LIDAR      0   // Disabled if not attached
#define PROTOBOT_MOD_ARM        0
#define PROTOBOT_MOD_GRIPPER    0
```

### Level 3 — ROS 2 Modules
Each hardware module maps to a ROS 2 launch argument and set of nodes. Modules self-announce via `/protobot/module_registry` topic at startup:

```bash
# Launch with specific modules
ros2 launch protobot_bringup protobot.launch.py \
  enable_lidar:=true \
  enable_arm:=true \
  enable_depth_camera:=true
```

---

## Supported Research Use Cases

ProtoBot has been designed to support the following research and educational domains out of the box:

| Domain | Supported Capabilities |
|--------|----------------------|
| Mobile Robotics | Differential drive, mecanum, odometry, SLAM |
| Computer Vision | Object detection, visual SLAM, depth estimation |
| Human-Robot Interaction | Voice I/O, gesture recognition, face tracking |
| Manipulation | Arm control, grasp planning, workspace mapping |
| Multi-Robot Systems | ROS 2 namespacing, fleet coordination |
| Embedded Systems | RTOS task analysis, real-time profiling |
| Machine Learning | Edge inference with RPi5 NPU and Helium extensions |
| Sensor Fusion | IMU + odometry + LIDAR EKF integration |

---

## ROS 2 Topics Reference

| Topic | Type | Direction | Description |
|-------|------|-----------|-------------|
| `/cmd_vel` | `geometry_msgs/Twist` | → STM32 | Velocity command |
| `/odom` | `nav_msgs/Odometry` | ← STM32 | Wheel odometry |
| `/imu/data` | `sensor_msgs/Imu` | ← STM32 | IMU data |
| `/scan` | `sensor_msgs/LaserScan` | ← STM32/RPi | LIDAR scan |
| `/battery_state` | `sensor_msgs/BatteryState` | ← STM32 | Battery telemetry |
| `/joint_states` | `sensor_msgs/JointState` | ↔ Both | Joint positions |
| `/camera/image_raw` | `sensor_msgs/Image` | ← RPi5 | Camera feed |
| `/protobot/module_registry` | `protobot_msgs/ModuleList` | ← STM32 | Attached modules |
| `/protobot/diagnostics` | `diagnostic_msgs/DiagnosticArray` | ← Both | System diagnostics |

---

## Development Guide

### Setting Up a Development Environment (Host PC)

```bash
# Install ROS 2 Jazzy on Ubuntu 24.04
# (same steps as RPi5, above)

# Install simulation dependencies
sudo apt install -y ros-jazzy-gazebo-ros-pkgs \
  ros-jazzy-rviz2 ros-jazzy-rqt*

# Run in simulation (no hardware needed)
ros2 launch protobot_bringup protobot_sim.launch.py
```

### Firmware Development Workflow

```
[Write C code in STM32CubeIDE]
        ↓
[Build → Flash via ST-LINK]
        ↓
[Monitor via SWV / UART console]
        ↓
[Verify ROS 2 topics on RPi5]
        ↓
[Tune PID / RTOS task timing]
```

### Adding a New Hardware Module

1. Design the physical mount (STL in `/hardware/models/`)
2. Wire to the 12-pin standard connector
3. Add a FreeRTOS task in `/firmware/stm32_protobot/Modules/`
4. Add micro-ROS publisher/subscriber for the new data
5. Create a ROS 2 package in `src/protobot_YOUR_MODULE/`
6. Register the module in `protobot_msgs/ModuleDescriptor`
7. Add launch argument to `protobot.launch.py`
8. Document in `/docs/modules/YOUR_MODULE.md`

---

## Contributing

Contributions are warmly welcome! This project thrives on community expertise.

### How to Contribute

1. **Fork** the repository
2. **Create** a feature branch: `git checkout -b feature/my-new-module`
3. **Commit** changes with clear messages
4. **Test** on real hardware or simulation
5. **Open a Pull Request** with description, photos/videos if applicable

### Contribution Guidelines

- Follow ROS 2 coding style for Python (`ruff`) and C++ (`clang-format`)
- STM32 firmware follows MISRA-C guidelines where applicable
- All new modules must include documentation in `/docs/modules/`
- Hardware designs must be submitted in open formats (KiCad, FreeCAD, STL)
- **Attribution required** — do not remove author credits from source files

### Issues & Feature Requests

Use GitHub Issues with the following labels:
- `bug` — Something isn't working
- `enhancement` — New feature or module idea
- `documentation` — Docs improvements
- `firmware` — STM32 / RTOS related
- `research` — Academic or experimental use case

---

## License

ProtoBot is released under the **ProtoBot Non-Commercial Open Recreation License (PNCORL)**.

In brief:
- ✅ You may build, modify, and use ProtoBot for personal, educational, or research purposes **free of charge**
- ✅ You may share your builds publicly with **proper attribution** to the original author
- ❌ You may **not** sell ProtoBot or any derivative product commercially without explicit written permission from the author
- ❌ You may **not** remove or alter author attribution notices

See the full [LICENSE](./LICENSE) file for complete terms.

---

##Author & Credits

**ProtoBot** was conceived, designed, and developed by its original author, Helmut Chaparro Sandoval.

All hardware designs, firmware, software, documentation, and associated intellectual property are the exclusive commercial property of the original author. Recreational and research use is granted to the public under the terms described in the LICENSE file.

If you build a ProtoBot, tag your project with `#ProtoBot` and feel free to open a GitHub Discussion to share your work — the community loves to see builds!

---

## 📚 Resources

| Resource | Link |
|----------|------|
| ROS 2 Jazzy Docs | https://docs.ros.org/en/jazzy/ |
| micro-ROS | https://micro.ros.org/ |
| Nav2 | https://nav2.ros.org/ |
| STM32 Nucleo N657X0-Q | https://www.st.com/en/evaluation-tools/nucleo-n657x0-q.html |
| STM32CubeIDE | https://www.st.com/en/development-tools/stm32cubeide.html |
| FreeRTOS | https://www.freertos.org/ |
| Raspberry Pi 5 | https://www.raspberrypi.com/products/raspberry-pi-5/ |
| RPLIDAR SDK | https://github.com/slamtec/rplidar_sdk |
| slam_toolbox | https://github.com/SteveMacenski/slam_toolbox |
| Flutter | https://flutter.dev/ |

---

<div align="center">

**ProtoBot** — *Build it. Break it. Learn from it.*

Made with ❤️ for the robotics community.

</div>
