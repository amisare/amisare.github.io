---
layout: post
title: "NNPopObjc：在 Objective-C 上进行面向协议的编程（下）"
subtitle: 'NNPopObjc：在 Objective-C 上进行面向协议的编程'
author: "顾海军"
header-style: text
tags:
  - Objective-C
---

在上半部分主要介绍了  [NNPopObjc](https://github.com/amisare/NNPopObjc) 的使用，包括默认协议扩展、约束协议扩展等。本文 (下) 主要介绍 **NNPopObjc** 的实现思路和原理。

# 神奇的宏

在 **NNPopObjc** 中使用宏实现了关键字`@nn_extension(...)`, `@nn_where(...)` 。但限于篇幅，这里我们不去详细的讲解这些宏是如何实现的。对于本节我们会解释[元编程](https://baike.baidu.com/item/元编程/6846171?fr=aladdin)的概念及其编程思维。一旦对**元编程**及其编程思维有了一定的认识，那么再去分析 **NNPopObjc** 中的宏实现就会变的非常简单了。


## 元编程

宏编程也被称为 C 语言系中的**元编程**，可以简单理解为代码作为函数的输入和输出。在 **NNPopObjc** 中使用宏实现了关键字`@nn_extension(...)`, `@nn_where(...)` ，为实现以上关键字， **NNPopObjc**  使用了大量的宏作为中间转换。这些宏大多基于 [metamacros.h](https://github.com/jspahrsummers/libextobjc/blob/master/extobjc/metamacros.h) 的实现或扩展。

## 元编程的思维

**NNPopObjc** 中使用了很多宏特性。本节选择具有代表性实现的宏函数 `nn_pop_if_less(A, B)` 作为讲解示例。

为了方便理解，这里我们对 `nn_pop_if_less(A, B)` 的定义做一些简化，定义如下：

- `A`，`B` 取值范围 [0..2]
- 当 `A < B` 时输出 `A`，否则输出 `B`

### 用元编程的思维去思考和实现：

使用元编程去实现，那么结果的输出应该在编译器阶段完成的，也就是说，元编程中宏函数的结果不需要应用程序的执行。下面定义虽然也能实现，但是真正的结果是由应用程序执行阶段获得的，因此并不是我们期望的元编程实现。

```
#define nn_pop_if_less(A, B)    (A < B ? A : B)
```

### 用元编程的思维去实现 `nn_pop_if_less(A, B)` ：

以下给出 `nn_pop_if_less(A, B)`  元编程的实现：

```
#define nn_pop_if_less(A, B) \
        nn_pop_if_less_(A, B)(A)(B)
    
#define nn_pop_if_less_(A, B) \
        nn_pop_if_less_##A##_##B

#define nn_pop_if_less_0_0(A) nn_pop_expand_
#define nn_pop_if_less_0_1(A) A nn_pop_consume_
#define nn_pop_if_less_0_2(A) A nn_pop_consume_
#define nn_pop_if_less_1_0(A) nn_pop_expand_
#define nn_pop_if_less_1_1(A) nn_pop_expand_
#define nn_pop_if_less_1_2(A) A nn_pop_consume_
#define nn_pop_if_less_2_0(A) nn_pop_expand_
#define nn_pop_if_less_2_1(A) nn_pop_expand_
#define nn_pop_if_less_2_2(A) nn_pop_expand_

#define nn_pop_expand_(B)   B
#define nn_pop_consume_(B)
```

### 分析实现

这里我们通过两个示例，使用类似数学推导的方式来分析 `nn_pop_if_less(A, B)` 的元编程实现。 每一步的推导代表一次宏替换。

示例 1  ：
`nn_pop_if_less(1, 2) = 1` 
```
  nn_pop_if_less(1, 2)
= nn_pop_if_less_(1, 2)(1)(2)
= nn_pop_if_less_##1##_##2(1)(2)
= nn_pop_if_less_1_2(1)(2)
= 1 nn_pop_consume_(2)
= 1
```

示例 2 ：
`nn_pop_if_less(1, 0) = 0` 
```
  nn_pop_if_less(1, 0)
= nn_pop_if_less_(1, 0)(1)(0)
= nn_pop_if_less_##1##_##0(1)(0)
= nn_pop_if_less_1_0(1)(0)
= nn_pop_expand_(0)
= 0
```

通过对上述两个示例的推导，可以看出宏函数的结果由编译器分析获得，获得的结果不需要应用程序的执行，那么上述的实现也就是我们所期望的元编程实现。

### 优化和改进

在上面的实现中，你也许会发现，宏函数的比较最终由一系列的 `nn_pop_if_less_A_B ` 扩展提供。那，如果参数的范围是 [0..20] 或者更多那？排列组合后的结果不敢想象。针对这个问题，[metamacros.h](https://github.com/jspahrsummers/libextobjc/blob/master/extobjc/metamacros.h) 提供了非常棒的实现方式。具体实现可参考宏函数 `metamacro_if_eq(A, B)` 。

可惜的是 **metamacros.h** 仅提供了 `metamacro_if_eq(A, B)` 用于`判等`的宏函数，而未提供其他判断条件的宏函数实现。为了方便开发，作者在 **NNPopObjc**  中对其进行了扩展并提供了全条件判断宏函数。

```
nn_pop_if_equal(A, B)
nn_pop_if_greater(A, B)
nn_pop_if_greater_or_equal(A, B)
nn_pop_if_less(A, B)
nn_pop_if_less_or_equal(A, B)
```

## 小结

本节介绍了一个非常重要的编程概念**元编程**。那么在理解了**元编程**之后，对于 **NNPopObjc** 中关键字 `@nn_extension(...)`, `@nn_where(...)` 的实现和理解就变的非常容易了。

# 最好的时机

凡是要对语言进行扩展的框架，大多数逃不出运行时的应用。同样为了实现协议的扩展， **NNPopObjc** 的注入实现也基于运行时。

在定义协议扩展时， **NNPopObjc** 会定义一个类作为协议扩展实现的容器类，然后通过运行时将容器类中的方法注入到遵守协议类中。

## 注入时机

使用程序注入，便会产生一个问题：在哪个时机进行注入。本节我们将对这个问题进行讨论。

### 动态方法解析/消息转发

在 main() 函数之后，**动态方法解析/消息转发**可以作为一个注入的时机。例如我们非常熟悉的在 Objective-C 中实现面向切面编程（AOP）的 [Aspects](https://github.com/steipete/Aspects)，一段时间非常热门的 iOS 动态热修复框架 [JSPatch](https://github.com/search?q=JSPatch) 。以上框架中的注入时机都选择在了**动态方法解析/消息转发**。

**NNPopObjc** 在 **0.5.0** 及以前的版本就采用了在**动态方法解析/消息转发**进行注入。如果选择**动态方法解析/消息转发**作为 **NNPopObjc** 的注入时机，那么程序就会有以下特点：

- 按需注入：由于在**遵守协议类**没有协议的实际实现，所以在调用未实现的协议方法时，就会触发 **动态方法解析/消息转发** 。在此时刻进行注入，我们可以仅对需要的方法进行注入，因此可以避免全量注入而可能引起的性能问题。
- 兼容性问题：期望对代码无侵入，就必须要 hook **动态方法解析/消息转发**期间的函数。虽然实现了对代码的无侵入，但如果其他代码也 hook 这些函数，就可能会导致函数调用混乱。比如本节开头提到的 **Aspects** 和 **JSPatch**，两个框架同时使用是就会存在兼容性问题。

### + load()

在 main() 函数之前，+ load() 可以作为一个注入的时机。也常被广大开发者做为方法交换，方法注入的选择。

使用在 + load() 中实现注入，程序就会有以下特点：

- 占用默认 + load() 方法：使用 + load() 方法作为注入时机，一定会占用一个已有的 + load() 。
  1. 协议扩展类 + load()：会导致开发者无法为扩展类自定义 + load() 方法。
  2. 遵守协议类 + load()：会导致严重的代码侵入。
- 性能问题：避免代码入侵，使用协议扩展类 + load() 作为注入时机。那么，在一个协议实现了多个扩展的情况下，为了实现注入，每个协议扩展的 + load() 都需要遍历一遍类列表。这样无疑会增加 + load() 耗时，影响应用启动。
- 时序问题：由于 + load() 方法的调用顺序是变化的，如果类或协议存在多个继承关系就可能会导致注入结果与期望的不同。

### \_\_attribute\_\_((constructor))

在 main() 函数之前，被 \_\_attribute\_\_((constructor)) 修饰的函数可以作为一个注入的时机。例如我们非常熟悉的在函数 hook 框架  [fishhook](https://github.com/facebook/fishhook)，阿里开源协程开发框架  [coobjc](https://github.com/alibaba/coobjc) ，都选择了在 \_\_attribute\_\_((constructor)) 函数位置作为注入时机。

使用在 \_\_attribute\_\_((constructor)) 函数中实现注入，程序就会有以下特点：

- 对遵守协议类 + load() 的影响：\_\_attribute\_\_((constructor)) 函数的执行在所有 + load() 方法之后，main() 函数之前。如果想在遵守协议类中对协议扩展的方法进行交换（MethodSwizz）是无法实现的，因为在遵守协议类的 + load() 中协议扩展的方法还未被注入，此时的方法实现并不存在。

## 初识 \_\_attribute\_\_((constructor)) 函数

来自 [GCC](https://gcc.gnu.org/onlinedocs/gcc-6.2.0/gcc/Common-Function-Attributes.html) 上的一些描述：

> constructor
> destructor
> constructor (priority)
> destructor (priority)
> The constructor attribute causes the function to be called automatically before execution enters main (). Similarly, the destructor attribute causes the function to be called automatically after main () completes or exit () is called. Functions with these attributes are useful for initializing data that is used implicitly during the execution of the program.
You may provide an optional integer priority to control the order in which constructor and destructor functions are run. A constructor with a smaller priority number runs before a constructor with a larger priority number; the opposite relationship holds for destructors. So, if you have a constructor that allocates a resource and a destructor that deallocates the same resource, both functions typically have the same priority. The priorities for constructor and destructor functions are the same as those specified for namespace-scope C++ objects (see C++ Attributes).
> 
> These attributes are not currently implemented for Objective-C. 
> 

嗯 …… ，最后一句是针对 GCC 的描述，对于 LLVM 和 Clang 可以忽略。

取其中有用的内容：

> The constructor attribute causes the function to be called automatically before execution enters main ().

构造属性的函数会在进入 main () 之前被自动调用。

那函数到底是在什么位置被调用那？在 XCode 中 \_\_attribute\_\_((constructor)) 函数中增加断点，可以得到如下函数调用栈：

```
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 2.1
  * frame #0: 0x00000001027cd83c NNPopObjc`popobjc::initializer(argc=1, argv=0x00007ffeed560da8, envp=0x00007ffeed560db8, apple=0x00007ffeed560fa0, vars=0x0000000102707170) at NNPopObjcInjection.mm:377:53
    frame #1: 0x00000001026c43a7 dyld_sim`ImageLoaderMachO::doModInitFunctions(ImageLoader::LinkContext const&) + 517
    frame #2: 0x00000001026c47b8 dyld_sim`ImageLoaderMachO::doInitialization(ImageLoader::LinkContext const&) + 40
    frame #3: 0x00000001026bf9a2 dyld_sim`ImageLoader::recursiveInitialization(ImageLoader::LinkContext const&, unsigned int, char const*, ImageLoader::InitializerTimingList&, ImageLoader::UninitedUpwards&) + 456
    frame #4: 0x00000001026bf90f dyld_sim`ImageLoader::recursiveInitialization(ImageLoader::LinkContext const&, unsigned int, char const*, ImageLoader::InitializerTimingList&, ImageLoader::UninitedUpwards&) + 309
    frame #5: 0x00000001026be7a6 dyld_sim`ImageLoader::processInitializers(ImageLoader::LinkContext const&, unsigned int, ImageLoader::InitializerTimingList&, ImageLoader::UninitedUpwards&) + 188
    frame #6: 0x00000001026be846 dyld_sim`ImageLoader::runInitializers(ImageLoader::LinkContext const&, ImageLoader::InitializerTimingList&) + 82
    frame #7: 0x00000001026b308c dyld_sim`dyld::initializeMainExecutable() + 199
    frame #8: 0x00000001026b70fc dyld_sim`dyld::_main(macho_header const*, unsigned long, int, char const**, char const**, char const**, unsigned long*) + 3831
    frame #9: 0x00000001026b21cd dyld_sim`start_sim + 122
    frame #10: 0x0000000105b028b7 dyld`dyld::useSimulatorDyld(int, macho_header const*, char const*, int, char const**, char const**, char const**, unsigned long*, unsigned long*) + 2308
    frame #11: 0x0000000105b00575 dyld`dyld::_main(macho_header const*, unsigned long, int, char const**, char const**, char const**, unsigned long*) + 818
    frame #12: 0x0000000105afb227 dyld`dyldbootstrap::start(dyld3::MachOLoaded const*, int, char const**, dyld3::MachOLoaded const*, unsigned long*) + 453
    frame #13: 0x0000000105afb025 dyld`_dyld_start + 37
```

其中 popobjc::initializer 是我们定义的 \_\_attribute\_\_((constructor)) 函数。
显然  \_\_attribute\_\_((constructor)) 函数由 dyld 中的 ImageLoaderMachO::doModInitFunctions 调用。dyld 是开源的，源码可以在 [dyld](https://opensource.apple.com/tarballs/dyld/) 下载，**NNPopObjc**中**获取线索**小节中也有对 dyld 源码的参考。对于更多 Image 加载过程的相关内容可以参考 dyld 源码。

## 小结

到此为止，本节介绍了常见的注入时机。这里 **NNPopObjc** 在0.6.0及以后的版本中也选择了 \_\_attribute\_\_((constructor)) 函数作为项目的注入时机。

# 留下线索

要实现对遵守协议类的方法注入，就必须获取以下信息：
- 协议
- 协议扩展类（协议扩展实现的容器类）
- 遵守协议类

在 **NNPopObjc** 的实现中，**协议**和**协议扩展类**的信息在 `@nn_extension` 对协议进行扩展时被保存到了**数据段**（data segment）中， 之后在 \_\_attribute\_\_((constructor)) 函数注入时，从**数据段**获得**协议**和**协议扩展类**的信息。

## 数据段/分段

[数据段（data segment）](https://baike.baidu.com/item/数据段/5136260?fr=aladdin)通常是指用来存放程序中已初始化的全局变量的一块内存区域。数据段属于静态内存分配 —— 百度百科。

分段（section）一个段包含多个分段。

### 定义一个变量到指定的数据段分段中

```
struct duart a __attribute__ ((used, section ("__DATA", "DUART_A"))) = { 0 };
```

- used：避免未被使用的段被编译器优化移除
- section：描述变量保存的段描述
- "__DATA"：描述变量保存的段为数据段
- "DUART_A"：描述变量保存到数据段中名为 "DUART_A" 的分段

## 保存注入信息到数据段

### **@nn_extension(...)** 的展开

下面是对 NNCodeProtocol 实现一个协议扩展
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

对上面的协议扩展的宏定义部分进行展开

```
@class NSObject;

static nn_pop_where_value_def w___NNPopObjc_NNCodeProtocol___NNCodeNameProtocol(Class self) {
    @autoreleasepool {}
    return ^nn_pop_where_value_def(__attribute__((objc_ownership(none))) Class self){
        if (self == ((void *)0)) {
            return nn_pop_where_value_unmatched;
            
        } BOOL
        is_match = (self = [NNCodeC class]);
        if (is_match == 0) {
            return nn_pop_where_value_unmatched;
        }
        return 0 ? nn_pop_where_value_matched_default : nn_pop_where_value_matched_constrained;
    }(self);
}

const nn_pop_extension_description_t s___NNPopObjc_NNCodeProtocol___NNCodeNameProtocol __attribute__((used, section("__DATA" "," "__nn_pop_objc__" ))) = {
    "NNCodeProtocol",
    "__NNPopObjc",
    "__NNPopObjc_NNCodeProtocol___NNCodeNameProtocol",
    w___NNPopObjc_NNCodeProtocol___NNCodeNameProtocol,
    1,
    {"NNCodeNameProtocol",},
};

@interface __NNPopObjc_NNCodeProtocol___NNCodeNameProtocol : NSObject < NNCodeProtocol ,NNCodeNameProtocol>

@end

@implementation __NNPopObjc_NNCodeProtocol___NNCodeNameProtocol

+ (void)sayHelloPop {
    printf("%s\n", [[NSString stringWithFormat:@"+[%@ %s] code says hello pop", self, sel_getName(_cmd)] UTF8String]);;
}

- (void)sayHelloPop {
    printf("%s\n", [[NSString stringWithFormat:@"-[%@ %s] code says hello pop", [self class], sel_getName(_cmd)] UTF8String]);;
}

@end
```

在展开中我们能够发现以下代码：

```
const nn_pop_extension_description_t s___NNPopObjc_NNCodeProtocol___NNCodeNameProtocol __attribute__((used, section("__DATA" "," "__nn_pop_objc__" ))) = {
    "NNCodeProtocol",
    "__NNPopObjc",
    "__NNPopObjc_NNCodeProtocol___NNCodeNameProtocol",
    w___NNPopObjc_NNCodeProtocol___NNCodeNameProtocol,
    1,
    {"NNCodeNameProtocol",},
};
```

在 NNCodeProtocol 的协议扩展中程序将一个名为 `s___NNPopObjc_NNCodeProtocol___NNCodeNameProtocol` 的结构体变量保存到了名为 `__nn_pop_objc__ ` 的数据段分段中。在 **NNPopObjc** 中所有协议扩展的数据都会保存在 `__nn_pop_objc__ ` 分段中。

### **nn_pop_extension_description_t** 结构体

在 **NNPopObjc** 中 `nn_pop_extension_description_t` 结构体保存了所有协议扩展注入时需要的信息。
下面是对 `nn_pop_extension_description_t` 结构体各字段的描述。

```
typedef struct {
    /// 协议名称
    const char *protocol;
    /// 协议扩展类前缀
    const char *prefix;
    /// 协议扩展类名称
    const char *clazz;
    /// 执行 where 表达式的函数指针
    where_fp where_fp;
    /// 遵守协议类需要满足的协议数量
    unsigned int confrom_protocols_count;
    /// 遵守协议类需要满足的协议列表
    const char *confrom_protocols[20];
} nn_pop_extension_description_t;
```

## 小结

在 **NNPopObjc** 中协议扩展注入的信息由结构体 `nn_pop_extension_description_t` 描述，注入信息的结构体变量被保存到了名为 `__nn_pop_objc__` 的数据段分段中。

# 获取线索

只要在函数中读取名为 `__nn_pop_objc__ ` 的数据段分段，就能够进行协议扩展注入了。

## **getsectbyname** 与 **getsectiondata**

获取分段数据可以通过 **getsectbyname** 和 **getsectiondata** 获取，但是在苹果系统中由于 `ASLR ` ，获取数据段分段信息只能使用函数 **getsectiondata** 。参考 [crash-reading-bytes-from-getsectbyname](https://stackoverflow.com/questions/28978788/crash-reading-bytes-from-getsectbyname)。

### **mach_header**

**getsectiondata** 接口定义

```
extern uint8_t *getsectiondata(
    const struct mach_header *mhp,
    const char *segname,
    const char *sectname,
    unsigned long *size);
```

这里需要一个 **mach_header** 结构体参数。**mach_header** 对 32 位和 64 为系统分别进行了定义，但有效字段是一致的。

以下是 32 mach_header 结构的描述：

> The 32-bit mach header appears at the very beginning of the object file for 32-bit architectures.

mach_header 保存在对象文件的头部，**mach_header** 结构体描述请参考头文件 `<mach/loader.h>` 。

## 获取 **mach_header**

### 通过 \_\_attribute\_\_((constructor)) 函数获取

\_\_attribute\_\_((constructor)) 函数是一种函数回调，但是回调函数的格式缺并没有给出。这里我们就要参考文中提到的 dyld 源码。
**最好的时机** 章节中提到 \_\_attribute\_\_((constructor)) 函数最终由 dyld 中 `ImageLoaderMachO::doModInitFunctions` 方法调用。

在 `ImageLoaderMachO::doModInitFunctions` 方法中我们会发现被调用的初始化函数为 `Initializer` 类型的函数，`Initializer` 定义如下：

```
struct ProgramVars
{
	const void*		mh;
	int*			NXArgcPtr;
	const char***	NXArgvPtr;
	const char***	environPtr;
	const char**	__prognamePtr;
};
typedef void (*Initializer)(int argc, const char* argv[], const char* envp[], const char* apple[], const ProgramVars* vars);
```

`ProgramVars ` 结构体中 `const void*		mh;` 即是我们需要的 **mach_header** 。根据 dyld 中定义的函数定义，实现 \_\_attribute\_\_((constructor)) 函数，如下：

```
typedef struct
#ifdef __LP64__
mach_header_64
#else
mach_header
#endif
nn_pop_mach_header;

struct ProgramVars {
    const void*        mh;
    int*            NXArgcPtr;
    const char***    NXArgvPtr;
    const char***    environPtr;
    const char**    __prognamePtr;
};

__attribute__((constructor)) void initializer(int argc,
                                              const char **argv,
                                              const char **envp,
                                              const char **apple,
                                              const ProgramVars* vars) {
    
    nn_pop_mach_header *mhp = (nn_pop_mach_header *)vars->mh;
    
    loadSection(mhp,
                nn_pop_metamacro_stringify(nn_pop_section_name),
                [](std::vector<ProtocolExtension *> protocolExtensions) {
        ......
    });
}
```

这样我们就能够在 \_\_attribute\_\_((constructor)) 函数中得到 **mach_header** 变量了。

但是，这里需要注意的是，\_\_attribute\_\_((constructor)) 函数的调用是所在 Mach-O 文件加载 doModInit 时调用。也就是说这里得到的 **mach_header** 是当前加载的 Mach-O 文件的 mhp 。那么就可能会影响 NNPopObjc 中 `__nn_pop_objc__` section 的加载：

1. NNPopObjc 作为动态库集成：得到的 **mach_header** 为 NNPopObjc 动态库的 Mach-O 文件的 mhp，只能加载 NNPopObjc 动态库 Mach-O 的 `__nn_pop_objc__` section 。
2. NNPopObjc 作为静态库集成：
    - NNPopObjc 中包含 OC 对象：得到的 **mach_header** 为最终连接的 Mach-O 文件的 mhp，只能加载最终连接 Mach-O 的 `__nn_pop_objc__` section 。
    - NNPopObjc 中不包含 OC 对象：\_\_attribute\_\_((constructor)) 不会被调用。
    

### 通过 **_dyld_register_func_for_add_image** 获取

通过 `_dyld_register_func_for_add_image` 注册回调函数获得 **mach_header** 。

```
/// Image loaded callback function.
/// @param mhp mhp
/// @param vmaddr_slide vmaddr_slide
void imageLoadedCallback(const struct mach_header *mhp, intptr_t vmaddr_slide) {
    
    nn_pop_mach_header *_mhp = (nn_pop_mach_header *)mhp;
    
    loadSection(mhp,
                nn_pop_metamacro_stringify(nn_pop_section_name),
                [](std::vector<ProtocolExtension *> protocolExtensions) {
        ......
    });
}

/// Initializer function is called by ImageLoaderMachO::doModInitFunctions at dyld project.
/// @note dyld project: https://opensource.apple.com/tarballs/dyld/
/// @note fix: The dynamic library section cannot be loaded when the protocol extensions
/// are implemented in a dynamic library.
__attribute__((constructor)) void initializer() {
    
    _dyld_register_func_for_add_image(imageLoadedCallback);
}
```

这里 `imageLoadedCallback` 回调函数会被调用多次，每次回调中的 mhp 参数对应一个 Mach-O 文件 。

### 其他方式

关于 **mach_header** 的一些其它获取方式可参考 <mach-o/dyld.h> 中相关函数。

## 小结

在 **NNPopObjc** 中使用 **_dyld_register_func_for_add_image** 注册回调的方式依次获取所有 **mach_header** 变量，并通过 `getsectiondata` 函数尝试读取 **mach_header** 对应 Mach-O 文件保存在 `__nn_pop_objc__ ` 数据段分段中用于注入的信息，最后进行扩展注入。

# 协议扩展注入

关于协议扩展注入这里就不做过多的介绍了。基于类列表查找，对与遵守协议且符合协议扩展条件的类进行方法注入即可。

# 结

**NNPopObjc** 为在 Objective-C 上进行面向协议的编程提供了可能。在面向协议的编程中，协议可以拥有自己的行为，使得程序减少类和继承带来的负面问题。让程序更加灵活和便于维护。

# 致谢与参考

**NNPopObjc** 思路和实现离不开开源社区的力量，在此由衷感谢！

以下是 **NNPopObjc** 实现中参考的项目及资料。

## 项目

- 相似项目

[libextobjc](https://github.com/jspahrsummers/libextobjc)
[ProtocolKit](https://github.com/forkingdog/ProtocolKit)

- 其他项目

[Aspects](https://github.com/steipete/Aspects)
[JSPatch](https://github.com/search?q=JSPatch)
[fishhook](https://github.com/facebook/fishhook)
[coobjc](https://github.com/alibaba/coobjc)

## 文章

- [面向协议编程与 Cocoa 的邂逅 (上)](https://onevcat.com/2016/11/pop-cocoa-1/) - 王巍
- [面向协议编程与 Cocoa 的邂逅 (下)](https://onevcat.com/2016/12/pop-cocoa-2/) - 王巍
- [Protocol-Oriented Programming in Swift](https://developer.apple.com/videos/play/wwdc2015/408/) - WWDC 2015 #Session 408
- [Protocol and Value Oriented Programming in UIKit Apps](https://developer.apple.com/videos/play/wwdc2016/419) - WWDC 2016 #Session 419
- [Practical Protocol-Oriented-Programming](https://academy.realm.io/posts/appbuilders-natasha-muraschev-practical-protocol-oriented-programming/) - Natasha Murashev
