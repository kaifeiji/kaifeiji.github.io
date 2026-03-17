---
title: Openspec 使用心得
date: 2026-03-17 11:30:06
categories:
- AI
tags:
- Openspec
---

TLDR;

核心概念与哲学：规格驱动开发、真相来源、意图锁定、行为契约

产物结构：proposal.md、design.md、tasks.md、spec.md

关键功能：opsx-propose、opsx-apply、opsx-verify、opsx-sync、opsx-archive

进阶使用：自定义schema、配合 AGENTS.md、分级结构、屏蔽 archive、防止 AI 话痨

<!-- more -->

repo：https://github.com/Fission-AI/OpenSpec

## 为什么用 Openspec

- 将模糊的人类意图转化为结构化的工程共识， 解决 Vibe Coding “上下文过载”问题
- 用规格降低 AI agent 的认知负荷，提升其执行效率和质量

## 使用方法

### 初始化项目

以下命令会在项目根目录下创建一个 openspec 目录，并在其中生成 IDE 相关的 prompts、skills 等文件：

``` bash
npm install -g @fission-ai/openspec
cd projectRoot
openspec init
```

### 生成需求、设计、任务和规范

有两种常见路径：

- 单步路径：`/opsx-propose` 会创建 change，并一次性生成 proposal、specs、design、tasks。

``` text
/opsx-new extensions-widgets-common-edit-add-batch-editing-support
/opsx-continue
*or*
/opsx-ff
```

- 分步路径：`/opsx-new` 只创建 change 脚手架，随后通过 `/opsx-continue` 或 `/opsx-ff` 生成 artifacts。

``` text
/opsx-propose extensions-widgets-common-edit-add-batch-editing-support
```

### 审阅和修改生成的 spec

人工审查 proposal.md、design.md、tasks.md 和 change 下的 delta specs（通常是 specs/<capability>/spec.md），确保它们准确地反映了需求、设计方案和任务列表。必要时，可以对它们进行修改和完善。

### 执行

AI agent 根据文档约束进行编码实现，并在本地进行测试验证，直到完成 tasks.md 中的所有内容，并满足 spec.md 中的需求和设计约束。

``` text
/opsx-apply extensions-widgets-common-edit-add-batch-editing-support
```

### 验证（可选）

``` text
/opsx-verify extensions-widgets-common-edit-add-batch-editing-support
```

### 同步（可选）

``` text
/opsx-sync extensions-widgets-common-edit-add-batch-editing-support
```

### 归档 / 批量归档

``` text
/opsx-archive extensions-widgets-common-edit-add-batch-editing-support
*or*
/opsx-bulk-archive
```

## 目录结构

- openspec
  - changes
    - archive
      - 2026-03-17-archived-feature-A
        - specs
          - feature-A-1
            - spec.md
          - feature-A-2
            - spec.md
        - design.md
        - proposal.md
        - tasks.md
    - active-feature-B
      - specs
        - feature-B
          - spec.md
      - design.md
      - proposal.md
      - tasks.md
  - explorations
    - explore-feature-C.md
  - schemas
    - templates
      -design.md
      -proposal.md
      - spec.md
      - tasks.md
  - specs
    - feature-A
      - spec.md

### 目录说明

- changes： 当前正在进行的工作
  - proposal.md: 需求分析
  - design.md: 设计方案
  - tasks.md: 任务列表
  - specs/<capability>/spec.md: 本次 change 对某个 capability 的 delta spec，可以有一个或多个
- archive： 已经完成的工作，包括流程和成果
- explorations： 还处于探索阶段的想法和方案
- schemas： spec 模板，规范了 spec 的结构和内容
- specs： 主 specs，描述系统当前稳定的行为，是长期的 source of truth

### 集中式、扁平化存储

目前 Openspec 的目录结构采用集中式存储，所有的 spec 都在 projectRoot/openspec 目录下，而且 changes 和 specs 目录下的项目也是扁平化的，只有一级目录结构。

在这种现状下，ExperienceBuilder + extensions 项目有两种选择：

- 尊重集中式、扁平化存储，在 spec 命名时加上 package 前缀，例如：jimu-core-feature-A、extensions-widgets-common-edit-feature-B。缺点是命名冗长，不够优雅。
- 在每个 package 都进行 `openspec init` 。缺点是会在每个 package 下生成一遍类似 .github/prompts 和 .github/skills 的 IDE 配置文件

需要注意的是，当前 Openspec 并不原生支持一个项目内多根 changes/specs 聚合扫描。所以“每个 package 一个 openspec”更接近多套独立的 Openspec 项目，而不是单套 Openspec 原生管理整个 mono-repo 的多个子根目录。

Enh issue:
https://github.com/Fission-AI/OpenSpec/issues/581
https://github.com/Fission-AI/OpenSpec/issues/662

## 自定义工作流

Openspec 的 schema 支持自定义工作流和 spec 产物。

fork 已有的 schema 模板，修改后进行 validate 来适配自己的工作流。

``` bash
openspec schema fork spec-driven bug-fix
*手动修改 schema 中的模板和提示词*
openspec schema validate bug-fix
```

针对 bug fix、typo 或 style 这类不会改变 capability 边界的 change，有两种选择：
1. 采用更轻量级的工作流，不生成 proposal.md、 design.md 和 spec.md，只在 tasks.md 中描述需求和方案，直接执行，最终 archive 也不会固化到 specs 目录。
2. 跳过 Openspec，直接在代码库中进行修改和 review。

