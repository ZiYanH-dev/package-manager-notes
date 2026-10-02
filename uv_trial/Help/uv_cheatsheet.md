# uv 命令速查表

## 两大使用场景

- 场景 A，从零开发：`uv init` → `uv add` → `uv sync` → `uv run`；
- 场景 B，接手项目：`uv sync` → `uv run`。

## 完整工作流

```
1. uv init          建项目，生成 pyproject.toml
2. uv add <pkg>     加依赖，改 pyproject.toml 并自动更新 uv.lock
3. uv sync          按 lock 安装进环境，删除不在 lock 内的多余包
4. uv run <script>  在环境中运行
```

- 项目模式默认使用 .venv，由 uv 自动管理，一般无需手动 `uv venv`；
- uv.lock 必须提交 git，保证可复现；.venv 不提交；
- 换机器或协作者接手时，clone 后执行 `uv sync` 重建环境；复用全局缓存，安装很快。

## 0 基础信息

```bash
uv --version
uv help
uv <command> --help
```

## 1 Python 版本管理

```bash
uv python list                    # 查看本地可用的 Python 版本
uv python install 3.11            # 下载指定版本，无需预先安装
uv python install 3.12
uv python pin 3.11                # 项目锁定版本，写入 .python-version
uv run --python 3.11 main.py      # 用指定版本运行一次
```

## 2 创建项目

```bash
uv init                           # 当前目录初始化，生成 pyproject.toml
uv init --python 3.12 my_project  # 指定 Python 版本与项目名
cd my_project
```

项目目录结构：

```
my_project/
├── pyproject.toml   # 项目元信息与依赖声明
├── uv.lock          # 精确版本锁文件，必须提交 git
├── .venv/           # uv 自动创建的虚拟环境
└── main.py
```

## 3 依赖管理

```bash
uv add httpx pydantic              # 加正式依赖
uv add pytest ruff --dev           # 加开发依赖，写入 dependency-groups
uv add torch --optional cuda       # 加可选依赖组
uv sync --extra cuda               # 安装指定可选组
uv remove httpx                    # 删除依赖
uv tree                            # 查看依赖树
uv tree --depth 2                  # 只看两层
uv lock                            # 按 pyproject.toml 生成或更新锁文件
uv lock --upgrade                  # 全量升级并更新锁
uv lock --upgrade-package httpx    # 只升级单个包
uv sync                            # 同步环境，删除多余包
uv sync --no-dev                   # 只装正式依赖
uv sync --frozen                   # 严格按锁文件安装，不一致时报错，CI 必用
```

## 4 运行代码

```bash
uv run main.py                     # 在项目 .venv 中运行
uv run -c "print('hello uv')"      # 运行单行代码
uv run mycli                       # 运行 [project.scripts] 注册的命令
uv run --with python-dotenv --with rich python demo.py    # 临时附加包运行
uv run --no-project --with httpx python -c "import httpx" # 纯临时环境，不加载当前项目
```

## 5 单文件脚本 PEP 723

新建 demo.py，头部写内联元数据：

```python
# /// script
# requires-python = ">=3.11"
# dependencies = [
#   "httpx>=0.27",
#   "python-dotenv"
# ]
# ///
import httpx
print(httpx.get("https://httpbin.org/ip").json())
```

```bash
uv run demo.py                  # 自动建临时环境，装依赖后运行
uv add --script demo.py rich    # 直接修改脚本内依赖
```

可执行化，适用于 Linux 与 macOS：

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.11"
# dependencies = ["httpx"]
# ///
print("可直接 ./demo.py 运行")
```

```bash
chmod +x demo.py
./demo.py
```

## 6 全局工具 uv tool

```bash
uv tool install ruff              # 安装全局命令行工具
uv tool install jupyter
uv tool list                      # 查看已装工具
uv tool upgrade ruff              # 升级
uv tool uninstall ruff            # 卸载
```

## 7 pip 兼容模式

`uv add` 与 `uv sync` 面向现代项目，即 pyproject.toml 加 uv.lock；`uv pip` 面向传统 pip 与 requirements.txt 工作流。

```bash
uv pip install requests                       # 等价 pip install，速度更快
uv pip install -r requirements.txt            # 从清单批量安装
uv export -o requirements.txt                 # 把 uv.lock 导出为 requirements.txt
uv pip sync requirements.txt                  # 严格按清单同步，删除多余包
uv pip compile pyproject.toml -o requirements.lock   # pip-tools 风格锁定
```

## 8 缓存管理

```bash
uv cache dir                      # 查看缓存位置
uv cache clean                    # 全部清理
uv cache prune                    # 清理长期未用条目，推荐定期执行
```

## 9 依赖漏洞扫描

```bash
uv audit
```

## 10 构建与发布

```bash
uv build                          # 构建 wheel 与 sdist，输出到 dist/
uv publish                        # 发布到 PyPI
```

## 11 CI 与镜像源

```bash
export PIP_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple   # 国内镜像
uv sync --frozen                  # CI 中严格按锁文件安装
```

GitHub Actions 中 `uv sync --frozen` 保证按提交的 uv.lock 复现环境。

## 12 pyproject.toml 常用片段

```toml
[project]
name = "my-db"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
  "pydantic>=2.0"
]

[project.optional-dependencies]
dev = ["pytest", "ruff"]
viz = ["matplotlib"]

[[tool.uv.index]]
name = "pytorch-cu124"
url = "https://download.pytorch.org/whl/cu124"

[tool.uv.sources]
torch = { index = "pytorch-cu124" }
```

旧写法 `[tool.uv] extra-index-url` 已废弃；配置镜像源与私有索引统一用 `[[tool.uv.index]]`，需要把包绑定到指定索引时再配 `[tool.uv.sources]`。

## 关键概念区分

1. `uv sync`：读 uv.lock，使环境与锁文件完全一致，会删除不在清单内的包；
2. `uv add`：修改 pyproject.toml，并自动执行 lock 更新锁文件；
3. `uv pip install` 与 `uv sync` 不要混用：uv pip 不维护 uv.lock，现代项目统一用 add 与 sync；
4. `.venv` 由 uv 自动管理，一般无需手动 `uv venv`。

## 场景选择

- 单文件原型与实验脚本，用 PEP 723 单文件模式；
- 正式项目与需要 CI 的项目，用 `uv init` 加 uv.lock 工作流；
- 带 requirements.txt 的老项目，先用 `uv pip` 过渡，再迁移到 pyproject.toml。
