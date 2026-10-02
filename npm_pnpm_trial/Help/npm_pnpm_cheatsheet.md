# npm 与 pnpm 命令速查表

## 包管理器的通用流程

npm 与 pnpm 遵循同一条流程：
1. 创建项目：执行 init 生成 package.json；该文件只是声明清单，此时未下载任何包；
2. 添加依赖：每次 add 更新 package.json，把包登记进清单；
3. 锁定版本：install 生成或复用锁文件，把本次解析出的精确版本钉死；
4. 落地下载：包文件写入 node_modules；npm 复制文件，pnpm 用硬链接与符号链接指向全局 store；
5. 查找引用：代码 import 时，Node 从当前目录逐级向上查找 node_modules，找到即用。

包管理器最终下载什么版本以锁文件为准；package.json 只是起点。

## 1 安装与初始化

| 目的 | npm | pnpm |
| --- | --- | --- |
| 初始化项目 | `npm init -y` | `pnpm init` |
| 安装全部依赖 | `npm install` | `pnpm install` |
| 按锁文件严格安装 | `npm ci` | `pnpm install --frozen-lockfile` |

## 2 增删依赖

| 目的 | npm | pnpm |
| --- | --- | --- |
| 加正式依赖 | `npm i lodash` | `pnpm add lodash` |
| 加开发依赖 | `npm i -D vitest` | `pnpm add -D vitest` |
| 加指定版本 | `npm i lodash@4` | `pnpm add lodash@4` |
| 删依赖 | `npm un lodash` | `pnpm remove lodash` |
| 更新全部 | `npm update` | `pnpm update` |
| 更新单个 | `npm update lodash` | `pnpm update lodash` |

## 3 全局工具

| 目的 | npm | pnpm |
| --- | --- | --- |
| 全局安装 CLI | `npm i -g typescript` | `pnpm add -g typescript` |
| 查看全局包 | `npm ls -g` | `pnpm ls -g` |
| 一次性运行 | `npx tsx x.ts` | `pnpm dlx tsx x.ts` |

## 4 运行脚本

| 目的 | npm | pnpm |
| --- | --- | --- |
| 运行 scripts 中的命令 | `npm run dev` | `pnpm dev` |
| 构建脚本 | `npm run build` | `pnpm build` |

pnpm 可省略 run，直接写 `pnpm dev`、`pnpm build`。

## 5 锁文件与缓存

| 目的 | npm | pnpm |
| --- | --- | --- |
| 锁文件 | `package-lock.json` | `pnpm-lock.yaml` |
| 缓存目录 | `~/.npm` | `~/Library/pnpm/store/v11` |
| 清理缓存 | `npm cache clean --force` | `pnpm store prune` |

## 6 关键行为对比

| 维度 | npm | pnpm |
| --- | --- | --- |
| node_modules 结构 | 扁平 hoist | 严格结构加符号链接 |
| 未声明依赖可否 import | 可，形成幽灵依赖 | 不可，直接报错 |
| 多项目磁盘占用 | 各自完整副本 | 共享 store |
| 构建脚本 postinstall | 默认执行 | v10 起需 allowBuilds 授权 |

## 7 高频命令对应

- `npm i` 对应 `pnpm add`；
- `npm run dev` 对应 `pnpm dev`；
- `npx` 对应 `pnpm dlx`；
- `npm ci` 对应 `pnpm install --frozen-lockfile`。
