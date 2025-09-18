# 🤖 CUCDFD 深度鉴伪系统

> ## 📁 支持格式

| 类型 | 格式 | 大小限制 |
|------|------|----------|
| 🎬 **视频** | mp4, avi, mov, mkv | 100MB |
| 🖼️ **图像** | jpg, png, jpeg, tif, webp | 50MB |
| 🎵 **音频** | wav, mp3, m4a, pcm, flac | 50MB |
| 📄 **文本** | txt, doc, docx, pdf | 10MB |

## 🏭 环境要求度伪造检测平台

## � 核心特性

- **🎬 视频检测** - DeepFake视频智能识别
- **� 音频检测** - 语音合成和音频篡改检测  
- **🖼️ 图像检测** - AI生成图像和篡改识别
- **🔄 多模态分析** - 音视频联合鉴伪技术
- **🌐 Web界面** - 桌面和移动端友好界面
- **📊 专业报告** - 可视化图表和PDF详细报告

## 🏗️ 系统架构

```
🌐 Web前端 → � 分配器 → 🤖 AI检测服务群
    ↓           ↓           ↓
  用户界面     任务调度     深度学习推理
  文件上传     负载均衡     结果生成
```

## 📁 支持格式

| 类型 | 格式 | 大小限制 |
|------|------|----------|
| 🎬 **视频** | mp4 | 10MB |
| 🖼️ **图像** | jpg, png, jpeg, tif, webp | 10MB |
| 🎵 **音频** | wav, mp3, m4a, pcm, flac | 10MB |

## � 环境要求

| 组件 | 最低配置 | 推荐配置 |
|------|----------|----------|
| 🖥️ **GPU** | NVIDIA ≥8GB | NVIDIA ≥16GB |
| 💾 **内存** | 8GB RAM | 16GB+ RAM |
| 💿 **存储** | 20GB | 50GB+ |
| � **系统** | Ubuntu 22.04+ | LTS版本 |

## ⚡ 快速启动

### 🔧 环境配置

```bash
# 1. 安装Miniconda
bash appendices/python-env/Miniconda3-py312_25.1.1-2-Linux-x86_64.sh
conda init bash && source ~/.bashrc

# 2. 部署Python环境
mkdir -p ~/miniconda3/envs/cucdfd
tar -xzvf appendices/python-env/cucdfd.tar.gz -C ~/miniconda3/envs/cucdfd
conda activate cucdfd && conda-unpack

tar -xzf cucdfd.tar.gz -C ~/miniconda3/envs/

# 3. 激活环境
conda activate cucdfd

# 4. 安装字体支持
cd appendices/fonts && unzip Fonts.zip && bash install_fonts.sh

# 5. 安装LibreOffice
cd ../libreoffice && bash install_libreoffice.sh
```

### 🚀 启动系统

```bash
# 一键启动所有服务（简化版本）
./start_all.sh

# 验证服务状态
screen -ls | grep cucdfd        # 查看后台服务
curl http://localhost:8080      # 测试Web界面
curl http://localhost:5000/simplecheck  # 测试分配器
```

### 🔍 检测使用

```bash
# Web界面访问
http://localhost:8080

# API接口示例
curl -X POST http://localhost:5000/taskdoor \
  -F "file=@test.jpg"
  
# 查询检测结果
curl -X POST http://localhost:5000/check \
  -d "taskid=YOUR_TASK_ID"
```

### 🛑 关闭系统

```bash
# 一键关闭所有服务
./stop_all.sh

# 清理系统缓存
python clean.py

### 🔗 访问系统

- **🖥️ 桌面版**: http://localhost:8080
- **📱 移动版**: http://localhost:8080/wap.html  
- **🎯 检测面板**: http://localhost:8080/panel

## 🔄 工作流程

1. **📤 文件上传** - Web界面提交检测任务
2. **🔄 任务调度** - 分配器智能分发到可用节点
3. **🤖 AI推理** - 多模型并行检测分析
4. **📊 结果生成** - 可视化图表和PDF报告
5. **📥 结果展示** - Web界面实时显示检测结果


## 📂 项目结构

```
cucdfd/
├── 🚀 start_all.sh             # 系统启动脚本
├── 🛑 stop_all.sh              # 系统停止脚本
├── 🧹 clean.py                 # 系统清理脚本
├── ⚙️ config.yml               # 主配置文件
├── 📝 README.md                # 本文档
├── 🌐 web/                     # Web前端服务
│   ├── 📄 web.py               # Web服务器
│   ├── 🚀 start_web_simple.sh  # Web启动脚本
│   ├── 🛑 stop_web_simple.sh   # Web停止脚本
│   └── 🖥️ index.html           # 主页面
├── 🔄 distributer/             # 任务分配器
│   ├── 📄 distributer.py       # 分配器主程序
│   ├── 🚀 start_distributer.sh # 分配器启动脚本
│   ├── 🛑 stop_distributer.sh  # 分配器停止脚本
│   └── ⚙️ config.yml           # 分配器配置
├── 🤖 cucdfd_service/          # AI检测服务
│   ├── 📄 service_api.py       # 检测服务API
│   ├── 🚀 start_service.sh     # 服务启动脚本
│   ├── 🛑 stop_service.sh      # 服务停止脚本
│   └── ⚙️ config.yml           # 服务配置
└── 📦 appendices/              # 环境依赖包
    ├── 🐍 python-env/          # Python环境包
    ├── 📄 libreoffice/         # LibreOffice安装包
    ├── 🔤 fonts/               # 字体包
    └── 🖥️ screen/              # Screen工具包
