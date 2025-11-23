# Yifu Guo - Personal Website

个人网站项目 | AI研究者个人主页

## 🌐 快速预览

一个现代、专业的AI研究者个人网站，结合了极简学术风格和精致视觉设计。

### ✨ 核心特性

- 🌓 **深色/浅色主题切换** - 自动保存用户偏好
- 🎨 **全宽Hero Section** - 渐变背景 + 个人展示
- 📄 **卡片式论文展示** - 带会议徽章和链接
- 📱 **完全响应式** - 桌面、平板、手机自适应
- ✨ **流畅动画** - 悬停效果、滚动渐入
- 🚀 **零依赖** - 纯HTML/CSS/JS，快速加载

## 📦 项目文件

```
.
├── yifu_website.html       # 主HTML文件
├── yifu_style.css          # CSS样式（含深色主题）
├── yifu_script.js          # JavaScript交互
├── personimage.png         # 个人照片
├── WEBSITE_GUIDE.md        # 详细使用指南
├── yifu_design_analysis.md # 设计说明文档
└── README_WEBSITE.md       # 本文件
```

## 🚀 使用方法

### 1. 本地预览

**方法A：直接打开**
```bash
# 双击 yifu_website.html 在浏览器中打开
open yifu_website.html  # macOS
```

**方法B：本地服务器（推荐）**
```bash
# 使用Python
python3 -m http.server 8000
# 访问 http://localhost:8000/yifu_website.html

# 或使用Node.js
npx http-server
# 访问显示的地址
```

### 2. 部署到GitHub Pages

```bash
# 1. 重命名为 index.html
mv yifu_website.html index.html

# 2. 提交到仓库
git add .
git commit -m "Add personal website"
git push origin main

# 3. 在GitHub仓库设置中启用Pages
# Settings → Pages → Source: main branch → Save

# 4. 访问 https://euyis1019.github.io
```

### 3. 其他部署选项

- **Netlify**: 拖拽文件夹即可部署
- **Vercel**: 连接GitHub仓库自动部署
- **Cloudflare Pages**: 快速全球CDN

## ✏️ 快速自定义

### 修改个人信息

在 `yifu_website.html` 中查找并修改：

```html
<!-- Hero Section -->
<h1 class="hero-title">你的名字</h1>
<p class="hero-subtitle">你的职位</p>

<!-- 社交链接 -->
<a href="https://github.com/你的用户名" ...>
```

### 更换配色

在 `yifu_style.css` 中修改：

```css
:root {
    --accent-color: #2563eb;  /* 主色调 */
}

/* Hero背景渐变 */
.hero {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}
```

### 添加论文

复制现有的论文卡片，修改内容即可：

```html
<div class="publication-card">
    <div class="pub-header">
        <span class="pub-badge pub-badge-neurips">NeurIPS 2025</span>
        <span class="pub-type">Poster</span>
    </div>
    <h3 class="pub-title">你的论文标题</h3>
    <!-- ... -->
</div>
```

## 📚 文档

- **[完整使用指南](WEBSITE_GUIDE.md)** - 详细的自定义教程
- **[设计说明](yifu_design_analysis.md)** - 设计理念和特色说明

## 🎨 设计特色

### 风格定位
**「现代学术极简 + 精致视觉」**

结合了：
- Jason Wei 的极简学术风格
- Yingchao Li 的视觉丰富设计
- 现代Web设计最佳实践

### 技术亮点

- **CSS变量** - 主题切换
- **Flexbox/Grid** - 现代布局
- **Intersection Observer** - 滚动动画
- **LocalStorage** - 记住用户偏好
- **响应式设计** - 移动优先

## 🔧 浏览器支持

- ✅ Chrome/Edge (推荐)
- ✅ Firefox
- ✅ Safari
- ✅ 移动浏览器

需要现代浏览器支持CSS变量和Intersection Observer API。

## 📈 未来计划

- [ ] 添加移动端汉堡菜单
- [ ] 集成博客系统（Jekyll/Hugo）
- [ ] 添加项目详情页
- [ ] SEO优化
- [ ] Google Analytics
- [ ] 多语言支持

## 🤝 贡献

这是你的个人网站，完全可自由修改！

如果你有改进建议或发现问题，欢迎：
- 提交Issue
- 发送Pull Request
- 联系我讨论

## 📄 许可

个人使用，无限制。

## 📧 联系方式

**Yifu Guo**
- GitHub: [@euyis1019](https://github.com/euyis1019)
- Email: 1572189162@qq.com

---

**Made with ❤️ for AI Researchers**

祝你的个人网站大放异彩！🚀

