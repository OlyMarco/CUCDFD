# 🔄 深度鉴伪分配器

> 智能任务调度与负载均衡系统

## 🚀 核心特性

- **🎯 智能分发** - 自动将任务分配给最优节点
- **📊 实时监控** - 节点状态实时监控与管理
- **🔒 权限管理** - 基于API密钥的安全认证
- **🌐 Web管理** - 可视化节点管理界面
- **📁 文件处理** - 支持文件上传和URL处理
- **💾 数据持久化** - SQLite数据库存储

## 🏗️ 系统架构

```
Web前端 → 分配器 → 工作节点群
   ↓        ↓         ↓
 用户交互   任务调度   AI推理
 文件上传   负载均衡   结果生成
```

## ⚡ 快速启动

### 📋 环境要求
- Python 3.8+
- Flask, SQLite3
- 至少一个工作节点

### 🔧 配置文件 (`config.yml`)
```yaml
# 基础配置
distributer_port: 5000
api_key: "your_admin_key"
max_file_size: 10485760  # 10MB

# 工作节点列表
worker_servers:
  - "192.168.1.100:8080"
  - "192.168.1.101:8080"

# 日志设置
verbose_logging: false
```

### 🚀 启动服务
```bash
# 启动
./start_distributer.sh

# 停止
./stop_distributer.sh

# 清理数据
./clean.py
```

## 📡 API接口

### 🔵 核心接口

| 接口 | 方法 | 功能 |
|------|------|------|
| `/taskdoor` | POST | 📤 提交检测任务 |
| `/check` | POST | 🔍 查询任务结果 |
| `/simplecheck` | GET | 📊 获取节点状态 |
| `/stats` | GET/POST | 📈 访问统计 |

### 🔧 管理接口 (需认证)

| 接口 | 方法 | 功能 |
|------|------|------|
| `/admin` | GET | 🎛️ 管理页面 |
| `/nodes` | GET/POST | 📋 节点管理 |
| `/nodes/<addr>` | PUT/DELETE | ✏️ 修改节点 |
| `/nodes/status` | GET | 📊 节点状态 |

## 🎛️ Web管理界面

访问 `http://your-server:5000/admin` 进入管理界面

### 功能特性
- 🍪 **Cookie登录** - 7天免登录
- 📊 **实时监控** - 5秒自动刷新
- 🔄 **节点管理** - 增删改查节点
- 📈 **状态概览** - 任务数统计

## 📁 项目结构

```
distributer/
├── 📄 distributer.py          # 主应用
├── 🗄️ distributer.db          # 数据库
├── ⚙️ config.yml              # 配置文件
├── 🚀 start_distributer.sh    # 启动脚本
├── 🛑 stop_distributer.sh     # 停止脚本
├── 🧹 clean.py                # 清理脚本
├── 🔄 db_to_json.py           # 数据导出
├── 📂 utils/                  # 工具模块
│   ├── 🗄️ database.py         # 数据库管理
│   ├── 🌐 node_manager.py     # 节点管理
│   ├── 📁 media_handler.py    # 媒体处理
│   ├── 📊 result_processor.py # 结果处理
│   └── ⚙️ config_manager.py   # 配置管理
├── 📂 templates/              # Web模板
└── 📂 static/                 # 静态文件
```

## 🔧 工具脚本

### 🧹 数据清理
```bash
./clean.py  # 清理数据库和临时文件
```

### 📤 数据导出
```bash
./db_to_json.py  # 导出数据库为JSON格式
```

## 📊 监控指标

- ✅ **节点健康** - 实时状态检查
- 📈 **任务统计** - 完成/失败统计
- 👥 **访问统计** - 用户访问记录
- ⏱️ **响应时间** - 性能监控

## 🔒 安全特性

- 🔑 **API认证** - 管理接口需要密钥
- 🍪 **Session管理** - Cookie自动登录
- 🛡️ **CORS支持** - 跨域请求控制
- 📝 **访问日志** - 详细操作记录

## 🐛 故障排除

### 常见问题

1. **节点显示offline** 
   - ✅ 检查节点服务是否启动
   - ✅ 验证网络连接

2. **任务提交失败**
   - ✅ 确认文件大小限制
   - ✅ 检查可用节点

3. **管理界面无法访问**
   - ✅ 验证API密钥配置
   - ✅ 检查端口是否开放

## 📚 更多信息

- 📖 配置详情：查看 `config.yml` 注释
- 🔗 API文档：访问主页面查看接口说明
- 📝 日志查看：检查控制台输出或日志文件

---

**💡 提示**: 首次运行会自动迁移现有数据到数据库格式
