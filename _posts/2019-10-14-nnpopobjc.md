---
layout: post
title: "NNPopObjc Objective-C 面向协议的编程"
subtitle: 'NNPopObjc Objective-C 面向协议的编程'
author: "顾海军"
header-style: text
tags:
  - Objective-C
---

## GitHub

*  [NNPopObjc](https://github.com/amisare/NNPopObjc) 

## 介绍

一些[文章](https://www.jianshu.com/p/0c34114b94e7)提到：
>OC无法做到面向协议开发，而Swift可以，因为Swift可以做到协议方法的具体实现，而OC不行

NNPopObjc 受面向协议编程的启发，为协议提供了实现扩展的功能。实现 Objective-C 面向协议的编程。

## 快速开始

### 声明协议

在 `.h` 文件中声明协议

```objective-c
@protocol NNDemoProtocol <NSObject>

@optional
- (void)sayHelloPop;
+ (void)sayHelloPop;

@end
```

### 扩展协议

扩展协议需要在 `.m` 中实现

```objective-c
/// 默认协议扩展
@nn_extension(NNDemoProtocol)

+ (void)sayHelloPop {
    DLog(@"+[%@ %s] code say hello pop", self, sel_getName(_cmd));
}

- (void)sayHelloPop {
    DLog(@"-[%@ %s] code say hello pop", [self class], sel_getName(_cmd));
}

@end
```

### 遵守协议

- 创建类

```objective-c
@interface NNDemoObjc : NSObject <NNDemoNameProtocol>

@end
```

- 实现类

```
@implementation NNDemoObjc

@end
```

### 使用类

- 调用方法

```objective-c
[NNDemoObjc sayHelloPop];
[[NNDemoObjc new] sayHelloPop];
```

- 输出日志

```objective-cc
+[NNDemoObjc sayHelloPop] code say hello pop
-[NNDemoObjc sayHelloPop] code say hello pop
```

## 安装

NNPopObjc 支持 CocoaPods 方式集成。

### 安装 CocoaPods

使用以下命令安装：

```bash
$ gem install cocoapods
```

### Podfile

将 NNPopObjc 添加到 `Podfile` ，通过 CocoaPods 集成 NNPopObjc 到 Xcode 项目：

```ruby
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '8.0'

target 'TargetName' do
pod 'NNPopObjc', '~> 0.2.1'
end
```

执行命令：

```bash
pod install
```

#### 如安装失败，提示：

```bash
[!] Unable to find a specification for `NNPopObjc`
```

尝试使用命令：

```bash
pod install --repo-update
```