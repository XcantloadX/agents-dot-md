# AGENTS.md — agents-dot-md 规范库入口

本仓库是 AGENTS.md 代码规范集合（`README.md`：一些 AGENTS.md 代码规范），不是可运行代码库。协议见 `LICENSE`（CC BY-NC-SA 4.0）。

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119. 本仓库所有规范文件中的上述关键词均按此解释，行文格式为“中文情态词 + 英文关键词”整体加粗（如 `**优先 RECOMMENDED**`、`**可 MAY**`），**严禁 MUST NOT**只加粗英文关键词。

## 目录结构

- `languages/Python.md` — Python 代码规范（当前唯一有效规范）。
- `languages/PowerShell.md`、`languages/Python-Qt.md`、`languages/Web.md` — 占位空文件，**严禁 MUST NOT**视为有效规范。
- `basis/Basis.md` — 占位空文件，**严禁 MUST NOT**视为有效规范。
- `basis/Git.md` — Git 提交与操作规范，git 相关任务**必须 MUST**完整读取并遵守。
- `meta/Writing-Language-Specs.md` — 如何编写语言规范文件（规范的规范）。
- `README.md`、`LICENSE` — 项目说明与协议。

## 覆盖优先级

处理任何代码任务时，**必须 MUST**按以下顺序覆盖（源自 `languages/Python.md` 并推广到全库）：

用户提示词要求 > 本仓库对应规范文件要求 > 该语言官方默认规范（如 PEP 8）要求

## Agent 工作流

1. 按需加载规范，不要全量预读。写 Python 代码前**必须 MUST**先完整读取 `languages/Python.md`；不写 Python 则**不建议 SHOULD NOT**读取它。
2. 空文件代表规范缺失：若任务命中的规范文件为空，**必须 MUST**直说规范缺失并只遵循用户提示词 + 语言官方默认规范，**严禁 MUST NOT**把 `Python.md` 的规则套用到其他语言。
3. 下游项目引用本仓库时，复制本文件做法：**应该 SHOULD**保留“路由 + 按需读取”模式，而不是把各语言全文粘进单个 AGENTS.md，避免漂移。
4. 涉及提交信息、分支标签或任何 git 操作时，**必须 MUST**先完整读取 `basis/Git.md` 并遵守；无 git 操作的纯代码任务**不建议 SHOULD NOT**预读它。

## 维护本仓库规范文件的约定

新增或修改 `languages/*.md` / `basis/*.md` 时，**必须 MUST**遵循 `meta/Writing-Language-Specs.md`（以 `languages/Python.md` 为范例）。本节仅索引，不重复正文。
