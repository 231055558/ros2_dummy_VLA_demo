# Dummy Robot Arm - ROS2 Project

A six-axis robotic arm control system based on ROS2 Humble with integrated MoveIt motion planning, visual perception, and intelligent agent capabilities.

## Project Overview

This project provides a complete robotic arm control solution, including:
- Motion planning and execution based on MoveIt
- Low-level USB communication controller
- Intel RealSense visual integration
- VLM (Vision Language Model) intelligent agent
- Various practical tools and example programs

---

## Demo Video

![Robot Arm Demo](演示视频.gif)


## Contact

For questions, suggestions, or collaboration inquiries, please reach out:

**Email**: [231055558@qq.com]

## Directory Structure

### 📦 Packages in src directory:

#### 1. **dummy_controller** - Core Controller Package
Low-level control and communication module for the robotic arm.

**Main Features:**
- `dummy_arm_controller.py` - USB controller for hardware communication
- `moveit_server.py` - MoveIt server node providing motion planning services
- `add_collision_object.py` - Collision object management for scene planning
- `depth_image_visualizer.py` - Depth image visualization tool
- `dummy_cli_tool/` - Low-level communication toolkit (based on fibre protocol)

#### 2. **dummy_moveit_config** - MoveIt Configuration Package
Configuration files for the MoveIt motion planning framework.

**Main Contents:**
- Kinematics configuration
- Planner configuration (OMPL)
- Controller configuration
- Launch files
  - `demo_real_arm.launch.py` - Real arm launch file
  - Other simulation and debug launch files

#### 3. **dummy-ros2_description** - Robot Description Package
URDF/Xacro model definitions for the robot.

**Main Contents:**
- `urdf/` - Robot URDF description files
- `meshes/` - 3D model files
- `config/` - RViz and other configurations
- `launch/` - Model display launch files

#### 4. **dummy_server** - MoveIt Python Interface
Python interface for MoveIt control.

**Main Contents:**
- `pymoveit2/` - Python MoveIt2 library
- `examples/` - Usage examples

#### 5. **dummy_tools** - Utility Toolkit
Various convenient control and testing tools.

**Main Tools:**
- `moveit_rviz_planner.py` - RViz automatic planner executor for visual planning
- `simple_moveit_controller.py` - Simplified MoveIt controller supporting joint and Cartesian space control
- `add_obstacles.py` - Add obstacles to planning scene
- `show_realsense_rgb.py` - RealSense camera RGB image display

#### 6. **vlm_agent** - Vision Language Model Agent
Intelligent agent system integrating vision and language models.

**Main Modules:**
- `agent.py` / `agent_total.py` - Main agent programs
- `vlm.py` - Vision language model interface
- `llm.py` - Large language model interface
- `stt.py` - Speech-to-text module
- `checker.py` - State checker
- `vlm_bbox_show.py` - Target detection bounding box visualization

---

## Available Demo Examples

### 1. Basic Motion Control
Use `simple_moveit_controller.py` for joint space and Cartesian space motion control.

### 2. RViz Visual Planning
Use `moveit_rviz_planner.py` for interactive planning and execution in RViz.

### 3. Obstacle Avoidance Planning
Use `add_obstacles.py` to add virtual obstacles and test collision avoidance.

### 4. Visual Perception
Use `show_realsense_rgb.py` to view RealSense camera images.

### 5. Intelligent Agent
Run the agent program in `vlm_agent` for vision-guided intelligent operations.

---

## Environment Setup

### Pre-configured VM Image (Optional)

If you prefer not to configure manually, download the pre-configured VMware image:
- File: dummy_ros2_vmware.rar
- Link: https://pan.baidu.com/s/1RXnZTWswST4H9HLcxrATJw?pwd=lhyn
- Password: lhyn

### System Requirements
- **OS**: Ubuntu 22.04
- **ROS Version**: ROS2 Humble

### Installation Steps

#### 1. Install ROS2 Humble

Use the fishros one-click installation script:

```bash
wget http://fishros.com/install -O fishros && . fishros
```

#### 2. Install Dependencies

**ARM Cross-compilation Toolchain:**
```bash
sudo apt update
sudo apt install gcc-arm-none-eabi
```

**ROS2 Control Messages:**
```bash
sudo apt install ros-humble-control-msgs
```

**MoveIt Motion Planning Framework:**
```bash
sudo apt install ros-humble-moveit
sudo apt install ros-humble-moveit-servo
```

#### 3. Install Intel RealSense SDK (Optional)

If using RealSense camera:

**Add GPG Key:**
```bash
sudo mkdir -p /etc/apt/keyrings
curl -sSL https://librealsense.intel.com/Debian/librealsense.pgp | sudo tee /etc/apt/keyrings/librealsense.pgp > /dev/null
```

**Add Repository:**
```bash
echo "deb [signed-by=/etc/apt/keyrings/librealsense.pgp] https://librealsense.intel.com/Debian/apt-repo $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/librealsense.list
```

**Install SDK:**
```bash
sudo apt-get update
sudo apt-get install librealsense2-dkms librealsense2-dev
sudo apt install librealsense2-utils
```

#### 4. Python Environment Setup

**Install pip:**
```bash
sudo apt install python3-pip
```

**Install pyusb:**
```bash
pip install pyusb
```

#### 5. USB Device Permissions

**Find Device VID and PID:**
```bash
lsusb
```

Look for output like:
```
Bus 002 Device 007: ID 1209:0d32 Generic ODrive Robotics ODrive v3
```

Where `1209` is VID and `0d32` is PID.

**Create udev Rule:**
```bash
sudo nano /etc/udev/rules.d/99-dummy-arm.rules
```

Add the following (replace with your VID and PID):
```
# Rule for Dummy Arm Controller
SUBSYSTEM=="usb", ATTRS{idVendor}=="1209", ATTRS{idProduct}=="0d32", MODE="0666"
```

**Apply Rules:**
```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Reconnect the USB device.

---

## Build and Run

### Build Workspace

```bash
# Navigate to workspace root (parent of src)
cd /path/to/dummy_demo

# Source ROS2 environment
source /opt/ros/humble/setup.bash

# Build
colcon build

# Source workspace environment
source install/setup.bash
```

### Start System

**Terminal 1 - Start MoveIt and RViz:**
```bash
source install/setup.bash
ros2 launch dummy_moveit_config demo_real_arm.launch.py
```

**Terminal 2 - Start Arm Controller:**
```bash
source install/setup.bash
ros2 run dummy_controller dummy_arm_controller
```

**Terminal 3 - Run Planner (Optional):**
```bash
source install/setup.bash
python3 src/dummy_tools/moveit_rviz_planner.py
```

---

## Troubleshooting

### USB Permission Issues
If the USB device cannot be accessed:
1. Check if udev rules are correctly configured
2. Verify VID and PID match
3. Reconnect the device

### MoveIt Planning Fails
1. Check if joint limits are reasonable
2. Verify target pose is within workspace
3. Check for collisions in RViz

### Build Errors
Ensure all dependencies are installed:
```bash
rosdep install --from-paths src --ignore-src -r -y
```

---

## References

**Note:** This project uses custom firmware for the end-effector gripper. Programs from referenced repositories may not be fully compatible, but generally won't cause damage. Use as reference only.

- [Muzi's Improved Dummy Arm](https://gitee.com/switchpi/dummy.git)
- [hata8210's MoveIt Integration](https://github.com/hata8210/dummy_moveit_ws.git)