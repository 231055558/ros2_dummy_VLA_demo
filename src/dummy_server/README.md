# Dummy2 机械臂 - ROS 2 与 VLM 集成项目

## 项目概述

这是一个针对自制 Dummy2 六轴机械臂的完整 ROS 2 与视觉语言模型（VLM）集成项目。项目成功实现了：

- **ROS 2 与下位机通信** - 通过自定义协议与机械臂控制器的可靠通信
- **MoveIt 2 运动规划** - 基于 RRTConnect 算法的轨迹规划与碰撞检测
- **VLM 视觉感知** - 集成智谱 AI GLM-4.5V 进行物体检测与定位
- **VLA 具身智能** - 通过 VLM + 提示词引导实现视觉-语言-动作的闭环控制

### 项目特色

本项目采用 **VLM + 提示词引导** 的方式实现 VLA（Vision-Language-Action），相比纯正的端到端 VLA 模型有以下特点：

- ✅ **低成本快速体验** - 无需大规模训练数据，快速验证具身智能概念
- ✅ **易于定制** - 通过修改提示词灵活调整机械臂行为
- ✅ **支持低端硬件** - 对计算资源要求低，适合教学和研究
- ⚠️ **精度有限** - 不如端到端模型精准，适合演示而非生产环境

---

## 核心功能演示

### 视频演示

![Dummy2 机械臂 VLM 演示](../../待转GIF.gif)

*上图展示了机械臂通过 VLM 视觉反馈自主完成物体抓取任务的完整流程。演示包括：物体检测 → 坐标转换 → 运动规划 → 自主抓取*

---

## 系统架构

```
┌─────────────────────────────────────────────────────────────┐
│                    用户指令 (语音/文本)                      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────┐
        │   LLM 任务规划与分解            │
        │   (智谱 AI GLM)                 │
        └────────────┬───────────────────┘
                     │
        ┌────────────▼───────────────────┐
        │   VLM 物体检测与定位            │
        │   (智谱 AI GLM-4.5V)            │
        └────────────┬───────────────────┘
                     │
        ┌────────────▼───────────────────┐
        │   相机校准与坐标转换            │
        │   (像素 → 世界坐标)             │
        └────────────┬───────────────────┘
                     │
        ┌────────────▼───────────────────┐
        │   MoveIt 2 运动规划             │
        │   (RRTConnect + 碰撞检测)       │
        └────────────┬───────────────────┘
                     │
        ┌────────────▼───────────────────┐
        │   ROS 2 Action 执行             │
        │   (轨迹跟踪与反馈)              │
        └────────────┬───────────────────┘
                     │
        ┌────────────▼───────────────────┐
        │   机械臂硬件控制                │
        │   (关节驱动与夹爪控制)          │
        └────────────────────────────────┘
```

---

## 快速开始

### 环境要求

- **操作系统** - Ubuntu 22.04 LTS
- **ROS 2** - Humble 或 Iron
- **Python** - 3.10+
- **硬件** - Dummy2 机械臂 + RealSense 相机

### 安装依赖

```bash
# 安装 ROS 2 和 MoveIt 2
sudo apt install ros-humble-moveit ros-humble-ros2-control

# 安装 Python 依赖
pip install -r requirements.txt
```

### 启动系统

1. **启动 MoveIt 演示环境：**
   ```bash
   ros2 launch dummy_moveit_config demo_real_arm.launch.py
   ```

2. **在另一个终端启动 VLM 代理：**
   ```bash
   cd src/vlm_agent
   python3 agent.py
   ```

3. **或使用简化版控制器：**
   ```bash
   cd src/dummy_tools
   python3 simple_moveit_controller.py
   ```

---

## 项目结构

