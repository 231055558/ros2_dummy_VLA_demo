# dummy_tools - 机械臂工具集

本目录包含用于 Dummy2 机械臂的各类实用工具，支持运动规划、视觉反馈和障碍物管理。

## 核心工具

### 1. **moveit_rviz_planner.py** - RViz MoveIt 自动规划执行器

通过 RViz 的 MoveIt 环境进行运动规划，规划成功后自动执行。

**主要功能：**
- 通过 RViz 中的障碍物进行安全规划
- 支持从 `targets.json` 动态读取目标位置
- 支持记录点模式（实时保存当前位姿）
- 支持巡航模式（快速切换预设位置）
- 夹爪控制（打开/关闭/使能）
- 机械臂使能/去使能

**使用方式：**
```bash
python3 moveit_rviz_planner.py
```

**菜单命令：**
- `e` / `enable` - 机械臂使能
- `d` / `disable` - 机械臂去使能
- `c` - 夹爪使能并关闭
- `x` - 打开夹爪并去使能
- `p` - 进入记录点模式
- `o` - 进入巡航模式
- `6` / `sequence` - 执行预设序列
- `7` / `current` - 显示当前状态
- 直接输入目标名 - 执行到该位置

**配置文件：**
- `targets.json` - 存储预设位置（单位：度）

---

### 2. **simple_moveit_controller.py** - 简化版 MoveIt 控制器

提供关节运动、末端位姿控制和正逆运动学求解的简洁接口。

**主要功能：**
- 关节空间运动规划（支持度/弧度）
- 笛卡尔空间位姿控制（XYZ + RPY）
- 正运动学（FK）- 获取末端位姿
- 逆运动学（IK）- 求解关节角
- 夹爪控制和使能管理

**使用方式：**
```bash
python3 simple_moveit_controller.py
```

**命令示例：**
```
enable 1                    # 使能机械臂
joints 0 0 0 0 0 0 deg     # 移动到指定关节角（度）
ee                          # 获取当前末端位姿
pose 0.3 0.2 0.5 0 0 0     # 移动到笛卡尔位姿（米，弧度）
c                           # 夹爪使能并关闭
x                           # 打开夹爪并去使能
```

---

### 3. **show_realsense_rgb.py** - RealSense RGB 实时显示

实时显示 RealSense 相机的 RGB 图像流。

**主要功能：**
- 支持自定义分辨率和帧率
- 显示自动曝光和白平衡状态
- 按 `q` 键退出

**使用方式：**
```bash
python3 show_realsense_rgb.py --width 1920 --height 1080 --fps 30
```

**参数：**
- `--width` - RGB 图像宽度（默认 1920）
- `--height` - RGB 图像高度（默认 1080）
- `--fps` - 帧率（默认 30）
- `--device` - 指定设备序列号（可选）

---

### 4. **add_obstacles.py** - 障碍物添加工具

向 MoveIt planning scene 中添加长方体障碍物。

**主要功能：**
- 创建并发布碰撞障碍物
- 支持自定义位置和尺寸
- 在 RViz 中可视化显示

**使用方式：**
```bash
python3 add_obstacles.py
```

---

## 目录结构

```
dummy_tools/
├── moveit_rviz_planner.py      # RViz 自动规划执行器
├── simple_moveit_controller.py # 简化版 MoveIt 控制器
├── show_realsense_rgb.py       # RealSense RGB 显示
├── add_obstacles.py            # 障碍物添加工具
├── agent/                      # VLM 代理模块
│   ├── vlm.py                  # 视觉语言模型接口
│   ├── llm.py                  # 大语言模型接口
│   ├── stt.py                  # 语音转文本
│   └── agent.py                # 主代理逻辑
└── README.md                   # 本文件
```

## 依赖

- ROS 2（Humble 或 Iron）
- MoveIt 2
- pyrealsense2（用于 RealSense 相机）
- 智谱 AI SDK（用于 VLM 功能）

## 快速开始

1. **启动 MoveIt 演示环境：**
   ```bash
   ros2 launch dummy_moveit_config demo_real_arm.launch.py
   ```

2. **在另一个终端运行工具：**
   ```bash
   python3 moveit_rviz_planner.py
   ```

3. **按菜单提示操作机械臂**

## 注意事项

- 所有工具都需要 ROS 2 环境已正确配置
- 使用前请确保机械臂已连接并通信正常
- 夹爪操作前建议先使能机械臂
- 记录点会自动保存到 `targets.json`
