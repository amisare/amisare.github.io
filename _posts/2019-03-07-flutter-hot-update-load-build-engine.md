---
layout: post
title: "Flutter你想要的热更新之编译构建 engine"
subtitle: 'Flutter你想要的热更新之编译构建 engine'
author: "顾海军"
header-style: text
tags:
  - Flutter
---

## Flutter你想要的热更新之编译构建 engine

本章目的是构建获得一个上一章节中修改后的 Flutter.framework 。

### 1. Flutter.framework

在 iOS 的 Flutter 项目中 Flutter.framework 是 Flutter 应用执行的基础。

#### 1.1 本地 Flutter.framework

通常情况下在创建 Flutter 项目不需要自己构建 Flutter.framework，因为 Flutter 环境中自带了各种模式（debug，release，profile）下需要的 Flutter.framework ，这些 framework 位于本地环境:

```
$FLUTTER_ROOT/bin/cache/artifacts/engine
```

在 iOS App 构建时，会执行 [xcode_backend.sh](https://github.com/flutter/flutter/blob/v1.0.0/packages/flutter_tools/bin/xcode_backend.sh) 根据构建类型将对应的 framework 拷贝到当前工程。[xcode_backend.sh](https://github.com/flutter/flutter/blob/v1.0.0/packages/flutter_tools/bin/xcode_backend.sh) 位于本地环境:

```
$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh。
```

#### 1.2 构建 Flutter.framework

Flutter.framework 是通过构建 engine 后的生成的。但是普通 Flutter 项目是不包含 engine 构建部分的，两者是分开的，也就是说要获得修改后的 Flutter.framework，需要先自行构建 engine，然后再将其集成到现有的 Flutter 工程中。


### 2. engine 的构建


#### 2.1 构建环境的搭建

Flutter 官方 [wiki](https://github.com/flutter/flutter/wiki/Setting-up-the-Engine-development-environment) 提供了各个平台搭建构建 engine 环境的详细文档。
因为本文主题是 Flutter 在 iOS 上的热更新，所本节主要介绍在 Mac 上 engine 构建环境的搭建及常见的问题。


##### 2.1.1 依赖工具

- git：源代码管理（ 2.x.x 以上版本，gclient 源码同步工具中使用到的部分 git 命令 1.x.x 不支持  ）。
- ssh：Mac 系统自带。
- Python：用于源码同步及构建（ 作者使用 2.7.x ）。
- unzip：解压工具。
- curl：文件下载工具。
- depot_tools：gclient 工具，安装方式：
```
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
echo 'export PATH=$PATH:"替换成本地克隆的depot_tools仓库路径"' >> ~/.bashrc
```
确认 depot_tools 环境：
```
gclient --version
```
- Xcode：最新版本的 Xcode
- IDE（可选）：开发调试用，官方推荐 [Android Studio with the Flutter plugin](https://flutter.io/using-ide/)，或者其他 IDE（Emacs，Vim，Neovim，Visual Studio Code，Nuclide） + [cquery 代码自动不全插件](https://github.com/cquery-project)。

##### 2.1.2 设置并同步构建环境

1. fork engine 仓库: 在自己 github 账户中 fork https://github.com/flutter/engine ，如果自己的 github 仓库中已有 frok 的 engine 仓库，请确保主分支与官方同步。
2. 配置 github ssh 访问: 配置自己 github 账户的 ssh 访问，确保自己的 github 账户及本地主机已经设置了 ssh 密钥。如果没有设置，参考 [generating-ssh-keys](https://help.github.com/articles/generating-ssh-keys/) 。
3. 创建 `engine` 文件夹: 创建一个空文件夹并命名为 `engine`，用来存放下载的源码和工具仓库。为了简单起见，最好不要使用其他名称。
4. 创建 `.gclient` 文件: 在 `engine` 目录下创建一个 `.gclient` 文件：，`.gclient` 文件内容如下：
    ```
    solutions = [
      {
        "managed": False,
        "name": "src/flutter",
        "url": "git@github.com:替换成你的githua账户/engine.git",
        "custom_deps": {},
        "deps_file": "DEPS",
        "safesync_url": "",
      },
    ]
    ```
5. 执行 `gclient sync`: 在 `engine` 目录中（也就是 `.gclient` 文件所在的目录），执行 `gclient sync` 。
	> - 国内 `gclient sync` 需要 vpn，这个过程取决于网络环境，时间可能会比较长，`gclient sync -v` 打印详细同步信息。
	> - 整个源码体积很大，环境同步前磁盘剩余空间最好在 20G 以上。
	> - 官文提示：命令将获取 Flutter 依赖的所有源文件。避免中断这个命令，中断命令可能会使存储仓库处于一个不稳定的状态，清理这个不稳定状态会非常繁琐。（，这个过程会自动执行 `git clone` 等操作）。**作者中断了几次也没出现什么问题。**
6. （可选）本地 engine 添加 upstream 为官方仓库: 添加 upstream 为官方仓库后，通过 `git fetch` 命令就会拉去官方最新版本代码，而不是自己 github 中 fork 的 engine 代码。在 `engine/src/flutter` 目录中执行：
	```
    git remote add upstream git@github.com:flutter/engine.git
    ```
7. （可选）安装 Oracle 的 Java JDK：用于 Android Flutter engine 构建。
8. （可选）安装 ant：用于 Android Flutter engine 构建。
	```
    brew install ant
    ```
    
##### 2.1.3 构建环境目录说明

1. engine/src：构建工具根路径，是一个 git 仓库，对应远程仓库 `https://github.com/flutter/buildroot.git` 。
2. engine/src/flutter：Flutter engine 源码仓库，对应自己 github 账户上 fork 的 engine 仓库。
	> 这就是我们要修改并构建获得 Flutter.framework 的 engine 仓库。

##### 2.1.4 构建环境搭建的常见问题

- 国内如果使用 `gclient sync` 同步构建环境失败，尝试使用 vpn ，并设置终端 http 代理。
   使用 socket 作为 http 代理时，会遇到下面的错误：
   ```
    [P18720 13:37:26.485 client.go:310 W] RPC failed transiently. Will retry in 1s    {"error":"failed to send request: Post https://chrome-infra-packages.appspot.com/prpc/cipd.Repository/ResolveVersion: dial tcp 173.252.100.21:443: i/o timeout", "host":"chrome-infra-packages.appspot.com", "method":"ResolveVersion", "service":"cipd.Repository", "sleepTime":"1s"}
   ```
   这是因为在 `gclient sync` 同步中部分工具无法通过 socket 代理下载，请参考 [用 polipo 把 shadowsocks 转为 http 代理](http://io.diveinedu.com/2016/03/16/用 Polipo 把 ShadowSocks 转为 HTTP 代理.html) 。

   安装并开启 polipo 后，设置 polipo 作为终端 http 代理
   ```
   export http_proxy=http://localhost:8123
   ```

<blockquote>
<details>
<summary><mark>用 polipo 把 shadowsocks 转为 http 代理</mark></summary>
<p></p>

#### 1. polipo 和 shadowsocks 简介

shadowsocks代理我想大家是耳熟能详了,在这里就不作过多的讲述.
不过有一点是需要指出的是,shadowsocks的1080代理端口的代理服务类型是传输层的socks代理.有些时候我们需要利用shadowsocks这个科学上网的隧道,同时需要HTTP代理的时候,我们就不得不想办法了.
我在这里就介绍另一款开源软件--`polipo`,一个小型快速的HTTP Web缓存代理服务器,还可以指定上级代理为socks类型.我们就利用这一点功能来实现我们的需求:
把shadowsocks的代理转换为http代理,让socks代理和http代理并行为我们提供科学上网服务.
这样尤其使得在unix-like系统中,很多命令行网络程序默认只能支持通过指定`http_proxy`环境变量来设置网络代理.这样利用`polipo`就把我们人人喜欢的shadowsocks服务科学上网功能扩展到http代理服务了.

#### 2. socks 转 http 代理需求和动机
这个功能在今天我更新`WebRTC`最新代码的时候,执行`gclient sync`命令的时候中间会有一部分网络下载不是通过git来更新的,所以设置git的socks代理也无效,那一部分是执行`download_from_google_storage`下载更新的,他是python的boto代码,不支持一般的代理环境变量`http_proxy`设置方法.只能在`$HOME/.boto`文件中定义HTTP代理配置.

先运行shadowsocks服务的客户端,开启科学上网功能,自动和全局无所谓.

#### 3. 安装 polipo

- Mac OS X平台安装
请确保您系统安装了brew,然后用`brew`命令来安装polipo软件包:
```bash
$ brew install polipo
```
就这样一条命令就可以安装好.

- Ubuntu 平台安装
Ubuntu Linux系统下安装方法如下:
```bash
$ apt-get install polipo
$ service polipo stop
```
安装好之后,就可以运行`polipo`并设置上级socks代理为shadowsocks端口,就成功把socks转换为http代理.如shadowsocks的本地端口一般为`localhost:1080`:
```bash
$ polipo socksParentProxy=localhost:1080
Established listening socket on port 8123.
```

#### 4. 设置 boto 配置文件

用户目录下新建文件`$HOME/.boto`,内容如下命令所示:
```bash
$ cat $HOME/.boto 
[Boto]
proxy = 127.0.0.1
proxy_port = 8123
```
`127.0.0.1:8123`是polipo命令在本地运行的HTTP代理服务默认IP端口.

#### 5. 更新 WebRTC 代码

上面配置了boto的配置文件后,我们还需要导出一个环境变量,使该配置文件可以被识别生效:

```bash
$ export NO_AUTH_BOTO_CONFIG=$HOME/.boto
```
到这个时候,我们就可以继续执行`gclient sync`来更新WebRTC的所需`chromium`代码仓库了.

如果不导出这个`NO_AUTH_BOTO_CONFIG`环境变量指向`.boto`配置文件的位置的话,就会在更新WebRTC代码的中间`download_from_google_storage `时出现如下警告提示:

```
________ running 'download_from_google_storage --no_resume --platform=darwin --no_auth --bucket chromium-gn -s src/buildtools/mac/gn.sha1' in '/Users/apple/opensource/webrtc_build/webrtc2016/src/chromium'
NOTICE: You have PROXY values set in your environment, but gsutil in depot_tools does not (yet) obey them.
Also, --no_auth prevents the normal BOTO_CONFIG environment variable from being used.
To use a proxy in this situation, please supply those settings in a .boto file pointed to by the NO_AUTH_BOTO_CONFIG environment var.
```

#### 6. 其他 polipo 转 http 代理的使用

```
http_proxy=http://localhost:8123 apt-get update
http_proxy=http://localhost:8123 curl www.google.com
http_proxy=http://localhost:8123 wget www.google.com

git config --global http.proxy 127.0.0.1:8123
git clone https://github.com/xxx/xxx.git
git config --global --unset-all http.proxy
```

</details>
</blockquote>

- 整个源码体积很大，环境同步前磁盘剩余空间最好在 20G 以上。

#### 2.2 本地构建 engine

1. 切换 engine 版本：切换到 Flutter 1.0.0 对应的 engine 版本。
    ```
    #进入 engine 仓库目录
    cd engine/src/flutter
    #切换 engine 版本
    git checkout 7375a0f414bde4bc941e623482221db2fc8c4ab5
    ```
2. 同步构建环境
	```
    #进入构建环境目录
    cd engine
    #同步构建环境
    gclient sync
    ```
3. 生成构建文件
	```
    #进入 src 目录
    cd engine/src
    #生成构建文件（分别生成真机构建文件，模拟器构建文件和服务端构建文件）
    ./flutter/tools/gn --ios --unoptimized && ./flutter/tools/gn --ios --simulator --unoptimized && ./flutter/tools/gn --unoptimized
    ```
4. 编译
	```
    #进入 src 目录
    cd engine/src
    #编译（分别编译真机文件，模拟器文件和服务端文件）
    ninja -C out/ios_debug_unopt && ninja -C out/ios_debug_sim_unopt && ninja -C out/host_debug_unopt
    ```

### 3.使用本地构建的 engine

1. 创建 Flutter App
	```
    flutter create myapp
    ```
2. 修改 myapp/pubspec.yaml 文件
	在文件末尾追加：
	```
    dependency_overrides:
    sky_engine:
      path: /path/to/flutter/engine/out/host_debug/gen/dart-pkg/sky_engine
    sky_services:
      path: /path/to/flutter/engine/out/host_debug/gen/dart-pkg/sky_services
    ```
	修改 `path:` 为 ios_debug_sim_unopt （真机为 ios_debug_unopt） 中 sky_engine 和 sky_services 的路径，如：
    ```
    dependency_overrides:
    sky_engine:
      path: /Users/guhaijun/Desktop/flutter_engine/engine/src/out/ios_debug_sim_unopt/gen/dart-pkg/sky_engine
    sky_services:
      path: /Users/guhaijun/Desktop/flutter_engine/engine/src/out/ios_debug_sim_unopt/gen/dart-pkg/sky_services
    ```
3. 设置 FLUTTER_ENGINE 环境变量
	在终端执行或修改XCode工程在 `XCode —> TARGETS -> Runner -> Build Phases -> Run Script` 的首行插入
	```
    export FLUTTER_ENGINE=/path/to/engine/src
    ```
    环境变量 FLUTTER_ENGINE 为 `engine/src` 路径，根据当前环境路径修改，如：
    ```
    export FLUTTER_ENGINE=/Users/guhaijun/Desktop/flutter_engine/engine/src
    ```
    <blockquote>
	flutter_tool 的一个 `bug` ：在使用本地 `engine` 时，`XCode` 工程缺少对 `--local-engine-src-path` 或 `$FLUTTER_ENGINE` 的获取和处理，导致 Flutter 的 `Xcode` 工程构建失败。
    错误提示：
    ```
    Launching lib/main.dart on iPhone XR in debug mode...
    Starting Xcode build...                                          
     ├─Assembling Flutter resources...                    0.7s

    Xcode build done.                                            2.7s
    Failed to build iOS app
    Error output from Xcode build:
    ↳
        ** BUILD FAILED **


    Xcode's output:
    ↳
        === BUILD TARGET Runner OF PROJECT Runner WITH CONFIGURATION Debug ===
        Unable to detect local Flutter engine build directory.
        Either specify a dependency_override for the sky_engine package in your pubspec.yaml and ensure --package-root is set if necessary,
        or set the $FLUTTER_ENGINE environment variable, or use --local-engine-src-path to specify the path to the root of your
        flutter/engine repository.
        Failed to package /Users/guhaijun/Desktop/flutter/engine_test/myapp.
        Command /bin/sh failed with exit code 255

    Could not build the application for the simulator.
    Error launching application on iPhone XR.
     2.0s
    ```
    </blockquote>
4. 运行 Flutter App
	```
    flutter run --local-engine-src-path=/path/to/engine/src  --local-engine=ios_debug_sim_unopt
    ```
    --local-engine-src-path 为 `engine/src` 路径，根据当前环境路径修改，如：
	```
    flutter run --local-engine-src-path=/Users/guhaijun/Desktop/flutter_engine/engine/src  --local-engine=ios_debug_sim_unopt
    ```






















