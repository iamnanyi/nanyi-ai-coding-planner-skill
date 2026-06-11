# Nanyi AI Coding Planner Skill

[English](README.en.md)

## 1. 这是什么

`nanyi-ai-coding-planner-skill` 是一个面向 AI Coding 的规划型 Skill。它帮助你把较大或复杂的开发目标拆成可审查、可交接、可分轮执行的小任务，并生成当前下一步可直接复制到新对话中的执行 Prompt。

这个 Skill 只负责规划、代码扫描、文档整理和下一步 Prompt 生成，不直接修改业务代码。

它的设计重点不是依赖最强模型或超长上下文，而是把任务状态沉淀到文档里。只要下一轮工具能读取这些文档，根据 `next-prompt.md` 找到当前 Prompt 文件并复制其内容开启新对话，就可以继续执行当前下一步。

每次生成执行 Prompt 时，Skill 会把当前 Prompt 作为新文件保存到 `prompts/` 目录，`next-prompt.md` 只作为当前入口。历史 Prompt 默认不作为执行上下文，除非需要人工回溯或排查历史偏差。

## 2. 它解决什么问题

AI Coding 在复杂任务中容易遇到这些问题：

- 一轮对话修改范围过大
- 上下文过长后遗漏关键约束
- 规划和实际代码状态脱节
- 后续对话缺少可靠交接信息
- 多任务并行时缺少互不覆盖的任务进度总账
- 团队里不同工具、不同模型的额度和可用性不稳定

`nanyi-ai-coding-planner-skill` 通过独立任务目录、任务总账和单任务 handoff，把这些信息固定下来。

这样做的结果是：任务不必强绑定某一个 AI 工具。哪个工具还有额度，就根据 `next-prompt.md` 复制当前 Prompt 文件到哪个工具里继续；模型能力普通一些也可以完成小步任务，因为上下文和约束已经写在文档中。

## 3. 核心工作流

1. 读取项目规则文件，例如 `CLAUDE.md`、`AGENTS.md`、`README`、`Makefile` 或 CI 配置。
2. 明确目标、非目标、允许修改范围、禁止修改范围和完成标准。
3. 基于真实代码扫描结果生成任务规划。
4. 维护任务目录内进度总账和单任务交接文档。
5. 将任务拆成小步。
6. 只生成当前下一步执行 Prompt，不一次性生成所有后续 Prompt。

## 4. 安装方式

使用 skills 命令安装：

```bash
npx skills add https://github.com/iamnanyi/nanyi-ai-coding-planner-skill --skill nanyi-ai-coding-planner-skill
```

手动安装：

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/iamnanyi/nanyi-ai-coding-planner-skill.git
cp -R nanyi-ai-coding-planner-skill/nanyi-ai-coding-planner-skill ~/.claude/skills/
```

## 5. 快速开始

只需要描述你要完成的最终目标，不需要提前拆成代码步骤。如果需求很长，也可以先让其他 AI 帮你把需求总结成目标描述，再交给本 Skill 规划。

新功能示例：

```text
使用 nanyi-ai-coding-planner-skill Skill。

目标：
我要实现一个功能开关系统，用于控制新功能灰度发布。它需要支持按环境开启、按用户分组开启、按百分比灰度开启，并提供一个统一的读取入口给业务代码使用。
```

旧功能迭代示例：

```text
使用 nanyi-ai-coding-planner-skill Skill。

目标：
在现有首页弹窗功能上增加会员展示规则。配置为“仅未订阅用户可见”的弹窗，只对未订阅用户展示；点击弹窗内容保持原有跳转逻辑；点击关闭按钮后 2 天内不再展示；点击弹窗外区域只关闭本次弹窗，不触发 2 天抑制。
```

重构任务示例：

```text
使用 nanyi-ai-coding-planner-skill Skill。

