# Superpowers 项目代码维基

## 项目概述

**Superpowers** 是一套完整的软件开发生方法论，基于一组可组合的技能（skills）和初始化指令构建，专为 AI 编码代理设计。该项目由 Jesse Vincent 创建，当前版本为 **5.1.0**，采用 MIT 许可证。

### 核心价值主张

Superpowers 的核心理念是：AI 代理在开始编写代码之前不会直接跳入实现，而是先退后一步了解用户的真实需求，通过对话提炼规格说明，展示设计以供审核，最后生成详细的实施计划并通过子代理驱动开发流程执行。这种方法确保了高质量的产出，同时减少了返工和沟通成本。

### 支持的平台

该项目支持多种 AI 编码代理平台，包括 Claude Code、Codex CLI、Codex App、Factory Droid、Gemini CLI、OpenCode、Cursor 和 GitHub Copilot CLI。不同平台通过各自的插件系统集成，但共享相同的技能库。

---

## 项目架构

### 整体架构图

```
superpowers/
├── skills/                    # 核心技能库
│   ├── brainstorming/         # 头脑风暴与设计
│   ├── subagent-driven-development/  # 子代理驱动开发
│   ├── writing-plans/         # 编写实施计划
│   ├── test-driven-development/      # 测试驱动开发
│   ├── systematic-debugging/  # 系统化调试
│   ├── using-git-worktrees/   # Git 工作树管理
│   ├── verification-before-completion/  # 完成前验证
│   ├── requesting-code-review/ # 请求代码审查
│   ├── receiving-code-review/ # 接收代码审查反馈
│   ├── finishing-a-development-branch/  # 完成开发分支
│   ├── executing-plans/       # 执行计划
│   ├── dispatching-parallel-agents/    # 并行代理调度
│   ├── using-superpowers/     # 使用指南
│   └── writing-skills/        # 技能编写指南
├── docs/                      # 文档与规范
├── tests/                     # 测试套件
├── hooks/                     # 钩子配置
├── .claude-plugin/            # Claude Code 插件配置
├── .cursor-plugin/            # Cursor 插件配置
├── .codex-plugin/             # Codex 插件配置
└── .opencode/                 # OpenCode 插件配置
```

### 设计哲学

Superpowers 的设计遵循几个核心原则：**测试驱动开发**要求先写测试再看失败；**系统性优于临时性**强调过程而非猜测；**复杂性降低**将简洁性作为主要目标；**证据优于声明**要求在宣称成功前进行验证。这些原则贯穿于所有技能的设计和实现中。

---

## 核心模块详解

### 1. 头脑风暴模块 (brainstorming)

**文件路径**: `skills/brainstorming/SKILL.md`

**职责**: 在编写任何代码之前激活，通过苏格拉底式对话帮助用户将想法精炼为完整的设计和规格说明。

**核心流程**:

- 探索项目上下文 - 检查文件、文档、最近的提交
- 提供视觉辅助工具（可选）- 用于涉及视觉内容的问题
- 逐一提出澄清性问题 - 理解目的、约束、成功标准
- 提出 2-3 种方案 - 包含权衡和推荐建议
- 分段展示设计 - 根据复杂度调整，每次获取用户批准
- 编写设计文档 - 保存到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
- 自审规格 - 检查占位符、矛盾、歧义、范围
- 用户审核书面规格

**关键原则**:

- **一次一问** - 不要用多个问题淹没用户
- **首选多选** - 比开放式问题更容易回答
- **YAGNI 原则** - 从所有设计中无情删除不必要的功能
- **增量验证** - 展示设计，获得批准后再继续

**强制门控**: 在呈现设计并获得用户批准之前，不得调用任何实现技能、编写任何代码或采取任何实现行动。

**视觉辅助组件**: 包含 `visual-companion.md` 指南和一个基于浏览器的伴随工具，用于展示模型、图表和可视化选项，位于 `scripts/` 目录中。

### 2. 子代理驱动开发模块 (subagent-driven-development)

