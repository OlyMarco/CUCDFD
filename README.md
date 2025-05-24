# CUCDFD 深度鉴伪系统

一个基于深度学习的多媒体内容真伪检测系统，支持视频、音频和图像的深度伪造检测。

## 📋 目录

- [系统概述](#系统概述)
- [系统架构](#系统架构)
- [环境要求](#环境要求)
- [快速开始](#快速开始)
- [详细使用说明](#详细使用说明)
- [配置说明](#配置说明)
- [API接口](#api接口)
- [监控和调试](#监控和调试)
- [故障排除](#故障排除)
- [高级配置](#高级配置)

## 📊 系统概述

CUCDFD 是一个综合性的深度伪造检测系统，集成了多种最先进的检测算法，能够：

- **视频检测**: 检测DeepFake等视频篡改
- **音频检测**: 识别语音合成等音频篡改
- **图像检测**: 发现图像拼接、修复、生成等篡改痕迹
- **AIGC检测**: 识别AI生成的内容

### 🎯 主要特性

- 🔍 多模态检测：支持视频、音频、图像三种媒体类型
- 🚀 分布式架构：支持多GPU分布处理
- 🌐 Web界面：提供友好的前端交互界面
- 📊 详细报告：生成专业的检测报告（PDF格式）
- 🔧 灵活配置：支持动态配置和扩展
- 📈 实时监控：提供系统状态监控和日志记录

## 🏗️ 系统架构

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Web前端服务    │───▶│   分配器服务     │───▶│   检测服务群    │
│   (端口: 8080)   │    │   (端口: 5000)   │    │ (端口: 35331+)  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                       │                       │
        │                       │                       │
        ▼                       ▼                       ▼
  前端静态资源              任务分发调度              AI模型推理
  API地址替换              负载均衡管理              结果生成返回
```

### 核心组件

1. **Web前端服务** (`web/`): 提供用户界面和静态资源服务
2. **分配器服务** (`distributer/`): 任务调度和负载均衡
3. **检测服务** (`cucdfd_service/`): AI模型推理和结果生成

## 💻 环境要求

### 硬件要求
- **GPU**: NVIDIA GPU (推荐显存 ≥ 8GB)
- **内存**: 建议 ≥ 16GB RAM
- **存储**: 建议 ≥ 50GB 可用空间
- **网络**: 稳定的网络连接

### 软件要求
- **操作系统**: Ubuntu 22.04+ / 其他 Linux 发行版
- **Python**: 3.8+ (通过 Conda 管理)
- **CUDA**: 11.7+ (与 PyTorch 版本匹配)
- **系统工具**: `screen`, `git`, `wget`, `ffmpeg`

### 依赖软件
- **Conda/Miniconda**: Python环境管理
- **LibreOffice**: PDF报告生成
- **Screen**: 后台服务管理

## 🚀 快速开始

### 1. 环境准备

#### 安装Conda环境（推荐使用预打包环境）

```bash
# 1. 下载并安装Miniconda（如果未安装）
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
conda init bash
source ~/.bashrc

# 2. 解压预打包环境
mkdir -p ~/miniconda3/envs/cucdfd
tar -xzf cucdfd.tar.gz -C ~/miniconda3/envs/cucdfd

# 3. 激活并解包环境
conda activate cucdfd
conda-unpack

# 4. 验证环境
python -c "import torch; print(f'PyTorch: {torch.__version__}, CUDA: {torch.cuda.is_available()}')"
```

#### 安装系统依赖

```bash
# 安装必要的系统工具
sudo apt update
sudo apt install screen git wget ffmpeg

# 验证安装
screen --version
ffmpeg -version
```

### 2. 进入项目

```bash
# 进入项目目录
cd /home/cucdfd/桌面/cucdfd

# 设置脚本执行权限
chmod +x start_all.sh stop_all.sh
```

### 3. 配置系统

编辑主配置文件 `config.yml`：

```yaml
# 分配器配置
distributer_module:
  port: 5000
  worker_servers:
    - name: "service_1"
      url: "http://0.0.0.0:35331"

# 服务配置
service_module:
  conda_env: "cucdfd"
  services:
    - name: "service_1"
      port: 35331
      gpu_id: 0

# Web前端配置
web_module:
  enabled: true
  port: 8080
  api_url: "http://localhost:5000"
```

### 4. 启动系统

```bash
# 启动所有服务
./start_all.sh

# 查看启动状态
screen -ls

# 访问前端界面
# 打开浏览器访问: http://localhost:8080
```

### 5. 停止系统

```bash
# 停止所有服务
./stop_all.sh

# 验证停止状态
screen -ls
```

## 📚 详细使用说明

### 系统启动流程

1. **环境检查**: 验证Conda环境和系统依赖
2. **服务启动**: 按顺序启动检测服务实例
3. **分配器启动**: 启动任务调度服务
4. **前端启动**: 启动Web界面服务
5. **状态验证**: 检查所有服务运行状态

### 任务处理流程

```
用户上传文件 → Web前端 → 分配器 → 检测服务 → AI模型推理 → 结果返回
     ↓              ↓        ↓        ↓          ↓
   文件选择     API转发   负载均衡   模型加载    报告生成
```

### 支持的文件类型

| 媒体类型 | 支持格式 | 最大文件大小 |
|---------|---------|-------------|
| 视频 | mp4 | 10MB |
| 图像 | png, jpg, jpeg, tif, webp | 10MB |
| 音频 | wav, m4a, pcm, mp3, flac | 10MB |

### Web界面使用

1. **访问地址**: http://localhost:8080
2. **上传文件**: 选择要检测的媒体文件
3. **提交任务**: 点击检测按钮开始处理
4. **查看进度**: 实时显示处理进度
5. **获取结果**: 下载检测报告和查看详细结果

## ⚙️ 配置说明

### 主配置文件 (`config.yml`)

```yaml
# 分配器模块配置
distributer_module:
  port: 5000                    # 分配器监听端口
  worker_servers:               # 工作服务器列表
    - name: "service_1"
      url: "http://0.0.0.0:35331"

# 服务模块配置  
service_module:
  conda_env: "cucdfd"           # Conda环境名称
  services:                     # 服务实例列表
    - name: "service_1"         # 服务名称
      port: 35331               # 服务端口
      gpu_id: 0                 # GPU设备ID

# Web前端模块配置
web_module:
  enabled: true                 # 是否启用Web服务
  port: 8080                    # Web服务端口
  api_url: "http://localhost:5000"  # API后端地址

# 全局设置
global:
  project_root: ""              # 项目根目录（自动检测）
  log_dir: "logs"               # 日志目录
  screen_prefix: "cucdfd"       # Screen会话前缀
```

### 多实例配置

支持配置多个检测服务实例以提高并发处理能力：

```yaml
service_module:
  services:
    - name: "service_1"
      port: 35331
      gpu_id: 0
    - name: "service_2"  
      port: 35332
      gpu_id: 1
    - name: "service_3"
      port: 35333
      gpu_id: 0          # 多个服务可共享GPU
```

对应更新分配器配置：

```yaml
distributer_module:
  worker_servers:
    - name: "service_1"
      url: "http://0.0.0.0:35331"
    - name: "service_2"
      url: "http://0.0.0.0:35332"
    - name: "service_3"
      url: "http://0.0.0.0:35333"
```

## 🔌 API接口

### 分配器接口

#### 提交检测任务
```bash
curl -X POST http://localhost:5000/taskdoor \
  -F "file=@example.mp4"
```

#### 查询任务状态
```bash
curl -X POST http://localhost:5000/check \
  -d "taskid=abc123"
```

#### 获取服务器状态
```bash
curl http://localhost:5000/simplecheck
```

### 检测服务接口

#### 提交任务到服务
```bash
curl -X POST http://localhost:35331/taskget \
  -F "file=@example.mp4" \
  -F "taskid=abc123"
```

#### 检查任务结果
```bash
curl -X POST http://localhost:35331/resultcheck \
  -d "taskid=abc123"
```

### 响应格式

**任务进行中**:
```json
{
  "status": 0.5,
  "message": "处理中"
}
```

**任务完成**:
```json
{
  "status": 1.0,
  "data": {
    "result_all": "伪造",
    "scores_all": "0.8556",
    "mvss_pic": "/static/abc123/mvss_plot.png",
    "result_pdf": "/static/abc123/Forensic_Report.pdf"
  }
}
```

## 📊 监控和调试

### 查看运行状态

```bash
# 查看所有Screen会话
screen -ls

# 连接到特定服务查看日志
screen -r cucdfd_distributer    # 分配器
screen -r cucdfd_svc_35331      # 检测服务
screen -r cucdfd_web            # Web前端

# 查看系统日志
tail -f logs/startup.log        # 启动日志
tail -f logs/shutdown.log       # 关闭日志
```

### 端口监控

```bash
# 检查端口占用情况
ss -tuln | grep -E "(5000|8080|35331)"

# 检查进程状态
ps aux | grep -E "(distributer|service_api|web.py)"

# GPU使用情况
watch -n 1 nvidia-smi
```

### 日志文件位置

```
logs/
├── startup.log          # 系统启动日志
└──  shutdown.log         # 系统关闭日志
```

## 🔧 故障排除

### 常见问题

#### 1. Conda环境问题
```bash
# 问题：conda: command not found
# 解决：
~/miniconda3/bin/conda init bash
source ~/.bashrc

# 问题：环境激活失败
# 解决：
conda env list
conda activate cucdfd
```

#### 2. 端口冲突
```bash
# 检查端口占用
ss -tuln | grep :5000

# 修改配置文件中的端口号
vim config.yml
```

#### 3. GPU不可用
```bash
# 检查CUDA安装
nvidia-smi
python -c "import torch; print(torch.cuda.is_available())"

# 重新安装PyTorch
conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia
```

#### 4. 服务启动失败
```bash
# 查看详细错误信息
screen -r cucdfd_distributer

# 检查配置文件
cat config.yml

# 重新启动服务
./stop_all.sh
./start_all.sh
```

#### 5. Web界面无法访问
```bash
# 检查Web服务状态
screen -r cucdfd_web

# 检查端口监听
ss -tuln | grep :8080

# 检查防火墙设置
sudo ufw status
```

### 重置系统

如果遇到严重问题，可以完全重置：

```bash
# 1. 停止所有服务
./stop_all.sh

# 2. 清理Screen会话
screen -wipe

# 3. 清理进程
pkill -f "distributer.py"
pkill -f "service_api.py"
pkill -f "web.py"

# 4. 重新启动
./start_all.sh
```

## 🔬 高级配置

### 性能优化

#### GPU内存优化
```bash
# 设置环境变量
export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:128
export OMP_NUM_THREADS=4
```

#### 并发处理优化
```yaml
# 增加服务实例数量
service_module:
  services:
    - name: "service_1"
      port: 35331
      gpu_id: 0
    - name: "service_2"
      port: 35332
      gpu_id: 0
    - name: "service_3"
      port: 35333
      gpu_id: 1
```

### 分布式部署

支持将服务部署在不同的机器上：

```yaml
distributer_module:
  worker_servers:
    - name: "server1_service1"
      url: "http://192.168.1.101:35331"
    - name: "server2_service1" 
      url: "http://192.168.1.102:35331"
```

### 安全配置

#### API密钥认证
```python
# 在分配器中设置API密钥
API_KEY = "your_secure_api_key"
```

#### 网络访问控制
```bash
# 使用防火墙限制访问
sudo ufw allow from 192.168.1.0/24 to any port 5000
```

## 📁 项目结构

```
cucdfd/
├── config.yml                 # 主配置文件
├── start_all.sh               # 系统启动脚本
├── stop_all.sh                # 系统停止脚本
├── README.md                  # 本文档
├── logs/                      # 日志目录
├── distributer/               # 分配器服务
│   ├── distributer.py
│   ├── utils/
│   └── README.md
├── cucdfd_service/            # 检测服务
│   ├── service_api.py
│   ├── service.py
│   ├── mmaction2/
│   ├── MABC_Net/
│   ├── SSDGSpeech5/
│   ├── word_proc_master/
│   └── README.md
├── web/                       # Web前端
│   ├── web.py
│   ├── start_web.sh
│   ├── stop_web.sh
│   ├── index.html
│   └── README.md
└── appendices/                # 附加文档
    ├── environment/           # 预打包环境说明
    ├── fonts/                 # 缺失字体说明
    └── libreoffice/           # libreoffice缺失说明
```

## 🆘 获取帮助

### 文档资源
- [环境配置指南](appendices/environment/env%20setup.md)
- [LibreOffice安装](appendices/libreoffice/libreoffice%20setup.md)
- [字体配置](appendices/fonts/Fonts%20setup.md)

### 常用命令参考

```bash
# 系统管理
./start_all.sh                 # 启动整个系统
./stop_all.sh                  # 停止整个系统
screen -ls                     # 查看所有会话
screen -r <session_name>       # 连接到指定会话

# 日志查看
tail -f logs/startup.log       # 实时查看启动日志
grep "ERROR" logs/*.log        # 搜索错误信息

# 状态检查
ss -tuln | grep -E "(5000|8080|35331)"  # 检查端口
ps aux | grep python           # 检查Python进程
nvidia-smi                     # 检查GPU状态
```

---

**注意**: 
- 确保所有脚本有执行权限: `chmod +x *.sh`
- 首次使用请仔细阅读环境配置指南
- 生产环境部署建议配置防火墙和访问控制
- 如遇问题请先查看日志文件和故障排除部分

**版本**: v1.0  
**更新日期**: 2025年M5  
**维护**: CUCDFD
