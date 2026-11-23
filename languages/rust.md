# Rust 工程规范

> 适用版本：Rust 1.56+（edition 2021）/ 1.85+（edition 2024）｜ 生态：安全系统、基础设施、WASM、CLI ｜ 识别信号：`.rs`、`Cargo.toml`
> 注：edition 2024 由 Rust 1.85.0 才稳定（doc.rust-lang.org/edition-guide/rust-2024），1.75 只支持到 edition 2021。
> 通用规范见 `core/`；本文件只写 Rust 特有的落地方式。

## 1. 语言定位与适用场景
内存安全无 GC、零成本抽象；适合系统、基础设施、性能敏感且安全的场景、WASM。学习曲线陡，业务快速迭代场景权衡使用。

## 2. 项目结构与模块划分
- crate 是基本单元；`src/lib.rs` 定义公开面，`src/main.rs` 是二进制入口；用 workspace 管理多 crate。
- 模块用 `mod`/`pub mod`；可见性默认私有，公开面最小化（`pub(crate)` 优先于 `pub`）。
- 按领域分模块，`mod.rs` 或同名文件组织；避免深层嵌套模块。

## 3. 命名规范
- 类型/trait/enum：PascalCase；函数/变量/模块：snake_case；常量：`UPPER_SNAKE_CASE`。
- 枚举变体 PascalCase；trait 名描述能力（`Iterator`、`Read`）；类型参数单大写（`T`、`E`）。
- 构造函数惯用 `new()`；标准无歧义转换用 `From`/`TryFrom` trait（官方 C-CONV 亦允许临时转换遵循 `as_`/`to_`/`into_` 前缀约定，按成本/所有权选择）——避免无规律地自定义 `to_x`。
- 缩写保持一致（`HttpServer` 非 `HTTPServer`，但这点按 rustfmt/社区约定）。

## 4. 类型系统与数据建模
- 用 `Option<T>` 表达可选、`Result<T, E>` 表达可失败，**不用 null**。
- 用 enum + 匹配让非法状态不可表示（tagged union 是核心武器）；用新类型（newtype）区分语义。
- 所有权/借用是核心：`&T` 不可变借、`&mut T` 可变借；明确生命周期，避免不必要克隆。
- 用 `PhantomData`/marker 类型表达类型状态（type-state pattern）；内部可变性用 `Cell`/`RefCell`（单线程）或 `Mutex`/`RwLock`。

## 5. 错误处理与异常策略
- 可恢复错误用 `Result<T, E>`，用 `?` 向上传播并添加上下文（`anyhow::Context` 或 `thiserror` 定义领域错误）。
- 不可恢复用 `panic!`（或 `expect` 带说明）；库代码避免 panic，边界（`main`/请求入口）统一处理。
- 用 `thiserror` 定义领域错误枚举，`anyhow` 用于应用层；禁止 `unwrap()`/`expect()` 出现在生产路径无说明。
- 错误链保留（source），边界转成对外一致的错误结构。

## 6. 注释与文档
- **结构声明必须有 rustdoc**：公开项用 `///`，重点写用途、`# Errors`、`# Panics`、`# Safety`（unsafe 函数必须）、示例（doctest）。**参数与返回类型已由签名表达，官方 rustdoc book 不要求重复写出**；是否补写属项目风格。
- 包/模块级用 `//!` 写概述；用 `cargo doc` 生成文档，doctest 必须能跑通。
- **函数体逻辑**：只写"为什么"——意图、`unsafe` 的原因与不变量；`//` 注释克制，代码能看懂的不写。
- 注释里可写 doctest 示例，既是文档也是测试。

## 7. 并发与异步模型
- 线程用 `std::thread`；消息传递用 `std::sync::mpsc`/`crossbeam-channel`；共享状态用 `Arc<Mutex<T>>`/`RwLock`。
- async 用 `tokio`/`async-std`：`.await`、task spawn；注意 Send/Sync 约束、不要在 async 里持锁跨 await。
- 取消用 `tokio::select!`；`CancellationToken` 位于 **tokio-util** crate（`tokio_util::sync::CancellationToken`，非 tokio 核心）；超时用 `tokio::time::timeout`。async-std 与 tokio 均为真实运行时，Rust 官方不指定，按项目选择。
- Rust 的类型系统防数据竞争（Send/Sync）；仍要注意死锁（锁顺序）与逻辑竞态。

## 8. 测试
- 内置 `#[test]` + `#[cfg(test)]`；集成测试放 `tests/`；doctest 随文档自动跑。
- mock 用 `trait` + 测试实现，或 `mockall`；只隔离外部依赖，不 mock 被测对象。
- `cargo test` 跑全部；覆盖率用 `cargo-llvm-cov`；属性测试用 `proptest`/`quickcheck`。
- 测试名描述行为；固件用 helper 函数/结构。

## 9. 依赖管理与构建
- `Cargo.toml` + `Cargo.lock`（应用必须提交锁文件）；版本语义化，定期 `cargo update` 审慎。
- 工作区用 `[workspace]`；feature flags 控制可选功能与条件编译。
- 引入依赖评估维护状态、unsafe 用量、许可证（`cargo deny`）。

## 10. 工具链
`rustfmt`（强制）、`clippy`（lint，pedantic 按项目约定）、`cargo test`、`cargo doc`、`cargo deny`（许可证/安全）、`cargo audit`（漏洞）、`cargo-llvm-cov`（覆盖率）、`cargo flamegraph`（性能）。
CI 跑 fmt + clippy(-D warnings) + test + doc。

## 11. 性能惯用法与常见陷阱
- 度量先行：`cargo flamegraph`/`perf`/`criterion`（基准），不凭感觉优化。
- 坑：不必要克隆（先借后克隆）、`collect` 的分配、迭代器的惰性求值理解错误、async 中阻塞调用阻塞 runtime、锁跨 await。
- 先写清晰代码，再按需优化（零成本抽象让惯用写法通常已经很快）；unsafe 必须有充分理由与 SAFETY 注释。

## 12. 代码评审检查清单
- [ ] 公开项有 rustdoc（含 Errors/Panics/Safety/示例）且 doctest 通过？
- [ ] 用 Option/Result 而非 null/panic、unwrap 有说明？
- [ ] 所有权/借用清晰、无不必要克隆、生命周期合理？
- [ ] enum 让非法状态不可表示、可见性最小？
- [ ] async 无阻塞调用、无持锁跨 await、取消超时明确？
- [ ] unsafe 有 SAFETY 注释与不变量说明？
- [ ] rustfmt/clippy/test/audit 全过？
