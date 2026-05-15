# Rust 学习仓库 — 全局约定

详细的项目架构与目录结构说明见 [`.rust.md`](.rust.md)。

## 工具链与构建

- 使用 Rust 2024 edition，最新稳定版本。在 `rust-toolchain.toml` 中固定版本。
- 每个项目的默认构建流程：`cargo build` → `cargo test` → `cargo +nightly fmt` → `cargo clippy -- -D warnings`。
- 使用 `cargo clippy -- -D warnings -W clippy::pedantic` 进行更严格的检查，允许特定 lint 时需注明原因。
- 定期运行 `cargo audit` 检查依赖的安全漏洞。
- 使用 `cargo-deny` 执行许可证策略和禁用特定 crate。
- 在 `Cargo.toml` 中启用所有 rustc lint：`#![warn(rust_2024_compatibility, missing_docs, missing_debug_implementations)]`。

## 错误处理

- 生产代码中禁止使用 `unwrap()` 或 `expect()`，必须使用 `?` 运算符或显式 match 处理错误。
- 库代码使用 `thiserror` 定义自定义错误类型（错误枚举）。应用代码使用 `anyhow` 处理错误。
- 传播错误时使用 `.context()` 或 `.with_context()` 添加上下文信息。
- 可能失败的函数返回 `Result<T>`，不要用 `Option` 表示错误。
- 应用中不可恢复的错误使用 `panic!`，库中必须返回 `Result`。
- 使用 `thiserror` 定义领域特定的错误类型枚举，通过 `#[source]` 包含源错误。

## 异步与并发

- 使用 Tokio 作为异步运行时，必须显式指定 features（如 `tokio = { version = "1", features = ["rt-multi-thread", "macros"] }`）。
- 优先使用消息传递（通道）而非共享状态。MPSC 用 `tokio::sync::mpsc`，更快的通道用 `flume`。
- 使用 Actor 模型将系统组织为子系统。每个 Actor 拥有自己的状态，通过通道通信。对于非 Send/Sync 类型（如 Tantivy Index），隔离在专用线程中，通过通道通信，禁止用 Mutex/RwLock 包装。Actor 需要有完善的启动/停止/重启逻辑，关闭信号考虑使用 AtomicBool。
- 并发 HashMap 使用 `DashMap` 代替 `Mutex<HashMap>` 或 `RwLock<HashMap>`，性能更好。
- 对于不频繁更新的共享数据（如配置），使用 `ArcSwap`，支持无锁读取。
- 始终考虑使用 config crate 进行配置管理，配置文件使用 YAML 格式。运行时可调的数据放在配置文件中，编译时确定的数据使用编译时常量。
- 异步 trait 使用原生 `async fn`（Rust 1.75 起稳定）。**例外**：当 trait 需要对象安全（用于 `dyn Trait` 动态分发如 `Arc<dyn TaskStorage>`）时，使用 `async-trait` crate 并在模块文档中说明原因。
- 始终处理任务 panic。使用 `tokio::spawn` 时做好错误处理，考虑使用 `tokio::task::JoinSet` 管理多个任务。
- 避免在异步上下文中执行阻塞操作，CPU 密集型或阻塞操作使用 `tokio::task::spawn_blocking`。
- 使用结构化并发模式，确保生成的任务被等待或显式分离（需说明理由）。

## 类型设计与 API

- 超过 5 个字段的结构体使用 `typed-builder` crate 实现构建者模式，提供编译时保证。简单构造函数用 `new()`。
- 类型尽可能精确，零值无效时优先使用 `NonZeroU32` 而非 `u32`。
- 所有类型必须实现 `Debug`，使用 `#[derive(Debug)]` 或手动实现（敏感数据需脱敏）。
- 库类型使用 `#[non_exhaustive]` 标记为非穷举，允许未来添加字段。
- 使用枚举表示状态机，适用时优先使用类型状态模式实现编译时状态强制。
- 利用 Rust 类型系统使非法状态不可表示，将不变量编码在类型中而非运行时检查。
- 当 `T` 有默认值（如 Vec/HashMap/HashSet）时不要用 `Option<T>`，`Option<T>` 仅用于真正可选的场景。

## 安全性

- 禁止使用 `unsafe` 代码块，即使在测试中也不允许。如确有必要，需详细记录安全不变量。
- 始终验证和清理外部输入（用户输入、网络数据、文件内容），必要时使用 `validator` crate。
- TLS 使用 `rustls` + `aws-lc-rs` 加密后端，禁止使用 `native-tls` 或 OpenSSL 绑定。
- 加密值使用常量时间比较，使用 `subtle` crate 的 `ConstantTimeEq`。
- 禁止日志记录、打印或暴露敏感数据（密码、令牌、密钥），敏感类型的 `Debug` 实现需谨慎处理。
- 内存中的密钥使用 `secrecy` crate 处理（防止意外日志记录/暴露）。
- 测试环境变量使用 `dotenvy` crate，禁止硬编码凭据。

## 序列化与数据

