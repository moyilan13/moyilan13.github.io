---
layout:     post
title:      "Mac安装Jekyll"
subtitle:   "即时浏览你的博客"
date:       2019-08-28 12:00:00
author:     "GuoHao"
header-img: "img/guitar.jpg"
catalog: true
tags:
    - 配置 
    - Mac
    - Jekyll
---


执行该命令安装：
```
gem install jekyll
```
<br/>
### 遇到报错一：  
Ruby version >= 2.4.0
<br/>
接着就去尝试升级 Ruby

参考：[MAC_Ruby 安装](https://www.jianshu.com/p/c073e6fc01f5)  
此外，还参考了该文章：
[Mac安装遇见Ruby version >= 2.2.2. 的问题](https://www.jianshu.com/p/48ad6365f3eb)  
  
先执行该命令安装 RVM Ruby 版本管理器：
```
$ curl -L get.rvm.io | bash -s stable
```

### 遇到报错二：
```
gpg: 无法检查签名：No public key
GPG signature verification failed for '/Users/guohao/.rvm/archives/rvm-1.29.9.tgz' - 'https://github.com/rvm/rvm/releases/download/1.29.9/1.29.9.tar.gz.asc'! Try to install GPG v2 and then fetch the public key:

    gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

or if it fails:

    command curl -sSL https://rvm.io/mpapis.asc | gpg --import -
    command curl -sSL https://rvm.io/pkuczynski.asc | gpg --import -

In case of further problems with validation please refer to https://rvm.io/rvm/security
```

根据提示，执行该命令获取密钥：
```
gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
```

谁知又遇到了报错：
```
gpg: 从公钥服务器接收失败：No route to host
```

后来在该链接找到办法：[mac cocoa pods 安装过程中出现问题](https://blog.csdn.net/tongwei117/article/details/79789111)

该链接中，获取密钥的服务器路径是：hkp://keys.gnupg.net，
换成该路径试试:
```
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
```
居然成功了。。。有点搞笑的是，原文章里的作者尝试该服务器路径是失败了。。。哭笑不得.jpg
<br/>
<br/>
<br/>
继续  
安装 jekyll和 jekyll bundler：
```
$ gem install jekyll
$ gem install jekyll bundler
```

### 遇到报错三：
```
ERROR:  Could not find a valid gem 'jekyll' (>= 0) in any repository
```
参考：
[Could not find a valid gem 'jekyll' 安装jekyll问题解决](https://www.iteye.com/blog/sunxboy-2217811)，
输入命令检查源：
```
$ gem sources -l
```
发现源为空，重新配置一下就好：
```
gem sources -a https://rubygems.org/
```

重新执行安装命令，安装好后，显示：
```
...
Done installing documentation for bundler after 2 seconds
2 gems installed
```

进入你的 Blog 所在目录，然后创建本地服务器：
```
$ jekyll s
```
### 遇到报错四：
```
Dependency Error: Yikes! It looks like you don't have jekyll-paginate or one of its dependencies installed. 
```
安装 jekyll-paginate 即可：
```
$ sudo gem install jekyll-paginate
```
你就可以在 http://127.0.0.1:4000/ 看到你的博客，你对本地博客的修改都会在这个地址进行显示，这大大提高了对博客的配置效率。**使用ctrl+c就可以停止 serve**
<br/>
<br/>
补充，配置环境
```
echo 'export PATH="/usr/local/lib/ruby/gems/2.6.0/gems/jekyll-4.0.0/exe:$PATH"' >> ~/.bash_profile
```
