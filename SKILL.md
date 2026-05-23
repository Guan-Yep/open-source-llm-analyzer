---
name: open-source-llm-analyzer
description: >
  Analyze open-source AI/LLM projects by deeply examining their codebase to identify all
  prompts, tool integrations, LLM API calls, and agent patterns. Generates a comprehensive
  Chinese-language analysis report (AI_MODEL_USAGE_ANALYSIS.md) with translated prompt
  documentation, Mermaid data-flow diagrams, and actionable insights. Use when the user
  provides a GitHub URL and asks to analyze, deconstruct, or study how a project uses LLMs
  -- for example "analyze this repo", "deconstruct how X uses LLMs", "extract all prompts".
---

# 开源项目大模型应用分析器

## 概述

深度分析任何使用大语言模型的开源 GitHub 仓库，系统性地识别项目中的所有提示词、
LLM API 调用、工具集成和 Agent 模式。产出完整的中文分析报告，包含提示词翻译文档、
Mermaid 数据流图和可落地的架构洞察。

## 工作流

分析分为五个顺序阶段，必须逐一完成，不可跳过。

### 阶段 1：克隆仓库

将 GitHub 仓库克隆到本地工作目录：

```bash
git clone {REPO_URL} ai_analysis/{REPO_NAME}
```

克隆完成后，向用户汇报项目名称和大致规模。

### 阶段 2：探索项目结构

全面扫描项目目录结构以理解其架构：

1. 使用 Glob 工具枚举完整目录树
2. 识别主要编程语言和框架
3. 定位关键目录：源码目录 (src/, lib/, skills/)、提示词目录 (prompts/, templates/)、配置目录 (config/, .github/)
4. 阅读 README.md、AGENTS.md 及顶层文档，理解项目目的

向用户汇报：编程语言、框架、关键目录、大致文件数量。

### 阶段 3：识别和提取所有提示词

严格按照 `references/workflow.md` 中定义的四类方法执行，**不可跳过任何一种**：

- **方法 1 — 文件名模式匹配**：Glob 搜索 `*prompt*.md`、`*template*.md`、`*instruction*.md`、`system*.txt`
- **方法 2 — 代码变量模式**：Grep 搜索 `_PROMPT`、`system_prompt`、`user_prompt`、`instruction`
- **方法 3 — API 调用特征**：Grep 搜索 `openai`、`anthropic`、`google`、`xai`、`gemini`、`chat.completions`、`generateContent`、`messages.create`、`openrouter`
- **方法 4 — 配置文件检查**：读取 `config.yaml`、`pyproject.toml`、`SKILL.md`、`.env`、`settings.py`

对每个发现的提示词：
- 读取其完整内容和前后各 20 行代码上下文
- 记录精确的文件路径、行号、变量名和完整英文原文
- 追踪导入链和调用链，理解提示词在代码中的流转路径

此外，使用 Task 工具启动 search 子 Agent 进行全代码库的交叉搜索，利用 `references/workflow.md`
中的详细搜索模式确保不遗漏散布在代码各处的提示词。

如果未发现任何提示词，清晰报告此结果，并直接在阶段 5 生成一份标注"未发现"的报告。

### 阶段 4：文档化所有提示词（翻译）

对阶段 3 发现的每个提示词，严格按 `references/templates.md` 中的模板一创建翻译文档：

1. 创建输出目录：`ai_analysis/translated_prompts/`
2. 为每个提示词编写 `translated_prompts/{功能描述}_prompt_zh.md`
3. 每个文档必须包含：元信息、中文翻译（保留原始格式）、关键参数、相关代码上下文
4. 按 `references/templates.md` 中的模板二创建 `translated_prompts/INDEX.md`

翻译质量要求：
- 保持技术术语的准确性
- 保留原始格式（Markdown、JSON 结构、代码块）
- 占位符 `{variable}` 在翻译中原样保留
- 对特殊概念添加必要的译注
- 原始提示词若已是中文，仅标注位置并跳过翻译

### 阶段 5：生成分析报告

严格按 `references/templates.md` 中的模板三创建 `ai_analysis/AI_MODEL_USAGE_ANALYSIS.md`。报告必须包含以下七个部分：

1. **项目概述** — 从 README 和代码中提取的项目名称、描述、功能和完整技术栈
2. **项目逻辑或数据流分析** — Mermaid `sequenceDiagram` 时序图，展示 LLM 在项目架构中的参与方式。
   使用 `rect rgb(...)` 彩色分区和 `autonumber` 自动编号。附每个阶段的文字说明。
3. **提示词分类统计** — 按类别对每个提示词进行分类统计的表格（系统提示词、用户提示词、评分提示词、安全声明等）
4. **大模型应用场景分析** — 每个独立 LLM 调用的：触发条件、使用的提示词（链接到翻译文档）、代码位置、输入/输出格式、作用和价值
5. **提示词工程技术** — 识别使用了哪些技术：Few-shot少样本学习、Chain-of-Thought思维链、
   Role-playing角色扮演、Constrained Decoding受限解码、Prompt Injection Defense提示注入防御、
   Intent-based Dynamic Prompting基于意图的动态提示、Structured Output Control结构化输出控制。
   说明是否使用模板引擎和动态生成。
6. **上下文构建** — 若项目是 LLM Agent（LLM makes the loop, LLM in the loop, LLM ends the loop），
   绘制 Agent 循环机制图。若项目为 LLM 提供了 Tools/Function Calling，
   列出每个工具的 JSON Schema、中文翻译后的 description，并解释每个工具在什么场景下提供什么上下文价值。
7. **关键发现与洞察** — 三个子部分：
   - 值得学习的提示词设计
   - 可能的优化空间
   - 总结：项目的大模型应用特色

## 设计规范

- **报告语言**：所有生成的文档使用中文。代码引用和文件路径保持原文。
- **输出目录**：所有产物放在克隆项目根目录下的 `ai_analysis/` 文件夹中。
- **进度汇报**：每完成一个阶段，向用户简要汇报进度。
- **不确定性标注**：无法确认的内容标注为 `[待确认]` 并说明原因。
- **特殊情况处理**：
  - 多语言提示词：已为中文的提示词，标注位置并跳过翻译
  - 加密/混淆内容：标注为 `[无法解析]` 并记录位置
  - 动态生成的提示词：分析生成逻辑，提取模板部分
  - 超大提示词：分段翻译，保持结构完整性

## 附带资源

### references/workflow.md
详细搜索策略文档，包含全部四种识别方法的完整模式表。执行阶段 3 时参考此文件以确保不遗漏任何搜索模式。

### references/templates.md
完整的输出模板集合——提示词翻译文档模板（模板一）、提示词索引模板（模板二）、
最终分析报告模板（模板三）。执行阶段 4 和阶段 5 时参考此文件以确保输出一致性。
