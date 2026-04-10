# vlm_agent - 视觉语言模型代理

本目录实现了一个基于视觉语言模型（VLM）的机械臂自主代理系统，支持通过自然语言指令和视觉反馈控制机械臂执行复杂任务。

## 核心模块

### 1. **vlm.py** - 视觉语言模型接口

封装与智谱 AI 视觉语言模型的交互，用于物体检测和目标定位。

**主要功能：**
- 图像编码为 Base64 格式
- 调用 GLM-4.5V 模型进行物体检测
- 返回归一化边界框坐标 [xmin, ymin, xmax, ymax]
- 智能处理像素坐标和归一化坐标的转换

**核心类：**
```python
class VLMDetector:
    def __init__(self, image_width=1920, image_height=1080)
    def get_object_bbox(self, target_description: str, image_path: Path) -> Optional[List[float]]
```

**使用示例：**
```python
from vlm import VLMDetector
from pathlib import Path

detector = VLMDetector(image_width=1920, image_height=1080)
bbox = detector.get_object_bbox("红色的方块", Path("image.jpg"))
# 返回: [0.25, 0.25, 0.5, 0.5]
```

---

### 2. **llm.py** - 大语言模型接口

调用大语言模型进行任务规划和决策。

**主要功能：**
- 自然语言任务理解
- 运动规划序列生成
- 多步骤任务分解

**核心函数：**
```python
def plan_robot_tasks(task_description: str) -> List[str]
```

---

### 3. **stt.py** - 语音转文本

支持语音输入，将语音转换为文本指令。

**主要功能：**
- 实时语音识别
- 支持中英文混合
- 返回识别的文本指令

---

### 4. **agent.py** - 主代理逻辑

整合 VLM、LLM 和机械臂控制，实现完整的自主任务执行流程。

**核心类：**

#### CameraCalibrator
相机坐标系到世界坐标系的转换。

```python
class CameraCalibrator:
    def __init__(self, **config)
    def convert(self, px: float, py: float) -> List[float]
```

**校准配置示例：**
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
封装所有机械臂物理动作。

```python
class RobotController:
    def move_to_location(self, location_name: str, memorized_locations: dict) -> bool
    def capture_and_save_image(self) -> str
    def perform_pick(self) -> bool
    def perform_place(self) -> bool
```

---

## 工作流程

```
用户指令 (语音/文本)
    ↓
LLM 任务规划 → 生成子任务序列
    ↓
循环执行每个子任务：
    ├─ 移动到观察位置
    ├─ 拍照
    ├─ VLM 物体检测 → 获取像素坐标
    ├─ 相机校准 → 转换为世界坐标
    ├─ 移动到目标位置
    ├─ 执行抓取/放置
    └─ 返回待命位置
    ↓
任务完成
```

---

## 使用方式

### 基础使用

```bash
python3 agent.py
```

### 环境变量配置

```bash
export ZHIPUAI_API_KEY="your_api_key_here"
```

### 自定义校准

编辑 `agent.py` 中的 `CALIBRATION_CONFIG`：

1. 在 RViz 中标记两个已知世界坐标的点
2. 记录这两个点在图像中的像素坐标
3. 更新配置中的 `point1_px`、`point1_world`、`point2_px`、`point2_world`

---

## 目录结构

```
vlm_agent/
├── vlm.py                      # VLM 接口
├── llm.py                      # LLM 接口
├── stt.py                      # 语音转文本
├── agent.py                    # 主代理逻辑
├── agent_total.py              # 完整代理实现
├── checker.py                  # 任务检查器
├── simple_moveit_controller.py # MoveIt 控制器
├── latest_capture.jpg          # 最新拍照
├── vlm_test_capture.jpg        # VLM 测试图像
└── README.md                   # 本文件
```

---

## 关键特性

### 1. 相机校准
- 支持两点校准法
- 自动计算像素到世界坐标的映射
- 处理图像坐标系和世界坐标系的差异

### 2. 智能坐标转换
- VLM 返回的归一化坐标自动转换为像素坐标
- 像素坐标通过校准转换为世界坐标
- 支持自动检测和修正坐标格式

### 3. 多步骤任务执行
- LLM 自动分解复杂任务
- 顺序执行子任务
- 支持错误恢复和重试

---

## 依赖

- ROS 2（Humble 或 Iron）
- MoveIt 2
- 智谱 AI SDK（`zai`）
- OpenCV（`cv2`）
- 其他：`python-dotenv`、`pyrealsense2`

---

## 配置说明

### 目标 Z 高度

```python
TARGET_Z_HEIGHT = 0.12  # 单位：米
```

设置机械臂抓取和放置时的 Z 轴高度。

### 图像尺寸

```python
CALIBRATION_CONFIG = {
    "image_width": 1920,
    "image_height": 1080,
    ...
}
```

需要与实际相机分辨率匹配。

---

## 故障排除

### VLM 返回像素坐标而非归一化坐标

系统会自动检测并转换。如果仍有问题，检查：
- 图像尺寸配置是否正确
- VLM 模型版本是否为 glm-4.5v

### 坐标转换不准确

重新进行相机校准：
1. 在 RViz 中标记两个已知点
2. 在图像中找到对应的像素坐标
3. 更新 `CALIBRATION_CONFIG`

### 机械臂无法连接

确保：
- ROS 2 环境已正确 source
- MoveIt 演示环境已启动
- 机械臂硬件已连接

---

## 注意事项

- 首次使用前必须进行相机校准
- VLM 调用需要有效的 API 密钥
- 建议在安全环境中测试
- 所有坐标单位为米（m）和弧度（rad）
