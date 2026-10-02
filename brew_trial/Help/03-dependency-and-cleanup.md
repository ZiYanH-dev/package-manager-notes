# 03 — 依赖、清理与诊断

## 核心概念速查

**brew deps**
查询依赖的命令；`--tree` 以树形展示，`--installed` 限定已安装范围。

**brew uses**
反向查询命令，列出哪些已安装的包依赖指定包；删除前用它评估影响。

**brew cleanup**
清理命令，默认删除旧版本缓存；`--dry-run` 预览将删除的内容，`--prune=N` 按天数清理缓存。

**brew doctor**
环境自检命令，输出 Homebrew 检测到的问题与修复建议。

**brew missing**
检查已安装包缺失依赖的命令。

## 依赖查询

```bash
brew deps git                    # git 的直接依赖
brew deps --tree git             # git 的依赖树
brew uses --installed git        # 哪些已安装的包依赖 git
```

## 清理

```bash
brew cleanup                     # 删除旧版本，保留当前在用版本
brew cleanup --dry-run           # 预览将删除的内容
brew cleanup git                 # 只清理 git 的旧版本
brew cleanup --prune=7           # 清理 7 天前的下载缓存
```

## 磁盘占用

```bash
du -sh /opt/homebrew/Cellar/*     # Cellar 中每个包的占用
du -sh /opt/homebrew/Caskroom/*   # Caskroom 中每个应用的占用
du -sh ~/Library/Caches/Homebrew  # 下载缓存目录大小
```

Cellar 与 Caskroom 的大小即包的实际占用；缓存目录可通过 `brew cleanup --prune` 回收。

## 诊断

```bash
brew doctor                      # 检查 Homebrew 环境问题
brew missing                     # 检查缺失的依赖
brew config                      # 查看 Homebrew 配置信息
```

## 常见问题

- `brew doctor` 的 Warning 多为提示性信息，按输出建议修复即可；
- 权限问题的根源是目录归属，保持 `/opt/homebrew/` 属于当前用户，不要使用 `sudo brew`；
- formula 找不到时先确认是否在第三方 Tap 中，用 `brew search` 全局检索；
- `brew update` 缓慢时可切换国内镜像源，例如中科大源或清华源。