```

## 🔌 核心接口

### 🔄 分配器API (端口:5000)

| 接口 | 方法 | 功能 | 示例 |
|------|------|------|------|
| `/taskdoor` | POST | 📤 提交检测任务 | `file=@test.jpg` |
| `/check` | POST | 🔍 查询任务结果 | `taskid=abc123` |
| `/simplecheck` | GET | 📊 节点状态检查 | 无参数 |

### 🤖 检测服务API (端口:35331/35332)

| 接口 | 方法 | 功能 | 参数 |
|------|------|------|------|
| `/taskget` | POST | 📤 接收检测任务 | `file`, `taskid` |
| `/resultcheck` | POST | 📥 查询处理结果 | `taskid` |
| `/statuscheck` | GET | ✅ 服务状态检查 | 无参数 |

### 🌐 Web界面 (端口:8080)

- 📱 **主界面**: `http://localhost:8080/`
- 📄 **移动端**: `http://localhost:8080/wap.html`

## 🛠️ 管理工具

### 📊 服务监控

```bash
# 查看后台服务状态
screen -ls | grep cucdfd

# 进入服务会话查看实时输出
screen -r cucdfd_distributer    # 分配器服务
screen -r cucdfd_svc_35331      # 检测服务1
screen -r cucdfd_svc_35332      # 检测服务2 
screen -r cucdfd_web            # Web前端

# 查看服务日志
tail -f logs/distributer.log    # 分配器日志
tail -f logs/service_35331.log  # 服务1日志
tail -f logs/web.log            # Web日志

# 系统资源监控
nvidia-smi                      # GPU使用情况
htop                            # CPU/内存使用
ss -tuln | grep -E ':5000|:8080|:35331|:35332'  # 端口监听
```

### 🧹 系统维护

```bash
# 完整系统清理
python clean.py                 # 清理所有缓存和临时文件

# 模块清理
cd distributer && python clean.py      # 清理分配器
cd cucdfd_service && python clean.py   # 清理检测服务

# 手动清理
rm -rf logs/*.log               # 清理日志文件
rm -rf /tmp/cucdfd_*           # 清理临时文件
```

### ⚙️ 配置管理

```bash
# 查看当前配置
cat config.yml                 # 主配置文件
cat distributer/config.yml     # 分配器配置
cat cucdfd_service/config.yml  # 服务配置

# 修改服务端口（需重启生效）
vim config.yml                 # 编辑配置后重启服务
```

## 📊 故障排除

### 🚨 常见问题

| 问题 | 症状 | 解决方案 |
|------|------|----------|
| **端口被占用** | 启动失败 | `ss -tuln \| grep :端口号` 查看占用，`kill -9 PID` 终止 |
| **GPU不可用** | CUDA错误 | `nvidia-smi` 检查驱动，`conda activate cucdfd` 重新激活环境 |
| **服务无响应** | 超时错误 | `screen -r cucdfd_服务名` 查看日志，重启对应服务 |
| **文件上传失败** | 413错误 | 检查文件大小限制(50MB)，确认格式支持 |
| **环境问题** | 导入错误 | 重新激活conda环境，检查依赖包完整性 |

### 🔧 快速诊断

```bash
# 一键系统检查
./start_all.sh --check        # 检查系统状态
screen -ls | grep cucdfd      # 查看服务运行
ss -tuln | grep -E ':5000|:8080|:35331|:35332'  # 检查端口

# 服务重启
./stop_all.sh && sleep 3 && ./start_all.sh

# 系统清理重置
python clean.py && ./start_all.sh

```

## � API 使用示例

### 🔄 分配器接口

```bash
# 提交检测任务
curl -X POST http://localhost:5000/taskdoor \
  -F "file=@test.jpg"
# 返回: {"taskid": "abc123", "status": 200}

# 查询任务结果
curl -X POST http://localhost:5000/check \
  -d "taskid=abc123"
# 返回: {"status": 1, "data": {...}}

# 检查服务状态
curl http://localhost:5000/simplecheck
# 返回: {"0.0.0.0:35331": "0", "0.0.0.0:35332": "1"}
```

### 📊 返回码说明

| 状态码 | 含义 | 处理建议 |
|--------|------|----------|
| **200** | ✅ 成功 | 正常处理 |
| **400** | ❌ 参数错误 | 检查请求格式 |
| **413** | ⚠️ 文件过大 | 压缩文件到50MB内 |
| **415** | ❌ 格式不支持 | 转换为支持格式 |
| **500** | 🔧 服务器错误 | 查看服务日志 |

---

## 📞 技术支持

- 🌟 **GitHub**: [项目主页](https://github.com/cucdfd/cucdfd)
- 🐛 **问题反馈**: [Issues](https://github.com/cucdfd/cucdfd/issues)
- 📧 **邮件**: support@cucdfd.org
- � **文档**: [在线文档](https://docs.cucdfd.org)

---

**📋 文档信息**
- **版本**: v2.0
- **更新日期**: 2025年9月
- **系统名称**: CUCDFD (白杨智鉴) 深度鉴伪系统
- **维护团队**: CUCDFD 开发团队

**🔗 相关文档**
- [🔄 分配器服务文档](distributer/README.md) - 任务分发调度服务
- [🤖 检测服务文档](cucdfd_service/README.md) - AI检测推理服务
- [🌐 Web前端文档](web/README.md) - 用户界面服务
- [🐍 Python环境配置](appendices/python-env/README.md) - AI环境安装配置
- [🔤 字体配置指南](appendices/fonts/README.md) - 字体安装配置
- [📄 LibreOffice配置指南](appendices/libreoffice/README.md) - PDF生成配置
- [🖥️ Screen会话管理](appendices/screen/README.md) - 后台服务管理