**文件路径**: `skills/subagent-driven-development/SKILL.md`

**职责**: 通过为每个任务分派新的子代理来执行实施计划，采用两阶段审查：先进行规格合规性审查，再进行代码质量审查。

**核心流程**:

- 读取计划文件，提取所有任务及其完整文本
- 为每个任务创建 TodoWrite 条目
- 为当前任务分派实施者子代理
- 实施者执行实现、测试、提交和自审
- 分派规格审查者子代理进行合规性检查
- 如有问题，实施者修复后重新审查
- 分派代码质量审查者子代理
- 如有问题，实施者修复后重新审查
- 标记任务完成，继续下一个任务
- 所有任务完成后，分派最终代码审查者

**为什么使用子代理**: 通过精确制作指令和上下文，确保子代理保持专注并在任务中成功。子代理不应继承会话的上下文或历史，而是应该精确构建它们所需的内容。

**模型选择策略**: 机械实现任务（隔离函数、清晰规格、1-2 个文件）使用快速便宜的模型；需要标准模型处理集成和判断任务；架构、设计和审查任务使用最强大的模型。

**状态处理**: 实施者子代理报告四种状态之一 - DONE（继续审查）、DONE_WITH_CONCERNS（完成但有疑虑）、NEEDS_CONTEXT（需要上下文）、BLOCKED（无法完成）。每种状态都有明确的处理流程。

### 3. 编写计划模块 (writing-plans)

**文件路径**: `skills/writing-plans/SKILL.md`

**职责**: 将已批准的规格说明转化为详细的实施计划，假设工程师对代码库没有上下文且品味有待提高。

**计划结构要求**:

```
# [功能名称] 实施计划

> **代理要求**: 使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 执行此计划

**目标**: [一句话描述构建内容]
**架构**: [2-3 句关于方法的描述]
**技术栈**: [关键技术和库]

---
```

**任务粒度**: 每个步骤是一个操作（2-5 分钟），例如"编写失败测试"、"运行确保失败"、"编写最小实现"、"运行确保通过"、"提交"。

**无占位符原则**: 每个步骤必须包含工程师需要的实际内容。禁止"TBD"、"TODO"、"稍后实现"等占位符。

**自审清单**: 编写完计划后，以新的眼光检查规格和计划是否匹配，包括规格覆盖检查、占位符扫描、类型一致性检查。

### 4. 测试驱动开发模块 (test-driven-development)

**文件路径**: `skills/test-driven-development/SKILL.md`

**职责**: 强制执行红-绿-重构循环，确保所有生产代码都有先失败的测试。

**铁律**: **没有先失败的测试就不能编写生产代码**。违反这条规则就是违反 TDD 的精神。

**红-绿-重构循环**:

```
RED: 编写失败测试 → 验证失败原因正确
    ↓
GREEN: 编写最小代码通过测试 → 验证所有测试通过
    ↓
REFACTOR: 重构清理 → 保持测试绿色
```

**核心原则**:

- **无例外** - 不要保留代码作为"参考"
- **不要适应** - 在编写测试时不要改编旧代码
- **不要看它** - 删除意味着删除
- **从测试实现** - 清空头脑，从测试重新开始

**测试质量标准**: 好的测试应该最小化（只测一件事）、清晰（名称描述行为）、显示意图（展示期望的 API）。

**常见合理化借口表**: 包含"太简单不需要测试"、"我之后再测试"、"之后测试也能达到同样目标"等借口及其对应的现实，帮助识别和避免这些思维陷阱。

### 5. 系统化调试模块 (systematic-debugging)

**文件路径**: `skills/systematic-debugging/SKILL.md`

**职责**: 提供四阶段根本原因分析流程，确保在尝试修复之前找到真正的问题所在。

**铁律**: **在根本原因调查之前不能提出修复方案**。症状修复是失败的另一种形式。

**四个阶段**:

**Phase 1: 根本原因调查** - 仔细阅读错误信息、一致地重现问题、检查最近的更改、在多组件系统中收集证据

