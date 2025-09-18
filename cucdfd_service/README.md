# 🤖 CUCDFD 检测服务

> 多模态AI深度伪造检测引擎

## 🚀 核心特性

- **🎬 视频检测** - DeepFake视频内容智能识别
- **🎵 音频检测** - 语音合成和音频篡改检测
- **🖼️ 图像检测** - AI生成图像和篡改识别
- **🔄 多模态分析** - 视频音轨一致性检测
- **📊 实时监控** - 异步处理进度实时跟踪
- **🚀 GPU加速** - 多GPU并行推理优化

## 🏗️ 系统架构

```
客户端请求 → Flask API → AI模型推理 → 结果生成
     ↓           ↓          ↓           ↓
   文件上传    任务队列    GPU推理    PDF报告
   状态查询    进度跟踪    结果融合    可视化图表
```

## 📁 支持格式

| 类型 | 格式 | 推荐规格 |
|------|------|----------|
| 🎬 **视频** | mp4 | 1080p, ≤30fps |
| 🖼️ **图像** | png, jpg, jpeg, tif, webp | ≤4K分辨率 |
| 🎵 **音频** | wav, m4a, pcm, mp3, flac | 44.1kHz, 16bit |

## 📋 环境要求

| 组件 | 最低配置 | 推荐配置 |
|------|----------|----------|
| 🖥️ **GPU** | NVIDIA ≥8GB | NVIDIA ≥16GB |
| 💾 **内存** | 8GB RAM | 16GB+ RAM |
| 💿 **存储** | 10GB | 20GB+ |

## ⚡ 快速启动

### 🌐 环境配置

```bash
# 激活Conda环境
conda activate cucdfd

# 验证依赖
python -c "import torch; print(f'PyTorch: {torch.__version__}, CUDA: {torch.cuda.is_available()}')"
python -c "import flask; print(f'Flask: {flask.__version__}')"

# 系统工具
sudo apt install -y screen ffmpeg libreoffice
```

### 🚀 启动服务

```bash
# 单个服务启动
python service_api.py -p 35331

# 批量启动（使用配置文件）
./start_service.sh

# 检查服务状态
curl http://localhost:35331/statuscheck
```

## 🔄 工作流程

1. **📤 任务提交** - `/taskget` 接收文件上传
2. **🔍 AI推理** - 多模型并行检测分析  
3. **📊 结果生成** - 可视化图表和PDF报告
4. **📥 结果查询** - `/resultcheck` 返回检测结果


## 🌐 API接口

### 核心接口

| 接口 | 方法 | 功能 | 参数 |
|------|------|------|------|
| 📤 `/taskget` | POST | 提交检测任务 | file, taskid |
| 📥 `/resultcheck` | POST | 查询任务结果 | taskid |
| ✅ `/statuscheck` | GET | 检查服务状态 | 无 |

### 使用示例

```bash
# 提交任务
curl -X POST http://localhost:35331/taskget \
  -F "file=@video.mp4" \
  -F "taskid=test_001"

# 查询结果
curl -X POST http://localhost:35331/resultcheck \
  -d "taskid=test_001"

# 检查状态
curl http://localhost:35331/statuscheck
```

## 🌐 响应格式

### 任务状态

| 状态值 | 含义 | 阶段 |
|--------|------|------|
| 📋 **0.0** | 已初始化 | 任务创建 |
| 🔄 **0.1-0.5** | 处理中 | AI推理分析 |
| 📈 **0.8-0.99** | 生成报告 | 结果整理 |
| ✅ **1.0** | 已完成 | 结果可用 |
| ❌ **<0** | 处理失败 | 错误状态 |

### 检测结果

```json
{
  "status": 1.0,
  "message": "已完成",
  "data": {
    "result_all": "伪造",
    "scores_all": "0.8556",
    "mvss_prediction": "该图像/视频存在伪造痕迹",
    "osn_prediction": "该图像/视频存在深度伪造",
    "mabc_prediction": "视频第0段为伪造，伪造时间段为1.23~4.56秒，置信度为0.888。",
    "scores_all_img": "0.8556",
    "scores_all_audio": "0.7234",
    "mvss_pic": "/static/video_result/xxx/mvss_plot.png",
    "osn_pic": "/static/video_result/xxx/osn_plot.png"
  }
}
```

## 🛠️ 服务管理

### 管理脚本

| 脚本 | 功能 | 用法 |
|------|------|------|
| 🚀 `start_service.sh` | 启动所有服务 | `./start_service.sh` |
| 🛑 `stop_service.sh` | 停止所有服务 | `./stop_service.sh` |

### 监控命令

```bash
# 查看服务状态
screen -ls | grep cucdfd_svc

# 进入服务会话
screen -r cucdfd_svc_35331

# 查看服务日志
tail -f logs/service_35331.log

# GPU使用监控
nvidia-smi
```

## 🔧 故障排除

### 常见问题

1. **GPU内存不足**
   ```bash
   # 检查GPU状态
   nvidia-smi
   
   # 清理GPU缓存
   python -c "import torch; torch.cuda.empty_cache()"
   ```

2. **服务启动失败**
   ```bash
   # 检查端口占用
   netstat -tlnp | grep 35331
   
   # 检查依赖包
   python -c "import torch, flask, mmcv"
   ```

3. **模型加载错误**
   ```bash
   # 检查模型文件
   ls -la weights/
   
   # 验证模型路径配置
   python -c "import os; print(os.path.exists('weights/model.pth'))"
   ```

---

**💡 提示**: 检测服务是CUCDFD系统的AI推理核心，确保GPU环境正确配置
  "result_pdf": "/static/video_result/20250120123456789/Forensic_Report.pdf"
}
```

### 图像检测结果

```json
{
  "img_show_path": "/static/pic_result/20250120123456789/test.jpg",
  "mvss_prediction": "该图像存在拼接痕迹",
  "osn_prediction": "该图像为真实图像",
  "result_all": "伪造",
  "scores_all": "0.7500",
  "scores_all_img": "0.7500",
  "mvss_pic": "/static/pic_result/20250120123456789/mvss/mvss_plot.png",
  "osn_pic": "/static/pic_result/20250120123456789/osn/osn_plot.png"
}
```

### 音频检测结果

```json
{
  "audio_prediction": "音频第0段为真实语音。音频第1段为伪造语音。",
  "result_all": "伪造",
  "scores_all": "0.6542",
  "scores_all_audio": "0.6542"
}
```

### 字段说明

| 字段名 | 类型 | 描述 |
|--------|------|------|
| `result_all` | string | 最终检测结论："真实" 或 "伪造" |
| `scores_all` | string | 综合置信度分数（0-1之间） |
| `scores_all_img` | string | 图像分析置信度分数 |
| `scores_all_audio` | string | 音频分析置信度分数 |
| `mvss_prediction` | string | MVSS模型检测结果描述 |
| `osn_prediction` | string | OSN模型检测结果描述 |
| `mabc_prediction` | string | MABC模型时序分析结果 |
| `mvss_pic` | string | MVSS可视化图片路径 |
| `osn_pic` | string | OSN可视化图片路径 |
| `result_pdf` | string | 详细检测报告PDF路径 |
