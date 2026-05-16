# p000-template — 项目约定

本项目是所有学习项目的模板，新建项目时复制此目录作为起点。

## 项目说明

- **类型**: 项目模板
- **学习目标**: 熟悉 Rust 项目结构与基本语法
- **涉及概念**: `main` 函数、`println!` 宏、Cargo 基本配置

## 项目结构

```
p000-template/
├── CLAUDE.md              # 本文件 — 项目约定
├── README.md              # 项目说明
├── Cargo.toml             # 包名、版本、依赖
├── Cargo.lock             # 依赖锁定文件
├── CHANGELOG.md           # 变更日志
├── src/
│   └── main.rs            # 入口文件
└── assets/                # 静态资源

以下配置文件统一放在根目录，所有项目共享：
- cliff.toml              # git-cliff 配置
- deny.toml               # cargo-deny 配置
- _typos.toml             # 拼写检查配置
- .pre-commit-config.yaml # pre-commit 钩子配置
```

## 运行方式

```bash
cargo run
```

## 使用此模板新建项目

```bash
# 1. 复制目录
cp -r p000-template p001-新项目名

# 2. 修改 Cargo.toml 中的 name
# [package]
# name = "新项目名"

# 3. 清理构建缓存（可选）
cd p001-新项目名 && cargo clean

# 4. 更新 README.md 和本文件的项目说明
```

## 代码规范

遵循根目录 [`CLAUDE.md`](../CLAUDE.md) 中的全局约定。
