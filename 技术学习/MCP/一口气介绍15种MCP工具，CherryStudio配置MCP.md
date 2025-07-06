# 🚀 一口气介绍15种MCP工具：CherryStudio配置全攻略

## 🌟 前言
MCP（模型控制协议）是当前AI领域的热点技术，每天都有大量新工具涌现。本文基于技术爬爬虾的实战视频，精选15款实用MCP工具，手把手教你用CherryStudio和Cline配置使用！

## 🔧 基础配置
### 1. 客户端选择
- 推荐两款**完全开源免费**的AI客户端：
  - Cherry Studio（使用Function Call功能）
  - Clan（使用系统提示词）

### 2. 模型配置
```json
{
  "推荐模型": "Claude 3.7（OpenRouter）",
  "备选方案": "DeepSeek-V3（成本更低）",
  "关键点": "必须支持Function Call功能"
}
```

### 3. 环境准备

- 手动安装必备组件：
  
  - Bun（类似Node.js的运行环境）
  - UV（Python运行环境）
- 注意：Windows系统需特殊处理路径（使用双反斜杠`\\`）

## 🛠️ 15种MCP工具详解

### 1️⃣ 🌐 网页内容抓取（Server Fetch）

- **功能**：让AI获取网页内容并总结
- **配置关键**：

  ```bash
  npx uvicorn server_fetch:app
  ```

### 2️⃣ 🗺️ 百度地图服务

- **特色功能**：
  
  - 行程规划（含地铁换乘建议）
  - 周边POI搜索
- **API申请**：需在[百度地图开放平台](https://lbsyun.baidu.com/)创建应用

### 3️⃣ 🤖 浏览器自动化（Playwright）

- **惊艳表现**：
  
  - 自动填写登录表单
  - 点击页面元素
- **前置要求**：

  ```bash
  npm install playwright
  ```

### 4️⃣ 💬 Slack机器人

- **配置流程**：
  
  1. 创建Slack App
  2. 添加`chat:write`等权限
  3. 获取`xoxb-`开头的Bot Token

### 5️⃣ 🔍 联网搜索（Brasearch）

- **突破限制**：让大模型获取实时网络信息
- **替代方案**：GoSearch（配置更简单）

### 6️⃣ 🧠 知识图谱

- **核心概念**：
  
  - 实体（Entities）
  - 关系（Relationships）
- **数据存储**：自动生成`memory.json`文件

### 7️⃣ 📂 文件系统操作

- **安全提示**：
  
  - 限制AI可访问的目录
  - Windows路径示例：

    ```json
    "path": "C:\\Users\\Public\\Documents"
    ```

### 8️⃣ ⌨️ 命令行执行（Desktop Commander）

- **实测功能**：
  
  - 创建文件
  - 写入内容
  - 检查系统信息

### 9️⃣ 🤔 思维链增强（Screen Thinking）

- **神奇效果**：将普通模型升级为推理模型

- **示例问答**：
  

  > Q: "strawberry有几个r？"\
  > A: 通过分步思考得出正确答案（3个r）

### 🔟 ☁️ Cloudflare集成

- **支持操作**：
  
  - KV存储
  - R2数据桶
  - D1数据库
- **认证方式**：OAuth自动授权流程

### 1️⃣1️⃣ 🗃️ PostgreSQL连接

- **安全设计**：默认只读权限
- **查询示例**：

  ```sql
  SELECT * FROM public.tags
  ```

### 1️⃣2️⃣ ⚙️ GitHub操作

- **必备Token权限**：
  
  - repo（所有仓库权限）
  - workflow（工作流权限）

### 1️⃣3️⃣ 🌐 浏览器实时分析（Brotli）

- **高阶应用**：解析网络请求面板
- **配置教程**：参考作者往期视频

### 1️⃣4️⃣ 🖥️ Git仓库管理

- **支持命令**：
  
  - 创建分支
  - 提交更改
  - 查看历史
- **注意**：需手动处理git push

### 1️⃣5️⃣ 📹 YouTube总结

- **依赖工具**：yt-dlp
- **工作流程**：
  
  1. 获取字幕
  2. AI生成摘要

## 💡 进阶技巧

- **组合使用**：例如地图+Slack实现自动行程通知
- **配置文件共享**：同一配置可在不同客户端通用
- **错误排查**：
  
  - 检查UV/Bun是否安装正确
  - 验证API密钥权限

## 🎯 下期预告

- 复杂MCP服务器的深度配置
- AI智能体开发实战
- 更多行业专用工具解析

> 📌 本文所有工具均来自开源项目[Awesome MCP](https://github.com/awesome-mcp)，持续更新中...