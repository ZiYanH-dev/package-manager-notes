# Help — npm 与 pnpm 学习资料

本目录是 npm 与 pnpm 的对比学习文档集合。

`npm_pnpm_cheatsheet.md` 是命令对照速查表；编号文档按主题讲解机制与原理。

## 工作流速览

从零建项目：
1. `pnpm init` 生成 package.json；
2. `pnpm add <包>` 加依赖；
3. `pnpm dev` 启动开发脚本。

接手已有项目：
1. `git clone` 拉取代码；
2. `pnpm install` 按锁文件装齐依赖；
3. `pnpm dev` 启动。

日常加依赖：
1. `pnpm add <包>` 或 `pnpm add -D <包>`，锁文件自动更新；
2. 把 package.json 与 pnpm-lock.yaml 一起提交 git。

切换到 pnpm：
1. 删除 npm 的锁文件与 node_modules；
2. `pnpm import` 转换旧锁；
3. `pnpm install`。

详细操作见 `05-migration-daily.md` 与 `npm_pnpm_cheatsheet.md`。

## 文档导航

| 文件 | 内容 |
| --- | --- |
| `npm_pnpm_cheatsheet.md` | 命令左右对照：安装、依赖、脚本、锁文件 |
| `01-basics.md` | package.json 字段模型、node_modules 的生成、scripts |
| `02-dependency-and-lock.md` | 直接与间接依赖、锁文件对比、版本约束、peerDependencies、overrides |
| `03-storage-and-disk.md` | 安装机制：npm 扁平化与 pnpm store 加硬链接的对比 |
| `04-strictness-security.md` | pnpm 严格 node_modules、幽灵依赖、v10 构建脚本授权 |
| `05-migration-daily.md` | 日常工作流、npm 到 pnpm 的迁移、.npmrc、真实踩坑记录 |
| `06-files-and-directories.md` | 全部相关文件与目录的位置、作用与可删性 |

## 阅读顺序

1. 先读 `01-basics.md` 建立整体模型；
2. 再读 `02-dependency-and-lock.md` 理解依赖与锁文件；
3. 关注磁盘占用读 `03-storage-and-disk.md`，关注报错读 `04-strictness-security.md`；
4. 想弄清文件分布读 `06-files-and-directories.md`；
5. 实际迁移与排错查询 `05-migration-daily.md` 与 `npm_pnpm_cheatsheet.md`。

## 环境备注

- 配套示例项目为 `/Users/jasonhuang/Desktop/trial/package_manager_trial/npm_pnpm_trial`；
- 真实踩坑来源为 `/Users/jasonhuang/Desktop/trial/web_dev_trial/typescript`，`02`、`04`、`05` 中的坑出自该目录。