**Phase 2: 模式分析** - 找到工作示例、与参考对比、识别差异、理解依赖

**Phase 3: 假设和测试** - 形成单一假设、最小化测试、验证后再继续

**Phase 4: 实施** - 创建失败测试用例、实现单一修复、验证修复

**多组件系统诊断**: 当系统有多个组件时，在提出修复方案之前添加诊断工具来收集证据。例如在每个组件边界记录数据进出情况。

**3+ 修复失败处理**: 如果尝试了 3 次或更多修复都失败了，这表明存在架构问题。应该停止并质疑基础架构是否正确。

### 6. Git 工作树管理模块 (using-git-worktrees)

**文件路径**: `skills/using-git-worktrees/SKILL.md`

**职责**: 确保工作发生在隔离的工作空间中，优先使用平台原生工具，回退到手动 git 工作树。

**步骤 0: 检测现有隔离** - 检查 `GIT_DIR` 和 `GIT_COMMON` 是否不同来判断是否已在工作树中

**步骤 1a: 原生工作树工具（首选）** - 检查是否有原生工作树创建工具如 `EnterWorktree`、`WorktreeCreate`、`/worktree` 命令等

**步骤 1b: Git 工作树回退** - 仅在步骤 1a 不适用时使用，在 `.worktrees/`、`worktrees/` 或 `~/.config/superpowers/worktrees/` 中创建工作树

**目录选择优先级**: 明确的用户偏好 > 现有项目本地目录 > 现有全局目录 > 默认 `.worktrees/`

**安全验证**: 在创建项目本地目录之前必须验证目录被忽略，使用 `git check-ignore` 检查。

**步骤 3: 项目设置** - 自动检测并运行适当的设置（npm install、cargo build、pip install 等）

**步骤 4: 验证干净基线** - 运行测试确保工作空间从干净状态开始

### 7. 完成前验证模块 (verification-before-completion)

**文件路径**: `skills/verification-before-completion/SKILL.md`

**职责**: 要求在声称工作完成、固定或通过之前运行验证命令并确认输出；始终是证据而非断言。

**铁律**: **没有新鲜验证证据就不能声称完成**。

**门控函数**: 在声称任何状态或表达满意度之前，必须执行完整命令、读取输出、检查退出码、验证输出是否确认声明。

**常见失败场景**: 测试通过（需要测试命令输出而非之前的运行）、代码审查通过（需要审查输出而非主观判断）、构建成功（需要构建命令退出 0 而非日志看起来好）。

### 8. 请求代码审查模块 (requesting-code-review)

**文件路径**: `skills/requesting-code-review/SKILL.md`

**职责**: 在问题级联之前分派代码审查子代理来捕获问题。

**何时请求审查**: 每个子代理驱动开发任务后、主要功能完成后、合并到主分支前（强制）；卡住时、重构前、修复复杂 bug 后（可选）。

**审查流程**: 获取 git SHA → 分派代码审查子代理（使用 `code-reviewer.md` 模板）→ 根据反馈采取行动（关键问题立即修复、重要问题继续前修复、次要问题记录稍后处理）。

### 9. 接收代码审查反馈模块 (receiving-code-review)

**文件路径**: `skills/receiving-code-review/SKILL.md`

**职责**: 接收代码审查反馈时的技术评估模式，要求技术严谨性而非表演性同意。

**响应模式**: 读取完整反馈 → 用自己的话复述理解 → 验证是否符合代码库现实 → 评估是否适合本代码库 → 技术确认或合理反驳 → 一次实施一项并测试每个

**禁止的响应**: "你说得对！"、"好观点！"、"我现在来实现"等表演性回复。

**何时反驳**: 当建议破坏现有功能、审查者缺乏完整上下文、违反 YAGNI、技术上不适合当前技术栈、存在遗留/兼容性原因时。

### 10. 完成开发分支模块 (finishing-a-development-branch)

**文件路径**: `skills/finishing-a-development-branch/SKILL.md`

