## 模拟器or真机
#### 一句话版
**用真机**
#### 详解
真机需要购置设备，需要root  
而模拟器直接PC装上就能用，基本直接带root，PC与模拟器互相操作也方便  
但是经实测，模拟器对逆向、调试等极不友好，原因如下   
  
主流游戏都会选择用Native代码(C++)实现核心功能，这样运行效率高并且有一定的破解门槛  
一般是将Native代码编译成库，用安卓常规开发方式例如Java作为壳，调用Native库  
编译好的Native库是与平台相关的，由于市场现状，市面上的apk基本上只提供arm指令集的库  
PC是x86架构。为了运行效率，主流模拟器都是基于硬件虚拟化的x86指令集的安卓  
也就是直接在x86硬件上跑x86指令，而不是走完全的软件模拟（实测纯软件模拟的模拟器效率极低，基本等几个小时都开不了机）  
为了能在x86机器上跑arm指令，主流模拟器基本上都是将arm翻译成x86再执行，详情可搜libhoudini  
这样就引入了额外的复杂度，现有的逆向工具对PC上开硬件虚拟化的安卓模拟器，模拟器里跑由arm转换成x86的指令，这种复杂情况处理的不是很好  
经实测CheatEngine、Ghidra、IDA pro等工具，在模拟器下都出现了各种各样的问题，包括但不限于无法下断点、无法读取寄存器、无法分析内存等  
但对于简单场景，例如只想开CheatEngine改一下数据，模拟器是足够的  
任何较为正式的逆向行为，都推荐直接上真机  

## 真机root
偷懒可以直接上网买root好的  
也可以买小米手机，通过官网解锁BootLoader，刷入Magisk，授权root  
可参考<https://miuiver.com/how-to-root-xiaomi-phone/>  
据说一加手机也很方便，没有试过  

## 模拟器与WSL不兼容
windows配WSL是常见组合  
WSL与主流模拟器都使用hyper-v，不知道什么原因不能同时存在  
解决方法是换一个和hyper-v兼容的模拟器  
不走硬件虚拟化的纯软件模拟器当然可以，但是慢到不能用  
Bluestacks Nougat 64bit，兼容hyper-v，但root比较麻烦  
推荐使用Bluestacks Nougat 64bit，再配合Bluestacks Tweaker开root  
可以直接从Bluestacks Tweaker官网找支持的Bluestacks版本，例如4.28  