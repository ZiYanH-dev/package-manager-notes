# npm 与 pnpm — 包管理器核心笔记

本目录是 npm 与 pnpm 的对比学习资料，配套一个最小示例项目。

## 定位

- npm 随 Node.js 一起安装，是生态默认的包管理器，所有教程都以它为基准；
- pnpm 是第三方实现，安装更快、磁盘更省、依赖更严格，由 Vue 与 Vite 团队成员维护，是前端社区的主流推荐。

结论：一个项目只使用一个包管理器；混用会产生两套锁文件并导致冲突，见 `Help/05-migration-daily.md`。

## 安装

```bash
# npm 随 Node.js 附带：nodejs.org 或 fnm
node -v
npm -v

# pnpm 用 npm 安装到全局，或 brew install pnpm
npm i -g pnpm
pnpm -v
```

## 最小示例

```bash
cd npm_pnpm_trial
pnpm add typescript tsx      # 加依赖，生成 pnpm-lock.yaml
pnpm dlx tsx 某文件.ts        # 直接运行 TS 文件，不落地 js
```

## 目录导航

| 文件 | 内容 |
| --- | --- |
| `Help/npm_pnpm_cheatsheet.md` | 命令左右对照表 |
| `Help/01-basics.md` | package.json 字段模型、node_modules 的生成 |
| `Help/02-dependency-and-lock.md` | 依赖类型、锁文件对比、版本约束 |
| `Help/03-storage-and-disk.md` | 安装机制差异与磁盘占用 |
| `Help/04-strictness-security.md` | pnpm 严格性、构建脚本授权 |
| `Help/05-migration-daily.md` | 迁移步骤、.npmrc、踩坑记录 |
| `Help/06-files-and-directories.md` | 文件与目录全景图 |
