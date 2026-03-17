---
cover:
title: cursor2.1的使用指南（重点概要）
date: 2025-11-24
categories:
  - 杂
tags:
  - 笔记
---

# cursor2.1的使用指南（重点概要）

目前cursor的最新版本是2.1，相比于前面的几个版本改动基本不大。

这篇博客基于我自己的学习和使用体验，罗列出cursor使用的重点概要。

## cursor的基本配置

cursor的操作页面和vscode基本差不多，如果有vscode 的使用经验的话，上手cursor还是很快的。
![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/9eeead138190481caed0397a9efa7142.png)

我们可以直接将vscode中的配置直接移接到cursor中，直接在设置中找到import Setting from VS Code，如果你电脑上有vscode的话，一键导入甚是方便。

一开始cursor是全英文的，如果需要汉化，就需要和vscode一样去添加插件。

## java开发环境设置

如果你希望在cursor生成的java代码可以执行和调试，那么就需要去配置一下cursor的java和meven环境(如果你有JAVA_HOME的环境变量和MAVEN_HOME的环境变量就不需要配，cursor会自动配置)，此外还需要下载一个插件集合**Extension Pack for Java**，

其作用如下：

- **Language Support for Java(TM) by Red Hat**：提供语法高亮、智能代码补全、代码检查、代码格式化（`Shift + Alt + F`（Windows/Linux）或 `Shift + Option + F`（Mac））、代码导航以及重构支持等功能，辅助高效编写和优化 Java 代码。

- **Debugger for Java**：实现轻量级 Java 程序调试，可设置断点，调试时查看变量值、对象属性和调用栈，追踪程序执行流程以排查问题。

- **Maven for Java**：用于管理 Maven 项目，能创建新项目，管理项目依赖，执行 Maven 构建任务，如清理、编译、打包项目等。

- **Test Runner for Java**：支持 JUnit 和 TestNG 等测试框架，方便运行和调试 Java 测试用例，展示测试结果及详细日志，助力开发者定位问题。

- **Project Manager for Java**：可在编辑器中管理多个 Java 项目，实现快速切换，导入本地 Java 项目，可视化展示项目模块、包和文件结构。

- **Gradle for Java**：针对 Gradle 构建工具，能创建 Gradle 项目，运行 Gradle 任务，管理项目构建、测试流程，查看 Gradle 任务和工程依赖 。

步骤：

1. ctrl + shift + P 打开搜索框，搜索：settings.json

2. 添加

   ```json
   "java.home": "JDK的文件位置",
     "java.configuration.maven.userSettings": "Maven的文件位置",
   ```

   到settings.json中

3. ctrl + shift + X 搜索插件：Extension Pack for Java，下载安装即可。

示意图：

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/504c29334da5408caebcbabfd068d2b4.png)

## cursor的tab键

和其他智能的编译器（如IDEA）提供的AI功能是一样的，tab提供代码的自动补全

但是更高级的是，cursor里面的tab可以根据你写的注释自动联想补全代码，我认为这个是最实用的。

例如：

```java
public class ArrayUtils {
	//编写一段冒泡的数组排序的函数

    ....//cursor可以根据的你注解为你生成你想要的代码
}
```

## cursor的AI对话功能

cursos的对话功能有三种模式：

- Agent:用于处理复杂编码任务的默认模式。Agent 会自动探索你的代码库、编辑多个文件、运行命令并修复错误，以完成你的请求。
- Ask:用于学习和探索的只读模式。Ask 会搜索你的代码库并提供答案，而不会进行任何更改——非常适合在修改前先理解代码。
- Plan:提前规划，并通过结构化待办列表与消息队列管理复杂任务，让长周期任务更易理解和追踪。

### Agent模式：

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/7099788923674270af63397b7bdcfb5f.png)

接下来就会在你的项目文件夹下生成你的java代码，你可以选择保留修改，和可以不保留修改。
![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/26663472c61940a7a71fa635d4c653a9.png)

### Ask模式：

在Ask模式下给它相同的提示词，这种模式不会修改项目的结构，也不会添加文件，只是给你相应的代码，

相当于只是给你建议。

### Plan模式：

这种模式在你给它提示词之后，它会给你罗列出一系列实现的步骤，如果有较细节的内容它会向你询问实现的细节，保证项目正生成的时候尽可能不出错。

## cursor的上下文

### cursor的.cursorignore文件

cursor之所以可以根据你的提示在项目中修改代码，其中的原因是因为cursor的上下问功能，通过为文件夹和目录创建索引，从而掌握项目的结构。

cursor可以通过创建.cursorignore文件指定忽略那些文件不创建索引，文件语法和git的忽略文件差不多。

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/3b9e8427f8e24cf18f47beadb8660f47.png)

### cursor的rules

Rules是给Cursor AI功能（规则适用于 Chat和 Cmd K）生成结果添加规则和限制，让 AI 生成的代码贴合团队规范，减少人工二次修改成本，主要的作用如下：

- 可约束代码风格（如强制用驼峰命名、要求函数必须写注释 ）
- 能限定技术选型（如禁止使用某老旧库、优先用项目指定工具类 ）
- 提前指定核心参数（如提前设置连接数据库的地址和账号密码等）

### cursor的@符号

在 Cursor 中使用 @ 符号在聊天中引用代码、文件、文档和其他上下文的指南，直接更具体的指定上下文环境！

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/f4f78b2ec8db448f8f4af6a9e9311572.png)

以下是所有可用 @ 符号的列表：

- @文件和文件夹，选择要搜索的文件名即可引用整个文件。你也可以将侧边栏中的文件直接拖拽到 Agent 中作为上下文。

- @Code - 引用整个文件夹以获得更广泛的上下文

- @Code - 使用 `@Code` 符号引用特定代码段@Docs- 访问文档和指南

- @Docs- `@Docs` 功能可让你用文档来辅助写代码，我们可以导入自己的文档也可以导入框架的文档

  ![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/ebedb2dc029941c79ea822196e5dad6c.png)

例如导入mybatis-plus的官方文档步骤：

1. 复制关官方文档地址
   ![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/c7ba072e1a814b0dbdac9ccaec72b386.png)

2. 添加到docs中

   ![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/7724a37b3a064b16a02b893cb30a8595.png)

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/fe3ea71b690e421d8ff5e4d0fc274baa.png)

这个导入文档功能也是十分的实用，遇到不会的直接一个@Docs问AI，不用去翻文档了。
