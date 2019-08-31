---
layout:     post
title:      "Mac下载Android源码与编译记录"
subtitle:   "Mokee定制Rom的Android源码"
date:       2019-08-29 12:00:00
author:     "GuoHao"
header-img: "img/old-house.jpg"
catalog: true
tags:
    - 源码
    - Mokee
    - Mac
---


>系统版本：MacOS Mojave 10.14 (18A2063)  
预备软件工具：git  
使用外接固态硬盘，将硬盘格式化为：Mac OS 拓展（区分大小写，日志式）  
编译源码需要硬盘文件系统支持大小写敏感，不然会报错。  
 
# 第一步，下载mokee定制repo
[参考文章](https://bbs.mokeedev.com/t/topic/21)  
 
# 第二步，下载源码
[参考文章](https://bbs.mokeedev.com/t/topic/627)  
 
repo init：用于下载整个项目的清单文件，并通过 -b 后的参数，决定分支（8.0 or 9.0等）  
repo sync：真同步代码，根据清单文件为每个子项目配置的源，逐个去下载子项目，个人猜测用 git clone；  
		   其中还可以给每个子项目修改源，避免有些指向谷歌的源不可用。
 
我下载了9.0分支，命令：
```
repo init -u https://github.com/MoKee/android -b mkp --depth 1
```
mkp：指 Android9.0，同理 mko 就是 Android8.0。  
depth：用于指定克隆深度，为1即表示只克隆最近一次commit。
  
repo sync 过程中遇到的错误：  
错误1:  
```
error: Cannot fetch platform/prebuilts/gradle-plugin from https://android.googlesource.com/platform/prebuilts/gradle-plugin
```
原因是连接不了谷歌服务器，我的解决办法是使用清华大学的镜像. 
具体操作：修改用户根目录.bash_profile文件（我的用户根目录没有.bashrc文件），添加：
```
export MK_AOSP_REMOTE=tuna
```
然后执行命令使生效：source ~/.bash_profile. 
之后再去repo sync，原本从谷歌下载的子模块，都会转到从清华大学的镜像下载. 
到此，错误1 解决！  

错误2:  
第一种：
```
Fetching project MoKee/android_packages_apps_SetupWizard
fatal: unable to access 'https://github.com/MoKee/android_packages_apps_SetupWizard/': Could not resolve host: github.com
Receiving objects:  87% (11083/12634), 870.27 MiB | 288.00 KiB/s 
```

第二种：
```
Fetching project MoKee/android_packages_apps_Backgrounds
remote: Enumerating objects: 310, done.        MiB | 1.03 MiB/s   
remote: Counting objects: 100% (310/310), done.        
remote: Compressing objects: 100% (140/140), done.        
error: RPC failed; curl 18 transfer closed with outstanding read data remaining
fatal: the remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
Receiving objects:  92% (11748/12634), 1.00 GiB | 803.00 KiB/s 
```
上述这种问题，多半是由于网络不稳定等原因造成的，继续执行 repo sync，直到完成即可。  
或者参考该文章，写一个自动repo的脚本. 
[参考该文章](https://blog.csdn.net/zhuzhuzhu22/article/details/50931155)  
到此，错误2 解决！  

# 第三步，配置系统环境
基础配置：  
系统版本：MacOS Mojave 10.14 (18A2063)   
硬盘：外界固态硬盘，将硬盘格式化为：Mac OS 拓展（区分大小写，日志式）  

先看谷歌官方的文章（需要科学上网）  
[编译环境的配置要求](https://source.android.com/source/requirements.html)  
[搭建编译环境](https://source.android.com/source/initializing.html)  
看完这两篇文章，基本ok了。  

### xocde配置
xocde我用的10.1，[苹果官网下载安装](https://developer.apple.com/download/more/)  
xocde的命令行工具，可以通过命令行安装：xcode-select --install  
安装成功后，再执行命令：xcode-select --install，会有如下信息：  
```
bogon:~ guohao$ xcode-select --install
xcode-select: error: command line tools are already installed, use "Software Update" to install updates
```

查看xcode-select的位置，执行命令：```xcode-select -p```,会有如下信息： 
```
bogon:~ guohao$ xcode-select -p
/Library/Developer/CommandLineTools
```

该位置下，有以下文件：
```
bogon:~ guohao$ cd /Library/Developer/CommandLineTools
bogon:CommandLineTools guohao$ ls
Library		Packages	SDKs		usr
bogon:CommandLineTools guohao$ cd SDKs/
bogon:SDKs guohao$ ls
MacOSX.sdk	MacOSX10.14.sdk
```
可知安装完 xcode 命令行工具后，会带有 MacOSX10.14.sdk，编译源码时会用到。  
在源码根路径，执行. 
vi build/soong/cc/config/x86_darwin_host.go. 
找到：
```
        darwinSupportedSdkVersions = []string{
                "10.10",
                "10.11",
                "10.12",
                "10.13",
                "10.14", 
        }
```
可知我下载的9.0源码已经支持10.14的sdk。



### make配置
关于make版本，也就是gmake（gnu make）：
```
bogon:~ guohao$ make --version
GNU Make 3.81
Copyright (C) 2006  Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.

This program built for i386-apple-darwin11.3.0
```
```
bogon:~ guohao$ gnumake -version
GNU Make 3.81
Copyright (C) 2006  Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.

This program built for i386-apple-darwin11.3.0
```
我的make版本是GNU Make 3.81。


### jdk配置
关于jdk的版本，谷歌官方文档里说的是MacOS的使用Oracle的jdk1.8。
而我的openJDK是：AdoptOpenJDK  
[在此处下载](https://adoptopenjdk.net，OpenJDK8(LTS))，选择HotSpot的JVM  
由于我的openJdk是之前就装好的，就试着用来编译一下，也没遇到什么问题。  
但是还是推荐大家遵循官方的推荐,使用Oracle的jdk1.8。  


### python配置
关于python，这里我遇到一个坑，官方里说python的版本是：  
python.org 中提供的 Python 2.6 - 2.7  
然后我用命令查看：  
```
bogon:~ guohao$ python --version
Python 2.7.10
```
因此我以为我的python版本是ok的，后来编译源码遇到报错：  
env: python2: No such file or directory  
最后再Mokee论坛请教导演，解决问题：  
brew install python@2  
安装后再查看命令：  
```
bogon:~ guohao$ python --version
Python 2.7.16
```
<br/>

到此说一下我的个人理解:
配置jdk，是用来编译java层的代码，  
配置make，是用来编译c语言的代码，  
配置python，是用来支持编译脚本工作的，我们下载源码，编译源码的脚本都需要python的支持。  
<br/>

### 其他的配置  
[参考该文章](https://bbs.mokeedev.com/t/topic/32)  
我安装了：  
brew install findutils  
brew install gnu-sed  
brew install coreutils   
下载xz 非命令  

但注意 MacOS Mojave 10.14 的系统，不需要装 coreutils，  
我装了之后，编译源码时遇到报错：
```
stat: cannot read file system information for ‘%z’: No such file or directory
/bin/bash: File: “/Volumes/T1/Mokee/mkp/out/target/product/osborn/cache.img”
ID: 100000900000017 Namelen: ? Type: hfs
Block size: 4096 Fundamental block size: 4096
Blocks: Total: 244190008 Free: 216603401 Available: 216603401
Inodes: Total: 4294967279 Free: 4293707630
+
0 : syntax error in expression (error token is ": “/Volumes/T1/Mokee/mkp/out/target/product/osborn/cache.img”
```
卸载coreutils：brew uninstall coreutils，即可解决问题。  
<br/>
<br/>

另一个报错：  
```
The boot animation could not be generated as  
ImageMagick is not installed in your system.
```
用命令下载安装：  
```
brew install imagemagick
```
<br/>
<br/>

根据官方文档说明：  
通过 MacPorts 获取 Make、Git 和 GPG 程序包，  
我用 brew工具，  
brew install git gnupg2 成功  
brew install gmake libsdl 失败，估计是我电脑已经装了make  
<br/>

优化编译环境:  
Mac 系统下默认只能同时打开 1024 个文件，而在进行 Android 源码编译时有可能会超出这个限制，  
因此需要解除这个限制。在~/.bash_profile中添加以下内容：
```
#set the number of open files to be 1024
ulimit -S -n 1024
```
<br/>

最后用编译命令：
```
. build/envsetup.sh
lunch mk_osborn-userdebug
mka bacon
```
<br/>

lunch的时候，会去下载坚果pro2的device，然后开始编译。  
如果遇到报错，根据错误信息去百度谷歌查找解决办法，然后继续mka bacon。  
终于在折腾的6、7天后，成功编译。  

以上列举的问题就是我编译时遇到的，特此做个记录。