**职责**: 通过验证测试 → 检测环境 → 呈现选项 → 执行选择 → 清理的流程指导完成开发工作。

**步骤 1: 验证测试** - 在呈现选项前必须验证测试通过

**步骤 2: 检测环境** - 确定工作空间状态（普通仓库 vs 工作树）

**步骤 3: 确定基础分支** - 检查从哪里分支出去

**步骤 4: 呈现选项** - 正常仓库和命名分支工作树显示 4 个选项：本地合并、推送创建 PR、保持分支、丢弃；分离 HEAD 显示 3 个选项。

**步骤 5: 执行选择** - 根据用户选择执行相应操作

**步骤 6: 清理工作空间** - 仅对选项 1 和 4 执行，在合并成功后删除工作树

### 11. 执行计划模块 (executing-plans)

**文件路径**: `skills/executing-plans/SKILL.md`

**职责**: 在单独的会话中执行已编写的实施计划，包含审查检查点。

**注意**: 告知用户 Superpowers 在有子代理支持的平台上效果更好，如 Claude Code 或 Codex。如果子代理可用，应使用 `superpowers:subagent-driven-development` 而不是此技能。

**流程**: 加载并审查计划 → 执行所有任务 → 报告完成。

### 12. 并行代理调度模块 (dispatching-parallel-agents)

**文件路径**: `skills/dispatching-parallel-agents/SKILL.md`

**职责**: 在面对 2 个或更多独立任务时可以并行工作而无共享状态或顺序依赖。

**何时使用**: 3 个以上不同根本原因导致的测试失败、多个独立子系统损坏、每个问题可以在不了解其他问题上下文的情况下理解、无共享状态。

**何时不使用**: 失败相关（修复一个可能修复其他）、需要完整系统状态、探索性调试、有共享状态（代理会相互干扰）。

**代理提示结构**: 每个代理应获得特定范围、清晰目标、约束和预期输出的明确描述。

### 13. 使用 Superpowers 模块 (using-superpowers)

**文件路径**: `skills/using-superpowers/SKILL.md`

**职责**: 建立如何查找和使用技能的指南，要求在任何响应之前使用 Skill 工具调用，包括澄清问题。

**指令优先级**: 用户显式指令（最高）> Superpowers 技能 > 默认系统提示（最低）。

**使用规则**: 在任何响应或操作之前调用相关或请求的技能。即使只有 1% 的可能性技能适用，也应调用技能检查。

### 14. 编写技能模块 (writing-skills)

**文件路径**: `skills/writing-skills/SKILL.md`

**职责**: 创建新技能、编辑现有技能或在部署前验证技能是否正常工作。

**TDD 映射**: 技能创建对应 TDD 中的测试用例、压力场景对应测试失败、SKILL.md 文档对应生产代码、无技能时代理违规对应基线行为。

**铁律**: **没有失败测试就不能编写技能**。这适用于新技能和编辑现有技能。

**RED-GREEN-REFACTOR 流程**: RED - 在没有技能的情况下运行压力场景；GREEN - 编写解决这些特定违规的技能；REFACTOR - 关闭发现的漏洞。

**技能类型**: 纪律执行技能（规则/要求）、技术技能（操作指南）、模式技能（心智模型）、参考技能（文档/API）。

---

## 辅助技能与参考文档

### 测试反模式 (testing-anti-patterns.md)

**路径**: `skills/test-driven-development/testing-anti-patterns.md`

**内容**: 记录常见的测试陷阱，如测试 mock 行为而非真实行为、为测试类添加仅测试方法、在不理解依赖的情况下 mock。

### 根本原因追踪 (root-cause-tracing.md)

**路径**: `skills/systematic-debugging/root-cause-tracing.md`

**内容**: 通过调用栈向后追踪 bug 以找到原始触发器的高级技术。

### 纵深防御 (defense-in-depth.md)

**路径**: `skills/systematic-debugging/defense-in-depth.md`

**内容**: 在找到根本原因后在多个层添加验证的高级技术。

### 条件等待 (condition-based-waiting.md)

