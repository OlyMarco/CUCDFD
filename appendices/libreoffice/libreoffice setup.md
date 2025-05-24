# CUCDFD 深度鉴伪系统 - LibreOffice 安装配置指南

## 概述

LibreOffice是CUCDFD系统生成PDF检测报告的必需组件。本指南提供完整的LibreOffice安装和配置说明。

## 系统要求

- **操作系统**: Ubuntu 22.04+ / 其他 Linux 发行版
- **磁盘空间**: 建议 ≥ 2GB 可用空间
- **内存**: 建议 ≥ 512MB RAM用于PDF转换

## 安装方法

### 方法一：系统包管理器安装（推荐）

#### Ubuntu/Debian系统
```bash
# 更新包列表
sudo apt update

# 安装LibreOffice核心包
sudo apt install -y libreoffice-core libreoffice-writer

# 验证安装
libreoffice --version
```

#### CentOS/RHEL系统
```bash
# 安装LibreOffice
sudo yum install -y libreoffice-core libreoffice-writer

# 或者在较新版本中使用dnf
sudo dnf install -y libreoffice-core libreoffice-writer

# 验证安装
libreoffice --version
```

### 方法二：手动安装（离线环境）

如果系统无法访问软件源或需要特定版本，可以手动安装：

#### 1. 下载 LibreOffice

```bash
# 创建下载目录
mkdir -p ~/downloads/libreoffice
cd ~/downloads/libreoffice

# 下载LibreOffice安装包（使用清华大学镜像源）
wget https://mirrors.tuna.tsinghua.edu.cn/libreoffice/libreoffice/stable/24.8.0/deb/x86_64/LibreOffice_24.8.0_Linux_x86-64_deb.tar.gz
```

#### 2. 解压安装包

```bash
# 解压下载的文件
tar -zxvf LibreOffice_24.8.0_Linux_x86-64_deb.tar.gz

# 进入解压目录
cd LibreOffice_24.8.0.3_Linux_x86-64_deb/DEBS/
```

#### 3. 提取并安装deb包

```bash
# 创建提取目录
mkdir -p ~/libreoffice_extract

# 提取所有deb包内容
for deb in *.deb; do
    echo "提取 $deb ..."
    ar x "$deb"
    tar -xf control.tar.gz -C ~/libreoffice_extract/ 2>/dev/null || true
    tar -xf data.tar.xz -C ~/libreoffice_extract/ 2>/dev/null || true
    rm -f control.tar.gz data.tar.xz debian-binary
done

echo "LibreOffice文件已提取到 ~/libreoffice_extract/"
```

#### 4. 安装 Java Runtime Environment (可选)

某些LibreOffice功能需要Java支持：

```bash
# 下载OpenJDK
cd ~/downloads
wget https://mirrors.tuna.tsinghua.edu.cn/Adoptium/21/jdk/x64_linux/OpenJDK21U-jdk_x64_linux_hotspot_21.0.4_7.tar.gz

# 解压JDK
tar -zxvf OpenJDK21U-jdk_x64_linux_hotspot_21.0.4_7.tar.gz
```

#### 5. 创建软链接和配置

```bash
# 创建用户bin目录
mkdir -p ~/bin

# 创建LibreOffice软链接
ln -sf ~/libreoffice_extract/opt/libreoffice24.8/program/soffice ~/bin/libreoffice

# 创建Java软链接（如果安装了Java）
if [ -d ~/downloads/jdk-21.0.4+7 ]; then
    ln -sf ~/downloads/jdk-21.0.4+7/bin/java ~/bin/java
fi

# 配置环境变量
if ! grep -q 'export PATH=$PATH:$HOME/bin' ~/.bashrc; then
    echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
fi

# 重新加载环境变量
source ~/.bashrc
```

## 验证安装

### 1. 检查版本信息

```bash
# 检查LibreOffice版本
libreoffice --version

# 检查安装路径
which libreoffice
```

### 2. 测试PDF转换功能

```bash
# 创建测试文档
cat > test_document.txt << 'EOF'
CUCDFD 深度鉴伪系统测试文档

这是一个用于测试LibreOffice PDF转换功能的测试文档。

系统组件：
- 检测服务
- 分配器服务  
- Web前端服务

测试时间：$(date)
EOF

# 测试转换为PDF（无头模式）
libreoffice --headless --convert-to pdf test_document.txt

# 检查PDF是否生成成功
if [ -f "test_document.pdf" ]; then
    echo "✅ PDF转换测试成功"
    ls -la test_document.pdf
    rm -f test_document.txt test_document.pdf
else
    echo "❌ PDF转换测试失败"
fi
```

## 配置优化

### 1. 无头模式配置

