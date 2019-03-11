---
layout: post
title: "Flutter你想要的热更新之项目集成"
subtitle: 'Flutter你想要的热更新之项目集成'
author: "顾海军"
header-style: text
tags:
  - Flutter
---

## Flutter你想要的热更新之项目集成

起初本章想叫 《Flutter你想要的热更新之工程化》来着，仔细想想也达不到`工程化`，毕竟`工程化`还包含很多内容。
叫`混合开发`吧，毕竟主题是`Flutter你想要的热更新`。折中，就叫《Flutter你想要的热更新之项目集成》吧。

### 1. 现有项目的集成方式

目前项目集成有闲鱼技术和`Flutter`官方的两种方式。
本文基于 [add2app 方式](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps) 的方式在项目中进行集成。

#### 1.1 闲鱼技术的 [Flutter混合工程改造实践](https://www.jianshu.com/p/64608e67af26)
不得不说先入为主的重要性，Flutter 初期在混合开发这方面确实做的不足，闲鱼技术在这个上面踩了不少的坑。加上闲鱼技术在各种平台上都注册了官方账号，并且文章同步发布，嗯……，你懂得，度娘搜到的基本上是他们的实践！

#### 1.2 Flutter 的 [add2app 项目](https://github.com/flutter/flutter/projects/28)
由于开发者对 Flutter 的混合模式的呼声越来越高，为了满足开发者要求，Flutter 专门为了这个问题建立了 wiki 并且进行了持续 4 个月 42 个版本的更新，提供了 [add2app 方式](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps) 的集成。

#### 1.3 两种开发模式的比较
1. [Flutter混合工程改造实践](https://www.jianshu.com/p/64608e67af26) 是真麻烦（毕竟在 Flutter 初期能解决问题的办法都是好办法）
2. [add2app 方式](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps) 是真简单

### 2. 项目集成

本文只介绍 iOS 的集成，采用 [add2app 方式](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps) ，
为集成自己构建的 Flutter.framework 会在官方步骤的基础上增加和删减部分操作。

#### 2.1 生成项目所需文件

为了方便，将下面内容放到同一目录中。
1. 新建项目目录 `engine_app2app`
2. 在 `engine_app2app/` 下，创建原生 Xcode 项目 `FlutterApp2App`
3. 在 `engine_app2app/` 下，创建用于 Flutter 集成的 flutter_module
```
flutter create -t module flutter_module
```
4. 组织 `3. Flutter你想要的热更新之编译构建engine` 文中 `engine/src/out` 目录下构建的 `Flutter.framework` , 结构如下
```
flutter_frameworks/
├── ios_debug //iOS arm64 framework
│   └── Flutter.framework
└── ios_debug_sim //iOS X86_64 framework
│   └── Flutter.framework
├── ios_debug_all //iOS arm64 X86_64 合并后的 framework
    └── Flutter.framework
```
并拷贝到 `engine_app2app/` 目录下。
各目录中包含了不同架构的 framework ：
	- ios_debug: arm64
	- ios_debug_sim: X86_64
	- ios_debug_all: arm64, X86_64 (需自己合并)

经过以上步骤，我们得到一个如下的文件目录结构

```
engine_app2app
├── FlutterApp2App
│   ├── FlutterApp2App
│   ├── FlutterApp2App.xcodeproj
│   ├── FlutterApp2AppTests
│   └── FlutterApp2AppUITests
├── flutter_frameworks
│   ├── ios_debug
│   │   └── Flutter.framework
│   ├── ios_debug_all
│   │   └── Flutter.framework
│   └── ios_debug_sim
│       └── Flutter.framework
└── flutter_module
    ├── .android
    ├── .idea
    ├── .ios
    ├── lib
    └── test
```

#### 2.2 使用 cocoapods 引用 Flutter

1. `FlutterApp2App` 目录下创建 Podfile 文件
    ```
    platform:ios,’8.0’

    use_frameworks!

    target :FlutterPrograms do

    end
    ```
2. （官方）在 Podfile 文件末尾加入：
    ```
    flutter_application_path = '../flutter_module'
    eval(File.read(File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')), binding)
    ```
3. （增加）在 Podfile 文件末尾加入：
    ```
    # 替换 Flutter 官方 Flutter.framework
    FileUtils.rm_r('../flutter_module/.ios/Flutter/engine/Flutter.framework', :force => true)
    FileUtils.cp_r('../flutter_frameworks/ios_debug_all/Flutter.framework', '../flutter_module/.ios/Flutter/engine/Flutter.framework')
    ```
4. 执行 pod install

#### 2.3 修改已有工程

1. 修改 `AppDelegate` 的父类为 `FlutterAppDelegate`
2. 在 iOS 项目中调用 `Flutter`
	AppDelegate.h 文件
    ```
    #import <UIKit/UIKit.h>
    #import <Flutter/Flutter.h>

    @interface AppDelegate : FlutterAppDelegate

    @end
    ```
    AppDelegate.m 文件
    ```
    #include "AppDelegate.h"
    #include <FlutterPluginRegistrant/GeneratedPluginRegistrant.h>

    @implementation AppDelegate

    - (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
        [GeneratedPluginRegistrant registerWithRegistry:self];
        // Override point for customization after application launch.

        NSString *assertsPath = [NSHomeDirectory() stringByAppendingPathComponent:@"Documents/flutter_assets"];
        NSURL *assertsUrl = [NSURL fileURLWithPath:assertsPath];

        NSLog(@"assertsPath:%@", assertsPath);

        FlutterDartProject *dartPro = [[FlutterDartProject alloc] initWithFlutterAssetsURL:assertsUrl];
        FlutterViewController *vc = [[FlutterViewController alloc] initWithProject:dartPro nibName:nil bundle:nil];

        self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
        self.window.rootViewController = vc;
        [self.window makeKeyAndVisible];

        return [super application:application didFinishLaunchingWithOptions:launchOptions];
    }

    @end
    ```
3. 将 `Flutter` 资源文件拷贝到沙盒的 `Documents/` 目录下
4. 在 XCode 中 `cmd + r` 执行

### 3. 总结

本文基于 Flutter 官方的 app2app 方式集成了自己构建生成的 Flutter.framework 。
该环境是无法进行 Flutter 调试的，因为我们没有按照官方文档增加以下脚本：
```
"$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh" build
"$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh" embed
```
之所以没有增加上面的脚本，是因为，上面的脚本会根据平台替换构建时的 Flutter.framework 。比如，模拟器调试时会使用 X86_64 的  Flutter.framework，在真机打包时会使用 arm64 的 Flutter.framework 。如果增加了上面的脚本，XCode构建时就会把我们已经替换的自己构建生成的 Flutter.framework ，替换回来！





























