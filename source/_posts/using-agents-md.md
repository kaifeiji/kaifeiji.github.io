---
title: AGENTS.md
date: 2026-03-18 10:46:18
categories:
- AI
tags:
- AGENTS.md
---

在企业级私有代码库中，AI Agent（如 Claude Code, Codex, Windsurf 等）正逐渐从简单的辅助工具演变为具备自主执行能力的协同者。然而，由于私有库缺乏开源社区的公开语料支持，Agent 极易陷入“随机鹦鹉”（Stochastic Parrots）的困境——即通过概率模拟生成代码，却完全不理解内部复杂的架构逻辑。本文旨在通过标准化 AGENTS.md，为 Agent 提供精准的非显性约束，从而跨越 AI 开发的效能鸿沟。

<!-- more -->

## 规范背景与战略意义：为何拒绝“自动生成的平庸”

在企业私有库中引入 AGENTS.md 不仅是工程习惯，更是核心的成本控制与效能战略。根据 AGENTBENCH 的最新研究，高质量的人为干预是决定任务成功率的关键：

* 效能分水岭： 研究显示，人为编写的上下文文件能使 Agent 的任务成功率（Success Rate）提升约 4%；与之相对，完全依赖 LLM 自动生成的上下文文件往往会引入大量描述性噪声，导致性能退化 3%。
* 成本赤字： 盲目使用 LLM 生成的上下文文件不仅会引发“上下文腐败”（Context Rot），还会因为冗余信息导致推理成本增加超过 20%，同时并未带来准确度的提升。
* 部落知识（Tribal Knowledge）： 私有库的逻辑往往隐藏在架构师的决策偏好中。缺失这些代码无法表达的背景（如“为何禁用标准库解析器而采用内部 SDK”），会导致 Agent 陷入无效的 grep 循环和重构陷阱。
* 模型异质性警示： 架构师必须意识到，上下文文件并非“银弹”。虽然 Qwen3 或 GPT-5.2 能从中获益，但在 Sonnet 4.5 等模型上，不当的上下文文件甚至可能导致成功率下降超过 2%。

由于人类开发者的“部落知识”无法通过静态扫描完全获取，我们必须通过高度精炼的结构化文档，将这些潜在逻辑显性化。只有剔除显性冗余，Agent 才能将注意力层（Attention Layer）聚焦于关键指令。

---

## 核心编写原则：规避“显性冗余”

高质量的上下文优于海量的上下文。Agent 的上下文窗口是极其昂贵的资源，我们的目标是：只提供代码不可推断的知识。

### 显性描述 vs. 非显性约束

* 架构审计（Architectural Audit）： 严禁使用 AGENTS.md 罗列文件树或简单的类说明。Agent 具备强大的执行能力，它能通过 ls 或 tree 自行获取路径。在文档中写“utils.py 位于 src/”属于纯粹的 Token 浪费。
* 高级开发者类比： 正如你不会在分配任务前先给资深工程师发一份《Clean Code》全文一样，也不要给 Agent 灌输通用的编程准则。

### 必须包含的三类非显性信息

1. 架构决策背景（The "Why"）： 解释特定模式的原因（例如：为了支持热重载，所有模块必须通过特定工厂类实例化）。
2. 技术债边界： 明确哪些代码是“坏味道”但严禁重构的，防止 Agent 进入无意义的优化死循环。
3. 确定性偏好： 在多种实现方案并存时，明确当前的“唯一正确路径”，消除 Agent 的随机性选择。

既然我们已经明确了剔除噪声的必要性，接下来必须通过高度索引化的结构，确保存留的信息能够实现最大的 Token 命中率 和 缓存效率。

---

## 标准化结构框架

统一的结构能够优化 Agent 的推理路径，并充分利用底层模型的 Prompt Caching（提示词缓存） 机制。

### 项目全景与技术栈 (Project Overview & Stack)

明确环境底座，防止 Agent 产生错误的 API 假设。

* 环境约束： 精确到小版本（如 Python 3.12+）。
* 关键组件： 简述主框架在该项目中的非标准用途。

### 关键执行命令 (Essential Commands)

必须置于文档头部。 尽早纠正 Agent 的惯性假设，防止其因多次尝试失败而烧掉 Token。

* 错误路径： python manage.py migrate
* 正确路径： uv run manage.py migrate（强制使用 uv 包管理器以确保依赖隔离）。

### 开发约定与风格 (Coding Conventions)

“示例胜过描述”。 避免文字堆砌，使用“错误 vs. 修正”的代码块。

* 业务实例： 展示 Lucas Number 计算或 Django 模型层封装的具体风格。

风格对齐示例：

