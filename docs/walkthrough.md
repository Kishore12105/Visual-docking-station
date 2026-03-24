# Visual Docking Station — Project Walkthrough

## What Was Built

A complete ROS2 Jazzy + Gazebo Harmonic visual docking simulation. A differential-drive robot spawns in a room, detects an ArUco marker on a docking station using its camera, and autonomously navigates into the dock.

## Project Structure

```
visual_docking/
├── urdf/    docking_robot.urdf.xacro    # diff-drive robot w/ camera
├── worlds/  room_world.sdf              # room + alcove + platform + marker
├── models/  aruco_marker/               # ArUco panel model + PNG texture
├── scripts/ aruco_detector_node.py      # OpenCV ArUco → /aruco/pose
│            dock_controller.py          # 3-phase visual servo controller
│            generate_aruco_marker.py    # one-time PNG generator
├── launch/  docking_simulation.launch.py
├── config/  aruco_params.yaml
└── visual_docking/__init__.py
```

## How to Run

```bash
cd ~/Downloads/files
source /opt/ros/jazzy/setup.bash
source install/setup.bash
ros2 launch visual_docking docking_simulation.launch.py
```

### Watch the debug camera feed
```bash
ros2 run rqt_image_view rqt_image_view /docking/debug_image
```

### Monitor docking status
```bash
ros2 topic echo /docking/status
```
Expected output: `SEARCHING` → `APPROACHING` → `DOCKED`

## Key Design Decisions

| Component | Decision |
|---|---|
| **Marker** | ArUco DICT_4X4_50 id=0 (simpler than AprilTag, faster detection) |
| **Detection** | OpenCV 4.8+ `ArucoDetector` + `solvePnP` (modern, non-deprecated API) |
| **Control** | Visual servo in camera frame — no TF transforms needed |
| **World** | 8×6m room + 1.9m-deep docking alcove, point light over dock |
| **Topics** | Ensure URDF plugin uses absolute paths (`/cmd_vel`, `/odom`) so ROS bridge matches the plugin topics exactly, allowing the robot to move. |
| **Friction** | Set the URDF `caster` joint's `<mu1>0.0</mu1>` and `<mu2>0.0</mu2>` so the fixed sphere does not drag and prevent the wheels from driving. |

## Docking Algorithm

```
State: SEARCHING
  → Rotate slowly (0.35 rad/s) until ArUco marker visible for < 3s
State: APPROACHING
  angular.z = -1.8 * lateral_error    (align with marker center)
  linear.x  =  0.35 * (depth - 0.55) (approach, slow below 1.2m)
  → Transition to DOCKED when depth < 0.55m
State: DOCKED
  → Stop. Mission complete.
```

## Verification Results

- ✅ `colcon build --symlink-install` → `1 package finished [2.12s]`
- ✅ `ros2 launch ... --show-args` → no errors, correct args listed
- ✅ ArUco PNG generated: `models/aruco_marker/meshes/aruco_marker.png` (640×640px)
- ✅ All scripts installed to `lib/visual_docking/`