- 使用 `serde` 进行序列化，JSON 兼容性使用 `#[serde(rename_all = "camelCase")]`。
- 单个字段映射使用 `#[serde(rename = "...")]`，向后兼容使用 `#[serde(alias = "...")]`。
- 有默认值的可选字段使用 `#[serde(default)]`，自定义默认值使用 `#[serde(default = "path::to::fn")]`。
- JSON 输出中省略 null 字段使用 `#[serde(skip_serializing_if = "Option::is_none")]`。
- 反序列化后立即验证数据，需要时使用自定义反序列化函数进行验证。
- 仅在 schema 确实动态时使用 `serde_json::Value`，优先使用强类型结构体。

## 测试

- 单元测试写在同一文件中使用 `#[cfg(test)] mod tests`，集成测试写在 `tests/` 目录。
- 测试名称使用描述性的 `test_should_` 前缀描述行为（如 `test_should_return_error_on_invalid_input`）。
- 参数化测试使用 `rstest`，属性测试使用 `proptest` 验证不变量。
- 显式测试错误情况，使用 `assert!(matches!(...))` 确保错误类型和消息正确。
- 外部依赖使用 `mockall` 或 `wiremock` 模拟，避免过度模拟，快速时优先使用真实实现。
- 追求高测试覆盖率，但聚焦于关键路径和边缘情况而非覆盖率数字。
- 慢速测试使用 `#[ignore]`，CI 中使用 `cargo test -- --ignored` 运行。
- 在文档注释中编写文档测试，同时作为示例和自动测试。

## 日志与可观测性

- 使用 `tracing` 进行结构化日志和诊断，生产代码中禁止使用 `println!` 或 `dbg!`。
- 使用适当的日志级别：`error!` 用于错误，`warn!` 用于警告，`info!` 用于重要事件，`debug!` 和 `trace!` 用于诊断。
- 异步函数使用 `tracing::instrument` 添加上下文，包含相关字段：`#[instrument(skip(large_param))]`。
- 使用 `tracing-subscriber` 配置输出，生产环境用 JSON 格式，开发环境用人类可读格式。
- 为请求/操作生命周期实现 span，使用 `span.in_scope()` 或 `instrument` 宏。

## 性能

- 优化前先做性能分析，使用 `cargo flamegraph`、`perf` 或 `samply`。
- 避免不必要的分配，使用 `&str` 而非 `String`，优先借用而非克隆。
- 已知最终大小时使用 `Vec::with_capacity()`，预分配集合避免重新分配。
- 优先使用迭代器而非显式循环，迭代器通常优化更好且组合性强。
- 通常适合栈的小向量使用 `SmallVec`，小堆分配使用 `smallbox`。
- 数据可能借用或拥有时使用 `Cow<str>`，借用时避免克隆。
- 热路径考虑使用 `#[inline]` 或 `#[inline(always)]`（需说明理由）。
- 性能基准测试使用最新 `criterion` crate，早期开发阶段不做基准测试。

## 依赖管理

- 最小化依赖，每个依赖都增加编译时间、二进制大小和攻击面。
- 谨慎固定版本，补丁更新用 `~`（如 `tokio = "~1.40"`），次要更新用 `^`（默认）。
- 优先使用纯 Rust crate 而非 FFI 绑定，更安全、更易移植、更易审计。
- 添加新依赖前审查其维护状态、安全历史和代码质量。
- 跨 crate 共享依赖使用 workspace 依赖：`[workspace.dependencies]`。

## 文档

- 所有公开项必须编写文档注释（`///`），文档注释中包含示例。
- 模块级文档使用 `//!`，说明模块用途和使用模式。
- 公开函数的文档注释中至少写一个示例，示例会自动测试。
- 文档注释中使用 `# Errors`、`# Panics`、`# Safety` 章节记录失败模式。
- 使用 `cargo doc --open` 生成文档，确保文档渲染正确、格式良好。

## 代码风格

- 文件顶部的 `use` 导入按以下顺序排列：std、外部依赖、本地模块。
- 遵循 Rust 命名规范：函数/变量用 `snake_case`，类型用 `PascalCase`，常量`SCREAMING_SNAKE_CASE`。
- 保持函数小而专注，遵循 KISS 原则，将复杂逻辑提取为命名良好的函数。除非绝对必要，函数不超过 150 行。
- 公开 API 中优先使用显式类型而非 `impl Trait` 以保持清晰，内部函数可用 `impl Trait`。
- 开发过程中禁止使用 `todo!()`，必须有明确的计划和完成路径。
- 项目顺序保持一致：导入、常量、类型、函数、测试。使用 `rustfmt` 自动格式化。
- 多行函数调用和结构体字面量使用尾随逗号，使 diff 更清晰。

## 仓库规范

- 每个项目以 `pNNN-名称` 格式命名（如 `p000-template`、`p001-rustcli`），`p` 前缀 + 编号表示学习顺序。
- 每个项目必须包含：`Cargo.toml`、`src/main.rs`（或 `src/lib.rs`）、`README.md`、`CLAUDE.md`。
- 新建项目时，复制 `p000-template` 作为起点，修改 `Cargo.toml` 中的 `name` 字段。
- 提交信息使用中文，格式：`<类型>: <简述>`，类型包括：`feat`、`fix`、`docs`、`refactor`、`test`、`chore`。