**路径**: `skills/systematic-debugging/condition-based-waiting.md`

**内容**: 用条件轮询替换任意超时的高级技术。

### Anthropic 最佳实践 (anthropic-best-practices.md)

**路径**: `skills/writing-skills/anthropic-best-practices.md`

**内容**: Anthropic 官方技能编写最佳实践的补充指南。

### 说服原则 (persuasion-principles.md)

**路径**: `skills/writing-skills/persuasion-principles.md`

**内容**: 基于研究的技能编写说服技巧（Cialdini, 2021; Meincke et al., 2025）。

---

## 插件与集成架构

### OpenCode 插件

**文件路径**: `.opencode/plugins/superpowers.js`

**核心导出**: `SuperpowersPlugin` - 异步函数，接收 `{ client, directory }` 参数。

**主要功能**:

- `config` 钩子: 将技能路径注入 OpenCode 配置，实现自动发现技能，无需手动符号链接
- `experimental.chat.messages.transform` 钩子: 将 bootstrap 内容注入每个会话的第一条用户消息

**Bootstrap 缓存机制**: 使用模块级缓存 `_bootstrapCache` 避免每个代理步骤重复磁盘操作。读取一次 SKILL.md 文件后缓存结果。

**工具映射**: 技能使用 Claude Code 工具名称，插件提供 OpenCode 等效替换的映射说明。

### Claude Code 插件

**文件路径**: `.claude-plugin/plugin.json`

**配置**: 标准 Claude 插件市场条目，包含名称、描述、版本、作者、许可证和关键词。

### Cursor 插件

**文件路径**: `.cursor-plugin/plugin.json`

**配置**: 使用 Cursor 的钩子系统指向 `hooks/hooks-cursor.json`。

### Codex 插件

**文件路径**: `.codex-plugin/plugin.json`

**配置**: 包含完整的界面配置，包括显示名称、简短描述、类别、功能、品牌颜色等。

### 钩子系统

**文件路径**: `hooks/hooks.json` 和 `hooks/hooks-cursor.json`

**配置**: 定义 SessionStart 钩子，在匹配 "startup|clear|compact" 时运行 `run-hook.cmd session-start` 命令。

---

## 依赖关系

### 项目级依赖

Superpowers 是一个**零依赖**项目设计原则，这是核心贡献要求之一。`package.json` 仅包含基本的项目元数据：

```json
{
  "name": "superpowers",
  "version": "5.1.0",
  "type": "module",
  "main": ".opencode/plugins/superpowers.js"
}
```

### 插件级依赖

每个目标平台的插件配置仅依赖平台自身的插件系统：

- **Claude Code**: 依赖 Claude 插件市场
- **Codex**: 依赖 Codex 插件系统
- **Cursor**: 依赖 Cursor 插件系统
- **OpenCode**: 依赖 OpenCode 插件管理器
- **Gemini CLI**: 依赖 Gemini 扩展系统
- **Factory Droid**: 依赖 Droid 插件系统
- **GitHub Copilot CLI**: 依赖 Copilot 插件系统

### 技能间依赖

技能之间存在明确的引用和依赖关系：

- `subagent-driven-development` 依赖: `using-git-worktrees`、`writing-plans`、`requesting-code-review`、`finishing-a-development-branch`
- `executing-plans` 依赖: `using-git-worktrees`、`writing-plans`、`finishing-a-development-branch`
- `systematic-debugging` 可能调用: `test-driven-development`、`verification-before-completion`
- `writing-skills` 依赖: `test-driven-development`

---

## 工作流程集成

### 基本工作流程

```
brainstorming → using-git-worktrees → writing-plans → 
subagent-driven-development/executing-plans → 
requesting-code-review → finishing-a-development-branch
```

**1. brainstorming** - 在编写代码之前激活，通过问题精炼想法，探索替代方案，分段展示设计以供验证，保存设计文档。

**2. using-git-worktrees** - 设计批准后激活，在新分支上创建隔离工作空间，运行项目设置，验证干净的测试基线。

