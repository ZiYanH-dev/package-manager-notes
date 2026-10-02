# 01 — 基础：Maven 的工作模型

## 核心概念速查

**pom.xml**
Maven 的项目描述文件，位于项目根目录，声明坐标、打包方式、依赖与构建配置；Maven 的一切行为都以它为起点。

**GAV 坐标**
groupId、artifactId、version 三元组，唯一标识一个 artifact；本地仓库中的 jar 路径也按它组织。

**packaging**
pom.xml 中的打包方式字段，取值 jar、war 或 pom；父模块必须取 pom。

**SNAPSHOT**
以 `-SNAPSHOT` 结尾的特殊版本号，表示开发中的不稳定版本，每次构建都可能取到新快照；发布版版本固定，两者容易混淆。

**仓库体系**
Maven 存储 jar 的三层结构：本地 `~/.m2/repository/`、中央 Maven Central、公司私有仓库；查找顺序为本地、私有、中央。

**settings.xml**
Maven 的用户级配置文件，默认在 `~/.m2/settings.xml`，配置本地仓库位置、镜像源与服务器凭证；与 pom.xml 的项目级配置相区分。

**properties**
pom.xml 中的属性定义机制，用 `${属性名}` 在全文件引用；用于统一版本号等重复值。

## Maven 解决的问题

Java 项目需要三类基础设施：
- 管理第三方 jar 依赖；
- 定义统一的构建步骤，覆盖编译、测试、打包、部署；
- 统一项目结构，使任何开发者拿到项目即可构建。

Maven 同时承担包管理器与构建工具两个角色；npm 只做包管理，构建还需另行配置 webpack 或 vite。

## pom.xml 核心字段

```xml
<project>
  <modelVersion>4.0.0</modelVersion>  <!-- Maven 模型版本，固定值 -->

  <!-- 坐标：唯一标识一个项目 -->
  <groupId>com.example</groupId>       <!-- 组织域名倒写 -->
  <artifactId>my-app</artifactId>      <!-- 项目名 -->
  <version>1.0.0</version>             <!-- 版本号 -->

  <packaging>jar</packaging>           <!-- jar | war | pom -->

  <name>My App</name>
  <description>示例项目</description>
</project>
```

## GAV 坐标

Maven 用 GAV 三元组唯一定位一个 artifact：

```
groupId:artifactId:version
com.google.guava:guava:33.0.0-jre
```

jar 在仓库中的路径按坐标组织：

```
~/.m2/repository/com/google/guava/guava/33.0.0-jre/guava-33.0.0-jre.jar
```

### SNAPSHOT 版本

版本号以 `-SNAPSHOT` 结尾表示开发中版本，例如 `1.0.0-SNAPSHOT`；依赖方每次构建都可能拉到最新快照。
发布版本的版本号不含 SNAPSHOT，内容固定不变。
快照适合团队内部联调；正式发布一律使用固定版本。

## 标准目录结构

```
my-app/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/          # 源代码
    │   └── resources/     # 配置文件
    └── test/
        ├── java/          # 测试代码
        └── resources/     # 测试配置
```

## 仓库体系

| 仓库 | 位置 | 说明 |
| --- | --- | --- |
| 本地仓库 | `~/.m2/repository/` | 下载缓存的 jar |
| 中央仓库 | Maven Central | 官方公共仓库，默认源 |
| 私有仓库 | 公司内网 | 如 Nexus、Artifactory |

查找顺序为本地、私有、中央。

### settings.xml

`settings.xml` 是 Maven 的用户级配置，默认位于 `~/.m2/settings.xml`；pom.xml 管项目，settings.xml 管机器环境。
常用配置包括本地仓库路径 `localRepository`、镜像 `mirror` 与私有仓库凭证 `servers`：

```xml
<settings>
  <localRepository>/path/to/repo</localRepository>
  <mirrors>
    <mirror>
      <id>aliyun</id>
      <mirrorOf>central</mirrorOf>
      <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
  </mirrors>
</settings>
```

### properties 与占位符

properties 在 pom.xml 中统一定义值，全文件用 `${属性名}` 引用；多模块项目用它统一依赖版本：

```xml
<properties>
  <jackson.version>2.17.0</jackson.version>
</properties>

<dependencies>
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>${jackson.version}</version>
  </dependency>
</dependencies>
```
