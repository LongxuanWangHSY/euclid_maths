---
name: euclids-gate-agenda
description: >
  欧几里德之门公众号未完成任务（Agenda）管理。追踪往期文章的课后思考题和预告专题，支持查询未完成任务、在新文章中附加上期思考题解答、标记任务完成。触发词：agenda、未完成任务、思考题解答、上期解答、课后习题、pending。当用户明确要求查看或处理 agenda、加入往期解答时使用。
---

# 欧几里德之门 Agenda 管理系统

## Agenda 结构

Agenda 存储在 article_index.json 的 `agenda` 字段中：

```json
{
  "agenda": {
    "pending_answers": [
      {"article_id": 1, "title": "鸽巢原理", "question_count": 3},
      {"article_id": 2, "title": "一道恒等式三把钥匙", "question_count": 3}
    ],
    "pending_topics": [
      "网格作图（grid construction）",
      "Lucas 定理：模素数下二项式系数的神奇规律"
    ]
  }
}
```

## 核心功能

### 1. 查询 Agenda

当用户问"有哪些未完成任务"或"agenda 里有什么"时：

- 读取 article_index.json
- 列出 pending_answers（按 article_id 排序）
- 列出 pending_topics（按添加顺序）
- 用表格形式展示

### 2. 在新文章中加入往期解答

**触发条件**：用户明确要求"调用 agenda"或"加入上期解答"或类似表述。其他情况不主动加入。

执行流程：

1. 读取 article_index.json
2. 查找 pending_answers 中 questions_answered 为 false 的最早文章
3. 读取该文章的完整内容，提取思考题
4. 在新文章开头（题目之前）增加「上期思考题解答」小节：

```
## 上期思考题解答

### 第X期「文章标题」思考题

**题1**：原题...
**解**：解答...

**题2**：原题...
**解**：解答...
```

5. 将该文章的 `questions_answered` 设为 `true`
6. 从 agenda.pending_answers 中移除该项
7. 更新 article_index.json

### 3. 直接解答往期思考题

当用户说"帮我解答第X期的思考题"：

1. 从 article_index.json 找到对应文章
2. 读取文章，提取思考题
3. 逐题给出完整解答
4. 将解答保存为独立文件：`NoX_思考题解答.md`
5. 更新 questions_answered 为 true，从 pending_answers 移除

### 4. Agenda 更新触发点

以下操作会自动更新 agenda：

| 操作 | Agenda 变化 |
|------|------------|
| 写新文章 | 新思考题 → pending_answers，新预告 → pending_topics |
| 解答往期题 | 对应项从 pending_answers 移除 |
| 按预告写完新文章 | 对应项从 pending_topics 移除 |

## 索引文件路径（固定，相对仓库根目录）

```
article_index.json
```
