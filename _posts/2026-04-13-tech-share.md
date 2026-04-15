---
layout: post
title: "实用开发小技巧：Maven配置、镜像源、git-cz、Git Flow、IDEA快捷操作"
date: 2026-04-15 20:24:00 +0800
categories: [开发]
math: false
mermaid: false
pin: false
tags: [Maven, Docker, Git, git-cz, Git Flow, IntelliJ IDEA, 效率工具, 后端开发]
---

## Maven 配置文件

### 简介

Maven 的配置以 `xml` 格式储存，一般有两个，一个是全局配置，一个是用户配置。其中全局配置位于 Maven 安装目录的 `conf/settings.xml`，用户配置位于 `~/.m2/settings.xml` （Windows同理，即: `%UserProfile%/.m2/settings.xml`）。可以使用以下命令查看：

```shell
mvn help:effective-settings -X | grep 'settings\.xml'
```

若要修改 Maven 配置文件，修改任意一个即可，不过若有重复项，用户配置会覆盖全局配置。

配置文件以 `settings` 标签为根标签，其子标签即为具体的配置项：
```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 https://maven.apache.org/xsd/settings-1.2.0.xsd">
  <!-- ...... -->
</settings>
```

### 镜像源配置

在实际开发中，可以通过配置 Maven 镜像源的方式加速 Maven 项目依赖项的下载，同时节省科学上网流量 ~~（实际上每月 300GB 根本用不完）~~ 。其原理是基于仓库 `id` 进行匹配，若镜像的 `mirrorOf` 匹配到了待下载文件的仓库 `id`，就会直接将请求 URL 替换为镜像仓库的 URL。下面是常用的镜像示例：

[阿里云 Maven](https://maven.aliyun.com/mvn/guide)

```xml
<mirrors>
  <mirror>
    <id>aliyunmaven</id>
    <name>Aliyun Public Repository</name>
    <url>https://maven.aliyun.com/repository/public</url>
    <mirrorOf>*</mirrorOf>
  </mirror>
</mirrors>
```

> 注：如果要配置多个镜像源，在 `mirrors` 里新增 `mirror` 即可。

注意这里的 `mirrorOf` 值为 `*`，也就是会替换所有的请求，如果你的项目使用了小众仓库，应修改此值。


### 本地仓库目录配置

Maven 会把下载的依赖项保存在本地仓库中，默认目录是 `~/.m2/repository` （Windows：`%UserProfile%/.m2/repository`），而Windows下 `%UserProfile%` 通常在 `C:\`，极易导致系统盘空间不足。因此，可以将本地仓库目录改为其它位置。下面以 `D:\maven_repo` 为例：

```xml
<localRepository>D:\maven_repo</localRepository>
```

## Docker 镜像源配置

在国内（不使用科学上网）拉取 Docker Hub 官方镜像时，经常会遇到网络不通的问题。为了节省科学上网流量，我们同样可以配置镜像源。

在 Linux 下，配置文件通常位于 `/etc/docker/daemon.json`；如果你使用 Docker Desktop （例如 Windows 中），配置文件可以通过 `Settings - Docker Engine` 编辑。

在根对象中新增以下属性即可配置镜像源：

```json
"registry-mirrors": [
  "https://docker.m.daocloud.io"
]
```

> 注：此处的镜像站仅作示例，具体可上网查询当前可用镜像源。若要添加多个镜像源，只需给此数组添加多个元素即可。

## `git cz` （Git Commitizen）

在使用 git 日常协作时，杂乱无章的 commit 信息会让代码历史难以追踪。`git cz` 提供了一套交互式命令行工具，引导开发者按照 **约定式提交（Conventional Commits）** 规范生成 commit 信息，便于后期自动生成更新日志或自动进行语义化版本发布。

### 安装

git-cz 是一个 npm 包，因此可以使用任何与 npm 仓库兼容的包管理器安装，例如：

```shell
npm install -g git-cz
```

> 注：此处不介绍 `npm` 等包管理器的安装，有需要者可自行上网查阅。

### 使用

安装成功后，即可不再使用 `git commit` ，而是改用 `git cz`，此工具会以问答的形式引导填写。

![效果图](assets/img/tech-share-git-cz.png)

## Git Flow 分支模型

> *参考资料：[Git Flow 菜鸟教程](https://www.runoob.com/git/git-flow.html)*

Git Flow 是一种基于 Git 的分支模型，旨在帮助团队更好地管理和发布软件。

Git Flow 由 Vincent Driessen 在 2010 年提出，并通过一套标准的分支命名和工作流程，使开发、测试和发布过程更加有序和高效。

### 分支介绍

Git Flow 主要由以下几类分支组成：`master`、`develop`、`feature`、`release`、`hotfix`。

![Git Flow 分支模型](assets/img/tech-share-git-flow.png)

`master` 分支：
- 永远保持稳定和可发布的状态。
- 每次发布一个新的版本时，都会从 `develop` 分支合并到 `master` 分支。

`develop` 分支：
- 用于集成所有的开发分支。
- 代表了最新的开发进度。
- 功能分支、发布分支和修复分支都从这里分支出去，最终合并回这里。

`feature` 分支：
- 用于开发新功能。
- 从 `develop` 分支创建，开发完成后合并回 `develop` 分支。
- 命名规范：`feature/feature-name`。

`release` 分支：
- 用于准备新版本的发布。
- 从 `develop` 分支创建，进行最后的测试和修复，然后合并回 `develop` 和 `master` 分支，并打上版本标签。
- 命名规范：`release/release-name`。

`hotfix` 分支：
- 用于修复紧急问题。
- 从 `master` 分支创建，修复完成后合并回 `master` 和 `develop` 分支，并打上版本标签。
- 命名规范：`hotfix/hotfix-name`。

## IDEA 快捷操作

> 注：此部分内容依赖于 IDE 设置，可以在 `设置 -> 按键映射` 中进行配置

- **快速跳转**；`Ctrl + 左键` 跳转到定义或用法；`Ctrl + Alt + 左键` 跳转到接口实现
- **向前跳转**：当你一路 `Ctrl + 左键` 跳进了源码深处，可以用 `Ctrl + Alt + ←` 返回上一个位置
- **悬停 Javadoc**：当本地有依赖项的源代码时，鼠标悬停在类或方法上一段时间后，就会弹出 Javadoc 悬浮窗；代码补全时的 Javadoc 提示可以通过 `设置 -> 编辑器 -> 常规 -> 代码补全 -> 以下时间后显示文档弹出窗口` 启用。
- **万能搜索**：双击 `Shift` 即可打开随处搜索弹窗
- **TODO 注释**：有很多 IDE 都会特殊标记 `// TODO` 开头的注释，主要用于指出未来应完成的功能。以后可以在 `视图 -> 工具窗口 -> TODO` 中查看所有的 TODO

