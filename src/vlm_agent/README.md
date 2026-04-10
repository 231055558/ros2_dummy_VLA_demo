# vlm_agent - Vision Language Model Agent

This directory implements an autonomous robotic arm agent system based on Vision Language Models (VLM), supporting control of the arm through natural language instructions and visual feedback for complex tasks.

## Core Modules

### 1. **vlm.py** - Vision Language Model Interface

Encapsulates interaction with Zhipu AI Vision Language Models for object detection and target localization.

**Main Features:**
- Encode images to Base64 format
- Call GLM-4.5V model for object detection
- Return normalized bounding box coordinates [xmin, ymin, xmax, ymax]
- Smart handling of pixel and normalized coordinate conversion

**Core Class:**
```python
class VLMDetector:
    def __init__(self, image_width=1920, image_height=1080)
    def get_object_bbox(self, target_description: str, image_path: Path) -> Optional[List[float]]
```

**Usage Example:**
```python
from vlm import VLMDetector
from pathlib import Path

detector = VLMDetector(image_width=1920, image_height=1080)
bbox = detector.get_object_bbox("Red cube", Path("image.jpg"))
# Returns: [0.25, 0.25, 0.5, 0.5]
```

---

### 2. **llm.py** - Large Language Model Interface

Calls Large Language Models for task planning and decision making.

**Main Features:**
- Natural language task understanding
- Motion planning sequence generation
- Multi-step task decomposition

**Core Function:**
```python
def plan_robot_tasks(task_description: str) -> List[str]
```

---

### 3. **stt.py** - Speech-to-Text

Supports voice input, converting speech into text instructions.

**Main Features:**
- Real-time speech recognition
- Supports mixed Chinese and English
- Returns recognized text instructions

---

### 4. **agent.py** - Main Agent Logic

Integrates VLM, LLM, and arm control to implement a complete autonomous task execution workflow.

**Core Classes:**

#### CameraCalibrator
Handles transformation from camera coordinate system to world coordinate system.

```python
class CameraCalibrator:
    def __init__(self, **config)
    def convert(self, px: float, py: float) -> List[float]
```

**Calibration Config Example:**
```python
CALIBRATION_CONFIG = {
    "image_width": 1920,
    "image_height": 1080,
    "point1_px": [30, 957],
    "point1_world": [-0.126, -0.350],
    "point2_px": [1469, 120],
    "point2_world": [0.183, -0.191]
}
```

#### RobotController
Encapsulates all physical robotic arm actions.

```python
class RobotController:
    def move_to_location(self, location_name: str, memorized_locations: dict) -> bool
    def capture_and_save_image(self) -> str
    def perform_pick(self) -> bool
    def perform_place(self) -> bool
```

---

## Workflow

```
User Instruction (Voice/Text)
    ↓
LLM Task Planning → Generate sub-task sequence
    ↓
Execute each sub-task in loop:
    ├─ Move to observation position
    ├─ Take photo
    ├─ VLM Object Detection → Get pixel coordinates
    ├─ Camera Calibration → Convert to world coordinates
    ├─ Move to target position
    ├─ Execute Pick/Place
    └─ Return to standby position
    ↓
Task Complete
```

---

## Usage

### Basic Usage

```bash
python3 agent.py
```

### Environment Variable Configuration

```bash
export ZHIPUAI_API_KEY="your_api_key_here"
```

### Custom Calibration

Edit `CALIBRATION_CONFIG` in `agent.py`:

1. Mark two points with known world coordinates in RViz
2. Find corresponding pixel coordinates in the image
3. Update `point1_px`, `point1_world`, `point2_px`, `point2_world` in config

---

## Directory Structure

```
vlm_agent/
├── vlm.py                      # VLM interface
├── llm.py                      # LLM interface
├── stt.py                      # Speech-to-text
├── agent.py                    # Main agent logic
├── agent_total.py              # Complete agent implementation
├── checker.py                  # Task checker
├── simple_moveit_controller.py # MoveIt controller
├── latest_capture.jpg          # Latest photo capture
├── vlm_test_capture.jpg        # VLM test image
└── README.md                   # This file
```

---

## Key Features

### 1. Camera Calibration
- Supports two-point calibration method
- Automatic calculation of pixel-to-world mapping
- Handles differences between image and world coordinate systems

### 2. Intelligent Coordinate Transformation
- Normalized coordinates from VLM automatically converted to pixel coordinates
- Pixel coordinates converted to world coordinates via calibration
- Supports automatic detection and correction of coordinate formats

### 3. Multi-step Task Execution
- LLM automatically decomposes complex tasks
- Sequential execution of sub-tasks
- Supports error recovery and retries

---

## Dependencies

- ROS 2 (Humble or Iron)
- MoveIt 2
- Zhipu AI SDK (`zai`)
- OpenCV (`cv2`)
- Others: `python-dotenv`, `pyrealsense2`

---

## Configuration Details

### Target Z Height

```python
TARGET_Z_HEIGHT = 0.12  # Unit: meters
```

Sets the Z-axis height for arm pick and place operations.

### Image Dimensions

```python
CALIBRATION_CONFIG = {
    "image_width": 1920,
    "image_height": 1080,
    ...
}
```

Must match actual camera resolution.

---

## Troubleshooting

### VLM returns pixel coordinates instead of normalized coordinates

The system automatically detects and converts. If issues persist, check:
- Image dimension configuration is correct
- VLM model version is glm-4.5v

### Coordinate transformation inaccurate

Redo camera calibration:
1. Mark two known points in RViz
2. Find corresponding pixel coordinates in image
3. Update `CALIBRATION_CONFIG`

### Arm cannot connect

Ensure:
- ROS 2 environment is correctly sourced
- MoveIt demo environment is started
- Arm hardware is connected

---

## Notes

- Camera calibration is mandatory before first use
- VLM calls require a valid API key
- Recommend testing in a safe environment
- All coordinate units are in meters (m) and radians (rad)
