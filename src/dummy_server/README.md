# Dummy2 Robot Arm - ROS 2 & VLM Integration Project

## Project Overview

A complete ROS 2 and Vision Language Model (VLM) integration project for the custom-built Dummy2 six-axis robotic arm. The project successfully implements:

- **ROS 2 & Microcontroller Communication** - Reliable communication with arm controller via custom protocol
- **MoveIt 2 Motion Planning** - Trajectory planning and collision detection based on RRTConnect algorithm
- **VLM Visual Perception** - Integration with Zhipu AI GLM-4.5V for object detection and localization
- **VLA Embodied Intelligence** - Closed-loop vision-language-action control via VLM + prompt guidance

### Project Highlights

This project implements VLA (Vision-Language-Action) using **VLM + prompt guidance**, with these advantages over pure end-to-end VLA models:

- ✅ **Low-cost Quick Experience** - No large-scale training data needed, quickly validate embodied AI concepts
- ✅ **Easy Customization** - Flexible arm behavior adjustment through prompt modification
- ✅ **Low Hardware Requirements** - Minimal computational resources, suitable for education and research
- ⚠️ **Limited Precision** - Less accurate than end-to-end models, suitable for demos rather than production

---

## Core Feature Demo

### Video Demo

![Dummy2 Robot Arm VLM Demo](../../待转GIF.gif)

*The image above shows the complete process of the arm autonomously completing object grasping tasks through VLM visual feedback. The demo includes: object detection → coordinate transformation → motion planning → autonomous grasping*

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                 User Instructions (Voice/Text)              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────┐
        │   LLM Task Planning & Decomposition │
        │   (Zhipu AI GLM)                 │
        └────────────┬───────────────────┘
                     │
        ┌────────────▼───────────────────┐
        │   VLM Object Detection & Localization │
        │   (Zhipu AI GLM-4.5V)            │
        └────────────┬───────────────────┘
                     │
        ┌────────────▼───────────────────┐
        │   Camera Calibration & Coordinate Transformation │
        │   (Pixel → World Coordinates)             │
        └────────────┬───────────────────┘
                     │
        ┌────────────▼───────────────────┐
        │   MoveIt 2 Motion Planning             │
        │   (RRTConnect + Collision Detection)       │
        └────────────┬───────────────────┘
                     │
        ┌────────────▼───────────────────┐
        │   ROS 2 Action Execution             │
        │   (Trajectory Tracking & Feedback)              │
        └────────────┬───────────────────┘
                     │
        ┌────────────▼───────────────────┐
        │   Robot Hardware Control                │
        │   (Joint Drive & Gripper Control)          │
        └────────────────────────────────┘
```

---

## Quick Start

### Environment Requirements

- **Operating System** - Ubuntu 22.04 LTS
- **ROS 2** - Humble or Iron
- **Python** - 3.10+
- **Hardware** - Dummy2 robot arm + RealSense camera

### Install Dependencies

```bash
# Install ROS 2 and MoveIt 2
sudo apt install ros-humble-moveit ros-humble-ros2-control

# Install Python dependencies
pip install -r requirements.txt
```

### Start System

1. **Start MoveIt demo environment:**
   ```bash
   ros2 launch dummy_moveit_config demo_real_arm.launch.py
   ```

2. **Start VLM agent in another terminal:**
   ```bash
   cd src/vlm_agent
   python3 agent.py
   ```

3. **Or use simplified controller:**
   ```bash
   cd src/dummy_tools
   python3 simple_moveit_controller.py
   ```

---

## Project Structure

```
dummy_demo/
├── src/
│   ├── dummy_controller/          # Microcontroller communication & control
│   ├── dummy_moveit_config/       # MoveIt 2 configuration
│   ├── dummy-ros2_description/    # URDF model & meshes
│   ├── dummy_server/              # pymoveit2 library (this directory)
│   ├── dummy_tools/               # Toolkit (planning, display, obstacles)
│   └── vlm_agent/                 # VLM agent system
├── install/                       # Build output
└── README.md                      # Project documentation
```

---

## Core Module Documentation

### 1. dummy_controller - Microcontroller Communication

Handles communication and control with robot hardware.

**Key Files:**
- `dummy_arm_controller.py` - Main controller
- `moveit_server.py` - MoveIt service interface
- `dummy_cli_tool/` - Command-line toolkit

**Features:**
- Joint state publishing
- Trajectory execution
- Gripper control
- Enable/disable management

### 2. dummy_moveit_config - Motion Planning Configuration

MoveIt 2 configuration package containing arm kinematics and planning parameters.

**Key Files:**
- `config/dummy-ros2.srdf` - Semantic robot description
- `config/kinematics.yaml` - Inverse kinematics solver configuration
- `config/moveit_controllers.yaml` - Controller configuration
- `launch/demo_real_arm.launch.py` - Launch script

### 3. dummy-ros2_description - Robot Model

URDF model and STL mesh files.

**Key Files:**
- `urdf/dummy-ros2.xacro` - Robot URDF definition
- `meshes/` - 3D meshes for each joint

### 4. dummy_tools - Toolkit

Tools for planning, display, and obstacle management.

**Main Tools:**
- `moveit_rviz_planner.py` - RViz automatic planner executor
- `simple_moveit_controller.py` - Simplified controller
- `show_realsense_rgb.py` - Camera display
- `add_obstacles.py` - Obstacle addition

See [dummy_tools/README.md](../dummy_tools/README.md)

### 5. vlm_agent - VLM Agent System

Implements vision language model integration and autonomous task execution.

**Core Modules:**
- `vlm.py` - VLM interface (object detection)
- `llm.py` - LLM interface (task planning)
- `agent.py` - Main agent logic
- `stt.py` - Speech-to-text

See [vlm_agent/README.md](../vlm_agent/README.md)

---

## pymoveit2 Library

This directory contains `pymoveit2` - a Python MoveIt 2 interface library based on ROS 2 actions and services.

**Main Classes and Functions:**

- `MoveIt2` - Main control class
- `MoveIt2Gripper` - Gripper control
- `MoveIt2Servo` - Real-time servo control

**Usage Example:**

```python
from pymoveit2 import MoveIt2
from geometry_msgs.msg import PoseStamped

