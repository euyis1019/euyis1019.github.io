# Jason Wei 网站设计分析

**网址**: https://www.jasonwei.net/

## 平台与技术栈

- **平台**: Squarespace
- **特点**: 使用 Squarespace 的模板系统，但经过精心定制

## 整体设计风格

### 核心特征
- **极简主义**: 干净、专业、无多余装饰
- **学术风格**: 突出内容而非视觉噱头
- **响应式设计**: 移动端友好

### 视觉风格关键词
- Minimalist（极简）
- Professional（专业）
- Clean（清爽）
- Academic（学术）
- Elegant（优雅）

## 设计元素详解

### 1. 布局结构

```
┌─────────────────────────────────────┐
│         导航栏（固定/粘性）          │
│   Jason Wei | Home Papers Blog ...  │
├─────────────────────────────────────┤
│                                     │
│         头像/照片（居中）            │
│                                     │
│         个人简介（居中）             │
│         I am an American...         │
│                                     │
│         链接按钮组（居中）           │
│    Twitter / CV / Scholar / Email   │
│                                     │
├─────────────────────────────────────┤
│                                     │
│         Papers（列表）               │
│         ┌─────────────────────┐    │
│         │ 2025 Apr | Title    │    │
│         │ 2024 Oct | Title    │    │
│         │ ...                 │    │
│         └─────────────────────┘    │
│                                     │
├─────────────────────────────────────┤
│         Talks & Media（列表）        │
│                                     │
└─────────────────────────────────────┘
```

### 2. 颜色方案

**主色调**:
- 背景: `#FFFFFF` (纯白)
- 主文本: `#000000` or `#1a1a1a` (深黑或近黑)
- 链接: `#0066CC` or 类似蓝色（Squarespace 默认）
- 链接悬停: 可能更深的蓝色

**配色特点**:
- 高对比度黑白配色
- 极简用色，只在链接处使用颜色
- 专业学术风格

### 3. 字体排印

**推测的字体栈** (Squarespace 常用):

```css
/* 标题字体 */
font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
/* 或者可能是 */
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;

/* 正文字体 */
font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
/* 或者 Georgia, Times New Roman 等 serif 字体 */
```

**字体大小层级**:
- H1 (名字): ~32-48px
- H2 (章节标题): ~24-28px  
- 正文: ~16-18px
- 小字: ~14px

**字体特点**:
- Sans-serif 字体为主
- 清晰易读
- 合适的行高 (line-height: 1.5-1.8)
- 适当的字间距

### 4. 间距系统

```css
/* 推测的间距系统 */
--spacing-xs: 8px;
--spacing-sm: 16px;
--spacing-md: 24px;
--spacing-lg: 32px;
--spacing-xl: 48px;
--spacing-2xl: 64px;

/* 应用 */
section {
  margin-bottom: var(--spacing-2xl);
  padding: var(--spacing-lg);
}
```

### 5. 表格样式

**Papers 和 Talks 列表使用表格布局**:

```css
table {
  width: 100%;
  border-collapse: collapse;
  margin: 2em 0;
}

td {
  padding: 0.5em 1em;
  vertical-align: top;
  border-bottom: 1px solid #eee; /* 可能有浅灰色分割线 */
}

/* 第一列（日期）*/
td:first-child {
  white-space: nowrap;
  color: #666; /* 较浅的颜色 */
  font-size: 0.9em;
}

/* 第二列（内容）*/
td:last-child {
  width: 100%;
}
```

### 6. 导航栏

```css
nav {
  position: sticky;
  top: 0;
  background: white;
  border-bottom: 1px solid #eee;
  padding: 1rem 2rem;
  z-index: 1000;
}

nav a {
  text-decoration: none;
  color: #000;
  margin: 0 1rem;
  font-weight: 500;
}

nav a:hover {
  color: #0066CC; /* 悬停变色 */
}
```

### 7. 链接样式

```css
a {
  color: #0066CC;
  text-decoration: none;
  transition: color 0.2s ease;
}

a:hover {
  color: #004499;
  text-decoration: underline;
}

/* 社交媒体链接可能有特殊样式 */
.social-links a {
  display: inline-block;
  margin: 0 0.5rem;
  padding: 0.5rem 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}
```

## 复现这个设计的方法

### 方法一：使用 Squarespace 模板

