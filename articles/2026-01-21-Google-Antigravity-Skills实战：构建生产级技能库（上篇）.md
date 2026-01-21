---
title: "Google Antigravity Skills实战：构建生产级技能库（上篇）"
author: "AI智汇派"
date: "2026-01-21"
categories:
  - 微信公众号
tags:
  - AI智汇派
---

在 Google Antigravity 这样的 Agent 优先（Agent-first）开发平台中，Gemini 1.5/2.0 等模型已经拥有了超过 100 万 Token 的上下文窗口。表面上看，我们可以把整个代码库、所有的文档和工具库一股脑塞进去。

但理想很丰满、现实很骨感。在工程实践中，这种“暴力加注”导致了两个致命问题：

1.  1. **上下文饱和（Context Saturation）与推理幻觉（Context Rot）**：当无关信息充斥上下文，模型的推理精度会迅速下降。模型会因为“噪音”太多而产生幻觉，甚至在关键决策时忘记了最初的指令。
    
2.  2. **经济性与延迟（Latency Overhead）**：每一轮对话都在发送数万个冗余 Token，不仅烧钱，更带来了无法忍受的响应延迟。
    

**这就是我们要讨论的核心：Agent Skills（代理技能）。** 它不是一种简单的扩展，而是一场关于“注意力管理”的范式演进。

* * *

### **二、 工具膨胀危机：量化你的 Agent 负担**

在目前的 Agentic 模式中，我们习惯于通过 MCP（模型上下文协议）加载各种工具服务器。让我们算一笔账：

*   • **GitHub MCP Server**：提供 50 个工具。
    
*   • **Playwright MCP Server**：提供 24 个工具。
    
*   • **Chrome DevTools MCP Server**：提供 26 个工具。
    

当我们要求 Agent “重构身份验证中间件”时，Agent 只需要深度的安全协议知识和文件读写能力。而此时的 Agent 上下文中正堆载着 100 多个不相关的工具定义（约 40K-50K Token）。

我们发送了一个简单的请求，却让 Agent 带着沉重的“行李”在泥潭里爬行。**工具膨胀（Tool Bloat）正在成为 Agent 落地生产环境的最大障碍。**

* * *

### **三、从单体上下文到渐进式披露**

为了破解这个困局，Anthropic 提出了 **Agent Skills** 标准，并由 Google Antigravity 进行了深度实现。

Skills 的核心思想是 **渐进式披露（Progressive Disclosure）**：

*   • **按需装备**：与其让模型在会话开始时就“背诵”所有能力，不如给它一个轻量级的“技能清单”（元数据）。
    
*   • **语义触发**：模型在初始状态只加载轻量级的技能描述（Description）。只有当用户的意图（Intent）匹配到特定技能时，Agent 才会动态地将该技能背后的“重型”指令和脚本加载进当前的上下文窗口。
    

**Skills 实际上将通才的 Gemini 变成了特种兵**。它让组织能够将安全检查、代码风格、部署步骤等最佳实践封装为可执行资产，由 AI 严格遵循。

* * *

### **四、 深度矩阵：Skills、MCP、Rules 与 Workflows**

在 Antigravity 架构中，区分这四个概念是架构师的基本功。它们分别解决了不同维度的约束问题：

| 
特性

 | **Skills (技能)** | **MCP (工具服务器)** | **Rules (规则)** | **Workflows (工作流)** |
| --- | --- | --- | --- | --- |
| **触发机制** | **Agent 自主识别**

并触发

 | 

持续挂载，由 Agent 调用

 | **永远开启**

（被动约束）

 | **用户手动**

显式触发

 |
| **架构** | 

基于文件、无服务器 (Serverless)

 | 

客户端-服务器模式

 | 

注入系统提示词

 | 

脚本化的宏命令

 |
| **生命周期** | 

任务级（执行完即释放）

 | 

会话级（持久连接）

 | 

永久（护栏作用）

 | 

瞬时（触发执行）

 |
| **最佳用途** | 

临时工程任务（如 Schema 验证）

 | 

状态化连接（如 DB/Slack）

 | 

规范约束（如 TypeScript 风格）

 | 

复杂序列自动化（如 /deploy）

 |

在 Antigravity 架构中，精准定义这几个的边界至关重要：

1.  1. **Skills vs. MCP**：
    

*   • **MCP (Model Context Protocol)**：重型武器。采用客户端-服务器架构，适合持久连接（如数据库、Slack 工作区）。
    
*   • **Skills**：特种兵。基于文件、无服务器（Serverless）、易失性。任务完成即释放，适合“生成变更日志”或“运行特定测试集”。
    
*   • **MCP 是“手”**（提供 `read_file` 等底层函数），**Skills 是“脑”**（提供方法论，告诉 Agent 什么时候、如何用手）。
    

3.  2. **Skills vs. Rules (`.agent/rules/`)**：
    

*   • **Rules** 是**被动约束**。它是“永远开启”的护栏（如“永远使用 TypeScript 严格模式”）。
    
*   • **Skills** 是**自主触发**。用户说出目标，Agent 的推理引擎识别匹配后自行“装备”该技能。
    

* * *

### **五、 技能的未来： codifying（代码化）最佳实践**

Skills 最大的价值在于它让“最佳实践”可执行化。  
想象一下，你的同事写了一套防 SQL 注入的规则。以前，这只是 Wiki 里的一页文档；现在，它可以是一个 **Database-Validator Skill**。当 Agent 想要写数据库代码时，它必须启用这个 Skill，调用里面的 Python 脚本进行校验。

**这种“模型推理 + 确定性脚本”的组合，才是构建高可靠 Agent 系统的唯一出路。**

* * *

### 本篇小结

我们剖析了百万级上下文背后的隐忧：工具膨胀与注意力弥散。Agent Skills 通过“轻量化元数据 + 动态加载指令”的架构，实现了性能与精度的平衡。它是 Agent 从“玩具”走向“工程化”的关键一步。

**【下篇预告】**  
原理听起来很硬核，代码怎么写？在下一篇中，我们将进入**实战进阶篇**。我会通过 5 个由浅入深的实战 Level，带你从零编写 `SKILL.md`，并展示如何利用 Python 脚本、Few-shot 示例和资源文件，整出一个具备架构师水准的Agent。

  

var first\_sceen\_\_time = (+new Date()); if ("" == 1 && document.getElementById('js\_content')) { document.getElementById('js\_content').addEventListener("selectstart",function(e){ e.preventDefault(); }); }

预览时标签不可点