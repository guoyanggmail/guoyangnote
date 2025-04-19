好的，这是根据您提供的 YouTube 视频文字稿和 AI 摘要生成的 Markdown 格式文章，已翻译成简体中文：

# 🤖 MCP 是如何对接大模型的？抓取 AI 提示词，拆解 MCP 的底层原理

大家好，我是技术爬爬虾！今天我们来深入探讨 MCP（模型上下文协议）是如何与大型语言模型（LLM）对接的。

## 🎬 前言

![](http://i0.hdslb.com/bfs/vchapter/28916190381_41.jpg)MCP 简单来说就是 AI 大模型的标准化工具箱。那么，AI 大模型是如何知道工具箱里有哪些工具，以及如何使用参数进行调用的呢？MCP 与 Function Call 的关系是什么？是不是只有支持 Function Call 的模型才能使用 MCP？

本期视频，我们将从大模型与提示词的角度，再次探讨 MCP 协议的底层原理。我使用 Cloudflare 的 AI Gateway 设计了一个小实验，抓取了与 AI 交互的所有请求数据，来验证一下视频开头提出的这些问题。在开始之前，建议大家先看一下上期视频，了解一下 MCP 的基础概念。本期视频节奏比较快，我们直接进入实战环节。

首先，我们来到 Cloudflare 的首页，没有账号的话先注册一个。在左侧菜单点击 AI Gateway，这是 Cloudflare 推出的一个大模型 API 的代理功能。配置好以后，所有的大模型 API 会先流经 Cloudflare，然后再到达 API 提供商。

## 🔍 提示词抓取

![](http://i0.hdslb.com/bfs/vchapter/28916190381_174.jpg)在这个过程中，Cloudflare 会为我们记录日志。这里点击 "Create Gateway"，随便起个名字，这里我叫 "take mp 技术啪啪虾"，最下面点击创建。点击右上角的 "AI 平台"，这里选择 "Open Router"。

接下来，我们来到 VS Code，下载并安装 "cl" 这款 AI 编程插件。来到它的设置，API 提供商这里我选择 "OpenAI compatible"，这里的 Base URL，我们就把 Cloudflare 这个链接复制过来。API Key 我们要去 Open Router 申请一下。在 Open Router 网站右上角找到 "Keys"，然后把这个 API Key 复制下来，粘贴到 client 里面。模型 ID 这里我还是填写免费的 "deepseek-v2-chat"。

接下来，我们来添加 MCP server。我们点击这个 MCP server 的小按钮，在市场这里我搜索 "time"，这次我要安装这个跟时间有关的 MCP server。MCP server 本质上就是运行在电脑上的一段程序。"time" 这个 MCP server 是使用 Python 编写的，所以首先我们要在电脑上安装好 Python 的运行环境。这里点击 "install"，接下来 AI 就会自动帮我们安装好这个 MCP server。第一步，创建文件夹。第二步，使用 pip 命令安装这个 Python 包。第三步，创建好了配置文件，点击 "save"。这里我要修改一下这个配置文件，在参数后面加上我本地的时区，保存。最后，为了验证一下，这里点击 "approve"，好，任务完成！这个与时间有关的 MCP server 就配置好了。有关配置 MCP server 的详细步骤，上期视频已经详细演示过了。

看到这个 server 里面有以下两个工具，一个是获取当前时间，再一个是进行时间转换。

现在我们的问题是，AI 是怎么知道有一个叫做 "time" 的 MCP server，并且它有两个工具可以使用的？

这里我新开一个对话，我们来测试一下，我问它 "现在的时间是什么？" 这里显示 "cl" 要调用这个 "get current time" 的 MCP 工具。我们回到 Cloudflare 这里看一下日志。这里有请求与响应两部分，请求的内容比较多，我先把它复制下来，在 VS Code 里面新建一个文件，我把它粘贴过来。

## 📝 日志拆解

![](http://i0.hdslb.com/bfs/vchapter/28916190381_332.jpg)这个请求的数据主要分成了两个部分。首先是 "system"，也就是系统提示词。然后下面是 "user"，也就是用户输入的问题，包括一些关于 VS Code 的环境变量。我们重点来看这个系统提示词。

我们看到这一段："MCP server 模型上下文协议允许系统与本地运行的 MCP server 之间进行通信，这个通信可以带来额外的工具以及资源来扩展你的能力，可以使用 `use_mcp` 来执行 MCP 协议或者 `access_mcp_resource` 来获取资源。" 这里又说可以使用的工具有 "get_current_time"，就是获取当前时间这个工具的传参格式，还有 "convert_time" 用来进行时间转换这个工具的传参格式。

这里可以看到有哪些 MCP 工具，还有如何调用，"cl" 直接把它写到了系统提示词里面，并且发送给了 AI。这样也解释了为什么 MCP 的通用性这么强，可以兼容市面上几乎所有的大模型。因为它不需要大模型有什么额外的能力，只要能看懂提示词就能使用 MCP。

我们看完了提示词，再看看响应部分。这里我把响应粘贴回来，我们看到 DeepSeek 给出的响应，它非常的干净利落，直接就是使用 MCP 工具。MCP server 的名字是这一串，它要用的工具是 "get_current_time"，然后 arguments 里面是传递的参数，然后 "cl" 就可以根据大模型的指令调用对应的 MCP server 获取当前时间。这里我点击 "approve"，完成这次 MCP 调用。看到左侧打印了 MCP 的调用结果，也就是当前时间。接下来 "cl" 又调用了一次大模型。

我们再通过日志看一下，这是最新的一次请求，在这次跟大模型的请求里，附上了用户的原始提问，还有 MCP 调用过程，然后大模型进行了最后的整理，返回了输出。也就是当前时间是 9 点 53 分，时区为上海。

我们再来捋一下 MCP 的调用过程：首先，"cl" 把用户提问加 MCP 的详细使用方法传递给大模型，其中 MCP 的详细使用方法是直接附加在系统提示词里面的。接下来，大模型决定使用哪个 MCP 工具以及如何传递参数。"cl" 根据大模型决定的调用方式来调用 MCP server。上期视频里说过，MCP server 的本质就是运行在本地电脑上的一个 Node.js 或者 Python 程序。"cl" 调用完 MCP server 以后拿到返回结果，加上之前的上下文再次传递给大模型，大模型进行最后的整理，然后把最终的结果呈现给用户。

接下来我们再做一个实验，MCP 与 Function Call 的关系是什么？众所周知，DeepSeek-V1 是不支持 Function Call 的。我们在 GitHub 的 issue 里可以看到，开发者们也是纷纷留言表达了这一诉求。根据我前面视频的分析，MCP 的相关知识是直接附加到系统提示词里面的。

## 🤝 与 Function Call 的关系

所以模型只要能读懂提示词，它就应该能使用 MCP，跟是否具备 Function Call 功能是没有关系的。

这里我来到 DeepSeek 的 API 开放平台，找到 API Keys，我创建一个 API Key。之前有一个完整视频介绍关于 DeepSeek API 的使用，这里我就不赘述了。我们把这套信息填写到 client 里面，API 提供商选择 "OpenAI compatible"，Base URL 我们填写这个 DeepSeek 官网的这个地址，API Key 填写刚才申请的那个，下面的模型名称，我们要填写这个 DeepSeek 的，也就是 "deepseek-v1"。好，保存一下。

还是刚才的问题，我们来测试一下 DeepSeek-V1 能不能使用 MCP。这里显示了模型的推理过程，看到调用了 MCP 工具，我点击 "approve"，这里成功给出了正确答案。

从这个例子我们就得到了验证，MCP 与 Function Call 只是功能相似，它们之间是相互独立的，没有依赖关系。MCP 的最大优点是整合了之前各家大模型不同的 Function Call 的标准，整合成了一个统一的标准协议。而且不仅仅是 Cloudflare，市面上几乎所有的大模型都可以接入 MCP。

今天的视频就到这里。希望大家通过这两个实验对 MCP 协议的底层原理有更多的了解。下面的视频我会介绍一系列好用的 MCP 工具。

感谢大家，我们下期再见！

## 💡 总结

本视频深入探讨了 MCP（模型上下文协议）与大型语言模型（LLM）的对接原理。通过 Cloudflare AI Gateway 的实验，作者抓取了 AI 交互的请求数据，验证了 MCP 如何通过提示词（Prompt）与 LLM 通信，以及 MCP 与 Function Call 的关系。视频还演示了即使不支持 Function Call 的 LLM（如 DeepSeek-V1）也能使用 MCP，强调了 MCP 的通用性和标准化优势。

## ✨ 亮点

- 💡 MCP 本质上是 AI 大模型的标准化工具箱，通过系统提示词告知 LLM 可用工具及其调用方式。#AI工具箱 #标准化协议 #MCP原理
- ⚙️ MCP 与 Function Call 功能相似但相互独立，模型只需能理解提示词即可使用 MCP，无需具备 Function Call 功能。#FunctionCall #独立性 #提示词
- ☁️ Cloudflare AI Gateway 可用于抓取 AI 交互数据，帮助理解 LLM 如何解析和使用 MCP 提供的工具信息。#AIGateway #数据抓取 #实验验证
- 🧩 MCP 通过将工具信息嵌入系统提示词，实现了与各种 LLM 的兼容，无需模型具备特定能力。#兼容性 #系统提示词 #通用协议
- ⏱️ 实验演示了 DeepSeek-V1 成功调用 MCP 工具，验证了 MCP 的通用性和与 Function Call 的独立性。#DeepSeek #实验验证 #独立性验证

## 🤔 问题

- MCP 工具箱中工具的安全性如何保障？
- 如何自定义和扩展 MCP 工具，以满足特定应用场景的需求？