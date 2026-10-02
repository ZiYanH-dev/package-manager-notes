# 06 — npm 与 pnpm 的文件与目录全景

## 核心概念速查

**package.json**
项目根目录的声明清单，含名称、脚本、依赖范围；手写或 init 生成，必须提交 git。

**pnpm-lock.yaml 与 package-lock.json**
两个包管理器的锁文件，钉死每包精确版本；必须提交 git，且同一项目只允许存在一个。

**node_modules/**
依赖安装目录，永远不提交 git；可随时删除并由锁文件重建。

**node_modules/.pnpm/**
pnpm 的内部虚拟存储，存放所有包的硬链接与符号链接结构；顶层未声明的子依赖都在这里。

**node_modules/.bin/**
项目依赖中 CLI 工具的符号链接目录；tsc、tsx 等命令由此可用。

**~/.npm/**
npm 的下载缓存目录；只加速下载，不减少项目内 node_modules 的磁盘占用，可安全清理。

**~/Library/pnpm/store/**
pnpm 全局内容寻址仓库，全部项目共享；删除会使所有项目的硬链接失效，只能用 `pnpm store prune` 安全清理。

## 项目内文件，提交 git

| 文件 | 生成方式 | 作用 | 提交 git |
| --- | --- | --- | --- |
| `package.json` | 手写或 `npm init`、`pnpm init` | 项目声明清单：名称、脚本、依赖范围 | 必须 |
| `package-lock.json` | npm 自动生成 | npm 的锁文件，钉死精确版本 | 用 npm 时必须 |
| `pnpm-lock.yaml` | pnpm 自动生成 | pnpm 的锁文件，作用同上 | 用 pnpm 时必须 |
| `.npmrc` | 手写 | 项目级配置：镜像源、提升策略 | 推荐 |
| `pnpm-workspace.yaml` | 手写或 pnpm 自动生成 | 工作区配置与构建脚本授权 allowBuilds | 推荐 |
| `tsconfig.json` | 手写 | TypeScript 编译配置 | 看项目 |

两把锁文件不要同时存在；确定包管理器后删除另一家的锁文件。

## 项目内目录，不提交 git

| 目录 | 生成方式 | 作用 | 提交 git |
| --- | --- | --- | --- |
| `node_modules/` | `npm install`、`pnpm install` | 依赖的实际安装位置 | 永不提交，写入 .gitignore |
| `node_modules/.package-lock.json` | npm 内部 | 扁平结构快照，用于快速比对 | 否 |
| `node_modules/.pnpm/` | pnpm 内部 | 虚拟存储：全部包的硬链接与符号链接 | 否 |
| `node_modules/.bin/` | 两者皆有 | CLI 工具的符号链接 | 否 |
| `dist/`、`build/` | 构建工具 | 编译产物输出 | 通常不提交 |

### node_modules/.pnpm 内部结构

```
node_modules/
├── .pnpm/
│   ├── lodash@4.17.21/
│   │   └── node_modules/
│   │       └── lodash/     ← 硬链接到全局 store 的物理文件
│   └── express@4.18.2/
│       └── node_modules/
│           ├── express/    ← 硬链接到 store
│           └── accepts/    ← express 的子依赖，同样硬链接
├── lodash    → .pnpm/lodash@4.17.21/node_modules/lodash
├── express   → .pnpm/express@4.18.2/node_modules/express
└── .bin/
    └── tsc     → ../typescript/bin/tsc
```

顶层只放显式声明的包，子依赖收纳在 .pnpm 内部；这是 pnpm 阻断幽灵依赖的实现机制。

## 用户级目录，跨项目共享

`pnpm store path` 可随时查看 store 实际路径。

| 路径 | 使用者 | 作用 | 可否删除 |
| --- | --- | --- | --- |
| `~/.npm/` | npm | 下载缓存，内容寻址；只加速下载，各项目 node_modules 仍是独立完整副本 | 可删，`npm cache clean --force`，下次重新下载 |
| `~/.npm/_logs/` | npm | 操作日志 npm-debug.log | 可删，排错时才看 |
| `~/.npm/_npx/` | npx | npx 临时安装的包缓存 | 可删 |
| `~/Library/pnpm/store/v11/` | pnpm | 全局内容寻址仓库，项目通过硬链接引用 | 不可直接删，会使所有项目的硬链接失效而须重新 install；用 `pnpm store prune` 安全清理 |
| `~/Library/pnpm/global/v11/` | pnpm | `pnpm add -g` 安装的全局工具 | 可删，重装即可 |
| `~/Library/pnpm/bin/` | pnpm | pnpm 全局可执行文件 | 不要删 |
| `~/.fnm/` | fnm | fnm 安装的各版本 Node | 可删，重装即可 |

store 是全部项目共享的；同一版本的包全机只存一份物理文件，项目通过硬链接引用。

### 存储结构对比

```
npm 的结构：
  项目 A:  node_modules/lodash   完整副本
  项目 B:  node_modules/lodash   又一份完整副本
  ~/.npm/                        下载缓存 tarball          3.3G

pnpm 的结构：
  ~/Library/pnpm/store/v11/      全局唯一物理副本，按内容哈希   961M
  项目 A:  node_modules/lodash   硬链接 → store
  项目 B:  node_modules/lodash   硬链接 → 同一 store
```

## .npmrc 的三个层级

优先级从高到低为项目级、用户级、全局级：

| 层级 | 路径 | 作用域 |
| --- | --- | --- |
| 项目级 | `项目根/.npmrc` | 当前项目，提交 git 与团队共享 |
| 用户级 | `~/.npmrc` | 本用户全部项目 |
| 全局级 | `$PREFIX/etc/npmrc` | 本机所有用户 |

常用配置：

```ini
registry=https://registry.npmmirror.com/   # 国内镜像
shamefully-hoist=true                      # pnpm 扁平兼容开关，见 04
auto-install-peers=true                    # 自动安装缺失的 peerDependencies
```

## 可安全清理的操作

| 操作 | 命令 | 效果 |
| --- | --- | --- |
| 删除项目 node_modules | `rm -rf node_modules` | 随时可删，`pnpm install` 重建 |
| 清 npm 缓存 | `npm cache clean --force` | 清空 `~/.npm/`，不影响已安装 |
| 清 pnpm 无引用包 | `pnpm store prune` | 删除无任何项目引用的包 |
| 查看 store 占用 | `du -sh ~/Library/pnpm/store/v11` | 本机约 961M |
| 查看 npm 缓存占用 | `du -sh ~/.npm` | 本机约 3.3G |
| 查看项目依赖占用 | `du -sh node_modules` | 单项目依赖体积 |

## 文件关系全景图

```
项目根目录/
├── package.json              ← 手写：依赖声明与脚本
├── pnpm-lock.yaml            ← pnpm 写：精确版本锁定
├── .npmrc                    ← 手写：镜像源与策略
├── pnpm-workspace.yaml       ← 手写：allowBuilds 授权
├── node_modules/             ← 安装产物，不提交 git
│   ├── .pnpm/                ← pnpm 内部虚拟存储
│   ├── .bin/                 ← CLI 工具符号链接
│   ├── lodash/               ← 符号链接指向 store
│   └── express/              ← 符号链接指向 store
└── .gitignore                ← 需包含 node_modules/

~ 用户主目录
├── .npm/                     ← npm 缓存与日志
├── Library/pnpm/
│   ├── store/v11/            ← pnpm 全局物理仓库，全部项目共享
│   ├── global/v11/           ← pnpm 全局安装的工具
│   └── bin/                  ← pnpm 可执行文件
├── .fnm/                     ← fnm 管理的 Node 版本
└── .npmrc                    ← 用户级配置
```
