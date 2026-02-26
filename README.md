# R-universe 配置指南

## 概述

R-universe 会自动从 GitHub 拉取源码、构建 R 包并提供 CRAN 风格的安装方式。

## 配置步骤

### 1. 创建 universe 仓库

在 GitHub 上创建仓库：`biodancerwangzhi/universe`

要求：
- 仓库名必须是 `universe`（固定名称）
- 必须是 public 仓库

### 2. 添加 packages.json

将本目录下的 `packages.json` 放到 `universe` 仓库根目录：

```json
[
  {
    "package": "crosscell",
    "url": "https://github.com/biodancerwangzhi/crosscell",
    "subdir": "crosscell-r"
  }
]
```

字段说明：
- `package`: R 包名（对应 DESCRIPTION 中的 Package 字段）
- `url`: 源码仓库地址
- `subdir`: R 包在仓库中的子目录

### 3. 注册 R-universe

访问 https://r-universe.dev/add 用 GitHub 账号登录，它会自动检测 `universe` 仓库。

### 4. 等待构建

注册后 R-universe 会：
1. 克隆 `biodancerwangzhi/crosscell` 完整仓库
2. 进入 `crosscell-r/` 子目录
3. 执行 `R CMD build` + `R CMD INSTALL`
4. 构建 Linux/macOS/Windows 二进制包

首次构建可能需要 30-60 分钟（Rust 编译较慢）。

### 5. 用户安装方式

构建成功后，用户可以：

```r
install.packages("crosscell", repos = "https://biodancerwangzhi.r-universe.dev")
```

## 构建依赖

R-universe 构建服务器需要：
- Rust toolchain（R-universe 已预装）
- CMake >= 3.26（HDF5 静态编译需要）

`crosscell-r/src/rust/Cargo.toml` 中已配置 `[workspace]` 和 `hdf5-static` feature，
确保在 R-universe 环境下能独立编译，不依赖根 workspace。

## 触发重新构建

R-universe 会在以下情况自动重新构建：
- `universe` 仓库的 `packages.json` 更新
- 源码仓库有新的 commit/tag
- 手动在 R-universe dashboard 触发

## 排查构建失败

查看构建日志：
```
https://biodancerwangzhi.r-universe.dev/crosscell
```

常见问题：
1. CMake 版本不够 → 检查 R-universe 是否支持，可能需要在 `configure` 脚本中安装
2. Cargo workspace 冲突 → 确认 `crosscell-r/src/rust/Cargo.toml` 有 `[workspace]`
3. 路径依赖找不到 → `path = "../../.."` 需要完整仓库克隆才能工作

## 相关文件

```
crosscell-r/
├── DESCRIPTION          # R 包元数据
├── NAMESPACE            # 导出函数
├── R/                   # R 代码
├── src/
│   ├── entrypoint.c     # extendr 入口
│   ├── Makevars         # Linux/macOS 编译配置
│   ├── Makevars.win     # Windows 编译配置
│   └── rust/
│       ├── Cargo.toml   # Rust 依赖（独立 workspace）
│       └── src/
│           └── lib.rs   # Rust 绑定代码
├── .Rbuildignore        # R CMD build 排除规则
└── LICENSE
```
