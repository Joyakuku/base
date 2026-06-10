# Memory 指南

`memory` 主要通过 `CLAUDE.md`
持久化项目规则。

它不是当前对话里的临时上下文，
而是 Claude 进入项目时默认携带的长期规则。

---

## 工作流程

配置 memory 时，
推荐按下面顺序推进：

1. 确认规则适合长期复用。
2. 判断应该放在项目级、目录级还是个人级。
3. 用短句写清稳定规则和验证标准。
4. 避免写入一次性背景和易变化信息。
5. 在真实任务中观察是否被正确遵守。
6. 定期清理过期规则。

---

## 常用概念

### Project Memory

项目级 memory 通常放在项目根目录的 `CLAUDE.md`。

它适合记录项目结构、开发规则、
测试要求和提交约定。

### Personal Memory

个人级 memory 通常放在 `~/.claude/CLAUDE.md`。

它适合记录个人偏好、
常用沟通方式和稳定工作习惯。

### Directory Memory

目录级 memory 放在子目录中的 `CLAUDE.md`。

它适合 monorepo 或大项目中的局部规则。

### Import

`@README.md` 这类引用可以把其它文档纳入 memory。

适合引用相对稳定的项目说明，
不适合引用经常变动的大文档。

---

## 适用场景

出现以下任一情况，
就应该尽快配置 memory：

- 你反复说明同一套代码规范。
- 团队有固定测试、命名、提交流程。
- 项目目录复杂，需要提前说明重点路径。
- 有长期稳定的项目知识希望持续生效。

---

## 常用命令

`/init`

初始化项目 memory。

`/memory`

查看或编辑 memory。

`# your rule`

快速写入一条规则。

`# remember this`

用自然语言追加记忆。

`@README.md`

在 `CLAUDE.md` 中引用文档。

---

## 存放位置

项目级：

```text
./CLAUDE.md
.claude/CLAUDE.md
```

适合放项目规则、目录说明、
测试标准和提交规范。

个人级：

```text
~/.claude/CLAUDE.md
```

适合放个人偏好和常用工作方式。

目录级：

```text
<subdir>/CLAUDE.md
```

适合 monorepo 或大项目的局部细则。

---

## 写什么最有价值

优先写高影响、长期稳定、
并且可执行的信息。

建议包含：

- 关键目录与边界。
- 哪些文件能改，哪些不要动。
- 容易遗漏的硬性规则。
- 提交前必须运行的测试或检查。
- 工具链约定。
- 最低验证标准。

工具链示例：

- `npm`
- `pnpm`
- `uv`
- `bun`

验证标准示例：

- lint 需要通过。
- test 需要通过。
- build 需要通过。

---

## 最小模板

```md
# Project Memory

## Project Overview

- TypeScript web application.

## Development Rules

- Run tests before commit.
- Prefer async/await.
- Keep API changes documented.

## Important Paths

- `src/` main application code
- `tests/` automated tests
- `docs/` documentation
```

---

## 不适合写入 memory

以下内容不建议写进 `CLAUDE.md`：

- 过长的大段背景知识。
- 每次任务不一定相关的信息。
- 会频繁变化的实时数据。
- 更适合做成 skill 的流程模板。
- 更适合做成 hook 的自动化流程。
- 会影响运行的命令名中文翻译。
- 会影响配置的 key 中文翻译。

如果某段内容更像流程模板，
通常更适合做成 skill，
而不是塞进 `CLAUDE.md`。

---

## 中国用户特别注意

- Windows 项目要写清路径规则。
- Windows 项目要写清 shell 环境。
- 特定包管理器应写入 memory。
- GitHub、内网、代理要求应写清。
- 镜像源、私有 registry 也值得记录。

---

## 常见坑

越长越好：

错误。
memory 应保持短、准、稳定。

项目规则和个人偏好混写：

协作成本会升高。
建议按层级拆分。

规则过期不更新：

Claude 会持续使用旧约定。
需要定期维护。

---

# 样例

下面给出几类常见 `CLAUDE.md` 样例。

---

## 样例 1：API 模块规则

该文件用于覆盖根目录 `CLAUDE.md`。

作用范围：

```text
/src/api/
```

### Request Validation

- 使用 Zod 做 schema validation。
- 所有输入都必须校验。
- 校验失败返回 `400`。
- 错误信息尽量提供字段级细节。

### Authentication

- 所有 endpoint 默认需要 JWT token。
- token 放在 `Authorization` header。
- token 默认 `24` 小时过期。
- 实现 refresh token 机制。

### Response Format

成功响应统一格式：

```json
{
  "success": true,
  "data": {
    "id": "example"
  },
  "timestamp": "2025-11-06T10:30Z",
  "version": "1.0"
}
```

