# 开源项目大模型应用分析 — 详细工作流参考

## 整体流程

```mermaid
flowchart TD
    A[接收 GitHub 仓库 URL] --> B[Step 1: git clone 到本地]
    B --> C[Step 2: 项目结构探索]
    C --> D[Step 3: 提示词识别与提取]
    D --> E{发现提示词?}
    E -->|是| F[Step 4: 提示词文档化]
    E -->|否| G[标注"未发现提示词"]
    F --> H[Step 5: 生成分析报告]
    G --> H
    H --> I[输出 ai_analysis/ 目录]
```

---

## 第一阶段：项目结构探索

1. 使用 Glob 工具扫描整个项目目录结构
2. 识别项目的主要编程语言和框架
3. 定位关键目录：src/、prompts/、templates/、config/、lib/ 等

### 输出要求
完成后汇报：编程语言、框架、关键目录、项目规模（文件数/测试数）。

---

## 第二阶段：提示词识别与提取

按以下四类方法逐一执行，**不可跳过任何一个方法**。

### 方法 1：文件名模式匹配

搜索以下文件名模式（使用 Glob）：

| 模式 | 说明 |
|------|------|
| `*prompt*.md` | Markdown 格式的提示词文件 |
| `*prompt*.txt` | 纯文本格式的提示词文件 |
| `*prompts*.md` / `*prompts*.txt` | 复数形式 |
| `*template*.md` | 模板文件 |
| `*instruction*.md` | 指令文件 |
| `system*.txt` | 系统提示词文件 |

### 方法 2：代码变量搜索

在所有代码文件中搜索以下模式（使用 Grep）：

| 搜索模式 | 说明 |
|----------|------|
| `*_prompt =` | Python 变量赋值 |
| `*_prompts =` | 复数形式 |
| `*Prompt =` | CamelCase 变量 |
| `PROMPT_*` | 常量大写前缀 |
| `system_prompt` | 系统提示词变量 |
| `user_prompt` | 用户提示词变量 |
| `instruction` | 通用指令变量 |

### 方法 3：API 调用特征搜索

搜索大模型 API 调用关键词：

| 搜索模式 | 对应提供商 |
|----------|-----------|
| `openai.ChatCompletion` / `chat.completions` | OpenAI |
| `anthropic.messages` | Anthropic |
| `messages.create` | 通用 Chat API |
| `generateContent` / `generativelanguage` | Google Gemini |
| `api.x.ai` / `x_search` | xAI / Grok |
| `openrouter.ai` | OpenRouter |
| `prompt=` / `system=` / `messages=[` | 通用调用模式 |

### 方法 4：配置文件检查

检查以下配置文件中的 LLM 相关配置：

- `config.yaml` / `config.yml`
- `config.json` / `pyproject.toml`
- `SKILL.md`（Agent Skills 项目特有）
- `.env` 文件 / `settings.py`
- `gemini-extension.json` / `openai.yaml`
- `package.json`（Node.js 项目的 LLM 依赖）

### 发现后的处理

对每个发现的提示词：
1. 读取完整内容（代码文件需读取前后 20 行上下文）
2. 记录：文件路径、行号、变量名、完整英文原文
3. 追踪导入和引用关系，确定调用链路

---

## 第三阶段：提示词文档化

### 翻译文档结构

为每个提示词创建翻译文档，文件命名：`translated_prompts/[功能描述]_prompt_zh.md`

```markdown
# 提示词翻译文档：[功能描述]

## 元信息
- 原文件位置: `path/to/file.py:line`
- 变量名称: `variable_name`
- 功能模块: [所属模块]
- 调用场景: [触发条件和用途]

## 中文翻译

[逐行翻译，保留 JSON/Markdown 等原始格式]

## 关键参数
- `{param}` - 参数说明
- `{param2}` - 参数说明

## 相关代码上下文
[该提示词在代码中的使用方式、调用链、错误处理]
```

### 索引文件

在 `translated_prompts/INDEX.md` 中维护索引表：

```markdown
| 序号 | 文档 | 原文件 | 功能描述 |
|------|------|--------|----------|
```

---

## 第四阶段：生成分析报告

创建 `AI_MODEL_USAGE_ANALYSIS.md`，包含以下 7 个部分（参见 `references/templates.md` 获取完整模板）：

1. **项目概述** — 名称、描述、功能、技术栈
2. **数据流分析** — Mermaid 时序图 + 分阶段说明
3. **提示词分类统计** — 按类别统计表格
4. **大模型应用场景分析** — 每个场景的触发条件、输入输出、作用
5. **提示词工程技术** — Few-shot/CoT/Role-playing 等
6. **上下文构建** — Agent 循环机制 + 工具定义
7. **关键发现与洞察** — 值得学习的设计 + 优化空间