为了确保LibreOffice在服务器环境中正常工作：

```bash
# 设置无头模式环境变量
export SAL_USE_VCLPLUGIN=svp

# 添加到启动脚本中
echo 'export SAL_USE_VCLPLUGIN=svp' >> ~/.bashrc
```

### 2. 字体配置

确保系统有足够的字体支持：

```bash
# 安装基本字体包
sudo apt install -y fonts-liberation fonts-dejavu fonts-noto

# 安装中文字体支持
sudo apt install -y fonts-wqy-microhei fonts-wqy-zenhei

# 刷新字体缓存
fc-cache -f -v
```

### 3. 内存和性能优化

```bash
# 创建LibreOffice配置目录
mkdir -p ~/.config/libreoffice/4/user

# 配置内存使用
cat > ~/.config/libreoffice/4/user/registrymodifications.xcu << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<oor:items xmlns:oor="http://openoffice.org/2001/registry">
  <item oor:path="/org.openoffice.Office.Common/Cache">
    <prop oor:name="GraphicManager" oor:type="xs:int">
      <value>10000000</value>
    </prop>
  </item>
</oor:items>
EOF
```

## 故障排除

### 常见问题及解决方案

#### 1. 权限问题

```bash
# 确保LibreOffice可执行
chmod +x ~/bin/libreoffice

# 检查目录权限
ls -la ~/libreoffice_extract/opt/libreoffice24.8/program/soffice
```

#### 2. 字体显示问题

```bash
# 检查字体缓存
fc-cache -f -v

# 列出可用字体
fc-list | grep -i "dejavu\|liberation"

# 重新安装字体
sudo apt install --reinstall fonts-dejavu fonts-liberation
```

#### 3. PDF转换失败

```bash
# 检查LibreOffice进程
ps aux | grep soffice

# 清理僵尸进程
pkill -f soffice

# 清理临时文件
rm -rf /tmp/.X*-lock
rm -rf ~/.config/libreoffice/4/user/.lock
```

#### 4. 环境变量问题

```bash
# 检查PATH
echo $PATH | grep -o "[^:]*bin[^:]*"

# 重新加载环境
source ~/.bashrc

# 手动测试路径
~/bin/libreoffice --version
```

### 日志和调试

```bash
# 启用详细日志
export SAL_LOG="+WARN+INFO"

# 测试转换并查看日志
libreoffice --headless --convert-to pdf test.txt 2>&1 | tee libreoffice.log

# 检查系统日志
journalctl -u display-manager | tail -20
```

## 维护和更新

### 定期维护

```bash
# 清理临时文件
find /tmp -name "lu*" -user $(whoami) -type d -exec rm -rf {} + 2>/dev/null

# 清理用户配置缓存
rm -rf ~/.config/libreoffice/4/user/temp/

# 更新字体缓存
fc-cache -f -v
```

### 更新LibreOffice

```bash
# 系统包管理器更新
sudo apt update && sudo apt upgrade libreoffice-core

# 手动更新（下载新版本并重复安装步骤）
# 备份当前配置
cp -r ~/.config/libreoffice ~/.config/libreoffice.backup
```

---

**📋 配置检查清单**

使用以下命令验证LibreOffice配置是否正确：

```bash
# 完整验证脚本
cat > verify_libreoffice.sh << 'EOF'
#!/bin/bash
echo "CUCDFD系统 - LibreOffice配置验证"
echo "================================"

# 检查LibreOffice安装
if command -v libreoffice &> /dev/null; then
    echo "✅ LibreOffice已安装: $(libreoffice --version)"
else
    echo "❌ LibreOffice未安装"
    exit 1
fi

# 检查无头模式支持
if libreoffice --help | grep -q "headless"; then
    echo "✅ 无头模式支持正常"
else
    echo "❌ 无头模式不支持"
fi

# 测试PDF转换
echo "测试内容" > test_conversion.txt
if libreoffice --headless --convert-to pdf test_conversion.txt &>/dev/null; then
    if [ -f "test_conversion.pdf" ]; then
        echo "✅ PDF转换功能正常"
        rm -f test_conversion.txt test_conversion.pdf
    else
        echo "❌ PDF文件未生成"
    fi
else
    echo "❌ PDF转换失败"
fi

# 检查字体
font_count=$(fc-list | wc -l)
echo "✅ 系统字体数量: $font_count"

echo "================================"
echo "LibreOffice配置验证完成"
EOF

chmod +x verify_libreoffice.sh
./verify_libreoffice.sh
rm -f verify_libreoffice.sh
```

**注意**: 在生产环境中部署时，请确保LibreOffice的安装路径和权限配置正确，以避免PDF报告生成失败。
