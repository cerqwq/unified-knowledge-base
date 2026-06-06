# 现代 CSS 技术

> 📅 创建时间：2026-05-30
> 🎯 目标：掌握现代 CSS 特性和技巧

---

## 一、CSS 变量（Custom Properties）

### 1.1 基础用法
```css
:root {
    /* 定义变量 */
    --color-primary: #4a90e2;
    --color-secondary: #7b68ee;
    --color-success: #52c41a;
    --color-warning: #faad14;
    --color-danger: #ff4d4f;
    
    /* 间距 */
    --spacing-xs: 4px;
    --spacing-sm: 8px;
    --spacing-md: 16px;
    --spacing-lg: 24px;
    --spacing-xl: 32px;
    
    /* 字体 */
    --font-family: 'Segoe UI', sans-serif;
    --font-size-sm: 14px;
    --font-size-md: 16px;
    --font-size-lg: 18px;
    
    /* 圆角 */
    --radius-sm: 4px;
    --radius-md: 8px;
    --radius-lg: 12px;
    
    /* 阴影 */
    --shadow-sm: 0 2px 4px rgba(0, 0, 0, 0.1);
    --shadow-md: 0 4px 8px rgba(0, 0, 0, 0.12);
    --shadow-lg: 0 8px 16px rgba(0, 0, 0, 0.15);
}

/* 使用变量 */
.button {
    background: var(--color-primary);
    padding: var(--spacing-sm) var(--spacing-md);
    border-radius: var(--radius-md);
    font-family: var(--font-family);
    box-shadow: var(--shadow-sm);
}

/* 变量作用域 */
.card {
    --card-bg: white;
    --card-padding: var(--spacing-lg);
    
    background: var(--card-bg);
    padding: var(--card-padding);
}

.card.dark {
    --card-bg: #1a1a1a;
}
```

### 1.2 动态主题
```css
/* 主题系统 */
[data-theme="light"] {
    --bg-primary: #ffffff;
    --bg-secondary: #f5f5f5;
    --text-primary: #333333;
    --text-secondary: #666666;
    --border-color: #e0e0e0;
}

[data-theme="dark"] {
    --bg-primary: #1a1a1a;
    --bg-secondary: #2a2a2a;
    --text-primary: #ffffff;
    --text-secondary: #cccccc;
    --border-color: #444444;
}

/* 使用主题变量 */
body {
    background: var(--bg-primary);
    color: var(--text-primary);
}

.card {
    background: var(--bg-secondary);
    border: 1px solid var(--border-color);
}
```

### 1.3 响应式变量
```css
:root {
    --container-width: 100%;
    --grid-columns: 1;
    --font-scale: 1;
}

@media (min-width: 576px) {
    :root {
        --container-width: 540px;
        --grid-columns: 2;
    }
}

@media (min-width: 768px) {
    :root {
        --container-width: 720px;
        --grid-columns: 3;
        --font-scale: 1.1;
    }
}

@media (min-width: 1024px) {
    :root {
        --container-width: 960px;
        --grid-columns: 4;
        --font-scale: 1.2;
    }
}

.container {
    max-width: var(--container-width);
    margin: 0 auto;
}

.grid {
    display: grid;
    grid-template-columns: repeat(var(--grid-columns), 1fr);
    gap: var(--spacing-md);
}

h1 {
    font-size: calc(2rem * var(--font-scale));
}
```

---

## 二、CSS Grid 高级布局

### 2.1 基础网格
```css
/* 基础网格 */
.grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
    gap: 16px;
}

/* 命名网格线 */
.layout {
    display: grid;
    grid-template-columns: 
        [sidebar-start] 250px 
        [sidebar-end content-start] 1fr 
        [content-end];
    grid-template-rows: 
        [header-start] 80px 
        [header-end main-start] 1fr 
        [main-end footer-start] 60px 
        [footer-end];
    gap: 16px;
    min-height: 100vh;
}

.header {
    grid-column: sidebar-start / content-end;
    grid-row: header-start / header-end;
}

.sidebar {
    grid-column: sidebar-start / sidebar-end;
    grid-row: main-start / main-end;
}

.main {
    grid-column: content-start / content-end;
    grid-row: main-start / main-end;
}

.footer {
    grid-column: sidebar-start / content-end;
    grid-row: footer-start / footer-end;
}
```

### 2.2 自动填充和自适应
```css
/* 自动填充 */
.auto-fill {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 16px;
}

/* 自动适应 */
.auto-fit {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 16px;
}

/* 网格区域 */
.dashboard {
    display: grid;
    grid-template-areas:
        "header header header"
        "sidebar main aside"
        "footer footer footer";
    grid-template-columns: 200px 1fr 200px;
    grid-template-rows: auto 1fr auto;
    gap: 16px;
    min-height: 100vh;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }

/* 响应式网格区域 */
@media (max-width: 768px) {
    .dashboard {
        grid-template-areas:
            "header"
            "main"
            "sidebar"
            "aside"
            "footer";
        grid-template-columns: 1fr;
    }
}
```