目标：
重构订单状态判断逻辑。把分散在多个页面和接口中的状态判断收敛到统一模块中，保持现有用户行为和接口返回不变，并补充必要的回归验证。
```

## 6. 生成的文档结构

典型输出结构：

```text
docs/tasks/
  homepage-popup-rules/
    project-progress-ledger.md
    plan.md
    steps.md
    next-prompt.md
    prompts/
      step-01-read-and-map.md
    review-checklist.md
    handoff.md
```

其中：

- `homepage-popup-rules/` 是单个大任务的独立文档目录；每个新大任务都应创建自己的 `{task_slug}/` 目录。
- `project-progress-ledger.md` 放在对应任务目录内，记录该大任务的长期进度、阻塞和任务索引。
- `plan.md` 记录单个任务的目标、边界、风险、代码现状和方案。
- `steps.md` 记录小步执行计划。
- `next-prompt.md` 保存当前下一步执行 Prompt 的入口，指向 `prompts/` 中当前应执行的 Prompt。
- `prompts/` 保存每次生成过的执行 Prompt 历史。执行当前步骤时默认不要读取旧 Prompt，避免额外上下文干扰。
- `review-checklist.md` 记录人工 Review 要点。
- `handoff.md` 记录单任务执行交接状态。

## 7. 模板文件

Skill 内置这些模板：

```text
nanyi-ai-coding-planner-skill/templates/project-progress-ledger-template.md
nanyi-ai-coding-planner-skill/templates/task-plan-template.md
nanyi-ai-coding-planner-skill/templates/task-handoff-template.md
```

生成任务文档时，Skill 会优先参考这些模板结构，再结合项目真实代码和项目规则填写内容。

## 8. 项目命令应该放在哪里

具体项目的安装、构建、测试、Lint、类型检查和发布命令不应该写死在 Skill 规则里。

这些命令应该来自项目本身，例如：

- `CLAUDE.md`
- `AGENTS.md`
- `README`
- `Makefile`
- CI 配置
- 其他项目级规则文件

如果项目没有提供验证命令，Skill 应该明确说明未找到，而不是编造某种语言或框架的默认命令。

## 9. 推荐使用方式

- 在较大功能、重构、Bug 修复或跨模块改造开始前先使用本 Skill。
- 为所有任务指定统一的任务集合根目录，例如 `docs/tasks` 或 `.ai_temp/docs/tasks`；每个大任务会在该根目录下创建独立 `{task_slug}/` 文件夹。
- 每一轮执行后要求执行者更新 `handoff.md`。
- 人工确认当前步骤后，再生成下一步 Prompt。
- 旧 Prompt 保存在任务的 `prompts/` 目录供人工追溯；除非需要诊断或审计，不要把旧 Prompt 提供给下一轮执行。
- 如果任务发生暂停、阻塞或目标变化，同步更新该任务目录内的 `project-progress-ledger.md`。
- 每步代码修改范围会尽量控制得比较小，建议自己审一遍；也可以把 `docs/tasks` 下的规划、进度和 check list 单独交给其他 AI 做辅助 Review。

## 10. 安全说明

- 不要把账号、密钥、Token、生产环境配置或敏感数据写入任务文档。
- 不要把私有路径、内部项目名或公司信息写入公开仓库。
- 涉及支付、权限、隐私、安全、数据库结构或公开 API 契约时，应先明确边界。
- 需要新增依赖、删除大量代码或修改生产配置时，应先获得用户确认。

## 11. 适合场景

- 复杂功能开发前的工程规划
- 现有功能上的规则迭代或边界补齐
- 跨模块重构前的影响范围分析
- 长任务拆分为多轮 AI Coding
- 需要人工 Review 的渐进式开发
- 需要保留任务进度总账和单任务交接记录的项目

## 12. 不适合场景

- 一两行即可完成的小改动
- 用户已经要求跳过规划并直接实现的任务
- 不需要代码扫描或交接文档的临时问题
- 需要立即执行线上操作、访问密钥或读取生产数据的任务

## 13. License

MIT License. See [LICENSE](LICENSE).