错误响应统一格式：

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "User message",
    "details": {
      "field": "error"
    }
  },
  "timestamp": "2025-11-06T10:30Z"
}
```

### Pagination

- 使用 cursor-based pagination。
- 返回 `hasMore`。
- 单页最大 `100`。
- 默认每页 `20`。

### Rate Limiting

- 登录用户每小时 `1000` 次。
- 公共接口每小时 `100` 次。
- 超限返回 `429`。
- 响应包含 `retry-after` header。

### Caching

- 使用 Redis 做 session caching。
- 默认缓存 `5` 分钟。
- 写操作后主动失效。
- cache key 带资源类型标签。

---

## 样例 2：个人开发偏好

文件名示例：

```text
personal-CLAUDE.md
```

### About Me

- Experience Level: 8 年全栈开发经验。
- Preferred Languages:
  TypeScript、Python。
- Communication Style: 直接、清晰、带例子。
- Learning Style: 喜欢图示和代码结合。

### Code Preferences

Error Handling：

偏好显式错误处理。

尽量使用清晰的 `try-catch`
和有意义的错误信息。

避免过于泛化的错误。

排查时尽量保留日志上下文。

Comments：

注释优先解释 why。

不要只是重复代码在做什么。

更适合解释业务逻辑，
或不明显的技术决策。

Testing：

偏好 TDD。

尽量先写测试，再实现。

测试关注行为，
而不是实现细节。

Architecture：

偏好模块化、低耦合设计。

重视可测试性和职责分离。

### Debugging Preferences

- `console.log` 建议带 `[DEBUG]` 前缀。
- 日志包含函数名和关键变量。
- 能给 stack trace 时尽量给。
- 日志里尽量保留时间戳。

### Communication

- 复杂概念先给例子再讲理论。
- 需要时提供 before / after 代码片段。
- 长回答最后给一个简要总结。

### Project Organization

常用项目结构：

```text
project/
  src/
    api/
    services/
    models/
    utils/
  tests/
  docs/
  docker/
```

### Tooling

- IDE: VS Code + vim keybindings。
- Terminal: Zsh + Oh-My-Zsh。
- Format: Prettier，100 字符换行。
- Linter: ESLint + airbnb config。
- Test:
  Jest + React Testing Library。

---

## 样例 3：项目配置

### Project Overview

- Name: E-commerce Platform。
- Tech Stack: Node.js、PostgreSQL。
- Frontend: React 18。
- Runtime: Docker。
- Team Size: 5 developers。
- Deadline: Q4 2025。

### Architecture

```md
@docs/architecture.md
@docs/api-standards.md
@docs/database-schema.md
```

### Code Style

- 使用 Prettier。
- 使用 ESLint + airbnb config。
- 最大行长 `100`。
- 使用 2-space indentation。

### Naming Conventions

- Files: kebab-case。
- Classes: PascalCase。
- Functions: camelCase。
- Variables: camelCase。
- Constants: UPPER_SNAKE_CASE。
- Database Tables: snake_case。

### Git Workflow

- 分支命名使用 `feature/description`。
- 修复分支使用 `fix/description`。
- 提交信息遵循 conventional commits。
- 合并前需要 PR。
- 所有 CI/CD 检查必须通过。
- 至少需要 `1` 个 approval。

### Testing Requirements

- 最低 `80%` 覆盖率。
- 关键路径必须有测试。
- unit tests 使用 Jest。
- E2E 使用 Cypress。
- 文件命名为 `*.test.ts`。
- 也可以使用 `*.spec.ts`。

### API Standards

- 使用 RESTful endpoints。
- 使用 JSON request / response。
- 正确使用 HTTP status code。
- API 统一带版本：`/api/v1/`。
- 所有 endpoint 要有示例文档。

### Database

- schema 变更必须走 migration。
- 不允许硬编码凭证。
- 使用 connection pooling。
- 开发环境开启 query logging。
- 定期备份。

### Deployment

- 使用 Docker-based deployment。
- 使用 Kubernetes orchestration。
- 使用 blue-green deployment。
- 失败自动回滚。
- deploy 前先跑数据库迁移。

### Common Commands

`npm run dev`

启动开发服务。

`npm test`

运行测试。

`npm run lint`

检查代码风格。

`npm run build`

生产构建。

`npm run migrate`

执行数据库迁移。

### Team Contacts

- Tech Lead:
  Sarah Chen，`@sarah.chen`。
- Product Manager:
  Mike Johnson，`@mike.j`。
- DevOps: Alex Kim，`@alex.k`。

### Known Issues

- PostgreSQL 连接池高峰期限制为 `20`。
- 临时方案：实现 query queuing。
- Safari 14 对 async generators
  有兼容问题。
- 临时方案：使用 Babel transpiler。

### Related Projects

- Analytics Dashboard:
  `/projects/analytics`。
- Mobile App: `/projects/mobile`。
- Admin Panel: `/projects/admin`。

---

## 速查

1. `CLAUDE.md`
   项目级 memory 的常见文件名。

2. `/init`
   初始化项目 memory。

3. `/memory`
   查看或编辑 memory。

4. `# your rule`
   快速写入一条规则。

5. `@README.md`
   在 memory 中引用其它文档。

6. `~/.claude/CLAUDE.md`
   个人级 memory 的常见位置。

---

## 后续扩展复习点

1. Memory 分层策略
   继续整理项目级、目录级、个人级 memory 的边界。

2. Memory 和 Skill 的区别
   复习什么时候写长期规则，
   什么时候沉淀成可执行工作流。

3. Memory 维护周期
   建立定期清理过期规则的检查清单。

4. 团队协作规范
   整理多人项目中哪些规则适合进入共享 memory。

5. Windows 项目约定
   补充 shell、路径、编码、换行符等本地环境规则。
