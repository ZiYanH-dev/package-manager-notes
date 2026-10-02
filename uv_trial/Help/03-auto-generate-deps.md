# 03 — 从代码反推依赖

## 核心概念速查

**pipreqs**
扫描项目内全部 .py 文件的 import 并反推 PyPI 包名的工具；只输出代码真正用到的顶层包。

**uvx pipreqs**
用 uvx 临时运行 pipreqs 的方式；无需预先安装该工具。

**uv add -r requirements.txt**
批量登记命令；读取整个文件写入 `[project].dependencies`，并自动更新 uv.lock。

**import 名与包名差异**
import 使用的名字与 PyPI 发行包名常不一致；例如 `cv2` 对应 `opencv-python`，`sklearn` 对应 `scikit-learn`，`PIL` 对应 `Pillow`。

## 核心结论

uv 本身没有扫描代码生成依赖的命令；配合 pipreqs 可以实现，流程为：
pipreqs 扫描 import 生成 requirements.txt，再执行 `uv add -r` 批量登记。

## uv 不内置该功能的原因

uv 的设计是声明式：人在 pyproject.toml 声明需要的包，uv 负责解析、锁版本、安装。
它不逆向扫描代码，原因有三：
- 代码中的 import 不一定都是项目依赖；可能是条件导入、测试残留或注释示例；
- import 名与包名常不一致，例如 `import cv2` 对应包 `opencv-python`；
- 扫描只能得到直接依赖的线索；间接依赖仍需解析器计算。

## 完整流程

在含 pyproject.toml 的项目根目录执行：

```bash
# 第 1 步：扫描所有 .py 的 import，反推包名，写入 requirements.txt
uvx pipreqs . --savepath requirements.txt --force

# 第 2 步：批量登记进 pyproject.toml，uv 自动更新 uv.lock
uv add -r requirements.txt
```

`uv add -r requirements.txt` 是 uv 官方支持的用法，读取整个文件并写入 `[project].dependencies`；从现成 requirements.txt 迁移时同样适用。

## pipreqs 的原理与注意事项

pipreqs 遍历项目内全部 .py 文件的 import 语句，把 import 名映射为 PyPI 发行包名后写入 requirements.txt。
它从源码树反推，与 `pip freeze` 方向相反；`pip freeze` 从已装环境导出，会带出全部间接依赖。

| 优点 | 注意事项 |
| --- | --- |
| 只列代码真正 import 的顶层包 | 包名偶有识别错误；`cv2`、`sklearn`、`PIL` 是常见例子 |
| 不带间接依赖，清单干净 | 默认不含版本约束；加 `--min` 可写最低版本 |
| 支持忽略指定目录 | 动态导入与 importlib 加载的包扫不到 |

第 1 步生成的 requirements.txt 应人工核对包名后，再执行 `uv add -r`。

### pipreqs 常用参数

```bash
uvx pipreqs . --savepath requirements.txt --force   # 覆盖已存在文件
uvx pipreqs . --min                                  # 附带最低版本号
uvx pipreqs . --ignore tests,.venv                   # 忽略指定目录
```

## 三种加依赖方式的对比

| 方式 | 命令 | 适用场景 |
| --- | --- | --- |
| 逐个添加 | `uv add fastapi` | 边写边加、依赖少、需精确控制版本 |
| 从文件批量 | `uv add -r requirements.txt` | 已有 requirements.txt 或 pipreqs 扫描结果 |
| 从代码反推 | pipreqs 加 `uv add -r` | 已有大量代码，一次性补齐依赖清单 |

不推荐 `pip freeze > requirements.txt` 再 `uv pip install -r` 的做法；freeze 导出环境中全部已装包，包含大量间接依赖，清单不可读也不应手动管理。

## 示例

```bash
cd /Users/jasonhuang/Desktop/trial/ai_trial/projects/study_helper/backend

# 先扫描预览，暂不登记
uvx pipreqs . --savepath /tmp/reqs.txt --force
cat /tmp/reqs.txt

# 核对包名后批量登记
uv add -r /tmp/reqs.txt
```

该项目的 22 个依赖当前已经齐全；这套流程适用于日后新增大量代码后，快速补齐依赖清单的场景。

## 注意事项

- pipreqs 的输出只是直接依赖的线索；间接依赖交给 uv 在 uv.lock 中解析，不要手动添加；
- 扫描结果中未实际使用的包不加，保持清单精简；
- requirements.txt 只是中间产物；权威清单是 pyproject.toml，登记完成后可删除该文件。
