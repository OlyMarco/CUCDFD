# 🖥️ CUCDFD Screen 配置

> 多会话管理与后台服务运行系统

## 🚀 核心特性

- **🔄 会话管理** - 多服务后台运行管理
- **📱 会话持久化** - 断线重连不丢失服务
- **⚡ 快速部署** - 用户级安装无需root权限
- **🎯 服务隔离** - 独立会话避免相互干扰
- **📊 状态监控** - 实时查看各服务运行状态
- **🔧 离线安装** - 预编译包直接部署

## 📋 环境要求

- Linux 系统
- 用户级安装权限
- 磁盘空间: ~5MB

## ⚡ 快速安装

```bash
# 创建用户bin目录
mkdir -p ~/bin

# 配置screen权限
chmod +x ./Screen/bin/screen

# 创建软链接
ln -sf "$(pwd)/Screen/bin/screen" ~/bin/screen

# 添加到PATH
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
source ~/.bashrc
```

## ✅ 验证安装

```bash
# 检查版本
screen --version

# 测试会话创建
screen -dmS test_session
screen -ls | grep test_session && echo "✅ Screen安装成功"

# 清理测试会话
screen -S test_session -X quit
```

## 🎯 CUCDFD 系统使用

### 会话管理命令

```bash
# 查看所有会话
screen -ls

# 进入会话（查看输出）
screen -r cucdfd_svc_35331
screen -r cucdfd_distributer
screen -r cucdfd_web

# 分离会话（Ctrl+A, D）
# 或直接退出终端，会话继续运行
```

### 系统服务会话

| 会话名 | 服务 | 用途 |
|--------|------|------|
| 🤖 **cucdfd_svc_35331** | AI检测服务1 | 深度学习推理 |
| 🤖 **cucdfd_svc_35332** | AI检测服务2 | 深度学习推理 |
| 🔄 **cucdfd_distributer** | 任务分配器 | 负载均衡调度 |
| 🌐 **cucdfd_web** | Web前端 | 用户界面服务 |

## 🔧 常用操作

```bash
# 创建新会话
screen -dmS session_name command

# 进入已有会话
screen -r session_name

# 强制进入会话（多人共享）
screen -x session_name

# 终止会话
screen -S session_name -X quit

# 发送命令到会话
screen -S session_name -X stuff "command\n"
```

## 🛠️ 故障排除

### 常见问题

1. **screen命令未找到**
   ```bash
   # 检查安装路径
   ls -la ~/bin/screen
   
   # 重新配置PATH
   export PATH=$PATH:$HOME/bin
   ```

2. **权限问题**
   ```bash
   chmod +x ~/bin/screen
   ```

3. **会话无法创建**
   ```bash
   # 检查/tmp权限
   ls -ld /tmp
   
   # 清理僵尸会话
   screen -wipe
   ```

## 📊 会话监控

```bash
# 实时监控所有CUCDFD会话
watch -n 2 'screen -ls | grep cucdfd'

# 检查会话CPU使用
ps aux | grep screen | grep cucdfd

# 查看会话日志（如果有重定向）
tail -f ~/cucdfd/logs/*.log
```

## 📋 安装检查清单

- ✅ Screen二进制文件已复制
- ✅ 执行权限已设置
- ✅ 软链接已创建
- ✅ PATH变量已配置
- ✅ 版本检查通过
- ✅ 测试会话正常

---

**💡 提示**: Screen是CUCDFD系统后台服务的核心管理工具，确保正确安装和配置