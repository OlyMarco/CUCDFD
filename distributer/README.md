# 分配器服务 (Distributer Service)

## 📋 概述

分配器服务是CUCDFD深度鉴伪系统的核心调度组件，作为Flask Web服务运行。它负责接收来自Web前端的检测任务请求，将任务分发给可用的检测服务实例，并管理任务状态和结果传递。

### 🎯 核心功能

- **任务分发**：根据检测服务的可用性，将新任务随机分配给空闲的服务器
- **状态管理**：定期检查检测服务的状态（空闲/繁忙）
- **任务跟踪**：为每个任务生成唯一的ID，并记录任务分配给了哪个服务器以及任务的当前状态
- **结果获取**：当客户端查询任务结果时，如果任务已完成，则从相应的检测服务获取结果数据（包括文件），并将其存储在本地，然后返回给客户端
- **配置灵活**：通过配置文件管理分配器端口和检测服务列表

### 🏗️ 架构位置

```
Web前端 (8080) ──→ 分配器服务 (5000) ──→ 检测服务群 (35331+)
    │                    │                      │
  用户交互              任务调度              AI模型推理
  文件上传              负载均衡              结果生成
```

## ⚙️ 配置说明

### 主配置文件

分配器支持两种配置方式：

#### 1. 使用项目主配置文件（推荐）

通过项目根目录的 `config.yml` 文件进行配置：

```yaml
# 分配器模块配置
distributer_module:
  port: 5000                    # 分配器监听端口
  worker_servers:              # 检测服务实例列表
    - name: "service_1"        # 服务名称
      url: "http://0.0.0.0:35331"  # 服务地址
    - name: "service_2"
      url: "http://0.0.0.0:35332"
```

#### 2. 使用本地配置文件（向后兼容）

在分配器目录下的 `config.yml` 文件：

```yaml
distributer_port: 5000
# 分配器的端口号，默认是5000

worker_servers:
  - "0.0.0.0:35331"  # 例如 "192.168.1.101:35331"
  - "0.0.0.0:35332"  # 例如 "192.168.1.102:35332"
#  - "another.worker.ip:port" # 可以根据需要添加更多服务器
```

### 配置参数说明

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `port` | int | 5000 | 分配器服务监听端口 |
| `worker_servers` | list | - | 检测服务实例地址列表 |

## 🚀 启动和停止

### 使用系统脚本启动（推荐）

分配器服务通过项目根目录的启动脚本管理：

```bash
# 启动整个系统（包括分配器）
./start_all.sh

# 停止整个系统
./stop_all.sh
```

### 单独管理分配器服务

```bash
# 进入分配器目录
cd distributer

# 激活Conda环境
conda activate cucdfd

# 使用主配置文件启动（推荐）
python distributer.py --config ../config.yml

# 使用本地配置文件启动
python distributer.py

# 后台启动（使用screen）
screen -dmS cucdfd_distributer python distributer.py --config ../config.yml

# 查看后台运行状态
screen -r cucdfd_distributer

# 从screen会话退出（不终止服务）
# 按 Ctrl+A，然后按 D

# 停止后台服务
screen -S cucdfd_distributer -X quit
```

### 服务状态检查

```bash
# 检查端口监听
ss -tuln | grep :5000
netstat -tlnp | grep :5000

# 检查进程状态
ps aux | grep distributer.py

# 检查Screen会话
screen -ls | grep distributer

# 测试服务响应
curl http://localhost:5000/
```

## 🔌 API接口文档

### 1. 提交检测任务

**接口**: `POST /taskdoor`

**功能**: 接收文件上传请求，将任务分发给可用的检测服务

**请求格式**:
```http
POST /taskdoor
Content-Type: multipart/form-data

# 通过文件上传
file: [媒体文件]

# 或通过URL提交
url: [媒体文件URL]
```

**响应格式**:
```json
{
  "taskid": "abc123",
  "status": 200
}
```

**状态码**:
- `200`: 任务提交成功
- `400`: 请求参数错误
- `413`: 文件过大
- `502`: URL处理失败
- `503`: 所有检测服务不可用
- `504`: 与检测服务通信超时

### 2. 查询任务状态和结果

**接口**: `POST /check`

**功能**: 查询指定任务的处理状态和结果

**请求格式**:
```http
POST /check
Content-Type: application/x-www-form-urlencoded

taskid=abc123def456
```

**响应格式（处理中）**:
```json
{
  "status": 0.6
}
```

