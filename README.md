# Rust 学习之旅

一个用于系统性学习 Rust 的项目集合，每个子项目对应一个技术章节。

## 项目列表

| 编号 | 项目 | 说明 |
|------|------|------|
| p000 | [template](p000-template/) | 项目模板 — Hello World |
| p001 | [rustcli](p001-rustcli/) | CLI 工具开发 |

> 更多项目将随学习进度逐步添加，编号表示学习顺序。

## 快速开始

### 环境安装

```bash
# 安装 Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 安装常用工具
cargo install cargo-generate        # 项目模板生成（可选）
cargo install --locked cargo-deny   # 依赖审计
cargo install typos-cli             # 拼写检查
cargo install git-cliff             # 变更日志
cargo install cargo-nextest --locked # 增强测试
pipx install pre-commit             # 提交前检查（可选）
```

### 运行项目

```bash
cd p000-template
cargo run
```

### 新建项目

```bash
cp -r p000-template p001-新项目名
# 修改 Cargo.toml 中的 name 字段
# 更新项目内的 README.md 和 CLAUDE.md
```

## 推荐 VS Code 插件

| 插件 | 说明 |
|------|------|
| rust-analyzer | Rust 语言支持 |
| Even Better TOML | TOML 文件支持 |
| Error Lens | 行内错误显示 |
| crates | Rust 包管理 |
| Rust Test Lens | 测试支持 |
| Rust Test Explorer | 测试概览 |

## 目录结构

```
Rust/
├── CLAUDE.md              # 全局约定
├── .rust.md               # 项目架构说明
├── README.md              # 本文件
├── .gitignore
├── cliff.toml             # changelog 配置（共享）
├── deny.toml              # 依赖审计配置（共享）
├── _typos.toml            # 拼写检查配置（共享）
├── .pre-commit-config.yaml# 提交前检查配置（共享）
├── .github/workflows/     # CI 配置
├── p000-template/         # 项目模板
│   ├── CLAUDE.md
│   ├── README.md
│   ├── Cargo.toml
│   └── src/
│       └── main.rs
├── p001-rustcli/          # CLI 工具开发
└── ...
```

## 更多信息

- 代码规范与全局约定：[`CLAUDE.md`](CLAUDE.md)
- 项目架构详情：[`.rust.md`](.rust.md)