### 2.3 网格动画
```css
/* 网格项动画 */
.grid-item {
    animation: gridFadeIn 0.5s ease forwards;
    opacity: 0;
}

@keyframes gridFadeIn {
    from {
        opacity: 0;
        transform: scale(0.8);
    }
    to {
        opacity: 1;
        transform: scale(1);
    }
}

/* 交错动画 */
.grid-item:nth-child(1) { animation-delay: 0.1s; }
.grid-item:nth-child(2) { animation-delay: 0.2s; }
.grid-item:nth-child(3) { animation-delay: 0.3s; }
.grid-item:nth-child(4) { animation-delay: 0.4s; }

/* 网格过渡 */
.grid {
    transition: grid-template-columns 0.3s ease;
}

.grid:hover {
    grid-template-columns: 2fr 1fr 1fr;
}
```

---

## 三、CSS 容器查询

### 3.1 基础容器查询
```css
/* 定义容器 */
.card-container {
    container-type: inline-size;
    container-name: card;
}

/* 容器查询 */
@container card (min-width: 400px) {
    .card {
        display: grid;
        grid-template-columns: 200px 1fr;
        gap: 16px;
    }
    
    .card-image {
        aspect-ratio: 1;
    }
}

@container card (min-width: 600px) {
    .card {
        grid-template-columns: 300px 1fr;
    }
    
    .card-title {
        font-size: 1.5rem;
    }
}

/* 容器查询单位 */
.card-title {
    font-size: max(1rem, 4cqi);  /* 容器宽度的 4% */
}

.card-content {
    padding: 2cqi;
}
```

### 3.2 嵌套容器查询
```css
/* 外层容器 */
.layout {
    container-type: inline-size;
    container-name: layout;
}

/* 内层容器 */
.sidebar {
    container-type: inline-size;
    container-name: sidebar;
}

/* 布局级别查询 */
@container layout (min-width: 1024px) {
    .layout {
        display: grid;
        grid-template-columns: 250px 1fr;
    }
}

/* 侧边栏内部查询 */
@container sidebar (max-width: 200px) {
    .nav-item {
        padding: 8px;
        font-size: 0.875rem;
    }
    
    .nav-icon {
        display: none;
    }
}
```

### 3.3 容器查询与组件设计
```css
/* 响应式组件 */
.component {
    container-type: inline-size;
    container-name: component;
}

/* 小容器样式 */
@container component (max-width: 300px) {
    .component {
        padding: 8px;
    }
    
    .component-title {
        font-size: 1rem;
    }
    
    .component-actions {
        flex-direction: column;
    }
}

/* 中容器样式 */
@container component (min-width: 300px) and (max-width: 600px) {
    .component {
        padding: 16px;
    }
    
    .component-title {
        font-size: 1.25rem;
    }
    
    .component-actions {
        flex-direction: row;
    }
}

/* 大容器样式 */
@container component (min-width: 600px) {
    .component {
        padding: 24px;
    }
    
    .component-title {
        font-size: 1.5rem;
    }
    
    .component-content {
        columns: 2;
    }
}
```

---

## 四、CSS 嵌套

### 4.1 基础嵌套
```css
/* 原生 CSS 嵌套 */
.card {
    background: white;
    border-radius: 8px;
    padding: 16px;
    
    /* 嵌套选择器 */
    & .title {
        font-size: 1.5rem;
        margin-bottom: 8px;
    }
    
    & .content {
        color: #666;
    }
    
    /* 伪类 */
    &:hover {
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }
    
    /* 伪元素 */
    &::before {
        content: '';
        display: block;
    }
    
    /* 媒体查询 */
    @media (max-width: 768px) {
        padding: 12px;
        
        & .title {
            font-size: 1.25rem;
        }
    }
}
```

### 4.2 嵌套与 BEM
```css
/* BEM 风格嵌套 */
.block {
    /* Block 样式 */
    
    &__element {
        /* Element 样式 */
        
        &--modifier {
            /* Modifier 样式 */
        }
    }
    
    &--modifier {
        /* Block modifier */
        
        & .block__element {
            /* Element with block modifier */
        }
    }
}

/* 实际示例 */
.button {
    display: inline-flex;
    align-items: center;
    padding: 8px 16px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    
    &__icon {
        margin-right: 8px;
    }
    
    &__text {
        font-weight: 500;
    }
    
    &--primary {
        background: #4a90e2;
        color: white;
        
        &:hover {
            background: #357abd;
        }
    }
    
    &--secondary {
        background: transparent;
        border: 1px solid #4a90e2;
        color: #4a90e2;
        
        &:hover {
            background: #4a90e2;
            color: white;
        }
    }
}
```

---

## 五、CSS 动画高级技巧

### 5.1 滚动驱动动画
```css
/* 滚动驱动动画 */
.progress-bar {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 4px;
    background: #4a90e2;
    transform-origin: left;
    
    animation: progressBar linear;
    animation-timeline: scroll();
}

@keyframes progressBar {
    from {
        transform: scaleX(0);
    }
    to {
        transform: scaleX(1);
    }
}

/* 滚动触发动画 */
.reveal {
    opacity: 0;
    transform: translateY(50px);
    
    animation: reveal linear both;
    animation-timeline: view();
    animation-range: entry 0% cover 40%;
}

@keyframes reveal {
    from {
        opacity: 0;
        transform: translateY(50px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}
```

