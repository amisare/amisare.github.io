---
layout: post
title: "Flutter你想要的热更新之为改善性能"
subtitle: 'Flutter你想要的热更新之为改善性能'
author: "顾海军"
header-style: text
tags:
  - Flutter
---

## Flutter你想要的热更新之为改善性能

本章将使用两个开源的 Flutter 项目来做性能改善前后的对比：
- [flutter_ui_challenge_filter_menu](https://github.com/amisare/flutter_ui_challenge_filter_menu)
- [HistoryOfEverything](https://github.com/2d-inc/HistoryOfEverything)

为了保证文章时效性，作者分别在自己的 github 账户中 fork 了以上两个仓库，并分别创建了 `flutter_hot_update` 分支：
- [flutter_ui_challenge_filter_menu](https://github.com/amisare/flutter_ui_challenge_filter_menu/tree/flutter_hot_update)
- [HistoryOfEverything](https://github.com/amisare/HistoryOfEverything/tree/flutter_hot_update)

由于 [flutter_ui_challenge_filter_menu](https://github.com/amisare/flutter_ui_challenge_filter_menu/tree/flutter_hot_update) 操作简单，所以本章将使用 [flutter_ui_challenge_filter_menu](https://github.com/amisare/flutter_ui_challenge_filter_menu/tree/flutter_hot_update) 作为 demo 来进行性能改善的介绍。

### 1. 体验 "翔" 的速度

1. 克隆 [flutter_ui_challenge_filter_menu](https://github.com/amisare/flutter_ui_challenge_filter_menu/tree/flutter_hot_update)
2. 参考 `Flutter你想要的热更新之编译构建engine` 中 `3.使用本地构建的 engine` 章节构建执行。

### 2. 修改 engine 编译项改善性能

回到 `Flutter你想要的热更新之编译构建engine` 中 `2.2 本地构建 engine` 章节，根据 Flutter 官方 [wiki](https://github.com/flutter/flutter/wiki/Setting-up-the-Engine-development-environment) 我们在`生成构建文件`是有一个 `--unoptimized` 参数，官方 [wiki](https://github.com/flutter/flutter/wiki/Setting-up-the-Engine-development-environment) 的描述：
```
`--unoptimized disables` C++ compiler optimizations and does not strip debug symbols.
```

这里我们通过去掉 `--unoptimized disables` 参数来实现，性能上的改善。

1. 生成构建文件
	```
    #进入 src 目录
    cd engine/src
    #生成构建文件（分别生成真机构建文件，模拟器构建文件和服务端构建文件）
    ./flutter/tools/gn --ios && ./flutter/tools/gn --ios --simulator
    ```
2. 编译
	```
    #进入 src 目录
    cd engine/src
    #编译（分别编译真机文件，模拟器文件和服务端文件）
    ninja -C out/ios_debug && ninja -C out/ios_debug_sim
	```

3. 使用 `ios_debug_sim` 参考 ` Flutter你想要的热更新之编译构建engine` 中 `3.使用本地构建的 engine` 章节构建执行。

### 3. 总结

本章仅仅是去掉了一个 engine 编译选项 `--unoptimized disables` ，就可以很明显的感受到 Flutter 性能的提升。不难想象在深入的了解 Flutter 及 Dart 之后，在性能方面我们会有更多的事情可以做。




























