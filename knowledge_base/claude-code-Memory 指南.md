# Memory 指南

`memory` 主要通过 `CLAUDE.md` 持久化项目规则。  
它不是当前对话的临时上下文，而是 Claude 进入项目时默认携带的长期规则。

---

## 什么时候该先配 memory

出现以下任一情况就该尽快配置：

- 你反复说明同一套代码规范
- 团队有固定测试、命名、提交流程
- 项目目录复杂，需要提前说明重点路径
- 有长期稳定的项目知识希望持续生效

---

## 常用命令

| 命令 / 写法 | 用途 |
|-------------|------|
| `/init` | 初始化项目 memory |
| `/memory` | 查看或编辑 memory |
| `# your rule` | 快速写入一条规则 |
| `# remember this` | 用自然语言追加记忆 |
| `@README.md` | 在 `CLAUDE.md` 中引用文档 |

---

## memory 放在哪

- **项目级**：`./CLAUDE.md` 或 `.claude/CLAUDE.md`  
  放项目规则、目录说明、测试标准、提交规范。
- **个人级**：`~/.claude/CLAUDE.md`  
  放个人偏好和常用工作方式。
- **目录级**：子目录下的 `CLAUDE.md`  
  适合 monorepo 或大项目的局部细则。

---

## 写什么最有价值

优先写“高影响、长期稳定、可执行”的信息：

- 关键目录与边界（哪些能改、哪些不要动）
- 容易遗漏的硬性规则（如提交前必须跑测试）
- 工具链约定（如 `npm` / `pnpm` / `uv`）
- 最低验证标准（lint、test、build 到什么程度算通过）

可直接参考的最小模板：

```md
# Project Memory

## Project Overview
- This is a TypeScript web application.

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

## 哪些内容不适合写进 memory

- 过长、每次都不一定相关的大段背景知识
- 会频繁变化的实时数据
- 明显更适合做成 skill 或 hook 的工作流细节
- 会影响运行的命令名或配置 key 的中文重命名

如果你发现某段内容更像“流程模板”，通常更适合去做 skill，而不是塞进 `CLAUDE.md`。

---

## 中国用户特别注意

- 如果你在 Windows 上工作，路径规则和 shell 说明最好明确写清楚。
- 如果项目依赖 `uv`、`npm`、`pnpm`、`bun` 等特定工具，也建议写入 memory。
- 如果项目所在团队有 GitHub、内网、代理、镜像源要求，也值得写在 memory 里。

---

## 常见坑

- **越长越好**：错误。应保持短、准、稳定。
- **项目规则和个人偏好混写**：协作成本高，建议分层。
- **规则过期不更新**：会让 Claude 持续使用旧约定。

# 以下提供一些样例：

# 1.API 模块规则（）

这个文件用于覆盖根目录 `CLAUDE.md`，作用范围是 `/src/api/` 下的内容。

## API 专属规范

### Request Validation

- 使用 Zod 做 schema validation
- 所有输入都必须校验
- 校验失败返回 400
- 错误信息尽量提供字段级细节

### Authentication

- 所有 endpoint 默认需要 JWT token
- token 放在 `Authorization` header
- token 默认 24 小时过期
- 实现 refresh token 机制

### Response Format

所有成功响应统一遵循下面的结构：

```json
{
  "success": true,
  "data": { /* actual data */ },
  "timestamp": "2025-11-06T10:30:00Z",
  "version": "1.0"
}
```

错误响应：

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "User message",
    "details": { /* field errors */ }
  },
  "timestamp": "2025-11-06T10:30:00Z"
}
```

### Pagination

- 使用 cursor-based pagination
- 返回 `hasMore`
- 单页最大 100
- 默认每页 20

### Rate Limiting

- 登录用户每小时 1000 次
- 公共接口每小时 100 次
- 超限返回 429
- 包含 `retry-after` header

### Caching

- 使用 Redis 做 session caching
- 默认缓存 5 分钟
- 写操作后主动失效
- cache key 带资源类型标签

# 2.我的开发偏好（personal-CLAUDE.md）

