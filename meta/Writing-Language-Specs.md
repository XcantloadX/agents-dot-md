# 如何编写语言规范文件

本文档说明如何在本仓库新增或修改 `languages/*.md` 语言规范文件。范例以 `languages/Python.md` 为准，遇冲突时以范例为准。

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## 文件分工

- `languages/<Lang>.md` — 单语言代码规范，是唯一可被 Agent 直接执行的规范正文。
- `basis/*.md` — 跨语言通用基础（如无则不写，不强行抽取共性）。
- `meta/*.md`（本目录）— 规范的规范，只讲“怎么写规范文件”，**严禁 MUST NOT**写具体代码规则。
- 根 `AGENTS.md` — 路由入口，只做索引和覆盖优先级声明，**严禁 MUST NOT**复制各语言规则正文。

动笔前**必须 MUST**完整读一遍 `languages/Python.md` 和根 `AGENTS.md`。

## 命名与开头

- 文件名用英文语言名，如 `languages/Rust.md`，大小写与官方名称一致。
- 首行标题为 `# <语言> 代码规范`。
- 紧随 RFC 2119 关键词声明段（照抄 `Python.md` 第 3 行原文）。

## 覆盖规则节

每个语言文件**必须 MUST**设 `## 覆盖规则` 为第一节，写清三层优先级：

用户提示词要求 > 本规范要求 > 该语言官方默认规范（如 PEP 8）要求

官方基线只写一个（如 PEP 8、ECMA-262），**严禁 MUST NOT**列多个风格指南。

## 分节

按以下顺序组织，无对应内容时省略整节，**严禁 MUST NOT**生造空节或调换顺序：

1. `## 覆盖规则`
2. `## 代码结构与风格`
3. `## 数据与类型`
4. `## Docstring 与注释`
5. `## 运行时行为`

节名照抄，不得改写（如不叫“错误处理”，统一叫“运行时行为”）。框架专属规范（如 `Python-Qt.md`）**应当 SHOULD**只写增量规则，并在开头声明其基线语言文件。

## 单条规则写法

- 每条规则单行单义，一个 `- ` 一条规则；条件用“除非…”前置，不写从句嵌套。
- 强度标记**必须 MUST**为“中文情态词 + 空格 + 英文关键词”并整体加粗，如 `**优先 RECOMMENDED**`、`**可 MAY**`；**严禁 MUST NOT**只加粗英文关键词，如 `**RECOMMENDED**`、`**MAY**`。（`Python.md` 第 22 行 `**必须** **MUST**` 分写为历史写法，新文件一律整体加粗。）
- 中文情态词与关键词**必须 MUST**对应，**严禁 MUST NOT**混用：MUST 配必须 / 总是 / 严格，MUST NOT 配禁止 / 严禁 / 严格禁止，SHOULD 配应当 / 应该 / 应 / 统一，SHOULD NOT 配避免 / 不要 / 不建议 / 不推荐 / 绝对不应该，RECOMMENDED 配优先 / 推荐 / 强烈建议，MAY 配可。
- 中文精炼，点到为止；涉及标识符、API、类型时用行内反引号，如 `` `list[T]` ``、` `logger.exception` `。
- 反例**应当 SHOULD**写在同一行内（`…而不是…`），不另起段落解释。

强度选择：

- `MUST / MUST NOT` — 违反即错（如 typing 缺失、吞异常、删注释）。
- `SHOULD / SHOULD NOT` — 默认照做，偏离需在提交说明中给出理由。
- `RECOMMENDED` — 推荐做法，不强制（如 `dataclass`、抽取 `TypeAlias`）。
- `MAY` — 允许的选项（如 `frozen=True`、`slots=True`）。

## 代码示例

- 能用行内反引号说清的，不贴块。
- 必须贴块时用 `plaintext` 块（如 `Python.md` 的异常决策树），不贴大段可运行代码。
- 示例只为解释规则存在，不引入新规则；示例中的规则**必须 MUST**在正文中有对应条目。

## 禁止事项

- **严禁 MUST NOT**把 `Python.md` 的规则套用到其他语言（如单引号、`dataclass`、`Literal` 都是 Python 专属）。
- **严禁 MUST NOT**在语言文件里写路由、工作流、贡献流程，那是根 `AGENTS.md` 和 `meta/` 的事。
- **严禁 MUST NOT**写“努力”“尽量”“一般”等无强度的模糊词；每条规则**必须 MUST**有 RFC 2119 关键词。
- 注释规则默认三条（参考 `Python.md`）：**严格禁止 MUST NOT**删注释除非过时；重构**应该 SHOULD**保留注释；逻辑变更**必须 MUST**同步更新注释。
- 行为准则默认 Fail Fast：**避免 SHOULD NOT**静默失败；库/API 层**严禁 MUST NOT**吞异常，异常与日志策略按语言生态写，不自行发挥。

## 模板

```plaintext
# <语言> 代码规范

The key words "MUST", ... as described in RFC 2119.

## 覆盖规则

... > 本规范要求 > ... 要求

## 代码结构与风格

- ...

## 数据与类型

- ...

## Docstring 与注释

- ...

## 运行时行为

- ...
```

## 提交前自检

1. 标题、RFC 2119 段、覆盖规则节齐全？
2. 每条规则的强度标记都是“中文情态词 + 英文关键词”整体加粗（如 `**优先 RECOMMENDED**`），无裸英文关键词，且单行单义？
3. 有空节或模糊词吗？有跨语言复制的规则吗？
4. 根 `AGENTS.md` 的目录结构与速览需要同步更新吗？（只加索引行，不复制正文。）
