# 05 — uv 的文件与目录全景

## 核心概念速查

**pyproject.toml**
项目根目录的声明文件，含元信息、Python 门槛、依赖与工具配置；必须提交 git。

**uv.lock**
uv 生成的精确版本锁文件，覆盖全部间接依赖；必须提交 git，不可手改。

**.python-version**
`uv python pin` 生成的 Python 版本记录文件；推荐提交 git。

**.venv/**
项目虚拟环境，依赖实际安装在 `lib/pythonX.Y/site-packages/`；不提交 git，随时可由 `uv sync` 重建。

**~/.cache/uv/**
uv 的下载缓存，内容寻址，全机共享；可安全清理，下次安装重新下载。

**~/.local/share/uv/**
uv 的持久数据目录，存放下载的 Python 解释器与全局工具；删除会丢失已装解释器与工具。

**archive-v0**
`~/.cache/uv/` 中最大的子目录；解压后的包内容按内容寻址存放，同一包全机只存一份。

## 项目内文件，提交 git

| 文件 | 生成方式 | 作用 | 提交 git |
| --- | --- | --- | --- |
| `pyproject.toml` | 手写或 `uv init` | 项目声明：名称、Python 版本、依赖 | 必须 |
| `uv.lock` | uv 自动生成 | 精确版本锁文件，覆盖间接依赖 | 必须 |
| `.python-version` | `uv python pin` | 项目使用的 Python 版本号 | 推荐 |
| `README.md` | 手写或 `uv init` | 项目说明 | 推荐 |
| `main.py` | `uv init` 生成 | 入口示例文件 | 看项目 |

### pyproject.toml 结构速览

```toml
[project]
name = "my-project"
version = "0.1.0"
requires-python = ">=3.12"       # Python 版本门槛
dependencies = [                  # 直接依赖
    "requests>=2.34.2",
    "fastapi>=0.111.0",
]

[dependency-groups]
dev = ["pytest", "ruff"]          # 开发依赖组，uv add --dev 写入

[project.optional-dependencies]
cuda = ["torch"]                  # 可选依赖组，uv add --optional 写入

[project.scripts]
mycli = "my_module:main"          # 注册 CLI 命令，uv run mycli

[[tool.uv.index]]                 # 镜像源配置，写法见下文与速查表第 12 节
name = "tsinghua"
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
```

## 项目内目录，不提交 git

| 目录 | 生成方式 | 作用 | 提交 git |
| --- | --- | --- | --- |
| `.venv/` | `uv sync`、`uv add` 自动创建 | 虚拟环境，依赖安装位置 | 否，写入 .gitignore |
| `__pycache__/` | Python 运行时 | 字节码缓存 .pyc | 否 |
| `dist/` | `uv build` | 构建产物 .whl 与 .tar.gz | 通常不提交 |
| `*.egg-info/` | `uv build` | 包元信息目录 | 否 |
| `.mypy_cache/`、`.pytest_cache/`、`.ruff_cache/` | 对应工具 | 各工具缓存 | 否 |

### .venv 内部结构

```
.venv/
├── bin/
│   ├── python               ← 符号链接，指向基础 Python
│   ├── pip                  ← venv 自带
│   └── pytest               ← 已安装的 CLI 工具
├── lib/
│   └── python3.12/
│       └── site-packages/   ← 所有依赖包的实际安装位置
│           ├── fastapi/
│           ├── pydantic/
│           └── requests/
└── pyvenv.cfg               ← venv 配置，指向基础 Python
```

.venv/bin/python 的指向由 Python 来源决定：
- 系统已有对应版本时链接到系统 Python，例如 `/Library/Frameworks/Python.framework/` 下；
- 系统没有、由 uv 下载时链接到 `~/.local/share/uv/python/` 内的解释器。

`uv_trial` 的 `.python-version` 是 3.13 且系统已有，因此链接到系统解释器；执行 `uv python pin 3.12` 后会链接到 uv 管理的 3.12。

## 用户级目录，跨项目共享

uv 采用 XDG 风格路径，macOS 与 Linux 一致，不使用 `~/Library/`；`uv cache dir` 与 `uv python dir` 可随时查看实际路径。

| 路径 | 作用 | 可否删除 |
| --- | --- | --- |
| `~/.cache/uv/` | 下载缓存，含 wheel、sdist、索引 | 可删，`uv cache clean`，下次重新下载 |
| `~/.local/share/uv/` | 持久数据：Python 解释器与工具环境 | 不可随意删，会丢失已装解释器与工具 |

### 数据目录结构

```
~/.local/share/uv/                           842M
├── python/                                  ← uv python install 下载的解释器
│   ├── cpython-3.12.13-macos-aarch64-none/    72M
│   ├── cpython-3.12-macos-aarch64-none        符号链接指向上者
│   ├── cpython-3.11.15-macos-aarch64-none/    77M
│   └── cpython-3.11-macos-aarch64-none        符号链接指向上者
└── tools/                                   ← uv tool install 安装的全局工具
    └── aider-chat/                          ← 每个工具一个独立 venv
```

解释器全机共享：`uv python install` 装的解释器只有一份，各项目 .venv 都链接到它。

### 缓存目录结构

```
~/.cache/uv/                                 3.2G
├── archive-v0/                              ← 解压后的包内容，内容寻址    2.9G
├── simple-v20/                              ← PyPI 索引元数据缓存         187M
├── wheels-v6/                               ← 下载过的 wheel              5.2M
├── sdists-v9/                               ← 下载过的源码包              4.0M
├── builds-v0/                               ← 构建缓存
├── interpreter-v4/                          ← 解释器元数据缓存
└── .tmp*/                                   ← 临时文件，可安全清理
```

archive-v0 是内容寻址缓存；同一包无论多少项目使用，只存一份物理内容，这是 uv 省磁盘的核心机制。

## 单文件脚本的临时环境

PEP 723 内联依赖的脚本运行时，uv 按脚本依赖哈希在 `~/.cache/uv/` 内创建临时隔离环境。
这些环境由 `uv cache prune` 清理，无需手动管理。

## 老项目迁移的中间文件

| 文件 | 来源 | 处理 |
| --- | --- | --- |
| `requirements.txt` | 手写、`pip freeze`、`pipreqs` | 迁入 pyproject.toml 后可删 |
| `requirements.lock` | `uv pip compile` | 有了 uv.lock 后不需要 |
| `setup.py`、`setup.cfg` | 旧式打包配置 | 迁移到 pyproject.toml 后可删 |
| `Pipfile`、`Pipfile.lock` | pipenv | 迁到 uv 后可删 |
| `poetry.lock` | poetry | 迁到 uv 后可删 |

## 可安全清理的操作

| 操作 | 命令 | 效果 |
| --- | --- | --- |
| 删除项目 .venv | `rm -rf .venv` | 随时可删，`uv sync` 重建 |
| 清全部缓存 | `uv cache clean` | 清空下载缓存，不影响已安装 |
| 清不活跃缓存 | `uv cache prune` | 只清长期未用条目，推荐定期执行 |
| 查看缓存占用 | `du -sh ~/.cache/uv` | 本机约 3.2G |
| 查看数据目录占用 | `du -sh ~/.local/share/uv` | 本机约 842M，含解释器与工具 |
| 查看项目环境占用 | `du -sh .venv` | 单项目依赖体积 |

## 文件关系全景图

```
项目根目录/
├── pyproject.toml            ← 手写：声明与工具配置
├── uv.lock                   ← uv 写：精确版本锁定，必须提交
├── .python-version           ← uv python pin 写入
├── main.py                   ← 项目代码
├── .venv/                    ← uv 自动管理，不提交
│   ├── bin/python            ← 符号链接，指向系统或 uv 管理的解释器
│   └── lib/python3.12/site-packages/   ← 依赖实际位置
└── .gitignore                ← 需包含 .venv/

