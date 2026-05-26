# h1_2_inspire_description

A self-contained ROS 2 description package for the **Unitree H1-2** humanoid robot equipped with **Inspire RH56DFTP** five-finger hands. All meshes are bundled inside the package — no external dependencies on other description packages are required.

## Supported ROS 2

Tested with **ROS 2 Humble** on Ubuntu 22.04.

## Contents

| File | Description |
|---|---|
| `urdf/h1_2_body.urdf.xacro` | H1-2 body without hands (27 DOF) |
| `urdf/inspire_hand_left.urdf.xacro` | Left Inspire hand fragment (12 joints) |
| `urdf/inspire_hand_right.urdf.xacro` | Right Inspire hand fragment (12 joints) |
| `urdf/h1_2_with_inspire_hands.urdf.xacro` | Full robot — body + both hands (58 joints total) |
| `urdf/inspire_hand_left_standalone.urdf.xacro` | Left hand with world link for isolated visualization |
| `urdf/inspire_hand_right_standalone.urdf.xacro` | Right hand with world link for isolated visualization |

### Mesh directories

```
meshes/
├── h1_2/        ← Unitree H1-2 body meshes (88 STL)
├── hand_left/   ← Inspire RH56DFTP left hand meshes (14 STL)
└── hand_right/  ← Inspire RH56DFTP right hand meshes (13 STL)
```

## Kinematics summary

### H1-2 body (27 DOF)
- 6 DOF per leg × 2 (hip yaw/pitch/roll, knee, ankle pitch/roll)
- 7 DOF per arm × 2 (shoulder pitch/roll/yaw, elbow, wrist roll/pitch/yaw)
- 1 DOF torso (yaw)

### Inspire RH56DFTP hands (12 joints each)
Each hand has 6 actuated joints and 6 mimic joints:

| Side | Actuated | Mimic |
|---|---|---|
| Left | `left_thumb_swing`, `left_thumb_1`, `left_index_1`, `left_middle_1`, `left_ring_1`, `left_little_1` | `*_2/_3` joints via `<mimic>` |
| Right | `right_thumb_1`, `right_thumb_2`, `right_index_1`, `right_middle_1`, `right_ring_1`, `right_little_1` | `*_3/_4/_2` joints via `<mimic>` |

### Hand mounting points

The hands attach to the last link of each arm via fixed joints:

```
left_wrist_yaw_link  ──(fixed, xyz=0.165 -0.1125 -0.0075, rpy=π π/2 π)──► left_hand_base_link
right_wrist_yaw_link ──(fixed, xyz=0.165  0.1125 -0.0075, rpy=0  π/2 0)──► right_hand_base_link
```

## Installation

```bash
cd ~/ros2_ws/src
git clone https://github.com/smorales2405/h1_2_inspire_description.git
cd ~/ros2_ws
colcon build --symlink-install --packages-select h1_2_inspire_description
source install/setup.bash
```

## Usage

### Full robot with both hands (joint sliders)
```bash
ros2 launch h1_2_inspire_description display_h1_2_with_hands.launch.py
```

### H1-2 body only
```bash
ros2 launch h1_2_inspire_description display_h1_2.launch.py
```

### Left hand only
```bash
ros2 launch h1_2_inspire_description display_hand_left.launch.py
```

### Right hand only
```bash
ros2 launch h1_2_inspire_description display_hand_right.launch.py
```

## Xacro architecture

The combined model is assembled via `xacro:include` — no copy-paste URDF:

```
h1_2_with_inspire_hands.urdf.xacro
├── xacro:include → h1_2_body.urdf.xacro         (H1-2 body fragment)
├── xacro:include → inspire_hand_left.urdf.xacro  (left hand fragment)
├── xacro:include → inspire_hand_right.urdf.xacro (right hand fragment)
├── joint: left_inspire_hand_mount_joint  (fixed, parent=left_wrist_yaw_link)
└── joint: right_inspire_hand_mount_joint (fixed, parent=right_wrist_yaw_link)
```

## Sources

- H1-2 URDF: adapted from [unitreerobotics/unitree_ros](https://github.com/unitreerobotics/unitree_ros) via [oscar-ramos/h1_2_utec](https://github.com/oscar-ramos/h1_2_utec)
- Inspire RH56DFTP URDF: exported from SolidWorks via [sw_urdf_exporter](http://wiki.ros.org/sw_urdf_exporter)

## Maintainer

Sergio Morales — smorales@utec.edu.pe  
Universidad de Ingeniería y Tecnología (UTEC), Lima, Peru