**3. writing-plans** - 有批准的设计时激活，将工作分解为可管理的任务（每个 2-5 分钟），每个任务有确切的文件路径、完整代码和验证步骤。

**4. subagent-driven-development 或 executing-plans** - 有计划时激活，每个任务分派新的子代理，进行两阶段审查，或在批次执行中有人工检查点。

**5. test-driven-development** - 实现期间激活，强制执行红-绿-重构：先写失败测试、看它失败、写最小代码、看它通过、提交。在测试之前删除编写的代码。

**6. requesting-code-review** - 任务之间激活，根据计划审查并报告问题，按严重程度报告，关键问题阻止进度。

**7. finishing-a-development-branch** - 任务完成时激活，验证测试，呈现选项（合并/PR/保持/丢弃），清理工作树。

---

## 测试套件

### 测试结构

```
tests/
├── brainstorm-server/         # 头脑风暴服务器测试
├── claude-code/              # Claude Code 集成测试
├── codex-plugin-sync/         # Codex 插件同步测试
├── explicit-skill-requests/   # 显式技能请求测试
├── opencode/                 # OpenCode 测试
├── skill-triggering/         # 技能触发测试
└── subagent-driven-dev/      # 子代理驱动开发集成测试
```

### 头脑风暴服务器测试

**文件路径**: `tests/brainstorm-server/`

**内容**: WebSocket 服务器的协议测试和生命周期测试，包括 `server.test.js`、`ws-protocol.test.js` 和 `windows-lifecycle.test.sh`。

**依赖**: `package.json` 和 `package-lock.json` 显示使用 npm 进行测试管理。

### Claude Code 测试

**文件路径**: `tests/claude-code/`

**内容**: 包括令牌使用分析脚本 (`analyze-token-usage.py`)、技能测试运行脚本 (`run-skill-tests.sh`)、各种集成测试脚本。

### 显式技能请求测试

**文件路径**: `tests/explicit-skill-requests/`

**内容**: 测试代理在显式请求技能时的行为，包含各种提示模板和运行脚本。

---

## 项目运行方式

### 安装到 Claude Code

```bash
/plugin install superpowers@claude-plugins-official
```

或使用 Superpowers 市场：

```bash
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

### 安装到 Codex CLI

通过插件搜索界面搜索 "superpowers" 并安装。

### 安装到 OpenCode

在 `opencode.json` 中添加插件：

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git"]
}
```

### 安装到 Gemini CLI

```bash
gemini extensions install https://github.com/obra/superpowers
gemini extensions update superpowers
```

### 安装到 Cursor

在 Cursor Agent 聊天中使用：

```
/add-plugin superpowers
```

### 安装到 GitHub Copilot CLI

```bash
copilot plugin marketplace add obra/superpowers-marketplace
copilot plugin install superpowers@superpowers-marketplace
```

---

## 贡献指南

### 贡献流程

1. Fork 仓库
2. 切换到 'dev' 分支
3. 为工作创建分支
4. 按照 `writing-skills` 技能创建和测试新技能及修改的技能
5. 提交 PR，填写 pull request 模板

### 重要注意事项

- **94% 的 PR 拒绝率** - 几乎所有被拒绝的 PR 都是由未阅读或未遵循指南的代理提交的
- **必须阅读 PR 模板** - 每个 PR 必须完全填写 PR 模板的所有部分
- **必须搜索现有 PR** - 在开 PR 前必须搜索已存在和已关闭的相关 PR
- **必须有人类参与** - 显示没有人类参与的 PR 将被关闭
- **不允许第三方依赖** - PR 添加可选或必需的第三方项目依赖不会被接受

### 新平台支持要求

如果 PR 添加对新平台（IDE、CLI 工具、代理运行器）的支持，必须包括会话转录证明集成端到端工作。测试消息为：

> "让我们做一个 React 待办事项列表"

一个工作的集成会在接受测试时自动触发 `brainstorming` 技能。

---

## 版本历史

当前版本: **5.1.0**