~ 用户主目录
├── .cache/uv/                ← 下载缓存
│   ├── archive-v0/           ← 内容寻址缓存，最大头
│   ├── simple-v20/           ← PyPI 索引缓存
│   ├── wheels-v6/            ← wheel 缓存
│   └── sdists-v9/            ← 源码包缓存
└── .local/share/uv/          ← 持久数据
    ├── python/               ← 下载的解释器，全项目共享
    └── tools/                ← 全局工具
```

## 与 npm 和 pnpm 的概念对照

| 概念 | npm 与 pnpm | uv |
| --- | --- | --- |
| 依赖声明文件 | `package.json` | `pyproject.toml` |
| 锁文件 | `package-lock.json`、`pnpm-lock.yaml` | `uv.lock` |
| 依赖安装目录 | `node_modules/` | `.venv/lib/pythonX.Y/site-packages/` |
| 下载缓存 | `~/.npm/`、`~/Library/pnpm/store/` | `~/.cache/uv/` |
| 全局工具 | `~/.fnm/` 全局安装 | `~/.local/share/uv/tools/` |
| 解释器版本管理 | `~/.fnm/` | `~/.local/share/uv/python/` |
| 项目级配置 | `.npmrc` | pyproject.toml 的 `[tool.uv]` |
| 运行时缓存 | 无 | `__pycache__/` 字节码 |
