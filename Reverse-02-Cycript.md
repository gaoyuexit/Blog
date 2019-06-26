---
title: 逆向工程-02-Cycript
date: 2018-4-28 12:26:41
categories: iOS
tags: iOS
---

# 逆向工程-02-Cycript

## 1. cycript的开启与关闭

### 开启:

Cydia安装`Cycript`, ssh进入iPhone
 
* 直接输入`cycript`, 就可以进入Cycript环境了
* `cycript -p 进程ID` 例如: `cycript -p 3303`
* `cycript -p 进程名称` 例如: `cycript -p SpringBoard`

### 关闭

* 取消输入: `Ctrl + C`
* 退出: `ctrl + D`

## 2. PS命令
1. `Cydia`安装`adv-cmds`
2. ps指令是`process status`的缩写, 使用`ps`指令可以列出系统当前的进程
3. 列出所有的进程 `ps -A`
4. 搜索关键词: `ps -A | grep 关键词`

## 3. 对于非完美越狱下的`cycript`的使用
对于非完美越狱下的`cycript`命令,会提示`killed 9`, 可以使用[bfinject](https://github.com/BishopFox/bfinject)解决

bfinject注意事项: 

1. `Electra`越狱要关闭`Tweaks`
2. `bfinject`执行命令, 需要在`bfinject`文件夹下, 执行命令例如: `bash bfinject -P Reddit -L test`
3. 如果`bfinject`命令报错: `Unknown jailbreak. Aborting.`, 可以参考解决[方案](https://github.com/BishopFox/bfinject/pull/18), 可以使用[这个版本的bfinject](https://github.com/klmitchell2/bfinject)中的`bfinject.tar`
4. `cycript`的[安装](https://juejin.im/post/5afd12e1f265da0b722b543d)及碰到的问题(我操作的时候都碰到了- -!)
5. [官网](http://www.cycript.org/)下载的`cycript`, 安装的环境变量推荐如下(我的习惯): 
    * 将下载好的名为`cycript_0.9.594`的`cycript sdk`放到目录`/usr/local`中, 即`/usr/local/cycript_0.9.594`
    * 在`/etc/paths.d/`目录下创建一个空文件, 即`sudo touch /etc/paths.d/cycript`
    * 在cycript空文件中输入`/usr/local/cycript_0.9.594`, 并保存
    
## 4. 常用语法
1. `UIApp` 等价于 `[UIApplication sharedApplication]`
2. 定义变量 `var window = UIApp.keyWindow`
3. `#内存地址 = 内存地址指向的对象`
4. `ObjectiveC.class` 已加载的所有OC类
5. 查看对象的所有成员变量 `*UIApp 可以打印UIApp中的所有成员变量`
6. 递归打印view的所有子控件(和LLDB一样的函数)

    ```
    LLDB: po [self.view recursiveDescription] //递归
    Cycript: UIApp.keyWindow.recursiveDescription().toString()
    ```
7. 筛选出当前页面内存中的某种类型的对象:  
    * `choose(UIViewController)` 打印当前页面内存中的所有控制器对象
    * `choose(UITableViewCell)` 打印当前页面内存中的所有cell对象
8. 在`cycripte`环境中, 可以写函数, 例如该环境下 `function sum(a, b) { return a + b; }` , 按下回车键, 相当于定义了这个函数, 我们可以在该环境下, 输入命令: `sum(10, 20)`, 就会得出结果
9. 根据上面可以定义函数的特性, 我们可以把常用的`cycript`代码封装到一个`.cy`文件中


    
    



