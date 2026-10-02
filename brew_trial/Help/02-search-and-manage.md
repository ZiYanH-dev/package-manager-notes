# 02 — 搜索、安装与服务管理

## 核心概念速查

**brew search**
按关键词搜索 formula 与 cask 的命令；加 `--cask` 限定只搜 GUI 应用。

**brew install 与 brew uninstall**
安装与卸载命令；`--cask` 处理 GUI 应用，`--zap` 连同配置文件一起删除。

**brew update 与 brew upgrade**
`update` 更新 Homebrew 自身与 formula 列表，`upgrade` 把已安装的包升到新版本；两者职责不同，容易混淆。

**brew services**
管理 formula 自带后台服务的命令；`start` 同时设置开机自启，`run` 只启动一次。

**Tap**
第三方包仓库源；用 `brew tap` 添加，`brew untap` 移除。

## 装新软件的流程

1. `brew search` 检索包名，确认官方源或第三方 Tap 中是否存在；
2. `brew info` 查看版本、依赖与安装方式；
3. `brew install` 安装；
4. 数据库等服务类软件用 `brew services start` 启动并设开机自启。

### 演示记录

以下命令实跑，输出经截取。

```bash
brew search --cask google-chrome
```

```text
google-chat
google-chrome
google-chrome@beta
google-chrome@canary
```

```bash
brew info git
```

```text
==> git: stable 2.55.0 (bottled), HEAD
Distributed revision control system
https://git-scm.com
Not installed
==> Dependencies
Required (2): pcre2, gettext
```

info 输出里 stable 2.55.0 是当前版本，bottled 表示有预编译 bottle，Dependencies 列出直接依赖。

## 搜索与信息查询

```bash
brew search git            # 搜索 formula
brew search --cask chrome  # 搜索 cask
brew info git              # 查看 formula 详情：版本、依赖、安装方式
brew info --cask chrome    # 查看 cask 详情
```

## 安装与卸载

```bash
brew install git                    # 安装 formula
brew install --cask chrome          # 安装 cask
brew install git node               # 一次安装多个
brew install git@3                  # 安装指定大版本
brew uninstall git                  # 卸载 formula
brew uninstall --cask chrome        # 卸载 cask
brew uninstall --zap --cask chrome  # 连同配置文件一起删除
brew rm git                         # uninstall 的别名
```

## 更新与版本锁定

```bash
brew update                     # 更新 Homebrew 自身与 formula 列表
brew outdated                   # 列出所有可升级的包
brew upgrade                    # 升级所有可升级的 formula
brew upgrade git                # 只升级 git
brew upgrade --cask --greedy    # 升级所有 cask，含自动更新的应用
brew pin python@3.13            # 锁定版本，upgrade 时跳过
brew unpin python@3.13          # 解除锁定
```

## 服务管理 brew services

Homebrew 可以管理 formula 自带的后台服务，例如 MySQL、Redis、Nginx：

```bash
brew services list              # 查看所有服务状态
brew services start mysql       # 启动并设置开机自启
brew services stop mysql        # 停止
brew services restart mysql     # 重启
brew services run mysql         # 启动一次，不设置开机自启
```

## Tap 管理

```bash
brew tap                        # 列出已添加的 Tap
brew tap user/repo              # 添加第三方 Tap
brew untap user/repo            # 移除 Tap
```

添加 Tap 之后即可安装其中的 formula；官方源之外的工具大多通过这种方式获取。

## 查看已安装

```bash
brew list                       # 已安装的 formula
brew list --cask                # 已安装的 cask
brew list --versions            # 已安装包及其版本
brew leaves                     # 顶层 formula，即不被其他包依赖的包
brew deps --tree --installed    # 全部已安装包的依赖树
```
