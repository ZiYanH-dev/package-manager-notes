# 04 — 日常工作流与排错

## 核心概念速查

**uv sync**
接手项目的第一步；创建 .venv 并按 uv.lock 装齐依赖，同时删除锁外的多余包。

**uv run**
在项目环境中执行脚本与命令的前缀；自动使用 .venv，无需手动 activate。

**PEP 723 内联元数据**
单文件脚本的标准：在 .py 文件头部用注释块声明依赖；`uv run 脚本.py` 自动建临时环境运行。

**uv tool**
安装与管理全局命令行工具的命令组；替代 pipx，每个工具一个独立环境。

**--frozen**
严格按现有 uv.lock 安装的参数；不重算版本，CI 必用。

## 标准工作流

克隆新项目后的首次启动：

```bash
cd backend          # 进入含 pyproject.toml 的目录
uv sync             # 创建虚拟环境并按 uv.lock 装齐依赖
uv run main.py      # 运行代码
uv run pytest       # 运行测试
```

日常新增依赖：

```bash
uv add httpx          # 加正式依赖
uv add pytest --dev   # 加开发依赖
uv run main.py        # add 后环境已同步，直接运行
```

依赖变更后提交两个文件，协作者 `uv sync` 即可对齐：

```bash
git add pyproject.toml uv.lock
git commit -m "add httpx"
```

### 演示记录

以下命令在 uv_trial 项目中实跑，输出为真实结果。

```bash
uv sync
```

```text
Resolved 6 packages in 14ms
Checked 5 packages in 3ms
```

```bash
uv run main.py
```

```text
Hello from uv-trial!
```

```bash
uv tree
```

```text
uv-trial v0.1.0
└── requests v2.34.2
    ├── certifi v2026.7.22
    ├── charset-normalizer v3.5.1
    ├── idna v3.19
    └── urllib3 v2.7.0
```

uv tree 中 requests 之下的五个包，就是 uv.lock 替你解析的间接依赖；项目里从未手写过它们。

## 脚本运行方式

```bash
uv run main.py                  # 用项目 .venv 运行
uv run -c "print('hi')"         # 运行单行代码
uv run --with rich python x.py  # 临时附加一个包运行，不写入项目依赖
uv run --python 3.11 main.py    # 指定 Python 版本运行一次
```

## PEP 723 单文件脚本

不建完整项目、只写单个 .py 并自带依赖时，在文件头部写内联元数据块：

```python
# /// script
# requires-python = ">=3.11"
# dependencies = ["httpx>=0.27", "python-dotenv"]
# ///
import httpx
print(httpx.get("https://httpbin.org/ip").json())
```

```bash
uv run demo.py                  # uv 自动建临时环境，装好依赖后运行
uv add --script demo.py rich    # 直接向脚本块追加依赖
```

适用场景：一次性数据处理、模型实验、发给同事的单文件可运行工具。

## 全局工具 uv tool

```bash
uv tool install ruff            # 安装全局命令行工具
uv tool list                    # 查看已装工具
uv tool upgrade ruff
uv tool uninstall ruff
```

## requirements.txt 兼容

```bash
uv pip install -r requirements.txt    # 传统方式安装，不维护 uv.lock
uv export -o requirements.txt         # 把 uv.lock 导出为 requirements.txt
uv pip compile pyproject.toml -o requirements.lock  # pip-tools 风格的锁定编译
```

现代项目不要长期停留在 `uv pip` 工作流；迁移完成后改用 `uv add` 与 `uv sync`，否则锁文件失同步。

## 常见报错与排查

### 版本冲突 No solution found

通常是 pyproject.toml 中约束互相矛盾，例如 A 要求 `django<5` 而 B 要求 `django>=5`；用 `uv tree 包名` 定位依赖来源，放宽或统一约束后执行 `uv lock`。

### Python version not found

requires-python 要求的版本本地不存在；执行 `uv python install 3.12` 让 uv 下载。

### command not found

直接敲了 pytest 而环境未激活；统一用 `uv run pytest`，uv 自动使用 .venv 中的可执行文件，通常无需手动 `source .venv/bin/activate`。

### Lock file is not up to date

pyproject.toml 已改而 uv.lock 未跟上，例如手改了 toml；执行 `uv lock` 重新生成，或直接 `uv sync` 自动跟进。

### uv sync 删除了某个包

这是设计行为；sync 会清掉不在 uv.lock 中的包，保持环境严格一致。
用 `uv pip install` 手动装的包会被移除，因此依赖变更统一走 `uv add`。

### 国内下载慢

```bash
export PIP_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple
uv sync
```

或在 pyproject.toml 的 `[[tool.uv.index]]` 中配置镜像源，见速查表第 12 节。

## 缓存管理

```bash
uv cache dir      # 查看缓存位置
uv cache clean    # 全部清理，空间不足时用
uv cache prune    # 只清长期不用的条目，推荐定期执行
```

## 安全扫描

```bash
uv audit          # 基于 PyPI advisory 数据库扫描依赖漏洞
```

## CI 与 GitHub Actions

```yaml
- uses: actions/setup-python@v5
- run: curl -LsSf https://astral.sh/uv/install.sh | sh
- run: uv sync --frozen     # 严格按锁文件安装，不重算版本
- run: uv run ruff check .
- run: uv run pytest
```

`--frozen` 的意义：CI 中不允许 uv 自行升版本，严格按提交的 uv.lock 复现环境。

## 核心规则清单

1. 依赖变更用 `uv add` 与 `uv remove`，不用 `uv pip install`；
2. uv.lock 必须提交 git；
3. 运行代码统一加 `uv run` 前缀，不手动 activate；
4. `uv sync` 删除锁外多余包是设计行为；
5. 从代码反推依赖用 pipreqs 加 `uv add -r`，见 `03-auto-generate-deps.md`。
