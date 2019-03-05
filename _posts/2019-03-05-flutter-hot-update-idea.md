---
layout: post
title: "Flutter你想要的热更新之思路"
subtitle: 'Flutter你想要的热更新之思路'
author: "顾海军"
header-style: text
tags:
  - Flutter
---

### Flutter你想要的热更新之思路

#### 1. Flutter 介绍

Flutter 是谷歌的移动 UI 框架，可以快速在 iOS 和 Android 上构建高质量的原生用户界面。其开发间段基于 JIT 的热重载，生产间段基于 AOT 的原生性能，以及 iOS 和 Android 的跨平台是它吸引开发者的亮点。

快速开发，原生性能，跨平台，拥有了这些的同时，开发者所期望的热更新迟迟不肯支持。当然 Flutter 团队肯定有自己的考虑。

这里是 github Flutter 仓库中关于热更新话题的一些讨论 [issues14330](https://github.com/flutter/flutter/issues/14330) 。

#### 2. iOS 热更新

从技术的角度来讲，未签名打包到 iOS 中的可执行文件是无法被 iOS 系统执行的。在 AOT 模式的生产环境中 Flutter 的可执行文件会被打包到 App.framework 中， App.framework 仅仅是 Flutter 一个容器，通过对 App.framework 的签名，这样 Flutter 应用就能够被 iOS 系统加载，从而被 Flutter.framework 调用执行。

在 AOT 模式的生产环境中生成的可执行文件的指令是原生指令，即这些指令直接由 CPU 执行。所以，基于 AOT 模式是不可能实现热更新的。

#### 3. Flutter 热更新思路

既然 Flutter 能基于 JIT 模式实现快速开发，没错，思路就是基于 JIT 实现热更新。

- 可行性
	1. Flutter 是完全的开源项目，我们可以获取所有源码，包括 Flutter ， Flutter Engine ，Dart SDK 以及配套的构建工具。所以不会存在闭源导致的阻碍。
	2. JIT 模式下 Flutter 应用是非原生指令，不会被 iOS 直接执行，类似 JS ，所以不涉及到签名。
	3. Flutter.framework 作为 Dart VM 的载体，用来执行 Flutter 应用。打包签名后， App 整个生命周期不需要再被修改。
- 性能考虑
	1. 既然基于 JIT 模式，那么一定会牺牲性能。
	2. 过早优化是万恶之源。先尝试实现，实现了以后，再进行优化，万一性能满足那？

#### 4. 开发环境

- 开发环境
文章主题是 Flutter 在 iOS 上实现热更新，所以所有实现均在 Mac 下完成。

- Flutter 版本
文章中所有实践均基于目前的第一个 stable 版本即 Flatter v1.0.0 。