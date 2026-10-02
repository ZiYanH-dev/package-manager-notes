# 05 — npm 到 pnpm 的迁移与日常排错

## 核心概念速查

**pnpm import**
迁移命令，读取现有 `package-lock.json` 生成 `pnpm-lock.yaml`；避免重新解析整棵依赖树。

**.npmrc**
npm 与 pnpm 共用的项目级配置文件，可设镜像源与依赖策略；pnpm 另有专属配置项。

**两套锁文件问题**
`package-lock.json` 与 `pnpm-lock.yaml` 同时存在时安装依据不明确；迁移完成后必须删除旧锁文件。

**pnpm dlx tsx**
临时执行 TypeScript 文件的方式；Node 只执行纯 JS，TS 语法需 tsx 内存转译后运行。

## 日常工作流

从零建项目：
1. `pnpm init` 生成 package.json；
2. `pnpm add <包>` 加依赖，自动写入依赖字段并更新锁文件；
3. `pnpm dev` 执行 scripts.dev。

接手已有项目：
1. `git clone` 拉取代码；
2. `pnpm install` 按锁文件装齐依赖；
3. `pnpm dev` 启动。

日常加依赖：
1. `pnpm add <包>` 加正式依赖，`pnpm add -D <包>` 加开发依赖；
2. 把 package.json 与 pnpm-lock.yaml 一起提交 git。

CI 或验证一致性：
1. 用 `pnpm install --frozen-lockfile` 严格按锁文件安装。

磁盘维护：
1. 定期执行 `pnpm store prune` 回收全局 store 空间。

### 演示记录

以下命令在 npm_pnpm_trial 项目中实跑；接手时目录里只有 package.json，没有 node_modules 与锁文件。

```bash
pnpm install
```

```text
.../esbuild@0.28.2/node_modules/esbuild postinstall: Done

devDependencies:
+ tsx 4.23.15
+ typescript 5.6.3 (7.0.2 is available)

Done in 1.5s using pnpm v11.22.0
```

这次 install 生成了 node_modules 与 pnpm-lock.yaml；esbuild 的 postinstall 正常执行，对应 04 的 allowBuilds 授权。

```bash
pnpm ls --depth 0
```

```text
├── tsx@4.23.15
└── typescript@5.6.3

2 packages
```

```bash
pnpm install --frozen-lockfile
```

```text
Already up to date
Done in 325ms using pnpm v11.22.0
```

```bash
pnpm store path
```

```text
/Users/jasonhuang/Library/pnpm/store/v11
```

## 迁移步骤

```bash
# 1. 删除 npm 的锁文件与 node_modules
rm -rf package-lock.json node_modules

# 2. 可选：读取旧锁转换，省去重新解析
pnpm import          # 生成 pnpm-lock.yaml

# 3. 用 pnpm 安装
pnpm install
```

迁移完成后目录中只保留 `pnpm-lock.yaml`；两套锁文件并存的后果见下文真实踩坑第一例。

## .npmrc 项目级配置

npm 与 pnpm 共用 `.npmrc` 语法；pnpm 另有专属配置项：

```ini
# .npmrc
registry=https://registry.npmmirror.com/    # 国内镜像，加速下载
auto-install-peers=true                     # 自动安装缺失的 peerDependencies
strict-peer-dependencies=false
shamefully-hoist=true                       # 兼容老项目的扁平开关，见 04
```

## 真实踩坑记录

来源为 `web_dev_trial/typescript` 项目的实际排错。

### 两套锁文件并存

- 现象：目录中同时存在 `package-lock.json` 与 `pnpm-lock.yaml`；
- 原因：先用 `npm install` 又用 `pnpm install`，两个包管理器各写各的锁；
- 处理：确定使用 pnpm 后删除 `package-lock.json`。

### allowBuilds 占位符未生效

- 现象：pnpm 自动生成的 `pnpm-workspace.yaml` 中，esbuild 一行的值是占位文本，未生效；
- 影响：esbuild 的构建脚本未执行，干净机器重装后运行异常；
- 处理：把值改为真实布尔值：

```yaml
allowBuilds:
  esbuild: true
```

### Node 直接运行 TypeScript 报错

- 现象：`node xxx.ts` 在 `enum Role {` 处报错；
- 原因：Node 只执行纯 JS，无法识别 TypeScript 类型语法；
- 处理：用 `pnpm dlx tsx xxx.ts` 内存转译直接执行；或先用 `pnpm exec tsc xxx.ts` 生成 js，再用 node 运行。

## 常见报错对照

| 报错 | 含义 | 处理 |
| --- | --- | --- |
| `Cannot find module 'x'` | 未安装，或严格模式拦截未声明依赖 | `pnpm add x` |
| `ERR_PNPM_BUILD_DISALLOWED` | 构建脚本被拦截 | `pnpm-workspace.yaml` 中设 `allowBuilds.x=true` |
| 安装后功能异常 | 原生二进制未编译 | `pnpm rebuild <包>` |
| 锁文件与 package.json 不一致 | 严格安装报错 | 重新 `pnpm install` 并提交新锁文件 |
