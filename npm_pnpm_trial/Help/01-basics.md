# 01 — 基础：package.json 与 node_modules 的生成

## 核心概念速查

**package.json**
项目的声明清单，位于项目根目录，写明名称、脚本、直接依赖及版本范围；包管理器据此安装依赖。

**锁文件 lock file**
包管理器自动生成的精确版本记录，npm 为 `package-lock.json`，pnpm 为 `pnpm-lock.yaml`；实际安装的版本以它为准。

**node_modules/**
依赖的实际安装目录，由 install 命令生成；可随时删除并从锁文件完整重建，因此不提交 git。

**dependencies**
package.json 中的正式依赖字段；产物运行时需要的包写在这里，会进入生产安装。

**devDependencies**
开发依赖字段；只在开发期使用，例如 TypeScript、tsx 与构建工具，不进入生产安装。

**scripts**
package.json 中的命令别名区；`npm run dev` 或 `pnpm dev` 执行其中的命令。

## 包管理器解决的问题

手写项目要自行下载、摆放、更新几十个第三方库，且版本难以对齐。包管理器做三件事：
1. 按 package.json 的声明自动下载依赖到 node_modules；
2. 用锁文件记录精确版本，保证本机、同事与 CI 装出的结果一致；
3. 提供 scripts 统一执行构建、测试与启动命令。

## package.json 核心字段

```jsonc
{
  "name": "my-app",
  "version": "1.0.0",
  "type": "module",                // module 为 ESM；省略或 commonjs 为 CJS
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc"
  },
  "dependencies": {                // 正式依赖，产物运行需要
    "lodash": "^4.17.21"
  },
  "devDependencies": {             // 开发依赖，只在开发期
    "typescript": "^5.6.3",
    "tsx": "^4.19.2"
  }
}
```

- `type: "module"` 声明 ESM 模块体系；省略或写 commonjs 则按 CJS 处理；
- dependencies 与 devDependencies 的边界：前者进入生产，后者只在开发环境；vite、tsx、TypeScript 等构建工具都应放 dev；
- scripts 把常用命令收口为短别名，团队成员无需记忆完整参数。

## node_modules 的生成

```bash
npm install     # 读 package.json → 下载 → 写 node_modules/ 与 package-lock.json
pnpm install    # 同上，但写 pnpm-lock.yaml；node_modules 采用符号链接结构，见 03
```

node_modules 永远不提交 git，写入 .gitignore；它可以从锁文件完整重建，仓库中提交的是 package.json 与锁文件。

## 初始化与常用命令

```bash
pnpm init                  # 生成 package.json
pnpm add typescript tsx    # 加依赖，自动写入 dependencies 或 devDependencies
pnpm dev                   # 执行 scripts.dev
```

## 与 uv 的概念对照

| 概念 | uv | npm 与 pnpm |
| --- | --- | --- |
| 依赖声明文件 | `pyproject.toml` | `package.json` |
| 锁文件 | `uv.lock` | `package-lock.json` 与 `pnpm-lock.yaml` |
| 安装目录 | `.venv/` | `node_modules/` |
| 加依赖 | `uv add` | `pnpm add` 或 `npm i` |
