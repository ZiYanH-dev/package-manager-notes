# 01 — uv 项目基础与 pyproject.toml

## 核心概念速查

**uv**
Rust 实现的 Python 包管理与环境管理工具；用单个工具替代 pip、venv、pyenv、pipx、pip-tools 的功能。

**pyproject.toml**
项目的声明文件，位于项目根目录，写明名称、Python 版本要求与直接依赖；由人维护。

**uv.lock**
uv 自动生成的精确版本锁文件，覆盖直接与全部间接依赖；TOML 格式可读但不可手改，必须提交 git。

**.venv/**
项目虚拟环境目录，依赖的实际安装位置；由 uv 自动创建与维护，不提交 git。

**uv init**
项目初始化命令，生成 pyproject.toml 与示例文件；不生成 uv.lock 与 .venv，二者在首次 add 或 sync 时出现。

**uv add**
添加依赖的命令；修改 pyproject.toml 后自动执行 lock 并同步环境。

**uv sync**
按 uv.lock 对齐环境的命令；安装缺失包，并删除锁外的多余包。

**uv run**
在项目 .venv 中执行脚本或命令的前缀；无需手动 activate。

## uv 的定位

uv 是 Rust 实现的 Python 包管理器，用单个工具覆盖传统工具链的职责：

| 传统工具 | 职责 | uv 对应 |
| --- | --- | --- |
| pip | 安装包 | `uv add`、`uv pip install` |
| venv | 创建虚拟环境 | 自动管理 `.venv` |
| pyenv | 管理 Python 版本 | `uv python install`、`uv python pin` |
| pipx | 安装全局命令行工具 | `uv tool install` |
| pip-tools | 锁定版本 | `uv.lock` 自动锁定 |

它的定位是统一管理 Python 解释器版本、虚拟环境与依赖的单一工具。

## uv 项目的文件构成

以 `uv_trial/` 为例：

```
uv_trial/
├── pyproject.toml   # 项目声明：名称、Python 版本、依赖
├── uv.lock          # 精确版本锁文件，必须提交 git
├── .venv/           # 虚拟环境，uv 自动创建
├── main.py
└── Help/
```

三个关键分工：
- pyproject.toml 由人维护，声明项目元信息与直接依赖；
- uv.lock 由 uv 生成，把直接与间接依赖解析为精确版本并锁死；它是 TOML 格式但不手改；
- .venv 不提交 git；依赖靠 uv.lock 在任何机器重建。

## 项目的三种生成方式

### 方式 A：从零初始化

```bash
uv init                          # 当前目录生成 pyproject.toml 骨架
uv init --python 3.12 my_project # 指定 Python 版本与项目名
cd my_project
```

`uv init` 生成 pyproject.toml、示例 main.py 与 README.md；uv.lock 与 .venv 在首次 `uv add` 或 `uv sync` 时才出现。

### 方式 B：从 requirements.txt 迁移

```bash
uv init --bare                # 只生成最小 pyproject.toml，不覆盖现有文件
uv add -r requirements.txt   # 批量登记原有依赖
```

执行后 uv 生成 uv.lock，旧项目即转为 uv 项目。

### 方式 C：既有项目观察

`study_helper/backend` 是已建成的 uv 项目；其 pyproject.toml 中 22 个依赖为显式声明，uv.lock 是 uv 解析出的全量精确版本，对应 `uv init` 之后逐步 `uv add` 的结果。

## pyproject.toml 模型

以 `uv_trial/pyproject.toml` 为例：

```toml
[project]
name = "uv-trial"
version = "0.1.0"
requires-python = ">=3.12"      # 项目支持的 Python 范围
dependencies = [
    "requests>=2.34.2",         # 直接依赖，只列顶层包
]
```

- dependencies 只放直接依赖，即代码中 import 的顶层包；间接依赖由 uv 在 uv.lock 中解析；
- 版本写约束而非写死，例如 `>=2.34.2` 表示下限，uv 在锁文件中选定兼容的最新版；
- requires-python 是版本门槛，uv 据此筛选兼容的解释器与包版本。

## 容易混淆的命令

| 命令 | 行为 |
| --- | --- |
| `uv add 包名` | 改 pyproject.toml，自动 lock，自动装进环境 |
| `uv sync` | 读 uv.lock，把环境对齐为锁文件状态，删除多余包 |
| `uv lock` | 只重算 uv.lock，不动环境 |
| `uv run xxx` | 用项目 .venv 执行脚本或命令 |
| `uv pip ...` | 兼容传统 pip 的模式，不维护 uv.lock |

现代项目优先使用 `uv add` 与 `uv sync`；混用 `uv pip install` 会导致锁文件失同步，详见 `02-dependency-management.md`。
