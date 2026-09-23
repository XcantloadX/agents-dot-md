# Go 代码规范

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## 覆盖规则

总体代码规范依照 Effective Go 与 Uber Go Style Guide。**严格 MUST** 按照如下顺序覆盖：

用户提示词要求 > 本规范要求 > `gofmt` / `go vet` 要求

## 包组织

- 默认保持扁平，小服务**优先 RECOMMENDED** 单包或数个领域包（如 `auth/` / `billing/` / `jobs/`），**禁止 MUST NOT** 先搭 `controller/` / `service/` / `repository/` / `domain/` 分层脚手架。
- 包名**必须 MUST** 小写、单个词、与目录同名；**禁止 MUST NOT** 新建 `utils/` / `helpers/` / `common/` 大杂烩包。
- `internal/` 仅**可 MAY** 用于 library 需要对外隐藏的子系统，普通可执行应用**不应该 SHOULD NOT** 默认套用。
- 包之间**禁止 MUST NOT** 出现横向循环依赖，装配点收敛到 `main`；HTTP handler 与其模板资源**应当 SHOULD** 同属 `web/` 包。

## 命名与语法

- 导出名用 `MixedCaps`，包外访问才大写；getter **禁止 MUST NOT** 加 `Get` 前缀；错误变量以 `Err` 开头。
- 新语法**应当 SHOULD** 用现行写法：`any` 不用 `interface{}`，`//go:build` 不用 `// +build`，整数区间用 `for i := range N`，`min` / `max` 用内置函数。
- 工具依赖**必须 MUST** 用 `go.mod` 的 `tool` 指令管理，不写 `tools.go` 空白 import。
- `go.mod` **应当 SHOULD** 声明 `toolchain` 保证可复现构建。

## 类型与接口

- 零值**应当 SHOULD** 开箱可用（如 `sync.Mutex` / `bytes.Buffer` 风格），避免必须调用构造才可用的类型。
- 接口小而专，由使用方定义，先写具体类型再抽象；**应当 SHOULD** 做到入参接最小接口、返回值给具体结构体。
- 泛型仅**可 MAY** 用于重复算法（如 `Min[T cmp.Ordered]`），**禁止 MUST NOT** 写通用 `Repository[T]` / 通用 Service 基类，不用 `any` 掩盖设计不清。
- 优先用标准库：`slices` / `maps` / `cmp` / `errors.Join` / `iter` / `math/rand/v2`，**不应该 SHOULD NOT** 手写已有的排序、拷贝、判等与多错误合并。

## 错误处理

- 错误是值，**必须 MUST** 显式检查，**禁止 MUST NOT** 用 `_` 丢弃；类型断言**必须 MUST** 用 `comma-ok` 形式。
- 向上传递时**必须 MUST** 用 `fmt.Errorf("... %w", err)` 附加当时动作，不裸抛 `err`；跨系统边界时保证信息可知是错误。
- 可被调用方判定处理的错误**应当 SHOULD** 定义为哨兵 `var ErrX = errors.New(...)`，判定用 `errors.Is` / `errors.As`。
- 错误只处理一次，**禁止 MUST NOT** 同一处既打 ERROR 日志又上抛让上层再打；library 代码**禁止 MUST NOT** `panic`，只在程序确实无法继续时 panic。

## 并发与 Context

- `context.Context` **必须 MUST** 作为第一个参数透传，**禁止 MUST NOT** 存进结构体；测试用 `t.Context()` 而不是 `context.Background()`。
- 每个 `go func()` **必须 MUST** 有明确退出条件（context 取消或 channel 关闭），**禁止 MUST NOT** 启动不知何时停止的 goroutine。
- 有界并发**优先 RECOMMENDED** 用 `errgroup.WithContext` + `SetLimit`，无须错误传播时**可 MAY** 用 Go 1.25 的 `WaitGroup.Go`；**不应该 SHOULD NOT** 手搓信号量 channel 或常驻 worker 池。
- 共享数据优先用 channel 编排，mutex 只做串行化；原子值用类型化 `atomic.Int64` / `atomic.Bool` / `atomic.Pointer[T]`。

## HTTP 服务

- Go 1.22+ 的标准 `net/http ServeMux` 已支持方法 + 路径参数（`GET /users/{id}` + `r.PathValue`），**不应该 SHOULD NOT** 默认引入 mux 框架；中间件就是 `func(http.Handler) http.Handler` 的函数组合。
- 生产 server **必须 MUST** 设置 `ReadHeaderTimeout` / `ReadTimeout` / `WriteTimeout` / `IdleTimeout`，出站 `http.Client` **必须 MUST** 自带超时，不用无超时的 `ListenAndServe` / `DefaultClient` 裸跑。
- 优雅停机**必须 MUST** 用 `signal.NotifyContext` + `srv.Shutdown(带超时的新context)` 排空在途请求，长连接**必须 MUST** 监听 `r.Context()` 以免卡住关闭。
- 配置复杂时用 Functional Options（`NewServer(addr, WithTimeout(...))`）并给出合理默认值。

## 日志与配置

- 结构化日志**应当 SHOULD** 用 `log/slog`，字段用 `slog.String` / `slog.Group` 组织；级别按语义：生命周期 INFO、可恢复 WARN、需处理 ERROR，高频内部状态 DEBUG。
- Logger 作为依赖传入，**禁止 MUST NOT** 在 library 里用包级全局 logger；`slog.Default()` 仅允许在 `main` 兜底。

## Docstring 与注释

- 注释与文档**应 SHOULD** 以精炼的中文编写。
- 导出符号**强烈建议 RECOMMENDED** 写文档注释（用途、前置条件、副作用），**不要 SHOULD NOT** 复述签名里已有的类型信息。
- 注释只写 why（非显而易见约束、权衡、上游行为），**禁止 MUST NOT** 逐行复述、提交注释掉的代码；错误信息面向行动，不写模糊的“出错了”。

## 测试

- 单元测试**推荐 RECOMMENDED** 表驱动 + `t.Run` 子测试，断言复杂结构用 `go-cmp` 而不是 `reflect.DeepEqual`，复用断言抽 helper 时**必须 MUST** 调 `t.Helper()`。
- 外部依赖用手写 fake / stub，**不应该 SHOULD NOT** 默认引入重型 BDD / mock 生成框架；文件系统用 `afero` 抽象，大输出用 `testdata` + golden 文件。
- 并发测试**禁止 MUST NOT** 用 `time.Sleep` 等待 goroutine，用 `testing/synctest`、channel 或显式同步；压测用 `b.Loop()` 新写法。
