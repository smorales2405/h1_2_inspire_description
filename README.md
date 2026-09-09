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
| `urdf/h1_2_hands_isaac.urdf` | **Flat URDF pre-processed for Isaac Sim 5.1** |

### Mesh directories

```
meshes/
├── h1_2/        ← Unitree H1-2 body meshes (28 STL)
├── hand_left/   ← Inspire RH56DFTP left hand meshes (13 STL)
└── hand_right/  ← Inspire RH56DFTP right hand meshes (13 STL)
```

## Kinematics summary

### H1-2 body (27 DOF)
- 6 DOF per leg × 2 (hip yaw/pitch/roll, knee, ankle pitch/roll)
- 7 DOF per arm × 2 (shoulder pitch/roll/yaw, elbow, wrist roll/pitch/yaw)
- 1 DOF torso (yaw)

### Inspire RH56DFTP hands (12 joints each)
Each hand has 6 actuated joints and 6 mimic joints:

Both hands use the **same joint names** (`*_thumb_swing`, `*_thumb_1..3`) and the
**same limits** (`*_thumb_swing` `[0, 1.70]`, `*_thumb_1` `[0, 0.92]`, fingers `[0, 1.6]`).

The **right hand's digits are the exact mirror of the left hand's** across the
palm's `y=0` plane — verified to 0.000000 mm vertex-for-vertex, at rest and
throughout the joint range. The left hand is the reference: to change the digit
geometry, edit `inspire_hand_left.urdf.xacro` and re-mirror the right, never the
other way round. The palm (`*_hand_base_link`) is each side's own vendor export.

| Side | Actuated | Mimic (ROS) / Independent (Isaac) |
|---|---|---|
| Left | `left_thumb_swing`, `left_thumb_1`, `left_index_1`, `left_middle_1`, `left_ring_1`, `left_little_1` | `*_2/_3` joints |
| Right | `right_thumb_swing`, `right_thumb_1`, `right_index_1`, `right_middle_1`, `right_ring_1`, `right_little_1` | `*_2/_3` joints |

### Hand mounting points

```
left_wrist_yaw_link  ──(fixed, xyz=0.165 -0.1125 -0.0075, rpy=π π/2 π)──► left_hand_base_link
right_wrist_yaw_link ──(fixed, xyz=0.165  0.1125 -0.0075, rpy=0  π/2 0)──► right_hand_base_link
```

---

## ROS 2 Installation and Usage

```bash
cd ~/ros2_ws/src
git clone https://github.com/smorales2405/h1_2_inspire_description.git
cd ~/ros2_ws
colcon build --symlink-install --packages-select h1_2_inspire_description
source install/setup.bash
```

### Visualization in RViz

```bash
# Full robot with both hands (joint sliders)
ros2 launch h1_2_inspire_description display_h1_2_with_hands.launch.py

# H1-2 body only
ros2 launch h1_2_inspire_description display_h1_2.launch.py

# Left hand only
ros2 launch h1_2_inspire_description display_hand_left.launch.py

# Right hand only
ros2 launch h1_2_inspire_description display_hand_right.launch.py
```

### Xacro architecture

```
h1_2_with_inspire_hands.urdf.xacro
├── xacro:include → h1_2_body.urdf.xacro
├── xacro:include → inspire_hand_left.urdf.xacro
├── xacro:include → inspire_hand_right.urdf.xacro
├── joint: left_inspire_hand_mount_joint  (fixed)
└── joint: right_inspire_hand_mount_joint (fixed)
```

---

## Isaac Sim 5.1 Import Guide

Use the file `urdf/h1_2_hands_isaac.urdf` — a flat URDF pre-processed for Isaac Sim 5.1.

### What was changed from the Xacro-based URDF

| # | Change | Detail |
|---|---|---|
| 1 | Empty material names fixed | 26 materials with `name=""` renamed to `<linkname>_mat` |
| 2 | Inertial added to 4 sensor/logo links | `logo_link`, `imu_link`, `camera_link`, `lidar_link` — mass=0.001 kg, I=1e-6 kg·m² |
| 3 | Collision added to 6 structural body links | `left/right_hip_yaw_link`, `left/right_ankle_pitch_link`, `left/right_wrist_yaw_link` — visual STL reused |
| 4 | 12 `<mimic>` tags removed | All hand secondary phalanges become independent revolute joints at q=0 |

### Prerequisites

- Isaac Sim 5.1 installed
- Extension **`isaacsim.asset.importer.urdf`** enabled — verify via **Window › Extensions**, search "urdf"
- The ROS 2 workspace must be built (`colcon build`) before launching Isaac Sim, **or** the package path must be set manually in the importer (see step 5 below)

### Step-by-step import

1. **Launch Isaac Sim 5.1**

2. **Open the URDF Importer panel**  
   Go to **File › Import** (or **Isaac Utils › Workflows › URDF Importer** if the panel is docked)

3. **Select the file**  
   Navigate to and select:
   ```
   <your_workspace>/src/h1_2_inspire_description/urdf/h1_2_hands_isaac.urdf
   ```

4. **Configure Import Options**

   | Option | Recommended | Notes |
   |---|---|---|
   | **Fix Base** | OFF (Moveable base) | Use ON only for static tests without locomotion |
   | **Self Collision** | **OFF** | Enable only after verifying collision geometry |
   | **Create Physics Scene** | ON | Required for simulation |
   | **Distance Scale** | 1.0 | URDF is already in meters |
   | **Joint Drive Type** | None or Position | Use None for passive display |
   | **Default Drive Stiffness** | 100–1000 | Tune per-joint after import |