### 5.2 视图过渡动画
```css
/* 视图过渡 */
@view-transition {
    navigation: auto;
}

::view-transition-old(root) {
    animation: fade-out 0.3s ease-out;
}

::view-transition-new(root) {
    animation: fade-in 0.3s ease-in;
}

@keyframes fade-out {
    from { opacity: 1; }
    to { opacity: 0; }
}

@keyframes fade-in {
    from { opacity: 0; }
    to { opacity: 1; }
}

/* 自定义过渡 */
.page-title {
    view-transition-name: page-title;
}

::view-transition-old(page-title) {
    animation: slide-out 0.3s ease-out;
}

::view-transition-new(page-title) {
    animation: slide-in 0.3s ease-in;
}

@keyframes slide-out {
    from { transform: translateX(0); opacity: 1; }
    to { transform: translateX(-100%); opacity: 0; }
}

@keyframes slide-in {
    from { transform: translateX(100%); opacity: 0; }
    to { transform: translateX(0); opacity: 1; }
}
```

### 5.3 复杂动画序列
```css
/* 动画序列 */
.sequence {
    animation: 
        fadeIn 0.5s ease forwards,
        slideIn 0.5s ease 0.3s forwards,
        scaleIn 0.5s ease 0.6s forwards;
}

@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}

@keyframes slideIn {
    from { transform: translateY(20px); }
    to { transform: translateY(0); }
}

@keyframes scaleIn {
    from { transform: scale(0.8); }
    to { transform: scale(1); }
}

/* 弹簧动画 */
.spring {
    animation: spring 1s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
}

@keyframes spring {
    0% {
        transform: scale(0);
        opacity: 0;
    }
    50% {
        transform: scale(1.1);
    }
    100% {
        transform: scale(1);
        opacity: 1;
    }
}

/* 打字机效果 */
.typewriter {
    overflow: hidden;
    border-right: 2px solid;
    white-space: nowrap;
    animation: 
        typing 2s steps(20) forwards,
        blink 0.5s step-end infinite alternate;
}

@keyframes typing {
    from { width: 0; }
    to { width: 100%; }
}

@keyframes blink {
    50% { border-color: transparent; }
}
```

---

## 六、CSS 层叠层（Cascade Layers）

### 6.1 基础用法
```css
/* 定义层 */
@layer base, components, utilities;

/* 基础层 */
@layer base {
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }
    
    body {
        font-family: system-ui, sans-serif;
        line-height: 1.6;
    }
}

/* 组件层 */
@layer components {
    .button {
        padding: 8px 16px;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }
    
    .card {
        background: white;
        border-radius: 8px;
        padding: 16px;
    }
}

/* 工具层 */
@layer utilities {
    .mt-4 { margin-top: 16px; }
    .mb-4 { margin-bottom: 16px; }
    .text-center { text-align: center; }
}
```

### 6.2 第三方样式管理
```css
/* 导入第三方样式到特定层 */
@layer vendor {
    @import url('bootstrap.css');
    @import url('font-awesome.css');
}

/* 自定义样式在更高优先级层 */
@layer custom {
    .button {
        /* 覆盖 Bootstrap 样式 */
        background: #4a90e2;
    }
}

/* 工具类最高优先级 */
@layer utilities {
    .hidden { display: none; }
    .visible { display: block; }
}
```

---

## 七、CSS 性能优化

### 7.1 选择器性能
```css
/* 好的选择器 */
.button { }
.card .title { }
#header { }

/* 避免的选择器 */
* { }  /* 通用选择器 */
div > * > * { }  /* 深层嵌套 */
:not(.special) { }  /* 复杂否定 */

/* 使用类选择器代替标签选择器 */
.card { }  /* 好 */
div { }  /* 避免 */
```

### 7.2 渲染性能
```css
/* 使用 transform 代替 top/left */
.moving-element {
    /* 避免 */
    /* top: 100px; */
    /* left: 100px; */
    
    /* 推荐 */
    transform: translate(100px, 100px);
}

/* 使用 opacity 代替 visibility */
.fade-element {
    /* 避免 */
    /* visibility: hidden; */
    
    /* 推荐 */
    opacity: 0;
    transition: opacity 0.3s ease;
}

/* 使用 will-change 提示浏览器 */
.animated-element {
    will-change: transform, opacity;
}

/* 避免布局抖动 */
.stable-layout {
    /* 设置固定尺寸 */
    width: 200px;
    height: 100px;
    
    /* 或使用 aspect-ratio */
    aspect-ratio: 2 / 1;
}
```

---

## 八、学习资源

| 资源 | 说明 |
|------|------|
| MDN CSS 文档 | 完整参考 |
| CSS-Tricks | 技巧和教程 |
| Smashing Magazine | 最佳实践 |
| web.dev | 性能优化指南 |

---

*本资料由 AI 整理，建议结合实际项目练习*
