# Python 代码规范

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## 覆盖规则

总体代码规范依照 PEP 8。**严格 MUST** 按照如下顺序覆盖：

用户提示词要求 > 本规范要求 > PEP8 要求

## 代码结构与风格

- 优先使用单引号字符串；需要包含单引号或多行文本时可使用双引号或三引号。
- 模块顶层常量使用全大写命名，私有变量/函数以 `_` 前缀。

## 数据与类型

- 简单数据结构**优先 RECOMMENDED** 使用 `@dataclass`；不可变对象**可 MAY** 使用 `frozen=True`，频繁实例化**可 MAY** 加 `slots=True`。
- 除非有足够充分理由，**不建议 SHOULD NOT** 使用裸露 dict。
- 泛型 API **应当 SHOULD** 使用 `TypeVar` / `ParamSpec` / `Generic`，并提供清晰的类型边界。
- 容器类型注解**应当 SHOULD** 使用内建泛型（`list[T]`, `dict[K, V]`）而不是来自 typing 的旧类型（`List[T] Dict[K, V]`）
- 严格执行 typing 标注，**必须** **MUST**带正确的参数与返回值 typing。
- 除非真的类型不限，否则**绝对不应该 SHOULD NOT** 使用 Any 或 object 作为类型标注。
- 除非 attr 名称为动态取得（例如外部变量传入），否则**绝对不应该 SHOULD NOT** 使用 `getattr`。
- 如果需要字符串作为类型枚举 ID，**优先 RECOMMENDED** 使用 Literal 而不是 str。例如用 `type: Literal['aaa', 'bbb']` 而不是 `type: str`。如果重复在多个地方出现，**推荐 RECOMMENDED** 抽取为独立的 TypingAlias。
- **禁止 MUST NOT** 在文件最开头添加 `from __future__ import annotations`。

## Docstring 与注释

- 注释与 docstring 都**应 SHOULD** 以精炼的中文编写。
- docstring **必须 MUST** 使用 rst 风格。
- 公共 API **强烈建议 RECOMMENDED** 带 docstring，描述用途、参数、返回值、异常。**不要 SHOULD NOT** 在 docstring 里赘述 typing 标记里已有的类型信息。
- 在执行代码重构或移动时，**应该 SHOULD** 保留已有注释；若逻辑调整，**必须 MUST** 同步更新相关注释内容。
- 复杂或大块逻辑**推荐 RECOMMENDED** 添加少量解释性注释，但**避免 SHOULD NOT** 逐行赘述。
- **严格禁止 MUST NOT** 删除已有注释，除非已过时。

## 运行时行为

- **避免 SHOULD NOT** 静默失败，代码编写以 Fail Fast 原则为主。
- 对于库/API 层代码，**严禁 MUST NOT** 吞掉异常而什么都不做，只是为了打印错误信息。
- 如果捕获了异常，**严禁 MUST NOT** `print(e)` 或 `str(e)` 来获取错误信息。**总是 MUST** 使用 `traceback` 打印或获取完整堆栈信息，并输出到日志或提示。
- 如果项目内使用 logging 进行日志输出，在 except 块内**总是 SHOULD** 使用 `logger.exception` 而不是 `logger.error` 输出信息，同时**避免 SHOULD NOT** 在 msg 参数里重复加上堆栈信息。
- 抛异常时**应当 SHOULD** 使用自定义异常类或标准异常；错误信息简短、可读、可定位。
  - ```plaintext
    关于异常捕获策略与包裹策略

    这个异常我预期到了吗？
    ├── 是 → 它属于我的抽象层吗？
    │        ├── 是 → 翻译，附加上下文和说明性信息（raise MyException('oops') from e）
    │        └── 否 → 直接透传（FileNotFound 本身已够清晰）
    └── 否 → 不要捕获，让它穿透到顶层
    ```
- **不推荐 SHOULD NOT** 为 import 语句包裹 try-except 块，除非这个依赖本身就是可选依赖。
- 日志**统一 SHOULD** 使用 `logging.getLogger(__name__)`，避免在模块中创建自定义 logger 名称。
