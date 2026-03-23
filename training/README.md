# 📋 训练指南 (Training Guide)

本文档旨在帮助你使用本项目提供的核心训练脚本，进行 YOLO 模型的训练、断点续训及批量实验。

---

## 1. 环境准备

确保你已经按照项目根目录 `README.md` 的说明安装了 Python 环境及依赖库。

```bash
conda activate task1  # 激活你的环境
pip install ultralytics
```

---

## 2. 脚本说明

`training/` 目录下包含两个主要脚本：

*   **`train.py`**:  **单模型训练脚本**。适用于日常训练、调试单个模型，支持断点续训。
*   **`train_batch.py`**: **批量实验脚本**。适用于需要一次性训练多个模型（如 YOLOv8/11/26 全系列对比）并统计耗时的场景。

---

## 3. 使用方法

### 3.1 训练单个模型 (`train.py`)

这是最常用的训练方式。你可以指定模型版本、数据集路径、训练轮数等参数。

**基本用法：**

```bash
# 使用默认参数训练 YOLOv8n (默认 epochs=100, batch=16)
python train.py --model yolov8n.pt
```

**自定义参数：**

```bash
# 训练 YOLO11s，设置 50 轮，Batch Size 为 32，使用 GPU 0
python train.py --model yolo11s.pt --epochs 50 --batch 32 --device 0
```

**断点续训 (Resume)：**

如果训练意外中断（如断电、程序崩溃），可以使用 `--resume` 参数从上次保存的检查点继续训练，无需从头开始。

```bash
# 从上次中断的地方继续训练
python train.py --model runs/train/Human_face_emotions/yolo11s_face_emotions/weights/last.pt --resume
```

### 3.2 批量训练 (`train_batch.py`)

如果你想复现论文中的实验，对比不同系列模型的性能，可以使用此脚本。它会依次训练列表中的所有模型，并生成一份详细的耗时报告 (`training_summary_xxx.json`)。

**用法：**

1.  打开 `train_batch.py`，在底部的 `models_to_run` 列表中修改你想要训练的模型（注释掉不需要的）。
2.  运行脚本：

```bash
# 训练所有模型，100轮 (自动批量大小)
python train_batch.py --epochs 100

# 快速测试，10轮
python train_batch.py --epochs 10
```

---

## 4. 训练输出文件

训练完成后，会在 `runs/train/{data_name}/` 目录下生成以下文件：

### 4.1 日志文件

| 文件名 | 说明 |
|--------|------|
| `train.log` | 训练过程日志，包含 GPU 信息、训练开始/结束时间等 |
| `training_summary_YYYYMMDD_HHMMSS.json` | JSON 格式的训练总结，包含各模型耗时、状态等信息 |

### 4.2 训练总结文档

| 文件名 | 说明 |
|--------|------|
| `summary.md` | Markdown 格式的训练总结，包含 GPU 信息、模型训练时间表格、成本估算等 |

### 4.3 各模型训练结果

每个模型的训练结果保存在 `runs/train/{data_name}/{model_name}_face_emotions/` 目录下：

| 文件名 | 说明 |
|--------|------|
| `weights/best.pt` | 最佳模型权重 |
| `weights/last.pt` | 最后一个 epoch 的权重 |
| `results.csv` | 训练指标 CSV 文件 |
| `results.png` | 训练曲线图 |
| `train.log` | 该模型的训练日志 |

---

## 5. 常见问题

*   **Q: 显存不足 (CUDA Out of Memory) 怎么办？**
    *   A: 减小 `--batch` 参数的值（例如从 16 改为 8 或 4）。
    *   A: 减小 `--imgsz` 参数的值（例如从 640 改为 416）。

*   **Q: 训练结果保存在哪里？**
    *   A: 默认保存在 `runs/train/` 目录下。每个实验会根据模型名称生成一个子文件夹，例如 `runs/train/Human_face_emotions/yolo11s_face_emotions`。

*   **Q: 如何查看训练过程的可视化图表？**
    *   A: Ultralytics 会在 `runs/train/xxx/` 目录下自动生成 `results.png` 等图表。
    *   A: 你也可以使用本项目提供的 Streamlit App 的"模型性能对比分析"功能，直接读取 `results.csv` 进行交互式分析。

*   **Q: 如何估算训练成本？**
    *   A: 训练脚本会自动根据 GPU 类型和训练耗时计算预计成本，结果保存在 `summary.md` 中。

---

## 6. 参数详解

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| `--model` | `yolov8n.pt` | 模型权重文件路径或名称 |
| `--data` | `../data/cable_damage/data.yaml` | 数据集配置文件路径 |
| `--epochs` | `100` | 训练总轮数 |
| `--batch` | `-1` | 批次大小 (`-1` 表示自动批量大小) |
| `--imgsz` | `640` | 输入图片尺寸 |
| `--device` | `0` | 训练设备 (0, 1... 或 cpu) |
| `--resume` | `False` | (仅 train.py) 是否开启断点续训 |

---

## 7. 数据集说明

本项目使用 Roboflow 开源数据集，包含 8 类人脸情绪：

| 英文名 | 中文名 |
|--------|--------|
| anger | 愤怒 |
| content | 满足 |
| disgust | 厌恶 |
| fear | 恐惧 |
| happy | 开心 |
| neutral | 中性 |
| sad | 悲伤 |
| surprise | 惊讶 |

数据集目录结构：
```
data/Human_face_emotions/
├── train/images/      # 训练集图片
├── train/labels/      # 训练集标签
├── valid/images/      # 验证集图片
├── valid/labels/      # 验证集标签
├── test/images/       # 测试集图片
├── test/labels/       # 测试集标签
└── data.yaml          # 数据集配置文件
```
