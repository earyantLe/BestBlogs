# BestBlogs.dev - Development Conventions

> 版本：v2.5.0
> 最后更新：2026-07-08
> 状态：公开同步版活文档

## 1. 代码风格

### 1.1 TypeScript / React

| 规则 | 要求 |
| --- | --- |
| 导出方式 | 组件和工具函数优先具名导出 |
| Props 类型 | 复杂组件必须定义 Props interface / type |
| 样式 | 优先使用 Tailwind CSS 和项目已有设计 token |
| 条件类 | 优先使用项目既有 class merge 工具 |
| 图标 | 优先复用现有图标库 |
| i18n | 用户可见文案进入 i18n 或内容文件，不在组件中散落硬编码长文案 |

```typescript
interface ButtonProps {
  variant: 'primary' | 'secondary'
  onClick?: () => void
}

export function Button({ variant, onClick }: ButtonProps) {
  return null
}
```

### 1.2 Java

| 规则 | 要求 |
| --- | --- |
| 版本特性 | 优先使用当前 JDK 支持的稳定特性 |
| 命名 | PascalCase 类名，camelCase 方法 / 变量 |
| 注释 | 解释「为什么」，避免重复代码本身 |
| 嵌套 | 超过 3 层缩进时优先提前 return 或拆分方法 |
| 金额 | 金额和计费相关字段使用精确数值类型 |

```java
public record UserDTO(String id, String name) { }

if (user == null) {
    return;
}
```

### 1.3 命名规范

| 层级 | Java | TypeScript |
| --- | --- | --- |
| 组件 / 类 | `*Controller`, `*Service`, `*Model` | PascalCase |
| Hooks | - | `use*` |
| 工具函数 | camelCase | camelCase |
| 常量 | UPPER_SNAKE_CASE | UPPER_SNAKE_CASE |
| 类型 / 接口 | PascalCase | PascalCase |

## 2. 关键约束

### 2.1 API 响应格式

后端接口保持统一 envelope，例如 `Response<T>`；前端 API client 可映射为等价的 `ApiResponse<T>`。新增接口不要各自发明响应格式。

### 2.2 多语言处理

用户可见内容必须明确语言来源和 fallback 策略。后端接口读取语言偏好时复用统一 helper，前端页面复用现有 i18n 体系。

### 2.3 数据查询

- 大批量查询必须分页。
- 列表接口需要稳定排序。
- 不把无界 `findAll()` 暴露给用户请求路径。
- 搜索、推荐和相似内容要有关闭或降级策略。

### 2.4 环境变量安全

- 前端只读取公开前缀变量。
- 密钥、token、连接字符串和私有 endpoint 只在服务端读取。
- 示例代码使用占位符，不写看似真实的密钥。

```typescript
const publicBaseUrl = process.env.NEXT_PUBLIC_API_BASE_URL
```

### 2.5 产品红线

任何前端、后端、文案和运营改动都不得触犯 `1-VISION.md` 的五条红线：

- 不以时间消耗为目标；
- 不替用户完成阅读、理解、思考和判断；
- 公共策展层质量门槛不向大众化让步；
- 不以量取胜；
- 不做泛化 AI 工具箱。

## 3. 模块边界

### 3.1 后端分层

```text
Controller
  -> Application Service
  -> Domain Service / Model
  -> Repository / Adapter
```

禁止：

- Controller 直接操作复杂领域状态；
- Domain 层依赖 Infrastructure 实现；
- 为了临时需求绕过服务层；
- 在多个模块复制同一份业务规则。

### 3.2 内容工作流

- 前台只消费完成态或明确可展示的降级态数据。
- 状态流转由领域 / 编排层管理。
- Controller 不直接改工作流状态。
- 私有内容不得进入公共入口。

### 3.3 文件组织

- 单文件过大时优先拆分为领域清晰的小模块。
- 新抽象必须减少真实复杂度，而不是只为了复用而复用。
- 共享工具进入 common / lib 前，需要确认至少两个稳定调用场景。

## 4. 安全规范

