---
layout: post
title: "NNDecimalNumber 提供链式调用计算 NSDecimalNumber"
subtitle: '提供 NSDecimalNumber 运算扩展 ，通过链式调用简化数值计算。'
author: "顾海军"
header-style: text
tags:
  - Objective-C
---

## GitHub

[NNDecimalNumber](https://github.com/amisare/NNDecimalNumber)

## 介绍

提供 NNString， NSNumber 的 NSDecimalNumber 的计算及比较`分类` ，通过链式调用简化数值计算。

## 使用

- 示例一：全部使用 NSNumber 计算

```
    //( ( ( ( 3 - 2 + 5 ) × 3 ) × ( 10 × 2 ) ) ÷ 2 )
    NSString *v = @(3).nn_sub(@(2)).nn_add(@(5)).nn_mul(@(3)).nn_mul(@(10).nn_mul(@(2))).nn_div(@(2));
```

- 示例二：全部使用 NSString 计算

```
    //( ( ( ( 3 - 2 + 5 ) × 3 ) × ( 10 × 2 ) ) ÷ 2 )
    NSString *v = @"3".nn_sub(@"2").nn_add(@"5").nn_mul(@"3").nn_mul(@"10".nn_mul(@"2")).nn_div(@"2");
```

- 示例三：使用 NSNumber 与 NSString 混合计算

```
    //( ( ( ( 3 - 2 + 5 ) × 3 ) × ( 10 × 2 ) ) ÷ 2 )
    NSString *v = @(3).nn_sub(@"2").nn_add(@(5)).nn_mul(@"3").nn_mul(@(10).nn_mul(@"2")).nn_div(@(2));
```

- 示例四：幂运算

```
    {
        //( ( 2 × 5 + ( 2 × 2 ) ) ^ 2 )
        NSString *v = @"2".nn_mul(@(5)).nn_add(@"2".nn_mul(@"2")).nn_power(@(2));
    }
    {
        //( ( 2 × 5 + ( 2 × 2 ) ) × 10 ^ 2 )
        NSString *v = @(2).nn_mul(@"5").nn_add(@"2".nn_mul(@(2))).nn_mulPowerOf10(@(2));
    }
```

- 示例五：数值比较

```
    NSString *v0 = @"100";
    NSNumber *v1 = @(100.1);
    if ([v0 nn_decimalIsEqualTo:v1]) {
        NSLog(@"v0 == v1 : v0 = %@, v1 = %@", v0, v1);
    }
    else if ([v0 nn_decimalIsGreaterThan:v1]) {
        NSLog(@"v0 > v1 : v0 = %@, v1 = %@", v0, v1);
    }
    else if ([v0 nn_decimalIsGreaterThanOrEqualTo:v1]) {
        NSLog(@"v0 <= v1 : v0 = %@, v1 = %@", v0, v1);
    }
    else if ([v0 nn_decimalIsLessThan:v1]) {
        NSLog(@"v0 < v1 : v0 = %@, v1 = %@", v0, v1);
    }
```

- 示例六：设置 NSDecimalNumberBehaviors

```
    // 设置全局 NSDecimalNumberBehaviors
    {
        [NSString nn_setDecimalNumberGlobalBehavior:[DecimalNumberGlobalBehavior class]];
        NSString *v = @"1".nn_div(@(3));
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
    // 当前计算式 NSDecimalNumberBehaviors
    {
        DecimalNumberBehavior *behavior = [DecimalNumberBehavior new];
        NSString *v = @"1".nn_behavior(behavior).nn_div(@(3)).nn_div(@(2));
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
    // 使用 NSDecimalNumberHandler
    {
        NSDecimalNumberHandler *behavior = [[NSDecimalNumberHandler alloc] initWithRoundingMode:NSRoundPlain
                                                                                          scale:4
                                                                               raiseOnExactness:false
                                                                                raiseOnOverflow:false
                                                                               raiseOnUnderflow:false
                                                                            raiseOnDivideByZero:false];
        NSString *v = @"1".nn_behavior(behavior).nn_div(@(3));
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
```

- 示例六：异常计算（异常计算结果统一为字符串：@"NaN"，即`[[NSDecimalNumber notANumber] stringValue]`）

```
    {
        //( 2 × [UIView new] )
        NSString *v = @"2".nn_mul([UIView new]);
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
    
    {
        //( 2 / 0 )
        NSString *v = @"2".nn_div(@"0");
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
    
    {
        //( nil / nil )
        NSString *v =  NN_Trust(nil).nn_div(nil);
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
    
    {
        //( nil / 0 )
        NSString *v =  NN_Trust(nil).nn_div(@(0));
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
    
    {
        //( nil / [UIView new] )
        NSString *v =  NN_Trust(nil).nn_div([UIView new]);
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
    
    {
        //( nil / [NSNull null] )
        NSString *v =  NN_Trust(nil).nn_div([NSNull null]);
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
    
    {
        //( [NSNull null] / [NSNull null] )
        NSString *v =  [NSNull null].nn_div([NSNull null]);
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
    
    {
        //( [NSNull null] / 0 )
        NSString *v =  [NSNull null].nn_div(@(0));
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
    
    {
        //( [NSNull null] / [UIView new] )
        NSString *v =  [NSNull null].nn_div([UIView new]);
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
    
    {
        //( [NSNull null] / nil )
        NSString *v =  [NSNull null].nn_div(nil);
        NSLog(@"%@ = %@", v, v.nn_formula);
    }
```

- 示例七：计算式输出

```
    //( ( ( ( 3 - 2 + 5 ) × 3 ) × ( 10 × 2 ) ) ÷ 2 )
    NSString *v = @(3).nn_sub(@(2)).nn_add(@(5)).nn_mul(@(3)).nn_mul(@(10).nn_mul(@(2))).nn_div(@(2));
    NSLog(@"%@", v.nn_formula); //打印值：( ( ( ( 3 - 2 + 5 ) × 3 ) × ( 10 × 2 ) ) ÷ 2 )
```

## 安装

### CocoaPods

安装最新版的 CocoaPods：

```bash
$ gem install cocoapods
```

在 `podfile` 中添加：

```ruby
pod 'NNDecimalNumber', '~> 2.0.0'
```

然后在终端执行：

```bash
$ pod install
```

如安装失败，提示：

```bash
[!] Unable to find a specification for `NNDecimalNumber`
```

尝试使用命令：

```bash
pod install --repo-update
```

## 其他
Inspiration [RLArithmetic](https://github.com/RylynnLai/RLArithmetic)