# Initialize
moveit2 = MoveIt2(node, "dummy_arm", "base_link")

# Move to joint position
moveit2.move_to_configuration([0, -1.57, 0, -1.57, 0, 1.57])

# Move to Cartesian pose
pose = PoseStamped()
pose.pose.position.x = 0.3
pose.pose.position.y = 0.2
pose.pose.position.z = 0.5
moveit2.move_to_pose(pose)
```

See [pymoveit2/](./pymoveit2/) directory

---

## Usage Examples

### Example 1: Basic Joint Motion

```python
import rclpy
from pymoveit2 import MoveIt2

rclpy.init()
node = rclpy.create_node("example")

moveit2 = MoveIt2(node, "dummy_arm", "base_link")

# Move to specified joint angles
moveit2.move_to_configuration([0.0, -1.57, 0.0, -1.57, 0.0, 1.57])

rclpy.shutdown()
```

### Example 2: Cartesian Space Motion

```python
from geometry_msgs.msg import PoseStamped
from tf_transformations import quaternion_from_euler

pose = PoseStamped()
pose.header.frame_id = "base_link"
pose.pose.position.x = 0.3
pose.pose.position.y = 0.2
pose.pose.position.z = 0.5

# Set orientation (RPY: 0, 0, 0)
q = quaternion_from_euler(0, 0, 0)
pose.pose.orientation.x = q[0]
pose.pose.orientation.y = q[1]
pose.pose.orientation.z = q[2]
pose.pose.orientation.w = q[3]

moveit2.move_to_pose(pose)
```

### Example 3: Gripper Control

```python
from pymoveit2 import MoveIt2Gripper

gripper = MoveIt2Gripper(node, "dummy_arm")

# Open gripper
gripper.open()

# Close gripper
gripper.close()
```

---

## FAQ

### Q: How to calibrate camera coordinate system?

A: Modify `CALIBRATION_CONFIG` in `vlm_agent/agent.py`:
1. Mark two points with known world coordinates in RViz
2. Record pixel coordinates of these points in the image
3. Update calibration parameters in config

### Q: What if VLM call fails?

A: Check the following:
- Environment variable `ZHIPUAI_API_KEY` is correctly set
- Network connection is normal
- API quota is sufficient

### Q: Robot arm cannot connect?

A: Ensure:
- ROS 2 environment is correctly sourced: `source install/setup.bash`
- MoveIt demo environment is started
- Robot hardware is connected and powered

### Q: How to customize motion planning parameters?

A: Modify parameters in `dummy_moveit_config/config/moveit_controllers.yaml`:
- `max_velocity_scaling_factor` - Maximum velocity scaling factor
- `max_acceleration_scaling_factor` - Maximum acceleration scaling factor
- `allowed_planning_time` - Planning timeout

---

## File Documentation

### Instructions

This directory contains the complete implementation of pymoveit2 library and usage examples.

### Dependencies

Main dependencies for using this project:

- ROS 2 Humble/Iron
- MoveIt 2
- pyrealsense2 (for camera)
- Zhipu AI SDK (for VLM)

### Building

Build using colcon:

```bash
cd /path/to/workspace
colcon build --merge-install --symlink-install --cmake-args "-DCMAKE_BUILD_TYPE=Release"
```

### Sourcing

After building, source the workspace:

```bash
source install/local_setup.bash
```

---

## Examples

The `examples/` directory contains multiple demo scripts:

- `ex_joint_goal.py` - Joint space motion
- `ex_pose_goal.py` - Cartesian space motion
- `ex_gripper.py` - Gripper control
- `ex_servo.py` - Real-time servo control
- `ex_collision_primitive.py` - Add primitive geometry obstacles
- `ex_collision_mesh.py` - Add mesh obstacles
- `ex_fk.py` - Forward kinematics solving
- `ex_ik.py` - Inverse kinematics solving

Run examples:

```bash
ros2 run dummy_server ex_joint_goal.py --ros-args -p joint_positions:="[0, -1.57, 0, -1.57, 0, 1.57]"
```

---

## Directory Structure

```
.
├── examples/              # Demo scripts
├── pymoveit2/             # pymoveit2 library
│   ├── moveit2.py         # Main control class
│   ├── moveit2_gripper.py # Gripper control
│   ├── moveit2_servo.py   # Servo control
│   ├── robots/            # Robot preset configurations
│   └── utils.py           # Utility functions
├── CMakeLists.txt         # CMake configuration
├── package.xml            # ROS 2 package configuration
└── README.md              # This file
```

---

## License

This project is licensed under the MIT License. See [LICENSE](./LICENSE) file for details.

---

## References

- [MoveIt 2 Official Documentation](https://moveit.ros.org/)
- [ROS 2 Official Documentation](https://docs.ros.org/en/humble/)
- [pymoveit2 Original Project](https://github.com/AndrejOrsula/pymoveit2)
- [Zhipu AI Documentation](https://open.bigmodel.cn/)
