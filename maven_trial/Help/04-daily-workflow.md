# 04 — 日常命令与多模块项目

## 核心概念速查

**mvn clean package 与 mvn clean install**
最常用的组合命令；install 比 package 多一步写入本地仓库，跨模块依赖必须用 install 才能被兄弟模块引用。

**-pl、-am、-amd**
多模块构建参数；`-pl` 指定模块，`-am` 追加其依赖的模块，`-amd` 追加依赖它的模块。

**mvn help:effective-pom**
输出继承与 profile 合并后最终生效的 pom；排查继承配置问题的首选命令。

**Maven Wrapper**
随项目分发的 mvnw 脚本，为项目锁定 Maven 版本；协作者无需预装 Maven，执行 `./mvnw` 即可用指定版本构建。

**-DskipTests 与 -Dmaven.test.skip=true**
两个跳过测试的参数；前者跳过执行但保留编译，后者连编译一起跳过，速度更快。

## 日常工作流

接手已有项目：
1. `git clone` 拉取代码；仓库带 `mvnw` 时直接用 `./mvnw`，未带时可执行 `mvn wrapper:wrapper` 生成；
2. `mvn clean install` 完成首次构建；它下载全部依赖、写入本地仓库，多模块项目同时把各模块装进本地仓库；
3. 多模块项目在根目录执行构建；改了被依赖的模块后先 install，兄弟模块才能拿到新版本。

日常开发循环：
1. 修改代码；
2. `mvn clean test` 验证；
3. 交付前 `mvn clean package` 打包。

新增依赖：
1. 在 pom.xml 的 `dependencies` 中声明；
2. 版本优先由父 pom 的 dependencyManagement 或 BOM 统一，子模块不写 version；
3. 下次构建时 Maven 自动下载；
4. 提交 pom.xml；target/ 不提交。

发布到远程仓库：
1. 在 settings.xml 的 `servers` 中配置凭证；
2. 执行 `mvn clean deploy`。

### 演示记录

演示项目为最小 pom，含一个类与一个 JUnit 测试，在临时目录实跑。
本机 settings.xml 配置的阿里云镜像在下载中生效，对应 01 的 settings.xml 示例。
以下输出经截取。

```bash
mvn clean package
```

```text
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] Building jar: /private/tmp/maven_demo/target/maven-demo-1.0.0.jar
[INFO] BUILD SUCCESS
[INFO] Total time:  17.549 s
```

```bash
mvn clean install
ls ~/.m2/repository/com/example/maven-demo/1.0.0/
```

```text
maven-demo-1.0.0.jar
maven-demo-1.0.0.pom
```

install 之后 jar 与 pom 进入本地仓库；兄弟模块从此可以引用这个坐标。

```bash
mvn dependency:tree
```

```text
com.example:maven-demo:jar:1.0.0
\- org.junit.jupiter:junit-jupiter:jar:5.10.2:test
   +- org.junit.jupiter:junit-jupiter-api:jar:5.10.2:test
   |  +- org.opentest4j:opentest4j:jar:1.3.0:test
   |  +- org.junit.platform:junit-platform-commons:jar:1.10.2:test
   |  \- org.apiguardian:apiguardian-api:jar:1.1.2:test
   +- org.junit.jupiter:junit-jupiter-params:jar:5.10.2:test
   \- org.junit.jupiter:junit-jupiter-engine:jar:5.10.2:test
      \- org.junit.platform:junit-platform-engine:jar:1.10.2:test
```

junit-jupiter 之下的整棵子树都是间接依赖；pom 里只声明了 junit-jupiter 一个包。

## 常用命令

```bash
mvn clean                        # 删除 target/ 目录
mvn clean compile                # 清理 + 编译
mvn clean test                   # 清理 + 跑测试
mvn clean package                # 清理 + 打包
mvn clean install                # 清理 + 安装到本地仓库
mvn clean deploy                 # 清理 + 发布到远程仓库

mvn dependency:tree              # 查看依赖树
mvn help:effective-pom           # 查看最终生效的 pom，含继承
mvn dependency:resolve           # 下载依赖
mvn dependency:purge-local-repository  # 清理本地缓存并重新下载
```

## 多模块项目

父 pom 声明模块列表，packaging 必须为 pom：

```xml
<groupId>com.example</groupId>
<artifactId>my-project</artifactId>
<version>1.0.0</version>
<packaging>pom</packaging>

<modules>
  <module>module-a</module>
  <module>module-b</module>
</modules>
```

```bash
mvn clean install                      # 按依赖顺序构建所有模块
mvn -pl module-a clean install         # 只构建 module-a
mvn -pl module-a -am clean install     # module-a 及其依赖的模块
mvn -pl module-a -amd clean install    # module-a 及依赖它的模块
```

## Maven Wrapper

Wrapper 把 Maven 版本随仓库分发，保证所有协作者使用同一版本构建：

```bash
mvn wrapper:wrapper                    # 生成 mvnw 脚本与 wrapper 目录
./mvnw clean install                   # 用项目锁定的 Maven 版本构建
```

生成后仓库内出现 `mvnw`、`mvnw.cmd` 与 `.mvn/wrapper/`；协作者克隆后直接执行 `./mvnw`，无需本机安装 Maven。

## 构建提速参数

```bash
mvn -T 4 clean install              # 4 线程并行构建
mvn -o clean install                # 离线模式，不联网下载
mvn -q clean install                # 静默模式，只输出 ERROR
mvn -DskipTests clean install       # 跳过测试执行
mvn -Dmaven.test.skip=true clean install  # 跳过测试编译与执行
```

## 本地仓库管理

```bash
mvn help:effective-settings | grep localRepository   # 查看本地仓库位置
rm -rf ~/.m2/repository                              # 整体清理，慎用，会全部重新下载
rm -rf ~/.m2/repository/com/example/                 # 只清理某组织的缓存
```

## 常见问题与注意事项

- `mvn clean install` 比 `mvn clean package` 慢，因为 install 多一步写入本地仓库；但跨模块依赖必须用 install；
- 多模块项目的依赖版本不一致时，用父 pom 的 `dependencyManagement` 统一管理，不要在每个子模块写 version；
- `-DskipTests` 跳过执行但保留编译，`-Dmaven.test.skip=true` 连编译一起跳过，后者更快；
- 排查继承与 profile 问题时用 `mvn help:effective-pom`，父 pom 带来的配置合并结果完整可见。
