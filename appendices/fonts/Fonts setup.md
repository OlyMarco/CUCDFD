# CUCDFD 深度鉴伪系统 - 字体安装配置指南

## 概述

字体配置是CUCDFD系统生成高质量PDF检测报告的重要组成部分。本指南提供完整的字体安装和配置说明，确保检测报告中的文本正确显示。

## 系统要求

- **操作系统**: Ubuntu 22.04+ / 其他 Linux 发行版
- **磁盘空间**: 建议 ≥ 500MB 可用空间（用于字体文件）
- **权限**: 用户级安装权限

## 字体安装方法

### 方法一：系统默认字体安装（推荐）

#### 安装基础字体包

```bash
# 更新包列表
sudo apt update

# 安装基础英文字体
sudo apt install -y fonts-liberation fonts-dejavu fonts-noto

# 安装中文字体支持
sudo apt install -y fonts-wqy-microhei fonts-wqy-zenhei fonts-arphic-ukai fonts-arphic-uming

# 安装额外的Unicode字体
sudo apt install -y fonts-noto-cjk fonts-noto-color-emoji

# 刷新字体缓存
sudo fc-cache -f -v
```

#### 验证系统字体安装

```bash
# 列出所有已安装的字体
fc-list

# 查看中文字体
fc-list :lang=zh-cn

# 查看英文字体
fc-list :lang=en
```

### 方法二：用户自定义字体安装

如果需要安装特定的字体文件（如Windows字体或其他自定义字体）：

#### 1. 创建字体目录

```bash
# 创建用户字体目录
mkdir -p ~/.local/share/fonts/cucdfd-fonts

# 设置合适的权限
chmod 755 ~/.local/share/fonts/cucdfd-fonts
```

#### 2. 安装字体文件

```bash
# 项目提供字体压缩包Fonts.zip
# unzip Fonts.zip -d /tmp/fonts_temp

# 复制字体文件到用户字体目录
# 支持的字体格式: .ttf, .otf, .woff, .woff2
cp /path/to/your/fonts/*.ttf ~/.local/share/fonts/cucdfd-fonts/
cp /path/to/your/fonts/*.otf ~/.local/share/fonts/cucdfd-fonts/

# 示例：安装常用Windows字体（如果有的话）
# cp Windows_Fonts/*.ttf ~/.local/share/fonts/cucdfd-fonts/
```

#### 3. 更新字体缓存

```bash
# 进入字体目录
cd ~/.local/share/fonts/cucdfd-fonts

# 生成字体索引文件（可选）
mkfontscale 2>/dev/null || true
mkfontdir 2>/dev/null || true

# 刷新系统字体缓存
fc-cache -fv

# 验证字体缓存更新
fc-cache -v | grep cucdfd-fonts
```

## 故障排除

### 常见问题及解决方案

#### 1. 字体缓存问题

```bash
# 清理并重建字体缓存
rm -rf ~/.cache/fontconfig
fc-cache -fv

# 如果仍有问题，清理系统字体缓存
sudo rm -rf /var/cache/fontconfig
sudo fc-cache -fv
```

#### 2. 中文字体显示问题

```bash
# 检查中文字体
fc-list :lang=zh-cn

# 如果中文字体不足，安装额外支持
sudo apt install -y fonts-noto-cjk-extra fonts-wqy-microhei fonts-wqy-zenhei

# 设置中文环境变量
export LANG=zh_CN.UTF-8
export LC_ALL=zh_CN.UTF-8
```

#### 3. PDF生成字体嵌入问题

```bash
# 测试字体嵌入
cat > test_embed.txt << 'EOF'
测试中文字体嵌入
Test English Font Embedding
EOF

# 使用LibreOffice转换时指定字体嵌入选项
libreoffice --headless --convert-to pdf --outdir . test_embed.txt

# 检查PDF中的字体信息（需要安装pdfinfo）
if command -v pdfinfo &> /dev/null; then
    pdfinfo test_embed.pdf | grep Font
fi

# 清理测试文件
rm -f test_embed.txt test_embed.pdf
```

#### 4. 权限和路径问题

