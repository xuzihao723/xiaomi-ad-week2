# Xiaomi Auto Drive - 第三周感知模块

> GitHub 主仓库保留代码、配置、评估证据和实验报告。完整模型权重、ONNX/TensorRT 导出文件及 1 分钟演示视频位于 [Week 3 Release](https://github.com/xuzihao723/xiaomi-auto-drive/releases/tag/week3-submission)。

本提交完成基于 CARLA 城市道路图像的 YOLOv8 四类别检测，类别为 `Car`、`Pedestrian`、`TrafficLight`、`TrafficSign`。最终方案使用道路参与者模型与交通控制模型进行类别专长融合，并针对旧实验报告列出的六项问题逐条完善。

## 六项问题处理结果

| 编号 | 问题 | 当前结论 | 主要证据 |
| --- | --- | --- | --- |
| 1 | CUDA 异常、训练未满 40 轮 | 已解决当前训练任务 | `training/results.csv`、`reports/cuda_stability.json` |
| 2 | 交通控制目标为伪标注 | 阶段性改善 | 人工复核独立 test 的 150 张图像、414 个现有交通控制框和 97 个漏标候选 |
| 3 | TrafficSign 仅 51 个 | 已解决 | 唯一实例增至 1156；严格 test mAP50=0.554 |
| 4 | 随机划分连续帧 | 已解决 | 按场景/连续帧块分组，跨 split 组重叠和精确哈希重叠均为 0 |
| 5 | 单一地图 | 已解决当前阶段 | Town05 + Town10HD_Opt，多天气、光照和随机种子 |
| 6 | 推理速度与部署 | 阶段性改善 | 完成 CPU/GPU、PyTorch/ONNX/TensorRT 同口径笔记本基准 |

## 问题 2：测试集人工复核

人工检查固定的 150 张独立测试图像，逐项复核 414 个现有交通控制目标，并检查 97 个低阈值未匹配候选。原始伪标签保持不覆盖，所有删除、重分类和新增操作均可追溯。

| 项目 | 复核前 | 复核操作 | 复核后 |
| --- | ---: | --- | ---: |
| TrafficLight | 403 | 删除误标、重分类、补漏标 | 338 |
| TrafficSign | 11 | 重分类、补漏标 | 20 |
| 交通控制总数 | 414 | 删除 92、重分类 6、新增 36 | 358 |

同一模型、同一 150 张图像上的指标变化：

| 指标 | 复核前 | 复核后 |
| --- | ---: | ---: |
| 总体 Precision | 0.808 | 0.736 |
| 总体 Recall | 0.536 | 0.611 |
| 总体 mAP50 | 0.640 | 0.645 |
| 总体 mAP50-95 | 0.469 | 0.475 |
| TrafficLight mAP50 | 0.734 | 0.774 |
| TrafficSign mAP50 | 0.338 | 0.320 |

TrafficSign 的测试目标从 11 个增加到 20 个后，mAP50 小幅降低，表明旧结果受漏标影响而偏乐观。该变化应理解为评估更严格，而不是选择性报告“全部提高”。本次没有把测试集用于训练，也没有声称完整 1000 张数据已经全部替换为人工标签。

复核证据位于 `evaluation/traffic_review/`，纠错决策位于 `configs/traffic_review_corrections.json`，复核前后完整指标位于 `reports/evaluation_metrics_fusion_*review*.json`。

复现复核标签视图：

```bash
python tools/apply_traffic_review.py \
  --source data/yolo_carla \
  --control-review evaluation/traffic_review/control_review_manifest.csv \
  --proposal-review evaluation/traffic_review/proposal_review_manifest.csv \
  --corrections configs/traffic_review_corrections.json \
  --output evaluation/traffic_review \
  --dataset-view data/yolo_carla_reviewed
```

## 问题 6：推理速度

测试条件：同一台 RTX 4070 Laptop GPU 笔记本；150 张预加载测试图像；`batch=1`、`imgsz=640`、预热 20 次。计时包含预处理、推理和 NMS/后处理，排除磁盘图像解码；GPU 计时显式同步 CUDA。

| 后端 | 平均延迟 | P95 | FPS |
| --- | ---: | ---: | ---: |
| PyTorch CPU | 18.97 ms | 22.30 ms | 52.71 |
| ONNX CPU | 23.94 ms | 25.72 ms | 41.77 |
| PyTorch GPU | 5.18 ms | 5.83 ms | 192.98 |
| ONNX GPU | 5.65 ms | 7.47 ms | 176.84 |
| TensorRT GPU FP16 | 3.12 ms | 3.72 ms | 320.63 |
| 双模型融合 CPU | 34.35 ms | 38.35 ms | 29.11 |
| 双模型融合 GPU | 9.65 ms | 10.35 ms | 103.60 |

TensorRT FP16 相对 PyTorch GPU 单模型加速 1.66 倍。ONNX 在本机没有快于对应 PyTorch 后端，报告保留该负结果。旧版 6.45 FPS 包含磁盘读取和首次运行等额外开销，不能与本次预加载纯软件推理口径直接混用。

完整记录位于 `reports/inference_benchmarks/` 和 `reports/inference_benchmark_summary.json`；导出模型位于 `weights/exported/`。这些数值不是车载端到端延迟，因为尚未计入真实摄像头、车辆总线、规划控制和执行器响应。

## 严格分组测试结果

严格场景/帧块分组 test 共 300 张图像、3472 个目标；跨 split 组重叠和精确图像哈希重叠均为 0。

| 类别 | Precision | Recall | mAP50 | mAP50-95 |
| --- | ---: | ---: | ---: | ---: |
| Car | 0.952 | 0.829 | 0.857 | 0.732 |
| Pedestrian | 0.885 | 0.804 | 0.882 | 0.615 |
| TrafficLight | 0.839 | 0.322 | 0.522 | 0.196 |
| TrafficSign | 0.843 | 0.404 | 0.554 | 0.071 |
| 总体 | 0.880 | 0.590 | 0.704 | 0.403 |

## 目录说明

- `configs/`：数据集、场景和人工复核配置。
- `detection/`：采集、训练、评估、融合推理和速度基准脚本。
- `evaluation/`：严格测试图表及人工复核证据。
- `reports/`：实验报告、指标 JSON、校验结果和速度对比图。
- `training/`：最终 40 轮训练配置、CSV 和曲线。
- `weights/`：PyTorch 权重以及导出的 ONNX、TensorRT 模型。
- `demo/`：最终系统多场景演示视频。
- `tools/`：数据校验、人工复核、后端检查、导出和报告生成工具。

## 环境与关键命令

验证环境为 Windows 11 + WSL2 Ubuntu 22.04、Python 3.10.12、PyTorch 2.10.0+cu126、Ultralytics 8.4.90、ONNX Runtime GPU 1.22.0、TensorRT 11.1.0.106。

```bash
python -m pip install -r requirements.txt

python detection/evaluate_class_fusion.py \
  --road-user-weights weights/road_user_best.pt \
  --traffic-control-weights weights/traffic_control_best.pt \
  --data configs/data_grouped.yaml --split test \
  --imgsz 640 --batch 8 --device 0 \
  --output-dir evaluation \
  --output-json reports/evaluation_metrics_fusion_test.json

python detection/benchmark_fps.py \
  --weights weights/exported/traffic_control_best_fp16.engine \
  --source data/yolo_carla/images/test \
  --device 0 --warmup 20 --max-images 150 --preload \
  --output-json reports/inference_benchmarks/tensorrt_gpu_fp16.json
```

## 当前局限性

- 完整训练数据仍含伪标注，本次只对独立 test 的 150 张图像进行人工复核。
- 交通灯和交通标志是远距离小目标，mAP50-95 仍低于车辆和行人。
- 双模型融合虽可通过 GPU 加速，但仍比单模型耗时。
- 所有速度结果来自笔记本软件推理，尚无真实车载计算平台的摄像头到控制输出端到端测试。
- Town05 与 Town10HD_Opt 仍不能代表全部真实城市、天气和传感器域。

代码仓库：https://github.com/xuzihao723/xiaomi-auto-drive