- **Wrong:** 缺乏类型提示，逻辑紧凑。
- **Right:**
  ```python
  def calculate_nth_lucas(n: int) -> int:
      # 必须在逻辑段落间保留空行
      if n == 0:
          return 2
  ```

### 显性边界与禁区 (Boundaries & Ground Rules)
设定物理与逻辑红线，这是确保私有库安全的核心。
*   **严禁项：** 禁止使用 `rm`、禁止未经允许的 `git push`、禁止修改 `tests/legacy/` 目录。
*   **不确定性条款（Clarification Clause）：** 当任务可能引发预设范围外的变动时，Agent 必须暂停并请求人工确认。禁止基于猜测填补知识空白。

### 部落知识快照 (Tribal Knowledge Snapshot)
捕捉那些无法从文档或代码中推导出的事实。
*   *示例：*“由于内部系统的大数精度限制，本项目禁止使用标准 JSON 解析器，必须调用全局 `custom_json_parser`。”

由于大型项目（Monorepo）的全局上下文极易稀释关键指令，我们需要通过分层策略来实现“渐进式披露”。

---

## 层次化上下文分发策略

在超大型代码库中，单一的全局文档会导致 **Attention Layer（注意力层）** 的稀释。应采用渐进式披露原则。

| 维度 | 根目录 AGENTS.md | 子目录（嵌套）AGENTS.md |
| :--- | :--- | :--- |
| **功能差异** | 定义全局底座、环境约束、红线禁区 | 定义具体业务逻辑、领域特定约束 |
| **覆盖范围** | 全局起始必读，长期有效 | 仅在 Agent 进入该子目录作业时触发 |
| **缓存优先级** | **最高**（长期缓存/Long-term Cache） | **中等**（任务级/Transient） |
| **主要目标** | 环境确定性、安全防范 | 逻辑推断加速、减少盲目 Grep |

**架构师洞察：** 在单体库中，多层级配置能显著减少 Agent 的无效搜索。当 Agent 进入 `/db` 目录时，嵌套文件会自动激活针对该层级的专用指令，将平均 Token 消耗降低 20% 以上。

---

## 编写技巧与质量控制

### “200 行原则” (Hard Constraint)
`AGENTS.md` 的篇幅必须保持精炼。**如果文档超过 200 行，通常意味着该文件已经失效。** 冗余信息会干扰 Agent 对高优先级指令（如安全禁区）的识别。

### 基于“失败驱动”的动态更新流程
不要一次性撰写海量文档，应采用基于 `pamelafox` 建议的反馈循环：
1.  **任务失败：** Agent 任务失败或写出不符合非显性约束的代码。
2.  **根因分析：** 确认失败是否由于缺失某种非显性规则。
3.  **增补约束：** 在 `AGENTS.md` 中以最简练的语言补齐该规则。
4.  **本地回滚：** 回滚所有本地代码更改。
5.  **回归验证：** 让 Agent 在新的上下文环境下重新执行。若成功，则证明该文件已成为**确定性护栏**。

### 符号链接（Symlink）与跨平台策略
由于不同厂商对文件名有不同偏好（如 Anthropic 强推 `CLAUDE.md`），建议将 `CONTRIBUTING.md` 软链接至 `AGENTS.md`。这是一种“Harness-agnostic”（平台无关）的权宜之计，确保人类开发规范与 AI 指令在单一事实来源（SSOT）下对齐。

---

## 附录：AGENTS.md 示例

```markdown AGENTS.md

1. Project Overview & Stack
- Env: Python 3.12+ (managed by `uv`)
- Core: Django 5.x / PostgreSQL
- Note: Asynchronous tasks must use `celery` with Redis broker; direct `asyncio` usage is prohibited.

2. Essential Commands (Must use `uv run`)
- Setup: `uv sync`
- Test: `uv run pytest`
- **Correct:** `uv run manage.py migrate`
- **Wrong:** `python manage.py migrate`

3. Coding Conventions (Style Over Description)
- All functions MUST have PEP 484 type hints.
- **Wrong Style:** Nested logic without whitespace.
- **Right Style:**
  def process_data(input_val: int) -> bool:
      # Step 1: Validation
      if input_val < 0: return False

      # Step 2: Core Logic
      return True

4. Boundaries & Ground Rules

* Clarification Clause: If a task requires changes outside explicit scope, PAUSE and ASK.
* NEVER execute rm or git push commands.
* NO TypeVar usage; prefer protocols.

5. Tribal Knowledge (The "Why")

* Q: Why custom_parser?
* A: Standard JSON parser fails with internal 128-bit UID precision.
* Q: Why no Wayland support?
* A: Internal legacy graphics drivers only support X11.
```