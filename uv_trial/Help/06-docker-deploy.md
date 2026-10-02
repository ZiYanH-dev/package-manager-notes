# 06 — 用 uv 构建 Docker 镜像

## 核心概念速查

**层缓存 layer cache**
Docker 按层复用构建结果的机制；某一层的输入不变时该层直接复用，依赖层与代码层分离的写法是提速关键。

**依赖层隔离**
Dockerfile 的写法原则：先 COPY 依赖声明并安装，最后 COPY 代码；代码改动不会使依赖层缓存失效。

**uv.lock 进入镜像**
锁文件必须 COPY 进镜像；缺失时 uv 退化为按约束现场解析，失去可复现性。

**多阶段构建 multi-stage build**
用 builder 阶段安装依赖、运行阶段只复制 .venv 的减体积写法；可剪掉 uv 二进制与构建工具。

**--no-dev 与 --no-install-project**
镜像构建常用参数；前者跳过开发依赖组，后者只装依赖、不把项目自身装进环境。

## 核心结论

- 装到哪都一样：无论 pip 还是 uv，包最终都装进镜像内部的环境；
- 差别在装法：pip 与 uv 在版本可复现、构建速度、缓存体积三方面表现不同；
- 决定性因素是 Dockerfile 的层缓存写法，而非工具本身。

## pip 与 uv 在 Docker 中的对比

| 维度 | pip | uv | Docker 中的意义 |
| --- | --- | --- | --- |
| 版本可复现 | 依赖手动 `pip freeze`，无锁文件则版本漂移 | uv.lock 钉死精确版本 | 决定构建与 CI 是否稳定 |
| 冷构建速度 | 纯 Python 实现，慢 | Rust 实现，快数倍 | 每次 FROM 都是冷环境，提速直接 |
| 缓存与镜像体积 | 默认缓存写进镜像，需 `--no-cache-dir` | 默认同样写缓存，需 `uv cache clean` | 不清理会增大镜像 |

## Dockerfile 的正确写法

核心原则是依赖层隔离：先装依赖，后拷代码，代码改动不触发依赖层缓存失效。

```dockerfile
FROM python:3.12-slim

# 先只拷贝依赖声明，复用层缓存
COPY pyproject.toml uv.lock ./

# 安装依赖；不装开发依赖，减小体积
RUN pip install uv && uv sync --no-dev --no-install-project

# 最后拷贝代码，代码改动不重装依赖
COPY . .

CMD ["uv", "run", "app.py"]
```

要点：
- pyproject.toml 与 uv.lock 单独一层，改代码时依赖层缓存不失效；
- `--no-dev` 跳过开发依赖组，减小镜像体积；
- `--no-install-project` 只装依赖，不把项目自身装入环境。

## uv 的特有注意点

| 场景 | 说明 |
| --- | --- |
| uv.lock 必须 COPY 进镜像 | 缺失锁文件时退化为网络现场解析 |
| 官方 Python 镜像已是隔离环境 | uv 的环境管理优势在此用不上，多阶段构建除外 |
| 多阶段构建 | 构建阶段用 uv 装依赖，运行阶段只留 .venv，剪掉 uv 二进制 |

多阶段构建的减体积写法：

```dockerfile
# 阶段一：构建，含 uv 与完整工具链
FROM python:3.12-slim AS builder
COPY pyproject.toml uv.lock ./
RUN pip install uv && uv sync --no-dev --frozen --no-install-project \
    && cp -r .venv /venv

# 阶段二：运行，只保留 /venv，不带 uv
FROM python:3.12-slim
COPY --from=builder /venv /venv
COPY . /app
WORKDIR /app
ENV PATH="/venv/bin:$PATH"
CMD ["python", "app.py"]
```

## 结论

| 场景 | 建议 |
| --- | --- |
| 本地开发 | uv，统一管理环境且快 |
| Docker 构建 | uv 更优，快且可复现；层缓存写法才是决定性因素 |
| 共同点 | 包最终都安装在环境内部 |
