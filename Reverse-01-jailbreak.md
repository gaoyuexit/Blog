---
title: 逆向工程-01-基础
date: 2018-4-25 15:36:41
categories: iOS
tags: iOS
---

# 逆向工程

## 1. 越狱

* 可以通过`PP助手`进行查询能否进行越狱, 并使用PP助手提供的方式进行越狱(操作简单)

* 因为手上只有一台iOS11.0.3系统的iPhone5s, PP助手没有相应的方式进行越狱
* 通过[wiki](https://en.wikipedia.org/wiki/IOS_jailbreaking)可以看到最高已经支持(11.2 - 11.4 beta3)系统版本的不完美越狱了. 也可以看到我手上的`iOS11.0.`可以通过`LiberiOS`和`Electra`两种方式来进行不完美越狱. 这里我选用的是`Electra`方式进行的[越狱](http://www.idownloadblog.com/2018/03/01/how-to-jailbreak-electra/)
* 这种不完美越狱的方式, 重启之后还需要进行`jailbreak`, 遇到的问题, 如果越狱后的`Cydia`部分页面显示无法连接到网络, 则需要使用VPN

## 2. Cydia

`Cydia`相当于越狱后的`AppStore`, 可以安装第三方的软件(插件, 布丁, App)
`Cydia`的作者: Jay Freeman (saurik)
`Cydia`的安装包格式是`deb`格式的(结合软件包管理工具apt)
`Cydia`安装软件的步骤: 

* 安装软件源 eg: PP助手源: http://apt.25pp.com
* 安装软件源: saurik(Cydia作者)的软件源: http://apt.saurik.com 或者 http://apt.saurik.com/cydia/
* 进入软件源找到相应的软件, 进行安装
* 如果`Cydia`安装软件安装不上,报错,失败等情况, 可以现在网络上下载`deb`格式的安装包, 然后将`deb安装包`放到目录`/var/root/Media/Cydia/AutoInstall`, 重启手机, `Cydia`就会自动安装deb文件
* 例如: `iFile`该软件是由很多个`deb安装包构成`, 将这个软件的所有`deb`文件放到目录`/var/root/Media/Cydia/AutoInstall`, 重启手机
* 因为我是iOS11.0.3系统通过`Electra`方式进行的越狱, 因为[Saurik与Electra越狱工具开发者不和](http://news.tongbu.com/96160.html), `saurik`并未对`Cycript`进行iOS 11的适配，包括Cydia本身。解决方案: [bfinject](https://github.com/BishopFox/bfinject)




## 3. 必备软件

### iPhone 端
* 补丁: 可以访问iOS设备的文件系统的补丁, 这样在Mac上使用iFun就可以查看iPhone的文件系统: 
    * Apple File Conduit "2"
    * 类似的还有afc2, afc2add
    
* 补丁: 可以绕过系统验证, 随意安装,运行破解的ipa安装包的补丁
    * AppSync Unified
    
* 软件: 可以在iPhone本身访问自己的iOS文件系统的软件
    * iFile
    * FilzaEscaped 这个软件即使设备不越狱, 可以查看iOS的文件系统
    
* OpenSSH
* VI IMproved (直接搜vim), 可以ssh登录到iPhone端, 使用vim编辑iPhone文件
  
### Mac 端 

* iFunBox(查看iPhone文件系统), PP助手 
    
    
## 4. 代码判断手机是否越狱
   
[参考1](https://blog.csdn.net/happyshaotang2/article/details/80060334)
[参考2](https://medium.com/@pinmadhon/how-to-check-your-app-is-installed-on-a-jailbroken-device-67fa0170cf56)
  
## 5. Mac远程登录到iPhone

> 想让Mac操纵iPhone, 需要Mac和iPhone建立连接, 方式: 使用OpenSSH的方式让Mac远程登录到iPhone (相当于Mac为客户端, iPhone为服务端)

### 方式一: 通过同一网络进行SSH登录

* SSL: (Secure Sockets Layer): 为网络通信提供安全及数据完整性的一种安全协议, 在传输层对网络进行加密 
* OpenSSL: 是SSL协议的开源实现 eg: 绝大部分的HTTPS请求等价于HTTP + OpenSSL https的SSL层验证
* SSH(Secure Shell): 安全外壳协议, 是一种为远程登录提供安全保障的协议, 使用SSH, 可以把所有传输的数据进行加密, `中间人`攻击方式就不能实现, 防止DNS欺骗和IP欺骗
* OpenSSH: 是SSH协议的免费开源实现
* OpenSSH的加密就是通过`OpenSSL`完成的

步骤: 

* SSH通过TCP协议进行通信, 所以确保Mac和iPhone在同一局域网下, 比如连着同一个WIFI
* 在Mac的终端输入 `ssh 账号名@服务器主机地址` eg: `ssh root@手机的ip地址`
* 输入手机root账号的默认密码, 就可以登录到iPhone了: `OpenSSH`的软件作者已经提供了: 默认密码 `alpine`, 可以在`OpenSSH`的说明中查看
* 退出root账号: exit

> iOS有两个常用账户: root, mobile

* root账户: 最高权限账户, $HOME是 /var/root
* mobile账户: 普通权限账户, $HOME是 /var/mobile, 只能操作一些普通文件, 不能操作系统级别的文件
* 登录mobile账户: `ssh mobile@手机iP`, 输入密码: `alpine`

> 越狱后, 为了手机的安全, 最好修改root/mobile账户的密码

* 登录到root账号
* root账号下, 输入命令`passwd`, 就可以修改root账号的默认密码
* root账号下, 输入命令`passwd mobile`, 就可以修改mobile账号的默认密码

> SSH协议一共两个版本: SSH-1, SSH-2
> 现在用的较多的是SSH-2, 客户端和服务端版本要保持一致才能通信

查看SSH版本: (查看配置文件的Protocol字段)
客户端(Mac端): /etc/ssh/ssh_config 文件中Protocol字段为2
服务端(iPhone): /etc/ssh/sshd_config 文件中Protocol字段为2
作为客户端查看`ssh_config`, 作为服务端查看`sshd_config`(服务器的配置文件)

> 端口: 端口是设备对外提供服务的窗口, 每个端口都有一个端口号(范围0-65535, 共2^16个)
> 有些端口号是保留的, 比如:  (Mac登录到iPhone的22端口)
    
    
* 21端口提供FTP服务
* 80端口提供HTTP服务
* 22端口提供SSH服务(可以查看/etc/ssh/sshd_config的Port字段)

> SSH的通信过程, 可以分为3部分

* 建立安全连接
    服务器会提供自己的身份证明: 第一次连接, 服务器会将公钥等信息发送给客户端, 客户端保存到know_hosts中(该服务器ip地址对应一个公钥信息, 客户端存的公钥和服务器的公钥是一样的, 可以查看对比一下), 以后的连接, 服务器发送公钥等信息, 客户端会在know_hosts中查找, 如果根据iP查找的公钥信息对应不上, 就会报错误信息: 提醒服务器的身份信息发生了变更
    如果排除了中间人攻击的情况, 确定要连接此服务器, 删掉之前的服务器公钥信息就行: 
    * `ssh-keygen -R 服务器iP地址`
    * 或者打开know_hosts文件, 删除掉该服务器iP地址对应的公钥的那一行(两个方式是同样的操作)
    
    ![ssh](https://raw.githubusercontent.com/gaoyuexit/Blog/master/images/Reverse-01-jailbreak/ssh.png)
* 客户端认证:
    SSH-2 提供了两种常见的客户端认证方式
    * 基于密码的客户端认证: 使用账号密码即可
    * 基于秘钥的客户端认证: 免密码认证, 安全
    * SSH-2 默认优先尝试`秘钥认证`,如果认证失败, 才会尝试`密码认证`
![SSH-login](https://raw.githubusercontent.com/gaoyuexit/Blog/master/images/Reverse-01-jailbreak/SSH-login.png)
秘钥认证步骤: 

    * 客户端(Mac): 输入指令: `ssh-keygen`生成公钥私钥, 如果本来存在,跳过生成的这一步
    * 将公钥内容追加到服务器端(iPhone)的授权文件尾部, 输入指令: `ssh-copy-id root@iP地址` 或者 (手动拷贝客户端的公钥到服务器端的authorized_keys文件尾部也可以, 如果没有authorized_keys这个文件, 需要自己手动创建)
    * 如果我们配置成功后, 秘钥登录仍然不成功的话, 我们可能是文件权限不足: 我们可以设置如下的文件权限
    
        * chmod 755 ~   //可以参照鸟哥的Linx私房菜
        * chmod 755 ~/.ssh
        * chmod 644 ~/.ssh/authorized_keys
    
    * 举例:
    
        * 文件拷贝: 如果在一台主机上,拷贝文件,我们可以使用cp指令
        * 如果是想将文件在客户端拷贝到服务端, 我们需要scp指令
        * 将客户端的公钥文件复制到服务端根目录下的.ssh文件夹中
        * `scp ~/.ssh/id_rsa_pub root@iP地址:~/.ssh`
        * `scp`是`secure copy`的缩写, 是基于SSH登录进行安全的远程文件拷贝, 把一个文件copy到远程另外一台主机上
        * `cat id_rsa_pub >> authorized_keys` 这个指令是将当前目录下是`id_rsa_pub`的内容追加到`authorized_keys`的尾部, 如果`authorized_keys`不存在则自动生成该文件
    
* 数据传输: - 输入的指令



### 方式二: 通过USB进行SSH登录
默认情况下, 由于SSH走的是TCP协议, Mac通过网络连接的方式SSH登录到iPhone, 要求iPhone连接WIFI
但是为了加快传输速度, 也可以通过USB连接方式进行SSH登录
> Mac上有个服务程序`usbmuxd`(它会开机自启动), 可以将Mac的数据通过USB传输到iPhone
> `usbmuxd`程序的位置 `/System/Library/PrivateFrameworks/MobileDevice.framework/Resources/usbmuxd`

![usbmuxb](https://raw.githubusercontent.com/gaoyuexit/Blog/master/images/Reverse-01-jailbreak/usbmuxb.png)

步骤: 

* 下载`usbmuxd`工具包(下载v1.0.8版本, 里面有个python脚本: `tcpreplay.py`) [地址](https://cgit.sukimashita.com/usbmuxd.git/) 我们仅使用这个脚本, 其余的文件用不到
* 将iPhone的22端口(ssh端口)映射到Mac本地的10010端口(也可以是其他端口, 只要不是保留端口就行)
* 命令行: `python tcpreplay.py -t 22:10010` (-t参数表示能够同时支持多个ssh连接) 注意: 要想保持端口映射状态, 不能终止此命令行(如果要执行其他终端命令, 则需要新开一个命令行界面)
* 端口映射完毕后, 如果想和iPhone的22端口通信, 直接和Mac本地的10010端口通信即可
* 新开一个命令行界面 `ssh root@localhost -p 10010` 进行ssh登录 (-p表示端口号) 或者 `ssh root@127.0.0.1 -p 10010` 命令
* `usbmuxd` 会将本地的10010端口的TCP协议数据, 通过usb连接转发到iPhone的22端口

> 远程拷贝文件也可以直接和本地的10010端口通信

比如: 将Mac上的`~/Desktop/1.txt`拷贝到iPhone的`~/test`路径下: `scp -P 10010 ~/Desktop/1.txt root@localhost:~/test`

`scp --help`会发现, 指定端口号用`-P`表示

> 脚本: 
因为上面的操作, 映射等命令较长, 我们可以把经常使用的命令放到shell脚本中, 然后执行脚本文件

例如: 新建一个`usb.sh`脚本, 里面写入

```
python ~/User/gaoyu/sh/tcpreplay.py -t 22:10010
```

这样, 如果我们想要映射10010端口到iPhone的22端口, 我们就可以通过`sh`, `bash`, `source`命令来执行这个`usb.sh`脚本

> `sh`, `bash`, `source`命令的区别: 

举例: 脚本中以下命令, 分别用`sh`, `bash`, `source`命令执行该脚本, 会发现三个命令都会在`Desktop`目录下建立`abc`文件夹, 但是`sh`, `bash`执行该脚本, 执行完毕后, 终端还是在原来的路径下, 但是`source`执行该脚本, 执行完毕后, 终端的目录进入到`~/Desktop`路径下

```
cd ~/Desktop
mkdir abc
```

* `sh`, `bash`: 当前shell环境会启动一个子进程来执行脚本文件, 执行后返回到父进程的shell环境
    * 执行`cd ~/Desktop`指令, 在子进程会进入到`cd的Desktop`目录, 但是在父进程中环境并没有改变, 也就是说目录没有改变 
* `source`: 在当前环境执行脚本, `source`也可以用`.`表示: 即执行脚本: `. test.sh`

既然如此, 那么我们可以分别创建两个脚本,`usb.sh`用来映射端口, `login.sh`用来登录iPhone
`login.sh`的内容为: `ssh root@localhost -p 10010`


> iPhone的终端中文乱码问题: 

默认情况下, iPhone的终端不支持中文输入和显示
解决方案, 在iPhone的目录下新建一个`~/.inputrc`文件, 文件内容如下: 

```
set convert-meta off //不将中文字符转化为转义字符
set output-meta on   //允许向终端输出中文

set meta-flag on     
set input-meta on    //允许向终端输入中文
```

