5. **Configure ROS Package Path** (if meshes fail to resolve)

   In the importer, find the **ROS Package List** field and add:

   | Package name | Path |
   |---|---|
   | `h1_2_inspire_description` | `/home/<user>/ros2_ws/src/h1_2_inspire_description` |

   Alternatively, source your workspace before launching Isaac Sim:
   ```bash
   source ~/ros2_ws/install/setup.bash
   ./isaac-sim.sh
   ```

6. **Click Import**

7. **Check the Output Log** (`Window › Output Log`) for any mesh resolution errors or physics warnings

8. **Save as USD**  
   Once the robot loads correctly:  
   **File › Save As** → save as `.usd` to avoid re-importing from URDF in future sessions

### Recommended initial checks after import

- [ ] **Robot Model**: verify pelvis is the root, arms and legs attach correctly
- [ ] **Finger joints**: check that all 24 hand joints (12 per hand) are present in the Articulation Inspector
- [ ] **Mimic joints**: the 12 secondary phalanges (`*_2`, `*_3`) are now **independent revolute joints** at position 0 (fully open). They will not follow the primary joints automatically — see note below
- [ ] **Self-collision OFF**: do not enable until collision meshes have been manually inspected in the viewport
- [ ] **Articulation root**: confirm that `pelvis` is set as the articulation root (`RigidBodyAPI` + `ArticulationRootAPI`)
- [ ] **Convert to USD**: after a successful import, save as USD for faster future loading

### Reconstructing mimic (finger coupling) in Isaac Sim

Isaac Sim does not support the URDF `<mimic>` tag natively. After importing, you have two options:

**Option A — Simple control (recommended for initial tests)**  
Drive only the 6 primary joints per hand. Leave secondary phalanges at q=0 (open). This is the default after import.

**Option B — Full coupling via Python**  
After importing, attach an `ArticulationController` script to the robot prim:

```python
from omni.isaac.core.articulations import Articulation

robot = Articulation("/World/h1_2_with_inspire_hands")
robot.initialize()

# Example: couple left_index_2 to left_index_1 (multiplier 1.05)
def apply_mimic(primary_joint: str, mimic_joint: str, multiplier: float):
    idx_p = robot.get_dof_index(primary_joint)
    idx_m = robot.get_dof_index(mimic_joint)
    q = robot.get_joint_positions()
    q[idx_m] = multiplier * q[idx_p]
    robot.set_joint_positions(q)

# Call apply_mimic() each simulation step for all 12 coupled pairs
```

Complete coupling table:

| Mimic joint | Drives from | Multiplier |
|---|---|---|
| `left_thumb_2_joint` | `left_thumb_1_joint` | 0.40 |
| `left_thumb_3_joint` | `left_thumb_1_joint` | 0.60 |
| `left_index_2_joint` | `left_index_1_joint` | 1.05 |
| `left_middle_2_joint` | `left_middle_1_joint` | 1.05 |
| `left_ring_2_joint` | `left_ring_1_joint` | 1.05 |
| `left_little_2_joint` | `left_little_1_joint` | 1.05 |
| `right_thumb_2_joint` | `right_thumb_1_joint` | 0.40 |
| `right_thumb_3_joint` | `right_thumb_1_joint` | 0.60 |
| `right_index_2_joint` | `right_index_1_joint` | 1.05 |
| `right_middle_2_joint` | `right_middle_1_joint` | 1.05 |
| `right_ring_2_joint` | `right_ring_1_joint` | 1.05 |
| `right_little_2_joint` | `right_little_1_joint` | 1.05 |

### Common problems and solutions

| Problem | Cause | Solution |
|---|---|---|
| Meshes not found / pink robot | `package://` path unresolved | Add `h1_2_inspire_description` path to the **ROS Package List** in the importer, or source the workspace before launching Isaac Sim |
| `pelvis` floats away on start | Floating base with gravity, no contacts | Set **Fix Base ON** for static tests, or add a ground plane and a PD controller |
| Physics explodes / instability | Large inertia ratio between body and finger links | Disable physics on sensor links (`imu_link`, `camera_link`, `lidar_link`) via the USD property panel |
| Finger secondary phalanges don't move | Mimic joints were removed | Expected — use Option A or B above to re-couple them |
| `logo_link` / sensor links have no collision | Intentional | These are visual/sensor-only frames; do not add collision |
| Wrist/hip collision unstable | STL mesh used for collision (complex geometry) | Replace collision on `left/right_wrist_yaw_link` with a `<cylinder>` primitive (~r=0.03, l=0.03) |
| Import warning: "no inertia" | Isaac version sensitivity | Already fixed in `h1_2_hands_isaac.urdf`; if it persists, increase mass of sensor links to 0.01 kg |
| USD shows T-pose only | No joint drives or states | Normal for a static load — use the **Articulation Inspector** (`Window › Physics › Articulation Inspector`) to manually move joints |

---

## Sources

- H1-2 URDF: adapted from [unitreerobotics/unitree_ros](https://github.com/unitreerobotics/unitree_ros) via [oscar-ramos/h1_2_utec](https://github.com/oscar-ramos/h1_2_utec)
- Inspire RH56DFTP URDF: exported from SolidWorks via [sw_urdf_exporter](http://wiki.ros.org/sw_urdf_exporter)

## Maintainer

Sergio Morales — smorales@utec.edu.pe  
Universidad de Ingeniería y Tecnología (UTEC), Lima, Peru
