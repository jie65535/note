---
tags: #rust
---
# 安装Rust环境

[官方文档](https://doc.rust-lang.org/cargo/getting-started/installation.html)

从`https://sh.rustup.rs`获取安装脚本，执行脚本安装

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source $HOME/.cargo/env && rustup default nightly && rustup update
```

建议切换到`nightly`通道，更新使用。

---

## 更换国内源
使用时发现它还得上github获取索引，如果访问github有困难，可以换源。
在 `~/.cargo/` 目录下 `touch config`，添加如下内容：
```
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index"
```
就可以更改为国内源

要通过系统 `git` 访问 `github` 的话，还可以加上以下内容
```
[net]
git-fetch-with-cli = true
```
如果有 `Cargo` 不支持的特殊身份验证要求，将其设置为 `true` 会很有帮助。否则会使用 `cargo` 自带的 `git`。