```bash
# 检查字体目录权限
ls -la ~/.local/share/fonts/

# 修复权限
chmod -R 644 ~/.local/share/fonts/cucdfd-fonts/*.ttf
chmod -R 644 ~/.local/share/fonts/cucdfd-fonts/*.otf
chmod 755 ~/.local/share/fonts/cucdfd-fonts/

# 检查字体文件完整性
find ~/.local/share/fonts/cucdfd-fonts/ -name "*.ttf" -o -name "*.otf" | while read font; do
    if file "$font" | grep -q "TrueType\|OpenType"; then
        echo "✅ $font"
    else
        echo "❌ $font (可能损坏)"
    fi
done
```

### 系统字体环境检查

```bash
# 创建完整的字体环境检查脚本
cat > check_font_environment.sh << 'EOF'
#!/bin/bash

echo "CUCDFD系统字体环境检查报告"
echo "=========================="
echo "检查时间: $(date)"
echo ""

# 1. 基础字体工具检查
echo "1. 字体工具检查:"
for tool in fc-list fc-cache mkfontscale mkfontdir; do
    if command -v $tool &> /dev/null; then
        echo "   ✅ $tool"
    else
        echo "   ❌ $tool (未安装)"
    fi
done
echo ""

# 2. 系统字体统计
echo "2. 系统字体统计:"
total_fonts=$(fc-list | wc -l)
echo "   总字体数量: $total_fonts"

chinese_fonts=$(fc-list :lang=zh-cn | wc -l)
echo "   中文字体数量: $chinese_fonts"

english_fonts=$(fc-list :lang=en | wc -l)
echo "   英文字体数量: $english_fonts"
echo ""

# 3. 重要字体系列检查
echo "3. 重要字体系列:"
for family in "Liberation" "DejaVu" "Noto" "WenQuanYi"; do
    count=$(fc-list | grep -i "$family" | wc -l)
    if [ $count -gt 0 ]; then
        echo "   ✅ $family: $count 个字体"
    else
        echo "   ❌ $family: 未找到"
    fi
done
echo ""

# 4. 字体目录检查
echo "4. 字体目录检查:"
for dir in "/usr/share/fonts" "/usr/local/share/fonts" "$HOME/.local/share/fonts"; do
    if [ -d "$dir" ]; then
        font_count=$(find "$dir" -name "*.ttf" -o -name "*.otf" | wc -l)
        echo "   ✅ $dir: $font_count 个字体文件"
    else
        echo "   ❌ $dir: 目录不存在"
    fi
done
echo ""

# 5. LibreOffice集成检查
echo "5. LibreOffice集成检查:"
if command -v libreoffice &> /dev/null; then
    echo "   ✅ LibreOffice已安装"
    
    # 测试基本功能
    echo "测试内容" > /tmp/font_test.txt
    if libreoffice --headless --convert-to pdf /tmp/font_test.txt --outdir /tmp &>/dev/null; then
        if [ -f "/tmp/font_test.pdf" ]; then
            echo "   ✅ PDF转换功能正常"
            rm -f /tmp/font_test.txt /tmp/font_test.pdf
        else
            echo "   ❌ PDF文件生成失败"
        fi
    else
        echo "   ❌ PDF转换功能异常"
    fi
else
    echo "   ❌ LibreOffice未安装"
fi
echo ""

echo "=========================="
echo "字体环境检查完成"
EOF

chmod +x check_font_environment.sh
./check_font_environment.sh
rm -f check_font_environment.sh
```

## 维护和更新

### 定期维护

```bash
# 定期清理字体缓存
fc-cache -fv

# 检查损坏的字体文件
find ~/.local/share/fonts/ -name "*.ttf" -o -name "*.otf" | while read font; do
    if ! file "$font" | grep -q "TrueType\|OpenType"; then
        echo "可能损坏的字体: $font"
    fi
done

# 更新系统字体包
sudo apt update && sudo apt upgrade fonts-*
```

### 备份和恢复

```bash
# 备份用户字体配置
tar -czf cucdfd_fonts_backup_$(date +%Y%m%d).tar.gz ~/.local/share/fonts/cucdfd-fonts/ ~/.config/fontconfig/

# 恢复字体配置
# tar -xzf cucdfd_fonts_backup_20250120.tar.gz -C ~/
```

---

**📋 字体配置检查清单**

- ✅ 系统基础字体已安装
- ✅ 中文字体支持已配置  
- ✅ 用户自定义字体已安装
- ✅ 字体缓存已更新
- ✅ LibreOffice字体集成正常
- ✅ PDF生成测试通过

**注意**: 在CUCDFD系统中，字体配置直接影响检测报告的质量和可读性。确保安装了足够的中英文字体，以支持各种字符的正确显示。
