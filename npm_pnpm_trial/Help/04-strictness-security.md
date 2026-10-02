# 04 — 严格性与安全

## 核心概念速查

**幽灵依赖 phantom dependency**
未在 package.json 声明、却因 npm 扁平化结构而能 import 的包；上游升级断开依赖后代码会失效，pnpm 严格模式下直接报错。

**严格 node_modules**
pnpm 的默认结构：顶层只暴露显式声明的包，子依赖收纳在 `.pnpm/` 内部；未声明的 import 直接报 `Cannot find module`。

**shamefully-hoist**
pnpm 的兼容开关，模仿 npm 扁平结构把所有包提升到顶层；仅用于兼容老项目，开启后失去严格性。

**postinstall**
包自带的安装后脚本，常用于编译原生二进制；npm 默认执行，pnpm v10 起默认拦截。

**allowBuilds**
pnpm v10 的构建脚本授权配置，写在 `pnpm-workspace.yaml`；逐包声明是否允许执行安装脚本。

## pnpm 的严格 node_modules

npm 扁平化后，未声明的包也会出现在 node_modules 顶层，因此可以被 import。
例如只安装了 `a`，但 `a` 依赖 `b`，扁平结构下 `b` 也能被直接 import，这就是幽灵依赖；上游升级不再依赖 `b` 时，代码会失效。

pnpm 默认严格：node_modules 顶层只暴露显式声明的包；未声明的 import 直接报 `Cannot find module`，在安装期就暴露问题。

```ts
// package.json 只声明了 a，未声明 b
import b from 'b'   // npm 扁平下可能能跑；pnpm 下直接报错
```

严格结构使依赖关系诚实、项目可移植，避免本机能运行而他人环境失败。

## 老项目的兼容措施

严格模式导致老项目报错时，优先补齐缺失的显式依赖：

```bash
pnpm add <缺失的包>
```

仍不兼容时再用降级开关；`.npmrc` 中写入：

```ini
shamefully-hoist=true
```

该开关模仿 npm 扁平结构，允许 import 未声明的包；仅作兼容手段，常态不开启。

## pnpm v10 构建脚本授权

许多包带有 postinstall 脚本，在安装后执行编译原生二进制等操作。
pnpm v10 起默认拦截这类脚本，必须显式批准；否则干净机器上安装后功能可能异常。

授权写在 `pnpm-workspace.yaml`，单包项目同样使用该文件：

```yaml
allowBuilds:
  esbuild: true     # tsx 的底层引擎，需允许其安装脚本
```

真实踩坑：`web_dev_trial/typescript` 中 pnpm 自动生成了 `pnpm-workspace.yaml`，但 esbuild 一行的值是占位文本，未生效；需手动改为 `true`。

检查与补救命令：

```bash
pnpm install          # 被拦截的包会给出提示
pnpm rebuild esbuild  # 手动补跑某个包的构建脚本
```

## 安全能力对比

| 能力 | npm | pnpm |
| --- | --- | --- |
| 漏洞扫描 | `npm audit` | `pnpm audit` |
| 锁文件完整性 | 内置 integrity 哈希 | 内置 integrity 哈希 |
| 构建脚本默认执行 | 是 | v10 起默认拦截，需 allowBuilds 授权 |

pnpm 的严格结构与脚本拦截是更安全的默认值，代价是多一行 allowBuilds 配置。