1. 选择 Squarespace 的极简模板（如 "Bedlow" 或 "Forma"）
2. 自定义样式：
   - 设置全白背景
   - 使用简洁的 sans-serif 字体
   - 移除不必要的装饰元素

### 方法二：使用 Jekyll/Hugo 静态站点生成器

**推荐主题**:
- Jekyll: `minimal-mistakes`, `academic`, `al-folio`
- Hugo: `academic` (现在叫 Wowchemy), `PaperMod`

**自定义 CSS**:
```css
/* custom.css */
:root {
  --main-bg: #ffffff;
  --main-text: #1a1a1a;
  --link-color: #0066CC;
  --border-color: #eee;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  color: var(--main-text);
  background: var(--main-bg);
  line-height: 1.6;
  max-width: 900px;
  margin: 0 auto;
  padding: 2rem;
}

header {
  text-align: center;
  margin-bottom: 3rem;
}

h1 {
  font-size: 2.5rem;
  margin-bottom: 1rem;
}

.bio {
  max-width: 600px;
  margin: 2rem auto;
  text-align: center;
  line-height: 1.8;
}

.social-links {
  margin: 2rem 0;
  text-align: center;
}

section {
  margin: 3rem 0;
}

h2 {
  font-size: 1.5rem;
  margin-bottom: 1.5rem;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid var(--border-color);
}

table {
  width: 100%;
  margin: 2rem 0;
}

td {
  padding: 0.75rem 0;
  vertical-align: top;
}

td:first-child {
  color: #666;
  white-space: nowrap;
  padding-right: 2rem;
}
```

### 方法三：纯 HTML/CSS 手写

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Your Name</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <nav>
        <a href="/">Your Name</a>
        <a href="/papers">Papers</a>
        <a href="/blog">Blog</a>
    </nav>
    
    <main>
        <header>
            <img src="profile.jpg" alt="Profile" class="profile-img">
            <h1>Your Name</h1>
            <p class="bio">
                I am a researcher...
            </p>
            <div class="social-links">
                <a href="#">Twitter</a>
                <a href="#">CV</a>
                <a href="#">Google Scholar</a>
                <a href="#">Email</a>
            </div>
        </header>
        
        <section>
            <h2>Papers</h2>
            <table>
                <tr>
                    <td>2025 Apr</td>
                    <td><a href="#">Paper title</a></td>
                </tr>
                <!-- More papers -->
            </table>
        </section>
    </main>
</body>
</html>
```

## 关键设计原则

1. **内容为王**: 设计服务于内容，不喧宾夺主
2. **空白空间**: 充足的留白让内容呼吸
3. **易读性**: 高对比度，合适的字体大小
4. **一致性**: 统一的间距、颜色、字体
5. **移动优先**: 响应式设计，移动端良好体验

## 与 Lilian Weng 对比

| 特性 | Lilian Weng | Jason Wei |
|------|-------------|-----------|
| 平台 | Jekyll (GitHub Pages) | Squarespace |
| 视觉吸引力 | ⭐⭐ | ⭐⭐⭐⭐ |
| 专业度 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 易用性 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 定制化 | ⭐⭐⭐ | ⭐⭐⭐ |

**Jason Wei 的优势**:
- 更现代的视觉设计
- 更好的间距和排版
- 图片展示（头像）
- 社交链接更突出
- 整体更有"个性"

**保持的共同点**:
- 都极简风格
- 专注内容
- 学术专业

## 推荐工具和资源

### 字体
- **系统字体栈**: 最快速，无需加载
- **Google Fonts**: Inter, Roboto, Open Sans
- **专业选择**: SF Pro (Apple), Segoe UI (Microsoft)

### 颜色
- **工具**: Coolors.co, Adobe Color
- **建议**: 保持 2-3 种颜色

### 开发框架
- **静态生成**: Hugo, Jekyll, Gatsby, Next.js
- **托管**: GitHub Pages, Netlify, Vercel
- **CMS**: Squarespace (最简单), WordPress

## 总结

Jason Wei 的网站是"精致极简主义"的典范：
- 比 Lilian Weng 更有视觉吸引力
- 但保持了学术网站应有的专业性
- 适合研究人员展示成果和个人品牌
- 易于维护和更新

这种风格特别适合你这样有技术背景但想要更好视觉呈现的研究者！
