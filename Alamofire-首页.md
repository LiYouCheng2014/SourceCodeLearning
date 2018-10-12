

Alamofire是一个用Swift编写的HTTP网络库。

- [特征](https://github.com/Alamofire/Alamofire#features)
- [组件库](https://github.com/Alamofire/Alamofire#component-libraries)
- [要求](https://github.com/Alamofire/Alamofire#requirements)
- [迁移指南](https://github.com/Alamofire/Alamofire#migration-guides)
- [通讯](https://github.com/Alamofire/Alamofire#communication)
- [安装](https://github.com/Alamofire/Alamofire#installation)
- 用法
  - **简介 -** [发出请求](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#making-a-request)，[响应处理](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#response-handling)，[响应验证](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#response-validation)，[响应缓存](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#response-caching)
  - **HTTP -** [HTTP方法](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#http-methods)，[参数编码](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#parameter-encoding)，[HTTP标头](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#http-headers)，[身份验证](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#authentication)
  - **大数据 -** [将数据](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#uploading-data-to-a-server)[下载到文件](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#downloading-data-to-a-file)，[将数据上传到服务器](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#uploading-data-to-a-server)
  - **工具 -** [统计指标](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#statistical-metrics)，[cURL命令输出](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#curl-command-output)
- 高级用法
  - **URL会话 -** [会话管理器](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#session-manager)，[会话代理](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#session-delegate)，[请求](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#request)
  - **路由 -** [路由请求](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#routing-requests)，[调整和重试请求](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#adapting-and-retrying-requests)
  - **模型对象 -** [自定义响应序列化](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#custom-response-serialization)
  - **连接 -** [安全性](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#security)，[网络可达性](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#network-reachability)
- [打开雷达](https://github.com/Alamofire/Alamofire#open-radars)
- [常问问题](https://github.com/Alamofire/Alamofire#faq)
- [积分](https://github.com/Alamofire/Alamofire#credits)
- [捐赠](https://github.com/Alamofire/Alamofire#donations)
- [执照](https://github.com/Alamofire/Alamofire#license)

## 特征

-  可链接的请求/响应方法
-  URL / JSON / plist参数编码
-  上传文件/数据/流/ MultipartFormData
-  使用请求或恢复数据下载文件
-  使用URLCredential进行身份验证
-  HTTP响应验证
-  上传和下载进度闭包与进度
-  cURL命令输出
-  动态调整和重试请求
-  TLS证书和公钥固定
-  网络可达性
-  综合单元和集成测试覆盖范围
-  [完整文档](https://alamofire.github.io/Alamofire)

## 组件库

为了使Alamofire专注于核心网络实施，[Alamofire软件基金会](https://github.com/Alamofire/Foundation)创建了额外的组件库，为Alamofire生态系统带来了额外的功能。

- [AlamofireImage](https://github.com/Alamofire/AlamofireImage) - 包含图像响应序列化器`UIImage`和`UIImageView`扩展，自定义图像过滤器，自动清除内存缓存和基于优先级的图像下载系统的图像库。
- [AlamofireNetworkActivityIndicator](https://github.com/Alamofire/AlamofireNetworkActivityIndicator) - 使用Alamofire控制iOS上网络活动指示器的可见性。它包含可配置的延迟计时器，有助于缓解闪烁，并且可以支持`URLSession`非Alamofire管理的实例。

## 要求

- iOS 8.0+ / macOS 10.10+ / tvOS 9.0+ / watchOS 2.0+
- Xcode 8.3+
- Swift 3.1+

## 迁移指南

- [Alamofire 4.0迁移指南](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Alamofire%204.0%20Migration%20Guide.md)
- [Alamofire 3.0迁移指南](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Alamofire%203.0%20Migration%20Guide.md)
- [Alamofire 2.0迁移指南](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Alamofire%202.0%20Migration%20Guide.md)

## 通讯

- 如果您**需要有关网络请求的帮助，请**使用[Stack Overflow](https://stackoverflow.com/questions/tagged/alamofire)和tag `alamofire`。
- 如果您需要**查找或了解API**，请查看[我们的文档](http://alamofire.github.io/Alamofire/)或[Apple的文档`URLSession`](https://developer.apple.com/documentation/foundation/url_loading_system)，在其上构建Alamofire。
- 如果您需要**有关Alamofire功能的帮助**，请使用[swift.org上的论坛](https://forums.swift.org/c/related-projects/alamofire)。
- 如果您想**讨论Alamofire最佳实践**，请[在swift.org上](https://forums.swift.org/c/related-projects/alamofire)使用[我们的论坛](https://forums.swift.org/c/related-projects/alamofire)。
- 如果您想**讨论功能请求**，请使用[swift.org上的论坛](https://forums.swift.org/c/related-projects/alamofire)。
- 如果您**发现了错误**，请打开问题并按照指南操作。越详细越好！
- 如果您**想贡献**，请提交拉取请求。

## 安装

### 的CocoaPods

[CocoaPods](https://cocoapods.org/)是Cocoa项目的依赖管理器。您可以使用以下命令安装它：

```
$ gem install cocoapods
```

> 构建Alamofire 4.0+需要CocoaPods 1.1+。

要使用CocoaPods将Alamofire集成到您的Xcode项目中，请在以下位置指定`Podfile`：

```
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '10.0'
use_frameworks!

target '<Your Target Name>' do
    pod 'Alamofire', '~> 4.7'
end
```

然后，运行以下命令：

```
$ pod install
```

### 迦太基

[Carthage](https://github.com/Carthage/Carthage)是一个分散的依赖管理器，它构建您的依赖项并为您提供二进制框架。

您可以使用以下命令安装Carthage with [Homebrew](https://brew.sh/)：

```
$ brew update
$ brew install carthage
```

要使用Carthage将Alamofire集成到您的Xcode项目中，请在以下位置指定`Cartfile`：

```
github "Alamofire/Alamofire" ~> 4.7
```

运行`carthage update`以构建框架并将内置拖动`Alamofire.framework`到Xcode项目中。

### Swift包管理器

在[斯威夫特软件包管理器](https://swift.org/package-manager/)是自动化的银行代码分配的工具，被集成到`swift`编译器。它处于早期开发阶段，但Alamofire支持在支持的平台上使用它。

一旦你设置了Swift软件包，添加Alamofire作为依赖项就像添加它的`dependencies`价值一样简单`Package.swift`。

#### Swift 3

```
dependencies: [
    .Package(url: "https://github.com/Alamofire/Alamofire.git", majorVersion: 4)
]
```

#### Swift 4

```
dependencies: [
    .package(url: "https://github.com/Alamofire/Alamofire.git", from: "4.0.0")
]
```

### 手动

如果您不想使用上述任何依赖管理器，可以手动将Alamofire集成到您的项目中。

#### 嵌入式框架

- 打开Terminal，`cd`进入您的顶级项目目录，然后运行以下命令“if”您的项目未初始化为git存储库：

  ```
  $ git init
  ```

- 通过运行以下命令将Alamofire添加为git [子模块](https://git-scm.com/docs/git-submodule)：

  ```
  $ git submodule add https://github.com/Alamofire/Alamofire.git
  ```

- 打开新`Alamofire`文件夹，然后将其拖到`Alamofire.xcodeproj`应用程序Xcode项目的Project Navigator中。

  > 它应该嵌套在应用程序的蓝色项目图标下面。无论是高于还是低于所有其他Xcode组并不重要。

- `Alamofire.xcodeproj`在Project Navigator中选择并验证部署目标是否与应用程序目标的目标匹配。

- 接下来，在Project Navigator中选择您的应用程序项目（蓝色项目图标）以导航到目标配置窗口，并在侧栏中的“Targets”标题下选择应用程序目标。

- 在该窗口顶部的标签栏中，打开“常规”面板。

- 单击`+`“嵌入式二进制文件”部分下的按钮。

- 您将看到两个不同的`Alamofire.xcodeproj`文件夹，每个文件夹都有两个不同版本的`Alamofire.framework`嵌套在`Products`文件夹中。

  > `Products`您选择哪个文件夹无关紧要，但无论您选择顶部还是底部都无关紧要`Alamofire.framework`。

- 选择`Alamofire.framework`iOS 的顶部，OS X的底部。

  > 您可以通过检查项目的构建日志来验证您选择的那个。为构建目标`Alamofire`将被列为要么`Alamofire iOS`，`Alamofire macOS`，`Alamofire tvOS`或`Alamofire watchOS`。

- 就是这样！

  > 该`Alamofire.framework`是自动添加为目标的依赖，链接框架和嵌入式框架，在文件拷贝建造阶段这是所有你需要建立在模拟器和设备。

## 打开雷达

以下雷达对Alamofire的当前实施有一定影响。

- [`rdar://21349340`](http://www.openradar.me/radar?id=5517037090635776) - 由于测试用例中的免费桥接问题，编译器抛出警告
- `rdar://26870455` - 后台URL会话配置在模拟器中不起作用
- `rdar://26849668` - 某些URLProtocol API无法正确处理 `URLRequest`
- [`rdar://36082113`](http://openradar.appspot.com/radar?id=4942308441063424)- `URLSessionTaskMetrics`未能在watchOS 3.0+上链接

## 已解决的雷达

在针对Alamofire项目提出申请后，以下雷达已经解决了。

- [`rdar://26761490`](http://www.openradar.me/radar?id=5010235949318144) - Swift字符串插值导致内存泄漏，常见用法（在Xcode 9 beta 6中于9/1/17解决）。

## 常问问题

### Alamofire这个名字的起源是什么？

Alamofire以[Alamo Fire花](https://aggie-horticulture.tamu.edu/wildseed/alamofire.html)命名，[Alamo Fire](https://aggie-horticulture.tamu.edu/wildseed/alamofire.html)花是得克萨斯州官方州花Bluebonnet的混合变体。

### 路由器与请求适配器中的逻辑属于什么？

简单的静态数据（如路径，参数和公共头）属于`Router`。动态数据，例如`Authorization`其值可以根据认证系统而改变的标题属于a `RequestAdapter`。

动态数据必须放入的原因`RequestAdapter`是支持重试操作。当一个`Request`重试，原来的请求不被重建意味着`Router`不会再次调用。在`RequestAdapter`被称为再次允许动态数据上重试前的原始请求更新`Request`。

## 积分

Alamofire由[Alamofire Software Foundation](http://alamofire.org/)拥有和维护。您可以在Twitter上关注他们[@AlamofireSF](https://twitter.com/AlamofireSF)以获取项目更新和发布。

### 安全披露

如果您认为自己发现了Alamofire的安全漏洞，则应尽快通过电子邮件向[security@alamofire.org报告](mailto:security@alamofire.org)。请不要将其发布到公共问题跟踪器。

## 捐赠

该[ASF](https://github.com/Alamofire/Foundation#members)正在筹钱留正式注册为联邦非营利组织。注册将允许我们的成员获得一些法律保护，并允许我们捐赠使用，免税。捐赠给ASF将使我们能够：

- 支付我们的年度法律费用，以保持非营利组织的良好状态
- 支付我们的邮件服务器，以帮助我们掌握所有问题和安全问题
- 可能为测试服务器提供资金，以便我们更轻松地测试边缘情况
- 可能资助开发人员全职工作我们的一个项目

社区对ASF库的采用令人惊叹。我们对您对项目的热情非常谦卑，并希望继续尽我们所能推动未来发展。在您的不断支持下，ASF将能够改善其覆盖范围，并为核心成员提供更好的法律保障。如果您使用我们的任何图书馆工作，请查看您的雇主是否有兴趣捐赠。您今天捐赠的任何金额都可以帮助我们实现目标，我们将不胜感激。

[![贝宝](https://camo.githubusercontent.com/f896f7d176663a1559376bb56aac4bdbbbe85ed1/68747470733a2f2f7777772e70617970616c6f626a656374732e636f6d2f656e5f55532f692f62746e2f62746e5f646f6e61746543435f4c472e676966)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=W34WPEE74APJQ)

## 执照

Alamofire根据MIT许可证发布。[有关](https://github.com/Alamofire/Alamofire/blob/master/LICENSE)详细信息，[请参阅LICENSE](https://github.com/Alamofire/Alamofire/blob/master/LICENSE)。