# 02 — 依赖管理

## 核心概念速查

**scope**
依赖的作用范围字段，控制该依赖在编译、测试、运行、打包哪个阶段可见；默认 compile。

**传递依赖**
依赖的依赖；A 依赖 B、B 依赖 C 时，A 自动获得 C，无需显式声明。

**exclusions**
依赖声明下的排除配置，用于阻断某个传递依赖进入本项目。

**dependencyManagement**
版本锁定区域，只声明版本而不真正引入依赖；实际引入仍在 `dependencies` 中完成，且可省略 version。

**BOM**
Bill of Materials，一种只含版本清单的特殊 pom；通过 `scope=import` 引入后整组依赖版本统一管理。

**mvn dependency:tree**
查看依赖树的命令；`-Dincludes` 可按坐标过滤，是排查版本冲突的首选工具。

## 声明依赖

```xml
<dependencies>
  <dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>33.0.0-jre</version>
    <scope>compile</scope>              <!-- 默认 compile，可省略 -->
  </dependency>
</dependencies>
```

## scope 依赖范围

| scope | 编译 | 测试 | 运行 | 打包 | 示例 |
|---|---|---|---|---|---|
| `compile` | 是 | 是 | 是 | 是 | 大部分库，如 Guava、Jackson |
| `provided` | 是 | 是 | 否 | 否 | Servlet API，由容器提供 |
| `runtime` | 否 | 是 | 是 | 是 | JDBC 驱动 |
| `test` | 否 | 是 | 否 | 否 | JUnit、Mockito |
| `system` | 是 | 是 | 否 | 否 | 本机 jar，不推荐 |
| `import` | — | — | — | — | 仅用于 dependencyManagement |

## 传递依赖与冲突解决

A 依赖 B、B 依赖 C 时，A 自动获得 C。
冲突按两条规则裁决：
1. 最短路径优先：`A → B → C:1.0` 与 `A → D → E → C:2.0` 并存时取 C:1.0；
2. 路径等长时，最先声明者生效。

```bash
mvn dependency:tree                          # 查看依赖树
mvn dependency:tree -Dincludes=com.google    # 过滤只看某个包
```

## 排除传递依赖

```xml
<dependency>
  <groupId>com.example</groupId>
  <artifactId>lib-a</artifactId>
  <version>1.0</version>
  <exclusions>
    <exclusion>
      <groupId>com.example</groupId>
      <artifactId>lib-bad</artifactId>  <!-- 阻断 lib-a 带入的某个传递依赖 -->
    </exclusion>
  </exclusions>
</dependency>
```

## dependencyManagement 与 BOM

`dependencyManagement` 只统一版本，不真正引入依赖；真正的引入在 `dependencies` 中完成，且可省略 version：

```xml
<!-- 父 pom 或独立 BOM 中定义 -->
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>3.2.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

多模块项目应在父 pom 的 dependencyManagement 中统一版本，子模块不写 version；这也是解决版本号冲突的标准做法。
