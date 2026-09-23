# Java 代码规范

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## 覆盖规则

总体代码规范依照 Google Java Format 与 Effective Java。**严格 MUST** 按照如下顺序覆盖：

用户提示词要求 > 本规范要求 > 构建工具格式化与检查要求

## 语言与类型

- 基线**应当 SHOULD** 为 Java 21 及以上，新代码**优先 RECOMMENDED** 使用 record、sealed 接口、模式匹配 switch 表达数据与分支，不用布尔组合与 `instanceof` 强转链。
- 数据载体**优先 RECOMMENDED** 用 `record`，需要校验时在紧凑构造器内完成；可变实体**应当 SHOULD** 封装并保持不变量。
- `var` 仅**可 MAY** 用于构造器调用等类型显而易见处，字段赋值与返回值类型模糊时**不应该 SHOULD NOT** 使用。
- **禁止 MUST NOT** 使用裸类型与 unchecked 警告放行，泛型边界**应当 SHOULD** 明确；`Optional` **禁止 MUST NOT** 用于字段与方法参数，仅用于返回值表达缺席。
- 常量、集合**优先 RECOMMENDED** 不可变（`List.of` / `Map.of` / `Set.of` / `toList()`），暴露出去的集合**必须 MUST** 防御性拷贝或不可变视图。
- 时间**必须 MUST** 用 `java.time`（`Instant` / `ZonedDateTime`），**禁止 MUST NOT** 用 `Date` / `Calendar` 与整型时间戳传业务语义。

## 命名与文件

- 类、接口、枚举、record **必须 MUST** 用 `UpperCamelCase`；方法、变量、参数 **必须 MUST** 用 `lowerCamelCase`；常量 **必须 MUST** 用 `UPPER_SNAKE_CASE`；包 **必须 MUST** 全小写、无下划线。
- 包名**应当 SHOULD** 按领域组织，不按技术分层堆砌；文件名**必须 MUST** 与顶层公开类型同名。
- 测试类名**应当 SHOULD** 为 `<被测类名>Test`，集成测试**应当 SHOULD** 以 `IT` 后缀区分。

## 类与方法设计

- 方法短小单一，编排方法读起来像目录，复杂条件**应当 SHOULD** 抽取为命名良好的谓词方法。
- 组合优于继承，工具类**必须 MUST** `final` + 私有构造；可变状态**应当 SHOULD** 最小可见，能 `final` 则 `final`。
- 接口**应当 SHOULD** 只在有多个实现或框架要求时创建，实现类命名**不应该 SHOULD NOT** 用 `Impl` 后缀，用技术语义命名（如 `JpaUserRepository` / `InMemoryUserRepository`）。
- **禁止 MUST NOT** 通配符 import 与未使用 import，import 顺序**应当 SHOULD** 为标准库、三方、项目三段。

## 异常与资源

- 业务失败**应当 SHOULD** 用 unchecked 领域异常表达，底层受检异常**必须 MUST** 在边界处翻译并附加上下文，**禁止 MUST NOT** 静默吞掉。
- 资源**必须 MUST** 用 try-with-resources 管理，**禁止 MUST NOT** 在 `finally` 里手写 close 链。
- 日志与抛错二选一，**禁止 MUST NOT** 同一处既 `log.error` 又原样上抛由上层再记一次；错误信息简短、可定位、不贴用户隐私。
- **禁止 MUST NOT** 使用 `System.out` / `printStackTrace` 输出，用 SLF4J 参数化日志。

## 并发

- IO 密集型任务**优先 RECOMMENDED** 用虚拟线程（`newVirtualThreadPerTaskExecutor`），沿用 `Thread` / `Executor` API 即可，不手写线程池参数玄学。
- 虚拟线程中**避免 SHOULD NOT** 频繁 `synchronized` 长临界区与 `ThreadLocal` 传上下文，跨线程上下文**应当 SHOULD** 用显式参数或 `ScopedValue`。
- 共享可变状态**必须 MUST** 有明确归属，并发工具类优先用 `java.util.concurrent` 现成结构，不重复造轮子。
- `parallelStream()` **不应该 SHOULD NOT** 默认使用，无实测数据不得引入。

## Docstring 与注释

- 注释与文档**应 SHOULD** 以精炼的中文编写。
- 公开 API **强烈建议 RECOMMENDED** 写 Javadoc（用途、前置条件、副作用、异常），**不要 SHOULD NOT** 复述签名里已有的类型信息。
- 注释只写 why（非显而易见约束、权衡、上游行为），**禁止 MUST NOT** 逐行复述、提交注释掉的代码。
- 重构 / 移动代码时**应该 SHOULD** 保留有效注释，逻辑变化时**必须 MUST** 同步更新。

## 测试

- 单元测试**推荐 RECOMMENDED** JUnit 5 + Mockito + AssertJ，用 `Arrange-Act-Assert`，命名**应当 SHOULD** 为 `method_shouldBehavior_whenCondition`。
- 需要构造特殊实体时**禁止 MUST NOT** 用反射设字段，用测试子类 / 工厂方法显式构造。
- 外部依赖一律 mock / fake，**不应该 SHOULD NOT** 在单元测试中启动真实网络与中间件。
