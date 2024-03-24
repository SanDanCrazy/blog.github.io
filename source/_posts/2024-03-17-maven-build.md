---
title: Maven构建生命周期
date: 2024-03-17 10:25:29
tags: 
- Maven
- Java
- 后端
categories:
- [Java, 构建工具]
toc: true
---

## Maven是什么

Maven是一个用来构建和管理任何基于Java项目的工具，主要的目标是允许开发人员在最短的时间内理解开发过程的一个完整的周期。为了实现这个目标，Maven有以下几点考虑：
<!--more-->
+ 让构建流程更简单
+ 提供一个统一的构建系统
+ 提供项目质量信息
+ 鼓励更好的开发实践

### 让构建流程更简单

使用Maven不需要理解它的底层机制，它为开发者隐藏了很多细节

### 提供一个统一的构建系统

Maven使用POM和一系列插件来构建一个项目，一旦熟悉一个Maven项目，你就会知道所有Maven项目是如何构建的。

### 提供项目质量信息

Maven会从POM和项目源代码中获取有用的项目信息，三方代码扫描产品也提供了Maven插件用来生成标准报告用于Maven解析。

### 鼓励更好的开发实践

Maven旨在为开发最佳实践提供正确的原则，并且可以简单地指导项目进行最佳实践。
例如目前单元测试的最佳实践：

+ 将测试代码放到一个单独，并列的源码树
+ 使用测试用例命名规范去定位和执行测试
+ 让测试用例准备自己的环境而不是自定义测试构建的准备工作

## Maven构建生命周期详解

![Component of lifecycle](lifecycle.svg)

mvn命令行形式一般为：
```
mvn [phase] [plugin:goal]
```
其中phase一般从不同的lifecycle中取值，因此个数不会多于三个，plugin:goal可以为0个或者多个。phase和plugin:goal按照命令行从左往右依次执行。下面详细讲解一下各个名词的概念。

### Lifecycle
maven只有三个内置的lifecycle：default（项目部署）, clean（项目清理，需要注意的是这里的clean和phase里的clean不是一个概念上的东西，lifecycle无法作为mvn命令的参数，因此mvn clean指的是执行clean lifecycle里的clean phase） and site（项目网站的创建，不常用）。每个lifecycle由多个phase组成，它们串行执行以完成lifecycle里的任务。执行其中一个phase时都会按照顺序从它所属的lifecycle里的第一个phase开始执行到该phase。

### Phase
构建流程里的组织单元，它们在lifecycle中的执行顺序是固定的，因此执行计划是可预知的。一个phase会绑定0个或者多个goal，当该phase执行时，所绑定的goal会全部执行（多个goal会按照pom文件中声明的顺序依次执行），并且该phase所在的lifecycle中之前的phase也会全部执行。

### Plugins
负责提供goal用来绑定到phase上执行或者单独执行plugin:goal。

### Goal
构建流程里的执行单元，它可以单独执行，也可以绑定到phase上执行。

>[Goal和Phase的区别](https://stackoverflow.com/questions/16205778/what-are-maven-goals-and-phases-and-what-is-their-difference)

注：在常用编辑器IDEA中，点开右边的Maven图标可以看到项目里的Lifecycle为下图所示内容，其实这里包含的全部都是phase。

![IDEA Maven插件](idea.jpg)

## 常见场景

### 1.mvn clean package

该命令的作用是清理target目录下的文件，在机器上构建打包，生成可执行的jar包或者war包。该命令的构建结果文件通常位于和src同级的target目录下。`clean`的作用是清理之前的构建文件，以免影响本次构建结果。`package`的作用是执行校验，编译，测试，打包。

该命令常用于在流水线里执行构建计划，并将打包出来的文件上传到指定的制品库中。

### 2.mvn clean install

该命令比上一条`mvn clean package`命令多执行了两个phase:`verify`和`install`。`verify`用于额外的验证检查，通常是集成测试或插件指定的测试，`install`用于将打包结果安装到本地的maven仓库中。

该命令常用于构建多个项目且项目间存在依赖项，例如A项目依赖B项目的alpha版本，但是B项目目前是beta版本，需要B项目进行一次版本升级到alpha后A项目才能获取到正确的依赖。因此对B版本运行`mvn clean install`后将打包结果安装到本地，A项目就能正确解析到所需B项目的版本。

### 3.mvn clean deploy

该命令比`mvn clean install`命令多执行了`deploy` phase，该命令的作用是将最终制品文件上传到maven仓库以发布上线或者共享文件。该命令通常需要settings.xml文件（maven的配置文件）的配合，例如在settings.xml中配置了私有仓库，并且针对snapshot和release类型的包分别配置了仓库链接，那么在`deploy`时，就会根据包版本结尾是否包含SNAPSHOT来判断上传到哪个仓库。

`deploy`具体依赖哪些配置项可以阅读[settings.xml文件格式解析](https://maven.apache.org/settings.html)

## 结尾

mvn命令除了主要的phase和goal，还包含很多用于控制系统环境的参数，例如 `-P` 用来激活某个`profile`，`-D` 用来指定java全局属性，`-ff` 遇到构建失败直接退出。