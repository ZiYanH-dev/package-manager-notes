# Help — Maven 学习资料

本目录是 Maven 的学习与参考文档集合。

## 工作流速览

接手已有项目：
1. `git clone` 拉取代码；
2. `mvn clean install` 首次构建，自动下载依赖并写入本地仓库；
3. 仓库带 `mvnw` 时用 `./mvnw` 代替 `mvn`，保证构建版本一致。

日常开发循环：
1. 修改代码；
2. `mvn clean test` 跑测试；
3. 交付前 `mvn clean package` 打包。

新增依赖：
1. 在 pom.xml 的 `dependencies` 中声明，版本优先交给父 pom 的 dependencyManagement；
2. 下次构建自动下载；
3. 提交 pom.xml。

详细操作见 `04-daily-workflow.md`。

## 文档导航

| 文件 | 内容 |
| --- | --- |
| `01-basics.md` | Maven 的定位、pom.xml 核心字段、GAV 坐标、SNAPSHOT、仓库体系、settings.xml |
| `02-dependency-management.md` | 依赖声明、scope、传递依赖、排除、dependencyManagement 与 BOM |
| `03-build-lifecycle.md` | 生命周期阶段、插件与 goal、profile |
| `04-daily-workflow.md` | 常用命令、多模块项目、Maven Wrapper、常见问题 |

## 阅读顺序

1. 先读 `01-basics.md` 建立整体模型；
2. 再读 `02-dependency-management.md` 理解依赖机制；
3. 需要自定义构建流程时读 `03-build-lifecycle.md`；
4. 实际开发与排错时查询 `04-daily-workflow.md`。
