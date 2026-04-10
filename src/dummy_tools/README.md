# dummy_tools - Robot Arm Toolkit

This directory contains various practical tools for the Dummy2 robot arm, supporting motion planning, visual feedback, and obstacle management.

## Core Tools

### 1. **moveit_rviz_planner.py** - RViz MoveIt Automatic Planner Executor

Performs motion planning through RViz's MoveIt environment and automatically executes successful plans.

**Main Features:**
- Safe planning with obstacles in RViz
- Dynamically read target positions from `targets.json`
- Record point mode (save current pose in real-time)
- Cruise mode (quickly switch between preset positions)
- Gripper control (open/close/enable)
- Arm enable/disable

**Usage:**
```bash
python3 moveit_rviz_planner.py
```

**Menu Commands:**
- `e` / `enable` - Enable arm
- `d` / `disable` - Disable arm
- `c` - Enable gripper and close
- `x` - Open gripper and disable
- `p` - Enter record point mode
- `o` - Enter cruise mode
- `6` / `sequence` - Execute preset sequence
- `7` / `current` - Display current state
- Direct input of target name - Execute to that position

**Configuration File:**
- `targets.json` - Store preset positions (unit: degrees)

---

### 2. **simple_moveit_controller.py** - Simplified MoveIt Controller

Provides a clean interface for joint motion, end-effector pose control, and forward/inverse kinematics solving.

**Main Features:**
- Joint space motion planning (supports degrees/radians)
- Cartesian space pose control (XYZ + RPY)
- Forward kinematics (FK) - Get end-effector pose
- Inverse kinematics (IK) - Solve joint angles
- Gripper control and enable management

**Usage:**
```bash
python3 simple_moveit_controller.py
```

**Command Examples:**
```
enable 1                    # Enable arm
joints 0 0 0 0 0 0 deg     # Move to specified joint angles (degrees)
ee                          # Get current end-effector pose
pose 0.3 0.2 0.5 0 0 0     # Move to Cartesian pose (meters, radians)
c                           # Enable gripper and close
x                           # Open gripper and disable
```

---

### 3. **show_realsense_rgb.py** - RealSense RGB Real-time Display

Display RGB image stream from RealSense camera in real-time.

**Main Features:**
- Support custom resolution and frame rate
- Display auto-exposure and white balance status
- Press `q` to exit

**Usage:**
```bash
python3 show_realsense_rgb.py --width 1920 --height 1080 --fps 30
```

**Parameters:**
- `--width` - RGB image width (default 1920)
- `--height` - RGB image height (default 1080)
- `--fps` - Frame rate (default 30)
- `--device` - Specify device serial number (optional)

---

### 4. **add_obstacles.py** - Obstacle Addition Tool

Add rectangular obstacles to MoveIt planning scene.

**Main Features:**
- Create and publish collision obstacles
- Support custom position and size
- Visualize in RViz

**Usage:**
```bash
python3 add_obstacles.py
```

---

## Directory Structure

```
dummy_tools/
├── moveit_rviz_planner.py      # RViz automatic planner executor
├── simple_moveit_controller.py # Simplified MoveIt controller
├── show_realsense_rgb.py       # RealSense RGB display
├── add_obstacles.py            # Obstacle addition tool
├── agent/                      # VLM agent module
│   ├── vlm.py                  # Vision language model interface
│   ├── llm.py                  # Large language model interface
│   ├── stt.py                  # Speech-to-text
│   └── agent.py                # Main agent logic
└── README.md                   # This file
```

## Dependencies

- ROS 2 (Humble or Iron)
- MoveIt 2
- pyrealsense2 (for RealSense camera)
- Zhipu AI SDK (for VLM features)

## Quick Start

1. **Start MoveIt demo environment:**
   ```bash
   ros2 launch dummy_moveit_config demo_real_arm.launch.py
   ```

2. **Run tools in another terminal:**
   ```bash
   python3 moveit_rviz_planner.py
   ```

3. **Follow menu prompts to control the arm**

## Notes

- All tools require ROS 2 environment to be properly configured
- Ensure arm is connected and communication is normal before use
- Recommend enabling arm before gripper operations
- Record points are automatically saved to `targets.json`
