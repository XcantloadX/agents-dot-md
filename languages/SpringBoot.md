# Spring Boot 代码规范

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## 覆盖规则

总体代码规范依照 Spring Boot 官方参考文档，并继承同目录 `JAVA.md` 的语言级要求。**严格 MUST** 按照如下顺序覆盖：

用户提示词要求 > 本规范要求 > `JAVA.md` 要求 > Spring Boot 官方默认

## 分层与包结构

- **应当 SHOULD** 按 Controller（接口层） -> Service（领域层） -> Repository / Infrastructure（基础设施层）分层，**禁止 MUST NOT** 跨层直调（Controller 直调 Repository、跨 feature 调他域 Repository）。
- 包**优先 RECOMMENDED** 按 feature 聚合（同一 feature 的 controller、service、repository、DTO、异常放一包），不按技术类型建巨型 `controller/` / `service/` 包。
- DTO 仅**可 MAY** 出现在 Controller 层，Service **应当 SHOULD** 操作实体模型；DTO 命名**应当 SHOULD** 以 `Dto` 后缀并提供 `toEntity` 转换。
- Repository 接口可见性**应当 SHOULD** 收敛到 feature 包内，不被其他 feature 直接依赖。

## Bean 与注入

- 注入**必须 MUST** 用构造器注入，**禁止 MUST NOT** 字段注入与 `new` 手工组装有状态 Bean。
- 外部系统客户端、配置类、条件装配**优先 RECOMMENDED** 用显式 `@Configuration` 定义 Bean，领域服务**可 MAY** 用 stereotype 注解；是否禁用 `@Component` / `@Service` 扫描由项目统一，不混用两种风格。
- Bean 默认单例，含可变状态的 Bean **必须 MUST** 说明线程安全策略；request / session 作用域**不应该 SHOULD NOT** 在无必要时引入。

## 事务与服务

- 事务**必须 MUST** 开在 Service 层，Controller **禁止 MUST NOT** 加 `@Transactional`；只读查询**应当 SHOULD** 标记 `readOnly = true`。
- 多个 Service 调用同属一个业务事务时**应当 SHOULD** 由外层 Service 编排，不在 Controller 拼事务。
- 领域校验失败用 4xx 异常，系统故障用 5xx 异常，Service **禁止 MUST NOT** 返回 HTTP 语义对象。

## Controller 与 API

- 返回值**应当 SHOULD** 用 `ResponseEntity` 显式表达状态码与头；资源已存在返回 409，不存在返回 404，请求非法返回 400，服务端故障返回 500。
- 入参**必须 MUST** 用 Bean Validation（`@Valid` / `@NotBlank` / `@Min` 等），Controller **必须 MUST** 有统一 `@ControllerAdvice` + `@ExceptionHandler` 收口。
- 设计**应当 SHOULD** 遵循 RESTful（名词复数、HTTP 动词表语义），文档**推荐 RECOMMENDED** 用 Springdoc OpenAPI 生成。
- 列表查询**优先 RECOMMENDED** 键集分页（seek），大数据量**不应该 SHOULD NOT** 默认用 offset 分页。

## 配置与环境

- 类型安全配置**必须 MUST** 用 `@ConfigurationProperties`，**禁止 MUST NOT** 满屏 `@Value` 散装取值。
- 配置文件用 `application.yaml` 按域分组，用 profile（dev / prod）隔离环境，敏感信息**禁止 MUST NOT** 提交到仓库。
- 超时、重试、连接池等运维参数**必须 MUST** 可配置，不硬编码。

## 外部调用与韧性

- 外部 HTTP **应当 SHOULD** 用 `RestClient`（Boot 3.2+）或 `WebClient`，每个外部系统一个专用 Client 类，放在所属 feature 的基础设施侧。
- 连接超时、读取超时、整体超时**必须 MUST** 同时设置；Client **必须 MUST** 经 `@Configuration` 创建，不在业务代码里 `new`。
- 外部异常**必须 MUST** 转译为领域异常，不把原始 HTTP 客户端异常抛到 Controller 层；下游不可用时按约定返回 503 / 504。
- 重试、熔断、限流**推荐 RECOMMENDED** 用 Resilience4j 注解实现，日志中记录 URL、状态码与上下文。

## 数据与迁移

- 表名复数 + `snake_case`，主键 `id`（`BIGSERIAL` 或 UUID），审计列 `created_at` / `updated_at`（`TIMESTAMPTZ`），外键 `<单数表名>_id` 并建索引。
- JSON 列用 `JSONB`，枚举用 `VARCHAR + CHECK`，约束命名加 `pk_` / `fk_` / `uq_` / `chk_` / `idx_` 前缀。
- 结构变更**必须 MUST** 走 Flyway 版本化迁移，文件名 `V<YYYYMMDDHHmm>_描述.sql`，**禁止 MUST NOT** 修改已上线迁移文件。
- JPA 实体**应当 SHOULD** 显式 `@Table(name = ...)`，列名与库表显式对齐。

## 日志与可观测

- 日志用 SLF4J：修改记 INFO，查询记 DEBUG，可恢复记 WARN，不可恢复记 ERROR；Service 层记业务日志，不在每层重复记同一错误。
- **禁止 MUST NOT** 记录用户隐私（用户 ID、邮箱、手机号等），用聚合指标或匿名标识代替。
- 生产**推荐 RECOMMENDED** 启用 Actuator（health / metrics），跨服务追踪用 Micrometer Tracing 透传 traceId。

## 测试

- Controller **必须 MUST** 有 `@WebMvcTest` + MockMvc 测试，覆盖正常、校验失败（400）、不存在（404）、冲突（409）与分页参数，并 `verify()` Service 调用。
- Service **必须 MUST** 有 Mockito 单元测试，覆盖正常、边界与异常；Repository **应当 SHOULD** 有集成测试基类派生的 `*IT`。
- JSON 模型**应当 SHOULD** 有序列化 / 反序列化测试；新功能不带测试**不应该 SHOULD NOT** 合并。
