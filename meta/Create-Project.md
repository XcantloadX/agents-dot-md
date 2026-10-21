# 创建新项目指导

本文件是给 Agent 看的创建新项目操作指导，只讲流程，**严禁 MUST NOT**涉及具体语言的代码规则。

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## 1. 确定技术栈

- 动手创建之前**必须 MUST**确定技术栈；用户未说明时**必须 MUST**显式询问用户，而不是凭语言环境自行猜测。
- 混合技术栈（如“前端 + 后端”）**应当 SHOULD**分项确认各部分，而不是笼统询问一句。

## 2. 创建项目

- **优先 RECOMMENDED**调用官方脚手架创建项目骨架（如 `uv init`、`npm create vite`、`cargo new`、`flutter create`），而不是手写配置文件。
- 脚手架不可用或生成骨架与需求明显不符时，**可 MAY**手写最小配置创建项目。
- 创建过程中涉及 git 操作前，**必须 MUST**先完整读取 `basis/Git.md` 并遵守。

## 3. 复制 prompt 文档

- **必须 MUST**从本仓库复制 prompt 文档到新项目，来源限于 `languages/` 与 `basis/` 下的规范文档；根 `AGENTS.md`、`meta/`、`README.md`、`LICENSE` **禁止 MUST NOT**复制。
- 仅复制项目内需要的文档，明显无关的文档**禁止 MUST NOT**复制（如 Python 项目不复制 `Web.md`）。
- 与项目技术栈对应的文档**必须 MUST**复制而不是遗漏（如 Python 项目复制 `Python.md`，配 `Basis.md` 与 `Git.md`）。
- 0 字节空文件是占位、表示规范缺失，**禁止 MUST NOT**复制。
- 用户未特别说明时，**必须 MUST**复制到项目根目录下 `/docs/prompts`，而不是其他位置；用户有指定时以用户指定为准。
- 复制时**必须 MUST**保留仓库内原始目录结构（如 `languages/Python.md` 复制为 `/docs/prompts/languages/Python.md`），而不是平铺合并到同一层；同时**必须 MUST**保持原文件名与原文内容，而不是改写或改名。

## 4. 创建 AGENTS.md

- 复制完成后**必须 MUST**在项目根目录创建 `AGENTS.md`，内容为指向 prompt 文件夹的 `@` 引用，如复制目标是 `/docs/prompts` 时内容为：

```plaintext
@docs/prompts/*
```

- `@` 引用路径**必须 MUST**与实际复制目标一致（`@<目标相对路径>/*`），而不是固定照抄上面的示例。
