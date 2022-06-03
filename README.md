# 静态分析
### 获取apk
一般游戏官网可直接下载android apk。也可从应用商店爬取，例如[google play](https://apps.evozi.com/apk-downloader/)。
### Android Killer
[Android Killer](https://github.com/liaojack8/AndroidKiller) 集成了Apk反编译、Apk打包等功能。
### JADX
[JADX](https://github.com/skylot/jadx) Apk反编译，可得Java源码。
### IDA pro
不知道如何描述了。这里主要用来反编译apk里的so文件。收费。
### Ghidra
[Ghidra](https://github.com/NationalSecurityAgency/ghidra) 功能与IDA相近，开源免费


# 动态调试
### 用真机而不是模拟器
主流游戏都会选择用Native代码(C++)实现核心功能，这样运行效率高并且有一定的破解门槛。一般是将Native代码编译成库，用安卓常规开发方式例如Java作为壳，调用Native库。编译好的Native库是平台相关的，市面上的apk基本上只提供arm指令集的库。

PC是x86架构。为了运行效率，主流模拟器都使用了硬件虚拟化技术，也就是直接在x86硬件上跑x86指令，而不是走纯软件模拟（实测纯软件模拟效率极低，基本一个小时都开不了机）。为了能在x86机器上跑arm指令，基本上都是将arm翻译成x86再执行（详情搜索libhoudini）。

现有的工具对 *PC上开硬件虚拟化跑安卓模拟器，模拟器里跑由arm转换成x86的指令* 这种复杂情况处理的不是很好。经实测，CheatEngine、Ghidra、IDA pro等工具，在模拟器下都出现了各种各样的问题，包括但不限于无法下断点、无法读取寄存器、无法分析内存等。但对于简单场景，例如只想开CheatEngine改一下数据，模拟器是足够的。复杂情况都推荐上真机。
### 真机root  
可以直接上网买root好的。  
也可以买小米手机，[通过官网解锁BootLoader，刷入Magisk，授权root](https://miuiver.com/how-to-root-xiaomi-phone/) 。  
据说一加手机也很方便，没有试过。

### 模拟器与WSL不兼容  
WSL与主流模拟器都使用hyper-v，不知道什么原因不能同时存在。解决方法是换一个和hyper-v兼容的模拟器。  
推荐Bluestacks Nougat 64bit 配合Bluestacks Tweaker开root。  
