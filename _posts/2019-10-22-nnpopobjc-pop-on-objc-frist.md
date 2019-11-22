---
layout: post
title: "NNPopObjc：在 Objective-C 上进行面向协议的编程（上）"
subtitle: 'NNPopObjc：在 Objective-C 上进行面向协议的编程'
author: "顾海军"
header-style: text
tags:
  - Objective-C
---

本文将介绍基于 [NNPopObjc](https://github.com/amisare/NNPopObjc) 在 Objective-C 上进行面向协议的编程。因为全部内容比较长，所以分成了上下两个部分，本文 (上) 主要介绍了 **NNPopObjc** 的使用，包括默认协议扩展、约束协议扩展等。下半部分，用于介绍 **NNPopObjc** 的实现思路和原理。

# 引子

基于 Swift 特性，Apple 在 2015 年 WWDC 上提出了面向协议编程 (Protocol Oriented Programming，以下简称 POP)的编程范式。相比与传统的面向对象编程 (OOP)，POP 显得更加灵活。但是由于 Objective-C 缺失协议扩展能力，导致 Objective-C 无法基于 POP 范式进行项目开发。**NNPopObjc** 用于为 Objective-C 提供协议扩展能力，从而实现 Objective-C 的面向协议编程。

# 协议扩展

通过协议扩展，可以为协议提供类方法、对象方法以及计算属性的默认实现。如果遵循该协议的类提供了自己所需方法或属性方法的实现，则将使用该类的实现，不使用扩展提供的实现。

> 协议定义中方法申明分为：`@required` 和 `@optional` ，由于 `@required` 声明的方法必须由遵守协议的类进行实现，所以由 `@required` 申明的方法一定会使用当前类的实现。

**NNPopObjc** 提供两个宏函数关键字：
- @nn_extension(...)
- @nn_where(...)

## 默认协议扩展

实现一个协议扩展，不对扩展进行任何约束，那么此扩展被认为是该协议的默认扩展。

声明一个 NNCodeProtocol 协议

``` 
@protocol NNCodeProtocol <NSObject>

@optional
+ (void)sayHelloPop;
- (void)sayHelloPop;

@end
```

为 NNCodeProtocol 协议提供默认实现

```
@nn_extension(NNCodeProtocol)

+ (void)sayHelloPop {
    DLog(@"+[%@ %s] code says hello pop", self, sel_getName(_cmd));
}

- (void)sayHelloPop {
    DLog(@"-[%@ %s] code says hello pop", [self class], sel_getName(_cmd));
}

@end
```

定义 NNCodeC 遵守 NNCodeProtocol 协议

```
// 类声明
@interface NNCodeC : NSObject <NNCodeProtocol>

@end
// 类实现
@implementation NNCodeC

@end
```

方法调用

```
[NNCodeC sayHelloPop];
[[NNCodeC new] sayHelloPop];
```

输出结果

```
+[NNCodeC sayHelloPop] code says hello pop
-[NNCodeC sayHelloPop] code says hello pop
```

## 约束协议扩展

#### @nn_where 关键字

**NNPopObjc** 提供 `@nn_where` 关键字为协议扩展提供约束，遵守协议的类需要满足约束才能够使用协议扩展提供的方法实现。

下面示例中通过 `@nn_where` 为 NNCodeProtocol 扩展添加了约束，约束要求`遵守协议的类必须是 NNCodeObjc `，其中 `self` 即当前要遵守协议的类。

```
@nn_extension(NNCodeProtocol, @nn_where(self == [NNCodeObjc class]))

+ (void)sayHelloPop {
    DLog(@"+[%@ %s] objc says hello pop", self, sel_getName(_cmd));
}

- (void)sayHelloPop {
    DLog(@"-[%@ %s] objc says hello pop", [self class], sel_getName(_cmd));
}

@end
```

方法调用

```
[NNCodeC sayHelloPop];
[[NNCodeC new] sayHelloPop];
[NNCodeObjc sayHelloPop];
[[NNCodeObjc new] sayHelloPop];
```

NNCodeObjc 优先使用满足约束协议的实现，输出结果

```
+[NNCodeC sayHelloPop] code says hello pop
-[NNCodeC sayHelloPop] code says hello pop
+[NNCodeObjc sayHelloPop] objc says hello pop
-[NNCodeObjc sayHelloPop] objc says hello pop
```

#### 类遵守协议列表

在约束协议扩展的 `@nn_where` 关键字后，可以跟随多个其他协议。遵守协议的类也必须同时要遵守该列表中所有的协议。

> 协议扩展增加协议列表后，在协议扩展中可以访问协议列表声明的方法，使协议扩展变的更加灵活。

定义一个协议

```
@protocol NNCodeNameProtocol <NSObject>

@optional
@property (nonatomic, strong) NSString* name;

@end
```

修改 NNCodeC 遵守该协议，并实现协议属性

```
@interface NNCodeC : NSObject < NNCodeProtocol, NNCodeNameProtocol>

@property (nonatomic, strong) NSString *name;

@end
```

修改 NNCodeProtocol 的约束协议扩展，访问 `name` 属性。

```
@nn_extension(NNCodeProtocol, @nn_where(), NNCodeNameProtocol)

+ (void)sayHelloPop {
    DLog(@"+[%@ %s] code says hello pop", self, sel_getName(_cmd));
}

- (void)sayHelloPop {
    DLog(@"-[%@ %s] code says hello pop, i am %@ ", [self class], sel_getName(_cmd), self.name);
}

@end
```

方法调用

```
[NNCodeC sayHelloPop];
NNCodeC *objc = [NNCodeC new];
objc.who = @"c";
[objc sayHelloPop];
```

输出结果

```
+[NNCodeC sayHelloPop] code says hello pop
-[NNCodeC sayHelloPop] code says hello pop, i am c
```

## 协议继承

在 NNCodeObjc 中扩展支持协议继承。

声明一个协议 NNCodeWhoProtocol 继承自 NNCodeProtocol

```
@protocol NNCodeWhoProtocol <NNCodeProtocol>

@optional
@property (nonatomic, strong) NSString* who;

@end
```

为 NNCodeWhoProtocol 协议提供扩展

```
@nn_extension(NNCodeWhoProtocol, @nn_where(), NNCodeNameProtocol)

- (void)sayHelloPop {
    DLog(@"-[%@ %s] code says hello pop", [self class], sel_getName(_cmd));
}

- (NSString *)who {
    NSString *who = [NSString stringWithFormat:@"-[%@ %s] code says I am %@", [self class], sel_getName(_cmd), self.name];
    return who;
}

- (void)setWho:(NSString *)who {
    self.name = who;
}

@end
```

方法调用

```
[NNCodeC sayHelloPop];
NNCodeC *objc = [NNCodeC new];
objc.who = @"c";
[objc sayHelloPop];
DLog(@"%@", c.who);
```

输出结果

```
+[NNCodeC sayHelloPop] code says hello pop
-[NNCodeC sayHelloPop] code says hello pop
-[NNCodeC who] code says I am c
```

# 同一协议多个扩展的问题

协议的扩展分为两类：

- 默认协议扩展
- 约束协议扩展

同一协议可以多个扩展，但在遵守协议类的角度，一个协议的默认协议扩展和约束协议扩展分别不能为多个，（即 默认协议扩展的个数 <= 1 && 约束协议扩展的个数 <= 1），如果存在多个就会产生所谓的“菱形缺陷”问题，程序执行时会触发 **NNPopObjc** assert 断言。

# 一个使用示例

[ProtocolNetwork](https://github.com/MDCC2016/ProtocolNetwork) 是 [喵神王巍](https://onevcat.com/#blog) 在 MDCC 16 (移动开发者大会) iOS 专场《面向协议编程与 Cocoa 的邂逅》主题演讲时使用的 Demo 工程， Demo 演示了 Swift 面向协议编程在 Cocoa 中的使用示例。

[ProtocolNetworkObjc](https://github.com/amisare/ProtocolNetworkObjc) 是基于 **NNPopObjc** 完全复刻的 Objective-C 版 ProtocolNetwork 。

由于 ProtocolNetworkObjc  完全复刻  ProtocolNetwork，具体代码演变及分析可参考：喵神王巍的博客 [面向协议编程与 Cocoa 的邂逅 (下)](https://onevcat.com/2016/12/pop-cocoa-2/) 中相关内容。限于篇幅，本文不对代码进行讲解。

本文下半部分将介绍 **NNPopObjc** 的实现思路和原理。