---
layout: post
title: "Flutter你想要的热更新之为dill瘦身"
subtitle: 'Flutter你想要的热更新之为dill瘦身'
author: "顾海军"
header-style: text
tags:
  - Flutter
---

## Flutter你想要的热更新之为dill瘦身

### 1. 换了个马甲的 dill

在 flutter_assets 有个 kernel_blob.bin 文件，其实在 `2. Flutter你想要的热更新之Flutter资源加载` 中就见过它，它出现在
在 [FlutterDartProject.mm](https://github.com/amisare/engine/blob/flutter_v1.0.0/shell/platform/darwin/ios/framework/Source/FlutterDartProject.mm) 中，文件头部定义了：
```
static const char* kApplicationKernelSnapshotFileName = "kernel_blob.bin";
```
没错，这就是我们要加载的 Flutter 可执行文件，那么这个 kernel_blob.bin 中到底是什么内容那？

进入这个文件的文件夹，在终端执行：
```
strings kernel_blob.bin
```
同样在 myapp/build 输出 app.dill：
```
strings app.dill
```
没错，它们是同一个文件，仅仅是换了个后缀和名称。

截取部分输出，内部包含了 Dart 源文件注释：
```
/// int _count = 0;
/// Widget build(BuildContext context) {
///   return Scaffold(
///     appBar: AppBar(
///       title: Text('Sample Code'),
///     ),
///     body: Center(
///       child: Text('You have pressed the button $_count times.'),
///     ),
///     bottomNavigationBar: BottomAppBar(
///       child: Container(height: 50.0,),
///     ),
///     floatingActionButton: FloatingActionButton(
///       onPressed: () => setState(() {
///         _count++;
///       }),
///       tooltip: 'Increment Counter',
///       child: Icon(Icons.add),
///     ),
///     floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked,
///   );
/// }
```

### 2. 什么是 dill

一句话： `Dart 的中间语言` ，[到 Dart sdk 项目中了解更多](https://github.com/dart-lang/sdk/wiki/Kernel-Documentation) 。

### 3. 为 dill 瘦身

一个 myapp 这么简单的 demo ， kernel_blob.bin 居然有 14.8M 之多。
对于 kernel_blob.bin 的瘦身，我目前只找到了一种方式：`禁止生成的 kernel_blob.bin 中包含 Dart 源文件注释`。

#### 3.1 修改 compile.dart

在 Flutter 开发环境仓库 [compile.dart](https://github.com/flutter/flutter/blob/v1.0.0/packages/flutter_tools/lib/src/compile.dart) 文件的 `class KernelCompiler` 类中修改 `command` 编译参数列表，为其增加参数成员 `--no-embed-source-text` ：

```
final List<String> command = <String>[
  engineDartPath,
  frontendServer,
  '--sdk-root',
  sdkRoot,
  '--strong',
  '--target=$targetModel',
  '--no-embed-source-text',
];
```

#### 3.2 重新构建 flutter_tools

根据官方 wiki [making-changes-to-the-flutter-tool](https://github.com/flutter/flutter/wiki/The-flutter-tool#making-changes-to-the-flutter-tool) 中说明，只要删除位于 Flutter 开发环境仓库 `bin/cache/` 目录中的 `flutter_tools.snapshot` 文件，重新执行 `flutter run` 即可生成新的 `flutter_tools` 工具。

#### 3.3 重新生成 kernel_blob.bin

1. 删除原 XCode `flutter_assets` 文件夹中的 `kernel_blob.bin` （不删除文件可能会无法生成瘦身后的新文件）。

2. 运行 Flutter App
	```
    flutter run --local-engine-src-path=/path/to/engine/src  --local-engine=ios_debug_sim_unopt
    ```
3. 瘦身后的 `kernel_blob.bin` 文件应该在 5.7M 左右。

### 4. 总结

本章我们通过修改 `flutter_tools` 实现了对 Flutter 的 kernel 文件 `kernel_blob.bin` 的瘦身。通过 strings 命令查看 `kernel_blob.bin` ，我们会发现还存在进一步瘦身的可能，要想进一步减小 `kernel_blob.bin` 体积就必须更深入的了解 Flutter 及 Dart 的实现。



