## About Me

- **Experience Level**: 8 年全栈开发经验
- **Preferred Languages**: TypeScript、Python
- **Communication Style**: 直接、清晰、带例子
- **Learning Style**: 喜欢图示和代码结合

## Code Preferences

### Error Handling

偏好显式错误处理，尽量使用清晰的 `try-catch` 和有意义的错误信息。  
避免过于泛化的错误；排查时尽量保留日志上下文。

### Comments

注释优先解释 **WHY**，不要只是重复代码在做什么。  
更适合解释业务逻辑或不明显的决策。

### Testing

偏好 TDD。  
尽量先写测试，再实现。  
测试关注行为，而不是实现细节。

### Architecture

偏好模块化、低耦合设计。  
重视可测试性和职责分离。

## Debugging Preferences

- `console.log` 建议带 `[DEBUG]` 前缀
- 日志包含函数名和关键变量
- 能给 stack trace 时尽量给
- 日志里尽量保留时间戳

## Communication

- 复杂概念先给例子再讲理论
- 需要时提供 before / after 代码片段
- 长回答最后给一个简要总结

## Project Organization

我常用的项目结构：

```text
project/
  ├── src/
  │   ├── api/
  │   ├── services/
  │   ├── models/
  │   └── utils/
  ├── tests/
  ├── docs/
  └── docker/
```

## Tooling

- **IDE**: VS Code + vim keybindings
- **Terminal**: Zsh + Oh-My-Zsh
- **Format**: Prettier（100 字符换行）
- **Linter**: ESLint + airbnb config
- **Test Framework**: Jest + React Testing Library

# 3.项目配置

## Project Overview

- **Name**: E-commerce Platform
- **Tech Stack**: Node.js, PostgreSQL, React 18, Docker
- **Team Size**: 5 developers
- **Deadline**: Q4 2025

## Architecture

@docs/architecture.md  
@docs/api-standards.md  
@docs/database-schema.md

## Development Standards

### Code Style

- 使用 Prettier
- 使用 ESLint + airbnb config
- 最大行长 100
- 2-space indentation

### Naming Conventions

- **Files**: kebab-case
- **Classes**: PascalCase
- **Functions/Variables**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Database Tables**: snake_case

### Git Workflow

- 分支命名：`feature/description` 或 `fix/description`
- 提交信息遵循 conventional commits
- 合并前需要 PR
- 所有 CI/CD 检查必须通过
- 至少 1 个 approval

### Testing Requirements

- 最低 80% 覆盖率
- 关键路径必须有测试
- unit tests 用 Jest
- E2E 用 Cypress
- 文件命名：`*.test.ts` 或 `*.spec.ts`

### API Standards

- 使用 RESTful endpoints
- JSON request / response
- 正确使用 HTTP status code
- API 统一带版本：`/api/v1/`
- 所有 endpoint 要有示例文档

### Database

- schema 变更必须走 migration
- 不允许硬编码凭证
- 使用 connection pooling
- 开发环境开启 query logging
- 定期备份

### Deployment

- Docker-based deployment
- Kubernetes orchestration
- blue-green deployment
- 失败自动回滚
- deploy 前先跑数据库迁移

## Common Commands

| Command | Purpose |
|---------|---------|
| `npm run dev` | 启动开发服务 |
| `npm test` | 运行测试 |
| `npm run lint` | 检查代码风格 |
| `npm run build` | 生产构建 |
| `npm run migrate` | 执行数据库迁移 |

## Team Contacts

- Tech Lead: Sarah Chen (`@sarah.chen`)
- Product Manager: Mike Johnson (`@mike.j`)
- DevOps: Alex Kim (`@alex.k`)

## Known Issues & Workarounds

- PostgreSQL 连接池高峰期限制为 20
- 临时方案：实现 query queuing
- Safari 14 对 async generators 有兼容性问题
- 临时方案：使用 Babel transpiler

## Related Projects

- Analytics Dashboard: `/projects/analytics`
- Mobile App: `/projects/mobile`
- Admin Panel: `/projects/admin`