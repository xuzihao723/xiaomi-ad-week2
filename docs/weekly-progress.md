# 阶段进度说明

## Week 1：环境搭建与仿真验证

- 安装并验证 WSL2 Ubuntu 22.04、CARLA 0.9.15 Windows Server 和 Python Client。
- 完成交通流、人工驾驶、运行截图与演示视频。

材料见 [`week1-environment/`](../week1-environment/) 和 [Week 1 Release](https://github.com/xuzihao723/xiaomi-auto-drive/releases/tag/week1-submission)。

## Week 2：数据采集与 KITTI 转换

- 配置 RGB 相机与 LiDAR，使用同步模式采集图像、点云和 Actor 真值。
- 生成 1000 帧城市道路数据并转换为 KITTI Object 格式。
- 完成图像、点云、标签和元数据自动验收。

材料见 [`week2-data-pipeline/`](../week2-data-pipeline/) 和 [Week 2 Release](https://github.com/xuzihao723/xiaomi-auto-drive/releases/tag/week2-submission)。

## Week 3：YOLOv8 四类别目标检测

- 在 Town05 与 Town10HD_Opt、多天气、多光照和多随机种子数据上完成四类检测。
- 使用场景/连续帧块严格分组，跨 split 组重叠和精确图像哈希重叠均为 0。
- 以稳定配置完成完整 40 轮训练，最终使用道路参与者和交通控制类别专长双模型融合。
- 严格 test 达到 Precision=0.880、Recall=0.590、mAP50=0.704、mAP50-95=0.403。
- 人工复核独立测试集 150 张图像、414 个现有交通控制目标和 97 个漏标候选；删除 92、重分类 6、新增 36。
- 完成 PyTorch、ONNX、TensorRT 的 CPU/GPU 同口径测速；TensorRT GPU FP16 达到 320.63 FPS。
- 更新 14 页中文实验报告，并对 PDF、代码、JSON、复核标签和最终 ZIP 反复校验。

材料见 [`week3-perception/`](../week3-perception/) 和 [Week 3 Release](https://github.com/xuzihao723/xiaomi-auto-drive/releases/tag/week3-submission)。

## Week 4：U-Net 语义分割（进行中）

目标是构建车道线/可行驶区域像素级数据集，训练 U-Net 分割模型，实现车道拟合与可行驶区域提取，并与第三周目标检测合成 1 分钟集成演示。
