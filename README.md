# Xiaomi Auto Drive

基于视觉的城市道路端到端自动驾驶仿真系统。本仓库按周整理 CARLA 仿真、数据采集、目标检测、语义分割以及后续规划控制成果。

> 本项目用于课程学习与仿真实验，不代表小米汽车官方产品。

## 项目进度

| 周次 | 阶段 | 核心成果 | 完整提交包 |
| --- | --- | --- | --- |
| Week 1 | 环境搭建 | WSL2、Ubuntu 22.04、CARLA 0.9.15 环境与基础仿真验证 | [xiaomi_week1.zip](https://github.com/xuzihao723/xiaomi-auto-drive/releases/download/week1-submission/xiaomi_week1.zip) |
| Week 2 | 数据管线 | RGB + LiDAR 同步采集、1000 帧 KITTI 数据和自动验收 | [xiaomi_week2.zip](https://github.com/xuzihao723/xiaomi-auto-drive/releases/download/week2-submission/xiaomi_week2.zip) |
| Week 3 | 目标检测 | YOLOv8 四类别检测、严格分组评估、人工复核和多后端测速 | [xiaomi_week3.zip](https://github.com/xuzihao723/xiaomi-auto-drive/releases/download/week3-submission/xiaomi_week3.zip) |
| Week 4 | 语义分割 | U-Net 车道线/可行驶区域分割与感知集成 | 进行中 |

## 第三周最终结果

最终方案采用类别专长双模型融合：`road_user_best.pt` 负责车辆和行人，`traffic_control_best.pt` 负责交通灯和交通标志。

| 类别 | Precision | Recall | mAP50 | mAP50-95 |
| --- | ---: | ---: | ---: | ---: |
| Car | 0.952 | 0.829 | 0.857 | 0.732 |
| Pedestrian | 0.885 | 0.804 | 0.882 | 0.615 |
| TrafficLight | 0.839 | 0.322 | 0.522 | 0.196 |
| TrafficSign | 0.843 | 0.404 | 0.554 | 0.071 |
| 总体 | 0.880 | 0.590 | 0.704 | 0.403 |

第三周还完成了两项可信度与部署改善：

- 人工复核旧独立 test 的 150 张图像、414 个现有交通控制框和 97 个漏标候选，保留完整纠错清单和原始标签。
- 笔记本同口径测速中，TensorRT GPU FP16 达到 320.63 FPS；双模型融合 CPU/GPU 分别为 29.11/103.60 FPS。所有速度均不是实际车载端到端延迟。

## 仓库结构

```text
xiaomi-auto-drive/
├── docs/                         # 系统架构与阶段进度
├── week1-environment/            # 环境搭建文档与截图
├── week2-data-pipeline/          # CARLA 数据采集与 KITTI 转换
├── week3-perception/             # YOLOv8 训练、评估、人工复核与部署基准
└── week4-segmentation/           # U-Net 分割与感知集成（进行中）
```

Git 主分支保存代码、配置、文档和可审阅结果。体积较大的数据、权重及演示视频放在对应 GitHub Release 附件中。

## 快速开始

```bash
cd week3-perception
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

详细复现命令见 [Week 3 README](week3-perception/README.md)，完整阶段记录见 [docs/weekly-progress.md](docs/weekly-progress.md)。

## 当前边界

- 完整训练集仍含伪标注；第三周只对 150 张独立测试图像进行了人工复核。
- 交通控制目标仍受远距离和小尺寸影响。
- TensorRT 等测速来自笔记本软件推理，不等于真实车载摄像头到控制输出端到端延迟。
- Town05 与 Town10HD_Opt 仍不能代表全部真实城市和传感器域。

## 技术栈

- CARLA 0.9.15
- Windows 11 + WSL2 Ubuntu 22.04
- Python 3.10+
- PyTorch / Ultralytics YOLOv8
- OpenCV / NumPy / Matplotlib / ReportLab
