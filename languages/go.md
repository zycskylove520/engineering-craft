# Go 工程规范

> 适用版本：Go 1.22+ ｜ 生态：云原生、后端服务、CLI、基础设施 ｜ 识别信号：`.go`、`go.mod`
> 通用规范见 `core/`；本文件只写 Go 特有的落地方式。

## 1. 语言定位与适用场景
擅长高并发后端、云原生基础设施、CLI 工具；编译快、部署单二进制、并发模型简单。不适合复杂领域建模的表达力（泛型有限）与 GUI。

## 2. 项目结构与模块划分
- 一个 module 一个 `go.mod`；按领域分 package，package 名短小、与目录一致、不用 `util`/`common` 之类的垃圾桶包。
- 用内部包 `internal/` 限制可见性；公开面通过导出标识符（首字母大写）控制。
- 避免包级循环依赖；接口定义在**使用方**包（consumer-defined interface）。

## 3. 命名规范
- 包名：小写单词、无下划线、无驼峰（`httputil` 不是 `httpUtil`）。
- 类型/函数/常量：官方术语为 MixedCaps/mixedCaps（导出/非导出，即俗称 PascalCase/camelCase，见 Effective Go §MixedCaps）；单方法接口用 `-er` 后缀（`Reader`、`Closer`，官方 §Interface names）。**"接口名不带 I 前缀"及 PascalCase/camelCase 措辞为通用/社区约定，非 Go 官方原文。**
- 文件名：小写 + 下划线；测试文件 `_test.go`。
- 缩写保持一致大写（`HTTPServer`、`userID`、`URL`）。

## 4. 类型系统与数据建模
- 用 struct + 组合表达数据结构；小接口、显式实现。
- 用自定义类型区分语义（`type UserID int64`）；用 `struct{}` 表达集合。
- 让零值可用（zero value usable）；用指针或 `*T` / `omitempty` 区分"未设置"；避免用 `nil` 表示合法业务状态无说明。
- 泛型（1.18+）用于真正的通用算法，不滥用。

## 5. 错误处理与异常策略
- **error 是值**：显式返回，逐层处理或包装；`fmt.Errorf("context: %w", err)` 保留错误链。
- 用 `errors.Is`/`errors.As` 判断，不比较字符串。
- 禁止静默吞错：`_` 丢弃 error 要有理由；不要 `if err != nil { return nil }` 丢掉上下文。
- panic 只用于真正不可恢复的程序错误；库代码不 panic，返回 error；recover 只在边界（如 server middleware）。
- 用 sentinel error（`var ErrX = errors.New(...)`）表达可判定的业务错误。

## 6. 注释与文档
- **结构声明必须有 godoc**：每个导出包有包注释（`// Package x ...`），每个导出类型/函数/常量/变量有以标识符开头的 `//` 注释，写清用途、参数、返回、错误条件（godoc 约定）。
- `//go:generate`、`//go:build` 等指令按规范写。
- **函数体逻辑**：只写"为什么"，不写"做什么"；代码自解释优先。
- `go doc` 用于在命令行**查看**文档；`pkgsite` 是 pkg.go.dev 的源码、用于**生成**站点文档；旧 `godoc` 工具已非主流（见 go.dev/blog/godoc）。三者定位不同，按需要选用。

## 7. 并发与异步模型
- goroutine + channel：用 channel 传递数据、用 `sync` 原语保护共享状态；"不要通过共享内存通信，而要通过通信共享内存"。
- 每个 goroutine 有明确退出路径（context 取消），避免泄漏；用 `sync.WaitGroup` 等待。
- `context.Context` 贯穿调用链，传递取消与超时（deadline）；IO/网络调用必须可取消。
- 竞态用 `go test -race` 检测；锁范围最小化，注意锁顺序防死锁。

## 8. 测试
- 标准 `testing` 包 + table-driven tests；测试名 `TestXxx`，子测试 `t.Run`。
- 用 `t.TempDir()` 做隔离；外部依赖用接口 + fake 实现，不 mock 被测对象（此为通用测试原则，Go 官方 CodeReviewComments §Interfaces 主张用真实实现的公开 API 测试、在使用方用 fake，但未逐字规定"不 mock 被测对象"）。
- `go test -race ./...` 检查竞态；`go test -cover` 看覆盖率；集成测试用 build tag 分离。
- 测试文件与被测包同包（测内部）或 `_test` 包（测公开 API），按需要选。

## 9. 依赖管理与构建
- `go.mod` + `go.sum` 必须提交；版本用语义化版本，定期 `go mod tidy`。
- 构建用 `go build ./...`；多目标用 Makefile / Taskfile 统一入口。
- 尽量少依赖；引入新依赖评估维护状态与许可证。

## 10. 工具链
`gofmt`（强制，无争议）、`go vet`、`staticcheck`/`golangci-lint`、`go test -race`、`pprof`（性能）。
CI 跑 `gofmt -l`（应为空）+ vet + lint + test -race。

## 11. 性能惯用法与常见陷阱
- 度量先行：`pprof`（CPU/heap/goroutine）、`benchstat` 对比 benchmark。
- 坑：goroutine 泄漏、切片共享底层数组导致意外修改、`append` 触发扩容的别名、range 变量闭包捕获（Go 1.22 已修）、接口 nil 与指针 nil 的陷阱、defer 在循环中的堆积。
- 避免过度分配，注意 string/[]byte 转换拷贝。

## 12. 代码评审检查清单
- [ ] 导出标识符都有 godoc（以标识符开头）？
- [ ] error 显式处理、有上下文、错误链保留、无静默吞错？
- [ ] 无 panic 滥用、库代码不 panic？
- [ ] context 贯穿、goroutine 有退出路径、无泄漏？
- [ ] 接口在使用方定义、包职责单一？
- [ ] `gofmt`/vet/lint/`-race` 全过？
