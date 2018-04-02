---
title: Cocoapods相关
date: 2018-3-25 15:36:41
categories: iOS
tags: iOS
---

## Cocoapods
1. `pod install` 会参照`Podfile.lock`文件中的版本去安装 
   `pod update` 会参照 `Podfile`文件中的版本去安装
    注意协同开发时候, pod库版本不一致的情况, 一定要保证版本同步
    可以根据`pod --help`查看各个命令的详细区别

2. cocoapods远程spec库包含了各种框架的描述信息(后缀名为: .podspec), 框架的描述信息包括(框架名称, 版本, 作者, 真正的代码存放地址等等...)
    - `pod setup`会clone远程的spec库到本地 (git仓库)
    - `pod search xx`会在本地spec库中生成的的检索索引文件中检索
    - 本地生成索引文件在`/Users/gaoyu/Library/Caches/CocoaPods`目录下, 该目录下有: 
    
        - Pods文件夹, 里面放着很多下载的库的源码和spec文件, 所以以后的pod install安装会特别快, 因为有此本地缓存, 这个可以删除
        - search_index.json 生成的检索索引文件, 如果删除掉, 再次`pod search xx`, 会提示`Creating search index for spec repo 'master'`, 该文件大概有十几兆那么大
    
    所以要生成自己的cocoapods框架的步骤如下: 

    - 上传自己的代码到远程仓库
    
    - 给自己的仓库加一个spec文件作为描述文件, 并填写项目的描述 `pod spec create YouLibName` 命令创建podspec
    
    - 将spec文件上传到 cocoapods的远程索引库中 可以使用`trunk`方式进行上传
        [方法](/https://guides.cocoapods.org/making/getting-setup-with-trunk.html) :
        - 注册trunk
        - 通过trunk推送podspec文件
        - 等待审核
    - 其他用户在更新本地索引库的时候会可以查找到你的框架
   
3. cocoapods 本地私有仓库的使用
    - 本地写好私有库并创建 podspec 文件
    - podspec 文件中的 
    
            
            将
            s.source = { :git => "https://github/xxx.git", tag => "#{s.version}"}
            直接改为: 
            s.source = { :git => "", tag => "#{s.version}"}
            
    
    - 项目工程中创建 Podfile 文件并填写如下: 
    
            
            
            # path代表会在某个路径下面寻找`YouLib.podspec`文件
            pod 'YouLib', :Path => '../YouLib'
            
    
    
4. cocoapods 本地私有仓库的示例工程
    - 方式一: 根据上面创建私有仓库的步骤进行完成之后, 在私有库文件夹下, 创建demo工程, demo工程也利用cocoapods的形式, 依赖私有仓库
    - 方式二: 使用命令行创建pod库的模板库 (非常推荐)
    直接使用下面的命令, 不用走👆的第三部, 根据命令行的提示来生成模板, 自动填充
    
            pod lib create LPExtension
    

5. cocoapods 远程私有库的使用 (✨✨非常重要✨✨)
    
    方式一: (参照cocoapods的流程)
    
    - 创建远程私有代码仓库和远程私有索引仓库(只包含podspec文件, 类似于cocoapods的远程索引库)
    
    - 书写自己的私有框架代码并创建podspec文件`pod lib create LPExtension`.然后书写代码. 注意修改podspec文件的`s.source = { :git => "https://github/xxx.git", tag => "#{s.version}"}`等其他关键信息
    - 将写好的框架上传到远程的私有代码库中
    - 使用`pod lib lint`, 检查当前所在目录下的podspec文件是否填写正确(本地验证) 本地验证不会验证`s.source`
    - 使用`pod spec lint`, 检查当前所在目录下的podspec文件是否填写正确(远程验证) 

    - 使用命令`pod repo add + 名称 + 远程spec地址`命令, 使本地的cocoapods索引库包含我们自己的远程索引库
    
        ```
        pod repo add LPExtension git@bitbucket.loopeer.com.git
        ```
        
    - 再去 `.cocoapods/repos`路径下, 包含`master(pods官方提供的公共的索引库)`, 还多了一个`LPExtension`自己的索引库
    或者使用命令`pod repo`查看已有的索引库
    
    - 将本地代码仓库中的podspec文件提交到本地的新的索引库中,并推送到`远程私有索引库`中
    
    ```
    // 将本地的LPExtension.podspec推送到本地的LPExtension的索引库, 并且系统会同步本地的LPExtension索引库和远程相关联的LPExtension索引库
    pod repo push LPExtension LPExtension.podspec
    // 如果出现警告提交不上去,可以加忽略警告的参数
    pod repo push LPExtension LPExtension.podspec --allow-warning
    ```
    
    - 使用的时候在podfile中填写两个source: 
    
    > 可以使用`pod repo`来查看source
    
    ```
    source 'git@github.loopeer.com.git'
    source 'https://github.com/Cocoapods/Specs.git'
    
    platform :ios, '8.0'
    
    target 'demo' do 
    use_frameworks!
    
    pod 'LPExtension'
    pod 'MJRefresh'
    
    end
    
    ```

    方式二: 直接填私有仓库的远程地址
    
    ```
    pod 'LPExtensions', :git => 'git@bitbucket.org:loopeer/lpextensions.git', :commit => '63be1f6'
    ```

6. cocoapods远程仓库的升级维护
    - 修改本地代码后, 修改podspec, 推送到远程仓库, 打tag
    - `pod repo push LPExtension LPExtension.podspec`
    - 如果出现警告提交不上去,可以加忽略警告的参数

        ```
        pod repo push LPExtension LPExtension.podspec --allow-warning
        ```

7. cocoapods子库的形式
    
    > 比如: 我们项目仅仅想使用AFNetworking的Reachability框架, 我们可以在Podfile文件中写入`pod 'AFNetworking/Reachability'`
        
    修改podspec文件,制作成自己的subspec
    - 原podspec注释掉`s.source_files = 'LPFramework/Classes/**/*'`  (因为我们将原来的库转换成几个子库, 所以每个子库的source_files是不一样的, 需要子库进行不同的设置)
    - 原podspec注释掉`s.dependency` (每个子库进行不同的设置依赖)
    
    - 添加子库
    
    ```
    s.subspec 'BannerView' do |bannerView|
    bannerView.source_files = ['Sources/BannerView/*.swift']
    bannerView.dependency 'Kingfisher', '~> 4.0.1'
    bannerView.dependency 'LPFramework/KFImageProcessor'
  end
 
  s.subspec 'KFImageProcessor' do |imageProcessor|
    imageProcessor.source_files = ['Sources/KFImageProcessor/*.swift']
    imageProcessor.dependency 'Kingfisher', '~> 4.0.1'
  end
  
    s.default_subspec = ['EmptyView', 'PlaceholderTextView', 'UserManager', 'CellAnimator', 'KFImageProcessor']

    ```
    
    
    - 使用
        - 方式一: 
        
        ```
        pod 'LPFramework/CrashReport', :git => 'git@bitbucket.org:loopeer/lpframework.git', :commit => '7a9a19f'
        ```
        
        - 方式二: (本地cocoapods repo包含自己私有索引库的情况下)
        
        
        ```
        pod 'LPFramework/CrashReport'
        ```
        
        
        - 方式三: (Podfile使用多个子库)
        
        ```
        pod 'LPFramework', :subspecs => ['EmptyView', 'UserManager', 'CellAnimator']
        ```


8. cocoapods库中的资源
    - XIB/nib的资源 
![bundle_1](https://raw.githubusercontent.com/gaoyuexit/Blog/master/images/2018-3-25-Cocoapods/bundle_1.png)

    - 图片的资源
    // 如果使用`pod lib create LPFramework`创建的pod模板, 则将图片放入路径`'LPFramework/Assets/*`

    ```
    s.resource_bundles = {
        'LPFramework' => ['LPFramework/Assets/*']
    }
    ```
    
    重新pod install
    这样形成的图片在以下的目录
    ![bundle_2](https://raw.githubusercontent.com/gaoyuexit/Blog/master/images/2018-3-25-Cocoapods/bundle_2.png)

    加载图片
    
    ```
    NSBundle *currentBundle = [NSBundle bundleForClass:[self class]];
    // 根据屏幕的scale拼接 @2x还是@3x 的图片
    NSString *imgPath = [currentBundle pathForResource:@"1@2x.png" ofType:nil inDirectory:@"LPFramework.bundle"];
    UIImage *img = [UIImage imageWithContentsOfFile:imgPath];
    ```

9. cocoapods 自动化

> 制作私有cocoapods库的步骤

    1. 创建远程的代码仓库和远程的cocoapods spec库
    2. 使用命令`pod repo add + 名称 + 远程spec仓库`命令, 使本地的cocoapods spec 库包含我们自己的远程spec库
    3. 执行 `pod lib create LPFramework`
    4. 链接远程代码仓库地址`git remote add origin '远程代码仓库地址'`
    5. 书写代码
    6. 修改 `spec`文件
    7. demo工程 `pod install`
    8. 提交本地代码到远程代码仓库
    9. 打标签, 提交代码到远程代码仓库
    10. 验证`spec`
    11. 将`spec`推送到本地spec库和远程的spec库

如果代码升级, 则 重复执行 `5-11`步骤的操作
这时候就可以使用自动化脚本来执行默认的操作
示例: 使用`Fastlane`(一个ruby脚本集合)

`lane` 表示一个航道(一个任务), 其中包括很多`Action`
我们只需要定义一个lane, 编写里面的action, 就可以完成自动化工作

使用`Fastlane`: 
    a. 安装 `Fastlane`
    b. 在代码库的目录中, `fastlane init` (使用付费appid登录)
    c. 在`fastlane`文件夹中创建`Fastfile`文件, 打开编写如下
    
    ```
    # 描述信息
    desc 'ManagerLib 使用这个航道, 可以快速的对自己的私有库, 进行升级维护'
    lane :ManagerLib do |options|
    # options 输入的参数
    
    tagName = options[:tag]
    targetName = options[:target]

    # 具体这个巷道上面执行哪些行为
    # 下面所有的路径都是相对于项目的根目录
    
    # 1. pod install
    cocoapods(
        clean: true,
        podfile: "./Example/Podfile"
    )

    # 2. git add .
    git_add(path: ".")
    #    git commit -m 'xxx'
    git_commit(path: ".", message: "版本升级维护")
    #    git push origin master
    push_to_git_remote

    # 3. git tag 标签名称
    add_git_tag(
        tag: tagName
    )
    #    git push --tags
    push_git_tags

    # 4. pod spec lint
    pod_lib_lint(allow_warnings: true)
    #    pod repo push LoopeerSpec xxx.podspec
    pod_push(path: "#{targetName}.podspec", repo: "LoopeerSpec", allow_warnings: true)
    
    end
    
    ```
    
d. 在项目根目录输入命令`fastlane lanes`, 检查Fastfile文件是否语法正确
e. 执行命令 `fastlane ManagerLib tag:0.1.0 target:LPFramework`



##组件化

> 引出: 项目发展庞大, 业务主线增多, 开发人员增多, 会暴露一系列的问题: 耦合严重, 编译速度慢, 测试不独立, 无法使用自己擅长的设计模式..

组件的优势: 

- 组件的独立: 独立编写, 独立 编译, 独立测试, 独立运行
- 资源的重用: 功能代码的重用
- 高效的迭代: 增删模块
- 可配合二进制化, 最大的提高项目的编译速度
 
组件化需要考虑的事情: 

- 需要把那些内容划分为一个组件: 基础组件, 功能组件, 业务组件
    - 基础组件
        - 基本配置: 常量, 宏
        - 分类: 各种系统类的扩展
        - 网络: 对AFN的封装, SDWebImage的封装
        - 工具: 日期时间处理, 文件处理, 设备信息..
    - 功能组件
        - 控件: 弹幕, 轮播图, 选项菜单
        - 功能: 断点下载, 音频处理
    - 业务组件
        - 业务线1: 子业务1, 子业务2

- 每个组件以什么样的形式存在

    > 一般 `业务组件`包含`基础组件`和`功能组件`, `基础组件`和`功能组件`之间不要存在依赖关系, 业务组件之间也不要存在依赖关系,
例如: 列表模块和播放模块之间, 点击列表调到播放界面两个模块, 如何通讯,如何传值, 而且不要存在依赖关系
    
    > 组件以Pod库的形式存在, 单独的测试功能

- 以怎样的形式集成各个组件
    
    > 以cocoapods的形式安装各个组件

- 组件之间如何通讯
    - 公开api
    - 通过中间件的中转
- 其他
    - 如何提高编译速度
    - 如何解决重复的流程操作
    - 库的升级维护
- 其他方式: 1. 主工程-子工程  2.framework形式 3.pods形式

- 解耦
    - 如果一个组件依赖其他公共功能, 如何处理
    
    > 1,copy公共代码  2,把组件依赖的代码,做成Pod库并依赖
    
    - 如果组件内部, 需要对接某个服务, 如何处理
    
    > eg: 组件内需要加载网络图片
        - 组件内部依赖外部框架(SDWebImage)
        - 使用block/delegate, 将这部分职能丢到外面去做
    
    
## 细节

```
UIImage *img = [UIImage imageNamed:@"1.png"];
NSString *path = [[NSBundle mainBundle] pathForResource:@"1.png" ofType:nil];
NSLog(@"%@ ---- %@", img, path);
```

1. 图片如果在目录中, 并且勾选了`Target Membership`, 则编译后会在编译形成的xx.app中, 显示包内容则会看到, img和path均不为空
`NSBundle mainBundle`就是编译形成的xx.app显示包内容的路径

2. 图片如果在目录中, 没有勾选了`Target Membership`, 则编译后形成的xx.app中, 显示包内容不会看到该图片, img和path均为nil
3. 如果图片在`Assets.xcassets`中,  则编译后形成的xx.app中, 显示包内容不会看到该图片,会看到assets.cer文件m, 则 img 不为nil, path 为nil
4. imageNamed图片不会被释放, pathForResource图片会被释放, 所以例如新特性等大图片, 可以不放在`Assets.xcassets`中

5. imageNamed 和 imageWithContentsOfFile加载图片的区别
    - imageNamed只能从mainBundle和cer文件中获取图片
    - imageWithContentsOfFile 可以从任何路径获取图片
    - imageName的方式会在使用的时候系统会cache，程序员是无法处理cache的，这是由系统自动处理的，对于重复加载的图像，速度会提升很多，这样反而用户体验好。所以如果某张图片需要在应用中使用多次，或者重复引用，使用imageName的方式会更好

    - imageWithContentsOfFile的方式，在使用完成之后系统会释放，不会缓存下来，所以也就没有这样的问题。一般也不会把所有的图片都会缓存。有些图片在应用中只使用一两次的，就可以用这样的方式，比如新手引导界面的图片等等，就适合这样的方式。没有明显的界限。