```
dummy_demo/
├── src/
│   ├── dummy_controller/          # 下位机通信与控制
│   ├── dummy_moveit_config/       # MoveIt 2 配置
│   ├── dummy-ros2_description/    # URDF 模型与网格
│   ├── dummy_server/              # pymoveit2 库（本目录）
│   ├── dummy_tools/               # 工具集（规划、显示、障碍物）
│   └── vlm_agent/                 # VLM 代理系统
├── install/                       # 编译输出
└── README.md                      # 项目说明
```

---

## 核心模块说明

### 1. dummy_controller - 下位机通信

负责与机械臂硬件的通信和控制。

**关键文件：**
- `dummy_arm_controller.py` - 主控制器
- `moveit_server.py` - MoveIt 服务接口
- `dummy_cli_tool/` - 命令行工具集

**功能：**
- 关节状态发布
- 轨迹执行
- 夹爪控制
- 使能/去使能管理

### 2. dummy_moveit_config - 运动规划配置

MoveIt 2 的配置包，包含机械臂的运动学和规划参数。

**关键文件：**
- `config/dummy-ros2.srdf` - 语义机器人描述
- `config/kinematics.yaml` - 逆运动学求解器配置
- `config/moveit_controllers.yaml` - 控制器配置
- `launch/demo_real_arm.launch.py` - 启动脚本

### 3. dummy-ros2_description - 机械臂模型

URDF 模型和 STL 网格文件。

**关键文件：**
- `urdf/dummy-ros2.xacro` - 机械臂 URDF 定义
- `meshes/` - 各关节的 3D 网格

### 4. dummy_tools - 工具集

提供规划、显示和障碍物管理的工具。

**主要工具：**
- `moveit_rviz_planner.py` - RViz 自动规划执行器
- `simple_moveit_controller.py` - 简化版控制器
- `show_realsense_rgb.py` - 相机显示
- `add_obstacles.py` - 障碍物添加

详见 [dummy_tools/README.md](../dummy_tools/README.md)

### 5. vlm_agent - VLM 代理系统

实现视觉语言模型的集成和自主任务执行。

**核心模块：**
- `vlm.py` - VLM 接口（物体检测）
- `llm.py` - LLM 接口（任务规划）
- `agent.py` - 主代理逻辑
- `stt.py` - 语音转文本

详见 [vlm_agent/README.md](../vlm_agent/README.md)

---

## pymoveit2 库

本目录包含 `pymoveit2` - 一个基于 ROS 2 actions 和 services 的 Python MoveIt 2 接口库。

**主要类和函数：**

- `MoveIt2` - 主控制类
- `MoveIt2Gripper` - 夹爪控制
- `MoveIt2Servo` - 实时伺服控制

**使用示例：**

```python
from pymoveit2 import MoveIt2
from geometry_msgs.msg import PoseStamped

# 初始化
moveit2 = MoveIt2(node, "dummy_arm", "base_link")

# 移动到关节位置
moveit2.move_to_configuration([0, -1.57, 0, -1.57, 0, 1.57])

# 移动到笛卡尔位姿
pose = PoseStamped()
pose.pose.position.x = 0.3
pose.pose.position.y = 0.2
pose.pose.position.z = 0.5
moveit2.move_to_pose(pose)
```

详见 [pymoveit2/](./pymoveit2/) 目录

---

## 使用示例

### 示例 1: 基础关节运动

```python
import rclpy
from pymoveit2 import MoveIt2

rclpy.init()
node = rclpy.create_node("example")

moveit2 = MoveIt2(node, "dummy_arm", "base_link")

# 移动到指定关节角度
moveit2.move_to_configuration([0.0, -1.57, 0.0, -1.57, 0.0, 1.57])

rclpy.shutdown()
```

### 示例 2: 笛卡尔空间运动

