# Jasna-Colab
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/CosmicNexusV/Jasna-Colab/blob/main/Jasna-Colab.ipynb) <br>

在 Google Colab 上一键部署并运行 [Jasna](https://github.com/Kruk2/jasna) 视频马赛克修复工具的 Jupyter Notebook。

---

## ✨ 功能特性

- 自动安装系统依赖及支持 NVENC/NVDEC 硬件加速的 FFmpeg
- 自动从源码编译安装 Jasna 及其底层 Nvidia 依赖（`vali`、`PyNvVideoCodec`）
- 自动挂载 Google Cloud Storage Bucket，用于持久化存储模型权重和引擎文件
- 自动下载官方预训练模型权重（修复模型 + 检测模型）
- 支持挂载 Google Drive，直接读取/保存视频
- 支持**批量处理**多个文件夹，自动跳过已处理文件
- 实时进度条显示处理进度、速度与剩余时间

---

## 🔧 前置要求

| 要求 | 说明 |
|------|------|
| Google Colab | 需选择 **GPU 运行时**（T4 / L4 / A100） |
| Google Cloud Storage | 用于持久化存储模型权重，避免每次重新下载 |
| Google Drive | 可选，用于存储较大视频文件 |
| Hugging Face | 修复模型权重从 HuggingFace 下载，需要网络可访问 |

> ⚠️ 请在 Colab 菜单中选择：`运行时` → `更改运行时类型` → `T4/L4 GPU`

---

## 📒 Notebook 步骤说明

| 步骤 | 说明 |
|------|------|
| 1 | 检查 GPU 信息（`nvidia-smi`） |
| 2 | 安装 `mkvtoolnix`（提供 `mkvmerge`） |
| 3 | 安装替换为支持 NVENC/NVDEC 的静态构建 FFmpeg |
| 4 | 从源码编译安装 Jasna 及底层依赖（`vali`、`PyNvVideoCodec`） |
| 5 | 验证 Jasna 是否可用（`python -m jasna --help`） |
| 6 | 修复 `LD_LIBRARY_PATH`，确保 PyTorch 正确识别 GPU |
| 7 | 挂载 Google Cloud Storage Bucket 并创建模型权重软链接 |
| 8 | 下载预训练模型权重（修复模型 + `rfdetr-v4` 检测模型） |

---

## 🚀 使用方法

### 单视频处理

```python
%cd /content/jasna

!./.venv/bin/python -m jasna \
  --input "/content/input_video.mp4" \
  --output "/content/output_video.mp4" \
  --max-clip-size 180 \
  --detection-model rfdetr-v4
```

也可以指定 Drive 中的路径：

```python
from google.colab import drive
drive.mount('/content/drive')

!./.venv/bin/python -m jasna \
    --input "/content/drive/MyDrive/input.mp4" \
    --output "/content/drive/MyDrive/output.MR.mp4" \
    --max-clip-size 180 \
    --temporal-overlap 10 \
    --encoder-settings "cq=28" \
    --detection-model rfdetr-v4
```

### 批量处理（Google Drive）

1. 运行挂载 Drive 的单元格
2. 修改批量处理预览单元格中的 `FOLDERS` 列表：

```python
FOLDERS = [
    "/content/drive/MyDrive/路径1/文件夹1",
    "/content/drive/MyDrive/路径2/文件夹2",
]
```

3. 运行预览单元格确认待处理情况，再运行批量处理单元格，脚本会：
   - 将视频复制到 Colab 本地磁盘（`/content/video_work`）以加速 I/O
   - 依次处理每个视频，输出文件名格式为 `原文件名.MR.mp4`
   - 自动跳过已存在输出的文件
   - 处理完成后将结果复制回 Drive 并清理本地临时文件

---

## 📦 模型权重说明

| 文件 | 用途 |
|------|------|
| `lada_mosaic_restoration_model_generic_v1.2.pth` | 马赛克修复（通用 v1.2） |
| `rfdetr-v4.onnx` | 马赛克检测（rfdetr v4） |

权重来源：
- 修复模型：[HuggingFace - ladaapp/lada](https://huggingface.co/ladaapp/lada)
- 检测模型：[Jasna Releases v0.5.0-alpha3](https://github.com/Kruk2/jasna/releases/tag/v0.5.0-alpha3)

---

## ⚠️ 注意事项

- Colab 免费版有会话时长限制，处理大量视频请使用 Colab Pro
- 每次新建会话都需要重新运行安装步骤（步骤 1-8），编译耗时较长，请耐心等待
- 模型权重通过 GCS Bucket 软链接持久化，无需每次重新下载
- 视频临时存放在 `/content/video_work`，会话结束后自动清除，请确保及时复制回 Drive
- 编译 `vali` 时需要指定 `PKG_CONFIG_PATH` 以使用最新版 FFmpeg 头文件

---

## 🔗 参考链接

- [Jasna 官方仓库](https://github.com/Kruk2/jasna)
- [vali（Nvidia 视频解码库）](https://codeberg.org/Kruk2/vali)
- [PyNvVideoCodec](https://codeberg.org/Kruk2/PyNvVideoCodec)
- [BtbN FFmpeg Builds](https://github.com/BtbN/FFmpeg-Builds)
- [ladaapp/lada HuggingFace 模型](https://huggingface.co/ladaapp/lada)
