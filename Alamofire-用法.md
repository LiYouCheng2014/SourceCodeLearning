## 用法

### 发出请求

```swift
import Alamofire

Alamofire.request("https://httpbin.org/get")
```

### 响应处理

处理Alamofire中`Response`的一个`Request`制作涉及将响应处理程序链接到`Request`。

```swift
Alamofire.request("https://httpbin.org/get").responseJSON { response in
    print("Request: \(String(describing: response.request))")   // original url request
    print("Response: \(String(describing: response.response))") // http url response
    print("Result: \(response.result)")                         // response serialization result

    if let json = response.result.value {
        print("JSON: \(json)") // serialized json response
    }

    if let data = response.data, let utf8Text = String(data: data, encoding: .utf8) {
        print("Data: \(utf8Text)") // original server data as UTF8 string
    }
}
```

在上面的示例中，`responseJSON`处理程序附加到完成`Request`后要执行的处理程序`Request`。不是阻止执行以等待来自服务器的响应，而是指定闭包形式的[回调](https://en.wikipedia.org/wiki/Callback_%28computer_programming%29)以在收到响应后处理响应。请求的结果仅在响应闭包的范围内可用。根据响应或从服务器接收的数据执行的任何执行都必须在响应闭包内完成。

> Alamofire中的网络是*异步*完成的。对于不熟悉这个概念的程序员来说，异步编程可能会让人感到沮丧，但这样做有[很好的理由](https://developer.apple.com/library/ios/qa/qa1693/_index.html)。

Alamofire默认包含五种不同的响应处理程序，包括：

```swift
//响应处理程序 - 反序列化响应
func response(
    queue: DispatchQueue?,
    completionHandler: @escaping (DefaultDataResponse) -> Void)
    -> Self

//响应数据处理程序 - 序列化为数据
func  responseData（
     queue：DispatchQueue ? ,
     completionHandler：@escaping（DataResponse <Data>）- >  Void）
     - >  Self

//响应字符串处理程序 - 序列化为字符串
func  responseString（
     queue：DispatchQueue ? ,
     encoding：String .Encoding ？，
     completionHandler：@escaping（DataResponse < String >）- >  Void）
     - >  Self

//响应JSON处理程序 - 序列化为任意
func  responseJSON（
     queue：DispatchQueue ? ,
     completionHandler：@escaping（DataResponse < Any >）- >  Void）
     - >  Self

//响应PropertyList（plist）处理程序 - 序列化为Any
func  responsePropertyList（
     queue：DispatchQueue ? ,
     completionHandler：@escaping（DataResponse < Any >）- >  Void））
     - >  Self
```

没有任何响应处理程序执行`HTTPURLResponse`从服务器返回的任何验证。

> 例如，`400..<500`和`500..<600`范围中的响应状态代码不会自动触发`Error`。Alamofire使用[响应验证](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#response-validation)方法链接来实现此目的。

#### 响应处理程序

该`response`处理程序不评价任何响应数据的。它只是直接从URL会话委托转发所有信息。这是Alamofire相当于`cURL`用来执行一个`Request`。

```swift
Alamofire.request("https://httpbin.org/get").response { response in
    print("Request: \(response.request)")
    print("Response: \(response.response)")
    print("Error: \(response.error)")

    if let data = response.data, let utf8Text = String(data: data, encoding: .utf8) {
    	print("Data: \(utf8Text)")
    }
}
```

> 我们强烈建议您利用其他响应序列化程序来利用`Response`和`Result`类型。

#### 响应数据处理程序

该`responseData`处理程序使用`responseDataSerializer`（服务器数据序列化到一些其它类型的对象），以提取`Data`由服务器返回。如果没有错误发生并`Data`返回，则响应`Result`将是a `.success`，并且`value`将是类型`Data`。

```swift
Alamofire.request("https://httpbin.org/get").responseData { response in
    debugPrint("All Response Info: \(response)")

    if let data = response.result.value, let utf8Text = String(data: data, encoding: .utf8) {
    	print("Data: \(utf8Text)")
    }
}
```

#### 响应字符串处理程序

该`responseString`处理器采用了`responseStringSerializer`转换的`Data`服务器返回到`String`与指定的编码。如果没有发生错误并且服务器数据成功序列化为a `String`，则响应`Result`将为a `.success`且`value`类型为`String`。

```swift
Alamofire.request("https://httpbin.org/get").responseString { response in
    print("Success: \(response.result.isSuccess)")
    print("Response String: \(response.result.value)")
}
```

> 如果未指定编码，Alamofire将使用`HTTPURLResponse`服务器中指定的文本编码。如果服务器响应无法确定文本编码，则默认为`.isoLatin1`。

#### 响应JSON处理程序

该`responseJSON`处理器采用了`responseJSONSerializer`转换的`Data`服务器返回到`Any`使用指定的类型`JSONSerialization.ReadingOptions`。如果没有发生错误并且服务器数据成功序列化为JSON对象，则响应`Result`将为a `.success`且`value`类型为`Any`。

```swift
Alamofire.request("https://httpbin.org/get").responseJSON { response in
    debugPrint(response)

    if let json = response.result.value {
        print("JSON: \(json)")
    }
}
```

> 所有JSON序列化都由框架中的`JSONSerialization`API 处理`Foundation`。

#### 链式响应处理程序

响应处理程序甚至可以链接：

```swift
Alamofire.request("https://httpbin.org/get")
    .responseString { response in
        print("Response String: \(response.result.value)")
    }
    .responseJSON { response in
        print("Response JSON: \(response.result.value)")
    }
```

> 重要的是要注意，在同一个上使用多个响应处理程序`Request`需要多次序列化服务器数据。每个响应处理程序一次。

#### 响应处理程序队列

默认情况下，响应处理程序在主调度队列上执行。但是，可以提供自定义调度队列。

```swift
let utilityQueue = DispatchQueue.global(qos: .utility)

Alamofire.request("https://httpbin.org/get").responseJSON(queue: utilityQueue) { response in
    print("Executing response handler on utility queue")
}
```

### 响应验证

默认情况下，无论响应的内容如何，Alamofire都会将任何已完成的请求视为成功。`validate`如果响应具有不可接受的状态代码或MIME类型，则在响应处理程序之前调用会导致生成错误。

#### 手动验证

```swift
Alamofire.request("https://httpbin.org/get")
    .validate(statusCode: 200..<300)
    .validate(contentType: ["application/json"])
    .responseData { response in
        switch response.result {
        case .success:
            print("Validation Successful")
        case .failure(let error):
            print(error)
        }
    }
```

#### 自动验证

自动验证`200..<300`范围内的状态代码，并且`Content-Type`响应的`Accept`标头与请求的标头匹配（如果提供了标头）。

```swift
Alamofire.request("https://httpbin.org/get").validate().responseJSON { response in
    switch response.result {
    case .success:
        print("Validation Successful")
    case .failure(let error):
        print(error)
    }
}
```

### 响应缓存

响应缓存在系统框架级别上处理[`URLCache`](https://developer.apple.com/reference/foundation/urlcache)。它提供了复合内存和磁盘缓存，并允许您操作内存和磁盘部分的大小。

> 默认情况下，Alamofire利用共享`URLCache`。要自定义它，请参阅“ [会话管理器配置”](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#session-manager)部分。

### HTTP方法

该`HTTPMethod`枚举列出定义的HTTP方法[RFC 7231第4.3节](https://tools.ietf.org/html/rfc7231#section-4.3)：

```swift
public enum HTTPMethod: String {
    case options = "OPTIONS"
    case get     = "GET"
    case head    = "HEAD"
    case post    = "POST"
    case put     = "PUT"
    case patch   = "PATCH"
    case delete  = "DELETE"
    case trace   = "TRACE"
    case connect = "CONNECT"
}
```

这些值可以作为`method`参数传递给`Alamofire.request`API：

```swift
Alamofire.request("https://httpbin.org/get") // method defaults to `.get`

Alamofire.request("https://httpbin.org/post", method: .post)
Alamofire.request("https://httpbin.org/put", method: .put)
Alamofire.request("https://httpbin.org/delete", method: .delete)
```

> 所述`Alamofire.request`方法参数的缺省值为`.get`。

### 参数编码

Alamofire支持三种类型的参数编码，包括：`URL`，`JSON`和`PropertyList`。它还可以支持任何符合`ParameterEncoding`协议的自定义编码。

#### 网址编码

该`URLEncoding`类型创建一个URL编码的查询字符串，该字符串将被设置为或附加到任何现有的URL查询字符串，或者设置为URL请求的HTTP正文。查询字符串是设置还是附加到任何现有URL查询字符串或设置为HTTP正文取决于`Destination`编码。该`Destination`枚举有三种情况：

- `.methodDependent`- 将编码的查询字符串结果应用于现有查询字符串`GET`，`HEAD`并将`DELETE`请求和集合应用为使用任何其他HTTP方法的请求的HTTP正文。
- `.queryString` - 将编码查询字符串结果设置或附加到现有查询字符串。
- `.httpBody` - 将编码的查询字符串结果设置为URL请求的HTTP正文。

`Content-Type`HTTP主体的编码请求的HTTP头字段设置为`application/x-www-form-urlencoded; charset=utf-8`。由于没有关于如何编码集合类型的已发布规范，因此附加`[]`了数组值（`foo[]=1&foo[]=2`）的键，以及为嵌套字典值（`foo[bar]=baz`）附加方括号括起来的键的约定。

##### 使用URL编码参数获取请求

```swift
let parameters: Parameters = ["foo": "bar"]

// All three of these calls are equivalent
Alamofire.request("https://httpbin.org/get", parameters: parameters) // encoding defaults to `URLEncoding.default`
Alamofire.request("https://httpbin.org/get", parameters: parameters, encoding: URLEncoding.default)
Alamofire.request("https://httpbin.org/get", parameters: parameters, encoding: URLEncoding(destination: .methodDependent))

// https://httpbin.org/get?foo=bar
```

##### 使用URL编码参数的POST请求

```swift
let parameters: Parameters = [
    "foo": "bar",
    "baz": ["a", 1],
    "qux": [
        "x": 1,
        "y": 2,
        "z": 3
    ]
]

// All three of these calls are equivalent
Alamofire.request("https://httpbin.org/post", method: .post, parameters: parameters)
Alamofire.request("https://httpbin.org/post", method: .post, parameters: parameters, encoding: URLEncoding.default)
Alamofire.request("https://httpbin.org/post", method: .post, parameters: parameters, encoding: URLEncoding.httpBody)

// HTTP body: foo=bar&baz[]=a&baz[]=1&qux[x]=1&qux[y]=2&qux[z]=3
```

##### 配置`Bool`参数编码

该`URLEncoding.BoolEncoding`枚举提供了编码以下方法`Bool`参数：

- `.numeric`-编码`true`作为`1`和`false`作为`0`。
- `.literal`- 编码`true`和`false`字符串文字。

默认情况下，Alamofire使用`.numeric`编码。

您可以创建自己的`URLEncoding`并`Bool`在初始化程序中指定所需的编码：

```swift
let encoding = URLEncoding(boolEncoding: .literal)
```

##### 配置`Array`参数编码

该`URLEncoding.ArrayEncoding`枚举提供了编码以下方法`Array`参数：

- `.brackets` - 每个值的键都附加一组空方括号。
- `.noBrackets` - 不附加括号。密钥按原样编码。

默认情况下，Alamofire使用`.brackets`编码，`foo=[1,2]`编码为`foo[]=1&foo[]=2`。

使用`.noBrackets`编码将编码`foo=[1,2]`为`foo=1&foo=2`。

您可以创建自己的`URLEncoding`并`Array`在初始化程序中指定所需的编码：

```swift
let encoding = URLEncoding(arrayEncoding: .noBrackets)
```

#### JSON编码

该`JSONEncoding`类型创建参数对象的JSON表示，该表示被设置为请求的HTTP主体。`Content-Type`编码请求的HTTP头字段设置为`application/json`。

##### 使用JSON编码参数的POST请求

```swift
let parameters: Parameters = [
    "foo": [1,2,3],
    "bar": [
        "baz": "qux"
    ]
]

// Both calls are equivalent
Alamofire.request("https://httpbin.org/post", method: .post, parameters: parameters, encoding: JSONEncoding.default)
Alamofire.request("https://httpbin.org/post", method: .post, parameters: parameters, encoding: JSONEncoding(options: []))

// HTTP body: {"foo": [1, 2, 3], "bar": {"baz": "qux"}}
```

#### Plist编码

的`PropertyListEncoding`用途`PropertyListSerialization`来创建参数对象的plist表示时，根据相关联的格式，并写入选项的值，其被设置为请求的主体。`Content-Type`编码请求的HTTP头字段设置为`application/x-plist`。

#### 自定义编码

如果提供的`ParameterEncoding`类型不符合您的需求，您可以创建自己的自定义编码。这是一个快速示例，说明如何构建自定义`JSONStringArrayEncoding`类型以将JSON字符串数组编码到`Request`。

```swift
struct JSONStringArrayEncoding: ParameterEncoding {
    private let array: [String]

    init(array: [String]) {
        self.array = array
    }

    func encode(_ urlRequest: URLRequestConvertible, with parameters: Parameters?) throws -> URLRequest {
        var urlRequest = try urlRequest.asURLRequest()

        let data = try JSONSerialization.data(withJSONObject: array, options: [])

        if urlRequest.value(forHTTPHeaderField: "Content-Type") == nil {
            urlRequest.setValue("application/json", forHTTPHeaderField: "Content-Type")
        }

        urlRequest.httpBody = data

        return urlRequest
    }
}
```

#### URLRequest的手动参数编码

该`ParameterEncoding`API可让网络请求之外使用。

```swift
let url = URL(string: "https://httpbin.org/get")!
var urlRequest = URLRequest(url: url)

let parameters: Parameters = ["foo": "bar"]
let encodedURLRequest = try URLEncoding.queryString.encode(urlRequest, with: parameters)
```

### HTTP头部

`Request`在global `request`方法中直接支持向a添加自定义HTTP标头。这样可以轻松地将HTTP标头附加到`Request`可以不断更改的标头上。

```swift
let headers: HTTPHeaders = [
    "Authorization": "Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==",
    "Accept": "application/json"
]

Alamofire.request("https://httpbin.org/headers", headers: headers).responseJSON { response in
    debugPrint(response)
}
```

> 对于不更改的HTTP标头，建议将它们设置为，`URLSessionConfiguration`以便它们自动应用于`URLSessionTask`底层创建的任何标头`URLSession`。有关更多信息，请参阅[会话管理器配置](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#session-manager)部分。

默认的Alamofire `SessionManager`为每个提供默认的标头集`Request`。这些包括：

- `Accept-Encoding`，默认为`gzip;q=1.0, compress;q=0.5`，每[RFC 7230§4.2.3](https://tools.ietf.org/html/rfc7230#section-4.2.3)。
- `Accept-Language`，缺省到顶部6偏好的语言在系统上，格式化等`en;q=1.0`，每[RFC 7231§5.3.5](https://tools.ietf.org/html/rfc7231#section-5.3.5)。
- `User-Agent`，其中包含有关当前应用程序的版本信息。例如：`iOS Example/1.0 (com.alamofire.iOS-Example; build:1; iOS 10.0.0) Alamofire/4.0.0`，根据[RFC7231§5.5.3](https://tools.ietf.org/html/rfc7231#section-5.5.3)。

如果需要自定义这些标头，`URLSessionConfiguration`则应创建自定义，`defaultHTTPHeaders`更新属性并将配置应用于新`SessionManager`实例。

### 认证

身份验证在系统框架级别由[`URLCredential`](https://developer.apple.com/reference/foundation/nsurlcredential)和处理[`URLAuthenticationChallenge`](https://developer.apple.com/reference/foundation/urlauthenticationchallenge)。

**支持的身份验证方案**

- [HTTP基础](https://en.wikipedia.org/wiki/Basic_access_authentication)
- [HTTP摘要](https://en.wikipedia.org/wiki/Digest_access_authentication)
- [Kerberos的](https://en.wikipedia.org/wiki/Kerberos_%28protocol%29)
- [NTLM](https://en.wikipedia.org/wiki/NT_LAN_Manager)

#### HTTP基本身份验证

在`authenticate`上一个方法`Request`自动将提供`URLCredential`给一个`URLAuthenticationChallenge`适当的时候：

```swift
let user = "user"
let password = "password"

Alamofire.request("https://httpbin.org/basic-auth/\(user)/\(password)")
    .authenticate(user: user, password: password)
    .responseJSON { response in
        debugPrint(response)
    }
```

根据您的服务器实现，`Authorization`标头也可能是合适的：

```swift
let user = "user"
let password = "password"

var headers: HTTPHeaders = [:]

if let authorizationHeader = Request.authorizationHeader(user: user, password: password) {
    headers[authorizationHeader.key] = authorizationHeader.value
}

Alamofire.request("https://httpbin.org/basic-auth/user/password", headers: headers)
    .responseJSON { response in
        debugPrint(response)
    }
```

#### 使用URLCredential进行身份验证

```swift
let user = "user"
let password = "password"

let credential = URLCredential(user: user, password: password, persistence: .forSession)

Alamofire.request("https://httpbin.org/basic-auth/\(user)/\(password)")
    .authenticate(usingCredential: credential)
    .responseJSON { response in
        debugPrint(response)
    }
```

> 值得注意的是，当使用`URLCredential`for进行身份验证时，`URLSession`如果服务器发出质询，则底层实际上最终会发出两个请求。第一个请求将不包括“可能”从服务器触发质询的凭证。然后，Alamofire接收挑战，附加凭证并由底层重试请求`URLSession`。

### 将数据下载到文件

在Alamofire中从服务器获取数据的请求可以下载内存中或磁盘上的数据。`Alamofire.request`到目前为止，所有示例中使用的API始终在内存中下载服务器数据。这对于较小的有效负载来说非常有用，因为它更有效，但对于较大的有效负载却非常糟糕，因为下载可以使整个应用程序运行在内存之外。因此，您还可以使用`Alamofire.download`API将服务器数据下载到磁盘上的临时文件。

> 这只能按现状运行`macOS`。其他平台不允许访问应用程序沙箱之外的文件系统。要在其他平台上下载文件，请参阅“ [下载文件目标”](https://github.com/Alamofire/Alamofire/blob/master/Documentation/Usage.md#download-file-destination)部分。

```swift
Alamofire.download("https://httpbin.org/image/png").responseData { response in
    if let data = response.result.value {
        let image = UIImage(data: data)
    }
}
```

> 该`Alamofire.download`API还应该如果你需要下载数据，同时您的应用程序在后台使用。有关更多信息，请参阅[会话管理器配置](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#session-manager)部分。

#### 下载文件目的地

您还可以提供`DownloadFileDestination`闭包以将文件从临时目录移动到最终目标。在临时文件实际移动到之前`destinationURL`，`DownloadOptions`将执行闭包中指定的内容。目前支持的两个`DownloadOptions`是：

- `.createIntermediateDirectories` - 如果指定，则为目标URL创建中间目录。
- `.removePreviousFile` - 如果指定，则从目标URL中删除以前的文件。

```swift
let destination: DownloadRequest.DownloadFileDestination = { _, _ in
    let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
    let fileURL = documentsURL.appendingPathComponent("pig.png")

    return (fileURL, [.removePreviousFile, .createIntermediateDirectories])
}

Alamofire.download(urlString, to: destination).response { response in
    print(response)

    if response.error == nil, let imagePath = response.destinationURL?.path {
        let image = UIImage(contentsOfFile: imagePath)
    }
}
```

您还可以使用建议的下载目标API。

```swift
let destination = DownloadRequest.suggestedDownloadDestination(for: .documentDirectory)
Alamofire.download("https://httpbin.org/image/png", to: destination)
```

#### 下载进度

很多时候向用户报告下载进度会很有帮助。任何人`DownloadRequest`都可以使用`downloadProgress`API 报告下载进度。

```swift
Alamofire.download("https://httpbin.org/image/png")
    .downloadProgress { progress in
        print("Download Progress: \(progress.fractionCompleted)")
    }
    .responseData { response in
        if let data = response.result.value {
            let image = UIImage(data: data)
        }
    }
```

该`downloadProgress`API也需要`queue`定义哪个参数`DispatchQueue`的下载进度闭合应该叫上。

```swift
let utilityQueue = DispatchQueue.global(qos: .utility)

Alamofire.download("https://httpbin.org/image/png")
    .downloadProgress(queue: utilityQueue) { progress in
        print("Download Progress: \(progress.fractionCompleted)")
    }
    .responseData { response in
        if let data = response.result.value {
            let image = UIImage(data: data)
        }
    }
```

#### 恢复下载

如果a `DownloadRequest`被取消或中断，则基础URL会话可以生成活动的恢复数据`DownloadRequest`。如果发生这种情况，可以重新使用恢复数据重新启动`DownloadRequest`它停止的位置。可以通过下载响应访问恢复数据，然后在尝试重新启动请求时重用。

> **重要提示：**在所有Apple平台的某些版本（iOS 10 - 10.2，macOS 10.12 - 10.12.2，tvOS 10 - 10.1，watchOS 3 - 3.1.1）`resumeData`上，后台URL会话配置中断。在`resumeData`生成逻辑中存在一个潜在的错误，其中数据写错了，并且总是无法恢复下载。有关错误和可能的解决方法的更多信息，请参阅此[Stack Overflow帖子](https://stackoverflow.com/a/39347461/1342462)。

```swift
class ImageRequestor {
    private var resumeData: Data?
    private var image: UIImage?

    func fetchImage(completion: (UIImage?) -> Void) {
        guard image == nil else { completion(image) ; return }

        let destination: DownloadRequest.DownloadFileDestination = { _, _ in
            let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
            let fileURL = documentsURL.appendingPathComponent("pig.png")

            return (fileURL, [.removePreviousFile, .createIntermediateDirectories])
        }

        let request: DownloadRequest

        if let resumeData = resumeData {
            request = Alamofire.download(resumingWith: resumeData)
        } else {
            request = Alamofire.download("https://httpbin.org/image/png")
        }

        request.responseData { response in
            switch response.result {
            case .success(let data):
                self.image = UIImage(data: data)
            case .failure:
                self.resumeData = response.resumeData
            }
        }
    }
}
```

### 将数据上传到服务器

使用JSON或URL编码参数向服务器发送相对少量的数据时，`Alamofire.request`API通常就足够了。如果您需要从文件URL或文件URL发送更大量的数据`InputStream`，那么`Alamofire.upload`API就是您要使用的。

> 该`Alamofire.upload`API还应该如果你需要上传的数据，同时您的应用程序在后台使用。有关更多信息，请参阅[会话管理器配置](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#session-manager)部分。

#### 上传数据

```swift
let imageData = UIImagePNGRepresentation(image)!

Alamofire.upload(imageData, to: "https://httpbin.org/post").responseJSON { response in
    debugPrint(response)
}
```

#### 上传文件

```swift
let fileURL = Bundle.main.url(forResource: "video", withExtension: "mov")

Alamofire.upload(fileURL, to: "https://httpbin.org/post").responseJSON { response in
    debugPrint(response)
}
```

#### 上传多部分表单数据

```swift
Alamofire.upload(
    multipartFormData: { multipartFormData in
        multipartFormData.append(unicornImageURL, withName: "unicorn")
        multipartFormData.append(rainbowImageURL, withName: "rainbow")
    },
    to: "https://httpbin.org/post",
    encodingCompletion: { encodingResult in
    	switch encodingResult {
    	case .success(let upload, _, _):
            upload.responseJSON { response in
                debugPrint(response)
            }
    	case .failure(let encodingError):
    	    print(encodingError)
    	}
    }
)
```

#### 上传进度

当您的用户等待上传完成时，有时可以方便地向用户显示上传的进度。任何人`UploadRequest`都可以使用`uploadProgress`和`downloadProgress`API 报告上传进度和响应数据的下载进度。

```swift
let fileURL = Bundle.main.url(forResource: "video", withExtension: "mov")

Alamofire.upload(fileURL, to: "https://httpbin.org/post")
    .uploadProgress { progress in // main queue by default
        print("Upload Progress: \(progress.fractionCompleted)")
    }
    .downloadProgress { progress in // main queue by default
        print("Download Progress: \(progress.fractionCompleted)")
    }
    .responseJSON { response in
        debugPrint(response)
    }
```

### 统计指标

#### 时间线

Alamofire在a的整个生命周期中收集时间，`Request`并创建`Timeline`在所有响应类型上作为属性公开的对象。

```swift
Alamofire.request("https://httpbin.org/get").responseJSON { response in
    print(response.timeline)
}
```

以上报告以下`Timeline`信息：

- `Latency`：0.428秒
- `Request Duration`：0.428秒
- `Serialization Duration`：0.001秒
- `Total Duration`：0.429秒

#### URL会话任务度量标准

在iOS和tvOS 10以及macOS 10.12中，Apple引入了新的[URLSessionTaskMetrics](https://developer.apple.com/reference/foundation/urlsessiontaskmetrics) API。任务指标封装了一些关于请求和响应执行的精彩统计信息。API非常类似`Timeline`，但提供了Alamofire无法计算的更多统计信息。可以通过任何响应类型访问度量标准。

```swift
Alamofire.request("https://httpbin.org/get").responseJSON { response in
    print(response.metrics)
}
```

值得注意的是，这些API仅适用于iOS和tvOS 10以及macOS 10.12。因此，根据您的部署目标，您可能需要使用这些内部可用性检查：

```swift
Alamofire.request("https://httpbin.org/get").responseJSON { response in
    if #available(iOS 10.0, *) {
        print(response.metrics)
    }
}
```

### cURL命令输出

调试平台问题可能令人沮丧。值得庆幸的是，Alamofire `Request`对象符合协议`CustomStringConvertible`和`CustomDebugStringConvertible`协议，以提供一些非常有用的调试工具。

#### CustomStringConvertible

```swift
let request = Alamofire.request("https://httpbin.org/ip")

print(request)
// GET https://httpbin.org/ip (200)
```

#### CustomDebugStringConvertible

```swift
let request = Alamofire.request("https://httpbin.org/get", parameters: ["foo": "bar"])
debugPrint(request)
```

输出：

```swift
$ curl -i \
    -H "User-Agent: Alamofire/4.0.0" \
    -H "Accept-Encoding: gzip;q=1.0, compress;q=0.5" \
    -H "Accept-Language: en;q=1.0,fr;q=0.9,de;q=0.8,zh-Hans;q=0.7,zh-Hant;q=0.6,ja;q=0.5" \
    "https://httpbin.org/get?foo=bar"
```