# 欧几里德之门 — 项目规范

## 项目简介

数学公众号「欧几里德之门」的内容仓库。所有文章为中文 Markdown，遵循项目写作技能模板。

## 编码节奏（必须遵守）

**每次写完代码后，主动执行以下流程，不等用户提醒：**

1. `/code-review` — 查 bug、逻辑错误、安全性
2. `/simplify` — 消除重复、精简结构
3. `/verify` — 实际跑一遍确认生效
4. 提示用户 commit

**复杂任务时主动调用 agent 并行审查**（如让代码审查员 + 安全工程师同时查），调用前检查是否需要 `agent-manager install` 安装对应角色。

**跨会话的重要决策、用户偏好、项目约定，写入 Claude Code 持久记忆**（`memory/MEMORY.md`，本会话自动加载），下次对话前先查看已加载的记忆。

## 写作规范

### 文章文件
- 文件名格式：`No{序号}_{中文标题}.md`（如 `No15_非构造性证明.md`）
- 存放目录：No 系列长文 → `articles/long/`；S 系列短文 → `articles/short/`；特辑、公告与独立文章 → `articles/other/`
- 配图统一放 `generated_images/`，文章内以相对路径引用
- 数学公式用 LaTeX：`$...$` 行内、`$$...$$` 独立行
- 文章分类参照 `.claude/skills/euclids-gate-math-writing/SKILL.md` 模板

### 语言
- 正文：中文
- 数学术语：中文为主，首次出现可附英文原文
- 代码/命令：英文

## AI 配置工作流

### Agent 管理
```bash
agent-manager search <关键词>     # 搜索可用 agent
agent-manager install <文件名>    # 按需安装
agent-manager remove <文件名>     # 用完卸载
agent-manager list                # 查看已安装
agent-manager core                # 还原核心 8 个
```
源仓库：`~/agency-agents/agency-agents-zh/`（中文版 266 个）、`~/agency-agents/agency-agents-upstream/`（英文版 233 个）

### MCP 服务器
配置文件：`.claude/mcp.json`
- `engram` — 本地记忆，跨会话保留决策和偏好
- `skillet` — 技能包管理器
- `mcpollinations` — AI 图片生成
- `md2card` — Markdown 转卡片
- `moda-image` — 魔搭图片生成
- `deepseek` — DeepSeek API 桥接（代码生成/审查/解释）
- `doubao` — 豆包 Seedream API 桥接（文生图/图生图/编辑）

### 多模型协作工作流

**模型分工：**
- **DeepSeek** — 代码生成、审查、解释、提示词工程
- **豆包 Seedream** — 图片生成（数学插图、示意图、可视化）
- **Claude（调度员）** — 理解上下文、路由任务、整合结果

**生图标准流程（两步法）：**
1. `deepseek_compose_image_prompt` — 把上下文提炼成精准的图片提示词（150-400 字，中文）
2. `doubao_text2image` — 用上一步的提示词生成图片

> 为什么？生图模型没有对话历史，需要 Claude + DeepSeek 把上下文「翻译」成它能理解的完整描述。这就像给设计师写 brief——不能只说「画个函数图」，要说清楚坐标范围、关键点、配色、标注、风格。

## Git 规范

- 提交信息用约定式提交（Conventional Commits）格式：`type: 中文描述`
- 常用 type：`feat` / `fix` / `cleanup` / `docs` / `refactor`
- 不要在提交中包含 API key 等敏感信息

## 跨 harness 交接约定

本仓库会被不同 AI harness（Claude Code / DeepSeek Harness 等）交替开发。各 harness 的对话记录彼此**可读但不互通**——都以本地明文落盘（Claude Code：`~/.claude/projects/<slug>/*.jsonl`；DeepSeek Harness：`~/.dsh/sessions/<slug>/*/session.jsonl.zstd`，zstd 压缩、未加密），按项目目录索引。**无缝衔接不靠读对方聊天记录，靠把状态写进仓库文件。**

### 切走前（落盘，缺一不可）
1. `devlog/` 补一条本次会话的决策记录 + 未完成项（尤其跨会话重要决策与用户偏好）
2. 把新约定固化进 `.claude/skills/*/SKILL.md` 或本文件，别只留在对话里
3. `article_index.json` 与磁盘对齐（文章、agenda、pending_topics 无漂移）
4. `git status` 确认无未提交关键改动；重大更新 commit（约定式提交）
5. 禁止把 API key 写进任何会被提交的文件或对话转写；密钥只放环境变量 / `settings.local.json`（已 ignore）

### 切回来（重建上下文，按顺序）
1. 读本文件 CLAUDE.md（项目规范 + 本约定）
2. 读 `devlog/` 最近一条 + `memory/MEMORY.md`（如存在）对齐上次状态
3. 读 `article_index.json` 的 agenda（pending_topics / pending_answers）确定队列
4. 读 `.claude/skills/` 六个 SKILL.md 加载写作工作流（写作任务前必读）
5. 最后 `git log` 补细节——对话原文仅作有损日志，不作事实依据

### 风险提示
- 对话转写是**明文**（未加密），任何能读用户目录的程序都可能窃取；切换 harness 的「可读性」即「可窃取性」
- 同一时间只用一个 harness 打开本目录，避免并发覆盖
- 不同 harness 的工具名/上下文系统不同（如 Claude Code 的 Write / agent-manager / engram 在 DeepSeek Harness 中不存在），切换后先确认工具映射再动手
