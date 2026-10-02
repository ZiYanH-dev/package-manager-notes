# 03 — 安装机制与磁盘占用

## 核心概念速查

**内容寻址仓库 content-addressable store**
pnpm 的全局包存储，位于 `~/Library/pnpm/store/`；文件按内容哈希存放，同一版本全机只存一份物理文件。

**硬链接 hard link**
项目 node_modules 与全局 store 之间的链接方式；多个项目共享同一份磁盘数据，不产生额外占用。

**符号链接 symlink**
node_modules 顶层包指向 `.pnpm/` 内部真实目录的链接；是 pnpm 严格结构的实现基础。

**扁平化 hoist**
npm v3 起的 node_modules 结构，把所有包尽量摊平到顶层；代价是产生幽灵依赖。

**幽灵依赖 phantom dependency**
未在 package.json 声明、却因扁平化结构而能 import 的包；pnpm 的严格结构使这类 import 直接报错，与 hoist 行为相对。

**pnpm store prune**
清理 store 中无任何项目引用的包的命令；磁盘回收的首选安全手段。

## npm 结构的演进

1. npm v2 及之前采用嵌套结构，每个包把依赖装在自己目录下，形成深树；磁盘开销大且 Windows 路径过长；
2. npm v3 起改为扁平化 hoist，把所有包尽量摊平到顶层；结构变浅，但引入幽灵依赖问题，详见 04；
3. dedupe 去重让相同版本只留一份；但多个项目各自维护完整 node_modules，跨项目不共享。

## pnpm 的机制

pnpm 采用与 npm 完全不同的结构：
1. 全局内容寻址仓库：所有包装进 `~/Library/pnpm/store/`，按内容哈希存储，同一版本只存一份物理文件；
2. 硬链接：项目 node_modules 内的文件硬链接到 store；多项目安装同一包时共享磁盘数据，几乎不产生额外占用；
3. 符号链接结构：顶层只放显式声明的包并指向 store；子依赖收纳在 `.pnpm/` 内部，不暴露到顶层。

```
全局 store:  ~/Library/pnpm/store/v11/   lodash@4.17.21 只存一份物理文件
项目 A:      node_modules/lodash   硬链接 → store
项目 B:      node_modules/lodash   硬链接 → 同一 store
```

## 效果对比

| | npm 扁平结构 | pnpm store 加硬链接 |
| --- | --- | --- |
| 10 个项目安装 lodash | 10 份完整副本 | 1 份物理文件 |
| 安装速度 | 每次复制，较慢 | 多数只建链接，快 |
| 磁盘敏感场景 | 占用随项目数线性增长 | 多项目共享，占用低 |

## 常用命令

```bash
pnpm store path               # 查看全局 store 路径
pnpm store prune              # 删除 store 中无任何项目引用的包
du -sh "$(pnpm store path)"   # 查看 store 占用
```

磁盘紧张时定期执行 `pnpm store prune`，并删除不再使用的项目的 node_modules。
npm 侧对应的是 `npm cache clean --force`；但 npm 缓存只加速下载，各项目的 node_modules 仍是独立完整副本。