**响应格式（已完成）**:
```json
{
  "status": 1,
  "data": {
    "config_id": "abc123def456",
    "filename": "test_video.mp4",
    "tampered_list": ["视频伪造-视频画面内容伪造"],
    "tampered_vidfragment_str": "视频第0段为伪造，伪造时间段为1.23~4.56秒，置信度为0.88。",
    "tampered_audiofragment_str": null,
    "tampered_imagefragment_str": null
  }
}
```

**响应格式（任务失败）**:
```json
{
  "status": -1.0,
  "message": "Failed to get results from worker or error during processing."
}
```

**状态码说明**:
- `1.0`: 处理完成
- `0.0 - 0.99`: 处理中（进度值）
- `-1.0`: 处理失败
- `-1.1`: 与检测服务通信超时

### 3. 获取检测服务状态

**接口**: `GET /simplecheck`

**功能**: 获取所有检测服务的实时状态信息

**响应格式**:
```json
{
  "0.0.0.0:35331": "0",
  "0.0.0.0:35332": "1"
}
```

**状态值说明**:
- `"0"`: 检测服务空闲，可接收任务
- `"1"`: 检测服务繁忙或不可用

### 4. 系统欢迎页

**接口**: `GET /`

**功能**: 返回分配器服务的基本信息

**响应格式**:
```json
{
  "message": "欢迎使用深度鉴伪接口！",
  "documentation": "接口使用方法请详细阅读相关文档。",
  "status": 200
}
```

### 5. 统计接口

**接口**: `POST /stats` 和 `GET /stats`

**功能**: 记录访问统计和查询统计数据

**POST请求**（记录访问）:
```json
{
  "message": "Visit recorded."
}
```

**GET请求**（查询统计，需要API密钥）:
```http
GET /stats?key=authen
```

## 📊 监控和调试

### 日志系统

```bash
# 日志文件位置
logs/
├── distributer.log          # 分配器主日志
└── error.log               # 错误日志

# 实时查看日志
tail -f logs/distributer.log

# 搜索错误信息
grep -i "error\|exception" logs/distributer.log

# 查看最近的任务分发记录
grep "task" logs/distributer.log | tail -10
```

### 性能监控

```bash
# 查看分配器性能指标
curl http://localhost:5000/simplecheck

# 监控端口连接数
ss -t | grep :5000 | wc -l

# 查看内存使用
ps aux | grep distributer.py

# 查看实时网络流量
iftop -i eth0 -P
```

### 常见问题排查

#### 1. 检测服务连接失败
```bash
# 检查检测服务状态
curl http://localhost:35331/statuscheck

# 测试网络连通性
telnet localhost 35331

# 查看防火墙设置
sudo ufw status
```

#### 2. 任务分发异常
```bash
# 检查配置文件
cat config.yml

# 验证配置文件格式
python -c "import yaml; print(yaml.safe_load(open('config.yml')))"

# 重启分配器服务
screen -S cucdfd_distributer -X quit
screen -dmS cucdfd_distributer python distributer.py
```

#### 3. 文件下载失败
```bash
# 检查static目录权限
ls -la static/

# 检查磁盘空间
df -h

# 清理临时文件
find static/ -type d -mtime +7 -exec rm -rf {} \;
```

## 🔧 高级配置

### 文件大小限制

在`distributer.py`中修改最大文件大小限制：

```python
MAX_FILE_SIZE = 10 * 1024 * 1024  # 10MB
```

### 状态检查频率

修改检测服务状态检查间隔：

```python
Timer(5, update_server_status).start()  # 每5秒检查一次
```

### 集群部署

```yaml
# 多机器集群配置
distributer_module:
  worker_servers:
    # 本地服务器
    - name: "local_service_1"
      url: "http://localhost:35331"
    - name: "local_service_2"
      url: "http://localhost:35332"
    
    # 远程服务器
    - name: "remote_service_1"
      url: "http://192.168.1.101:35331"
    - name: "remote_service_2"
      url: "http://192.168.1.102:35331"
```

## 📁 文件结构

```
distributer/
├── distributer.py              # 主服务文件
├── config.yml                  # 本地配置文件（可选）
├── README.md                   # 本文档
├── utils/                      # 工具模块
│   ├── __init__.py
│   ├── statis.py              # 统计功能
│   ├── wgetdl.py              # 媒体下载器
│   └── task_utils.py          # 任务管理工具
└── static/                    # 结果文件存储（运行时创建）
    └── {taskid}/              # 按任务ID分组的结果文件
```

## 📚 相关文档

- [项目主文档](../README.md) - 完整系统说明
- [检测服务文档](../cucdfd_service/README.md) - 检测服务详细说明
- [Web前端文档](../web/README.md) - Web界面使用说明
- [环境配置指南](../appendices/environment/env%20setup.md) - 环境安装配置