# 个人网站使用指南

## 📦 文件说明

你的个人网站包含以下文件：

```
├── yifu_website.html    # 主HTML文件（网站结构和内容）
├── yifu_style.css       # CSS样式文件（视觉设计和主题）
├── yifu_script.js       # JavaScript文件（交互功能和深色模式）
└── personimage.png      # 你的个人头像照片
```

## 🎨 设计特色

### ✅ 已实现的功能

1. **深色/浅色主题切换** 🌙☀️
   - 点击右上角的月亮/太阳图标即可切换
   - 自动记住用户偏好（localStorage）
   - 支持系统主题检测

2. **全宽Hero Section** 🎭
   - 渐变背景（紫色系）
   - 个人照片展示
   - 社交媒体链接
   - 视觉吸引力强

3. **卡片式论文展示** 📄
   - 会议徽章（AAAI、NeurIPS、PRICAI等）
   - 悬停动画效果
   - 论文链接、代码、媒体报道
   - 清晰的视觉层次

4. **响应式设计** 📱
   - 自适应桌面、平板、手机
   - 流畅的动画和过渡效果

5. **交互特效** ✨
   - 滚动渐入动画
   - 导航栏阴影效果
   - Hero区域视差效果
   - 悬停效果

## 🚀 快速开始

### 方法一：本地预览

1. 直接双击 `yifu_website.html` 文件在浏览器中打开
2. 或者使用本地服务器（推荐）：

```bash
# 使用Python
python3 -m http.server 8000

# 或者使用Node.js
npx http-server

# 然后访问 http://localhost:8000/yifu_website.html
```

### 方法二：部署到GitHub Pages

1. 将文件重命名为 `index.html`：
```bash
mv yifu_website.html index.html
```

2. 提交到GitHub仓库：
```bash
git add index.html yifu_style.css yifu_script.js personimage.png
git commit -m "Add personal website"
git push origin main
```

3. 在GitHub仓库设置中启用GitHub Pages
   - Settings → Pages → Source: main branch → Save

4. 访问 `https://euyis1019.github.io`

## ✏️ 自定义指南

### 1. 修改个人信息

**Hero Section（首屏）**
```html
<!-- 在 yifu_website.html 中查找并修改 -->
<h1 class="hero-title">你的名字</h1>
<p class="hero-subtitle">你的职位/研究方向</p>
<p class="hero-description">你的个人Slogan</p>
```

**About Me（关于我）**
```html
<p class="about-text">
    修改这里的个人介绍文字...
</p>
```

### 2. 添加/删除论文

复制以下模板，修改内容：

```html
<div class="publication-card">
    <div class="pub-header">
        <span class="pub-badge pub-badge-neurips">会议名称 2025</span>
        <span class="pub-type">Poster/Oral</span>
    </div>
    <h3 class="pub-title">论文标题</h3>
    <p class="pub-authors"><strong>作者身份</strong></p>
    <p class="pub-abstract">论文摘要简介...</p>
    <div class="pub-links">
        <a href="#" target="_blank" class="pub-link">
            <i class="fas fa-file-pdf"></i> PDF
        </a>
        <a href="#" target="_blank" class="pub-link">
            <i class="fab fa-github"></i> Code
        </a>
    </div>
</div>
```

**会议徽章样式**：
- `.pub-badge-neurips` - NeurIPS（紫色渐变）
- `.pub-badge-aaai` - AAAI（粉红渐变）
- `.pub-badge-pricai` - PRICAI（蓝色渐变）
- `.pub-badge-under-review` - Under Review（橙黄渐变）

### 3. 修改配色方案

**在 `yifu_style.css` 中修改**：

```css
:root {
    /* 主色调 - 链接、按钮等 */
    --accent-color: #2563eb;  /* 改成你喜欢的颜色 */
    
    /* 其他颜色变量 */
    --bg-primary: #ffffff;
    --text-primary: #1a1a1a;
}
```

**Hero Section背景渐变**：

```css
/* 在 CSS 中查找 .hero 类，修改背景 */
.hero {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

/* 其他渐变选项 */
/* 蓝紫渐变：linear-gradient(135deg, #6366f1 0%, #8b5cf6 50%, #d946ef 100%) */
/* 橙红渐变：linear-gradient(135deg, #f093fb 0%, #f5576c 100%) */
/* 绿蓝渐变：linear-gradient(135deg, #4facfe 0%, #00f2fe 100%) */
```

