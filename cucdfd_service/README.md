# CUCDFD 检测服务 API 文档

## 📋 目录

- [服务概述](#service-overview)
- [系统架构](#architecture)
- [环境配置](#environment-setup)
- [服务部署](#service-deployment)
- [API接口文档](#api-reference)
- [使用示例](#usage-examples)
- [结果数据结构](#result-structure)
- [故障排除](#troubleshooting)

## <a id="service-overview"></a>📊 服务概述

CUCDFD检测服务是系统的核心AI推理组件，基于Flask框架构建，提供多模态深度伪造检测能力。服务采用异步处理架构，支持多GPU并行推理，能够高效处理视频、音频和图像的真伪检测任务。

### 🎯 核心功能

| 功能模块 | 描述 | 技术实现 |
|----------|------|----------|
| **视频检测** | DeepFake视频内容检测 | MMAction2 + 时序分析 |
| **音频检测** | 语音合成和音频篡改检测 | MABC-Net + SSDGSpeech5 |
| **图像检测** | AI生成图像和篡改检测 | OSN + MVSS模型 |
| **多模态分析** | 视频音轨一致性分析 | MABC网络 |

### 🔍 支持的检测类型

- **视频伪造检测**：DeepFake、FaceSwap、表情操控
- **音频伪造检测**：语音合成、声纹伪造
- **图像伪造检测**：AI生成、拼接篡改
- **多模态融合**：视频音轨一致性分析

### 📁 支持的文件格式

| 媒体类型 | 支持格式 | 最大文件大小 | 推荐规格 |
|---------|---------|-------------|----------|
| **视频** | mp4 | 无限制（受磁盘空间限制） | 1080p, ≤30fps |
| **图像** | png, jpg, jpeg, tif, webp | 无限制（受磁盘空间限制） | ≤4K分辨率 |
| **音频** | wav, m4a, pcm, mp3, flac | 无限制（受磁盘空间限制） | 44.1kHz, 16bit |

## <a id="architecture"></a>🏗️ 系统架构

### 服务架构图

```
分配器服务 (5000) ──→ 检测服务实例 (35331+)
       │                     │
   任务调度分发          AI模型推理处理
   负载均衡管理          结果生成返回
       │                     │
       └─────── GPU资源池 ────┘
              (CUDA推理)
```

### 核心组件

| 组件 | 文件 | 功能 |
|------|------|------|
| **API服务器** | `service_api.py` | Flask Web API接口 |
| **推理引擎** | `service.py` | 核心检测逻辑和模型管理 |
| **视频检测** | `mmaction2/` | 视频内容分析模块 |
| **音频检测** | `MABC_Net/`, `SSDGSpeech5/` | 音频特征提取和分类 |
| **图像检测** | `word_proc_master/` | 图像分析和报告生成 |

### 工作流程

1. **任务接收**：通过`/taskget`接口接收检测任务
2. **文件预处理**：格式验证、转换和特征提取
3. **模型推理**：根据文件类型调用相应AI模型
4. **结果融合**：多模型结果融合和置信度计算
5. **报告生成**：生成可视化图表和PDF报告
6. **结果返回**：通过`/resultcheck`接口返回结果

## <a id="environment-setup"></a>💻 环境配置

### 硬件要求

| 组件 | 最低配置 | 推荐配置 |
|------|----------|----------|
| **GPU** | NVIDIA GPU (≥8GB显存) | NVIDIA GPU (≥16GB显存) |
| **内存** | 8GB RAM | 16GB+ RAM |
| **存储** | 10GB 可用空间 | 20GB+ 可用空间 |

### 软件依赖

#### 系统环境
```bash
# 必需的系统工具
sudo apt update
sudo apt install -y screen ffmpeg htop git libreoffice

# 验证安装
screen --version
ffmpeg -version
libreoffice --version
```

#### Python环境
```bash
# 激活Conda环境
conda activate cucdfd

# 验证核心依赖
python -c "import torch; print(f'PyTorch: {torch.__version__}, CUDA: {torch.cuda.is_available()}')"
python -c "import flask; print(f'Flask: {flask.__version__}')"
```

#### 模型依赖检查
```bash
# 检查模型文件完整性
ls -la mmaction2/checkpoints/
ls -la MABC_Net/ckpt/
ls -la word_proc_master/MVSSNet/ckpt/
```

## <a id="service-deployment"></a>🚀 服务部署

### 配置说明

检测服务使用服务目录内的`config.yml`文件进行配置：

```yaml
# 服务配置
services:
  - name: "service_1"             # 服务名称/标识符
    port: 35331                   # 监听端口
    gpu_id: 0                     # 使用的GPU设备ID
  - name: "service_2"
    port: 35332
    gpu_id: 0                     # 可以指定同一个GPU
```

**配置参数说明**:
- `name`: 服务的描述性名称，用于标识和日志记录
- `port`: 服务监听端口，确保端口未被占用
- `gpu_id`: 指定使用的GPU设备ID (0, 1, 2, ...)，多个服务可以共享同一个GPU

### 启动服务

#### 1. 使用启动脚本（推荐）
```bash
# 进入服务目录
cd /home/cucdfd/桌面/cucdfd/cucdfd_service

# 使用脚本启动所有配置的服务
./start_service.sh

# 查看启动状态
screen -ls | grep cucdfd_svc
```

#### 2. 手动单实例启动
```bash
# 进入服务目录
cd /home/cucdfd/桌面/cucdfd/cucdfd_service

# 激活环境
conda activate cucdfd

# 单个服务实例启动
python service_api.py --port 35331

# 后台启动（使用screen）
screen -dmS cucdfd_svc_35331 python service_api.py --port 35331

# 查看服务状态
screen -r cucdfd_svc_35331

# 从screen会话退出（不终止服务）
# 按 Ctrl+A，然后按 D
```

#### 3. 手动多实例启动
```bash
# 从服务目录启动多个GPU实例
cd /home/cucdfd/桌面/cucdfd/cucdfd_service
conda activate cucdfd

# 启动第一个实例（GPU 0）
export CUDA_VISIBLE_DEVICES=0
screen -dmS cucdfd_svc_35331 python service_api.py --port 35331

# 启动第二个实例（GPU 1）
export CUDA_VISIBLE_DEVICES=1
screen -dmS cucdfd_svc_35332 python service_api.py --port 35332

# 检查所有实例
screen -ls | grep cucdfd_svc
ss -tuln | grep -E "(35331|35332)"
```

#### 4. 批量管理脚本示例
```bash
# 创建批量启动脚本
cat > start_custom_services.sh << 'EOF'
#!/bin/bash
cd /home/cucdfd/桌面/cucdfd/cucdfd_service
conda activate cucdfd

# 读取配置并启动服务
python -c "
import yaml
import os

with open('config.yml', 'r') as f:
    config = yaml.safe_load(f)

for service in config['services']:
    name = service.get('name', f'service_{service[\"port\"]}')
    port = service['port']
    gpu_id = service['gpu_id']
    
    cmd = f'screen -dmS cucdfd_svc_{port} bash -c \"export CUDA_VISIBLE_DEVICES={gpu_id}; python service_api.py --port {port}\"'
    print(f'启动服务: {name} (端口: {port}, GPU: {gpu_id})')
    os.system(cmd)
"
EOF

chmod +x start_custom_services.sh
./start_custom_services.sh
```

#### 5. 系统集成启动（可选）

如果需要与整个系统一起启动，可以通过项目根目录的系统脚本：

```bash
# 从项目根目录启动整个系统
cd /home/cucdfd/桌面/cucdfd
./start_all.sh
```

### 停止服务

#### 1. 使用停止脚本（推荐）
```bash
# 进入服务目录
cd /home/cucdfd/桌面/cucdfd/cucdfd_service

# 停止所有配置的服务
./stop_services.sh
```

#### 2. 手动停止服务
```bash
# 停止特定实例
screen -S cucdfd_svc_35331 -X quit

# 停止所有检测服务实例
screen -ls | grep cucdfd_svc | awk '{print $1}' | xargs -I {} screen -S {} -X quit

# 强制终止（如果需要）
pkill -f "service_api.py"
```

### 服务管理脚本

项目提供了完整的服务管理脚本：

| 脚本 | 功能 | 使用方法 |
|------|------|----------|
| `start_service.sh` | 启动所有配置的服务 | `./start_service.sh` |
| `stop_services.sh` | 停止所有服务实例 | `./stop_services.sh` |

**脚本特性**:
- 自动读取`config.yml`配置
- 智能的GPU环境变量设置
- 详细的启动/停止日志
- 错误检测和状态验证
- Screen会话管理

### 服务状态检查

```bash
# 检查端口监听
ss -tuln | grep -E "(35331|35332)"

# 检查screen会话
screen -ls | grep cucdfd_svc

# 检查进程状态
ps aux | grep service_api.py

# 检查GPU使用情况
nvidia-smi

# 测试服务响应
curl http://localhost:35331/statuscheck
```

### 启动方式对比

| 启动方式 | 工作目录 | 配置文件 | 适用场景 |
|---------|---------|---------|----------|
| **服务脚本启动** | `cucdfd_service/` | `config.yml` | 独立开发，测试部署 |
| **手动启动** | `cucdfd_service/` | 手动配置 | 调试，单实例测试 |
| **系统集成启动** | 项目根目录 | 全局配置 | 生产环境，完整系统 |

**推荐使用方式**：
1. **开发测试**：使用`./start_service.sh`脚本启动
2. **调试模式**：手动启动单个实例
3. **生产部署**：系统集成启动

### 配置文件管理

```bash
# 查看当前配置
cat config.yml

# 备份配置
cp config.yml config.yml.backup

# 验证配置语法
python -c "import yaml; yaml.safe_load(open('config.yml'))"

# 动态修改端口配置
sed -i 's/port: 35331/port: 35341/' config.yml
```

## <a id="api-reference"></a>🔌 API接口文档

### 1. 提交检测任务

**接口**: `POST /taskget`

**功能**: 接收文件上传，创建异步检测任务

**请求格式**:
```http
POST /taskget
Content-Type: multipart/form-data

参数:
- file: [必需] 要检测的多媒体文件
- taskid: [必需] 用户定义的唯一任务ID
```

**响应格式**:
```json
{
  "status": 200
}
```

**错误响应**:
```json
{
  "error": "No file provided"
}
```

**状态码**:
- `200`: 任务提交成功
- `400`: 缺少文件或参数错误
- `415`: 不支持的文件格式

### 2. 查询任务结果

**接口**: `POST /resultcheck`

**功能**: 查询指定任务的处理状态和结果

**请求格式**:
```http
POST /resultcheck
Content-Type: application/x-www-form-urlencoded

参数:
- taskid: [必需] 要查询的任务ID
```

**响应格式（初始化）**:
```json
{
  "status": 0.0,
  "message": "已初始化"
}
```

**响应格式（处理中）**:
```json
{
  "status": 0.3,
  "message": "进行中"
}
```

**响应格式（已完成）**:
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
    "scores_all_audio": "0.7234"
  }
}
```

**响应格式（处理失败）**:
```json
{
  "status": -1.0,
  "message": "错误",
  "error_details": "GPU内存不足或模型加载失败"
}
```

**进度状态说明**:
- `0.0`: 已初始化
- `0.05`: 文件保存完成
- `0.1 - 0.2`: 文件预处理和帧提取
- `0.3 - 0.5`: 图像/视频分析
- `0.7`: 多模态分析完成
- `0.8 - 0.99`: 结果生成和报告制作
- `1.0`: 处理完成
- `< 0`: 处理失败（不同负值表示失败阶段）

### 3. 检查服务状态

**接口**: `GET /statuscheck`

**功能**: 检查服务是否正常运行

**响应格式**:
```json
{
  "status": "free"
}
```

**状态值说明**:
- `"free"`: 服务正常运行，可接收任务

## <a id="usage-examples"></a>📚 使用示例

### Python示例代码

```python
import requests
import time
import random
import string
import json

class CUCDFDClient:
    """CUCDFD检测服务客户端"""
    
    def __init__(self, base_url="http://127.0.0.1:35331"):
        self.base_url = base_url
        self.session = requests.Session()
        self.session.timeout = 30
    
    def generate_task_id(self, length=8):
        """生成随机任务ID"""
        return "".join(random.choice(string.ascii_letters + string.digits) 
                      for _ in range(length))
    
    def check_service_status(self):
        """检查服务状态"""
        try:
            response = self.session.get(f"{self.base_url}/statuscheck")
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            print(f"服务状态检查失败: {e}")
            return None
    
    def submit_task(self, file_path, task_id=None):
        """提交检测任务"""
        if task_id is None:
            task_id = self.generate_task_id()
        
        url = f"{self.base_url}/taskget"
        
        try:
            with open(file_path, "rb") as f:
                files = {"file": (file_path.split('/')[-1], f)}
                data = {"taskid": task_id}
                
                response = self.session.post(url, files=files, data=data)
                response.raise_for_status()
                
                result = response.json()
                if result.get("status") == 200:
                    print(f"✅ 任务提交成功: {task_id}")
                    return task_id
                else:
                    print(f"❌ 任务提交失败: {result}")
                    return None
                
        except FileNotFoundError:
            print(f"❌ 文件未找到: {file_path}")
            return None
        except requests.exceptions.RequestException as e:
            print(f"❌ 任务提交失败: {e}")
            return None
    
    def check_task_result(self, task_id):
        """查询任务结果"""
        url = f"{self.base_url}/resultcheck"
        data = {"taskid": task_id}
        
        try:
            response = self.session.post(url, data=data)
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            print(f"❌ 结果查询失败 (Task ID: {task_id}): {e}")
            return None
    
    def wait_for_result(self, task_id, max_wait_time=600, poll_interval=10):
        """等待任务完成并返回结果"""
        print(f"🔄 开始等待任务 {task_id} 完成...")
        
        start_time = time.time()
        while time.time() - start_time < max_wait_time:
            result = self.check_task_result(task_id)
            
            if result is None:
                print(f"   查询失败，{poll_interval}秒后重试...")
                time.sleep(poll_interval)
                continue
            
            status = result.get("status", 0)
            message = result.get("message", "")
            
            # 显示进度
            if isinstance(status, (int, float)):
                if 0 <= status < 1:
                    progress = int(status * 100)
                    print(f"   进度: {progress}% - {message}")
                elif status >= 1.0:
                    print(f"✅ 任务完成! 最终状态: {status}")
                    return result
                elif status < 0:
                    print(f"❌ 任务失败! 状态: {status}")
                    error_details = result.get("error_details", "未知错误")
                    print(f"   错误详情: {error_details}")
                    return result
            
            time.sleep(poll_interval)
        
        print(f"⏰ 等待超时 ({max_wait_time}秒)")
        return None
    
    def process_file(self, file_path, task_id=None):
        """完整的文件处理流程"""
        # 1. 检查服务状态
        status = self.check_service_status()
        if status is None:
            print("❌ 服务不可用")
            return None
        
        print(f"📊 服务状态: {status.get('status', 'unknown')}")
        
        # 2. 提交任务
        if task_id is None:
            task_id = self.submit_task(file_path)
        else:
            task_id = self.submit_task(file_path, task_id)
        
        if task_id is None:
            return None
        
        # 3. 等待结果
        result = self.wait_for_result(task_id)
        
        # 4. 处理结果
        if result and result.get("status") == 1.0:
            data = result.get("data", {})
            print(f"\n📋 检测结果摘要:")
            print(f"   结论: {data.get('result_all', 'N/A')}")
            print(f"   置信度: {data.get('scores_all', 'N/A')}")
            
            # 显示详细结果
            if 'mvss_prediction' in data:
                print(f"   MVSS检测: {data['mvss_prediction']}")
            if 'mabc_prediction' in data:
                print(f"   MABC分析: {data['mabc_prediction']}")
        
        return result

# 使用示例
if __name__ == "__main__":
    # 创建客户端
    client = CUCDFDClient("http://127.0.0.1:35331")
    
    # 测试文件路径（请替换为实际文件路径）
    test_files = [
        "/path/to/test_video.mp4",
        "/path/to/test_image.jpg", 
        "/path/to/test_audio.wav"
    ]
    
    for file_path in test_files:
        print(f"\n{'='*50}")
        print(f"🔍 开始检测文件: {file_path}")
        print(f"{'='*50}")
        
        result = client.process_file(file_path)
        
        if result:
            print(f"📄 完整结果:")
            print(json.dumps(result, indent=2, ensure_ascii=False))
        else:
            print(f"❌ 检测失败")
        
        print(f"\n{'='*50}")
        time.sleep(2)  # 避免过于频繁的请求
```

### cURL示例

```bash
# 1. 检查服务状态
curl -X GET http://localhost:35331/statuscheck

# 2. 提交检测任务
curl -X POST http://localhost:35331/taskget \
  -F "file=@/path/to/your/video.mp4" \
  -F "taskid=test_task_001"

# 3. 查询任务结果
curl -X POST http://localhost:35331/resultcheck \
  -d "taskid=test_task_001"

# 4. 批量状态检查脚本
#!/bin/bash
TASK_ID="test_task_001"
BASE_URL="http://localhost:35331"

echo "开始监控任务: $TASK_ID"
while true; do
    RESULT=$(curl -s -X POST $BASE_URL/resultcheck -d "taskid=$TASK_ID")
    STATUS=$(echo $RESULT | python3 -c "import sys, json; print(json.load(sys.stdin).get('status', 0))")
    
    if (( $(echo "$STATUS >= 1.0" | bc -l) )); then
        echo "任务完成: $RESULT"
        break
    elif (( $(echo "$STATUS < 0" | bc -l) )); then
        echo "任务失败: $RESULT"
        break
    else
        echo "进度: $(echo "scale=1; $STATUS * 100" | bc)%"
        sleep 10
    fi
done
```

## <a id="result-structure"></a>📋 结果数据结构

### 视频检测结果

```json
{
  "mvss_prediction": "该视频存在伪造痕迹",
  "osn_prediction": "该视频存在深度伪造", 
  "mabc_prediction": "视频第0段为伪造，伪造时间段为1.23~4.56秒，置信度为0.888。",
  "deep_audio_trace": "采用 SSDG 网络结构...",
  "result_all": "伪造",
  "scores_all": "0.8556",
  "scores_all_img": "0.8556",
  "scores_all_audio": "0.7234",
  "mvss_pic": "/static/video_result/20250120123456789/mvss_plot.png",
  "osn_pic": "/static/video_result/20250120123456789/osn_plot.png",
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

## <a id="troubleshooting"></a>🔧 故障排除

### 常见问题解决方案

#### 1. 服务启动失败

**问题**: 端口被占用
```bash
# 查找占用进程
lsof -i :35331
ss -tulpn | grep :35331

# 终止占用进程
kill -9 <PID>

# 或修改配置使用其他端口
```

**问题**: GPU设备不可用
```bash
# 检查GPU状态
nvidia-smi

# 检查CUDA环境
python -c "import torch; print(torch.cuda.is_available())"

# 检查环境变量
echo $CUDA_VISIBLE_DEVICES
```

**问题**: 模型文件缺失
```bash
# 检查模型文件
find . -name "*.pth" -o -name "*.pt" -o -name "*.ckpt"

# 检查关键模型路径
ls -la word_proc_master/MVSSNet/ckpt/mvssnet_*.pth
ls -la MABC_Net/ckpt/last.ckpt
```

#### 2. 推理过程错误

**问题**: GPU内存不足
```bash
# 解决方案1: 减小批处理大小
export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:128

# 解决方案2: 清理GPU缓存
python -c "import torch; torch.cuda.empty_cache()"

# 解决方案3: 重启服务释放内存
screen -S cucdfd_svc_35331 -X quit
sleep 5
screen -dmS cucdfd_svc_35331 python service_api.py --port 35331
```

**问题**: LibreOffice PDF转换失败
```bash
# 检查LibreOffice安装
libreoffice --version

# 手动测试PDF转换
libreoffice --headless --convert-to pdf test.docx

# 安装缺失的字体
sudo apt install fonts-liberation fonts-dejavu
```

#### 3. API响应异常

**问题**: 任务状态查询失败
```bash
# 检查任务结果目录
ls -la static/*/
find static/ -name "data.json"

# 检查任务进度文件
cat task_progress.json
```

**问题**: 文件上传失败
```bash
# 检查文件格式支持
python -c "
from utils import allowed_file
print(allowed_file('test.mp4'))
print(allowed_file('test.jpg'))
print(allowed_file('test.wav'))
"

# 检查磁盘空间
df -h
```

### 系统重置和恢复

#### 服务重置流程
```bash
#!/bin/bash
# reset_service.sh
echo "开始重置检测服务..."

# 1. 停止所有服务实例
screen -ls | grep cucdfd_svc | awk '{print $1}' | xargs -I {} screen -S {} -X quit

# 2. 清理GPU内存
python -c "import torch; torch.cuda.empty_cache()" 2>/dev/null || true

# 3. 清理临时文件和任务状态
rm -rf static/*/temp/
rm -f task_progress.json

# 4. 重新启动服务
cd cucdfd_service
conda activate cucdfd
screen -dmS cucdfd_svc_35331 python service_api.py --port 35331

echo "服务重置完成"
```

#### 数据恢复
```bash
# 备份重要结果
tar -czf results_backup_$(date +%Y%m%d).tar.gz static/*/data.json task_progress.json

# 恢复结果数据
tar -xzf results_backup_20250120.tar.gz
```

### 日志系统

```bash
# 查看任务处理日志
grep "processing" logs/service_*.log

# 查看错误日志
grep -i "error\|exception" logs/service_*.log

# 查看GPU内存错误
grep -i "cuda\|memory" logs/service_*.log

# 实时监控日志
tail -f logs/service_35331.log
```

---

**📋 文档信息**
- **版本**: v1.0
- **更新日期**: 2025年5月
- **模块**: 检测服务API
- **维护**: CUCDFD团队

**🔗 相关文档**
- [项目主文档](../README.md)
- [分配器服务文档](../distributer/README.md)  
- [Web前端文档](../web/README.md)
- [环境配置指南](../appendices/environment/env%20setup.md)