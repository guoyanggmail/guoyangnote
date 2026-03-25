# 郭杨的技术笔记库 📚

> 10+ 年 Android 开发的知识沉淀 —— 技术学习、架构思考、求职经验、工具探索

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-在线访问-blue?logo=github)](https://guoyanggmail.github.io/guoyangnote/)

## 🚀 在线访问

📌 **[https://guoyanggmail.github.io/guoyangnote/](https://guoyanggmail.github.io/guoyangnote/)**

## 🛠️ 本地运行（基于 Docsify）

本项目使用 [Docsify](https://docsify.js.org/) 构建，**无需编译**，所有笔记均为 Markdown 文件，在浏览器端实时渲染。

### 1. 安装 docsify-cli

```bash
npm i docsify-cli -g
```

### 2. 克隆项目

```bash
git clone https://github.com/guoyanggmail/guoyangnote.git
cd guoyangnote
```

### 3. 启动本地预览

```bash
docsify serve docs
```

访问 [http://localhost:3000](http://localhost:3000) 即可预览效果。

## ✏️ 如何添加新笔记

1. 在 `docs/` 对应分类目录下新建 `.md` 文件，例如：
   ```
   docs/技术学习/Android/自定义View原理.md
   ```

2. 在 `docs/_sidebar.md` 中添加对应链接：
   ```markdown
   - 技术学习
     - Android
       - [自定义View原理](/技术学习/Android/自定义View原理)
   ```

3. （可选）在 `docs/_navbar.md` 顶部导航中也加入链接。

4. 刷新浏览器即可看到新内容，**无需重启服务**。

## 📁 项目结构

```
guoyangnote/
├── README.md              # 本文件（GitHub 项目介绍）
└── docs/                  # 网站根目录
    ├── index.html         # Docsify 配置入口
    ├── README.md          # 网站首页
    ├── _sidebar.md        # 左侧导航
    ├── _navbar.md         # 顶部导航
    ├── 求职相关/          # 面试题、简历等
    └── 模板工具/          # 提示词模板等
```

## 📬 联系我

- GitHub：[@guoyanggmail](https://github.com/guoyanggmail)

---

> 笔记持续更新中，⭐ Star 收藏，欢迎关注！
