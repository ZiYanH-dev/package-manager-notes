# Help — Homebrew 学习资料

本目录是 Homebrew 的学习与参考文档集合。

## 工作流速览

装新软件：
1. `brew search` 定位包名；
2. `brew info` 查看版本与依赖；
3. `brew install` 安装；需要常驻运行的软件补 `brew services start`。

日常维护：
1. 每周执行 `brew update` 更新列表；
2. `brew outdated` 查看可升级项；
3. `brew upgrade` 升级；
4. `brew cleanup` 清理旧版本；
5. 定期执行 `brew doctor` 检查环境。

换新电脑迁移：
1. 旧机执行 `brew bundle dump --file=~/Brewfile`；
2. 新机拷贝该文件后执行 `brew bundle`。

详细操作见 `02-search-and-manage.md` 与 `04-daily-workflow.md`。

## 文档导航

| 文件 | 内容 |
| --- | --- |
| `01-basics.md` | Homebrew 的定位、formula 与 cask 的区别、核心目录结构 |
| `02-search-and-manage.md` | 搜索、安装、卸载、更新、版本锁定、服务管理、Tap 管理 |
| `03-dependency-and-cleanup.md` | 依赖查询、版本清理、磁盘占用、环境诊断 |
| `04-daily-workflow.md` | 日常维护、Brewfile、环境迁移、常见问题 |

## 阅读顺序

1. 先读 `01-basics.md` 建立整体模型；
2. 日常安装与卸载查询 `02-search-and-manage.md`；
3. 清理与排错查询 `03-dependency-and-cleanup.md`；
4. 自动化与环境迁移查询 `04-daily-workflow.md`。