### 4. 更换个人照片

1. 将你的照片命名为 `personimage.png`
2. 放在网站文件同一目录
3. 或者在HTML中修改路径：

```html
<img src="你的照片路径.jpg" alt="Yifu Guo" class="hero-image">
```

**推荐照片规格**：
- 正方形（1:1比例）
- 至少 500x500 像素
- JPG或PNG格式

### 5. 添加真实的社交链接

```html
<!-- 在HTML中查找并修改 -->
<div class="social-links">
    <a href="https://github.com/你的用户名" target="_blank" class="social-link">
        <i class="fab fa-github"></i>
    </a>
    <a href="mailto:你的邮箱@email.com" class="social-link">
        <i class="fas fa-envelope"></i>
    </a>
    <a href="https://scholar.google.com/你的链接" target="_blank" class="social-link">
        <i class="fas fa-graduation-cap"></i>
    </a>
    <a href="https://twitter.com/你的用户名" target="_blank" class="social-link">
        <i class="fab fa-twitter"></i>
    </a>
</div>
```

### 6. 博客功能

**临时方案**（当前）：
博客卡片是占位符，链接到 `#`

**未来集成方案**：

**选项A：链接到外部博客**
```html
<a href="https://你的博客网址.com/文章标题" class="blog-link">Read more →</a>
```

**选项B：使用Markdown + Jekyll**
1. 转换为Jekyll网站
2. 在 `_posts/` 文件夹中添加Markdown文章
3. 使用Liquid模板自动生成博客列表

**选项C：使用独立博客页面**
1. 创建 `blog.html`
2. 每篇文章一个HTML文件
3. 在Blog section链接到具体文章

## 🎨 进阶自定义

### 添加Google Analytics

在 `</head>` 前添加：

```html
<!-- Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=YOUR-GA-ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'YOUR-GA-ID');
</script>
```

### 添加SEO优化

在 `<head>` 中添加：

```html
<meta name="description" content="Yifu Guo - AI Researcher specializing in Multimodal Learning, Agents, and Continual Learning">
<meta name="keywords" content="AI, Machine Learning, Continual Learning, Multimodal, NeurIPS, AAAI">
<meta name="author" content="Yifu Guo">

<!-- Open Graph for social media -->
<meta property="og:title" content="Yifu Guo - AI Researcher">
<meta property="og:description" content="Personal website showcasing research in AI and Machine Learning">
<meta property="og:image" content="https://你的网站.com/personimage.png">
<meta property="og:url" content="https://你的网站.com">
```

### 添加Favicon

```html
<link rel="icon" type="image/png" href="favicon.png">
```

## 📱 响应式预览

网站已针对以下设备优化：
- 💻 **桌面**（>1024px）：完整布局
- 📱 **平板**（768px-1024px）：两列布局
- 📱 **手机**（<768px）：单列布局

## 🔧 常见问题

### Q: 如何移除某个section？
A: 在HTML中删除对应的 `<section>` 标签即可

### Q: 导航栏链接点击后没反应？
A: 确保 `yifu_script.js` 文件正确加载

### Q: 深色模式不工作？
A: 检查浏览器控制台是否有JavaScript错误

### Q: 想要不同的Hero背景（图片而非渐变）？
A: 修改CSS：
```css
.hero {
    background: url('你的背景图.jpg') center/cover;
}
```

### Q: 如何添加CV下载链接？
A: 在社交链接区域添加：
```html
<a href="myresume.pdf" download class="social-link" aria-label="Download CV">
    <i class="fas fa-download"></i>
</a>
```

## 🚀 下一步建议

1. ✅ **立即可做**：
   - 更换为真实个人照片
   - 更新所有社交媒体链接
   - 检查论文链接是否正确

2. 📝 **短期计划**：
   - 添加真实博客内容
   - 上传CV文件并添加下载链接
   - 添加Google Analytics追踪

3. 🎯 **长期优化**：
   - 添加项目详情页面
   - 集成博客系统（Jekyll/Hugo）
   - 添加评论系统（Disqus/Giscus）
   - 优化SEO和加载速度

## 📧 需要帮助？

如果遇到问题或需要进一步定制，随时联系我！

---

**祝你的个人网站大获成功！** 🎉

