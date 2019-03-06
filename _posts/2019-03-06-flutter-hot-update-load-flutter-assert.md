---
layout: post
title: "Flutter你想要的热更新之加载 Flutter 资源"
subtitle: 'Flutter你想要的热更新之加载 Flutter 资源'
author: "顾海军"
header-style: text
tags:
  - Flutter
---

## Flutter你想要的热更新之加载 Flutter 资源

本章目的是要搞清楚在 iOS 端 Flutter 是如何获取需要加载的 Flutter 资源。

### 1. 准备工作

本章的准备中做仅仅是为了方便了解 Flutter 项目的结构，及查看相关代码。

1. 搭建开发环境
	搭建 Flutte 非常简单，这里不在赘述，请参考官网文档。
    [https://flutter.io/docs/get-started/install](https://flutter.io/docs/get-started/install)

2. 创建一个Flutter App
	使用终端创建一个 Flutter 应用 myapp ，请参考官方文档。
    [https://flutter.io/docs/get-started/test-drive?tab=terminal#create-app](https://flutter.io/docs/get-started/test-drive?tab=terminal#create-app)

### 2. FlutterViewController，FlutterDartProject 以及 FlutterEngine 三者的关系

类声明头文件位于 `Flutter.framework` 中

1. FlutterDartProject 用来获取 Flutter 资源和创建 FlutterEngine 实例。
2. FlutterEngine 是 Flutter engine 对象，用来渲染 FlutterViewController 的 FlutterView 。
3. FlutterViewController 继承自 UIViewController ，Flutter 根据需要做了扩展，角色同 UIViewController 。

创建顺序 `FlutterDartProject -> FlutterEngine -> FlutterViewController`

以下是 FlutterViewController 创建代码：

```
FlutterDartProject *dartPro = [[FlutterDartProject alloc] initWithPrecompiledDartBundle:nil];
FlutterViewController *vc = [[FlutterViewController alloc] initWithProject:dartPro nibName:nil bundle:nil];
```

### 3. FlutterDartProject 资源获取的实现

- Flutter.framework 是由 Flutter engine 构建产生，阅读 engine 源码，一定要确定 engine 与 Flutter 对应的版本。当前 Flutter 版本对应的 engine 版本号记录在 Flutter 仓库的 [bin/internal/engine.version](https://github.com/flutter/flutter/blob/master/bin/internal/engine.version) 文件中。本文 Flutter 1.0.0 版本的 engine 版本号为 [7375a0f414bde4bc941e623482221db2fc8c4ab5](https://github.com/flutter/flutter/blob/v1.0.0/bin/internal/engine.version)。
- 
- 为了方便阅读 Flutter 1.0.0 版本的 engine ，作者在自己的 github 上 fork 了 engine 并标记了对应 Flutter 1.0.0 版本的 tag 标签 [flutter_v1.0.0](https://github.com/amisare/engine/tree/flutter_v1.0.0) 。文章中关于代码引用将直接使用该 git 仓库的链接。

#### 3.1. FlutterDartProject 的三个初始化方法：
> 其中后两个初始化函数已经在 engine 的后续版本中被删除，所以我们这里只关注第一个初始化函数就可以了。

```
/**
 * Initializes with a specific
 */
- (instancetype)initWithPrecompiledDartBundle:(NSBundle*)bundle NS_DESIGNATED_INITIALIZER;

/**
 * Initializes with a specific set of Flutter Assets, with a specified location of
 * main() and Dart packages.
 */
- (instancetype)initWithFlutterAssets:(NSURL*)flutterAssetsURL
                             dartMain:(NSURL*)dartMainURL
                             packages:(NSURL*)dartPackages NS_DESIGNATED_INITIALIZER;

/**
 * Initializes from a specific set of Flutter Assets.
 */
- (instancetype)initWithFlutterAssetsWithScriptSnapshot:(NSURL*)flutterAssetsURL
    NS_DESIGNATED_INITIALIZER;
    
```

#### 3.2. FlutterDartProject 初始化函数的实现
> 初始化函数实现位于 [shell/platform/darwin/ios/framework/Source/FlutterDartProject.mm](https://github.com/amisare/engine/blob/flutter_v1.0.0/shell/platform/darwin/ios/framework/Source/FlutterDartProject.mm) 。

##### 3.2.1 `initWithPrecompiledDartBundle:` 函数实现


1. 使用内部成员 _precompiledDartBundle 持有 bundle 。
2. 调用 `DefaultSettingsForProcess` 返回一个 [blink::Settings](https://github.com/amisare/engine/blob/flutter_v1.0.0/common/settings.h) 对象。


##### 3.2.2 `DefaultSettingsForProcess` 函数实现

1. 从 Flutter.framework 获取 `settings.icu_data_path`
2. 如果是 Dart VM 是 `IsRunningPrecompiledCode` （AOT 模式）
    1. 尝试获取 `settings.application_library_path`
    > 分别尝试从 `bundle` , `main bundle 的 Info.plist 的 FLTLibraryPath 字段` 以及 App.framework 获取 settings.application_library_path
3. 获取 `settings.assets_path`
    1. 获取失败
        1. 抛出异常
    2. 获取成功
        1. 如果是 Dart VM 不是 `IsRunningPrecompiledCode` （JIT 模式）
            1. 获取 `settings.application_kernel_asset`

##### 3.2.3 结论

1. 分析 `DefaultSettingsForProcess` 可以发现，在 JIT 模式下只需要设置以下三个参数即可满足 Flutter 运行条件：
	- `settings.icu_data_path`
	- `settings.assets_path`
	- `settings.application_kernel_asset`

2. 实现中 `settings.application_kernel_asset` 参数最终是被固定写死的。

### 4. 为 FlutterDartProject 自定义初始化函数

因为 FlutterDartProject 的初始化函数源码中包含了很多内部类型，所以我们需要修改 engine 源码，在其原始实现中自定义 FlutterDartProject 的初始化函数和 `DefaultSettingsForProcess` 函数。

#### 4.2 自定义 `initWithFlutterAssetsURL:` 初始化函数

为减少篇幅，新增代码见 [FlutterDartProject 自定义初始化函数](https://github.com/amisare/engine/commit/79cdbc5997e21b14f515022db5f9dd46423f848f)。






