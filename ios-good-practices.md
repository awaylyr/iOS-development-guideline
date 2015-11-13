# iOS Good Practices

# 目录
1. [工程配置](#1-工程配置)
    - [Git 工作流程](#git-工作流程)
    - [工程版本管理](#工程版本管理)
    - [工程 Targets 管理](#工程-targets-管理)
    - [Cocoapods](#cocoapods)
    - [Storyboard](#storyboard)

2. [模块设计概要](#2-模块设计概要)
    - [工程结构](#工程结构)
    - [MVVM & MVC](#mvvm--mvc)
    - [KVO, Notification, Delegation, Block, Target-Action](#kvo-notification-delegation-block-target-action)
    - [模块化 & URL Route](#模块化-&-url-route)

# 1. 工程配置

### Git 工作流程

参考了 Github 的 Git 工作流程[Github flow](http://scottchacon.com/2011/08/31/github-flow.html)进行开发。

- 保持 master 分支上的代码时可以随时发布的
- 保持 develop 上的代码是稳定可执行的
- 需要增加新功能时，从最新的 develop 分支上创建一个新的 feature 分支，(ie: new-message-center)
- 新功能开发时保持在新建的 feature 分支上，尽量以最小粒度 commit 代码，commit 的内容以方便阅读，不需查看代码变动能知道大概改了什么为准。当达到一个稳定的版本时，push 到远程。
- 当新分支上的代码已经完成，或者需要与其他人协作时，可创建 pull request 请求 develop 分支进行合并（这步暂省略为直接请求项目主负责人主动将 feature 的代码合并回 develop）
- 为每个发布版本打 tag 版本号。

**gitignore**  
在项目根目录页放置项目通用的 .gitignore 文件，用于忽略无需进行版本管理的文件。现因为基本上每个工程要忽略的文件类型一致，直接在每台开发机器上自己设定全局的 .gitignore 文件。配置可参考[gitignore.io](https://www.gitignore.io/)


### 工程版本管理
工程版本号管理使用 Git 的 tag/branch/commit count 来生成版本号。一个典型的开发版版本号是
```
2.3+120 (2736.117)
// 2.3 为最近一个tag+0.1， 120为从最近一个 tag 到当前分支的 commit count.
// 2736 为 master 分支上的 commit count, 117 为当前分支的 commit count.
```

新建工程后在 `Targets -> Build Phases -> New Run Script Phase`，新建名为`Versioning From Git Tag`的脚本，引用
```
"${SRCROOT}/Scripts/versioningFromGitTag.sh"
```

其中 versioningFromGitTag.sh 可在本文档同目录下的 `ProjectScripts` 目录里获取。

设置完后，在确定要发版本前，切换到 master 分支，打上要发布的版本号 tag，开始打包流程。（无需修改工程文件的版本号）

### 工程 Targets 管理

新建工程后，一般会创建一个内部开发版本的 Target，主要用于方便内部测试时使用，如可以在内部开发版本增加内外网切换功能。

如旅游顾问 Mansinthe 工程，从 Mansinthe Target 复制了 MansintheInner Target。

设置 Bundle Indentifier 与生产环境版本区分允许同时安装两个版本, 设置 Bundle name 方便从名字区分内部版和生产环境版。

在 `Preprocessor Macros`中添加 INNER_VER=1，代码中使用 #ifdef INNER_VER 来区分是否为内部测试版。

### Cocoapods

所有项目使用 Cocoapods 进行第三方库、内部公共类库管理，国内的 SDK 因在 Cocoapods 源上的更新速度较慢，使用手动管理放入 ThirdParty 目录下。

建议将 Gem source 改为淘宝在国内做的镜像，[淘宝 RubyGems 镜像](https://ruby.taobao.org/)。一般情况下，pod install, pod update 使用 --no-repo-update 来忽略可能有的远程更新（墙内这步更新较慢）。

项目内，尽量保持同步更新 Cocoapods 的版本，不更新到测试版。

### Storyboard
尽量使用 storyboard 来创建视图，通常按不同业务模块分成不同的 storyboard，如果模块内的视图多余三个，考虑再细分成子 storyboard。


# 2. 模块设计概要
### 工程结构
除旧有项目外，新项目的结构基本会按照以下的结构进行搭建，工程的文件夹结构与工程内group结构基本保持一致。
```
+ Project
    + Client                # 网络通讯模块代码目录
    + CommonUI              # 工程内通用的视图组件代码，部分已放入 MSTUIKit 中，通过 cocoapods 引用。
    + Configuration         # 工程视图、皮肤、AppKey 等
    + Internal              # 基础组件，现基本已放入 MSTInternal 中，通过 cocoapods 引用。
    + Modules               # 细分模块目录
        + Module1
            + Client            # (可选), 模块相关的网络请求相关处理放入这里
            + Controller        # 存放视图控制器，ViewController 内不应包含数据逻辑、业务逻辑处理(放入ViewModel)
            + Model             # 存放模块相关数据结构
            + Store             # (可选), 当前模块有较重的独立存储逻辑处理时，可以单独出一个 Store 目录
            + View              # 存放模块视图相关文件，包括.xib, .storyboard, View 文件
            + ViewModel         # 存放模块相关业务逻辑文件
        + Module2
        ...

    + Resource              # 资源目录
        + Database              # 内置数据库文件目录
        + Images                # 其他不放入 Images.xcassets 的图片文件
        + Images.xcassets       # 图片资源文件

    + Skeleton              # App 主骨架代码目录 (AppDelegate, RootViewController ...)
    + Supporting Files      
    + ThirdParty            # 存放未使用 cocoapods 管理的第三方类库，目前主要是国内第三方 SDK (crash, track, share, push)

+ ProjectTests
    + Modules               # 测试目录结构基本同 Projects 下，按不同模块划分, 但 Module 下不再细分不同目录
+ Scripts                   # 放置脚本文件，如自动版本号更新脚本 versioningFromGitTag.sh

- podfile                   # 主要使用 cocoapods 引用内部公共基础库、第三方库
- .clang-format             # 代码风格约束文件，与 Clang-format 插件配合使用，详见 Objective-C 编码风格文档

```

### MVVM & MVC
各项目主体采用 MVVM 模式，不同于 MVC 的地方在于，将 ViewController 的职责再细分，将业务、数据逻辑代码放入 ViewModel 中，ViewController 基本只用于管理视图，对 ViewModel 的变化调整更新视图。
```
- Model         # 数据结构定义，同时处理部分轻逻辑
- View          # 视图，仅负责视图的展示、刷新
- ViewModel     # 负责数据的获取、存储、更新、处理，通知 Controller 数据变化
- Controller    # 负责视图声明周期控制，ViewModel 变化时更新视图
```

### KVO & NSNotification & Delegation & Block & Target-Action

- Key-Value Observing (KVO)

对某对象的值感兴趣时，可使用 KVO 进行消息传递。前提：1.接收者要知道发送者，2.接收者要知道发送者的生命周期，需要在发送者销毁前注销观察者身份。
在使用 ReactiveCocoa 框架时，用信号通知取代 KVO 代码更简便。

    Sender: NSOperation
    Recipient: NSOperationQueue
    Message: isExecuting isFinished isCancelled


- Notification

通常用于在两个不相关的模块中传递消息，发送者、接收者不需要知道双方。

    Sender: NSManagedObjectContext
    Recipient: Unknown
    Message: NSManagedObjectContextDidChangedNotificaion

- Delegation & Block

    - 如果对象有超过一个以上不同的事件源，使用 delegation；  
    - 如果一个对象是单例，不要使用 delegation；  
    - 相对 delegate 来说 block 的可读性更高；
    - delegation 更多面向过程，block 更多面向结果。如果需要一步步得到通知，使用 delegation；如果只是希望得到请求的信息（或错误），使用 block.


    // delegation
    Sender : UITableViewDelegate
    Recipient: UITableView
    Message: tableView:didSelectRowAtIndexPath:

    // block
    Sender: NSURLSession
    Recipient: Some Object
    Message: completionHandler

- Target-Action

Target-Action 在消息的发送者和接收者之间建立了一个松散的关系。消息的接收者不知道发送者，甚至消息的发送者也不知道消息的接收者会是什么。基于 target-action 传递机制的一个局限是，发送的消息不能携带自定义的信息。在 Mac 平台上 action 方法的第一个参数永远接收者。iOS 中，可以选择性的把发送者和触发 action 的事件作为参数。除此之外就没有别的控制 action 消息内容的方法了。

- Signals

使用 ReactiveCocoa，允许串联、合并、修改信号。通常在 ViewController 与 ViewModel 的通讯中使用。


如何选择：

```
                                          +--------------------------+           +---------------------------+
                                          |                          |           |                           |
                                          |       Notification       |           |       Delegation          |
                                          |                          |           |                           |
                                          +------------^-----^-------+           +-------------^-------------+
                                                       |     |                                 |
                                                      yes    +-------------------no-----+     yes
                                                       |                                |      |
                                          +------------+-------------+        +---------+------+-------------+
                                          |                          |        |                              |
                                          |   Multiple recipients?   +-------->   Sender knows recipient?    |
              +                           |                          |        |                              |
              |                           +------------^-------------+        +------------------------------+
              |                                        |
              |                                        no
              |                                        |
+-------------v-------------+             +------------+-------------+        +------------------------------+
|                           |             |                          |        |                              |
|  UI Event without custom  |             |    Recipient knows       |        |             KVO              |
|  payload?                 +-----no------>        Sender?           |        |                              |
|                           |             |                          |        +--------------^---------------+
+-------------+-------------+             +------------+-------------+                       |
              |                                        |                                    yes
             yes                                      yes                                    |
              |                                        |                                     |
+-------------v-------------+             +------------v-------------+        +--------------+---------------+
|                           |             |         Two way          |        |  Only about changed values   |
|        Target/Action      |             |      Communication?      +---no----> & sender is KVO compliant   |
|                           |             |                          |        |                              |
+---------------------------+             +------------+-------------+        +--------------+---------------+
                                                       |                                     |
                                                      yes                                    no
                                                       |                                     |
                                          +------------v-------------+        +--------------v---------------+
+-----------------------------+           |                          |        |                              |
|   Sender can guarantee      |           |     Message is direct    |        |                              |
|  to nil out reference to   <-----yes----+  response to method call <---no---+      Multiple recipients     |
|     a callback block?       |           |      (call back)?        |        |                              |
+-------------+----+----------+           |                          |        |                              |
              |    |                      +------------+-------------+        +--------------+---------------+
             yes   |                                   |                                     |
              +    +----no--------------------+        no                                   yes
              |                               |        |                                     |
+-------------v----------------+          +---v--------v-------------+        +--------------v---------------+
|                              |          |                          |        |                              |
|            Block             |          |         Delegation       |        |          Notification        |
|                              |          |                          |        |                              |
+------------------------------+          +--------------------------+        +------------------------------+

```

## 模块化 & URL Route
// TODO: