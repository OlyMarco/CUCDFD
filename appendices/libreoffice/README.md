# 📄 CUCDFD LibreOffice 配置

> PDF报告生成核心组件

## 🚀 核心特性

- **📄 PDF转换** - 高质量文档转PDF引擎
- **🖥️ 无头模式** - 服务器环境优化运行
- **🔧 离线安装** - 支持无网络环境部署
- **⚡ 快速启动** - 自动化安装配置
- **🌐 中文支持** - 完整的中文字体渲染
- **💾 轻量化** - 最小化安装包配置

## 📋 环境要求

- Ubuntu 22.04+ / Linux 系统
- 磁盘空间: ≥ 2GB
- 内存: ≥ 512MB RAM

## ⚡ 快速安装

### 🌐 方法一：在线安装

```bash
# 更新包列表
sudo apt update

# 安装LibreOffice核心包
sudo apt install -y libreoffice-core libreoffice-writer

# 验证安装
libreoffice --version
```

### 💿 方法二：离线安装

```bash
# 解压安装包
tar -zxvf LibreOffice_25.8.1_Linux_x86-64_deb.tar.gz
cd LibreOffice_25.8.1.1_Linux_x86-64_deb/DEBS/

# 创建提取目录
mkdir -p ./libreoffice_extract

# 批量提取deb包
for deb in *.deb; do
    ar x "$deb"
    tar -xf data.tar.xz -C ./libreoffice_extract/ 2>/dev/null || true
    rm -f control.tar.gz data.tar.xz debian-binary
done

# 配置环境
mkdir -p ~/bin
ln -sf "$(pwd)/libreoffice_extract/opt/libreoffice25.8/program/soffice" ~/bin/libreoffice
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
source ~/.bashrc
```

## ✅ 验证安装

```bash
# 检查版本
libreoffice --version

# 测试PDF转换
echo "CUCDFD 测试文档" > test.txt
libreoffice --headless --convert-to pdf test.txt

# 验证结果
if [ -f "test.pdf" ]; then
    echo "✅ PDF转换成功"
    rm -f test.txt test.pdf
else
    echo "❌ 转换失败"
fi
```

## 📊 功能对比

| 安装方式 | 优势 | 适用场景 |
|----------|------|----------|
| 🌐 **在线安装** | 简单快速 | 有网络环境 |
| 💿 **离线安装** | 版本可控 | 内网部署 |

## 🔧 故障排除

### 常见问题

1. **缺少依赖**
   ```bash
   sudo apt install -y libfontconfig1 libxrender1
   ```

2. **权限问题**
   ```bash
   chmod +x ~/bin/libreoffice
   ```

3. **字体缺失**
   - ✅ 确保已安装中文字体包
   - ✅ 检查字体缓存更新

## 📋 配置检查清单

- ✅ LibreOffice已安装
- ✅ 版本信息正确
- ✅ PDF转换正常
- ✅ 中文字体支持
- ✅ 无头模式可用
- ✅ 环境变量配置

---

**💡 提示**: LibreOffice是PDF报告生成的核心组件，确保正确安装和配置

