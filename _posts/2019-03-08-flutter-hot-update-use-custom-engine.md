---
layout: post
title: "Flutter你想要的热更新之自定义初始化函数的使用"
subtitle: 'Flutter你想要的热更新之自定义初始化函数的使用'
author: "顾海军"
header-style: text
tags:
  - Flutter
---

## Flutter你想要的热更新之自定义初始化函数的使用

本章将基于上篇 `Flutter你想要的热更新之编译构建engine` 构建并使用 [FlutterDartProject 自定义初始化函数](https://github.com/amisare/engine/commit/79cdbc5997e21b14f515022db5f9dd46423f848f)

### 1. 修改 engine 并构建

1. 根据 [FlutterDartProject 自定义初始化函数](https://github.com/amisare/engine/commit/79cdbc5997e21b14f515022db5f9dd46423f848f) 修改 engine 代码。
2. 编译
	```
    #进入 src 目录
    cd engine/src
    #编译（分别编译真机文件，模拟器文件和服务端文件）
    ninja -C out/ios_debug_unopt && ninja -C out/ios_debug_sim_unopt && ninja -C out/host_debug_unopt
    ```
3. 运行 Flutter App
	```
    flutter run --local-engine-src-path=/path/to/engine/src  --local-engine=ios_debug_sim_unopt
    ```

### 2. 修改 AppDelegate

修改 AppDelegate ，初始化 FlutterDartProject ，并将资源加载的路径指向沙盒的 Documents/flutter_assets 目录

```
#include "AppDelegate.h"
#include "GeneratedPluginRegistrant.h"
#import <Flutter/Flutter.h>

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application
didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [GeneratedPluginRegistrant registerWithRegistry:self];
    // Override point for customization after application launch.
    
    NSString *assertsPath = [NSHomeDirectory() stringByAppendingPathComponent:@"Documents/flutter_assets"];
    NSURL *assertsUrl = [NSURL fileURLWithPath:assertsPath];
    
    FlutterDartProject *dartPro = [[FlutterDartProject alloc] initWithFlutterAssetsURL:assertsUrl];
    FlutterViewController *vc = [[FlutterViewController alloc] initWithProject:dartPro nibName:nil bundle:nil];
    
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
    self.window.rootViewController = vc;
    [self.window makeKeyAndVisible];
    
    return [super application:application didFinishLaunchingWithOptions:launchOptions];
}

@end
```

### 3. 提供 flutter_assets

1. 经过 [2. 修改 AppDelegate](#2. 修改 AppDelegate) 的修改，因为 `FlutterDartProject` 不再默认加载工程中的 `flutter_assets` 文件，且沙盒没有 `flutter_assets` 资源 `assertsUrl` 为空，所以直接 `cmd + r` 执行，首页应该是`白屏`。

2. 将工程中的 `flutter_assets` 拷贝到沙盒 `Documents` 目录中，`cmd + r` 执行。

### 4. 总结

到目前为止，我们已经实现了将 flutter_assets 放到工程外的沙盒进行加载了！





















