# 04 — 日常维护与 Brewfile

## 核心概念速查

**Brewfile**
环境声明文件，列出 Tap、formula 与 cask 的完整清单；配合 `brew bundle` 在新机器上一键还原整套环境。

**brew bundle**
按 Brewfile 执行安装的命令；`dump` 从已安装环境反向生成 Brewfile，`check` 校验差异，`cleanup --force` 卸载清单之外的包。

**brew bundle dump**
从当前已安装环境生成 Brewfile 的命令；`--file` 指定输出路径，是环境迁移的起点。

## 日常维护工作流

```bash
# 每周执行一次
brew update && brew upgrade && brew cleanup

# 定期执行
brew doctor
brew missing
```

### 演示记录

以下命令实跑，输出经截取。

```bash
brew outdated
```

```text
ada-url
c-ares
ca-certificates
cairo
certifi
cffi
```

```bash
brew leaves
```

```text
cloudflared
ffmpeg
fnm
gcc
glances
htop
```

```bash
brew deps --tree git
```

```text
git
├── pcre2
└── gettext
    ├── json-c
    └── libunistring
```

```bash
brew cleanup --dry-run
```

```text
Would remove: /opt/homebrew/Library/Homebrew/vendor/portable-ruby/4.0.6 (1,707 files, 34.6MB)
==> This operation would free approximately 158.5MB of disk space.
```

```bash
brew doctor
```

```text
Warning: The current git origin is:
  https://mirrors.ustc.edu.cn/brew.git
```

doctor 报的这条警告对应本机配置的中科大镜像源，属提示性信息，不影响使用。

## Brewfile

Brewfile 把整台机器的软件需求写成一个声明文件，作用类似 Node 的 `package.json`：

```ruby
# ~/Brewfile
tap "homebrew/cask"

brew "git"
brew "node"
brew "python@3.13"
brew "mysql"
brew "redis"

cask "google-chrome"
cask "visual-studio-code"
cask "iterm2"
cask "font-fira-code"
```

`homebrew/cask-fonts` Tap 已被官方并入主 cask 仓库；字体类 cask 直接写 `cask "font-fira-code"` 即可，无需单独添加该 Tap。

```bash
brew bundle                     # 按 Brewfile 安装全部内容
brew bundle dump                # 根据已安装环境生成 Brewfile
brew bundle dump --force        # 覆盖已有 Brewfile
brew bundle check               # 检查 Brewfile 中的项是否都已安装
brew bundle cleanup --force     # 卸载 Brewfile 之外已安装的包
```

## 环境迁移

```bash
# 旧机器导出
brew bundle dump --file=~/Brewfile

# 新机器恢复
brew bundle --file=~/Brewfile
```

换新电脑时拷贝 Brewfile 后执行 `brew bundle` 即可还原环境。

## 常见问题与注意事项

- 不要使用 `sudo brew`；Homebrew 设计为非 root 运行，sudo 反而引发权限问题；
- `brew upgrade` 可能影响依赖当前版本的项目；升级前用 `brew list --versions` 确认版本变化；
- cask 应用不会随普通升级自动更新；批量更新 GUI 应用要用 `brew upgrade --cask --greedy`；
- `brew services` 只对 formula 有效；cask 不支持服务管理。
