---
layout: post
title:  "CraftCMS 插件概述"
date: 2016-02-21 17:37:12 +0800
comments: true
categories: 软件开发
tags: [Craft, CraftCMS, CraftCMS 插件]
keywords: Craft, CraftCMS, CraftCMS 插件
description: CraftCMS插件概述
---

本文将简要介绍 CraftCMS 插件。


## 什么是 CraftCMS

[CraftCMS](https://craftcms.com/)是一个优秀的内容管理系统, 基于 PHP 框架 Yii(1.x) 构建, 借用 Craft 官方的定义:

Craft is a content management system (CMS) that’s laser-focused on doing one thing really, really well: managing content. And since content comes in all shapes and sizes, we’ve built Craft to be as flexible as possible, without compromising on the ease of use for content authors.

**Craft** 和 **WordPress** 不同的是, 后者的“内容”更关注于 Blog, 而前者的“内容”则更加泛化(**comes in all shapes and size**)。

Craft 有很多特性, 比如说发布内容时实时预览, 一键更新, 自定义域, 响应式设计等等, 更多内容, 请看[这里](http://code.tutsplus.com/tutorials/introduction-to-craft-cms--cms-22982)和[这里](https://viget.com/extend/why-we-love-craft-cms)(可能需要梯子)。

如果你需要一个站点来**展示**业务, 新闻, 产品等信息, 那 **Craft** 可能正好适合, 因为它就是方便你管理并展示内容。从这点也可以看出，**Craft 关注于内容，而不是那些复杂的业务逻辑**。

## Craft 插件 (plugin)

### 1.Craft Site 整体结构

我们先看一下一个完整的 Craft Site 是一个什么样的组织结构：

{% img center /images/craft-site-structures.png %}

主要是 **craft** 文件夹, 与其平行的 **public** 目录指向站点根目录, 也就是 **craft** 目录通过 url 是访问不到的，下面重点关注下 Craft 目录下的内容。

-   `craft/app` 目录是 Craft 的主程序所在位置, 当你从 Craft 官方站点下载下来 Craft 之后, craft/app 文件夹里的内容你就无需再动了, 不管是后续开发插件也好, 或者是直接部署使用也好, craft/app 不需要改动, 就算你改动了, 当 Craft 更新的时候也会重新覆盖掉你的改动。
-   `craft/config` 目录是站点的配置文件所在位置, 包括数据库连接字段, 自定义路由信息, 通用配置等等。
-   `craft/plugins` 目录是插件所在位置, 如果有必要新增功能的话, 需要写成插件的形式, 然后在站点的 admin 后台进行插件安装。有关插件的详细内容稍后介绍。
-   `craft/storage`, 顾名思义, 存储用, 备份文件, 运行时缓存, 日志等都存放在这里。
-   `craft/templates`, 存储页面的模板文件, 开发者需要在这里定义站点的页面, Craft 使用 **Twig** 来解析模板文件, 点击[这里](https://craftcms.com/docs/twig-primer)查看 Twig 的信息。

总体上, 基于 Craft 构建自己的站点需要做的就是：

-   搭建好运行环境**(比较典型的是 PHP + MySQL + Apache)**
-   将 Craft 放到 Web Server 目录下
-   基于自己的环境和需要对 Craft 进行配置
-   在 craft/templates 中定义自己的页面模板
-   根据需要, 开发插件并安装
-   使用

### 2.Plugin

如果你想为 Craft 新增功能, 那么需要写一个插件。 看一下插件的目录结构。从官方站点下载 Craft 之后，`plugin` 目录下是没有内容的，之后我安装了三个插件（即将插件拷贝到 `plugin` 目录下）: **businesslogic**, **cocktailrecipes**, **seomatic** 。

{% img center /images/plugin-folder-structures-1.png %}

**businesslogic** 插件的目录结构是这样的：

{% img center /images/plugin-folder-structures-2.png %}

其实, 一个标准的插件, 目录组织如下：

{% img center /images/plugin-std-folder.png %}

实际开发中, 并不需要像上图那样完备, 下面只对几个比较重要的目录作说明, 完整介绍见[这里](https://craftcms.com/docs/plugins/introduction)。

首先明确的一点是, 开发中常见的一种现象是**约定大于配置**, 所以规范最重要, 这里需要注意的就是**命名规范**, 包括插件名字, 插件中文件名, 类名等等, 有关如何命名, 参见[这里](https://craftcms.com/docs/plugins/introduction#anatomy-of-a-plugin)。

#### Primary Plugin Class

插件中最重要且不可缺少的文件是 **Primary Plugin Class**, 对应于 **businesslogic** 插件中的：

{% img center /images/primary-plugin-class.png %}

此文件相当于插件的自述文件, 在此文件中描述了插件的必要信息, 诸如 `Name`, `Description` , `Version` ,以及一些事件相应函数 `onAfterInstall()` 等等, 命名是按照 `插件名+Plugin`, 且首字母均为大写。

#### Controllers

如果插件需要处理 HTTP 请求的话, 那么需要由 Controller 来处理, 比如：

-   提交表单至controller中的某个action
-   AJAX请求触发controller的某个action
-   路由至某个特定URL, 触发某个action

前面提到过, Craft 基于 Yii 这个优秀的 MVC 框架构建, 关于 Yii(1.x) 的详细文档, 参见[这里](http://www.yiichina.com/doc/guide/1.1)。

#### Services

当业务变得复杂的时候, 使用 `service` 来处理, If your plugin provides any core functionality, it should be placed within a service. Services’ public methods are available globally throughout the system, for both your plugin and other plugins to access. They can be accessed via `craft()->serviceHandle->methodName()`.

需要说明, Controller 是在调用逻辑上介于 **调用者**和 `Services` 之间, 也就是说, 核心功能由 Services 提供。

Services 中的方法主要有以下特点：

-   业务逻辑比较复杂
-   需要和数据库交互
-   需要提供给外部使用（比如从别的Plugin进行调用）

#### Variables

你可以把 Variables 理解为**面向展示**用的 Services , 也就是说, 如果你想在页面模板文件中输出些什么东西, 你可以调用这里面的方法(方法的返回值即为输出内容)。That data can take basically any form... whether it's an array, a string, an integer, or even an object. Any type of PHP variable can be passed through a Variable method back to your Twig template.

当然, Variables 里也不应该包含复杂的逻辑或者是数据库相关的操作, 必要的时候, 你可以在 Variables 里通过调用 Services 的方法来完成特定功能。

#### Models

Any data that your plugin manages should be represented by models, and you should use them to pass that data around between your controllers, services, and templates.

#### Records

如果你写的插件需要持久化数据到数据库, 那你就需要创建一个 `record`, **Records** 有两个目的：

-   当你的插件被安装好后, record类所表示的实体就会自动被Craft识别并在数据库中为你创建好表
-   在插件中, 实体的增删查改需要通过record来完成

#### Templates

插件也可以有自己的页面模板文件, 当你在插件的 `Primary Plugin Class` 里将方法 `hasCpSection()` 的返回值设为 `true` 的时候, 标明该插件将会在站点的后台管理中有一个单独的用来配置的页面, 类似于下图中插件 **seomatic** 在后台的配置界面。

{% img center /images/seomatic-admin-section.png %}

这个配置界面就是 Templates 下的那些模板文件确定的。

还有其他的目录, 比如 **seomatic** 这个插件的目录中包含的文件夹就比较多。

{% img center /images/seomatic-plugin-folders.png %}

## 实际开发

以上就是对 **Craft** 插件的一个概述, 主要是想把整个结构梳理清楚。在有了一个大局观, 清楚了各个部件之间的关系后, 具体的开发就是一些拼图块, 堆积木了。

-   插件开发的官方文档: [plugins/introduction](https://craftcms.com/docs/plugins/introduction).
-   Craft 插件开发之 Hello World: [the-basics-of-craft-plugin-development](https://craftpl.us/tutorials/the-basics-of-craft-plugin-development).
-   GitHub 上一个实际的插件例子: [cocktailrecipes](https://github.com/amacneil/cocktailrecipes).
