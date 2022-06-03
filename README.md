# 静态分析
### 获取apk
一般游戏官网可直接下载android apk。也可从应用商店爬取，例如[google play](https://apps.evozi.com/apk-downloader/)。
### Android Killer
[Android Killer](https://github.com/liaojack8/AndroidKiller) 集成了Apk反编译、Apk打包等功能。
### JADX
[JADX](https://github.com/skylot/jadx) Apk反编译，可得Java源码。
### IDA Pro
不知道如何描述了。这里主要用来反编译apk里的so文件。收费。
### Ghidra
[Ghidra](https://github.com/NationalSecurityAgency/ghidra) 功能与IDA相近，开源免费
### 实际操作
* 获取apk
* 通过Android Killer与JADX解包、反编译
* 查看AndroidManifest.xml得到MainActivity
* 从MainActivity入手查看Java代码，通常关键部分最后会调用到Native库
* 从解包出来的lib文件夹下，应该能找到.so库，通过IDA Pro或Ghidra反编译
* 被外部调用的函数是要导出的，可以从函数表、导出表或者全局搜索，直接找到对应名字的函数


# 动态调试
### 用真机而不是模拟器
主流游戏都会选择用Native代码(C++)实现核心功能，这样运行效率高并且有一定的破解门槛。一般是将Native代码编译成库，用安卓常规开发方式作为壳，调用Native库。编译好的Native库是平台相关的，市面上的apk基本上只提供arm指令集的库。

PC是x86架构。为了运行效率，主流模拟器都使用了硬件虚拟化技术，也就是直接在x86硬件上跑x86指令，而不是走纯软件模拟（实测纯软件模拟效率极低，基本一个小时都开不了机）。为了能在x86机器上跑arm指令，基本上都是将arm翻译成x86再执行（详情搜索libhoudini）。

现有的工具对 *PC上开硬件虚拟化跑安卓模拟器，模拟器里跑由arm转换成x86的指令* 这种复杂情况处理的不是很好。经实测，CheatEngine、Ghidra、IDA pro等工具，在模拟器下都出现了各种各样的问题，包括但不限于无法下断点、无法读取寄存器、无法分析内存等。但对于简单场景，例如只想开CheatEngine改一下数据，模拟器是足够的。复杂情况都推荐上真机。

### 真机root  
可以直接上网买root好的。  
也可以买小米手机，[通过官网解锁BootLoader，刷入Magisk，授权root](https://miuiver.com/how-to-root-xiaomi-phone/) 。  
据说一加手机也很方便，没有试过。

### 模拟器与WSL不兼容  
WSL与主流模拟器都使用hyper-v，不知道什么原因不能同时存在。解决方法是换一个和hyper-v兼容的模拟器。  
推荐Bluestacks Nougat 64bit 配合Bluestacks Tweaker开root。  

### Apk改debuggable
据说debuggable为false时无法调试。实测debuggable为false时，IDA Pro仍可调试，但无法am start -D启动时等待调试器。建议改了避免有奇怪的问题发生。  

参考[令App可调试的几种方法](https://www.cnblogs.com/lsgxeva/p/13490991.html)  

推荐Android Killer打开AndroidManifest.xml，application加上android:debuggable="true"，点Compile

### IDA Pro动态调试
```
adb push <IDA Pro>/dbgsrv /data/local/tmp # 将IDA debug server传到手机，一般用dbgsrv/android_server
adb forward tcp:23946 tcp:23946 # dbgsrv的默认端口
adb shell
su
/data/local/tmp/dgbsrv/android_server # 手机上起debug server
# 起server后，开游戏，IDA -> Debugger -> Attach to process

# 如果需要app启动就暂停，等调试器挂上了再继续运行
am start -D package/activity
# 此时目标app会等待debugger，此时将IDA attach上

# 再host执行
adb jdwp # 会列出所有可debug进程，里面理应有目标进程，这一步无所谓
adb shell pidof package # 打印pid
adb forward tcp:7777 jdwp:pid # 上面的pid
jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=7777
# host这一步做完以后，目标app等待debugger的就算等完了，剩下全看IDA debugger的操作
# 最好jdb连上，目标app取消等待了以后，给jdb关了，避免jdb将app暂停，或者其他冲突
```

### Ghidra动态调试
Ghidra动态调试很多坑。  
* windows下不能起本地gdb client，需要ssh到linux下开gdb client，再连到gdb server
* Ghidra的ssh client实现有问题，连不上默认配置的ssh server，需要调ssh server配置
* Ghidra调试非常不稳定，经常卡死，基本没顺畅调试成功过  

这里仍然列出步骤以供参考
##### WSL配置openssh
```
sudo apt install openssh-server

# ghidra的ssh client太旧了，需要改一下ssh server的配置，要不然连不上
# 参考https://github.com/NationalSecurityAgency/ghidra/discussions/3487
vim /etc/ssh/sshd_config
添加以下几行
AllowUsers
Port 8888 # 这是ssh端口号
KexAlgorithms +diffie-hellman-group1-sha1
Ciphers +aes128-cbc
PasswordAuthentication yes
ChallengeResponseAuthentication yes
UsePAM yes
X11Forwarding yes
AcceptEnv LANG LC_*

# 起ssh server
# WSL没有systemctl，有service
# 每次开WSL都要起一下，或者加到自启动
sudo service ssh start
```
##### 起游戏并attach
```
adb push <NDK>/prebuilt/android-arm/gdbserver /data/local/tmp # gdbserver

# adb forward只listen了127.0.0.1
# 后续从WSL里连不上gdbserver
# 需要让adb forward listen所有地址
# 参考https://stackoverflow.com/questions/56130335/adb-port-forwarding-to-listen-on-all-interfaces
# nodaemon会block，后续得另开一个窗口做其他操作
# 去掉nodaemon就正常返回
# 但实测必须nodaemon
# 如果adb server已经起了，需要adb kill-server
adb -a nodaemon server start
adb -a forward tcp:9090 tcp:9090

adb shell
su
am start xxx
/data/local/tmp/gdbserver/gdbserver :9090 --attach pid
```
##### Ghidra调试
开Ghidra -> Debug Tool -> (Menu) Debugger -> Debug xxx -> in GDB via ssh  
GDB launch command填gdb-multiarch，而不是gdb，因为gdb server是arm的    
其他正常填ssh server的  

连接完成后去右侧gdb窗口 target remote ip:port  
注意此时是在ssh里，ip要填host的ip，而不是127.0.0.1  
