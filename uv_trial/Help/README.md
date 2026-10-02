# Help — uv 学习资料

本目录是 uv 的学习与参考文档集合。

`uv_cheatsheet.md` 是命令速查表；编号文档按主题讲解机制与日常工作方式。

## 工作流速览

从零建项目：
1. `uv init` 生成 pyproject.toml；
2. `uv add <包>` 加依赖；
3. `uv run main.py` 运行。

接手已有项目：
1. `git clone` 拉取代码；
2. `uv sync` 按锁文件重建环境；
3. `uv run main.py` 运行。

日常加依赖：
1. `uv add <包>` 加正式依赖，`uv add <包> --dev` 加开发依赖，锁文件自动更新；
2. 把 pyproject.toml 与 uv.lock 一起提交 git。

升级与 CI：
1. 升级执行 `uv lock --upgrade` 后 `uv sync`；
2. CI 用 `uv sync --frozen` 严格按锁文件安装。

详细操作见 `04-daily-workflow.md` 与 `uv_cheatsheet.md`。

## 文档导航

| 文件 | 内容 |
| --- | --- |
| `uv_cheatsheet.md` | 命令速查：基础、项目、脚本、pip 兼容、缓存、CI |
| `01-project-basics.md` | uv 项目的构成、文件结构、pyproject.toml 模型 |
| `02-dependency-management.md` | 直接与间接依赖、uv.lock、版本约束、add 与 sync 的区别 |
| `03-auto-generate-deps.md` | 按代码 import 反推依赖：pipreqs 与 uv add -r |
| `04-daily-workflow.md` | 日常工作流、PEP 723 脚本、uv tool、报错排查、CI |
| `05-files-and-directories.md` | 全部相关文件与目录：pyproject、.venv、缓存、Python 安装位置 |
| `06-docker-deploy.md` | 用 uv 构建 Docker 镜像：层缓存与多阶段构建 |

## 阅读顺序

1. 先读 `01-project-basics.md` 建立整体模型；
2. 再读 `02-dependency-management.md` 理解依赖与锁文件；
3. 批量生成依赖声明读 `03-auto-generate-deps.md`；
4. 弄清文件分布读 `05-files-and-directories.md`；
5. 实际开发与排错查询 `04-daily-workflow.md` 与 `uv_cheatsheet.md`。

## 环境备注

- 配套示例项目为 `/Users/jasonhuang/Desktop/trial/package_manager_trial/uv_trial`；
- 另一个真实 uv 项目为 `study_helper/backend`，可对照观察其 pyproject.toml 与 uv.lock。
