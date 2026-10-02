# 01 — 基础：Homebrew 的工作模型

## 核心概念速查

**Formula**
Homebrew 的包定义脚本，位于官方或第三方 Tap 仓库中，描述一个命令行工具或库的下载、编译与安装方式；`brew install` 不加 `--cask` 时安装的就是 formula。

**Cask**
Homebrew 的 macOS GUI 应用定义，安装到 `/Applications/`；安装时必须显式加 `--cask`，与 formula 相区分。

**Cellar**
`/opt/homebrew/` 下的目录，是所有 formula 的实际安装位置；每个 formula 的每个版本占一个子目录。

**Keg**
Cellar 中某个 formula 的一个具体版本目录，例如 `git/4.0.0`；升级后新旧版本以多个 Keg 的形式并存。

**Tap**
Formula 与 cask 的仓库源，默认使用官方源；可用 `brew tap` 添加第三方仓库。

**Bottle**
预编译的二进制包，安装时直接下载而无需本地编译，是 Homebrew 默认的安装方式。

**Keg-only**
Formula 的一种属性，表示不向 `bin/` 创建符号链接；用于避免与系统自带软件冲突，例如 `openjdk`。

## Homebrew 解决的问题

macOS 不自带包管理器，手动安装软件存在三类成本：
- 图形应用要去官网下载 `.dmg` 并手动拖入 Applications，更新与卸载都要手动处理；
- 命令行工具要用 `curl` 下载源码再执行 `./configure && make && make install`；
- 版本管理与升级完全靠自己维护。

Homebrew 用统一的命令完成安装、更新、卸载，并自动处理依赖。

## Formula 与 Cask 的区别

| | Formula | Cask |
|---|---|---|
| 安装对象 | 命令行工具与库 | GUI 桌面应用 |
| 安装路径 | `/opt/homebrew/bin/` 链接自 Cellar | `/Applications/` |
| 示例 | `brew install git` | `brew install --cask google-chrome` |
| 调用方式 | 默认即 formula | 必须显式加 `--cask` |

## 核心目录结构

```
/opt/homebrew/              # Apple Silicon 路径；Intel 机型在 /usr/local/
├── Cellar/                 # 所有 formula 的实际安装位置
│   ├── git/4.0.0/
│   └── node/22.0.0/
├── bin/                    # 符号链接，指向 Cellar 内的可执行文件
├── Caskroom/               # 所有 cask 的实际安装位置
├── etc/                    # 配置文件
├── var/                    # 运行时数据，如日志与数据库
├── Homebrew/               # Homebrew 自身代码
└── Library/                # 官方 Tap 与 formula 定义
```

Cellar 中的每个版本目录就是一个 Keg；`bin/` 里的可执行文件是指向当前版本 Keg 的符号链接。
