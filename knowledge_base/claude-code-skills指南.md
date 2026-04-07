# Skills 指南

`skills` 通过 `SKILL.md` 封装可复用工作流。  
它不是一次性 prompt，而是可长期复用、可共享、可按需加载的能力单元。

---

## 什么时候该先做 skill

出现以下任一情况，优先沉淀成 skill：

- 你反复执行同一类任务（审查、重构、文档生成）
- 输出格式需要长期保持一致
- 需要模板、脚本、参考资料配合执行
- 希望团队成员都按同一流程完成任务

---

## skill 的基本结构

```text
skill-name/
├── SKILL.md
├── templates/
├── scripts/
└── references/
```

- `SKILL.md`：定义触发场景、执行步骤、输出要求
- `templates/`：放固定输出模板
- `scripts/`：放辅助脚本
- `references/`：放规则或背景资料

---

## skills 放在哪

| 类型 | 路径 | 适用场景 |
|------|------|----------|
| 个人级 | `~/.claude/skills/<skill-name>/SKILL.md` | 个人习惯工作流 |
| 项目级 | `.claude/skills/<skill-name>/SKILL.md` | 团队共享流程 |
| 插件级 | `<plugin>/skills/...` | 随插件分发能力 |

---

## 写什么最有价值

优先写“可触发、可执行、可验收”的信息：

简单理解：

1. Claude 先只知道有哪些 skills，以及它们大概干什么
2. 真正需要某个 skill 时，再读取 `SKILL.md`
3. 只有在需要时，才进一步读模板、脚本或参考资料

这意味着你可以装很多 skills，而不会一开始就把上下文塞爆。

可直接参考的最小模板：
---
name: code-review
description: Review changed files and report risks by severity.
---

## Trigger
- Use when user asks for code review.

## Steps
1. Inspect changed files.
2. Identify risks and regressions.
3. Provide findings with severity.

## Output
- Ordered findings
- Open questions
- Suggested fixes
```


## 哪些内容不适合写进 skill

- 只在单次任务里有效的临时提示词
- 纯背景介绍、缺少执行步骤的长文
- 触发条件模糊、无法判断何时使用的描述
- 把 frontmatter key 或 skill 名称随意翻译/改名


## 中国用户特别注意

- 使用脚本时先明确 shell 环境（PowerShell / Git Bash / WSL）
- 若依赖 `python`、`node`、`uv`、`npm`，在 skill 内写清前置条件
- 跨平台路径、换行符、权限差异建议在说明中显式标注


## 常见坑

- **description 太泛**：Claude 难以正确触发，容易漏用或误用。
- **一个 skill 包太多事**：建议一类问题一个 skill，保持聚焦。
- **结构不清晰**：缺少触发、步骤、输出三段时执行稳定性会下降。

