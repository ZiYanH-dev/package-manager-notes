# 02 — 依赖管理与 uv.lock

## 核心概念速查

**直接依赖 direct dependency**
代码中直接 import 的包，写在 `[project].dependencies`；人只声明这一层。

**间接依赖 transitive dependency**
直接依赖自身引入的包；不声明、不手动管理，全部由 uv 解析进 uv.lock。

**uv.lock**
uv 自动生成的锁文件，把全部依赖解析为含哈希的精确版本；保证任何时间、任何机器安装结果一致，必须提交 git。

**版本约束**
pyproject.toml 中对依赖版本的声明方式，例如 `>=`、`==`、`~=` 与范围写法；推荐只写下限。

**dependency-groups**
pyproject.toml 中的开发依赖组，由 `uv add --dev` 写入；`uv sync --no-dev` 可跳过。

**uv pip**
兼容传统 pip 工作流的模式；不维护 uv.lock，与现代项目工作流互斥。

## 依赖的两个层级

### 直接依赖

代码中直接 import 的包，写在 `[project].dependencies`；例如代码 `import fastapi`，则 `fastapi` 是直接依赖。

### 间接依赖

直接依赖自身引入的包；例如 `fastapi` 内部依赖 `starlette`、`pydantic`、`anyio`。
这一层不声明、不手动管理。

职责边界：人只负责直接依赖，间接依赖全部交给 uv。

## uv.lock 的作用

uv.lock 是 uv 自动生成的精确版本锁文件：
- 它把直接依赖与所有层级的间接依赖整体解析；
- 给每个包钉死含哈希的精确版本，例如 `fastapi==0.111.0`；
- 它是人类可读的 TOML，但只通过 `uv add` 与 `uv lock` 维护，不手改。

锁定的理由：pyproject.toml 写的是范围，例如 `fastapi>=0.111.0`；uv.lock 写的是定值。
锁文件保证今天装出的环境，与三个月后他人 `uv sync` 得到的结果完全一致。

```
pyproject.toml  ──uv lock 解析──▶  uv.lock  ──uv sync 安装──▶  .venv/
 "我要 fastapi>=0.111"        "fastapi==0.111.0        实际装好的包
                               starlette==0.37.2       及其精确版本
                               pydantic==2.7.1 ..."
```

## 版本约束写法

| 写法 | 含义 |
| --- | --- |
| `"fastapi>=0.111.0"` | 至少 0.111.0 |
| `"fastapi==0.111.0"` | 精确等于，写死失去升级弹性，不推荐 |
| `"fastapi>=0.111,<0.120"` | 范围 |
| `"fastapi~=0.111.0"` | 兼容版本，等价 `>=0.111.0,<0.112.0` |
| `"uvicorn[standard]>=0.30"` | 带额外功能组 extra |

实践建议：正式依赖只写下限，让 uv 在锁文件中选定兼容的最新版。

## add、sync、lock 的区别

| 命令 | 改 pyproject | 改 uv.lock | 改环境 .venv | 用途 |
| --- | --- | --- | --- | --- |
| `uv add 包` | 是，加入 | 是，更新 | 是，安装 | 新增依赖 |
| `uv remove 包` | 是，删除 | 是，更新 | 是，卸载 | 删除依赖 |
| `uv lock` | 否 | 是，重算 | 否 | 只更新锁文件 |
| `uv sync` | 否 | 否，只读 | 是，对齐 | 按锁文件对齐环境 |

`uv sync` 的行为细节：
- 安装 uv.lock 中记录而环境中缺失的包；
- 删除环境中存在但锁外的多余包，保持严格一致；
- `uv sync --frozen` 不重算锁文件，严格按现有 uv.lock 安装，CI 强烈推荐；
- `uv sync --no-dev` 只装正式依赖，跳过开发依赖组。

三者的分工：`add` 修改声明并同步；`sync` 按锁文件对齐环境；`lock` 只重算锁文件、不动环境。

## 开发依赖与正式依赖

pytest、ruff、mypy 这类只在开发期使用的包放进开发依赖组，不进入生产安装：

```bash
uv add pytest ruff --dev     # 写入 [dependency-groups].dev
```

`uv sync` 默认两组都装；`uv sync --no-dev` 只装正式依赖。

## 升级依赖

```bash
uv lock --upgrade                    # 全部升到兼容的最新版
uv lock --upgrade-package requests   # 只升某一个
uv sync                              # 让环境跟上新锁文件
```

## 查看依赖关系

```bash
uv tree                                  # 完整依赖树
uv tree --depth 2                        # 只看两层
uv tree --package requests               # 以 requests 为根的子树
uv tree --inverted --package requests    # 反向查看谁依赖 requests
```

## 常见误区

1. 手动管理间接依赖是不必要的；uv.lock 全部解析；
2. 不要手改 uv.lock；改 pyproject 后用 `uv add` 或 `uv lock` 重新生成；
3. 不要混用 `uv pip install` 与 `uv sync`；uv pip 不维护锁文件，会造成环境与锁不一致；
4. uv.lock 必须提交 git；pyproject 只是范围，锁文件才是定版，是可复现环境的根本。
