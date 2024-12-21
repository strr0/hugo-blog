---
title: "Maven 打包配置"
date: 2024-10-17T10:00:00+08:00
categories: ["operations"]
tags: ["java"]
draft: false
---

## 介绍

Maven 打包

## 1 使用

### 1.1 常规方式（代码依赖不分离）

插件如下
|            插件             |       版本        |          备注           |
| :-------------------------: | :---------------: | :---------------------: |
|  spring-boot-maven-plugin   |      3.1.7        |       项目代码打包       |

#### 1.1.1 添加打包插件

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>${spring-boot.version}</version>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

#### 1.1.2 运行打包命令

运行打包命令，插件会自动识别启动类，并打包成一体包，直接启动即可
```bash
mvn clean package -DskipTests
```

### 1.2 其他方式（代码依赖分离）

插件如下
|            插件             |       版本        |                  备注                 |
| :-------------------------: | :---------------: | :-----------------------------------: |
|  maven-dependency-plugin    |      3.6.1        |             依赖输出                   |
|  maven-jar-plugin           |      3.3.0        |  项目代码打包（可配置包含及忽略文件）  |
|  maven-assembly-plugin      |      3.6.0        |  其他文件打包（如依赖及配置文件等）   |

#### 1.2.1 依赖输出

插件配置

- outputDirectory - 输出路径
- excludeTransitive - 排除传递依赖，一般情况设置为 false
- stripVersion - 去掉版本号
- excludeArtifactIds - 排除指定的依赖
- includeScope - 包含指定 scope

添加插件及设置如下配置
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>${maven-dependency-plugin.version}</version>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>${project.build.directory}/lib</outputDirectory>
                <excludeTransitive>false</excludeTransitive>
                <stripVersion>false</stripVersion>
                <includeScope>runtime</includeScope>
            </configuration>
        </execution>
    </executions>
</plugin>
```

#### 1.2.2 项目代码打包（不含依赖）

插件配置

- mainClass - 启动类路径
- addClasspath - 设置 classpath
- classpathPrefix - classpath 前缀
- excludes - 忽略文件

添加插件及设置如下配置
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>${maven-jar-plugin.version}</version>
    <configuration>
        <archive>
            <manifest>
                <mainClass>com.example.demo.DemoApplication</mainClass>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
            </manifest>
        </archive>
        <excludes>
            <exclude>**/*.yml</exclude>
            <exclude>**/*.properties</exclude>
            <exclude>**/*sh</exclude>
            <exclude>**/bin</exclude>
            <exclude>**/config</exclude>
        </excludes>
    </configuration>
</plugin>
```

#### 1.2.3 其他文件打包

插件配置

- descriptor - 指定 assembly 配置文件路径

添加插件及设置如下配置
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>${maven-assembly-plugin.version}</version>
    <configuration>
        <appendAssemblyId>false</appendAssemblyId>
        <outputDirectory>target/</outputDirectory>
        <descriptors>
            <descriptor>zip-desc.xml</descriptor>
        </descriptors>
    </configuration>
    <executions>
        <execution>
            <id>package-zip</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

assembly 文件配置
```xml
<assembly>
    <id>zip-desc</id>
    <!-- 打包类型，这里设置zip，则最终输出一个zip包 -->
    <formats>
        <format>zip</format>
    </formats>
    <!-- 设置项目外目录的处理方式，一条规则一个fileSet -->
    <fileSets>
        <fileSet>
            <directory>src/main/resources</directory>
            <outputDirectory>/config</outputDirectory>
            <excludes>
                <exclude>src/main/resources/mapper</exclude>
                <exclude>src/main/resources/META-INF</exclude>
            </excludes>
        </fileSet>

        <fileSet>
            <directory>target/lib</directory>
            <outputDirectory>/lib</outputDirectory>
        </fileSet>

        <fileSet>
            <directory>target</directory>
            <outputDirectory>/</outputDirectory>
            <includes>
                <include>*.jar</include>
            </includes>
        </fileSet>
    </fileSets>
</assembly>
```

#### 1.1.2 运行打包命令

运行打包命令，插件会生成一个代码和依赖分离的 zip 包
```bash
mvn clean package -DskipTests
```

## 2 其他

使用代码依赖分离的打包方式可以实现代码的增量更新，如更新了某个模块，只需替换对应 jar 包即可。