```python
from geometry_msgs.msg import PoseStamped
from tf_transformations import quaternion_from_euler

pose = PoseStamped()
pose.header.frame_id = "base_link"
pose.pose.position.x = 0.3
pose.pose.position.y = 0.2
pose.pose.position.z = 0.5

# 设置姿态（RPY: 0, 0, 0）
q = quaternion_from_euler(0, 0, 0)
pose.pose.orientation.x = q[0]
pose.pose.orientation.y = q[1]
pose.pose.orientation.z = q[2]
pose.pose.orientation.w = q[3]

moveit2.move_to_pose(pose)
```

### 示例 3: 夹爪控制

```python
from pymoveit2 import MoveIt2Gripper

gripper = MoveIt2Gripper(node, "dummy_arm")

# 打开夹爪
gripper.open()

# 关闭夹爪
gripper.close()
```

---

## 常见问题

### Q: 如何校准相机坐标系？

A: 在 `vlm_agent/agent.py` 中修改 `CALIBRATION_CONFIG`：
1. 在 RViz 中标记两个已知世界坐标的点
2. 记录这两个点在图像中的像素坐标
3. 更新配置中的校准参数

### Q: VLM 调用失败怎么办？

A: 检查以下几点：
- 环境变量 `ZHIPUAI_API_KEY` 是否正确设置
- 网络连接是否正常
- API 配额是否充足

### Q: 机械臂无法连接？

A: 确保：
- ROS 2 环境已正确 source：`source install/setup.bash`
- MoveIt 演示环境已启动
- 机械臂硬件已连接并通电

### Q: 如何自定义运动规划参数？

A: 修改 `dummy_moveit_config/config/moveit_controllers.yaml` 中的参数：
- `max_velocity_scaling_factor` - 最大速度缩放因子
- `max_acceleration_scaling_factor` - 最大加速度缩放因子
- `allowed_planning_time` - 规划超时时间

---

## 文件说明

### Instructions

本目录包含 pymoveit2 库的完整实现和使用示例。

### Dependencies

这些是使用本项目的主要依赖：

- ROS 2 Humble/Iron
- MoveIt 2
- pyrealsense2（用于相机）
- 智谱 AI SDK（用于 VLM）

### Building

使用 colcon 编译：

```bash
cd /path/to/workspace
colcon build --merge-install --symlink-install --cmake-args "-DCMAKE_BUILD_TYPE=Release"
```

### Sourcing

编译完成后，source 工作空间：

```bash
source install/local_setup.bash
```

---

## Examples

`examples/` 目录包含多个演示脚本：

- `ex_joint_goal.py` - 关节空间运动
- `ex_pose_goal.py` - 笛卡尔空间运动
- `ex_gripper.py` - 夹爪控制
- `ex_servo.py` - 实时伺服控制
- `ex_collision_primitive.py` - 添加原始几何障碍物
- `ex_collision_mesh.py` - 添加网格障碍物
- `ex_fk.py` - 正运动学求解
- `ex_ik.py` - 逆运动学求解

运行示例：

```bash
ros2 run dummy_server ex_joint_goal.py --ros-args -p joint_positions:="[0, -1.57, 0, -1.57, 0, 1.57]"
```

---

## Directory Structure

```
.
├── examples/              # 演示脚本
├── pymoveit2/             # pymoveit2 库
│   ├── moveit2.py         # 主控制类
│   ├── moveit2_gripper.py # 夹爪控制
│   ├── moveit2_servo.py   # 伺服控制
│   ├── robots/            # 机器人预设配置
│   └── utils.py           # 工具函数
├── CMakeLists.txt         # CMake 配置
├── package.xml            # ROS 2 包配置
└── README.md              # 本文件
```

---

## 许可证

本项目采用 MIT 许可证。详见 [LICENSE](./LICENSE) 文件。

---

## 参考资源

- [MoveIt 2 官方文档](https://moveit.ros.org/)
- [ROS 2 官方文档](https://docs.ros.org/en/humble/)
- [pymoveit2 原始项目](https://github.com/AndrejOrsula/pymoveit2)
- [智谱 AI 文档](https://open.bigmodel.cn/)
