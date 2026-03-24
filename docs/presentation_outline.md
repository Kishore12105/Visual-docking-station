# Presentation Outline: Autonomous Visual Docking Station

This document provide a slide-by-slide structure for a professional presentation about the Visual Docking Station project.

---

## Slide 1: Title Slide
*   **Title:** Autonomous Visual Docking Station
*   **Subtitle:** Precision Navigation using ROS 2 Jazzy, Gazebo Harmonic, and Computer Vision
*   **Presenter Name:** [Your Name]
*   **Date:** March 2026

## Slide 2: Project Objectives
*   **Goal:** Develop a fully autonomous mobile robot capable of "homing" to a docking station in a simulated home environment.
*   **Requirements:**
    *   Autonomous searching and detection of the dock.
    *   Visual servoing for precision alignment.
    *   Stop precisely at the docking interface.
    *   Simulation in a realisticGazebo environment.

## Slide 3: System Architecture
*   **Operating System:** Ubuntu 24.04 (Noble)
*   **Framework:** ROS 2 Jazzy Jalisco
*   **Simulation Engine:** Gazebo Harmonic (Ignition)
*   **Key Packages:**
    *   `rclpy`: Python client library for ROS 2.
    *   `cv_bridge`: Interface between ROS and OpenCV.
    *   `ros_gz_bridge`: Communication between ROS and Gazebo.

## Slide 4: Simulation Environment
*   **World Design:** A custom 8×6m room containing:
    *   A dedicated **Docking Alcove** (1.9m deep).
    *   A green **Docking Platform**.
    *   An **ArUco Marker Panel** (ID: 0) providing the visual target.
*   **Lighting:** Strategically placed point lights to ensure robust camera detection within the alcove.

## Slide 5: Robot Hardware Modeling (SDF)
*   **Design:** A custom Differential-Drive mobile robot.
*   **Key Features:**
    *   **Sensors:** Forward-facing RGB Camera (640x480).
    *   **Actuators:** Two primary motorized wheels + a low-friction ball caster.
    *   **Physics:** Optimized inertia and friction parameters (μ=0 for caster) to ensure smooth motion and prevent wheel-dragging.

## Slide 6: Visual Perception Logic
*   **Technology:** OpenCV ArUco Module.
*   **Detection Pipeline:**
    1.  Capture camera stream from `/camera/image_raw`.
    2.  Use `ArucoDetector` to identify Marker ID 0.
    3.  Apply `solvePnP` to estimate the marker's 6D pose relative to the camera.
*   **Output:** Publishes `/aruco/pose` containing depth (Z) and lateral lateral error (X).

## Slide 7: Control Strategy: The 3-Phase State Machine
*   **Phase 1: SEARCHING**
    *   Robot rotates slowly (360°) until the marker is in view.
*   **Phase 2: APPROACHING**
    *   **Lateral Control:** Angular velocity proportional to X-error (P-Control).
    *   **Linear Control:** Forward velocity proportional to distance (slows down as it nears the dock).
*   **Phase 3: DOCKED**
    *   Automatic stop when distance < 0.55m. Transitions to a safe "Mission Complete" state.

## Slide 8: Integration Challenges & Solutions
*   **Challenge 1: Physics Friction**
    *   *Issue:* Caster dragging prevented movement.
    *   *Solution:* Implemented zero-friction `<mu>` tags in the SDF.
*   **Challenge 2: Topic Scoping**
    *   *Issue:* Gazebo Harmonic scopes topics under `/model/`.
    *   *Solution:* Configured `ros_gz_bridge` with explicit remappings for `/cmd_vel` and `/odom`.
*   **Challenge 3: Dependency Conflicts**
    *   *Issue:* NumPy 2.x incompatibility with older OpenCV.
    *   *Solution:* Pinned environments to NumPy < 2.0 and OpenCV 4.9.

## Slide 9: Results & Demonstration
*   **Success metrics:**
    *   Autonomous detection from any initial position in the room.
    *   High-precision alignment (sub-centimeter error).
    *   Reliable stop at the docking platform.
*   **Visual Proof:** Debug camera feed with HUD overlay showing marker tracking and distance.

## Slide 10: Conclusion & Future Work
*   **Summary:** Successfully integrated modern ROS 2 and Gazebo tools for a visual servoing application.
*   **Next Steps:**
    *   Implement IR sensors for 100% reliability in low-light.
    *   Add Obstacle Avoidance for more complex room layouts.
    *   Deploy to physical hardware (Raspberry Pi/Jetson Nano).