- 用户可控输入必须校验。
- 用户可控重定向必须使用安全重定向 helper，只允许同站相对路径。
- 前端请求敏感后端 API 时走受控代理层。
- Token 不进入 localStorage。
- 日志和事件不写完整 token、邮箱、用户 ID、连接字符串或长堆栈。
- 上传、抓取、URL 探测和外部回调必须评估 SSRF、重定向绕过和超时。

## 5. 测试规范

测试策略见 `9-TESTING.md`。开发时至少遵循：

- 核心领域逻辑优先补单元测试；
- 涉及权限、Pro、配额、计费、隐私和外部调用的变更必须覆盖失败 / 关闭态；
- 修复线上或已知回归时补最小复现测试；
- 不使用真实密钥、真实生产账号或不可重复的线上数据；
- 前端测试断言用户可见结果，不只断言内部状态。

## 6. Git 与提交规范

提交信息遵循 Conventional Commits：

```text
<type>(<scope>): <subject>

[optional body]

Refs: <task-or-issue>
```

常用 type：

| Type | 用途 |
| --- | --- |
| `feat` | 新功能 |
| `fix` | Bug 修复 |
| `refactor` | 不改变行为的重构 |
| `perf` | 性能优化 |
| `docs` | 文档变更 |
| `test` | 测试补充 |
| `style` | 样式或格式 |
| `ops` | 运维、配置、开关 |
| `chore` | 构建工具、依赖升级 |

推荐 scope 对齐产品结构：

- `briefing`
- `weekly`
- `topic`
- `explore`
- `my-brief`
- `follow`
- `reading`
- `review`
- `copilot`
- `translate`
- `interest`
- `pro`
- `search`
- `openapi`
- `landing`
- `settings`
- `auth`
- `infra`

每个提交只做一件事，方便回溯、review 和 changelog 生成。

## 7. 可观测性约定

涉及外部调用、Job、告警、Pro、配额、计费或核心漏斗时，必须遵循 `11-OPERATIONS.md`：

- 事件名集中管理，避免散落硬编码；
- 外部调用记录成功、失败、超时和限流；
- Job 记录开始、成功、失败、跳过和耗时；
- 告警通过统一分发入口触发；
- 指标命名稳定，不把高基数字段放进标签；
- 敏感字段必须脱敏。

## 8. 性能规范

- 用户请求路径避免无界查询和串行慢调用。
- 大列表使用分页、虚拟滚动或渐进加载。
- 缓存必须有失效策略。
- 新索引、新读模型和新缓存需要验证关闭态。

## 9. 文档规范

### 9.1 何时更新文档

| 变化 | 更新位置 |
| --- | --- |
| 产品能力 / 路线 | `2-PRODUCT.md` / `8-CURRENT_STATE.md` |
| 架构边界 / 状态机 | `4-ARCHITECTURE.md` |
| 品牌表达 | `3-BRAND.md` |
| 设计和组件 | `5-DESIGN.md` / `6-UI-SPEC.md` |
| 开发规范 | `7-CONVENTIONS.md` |
| 测试规则 | `9-TESTING.md` |
| 术语 | `10-TERMINOLOGY.md` |
| 运维原则 | `11-OPERATIONS.md` |

### 9.2 中文排版

- 中文内引用短语用 `「」` 或 `『』`。
- 中文和英文之间加半角空格。
- 中文和阿拉伯数字之间加半角空格。
- 百分号 `%` 紧贴数字。

示例：

- 推荐：`Anthropic 官方账号宣布 Claude Code 限额提升`
- 不推荐：`Anthropic官方账号宣布Claude Code限额提升`

### 9.3 内容文件

`bestblogs-app/content/**` 中的 Markdown 内容应：

- 使用稳定 slug；
- 提供 title / description 等 frontmatter；
- 长正文用 Markdown，不把段落级文本塞进 UI JSON；
- 禁止内嵌脚本和不受控 HTML。

## 10. 相关文档

- `1-VISION.md`
- `2-PRODUCT.md`
- `4-ARCHITECTURE.md`
- `5-DESIGN.md`
- `6-UI-SPEC.md`
- `9-TESTING.md`
- `10-TERMINOLOGY.md`
- `11-OPERATIONS.md`