## 配合 AGENTS.md

### 防止上下文污染和 token 浪费

提示 AI agent 只读 openspec 中相关的 specs 和活跃的 changes，非必要不读 archive 中的内容。

### mono-repo 实践

对于 mono-repo 来说，最理想的状况是每个 package 都有一个 openspec 目录，存放该 package 的 spec 文件。这样可以让每个 package 的 spec 更加独立，便于维护和管理。

但需要明确，这种方式在当前实现下意味着每个 package 都是一个独立的 Openspec 项目，而不是单个 Openspec 实例原生管理多个 package 的 changes/specs。

可以在 AGENTS.md 添加提示词，引导 Openspec 在各个 package 下创建并管理 openspec 目录。需要在 AGENTS.md 中维护一份路径配置，而且有时候大模型的遵从性并非百分之百。

### 最小化提示词

不应该放 AI agent 自己能从代码推断的信息，例如：

- 这是一个 Typescript 项目
- jimu-core package 提供了一些核心功能
- 样式和主题采用的基础库是 emotion

应该放 AI agent 不知道的内容，例如：

- 团队内对命名、目录组织的约定
- 编写 Unit test 时的粒度和覆盖要求
- 依赖 esri 库的历史版本功能和重要的 BREAKING CHANGE （objectId、graphic.layer...）

## 关于 capability 和 spec

### 什么是 spec?

Openspec 的核心思想是将散落在聊天记录、文档、ISSUR、PR 中的需求、设计、任务等共识，沉淀成结构化的 spec，形成一个单一的事实来源，便于团队成员（特别是 AI agent）查阅和维护。

spec 是由一系列的 capability 组成的，通常 capability 是下面这些东西：

1. 一个用户可感知的业务能力，例如 auth、payments、edit-widget、notifications、search。

2. 一个系统层面的行为域，例如 session-management、rate-limiting、job-scheduling、feature-flags。

3. 一个在长期会反复被修改、讨论、验证的能力边界
也就是“以后很多 change 都可能继续改它”的那类对象。

capability 通常不是下面这些：

1. 一个很小的交互点，比如 click-save-button、open-modal 这种太细了。

2. 一次性的缺陷或需求单，比如 fix-empty-title-bug、improve-cache-hit-rate 这种更像 change 名，不像 capability 名。

3. 纯实现细节，比如 use-widget-service、redis-lock-helper、button-component-color 这种更像设计或代码组织，不像行为契约。

### Openspec 自动化管理 specs

在执行 /opsx:sync 或 /opsx:archive 时，Openspec 会进行自动化版本控制和变更管理，将 changes 中的 delta specs 同步到 specs 目录下。这里的同步不是简单的复制，而是按 requirement 的增删改重命名规则进行应用：

- 如果是全新的 capability，会在 specs 目录下创建一个新的目录，并创建对应的 spec.md。
- 如果 capability 已经存在，会按 `ADDED / MODIFIED / REMOVED / RENAMED` 这些 requirement 操作更新主 spec。
- 需要特别注意：当前实现里，`MODIFIED` 更接近按整个 requirement block 替换，而不是只改局部差异。因此写 delta spec 时，通常应该提供完整更新后的 requirement block。
- 如果对其它的 capability 也有修改，会分别对这些 capability 的主 spec 进行处理。

### 对specs 进行人工审阅和维护

虽然 Openspec 提供了自动化的工具来生成和管理 spec，但人工审阅和维护仍然是非常重要的。团队成员需要定期审阅 changes 中的 spec.md，确保它们准确地反映了需求、设计方案和任务列表。必要时，可以对它们进行修改和完善。

### 老项目的起步策略

老项目接入 Openspec 时，最常见的问题是：没有现成的主 specs。

这意味着 Openspec 仍然可以用，但一开始更多是作为 change 管理工具来使用，spec 作为长期系统记忆的价值还没有完全建立起来。

需要先接受两个现实：

- Openspec 当前没有官方提供的 baseline generator，不能一键从现有代码库自动生成一整套主 specs。
- 老项目通常也不适合一开始就全量补齐所有 capability 的 spec，这样成本太高，而且很容易写成没人维护的文档。

比较现实的起步方式有三种：

1. 先把 Openspec 当作结构化的 change workflow 来使用，只管理 proposal、tasks、design 和实现过程；对不涉及行为约束变化的 change，可以不立即沉淀 spec。
2. 先为最核心、最稳定、最常改的 capability 补一小批主 specs，形成后续演进的锚点。
3. 采用渐进式策略：每次改到哪个 capability，就顺手把对应的主 spec 建起来，而不是一次性全量建模。

其中第 3 种通常最适合老项目。因为 Openspec 的哲学本来就是 brownfield-first、iterative、lightweight，更适合边改边沉淀，而不是先做一轮大规模文档工程。

实际落地时，建议优先补下面几类 capability：

- 对外承诺明确、用户可感知的核心功能，例如 auth、payment、edit-widget、search。
- 经常改、容易回归、多人协作时容易产生理解偏差的功能域。
- 后续希望持续让 AI agent 参与修改的模块。

不建议优先 spec 化的内容包括：

- 很底层的纯实现细节。
- 很快会被重写的旧模块。
- 一次性的 bug、脚本或临时迁移逻辑。

一个务实的目标是：先补 5 到 10 个高价值 capability 的主 specs，后续规定凡是改到这些 capability 的 change，都尽量同步更新 spec；其它区域则继续按需逐步补齐。
