# 02 — 依赖类型、版本约束与锁文件

## 核心概念速查

**直接依赖 direct dependency**
在 package.json 中显式声明的包；由 `pnpm add` 或 `npm i` 写入。

**间接依赖 transitive dependency**
直接依赖自身引入的包；不显式声明但必须存在，版本由包管理器解析。

**peerDependencies**
包作者声明的宿主依赖，表示本包要求使用方提供某个包，典型如插件要求宿主装好框架本体；与 dependencies 的自带安装相区分，应用项目一般不写。

**语义化版本约束 semver**
`^` 允许次版本与补丁升级，`~` 只允许补丁升级，精确写法锁死版本；`^` 对 0.x 版本只允许补丁级变化。

**锁文件 lock file**
记录本次安装解析出的每包精确版本与完整性哈希；npm 生成 `package-lock.json`，pnpm 生成 `pnpm-lock.yaml`，必须提交 git。

**npm ci 与 --frozen-lockfile**
按锁文件严格安装的命令与参数；锁文件与 package.json 不一致时直接报错，CI 必用。

**overrides**
覆盖传递依赖版本的配置；npm 写在 package.json 的 overrides 字段，pnpm 写在 pnpm-workspace.yaml 的 overrides 键。

## 依赖的两个层级

- 直接依赖是 `pnpm add` 显式安装、写在 package.json 里的包；
- 间接依赖是所装包自身依赖的包；不直接使用但必须存在。

包管理器的核心问题，是把整棵依赖树装对、装快、装一致。

### 依赖的声明字段

- `dependencies` 存放正式依赖；
- `devDependencies` 存放开发依赖；
- `peerDependencies` 由包作者声明，表示本包要求使用方提供某个包，典型场景是插件声明需要宿主提供框架本体；pnpm 的 `auto-install-peers=true` 会自动安装缺失的 peer，应用项目通常不写 peerDependencies。

## 版本约束写法

```jsonc
"lodash": "^4.17.21"   // ^ : >=4.17.21 <5.0.0，次版本与补丁自动升
"react":  "~18.3.0"    // ~ : >=18.3.0 <18.4.0，只升补丁
"vue":    "3.4.0"      // 精确版本，锁死
"semver": "^0.17.2"    // ^ 对 0.x 特殊：等价 >=0.17.2 <0.18.0
```

`^` 是 `npm i` 与 `pnpm add` 的默认写法；0.x 阶段的次版本按破坏性变更处理，因此 `^0.17.2` 只允许补丁升级；出问题时改用精确版本或 `~` 收窄范围。

## 锁文件的作用

package.json 只声明范围，例如 `^4.17.21`；今天安装得到 4.17.21，下月可能变成 4.18.0。
锁文件把实际解析出的精确版本钉死，保证任何机器、任何时间安装结果一致。

| | npm | pnpm |
| --- | --- | --- |
| 锁文件 | `package-lock.json` | `pnpm-lock.yaml` |
| 格式 | JSON | YAML |
| 提交 git | 必须 | 必须 |
| 严格安装 | `npm ci` | `pnpm install --frozen-lockfile` |

`npm ci` 与 `--frozen-lockfile` 严格按锁文件安装；package.json 与锁文件不一致时直接报错，CI 环境必用。

## npm 与 pnpm 加依赖对照

```bash
npm i lodash            # 写入 dependencies
npm i -D vitest         # 写入 devDependencies
pnpm add lodash         # 写入 dependencies
pnpm add -D vitest      # 写入 devDependencies
```

## overrides 覆盖传递依赖版本

传递依赖存在漏洞或版本冲突、而上游未修复时，用 overrides 强制指定其版本：

```jsonc
// npm：package.json
{
  "overrides": {
    "semver": "^7.5.4"
  }
}
```

```yaml
# pnpm：pnpm-workspace.yaml
overrides:
  semver: ^7.5.4
```

覆盖后重新 install，并提交更新后的锁文件。

## 两套锁文件并存的问题

同一目录先跑 `npm install` 再跑 `pnpm install`，会同时生成 `package-lock.json` 与 `pnpm-lock.yaml`；此后无法判断安装以哪个为准。
规则是一个项目只用一个包管理器；切换到 pnpm 后删除 npm 的锁文件：

```bash
rm package-lock.json
pnpm import     # 可选：读取旧 npm 锁文件转换，避免重新解析
pnpm install
```

迁移细节见 `05-migration-daily.md`。
