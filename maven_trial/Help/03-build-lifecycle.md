# 03 — 构建生命周期与插件

## 核心概念速查

**生命周期 lifecycle**
Maven 对构建过程的抽象，共三条独立链条：default 负责构建，clean 负责清理，site 负责站点文档。

**阶段 phase**
生命周期上的一个步骤；执行某阶段时它前面的阶段自动依次执行，例如 `mvn package` 会先执行 compile 与 test。

**插件 plugin 与 goal**
插件是完成实际工作的组件，goal 是插件内的一个具体任务；阶段绑定到插件的 goal 上执行，例如 `maven-surefire-plugin:test`。

**profile**
按环境切换配置的机制；用 `-P` 激活，可为 dev 与 prod 提供不同属性。

## 三条生命周期

| 生命周期 | 用途 | 主要阶段 |
|---|---|---|
| default | 项目构建 | validate → compile → test → package → verify → install → deploy |
| clean | 清理 | pre-clean → clean → post-clean |
| site | 站点文档 | pre-site → site → post-site → site-deploy |

## default 生命周期的核心阶段

```
validate     验证项目配置正确
compile      编译 src/main/java
test         运行 src/test/java 中的测试
package      打包为 jar 或 war
verify       检查产物是否满足质量要求
install      安装到本地仓库 ~/.m2/repository
deploy       上传到远程仓库，如 Nexus
```

执行后面的阶段时，前面的阶段自动执行：

```bash
mvn package        # 依次执行 validate → compile → test → package
mvn install        # 在 package 之上追加 install
```

## 插件与 goal

Maven 本身只是框架，实际工作由插件完成：

| 阶段 | 默认绑定的插件:goal |
|---|---|
| compile | `maven-compiler-plugin:compile` |
| test | `maven-surefire-plugin:test` |
| package | `maven-jar-plugin:jar` 或 `maven-war-plugin:war` |
| install | `maven-install-plugin:install` |

在 pom.xml 中配置插件：

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.11.0</version>
      <configuration>
        <source>21</source>
        <target>21</target>
      </configuration>
    </plugin>
  </plugins>
</build>
```

也可以绕过生命周期直接调用 goal：

```bash
mvn compiler:compile           # 只编译，不跑测试
mvn surefire:test              # 只跑测试
```

## profile 多环境配置

```xml
<profiles>
  <profile>
    <id>dev</id>
    <properties>
      <db.url>jdbc:mysql://localhost:3306/dev</db.url>
    </properties>
  </profile>
  <profile>
    <id>prod</id>
    <properties>
      <db.url>jdbc:mysql://prod:3306/app</db.url>
    </properties>
  </profile>
</profiles>
```

```bash
mvn package -P dev              # 激活 dev profile
mvn package -P dev,prod         # 同时激活多个
mvn help:active-profiles        # 查看当前激活的 profile
```
