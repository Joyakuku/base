# Skills 指南

`skills` 通过 `SKILL.md`
封装可复用工作流。

它不是一次性 prompt，
而是可长期复用、可共享、
可按需加载的能力单元。

---

## 工作流程

创建 skill 时，
推荐按下面顺序推进：

1. 先确认任务会反复出现。
2. 写清触发条件和不适用场景。
3. 把执行步骤写成可操作清单。
4. 把固定格式放入 `templates/`。
5. 把辅助操作沉淀到 `scripts/`。
6. 把长背景资料放入 `references/`。
7. 用真实任务验证输出是否稳定。

---

## 常用概念

### `SKILL.md`

`SKILL.md` 是 skill 的入口文件。

它应说明触发场景、执行步骤、
输出要求和注意事项。

### Frontmatter

frontmatter 用于声明 skill 元信息。

常见字段：

- `name`
  skill 名称。

- `description`
  skill 触发描述。

### Templates

`templates/` 存放固定输出结构。

适合报告、审查清单、交付包目录等稳定格式。

### Scripts

`scripts/` 存放可复用脚本。

适合渲染、校验、生成、批处理等机械步骤。

### References

`references/` 存放参考资料。

适合长规则、示例、背景知识和术语表。

---

## 适用场景

出现以下任一情况，
优先沉淀成 skill：

- 你反复执行同一类任务。
- 输出格式需要长期保持一致。
- 需要模板、脚本、参考资料配合执行。
- 希望团队成员都按同一流程完成任务。

常见任务示例：

- 代码审查。
- 重构。
- 文档生成。
- 实验报告生成。
- 固定格式的项目交付。

---

## 基本结构

```text
skill-name/
  SKILL.md
  templates/
  scripts/
  references/
```

`SKILL.md`

定义触发场景、执行步骤、
输出要求和注意事项。

`templates/`

存放固定输出模板。

`scripts/`

存放辅助脚本。

`references/`

存放规则、背景资料或示例文档。

---

## 存放位置

个人级：

```text
~/.claude/skills/
  <skill-name>/SKILL.md
```

适合个人习惯工作流。

项目级：

```text
.claude/skills/
  <skill-name>/SKILL.md
```

适合团队共享流程。

插件级：

```text
<plugin>/skills/
  <skill-name>/SKILL.md
```

适合随插件分发能力。

---

## 写什么最有价值

优先写可触发、可执行、
并且可验收的信息。

一个好的 skill 至少应该说明：

- 什么时候使用。
- 什么时候不要使用。
- 执行步骤是什么。
- 需要读取哪些模板或资料。
- 需要运行哪些脚本。
- 最终输出应长什么样。
- 完成后如何验证。

---

## 加载方式

可以简单理解为三步：

1. Claude 先知道有哪些 skills。
2. 需要某个 skill 时，读取 `SKILL.md`。
3. 只有必要时，继续读取模板和资料。

因此可以安装很多 skills，
而不会在一开始就塞满上下文。

---

## 最小模板

```md
---
name: code-review
description: Review changed files
  and report risks by severity.
---

# Code Review

## Trigger

Use when user asks for code review.

## Steps

1. Inspect changed files.
2. Identify risks and regressions.
3. Provide findings with severity.

## Output

- Ordered findings.
- Open questions.
- Suggested fixes.
```

---

## 不适合写进 skill

以下内容不建议写进 skill：

- 只在单次任务里有效的临时提示词。
- 纯背景介绍，缺少执行步骤的长文。
- 触发条件模糊的描述。
- 无法判断何时使用的描述。
- 把 frontmatter key 随意翻译。
- 把 skill 名称随意改名。

---

## 中国用户特别注意

- 使用脚本时先明确 shell 环境。
- PowerShell、Git Bash、WSL 要分清。
- 依赖 `python` 时写清版本。
- 依赖 `node` 时写清版本。
- 依赖 `uv`、`npm` 时写清前置条件。
- 跨平台路径差异要显式标注。
- 换行符和权限差异也建议说明。

---

## 常见坑

description 太泛：

Claude 难以正确触发，
容易漏用或误用。

一个 skill 包太多事：

建议一类问题一个 skill，
保持聚焦。

结构不清晰：

缺少触发、步骤、输出三段时，
执行稳定性会下降。

---

## 编写建议

名称要具体：

`code-review` 比 `helper` 更容易触发。

描述要像触发条件：

说明用户提出什么需求时，
应该使用这个 skill。

步骤要可执行：

避免只写原则。
尽量写清先做什么，
再做什么，
最后交付什么。

资料要分层：

`SKILL.md` 保持短。
大段规则放进 `references/`。
固定格式放进 `templates/`。
复杂操作放进 `scripts/`。

---

## 速查

1. `SKILL.md`
   skill 的入口说明文件。

2. `name`
   skill 的稳定名称。

3. `description`
   触发条件描述，越具体越好。

4. `templates/`
   存放固定输出模板。

5. `scripts/`
   存放辅助脚本。

6. `references/`
   存放长参考资料。

7. `Trigger`
   说明什么时候应该使用 skill。

8. `Steps`
   说明执行顺序。

9. `Output`
   说明最终交付格式。

---

## 后续扩展复习点

1. Skill 粒度设计
   继续整理一个 skill 应覆盖多大范围，
   什么时候需要拆分。

2. Description 写法
   练习把描述写成可触发条件，
   避免过泛导致误用。

3. Skill 与 Memory 分工
   复习长期规则、工作流、模板、脚本分别放在哪里。

4. 参考资料分层
   整理 `SKILL.md`、`templates/`、
   `scripts/`、`references/` 的边界。

5. 跨平台脚本
   补充 PowerShell、Bash、Python、
   Node.js 脚本在不同系统中的注意点。
