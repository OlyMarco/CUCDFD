# CUCDFD 深度鉴伪系统环境配置指南

## 概述

本指南提供完整的 CUCDFD 系统环境配置说明，包括 Conda 环境的安装、配置和故障排除。

## 环境要求

### 硬件要求
- **GPU**: NVIDIA GPU (推荐显存 ≥ 8GB)
- **内存**: 建议 ≥ 16GB RAM
- **存储**: 建议 ≥ 50GB 可用空间

### 软件要求
- **操作系统**: Ubuntu 22.04+ / 其他 Linux 发行版
- **Python**: 3.8+ (通过 Conda 管理)
- **CUDA**: 11.7+ (与 PyTorch 版本匹配)
- **Git**: 用于代码管理

## 方法一：使用预打包环境（推荐）

### 1. 获取环境包

确保您已获得 `cucdfd.tar.gz` 环境包文件。

### 2. 安装 Miniconda

如果系统中没有 Conda，请先安装：

```bash
# 下载 Miniconda 安装脚本
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# 安装 Miniconda
bash Miniconda3-latest-Linux-x86_64.sh

# 初始化 Conda（需要重新启动终端或 source ~/.bashrc）
conda init bash
source ~/.bashrc
```

### 3. 解压并配置环境

```bash
# 创建目标目录
mkdir -p ~/miniconda3/envs/cucdfd

# 解压环境包到指定目录
tar -xzf cucdfd.tar.gz -C ~/miniconda3/envs/cucdfd

# 激活环境
conda activate cucdfd

# 解包环境（重要步骤）
conda-unpack

# 验证环境是否正确配置
python -c "import torch; print(f'PyTorch version: {torch.__version__}'); print(f'CUDA available: {torch.cuda.is_available()}')"
```

### 4. 验证安装

```bash
# 检查关键依赖包
python -c "
import torch
import torchvision
import cv2
import numpy as np
import sklearn
import mmcv
print('所有关键依赖包已正确安装')
"

# 检查 CUDA 和 GPU
nvidia-smi
python -c "
import torch
if torch.cuda.is_available():
    print(f'GPU 数量: {torch.cuda.device_count()}')
    for i in range(torch.cuda.device_count()):
        print(f'GPU {i}: {torch.cuda.get_device_name(i)}')
else:
    print('警告: CUDA 不可用')
"
```

## 方法二：从头创建环境

```bash
# 切换到项目目录
cd /home/cucdfd/桌面/cucdfd

# 创建新的conda环境
conda create -n cucdfd python=3.8

# 激活环境
conda activate cucdfd

# 安装项目依赖（参考cucdfd_service的requirements.txt）
# 请根据实际的requirements.txt文件安装依赖包
```

## 故障排除

### 常见问题及解决方案

#### 1. Conda 环境激活失败

```bash
# 问题：bash: conda: command not found
# 解决：重新初始化 conda
~/miniconda3/bin/conda init bash
source ~/.bashrc
```

#### 2. CUDA 版本不匹配

```bash
# 检查 CUDA 版本
nvidia-smi

# 检查 PyTorch CUDA 版本
python -c "import torch; print(torch.version.cuda)"

# 重新安装匹配的 PyTorch
conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia
```

#### 3. conda-unpack 命令不存在

```bash
# 安装 conda-pack
conda install conda-pack -c conda-forge

# 或者跳过 unpack 步骤，直接尝试使用环境
```

#### 4. 导入错误

```bash
# 问题：ModuleNotFoundError
# 解决：检查包是否正确安装
conda list | grep 包名
pip list | grep 包名

# 重新安装缺失的包
conda install 包名
# 或
pip install 包名
```

#### 5. 权限问题

```bash
# 确保对 conda 环境目录有写权限
chmod -R u+w ~/miniconda3/envs/cucdfd/
```

### 环境重建

如果环境损坏，可以重新创建：

```bash
# 删除旧环境
conda deactivate
conda env remove -n cucdfd

# 重新解压或创建环境
# 按照上述步骤重新配置
```

## 性能优化建议

### 1. 内存优化

```bash
# 设置环境变量优化内存使用
export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:128
export OMP_NUM_THREADS=4
```

### 2. GPU 优化

```bash
# 检查 GPU 状态
watch -n 1 nvidia-smi

# 设置 CUDA 设备
export CUDA_VISIBLE_DEVICES=0  # 使用第一块 GPU
```

### 3. 网络优化

```bash
# 设置代理（如果需要）
export http_proxy=http://proxy:port
export https_proxy=http://proxy:port
```

## 环境备份

### 导出环境

```bash
# 导出 conda 环境
conda env export > cucdfd_environment.yml

# 导出 pip 包列表
pip freeze > requirements.txt

# 打包整个环境
conda pack -n cucdfd -o cucdfd.tar.gz
```

### 恢复环境

```bash
# 从 YAML 文件恢复
conda env create -f cucdfd_environment.yml

# 从 tar.gz 包恢复
mkdir -p ~/miniconda3/envs/cucdfd
tar -xzf cucdfd.tar.gz -C ~/miniconda3/envs/cucdfd
conda activate cucdfd
conda-unpack
```

## 系统集成验证

### 验证CUCDFD系统环境

```bash
# 进入项目目录
cd /home/cucdfd/桌面/cucdfd

# 激活环境
conda activate cucdfd

# 检查检测服务依赖
cd cucdfd_service
python -c "
try:
    import torch
    import mmcv
    import mmaction
    print('✓ 视频检测模块依赖正常')
except ImportError as e:
    print(f'✗ 视频检测模块依赖缺失: {e}')

try:
    from flask import Flask
    print('✓ API服务依赖正常')
except ImportError as e:
    print(f'✗ API服务依赖缺失: {e}')

try:
    import cv2
    import PIL
    print('✓ 图像处理依赖正常')
except ImportError as e:
    print(f'✗ 图像处理依赖缺失: {e}')
"

# 检查分配器依赖
cd ../distributer
python -c "
try:
    import flask
    import requests
    import yaml
    print('✓ 分配器服务依赖正常')
except ImportError as e:
    print(f'✗ 分配器服务依赖缺失: {e}')
"

# 检查Web前端依赖
cd ../web
python -c "
try:
    import flask
    print('✓ Web前端依赖正常')
except ImportError as e:
    print(f'✗ Web前端依赖缺失: {e}')
"
```

### 系统工具验证

```bash
# 验证必需的系统工具
echo "检查系统工具..."

# 检查Screen
if command -v screen &> /dev/null; then
    echo "✓ Screen 已安装: $(screen --version | head -1)"
else
    echo "✗ Screen 未安装，请运行: sudo apt install screen"
fi

# 检查FFmpeg
if command -v ffmpeg &> /dev/null; then
    echo "✓ FFmpeg 已安装: $(ffmpeg -version | head -1)"
else
    echo "✗ FFmpeg 未安装，请运行: sudo apt install ffmpeg"
fi

# 检查LibreOffice
if command -v libreoffice &> /dev/null; then
    echo "✓ LibreOffice 已安装: $(libreoffice --version)"
else
    echo "✗ LibreOffice 未安装，请运行: sudo apt install libreoffice"
fi
```

---

**注意**: 请确保在配置过程中保持网络连接稳定，某些包的下载可能需要较长时间。配置完成后，建议使用项目根目录的 `./start_all.sh` 脚本测试整个系统的启动。