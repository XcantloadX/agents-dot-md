# 更新 prompt 文档指导

本文件是给 Agent 看的更新流程操作指导：更新下游项目内复制自本仓库的 prompt 文档与 `AGENTS.md`。只讲流程，**严禁 MUST NOT**涉及具体语言的代码规则。下游项目内的相等文档与本文件有冲突时，以本文件为准。

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## 1. 确定更新对象

- 更新前**必须 MUST**先确定目标项目内的 prompt 文件夹（用户未特别说明时为 `/docs/prompts`）与根 `AGENTS.md`；找不到时可按创建时的位置反转推测，找不到才询问用户。
- prompt 文件夹内的文档以 `languages/`、`basis/` 原始目录结构存放，更新流程与创建流程（见 `Create-Project.md`）共用同一套结构约定。

## 2. 更新前对比 diff

- 对每份待更新文档，**必须 MUST**先对比源仓库最新内容与项目内现存副本（`diff` 或等价手段），而不是直接覆盖。
- 项目内副本与源仓库最新版本一致时，**可 MAY**跳过该文件。
- 对比出的差异一律视为潜在冲突：**禁止 MUST NOT**未经用户确认直接覆盖，**必须 MUST**向用户询问如何处理（以源仓库为准覆盖 / 以项目内副本为准 / 人工合并），并按用户决定执行。
- 目标文件不存在（新增文档）时直接带目录结构复制，无需询问；涉及删除的任何操作，见第 3 节。

## 3. 执行更新

- 更新时**必须 MUST**保留原始目录结构与文件名（如 `languages/Python.md` 停留在 `/docs/prompts/languages/Python.md`），而不是平铺合并或改名。
- 0 字节空占位文件**禁止 MUST NOT**写入或复制进项目。
- 更新完成后**应当 SHOULD**复查各文件与源仓库一致，而不是只凭复制命令成功判定。
- 源仓库已删除或改名的文档，**禁止 MUST NOT**默认删除项目内副本，**必须 MUST**提醒用户并确认后再删。

## 4. 更新 AGENTS.md

- 更新后**必须 MUST**检查根 `AGENTS.md` 的 `@` 引用路径是否与实际 prompt 文件夹一致；不一致时**必须 MUST**同步修正（`@<目标相对路径>/*`）。
- `AGENTS.md` 统一维持引用形式，**禁止 MUST NOT**把规范正文粘贴进 `AGENTS.md`。
