# 🔤 CUCDFD 字体配置

> 高质量PDF报告字体支持系统

## 🚀 核心特性

- **📝 多语言支持** - 中英文字体完美兼容
- **📄 PDF优化** - 专为检测报告优化的字体集
- **⚡ 快速安装** - 一键安装字体包
- **🔧 自动配置** - 智能检测和缓存更新
- **💾 空间优化** - 精选字体，节省存储空间

## 📋 环境要求

- Ubuntu 22.04+ / Linux 系统
- 磁盘空间: ≥ 500MB
- 用户级安装权限

## ⚡ 快速安装

```bash
# 创建字体目录
mkdir -p ~/.local/share/fonts/cucdfd-fonts

# 解压字体包
unzip Fonts.zip -d ~/.local/share/fonts/cucdfd-fonts/

# 设置权限
chmod 755 ~/.local/share/fonts/cucdfd-fonts

# 更新字体缓存
fc-cache -fv
```

## ✅ 验证安装

```bash
# 检查字体列表
fc-list | grep cucdfd

# 验证中文字体
fc-list :lang=zh-cn

# 测试字体渲染
echo "字体安装成功 Font Test OK" | convert label:@- test.png
```

## 📊 字体包内容

| 类型 | 字体名称 | 用途 | 大小 |
|------|----------|------|------|
| 🇨🇳 **中文** | Noto Sans CJK | 报告正文 | ~50MB |
| 🇺🇸 **英文** | Liberation Sans | 标题标签 | ~10MB |
| 🔢 **等宽** | Source Code Pro | 数据展示 | ~5MB |

## 🔧 故障排除

### 常见问题

1. **字体未生效**
   ```bash
   # 强制刷新缓存
   sudo fc-cache -f -v
   ```

2. **PDF显示异常**
   - ✅ 检查LibreOffice字体配置
   - ✅ 验证字体文件完整性

3. **权限问题**
   ```bash
   # 修复权限
   chmod -R 644 ~/.local/share/fonts/cucdfd-fonts/
   ```

## 📋 安装检查清单

- ✅ 字体目录已创建
- ✅ 字体文件已解压
- ✅ 权限配置正确
- ✅ 字体缓存已更新
- ✅ 中文显示正常
- ✅ PDF生成测试通过

---

**💡 提示**: 字体配置影响PDF报告质量，请确保完整安装