版本更新信息保存在 `.version-bump.json` 和 `RELEASE-NOTES.md` 中。

---

## 文件清单

### 核心技能文件

| 技能名称 | 主要文件 | 辅助文件 |
|---------|---------|---------|
| brainstorming | SKILL.md | visual-companion.md, spec-document-reviewer-prompt.md, scripts/ |
| subagent-driven-development | SKILL.md | implementer-prompt.md, spec-reviewer-prompt.md, code-quality-reviewer-prompt.md |
| writing-plans | SKILL.md | plan-document-reviewer-prompt.md |
| test-driven-development | SKILL.md | testing-anti-patterns.md |
| systematic-debugging | SKILL.md | root-cause-tracing.md, defense-in-depth.md, condition-based-waiting.md, condition-based-waiting-example.ts, find-polluter.sh, CREATION_LOG.md, test-*.md |
| using-git-worktrees | SKILL.md | - |
| verification-before-completion | SKILL.md | - |
| requesting-code-review | SKILL.md | code-reviewer.md |
| receiving-code-review | SKILL.md | - |
| finishing-a-development-branch | SKILL.md | - |
| executing-plans | SKILL.md | - |
| dispatching-parallel-agents | SKILL.md | - |
| using-superpowers | SKILL.md | references/codex-tools.md, references/copilot-tools.md, references/gemini-tools.md |
| writing-skills | SKILL.md | anthropic-best-practices.md, graphviz-conventions.dot, persuasion-principles.md, render-graphs.js, testing-skills-with-subagents.md, examples/CLAUDE_MD_TESTING.md |

### 配置文件

| 文件路径 | 用途 |
|---------|------|
| package.json | 项目元数据 |
| .claude-plugin/plugin.json | Claude Code 插件配置 |
| .cursor-plugin/plugin.json | Cursor 插件配置 |
| .codex-plugin/plugin.json | Codex 插件配置 |
| .opencode/plugins/superpowers.js | OpenCode 插件实现 |
| hooks/hooks.json | 标准钩子配置 |
| hooks/hooks-cursor.json | Cursor 钩子配置 |
| .opencode/INSTALL.md | OpenCode 安装指南 |

### 文档文件

| 文件路径 | 用途 |
|---------|------|
| README.md | 项目主文档 |
| CLAUDE.md | 贡献者指南 |
| GEMINI.md | Gemini 特定指南 |
| LICENSE | MIT 许可证 |
| CODE_OF_CONDUCT.md | 行为准则 |
| RELEASE-NOTES.md | 版本发布说明 |

---

## 关键设计决策

### 1. 零依赖原则

Superpowers 核心不添加可选或必需的第三方依赖。这是经过深思熟虑的设计决策，确保插件在任何环境下都可预测和可靠运行。

### 2. 技能自动触发

技能通过引导脚本在会话开始时自动注册，使技能在正确时刻自动触发，无需用户每次手动激活。用户只需正常使用 AI 代理，技能就会在需要时激活。

### 3. 两阶段审查

子代理驱动开发中的两阶段审查（先规格合规，再代码质量）确保实现既符合需求又具有良好的代码质量。这个顺序是强制的，先进行代码质量审查是错误的行为。

### 4. 强制门控

brainstorming 技能的硬门控（HARD-GATE）确保在呈现设计并获得用户批准之前不进行任何实现。这防止了常见的"直接跳入编码"反模式。

### 5. 纪律优于灵活

对于纪律执行技能（如 TDD、验证前完成），规则是强制的，不允许适应。这些技能的设计目的是防止常见的合理化借口，强制执行最佳实践。

---

## 总结

Superpowers 是一个精心设计的软件开发生方法论，通过一组可组合的技能和自动化触发机制，为 AI 编码代理提供了一致的高质量开发流程。其架构设计强调零依赖、自动触发、纪律执行和系统性优于临时性。核心技能覆盖了从需求澄清（brainstorming）到项目交付（finishing-a-development-branch）的完整开发周期，确保每一步都有明确的指导原则和验证机制。
