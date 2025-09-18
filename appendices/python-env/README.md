# 🐍 CUCDFD Python 环境

> AI深度学习运行环境配置系统

## 🚀 核心特性

- **🤖 AI模型支持** - PyTorch + CUDA 深度学习环境
- **📦 预打包环境** - 一键部署完整环境
- **⚡ 快速安装** - Conda环境管理优化
- **🔧 依赖管理** - 自动解决包依赖问题
- **🎯 GPU加速** - NVIDIA CUDA 完整支持
- **✅ 验证工具** - 环境完整性自动检测

## 📋 环境要求

| 类型 | 要求 | 推荐配置 |
|------|------|----------|
| 🖥️ **GPU** | NVIDIA GPU | 显存 ≥ 8GB |
| 💾 **内存** | ≥ 8GB RAM | ≥ 16GB RAM |
| 💿 **存储** | ≥ 20GB | ≥ 50GB |
| 🐧 **系统** | Ubuntu 22.04+ | LTS版本 |

## ⚡ 快速安装

### 📦 方法一：预打包环境（推荐）

```bash
# 安装Miniconda
bash Miniconda3-py312_25.1.1-2-Linux-x86_64.sh
conda init bash && source ~/.bashrc

# 部署环境包
mkdir -p ~/miniconda3/envs/cucdfd
tar -xzvf cucdfd.tar.gz -C ~/miniconda3/envs/cucdfd
conda activate cucdfd
conda-unpack

# 验证安装
python -c "import torch; print(f'PyTorch: {torch.__version__}'); print(f'CUDA: {torch.cuda.is_available()}')"
```

### 🔧 方法二：从头创建

```bash
# 创建新环境
conda create -n cucdfd python=3.8
conda activate cucdfd

# 安装核心依赖
conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia
pip install -r cucdfd_service/requirements.txt
```

## ✅ 环境验证

```bash
# 检查核心依赖
python -c "
import torch, torchvision, cv2, numpy, sklearn, mmcv
print('✅ 核心依赖正常')
"

# 检查GPU状态
nvidia-smi
python -c "
import torch
if torch.cuda.is_available():
    print(f'✅ GPU数量: {torch.cuda.device_count()}')
    print(f'✅ GPU型号: {torch.cuda.get_device_name(0)}')
else:
    print('⚠️  CUDA不可用')
"

# 系统工具检查
for tool in screen ffmpeg libreoffice; do
    command -v $tool && echo "✅ $tool 已安装" || echo "❌ $tool 未安装"
done
```

## 🔧 故障排除

### 常见问题

1. **Conda命令未找到**
   ```bash
   ~/miniconda3/bin/conda init bash && source ~/.bashrc
   ```

2. **CUDA版本不匹配**
   ```bash
   # 检查版本
   nvidia-smi
   python -c "import torch; print(torch.version.cuda)"
   
   # 重新安装匹配版本
   conda install pytorch pytorch-cuda=11.7 -c pytorch -c nvidia
   ```

3. **包导入错误**
   ```bash
   # 检查安装状态
   conda list | grep 包名
   
   # 重新安装
   conda install 包名 或 pip install 包名
   ```




## 📊 模块依赖检查

| 模块 | 核心依赖 | 状态检查 |
|------|----------|----------|
| 🤖 **检测服务** | torch, mmcv, mmaction | `python -c "import torch, mmcv"` |
| 🔄 **分配器** | flask, requests, yaml | `python -c "import flask, requests, yaml"` |
| 🌐 **Web前端** | flask | `python -c "import flask"` |
| 🎬 **视频处理** | cv2, PIL | `python -c "import cv2, PIL"` |

## 🛠️ 系统工具

```bash
# 安装必需工具
sudo apt update
sudo apt install -y screen ffmpeg libreoffice

# 验证安装
screen --version
ffmpeg -version
libreoffice --version
```

## 📋 环境检查清单

- ✅ Miniconda已安装
- ✅ cucdfd环境已创建
- ✅ PyTorch + CUDA正常
- ✅ 核心依赖包完整
- ✅ GPU加速可用
- ✅ 系统工具就绪

---

**💡 提示**: 环境配置完成后，使用 `./start_all_simple.sh` 测试系统启动
