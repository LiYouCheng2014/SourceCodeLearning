## 高级用法

Alamofire建立在`URLSession`Foundation URL加载系统之上。要充分利用此框架，建议您熟悉底层网络堆栈的概念和功能。

**推荐阅读**

- [URL加载系统编程指南](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html)
- [URLSession类参考](https://developer.apple.com/reference/foundation/nsurlsession)
- [URLCache类参考](https://developer.apple.com/reference/foundation/urlcache)
- [URLAuthenticationChallenge类参考](https://developer.apple.com/reference/foundation/urlauthenticationchallenge)

### 会话管理器

顶级便捷方法，例如`Alamofire.request`使用默认实例`Alamofire.SessionManager`配置的默认实例`URLSessionConfiguration`。

因此，以下两个陈述是等效的：

```swift
Alamofire.request("https://httpbin.org/get")

let sessionManager = Alamofire.SessionManager.default
sessionManager.request("https://httpbin.org/get")
```

应用程序可以为后台和短暂会话创建会话管理器，以及自定义默认会话配置的新管理器，例如默认标头（`httpAdditionalHeaders`）或超时间隔（`timeoutIntervalForRequest`）。

#### 使用默认配置创建会话管理器

```swift
let configuration = URLSessionConfiguration.default
let sessionManager = Alamofire.SessionManager(configuration: configuration)
```

#### 使用后台配置创建会话管理器

```swift
let configuration = URLSessionConfiguration.background(withIdentifier: "com.example.app.background")
let sessionManager = Alamofire.SessionManager(configuration: configuration)
```

#### 使用临时配置创建会话管理器

```swift
let configuration = URLSessionConfiguration.ephemeral
let sessionManager = Alamofire.SessionManager(configuration: configuration)
```

#### 修改会话配置

```swift
var defaultHeaders = Alamofire.SessionManager.defaultHTTPHeaders
defaultHeaders["DNT"] = "1 (Do Not Track Enabled)"

let configuration = URLSessionConfiguration.default
configuration.httpAdditionalHeaders = defaultHeaders

let sessionManager = Alamofire.SessionManager(configuration: configuration)
```

> 这是**不**推荐`Authorization`或`Content-Type`头。相反，使用`headers`参数在顶层`Alamofire.request`的API，`URLRequestConvertible`并`ParameterEncoding`分别。

### 会话代表

默认情况下，Alamofire `SessionManager`实例创建一个`SessionDelegate`对象来处理由底层生成的所有各种类型的委托回调`URLSession`。每个委托方法的实现处理这些类型的调用的最常见用例，从顶层API中抽象出复杂性。但是，高级用户可能会因各种原因发现需要覆盖默认功能。

#### 覆盖闭包

自定义`SessionDelegate`行为的第一种方法是使用覆盖闭包。每个闭包使您能够覆盖匹配`SessionDelegate`API 的实现，但仍然使用所有其他API的默认实现。这样可以轻松自定义委托功能的子集。以下是一些可用的覆盖闭包的示例：

```swift
/// Overrides default behavior for URLSessionDelegate method `urlSession(_:didReceive:completionHandler:)`.
open var sessionDidReceiveChallenge: ((URLSession, URLAuthenticationChallenge) -> (URLSession.AuthChallengeDisposition, URLCredential?))?

/// Overrides default behavior for URLSessionDelegate method `urlSessionDidFinishEvents(forBackgroundURLSession:)`.
open var sessionDidFinishEventsForBackgroundURLSession: ((URLSession) -> Void)?

/// Overrides default behavior for URLSessionTaskDelegate method `urlSession(_:task:willPerformHTTPRedirection:newRequest:completionHandler:)`.
open var taskWillPerformHTTPRedirection: ((URLSession, URLSessionTask, HTTPURLResponse, URLRequest) -> URLRequest?)?

/// Overrides default behavior for URLSessionDataDelegate method `urlSession(_:dataTask:willCacheResponse:completionHandler:)`.
open var dataTaskWillCacheResponse: ((URLSession, URLSessionDataTask, CachedURLResponse) -> CachedURLResponse?)?
```

以下是如何使用`taskWillPerformHTTPRedirection`以避免重定向到任何`apple.com`域的简短示例。

```swift
let sessionManager = Alamofire.SessionManager(configuration: URLSessionConfiguration.default)
let delegate: Alamofire.SessionDelegate = sessionManager.delegate

delegate.taskWillPerformHTTPRedirection = { session, task, response, request in
    var finalRequest = request

    if
        let originalRequest = task.originalRequest,
        let urlString = originalRequest.url?.urlString,
        urlString.contains("apple.com")
    {
        finalRequest = originalRequest
    }

    return finalRequest
}
```

#### 子类

覆盖默认实现的另一种方法`SessionDelegate`是将其子类化。子类化允许您完全自定义API的行为或为API创建代理，并仍使用默认实现。创建代理允许您记录事件，发出通知，提供前后钩实现等。这是一个`SessionDelegate`在重定向发生时子类化和记录消息的快速示例。

```swift
class LoggingSessionDelegate: SessionDelegate {
    override func urlSession(
        _ session: URLSession,
        task: URLSessionTask,
        willPerformHTTPRedirection response: HTTPURLResponse,
        newRequest request: URLRequest,
        completionHandler: @escaping (URLRequest?) -> Void)
    {
        print("URLSession will perform HTTP redirection to request: \(request)")

        super.urlSession(
            session,
            task: task,
            willPerformHTTPRedirection: response,
            newRequest: request,
            completionHandler: completionHandler
        )
    }
}
```

一般来说，默认实现或覆盖闭包应提供所需的必要功能。子类只应作为最后的手段使用。

> 请记住，`subdelegates`在默认实现中初始化和销毁它是很重要的。子类化时要小心，不要引入内存泄漏。

### 请求

的结果`request`，`download`，`upload`或`stream`方法是a `DataRequest`，`DownloadRequest`，`UploadRequest`和`StreamRequest`其中所有的继承`Request`。所有`Request`实例始终由拥有的会话管理器创建，并且从不直接初始化。

每个子类有专门的方法，比如`authenticate`，`validate`，`responseJSON`和`uploadProgress`每个以利于方法链接返回调用者的实例。

请求可以暂停，恢复和取消：

- `suspend()`：暂停基础任务和调度队列。
- `resume()`：恢复基础任务和调度队列。如果拥有管理器未`startRequestsImmediately`设置为`true`，则请求必须调用`resume()`才能启动。
- `cancel()`：取消基础任务，产生传递给任何已注册响应处理程序的错误。

### 路由请求

随着应用程序规模的扩大，在构建网络堆栈时采用通用模式非常重要。该设计的一个重要部分是如何路由您的请求。Alamofire `URLConvertible`和`URLRequestConvertible`协议以及`Router`设计模式可以提供帮助。

#### URLConvertible

采用该`URLConvertible`协议的类型可用于构造URL，然后用于在内部构造URL请求。`String`，`URL`以及`URLComponents`符合`URLConvertible`在默认情况下，允许任何人将其交给`url`参数的`request`，`upload`和`download`方法：

```swift
let urlString = "https://httpbin.org/post"
Alamofire.request(urlString, method: .post)

let url = URL(string: urlString)!
Alamofire.request(url, method: .post)

let urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: true)!
Alamofire.request(urlComponents, method: .post)
```

鼓励以显着方式与Web应用程序交互的应用程序使自定义类型符合将`URLConvertible`特定于域的模型映射到服务器资源的便捷方式。

##### 类型安全路由

```swift
extension User: URLConvertible {
    static let baseURLString = "https://example.com"

    func asURL() throws -> URL {
    	let urlString = User.baseURLString + "/users/\(username)/"
        return try urlString.asURL()
    }
}

let user = User(username: "mattt")
Alamofire.request(user) // https://example.com/users/mattt
```

#### URLRequestConvertible

采用该`URLRequestConvertible`协议的类型可用于构造URL请求。默认情况下`URLRequest`符合`URLRequestConvertible`，允许它直接传递给`request`，`upload`和`download`方法（这是为各个请求指定自定义HTTP正文的推荐方法）：

```swift
let url = URL(string: "https://httpbin.org/post")!
var urlRequest = URLRequest(url: url)
urlRequest.httpMethod = "POST"

let parameters = ["foo": "bar"]

do {
    urlRequest.httpBody = try JSONSerialization.data(withJSONObject: parameters, options: [])
} catch {
    // No-op
}

urlRequest.setValue("application/json", forHTTPHeaderField: "Content-Type")

Alamofire.request(urlRequest)
```

鼓励以显着方式与Web应用程序交互的应用程序使自定义类型符合`URLRequestConvertible`确保所请求端点的一致性的方式。这种方法可用于抽象出服务器端的不一致性，并提供类型安全的路由，以及管理身份验证凭据和其他状态。

##### API参数抽象

```swift
enum Router: URLRequestConvertible {
    case search(query: String, page: Int)

    static let baseURLString = "https://example.com"
    static let perPage = 50

    // MARK: URLRequestConvertible

    func asURLRequest() throws -> URLRequest {
        let result: (path: String, parameters: Parameters) = {
            switch self {
            case let .search(query, page) where page > 0:
                return ("/search", ["q": query, "offset": Router.perPage * page])
            case let .search(query, _):
                return ("/search", ["q": query])
            }
        }()

        let url = try Router.baseURLString.asURL()
        let urlRequest = URLRequest(url: url.appendingPathComponent(result.path))

        return try URLEncoding.default.encode(urlRequest, with: result.parameters)
    }
}

Alamofire.request(Router.search(query: "foo bar", page: 1)) // https://example.com/search?q=foo%20bar&offset=50
```

##### CRUD和授权

```swift
import Alamofire

enum Router: URLRequestConvertible {
    case createUser(parameters: Parameters)
    case readUser(username: String)
    case updateUser(username: String, parameters: Parameters)
    case destroyUser(username: String)

    static let baseURLString = "https://example.com"

    var method: HTTPMethod {
        switch self {
        case .createUser:
            return .post
        case .readUser:
            return .get
        case .updateUser:
            return .put
        case .destroyUser:
            return .delete
        }
    }

    var path: String {
        switch self {
        case .createUser:
            return "/users"
        case .readUser(let username):
            return "/users/\(username)"
        case .updateUser(let username, _):
            return "/users/\(username)"
        case .destroyUser(let username):
            return "/users/\(username)"
        }
    }

    // MARK: URLRequestConvertible

    func asURLRequest() throws -> URLRequest {
    	let url = try Router.baseURLString.asURL()

        var urlRequest = URLRequest(url: url.appendingPathComponent(path))
        urlRequest.httpMethod = method.rawValue

        switch self {
        case .createUser(let parameters):
            urlRequest = try URLEncoding.default.encode(urlRequest, with: parameters)
        case .updateUser(_, let parameters):
            urlRequest = try URLEncoding.default.encode(urlRequest, with: parameters)
        default:
            break
        }

        return urlRequest
    }
}

Alamofire.request(Router.readUser("mattt")) // GET https://example.com/users/mattt
```

### 适应和重试请求

如今，大多数Web服务都支持某种身份验证系统。今天比较常见的一个是OAuth。这通常涉及生成访问令牌，授权您的应用程序或用户调用各种支持的Web服务。虽然创建这些初始访问令牌可能很麻烦，但是当访问令牌过期并且您需要获取新令牌时，它可能会更加复杂。需要考虑许多线程安全问题。

在`RequestAdapter`和`RequestRetrier`协议的建立是为了使它更容易为一组特定的Web服务创建一个线程安全的身份验证系统。

#### RequestAdapter

该`RequestAdapter`协议允许在创建之前`Request`对每个上面的内容`SessionManager`进行检查和调整。使用适配器的一种非常具体的方法是向`Authorization`特定类型的身份验证后面的请求附加标头。

```swift
class AccessTokenAdapter: RequestAdapter {
    private let accessToken: String

    init(accessToken: String) {
        self.accessToken = accessToken
    }

    func adapt(_ urlRequest: URLRequest) throws -> URLRequest {
        var urlRequest = urlRequest

        if let urlString = urlRequest.url?.absoluteString, urlString.hasPrefix("https://httpbin.org") {
            urlRequest.setValue("Bearer " + accessToken, forHTTPHeaderField: "Authorization")
        }

        return urlRequest
    }
}

let sessionManager = SessionManager()
sessionManager.adapter = AccessTokenAdapter(accessToken: "1234")

sessionManager.request("https://httpbin.org/get")
```

#### RequestRetrier

该`RequestRetrier`协议允许重试执行`Request`的`Error`一段时间。同时使用`RequestAdapter`和`RequestRetrier`协议时，您可以为OAuth1，OAuth2，Basic Auth甚至指数退避重试策略创建凭据刷新系统。可能性是无止境。以下是如何为OAuth2访问令牌实现刷新流程的示例。

> **免责声明：**这**不是**一个全球性的`OAuth2`解决方案。它只是一个示例，演示了如何`RequestAdapter`结合使用它`RequestRetrier`来创建一个线程安全的刷新系统。

> 重申一下，**请勿复制**此示例代码并将其放入生产应用程序中。这只是一个例子。每个身份验证系统必须针对特定平台和身份验证类型进行定制。

```swift
class OAuth2Handler: RequestAdapter, RequestRetrier {
    private typealias RefreshCompletion = (_ succeeded: Bool, _ accessToken: String?, _ refreshToken: String?) -> Void

    private let sessionManager: SessionManager = {
        let configuration = URLSessionConfiguration.default
        configuration.httpAdditionalHeaders = SessionManager.defaultHTTPHeaders

        return SessionManager(configuration: configuration)
    }()

    private let lock = NSLock()

    private var clientID: String
    private var baseURLString: String
    private var accessToken: String
    private var refreshToken: String

    private var isRefreshing = false
    private var requestsToRetry: [RequestRetryCompletion] = []

    // MARK: - Initialization

    public init(clientID: String, baseURLString: String, accessToken: String, refreshToken: String) {
        self.clientID = clientID
        self.baseURLString = baseURLString
        self.accessToken = accessToken
        self.refreshToken = refreshToken
    }

    // MARK: - RequestAdapter

    func adapt(_ urlRequest: URLRequest) throws -> URLRequest {
        if let urlString = urlRequest.url?.absoluteString, urlString.hasPrefix(baseURLString) {
            var urlRequest = urlRequest
            urlRequest.setValue("Bearer " + accessToken, forHTTPHeaderField: "Authorization")
            return urlRequest
        }

        return urlRequest
    }

    // MARK: - RequestRetrier

    func should(_ manager: SessionManager, retry request: Request, with error: Error, completion: @escaping RequestRetryCompletion) {
        lock.lock() ; defer { lock.unlock() }

        if let response = request.task?.response as? HTTPURLResponse, response.statusCode == 401 {
            requestsToRetry.append(completion)

            if !isRefreshing {
                refreshTokens { [weak self] succeeded, accessToken, refreshToken in
                    guard let strongSelf = self else { return }

                    strongSelf.lock.lock() ; defer { strongSelf.lock.unlock() }

                    if let accessToken = accessToken, let refreshToken = refreshToken {
                        strongSelf.accessToken = accessToken
                        strongSelf.refreshToken = refreshToken
                    }

                    strongSelf.requestsToRetry.forEach { $0(succeeded, 0.0) }
                    strongSelf.requestsToRetry.removeAll()
                }
            }
        } else {
            completion(false, 0.0)
        }
    }

    // MARK: - Private - Refresh Tokens

    private func refreshTokens(completion: @escaping RefreshCompletion) {
        guard !isRefreshing else { return }

        isRefreshing = true

        let urlString = "\(baseURLString)/oauth2/token"

        let parameters: [String: Any] = [
            "access_token": accessToken,
            "refresh_token": refreshToken,
            "client_id": clientID,
            "grant_type": "refresh_token"
        ]

        sessionManager.request(urlString, method: .post, parameters: parameters, encoding: JSONEncoding.default)
            .responseJSON { [weak self] response in
                guard let strongSelf = self else { return }

                if 
                    let json = response.result.value as? [String: Any], 
                    let accessToken = json["access_token"] as? String, 
                    let refreshToken = json["refresh_token"] as? String 
                {
                    completion(true, accessToken, refreshToken)
                } else {
                    completion(false, nil, nil)
                }

                strongSelf.isRefreshing = false
            }
    }
}

let baseURLString = "https://some.domain-behind-oauth2.com"

let oauthHandler = OAuth2Handler(
    clientID: "12345678",
    baseURLString: baseURLString,
    accessToken: "abcd1234",
    refreshToken: "ef56789a"
)

let sessionManager = SessionManager()
sessionManager.adapter = oauthHandler
sessionManager.retrier = oauthHandler

let urlString = "\(baseURLString)/some/endpoint"

sessionManager.request(urlString).validate().responseJSON { response in
    debugPrint(response)
}
```

一旦`OAuth2Handler`被同时作为应用`adapter`和`retrier`为`SessionManager`，它会自动刷新访问令牌，并在重试失败，他们以相同的顺序所有失败的请求的处理无效访问令牌错误。

> 如果您需要按照创建它们的顺序执行它们，则可以按任务标识符对它们进行排序。

上面的示例仅检查`401`响应代码，该代码不够健壮，但确实演示了如何检查无效的访问令牌错误。在生产应用程序中，尽管取决于OAuth2实现，但仍希望检查`realm`并且最有可能是`www-authenticate`标头响应。

另一个重要的注意事项是该认证系统可以在多个会话管理器之间共享。例如，您可能需要对同一组Web服务使用a `default`和`ephemeral`会话配置。上面的示例允许`oauthHandler`在多个会话管理器之间共享同一实例，以管理单个刷新流。

### 自定义响应序列化

Alamofire为数据，字符串，JSON和属性列表提供内置的响应序列化：

```swift
Alamofire.request(...).responseData { (resp: DataResponse<Data>) in ... }
Alamofire.request(...).responseString { (resp: DataResponse<String>) in ... }
Alamofire.request(...).responseJSON { (resp: DataResponse<Any>) in ... }
Alamofire.request(...).responsePropertyList { (resp: DataResponse<Any>) in ... }
```

这些响应包括反序列化的*值*（数据，字符串，任何）或*错误*（网络，验证错误），以及*元数据*（URL请求，HTTP标头，状态代码，[度量标准](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#statistical-metrics) ......）。

您可以通过多种方式自定义所有响应元素：

- [响应映射](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#response-mapping)
- [处理错误](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#handling-errors)
- [创建自定义响应序列化程序](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#creating-a-custom-response-serializer)
- [通用响应对象序列化](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#generic-response-object-serialization)

#### 响应映射

响应映射是生成自定义响应的最简单方法。它可以转换响应的值，同时保留最终的错误和元数据。例如，您可以将json响应`DataResponse<Any>`转换为包含应用程序模型的响应，例如`DataResponse<User>`。您使用以下`DataResponse.map`方法执行响应映射：

```swift
Alamofire.request("https://example.com/users/mattt").responseJSON { (response: DataResponse<Any>) in
    let userResponse = response.map { json in
        // We assume an existing User(json: Any) initializer
        return User(json: json)
    }

    // Process userResponse, of type DataResponse<User>:
    if let user = userResponse.value {
        print("User: { username: \(user.username), name: \(user.name) }")
    }
}
```

当转换可能引发错误时，请`flatMap`改为使用：

```swift
Alamofire.request("https://example.com/users/mattt").responseJSON { response in
    let userResponse = response.flatMap { json in
        try User(json: json)
    }
}
```

响应映射非常适合您的自定义完成处理程序：

```swift
@discardableResult
func loadUser(completionHandler: @escaping (DataResponse<User>) -> Void) -> Alamofire.DataRequest {
    return Alamofire.request("https://example.com/users/mattt").responseJSON { response in
        let userResponse = response.flatMap { json in
            try User(json: json)
        }

        completionHandler(userResponse)
    }
}

loadUser { response in
    if let user = response.value {
        print("User: { username: \(user.username), name: \(user.name) }")
    }
}
```

当map / flatMap闭包可以处理大量数据时，请确保在主线程之外执行它：

```swift
@discardableResult
func loadUser(completionHandler: @escaping (DataResponse<User>) -> Void) -> Alamofire.DataRequest {
    let utilityQueue = DispatchQueue.global(qos: .utility)

    return Alamofire.request("https://example.com/users/mattt").responseJSON(queue: utilityQueue) { response in
        let userResponse = response.flatMap { json in
            try User(json: json)
        }

        DispatchQueue.main.async {
            completionHandler(userResponse)
        }
    }
}
```

`map`并且`flatMap`还可以[下载回复](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#downloading-data-to-a-file)。

#### 处理错误

在实现自定义响应序列化程序或对象序列化方法之前，考虑如何处理可能发生的任何错误非常重要。有两个基本选项：沿未修改的方式传递现有错误，在响应时处理; 或者，将所有错误包装在`Error`特定于您的应用的类型中。

例如，这是一个简单的`BackendError`枚举，将在后面的例子中使用：

```swift
enum BackendError: Error {
    case network(error: Error) // Capture any underlying Error from the URLSession API
    case dataSerialization(error: Error)
    case jsonSerialization(error: Error)
    case xmlSerialization(error: Error)
    case objectSerialization(reason: String)
}
```

#### 创建自定义响应序列化程序

Alamofire为字符串，JSON和属性列表提供内置的响应序列化，但是其他可以在扩展中添加`Alamofire.DataRequest`和/或`Alamofire.DownloadRequest`。

例如，以下是如何实现使用[Ono](https://github.com/mattt/Ono)的响应处理程序：

```swift
extension DataRequest {
    static func xmlResponseSerializer() -> DataResponseSerializer<ONOXMLDocument> {
        return DataResponseSerializer { request, response, data, error in
            // Pass through any underlying URLSession error to the .network case.
            guard error == nil else { return .failure(BackendError.network(error: error!)) }

            // Use Alamofire's existing data serializer to extract the data, passing the error as nil, as it has
            // already been handled.
            let result = Request.serializeResponseData(response: response, data: data, error: nil)

            guard case let .success(validData) = result else {
                return .failure(BackendError.dataSerialization(error: result.error! as! AFError))
            }

            do {
                let xml = try ONOXMLDocument(data: validData)
                return .success(xml)
            } catch {
                return .failure(BackendError.xmlSerialization(error: error))
            }
        }
    }

    @discardableResult
    func responseXMLDocument(
        queue: DispatchQueue? = nil,
        completionHandler: @escaping (DataResponse<ONOXMLDocument>) -> Void)
        -> Self
    {
        return response(
            queue: queue,
            responseSerializer: DataRequest.xmlResponseSerializer(),
            completionHandler: completionHandler
        )
    }
}
```

#### 通用响应对象序列化

泛型可用于提供自动，类型安全的响应对象序列化。

```swift
protocol ResponseObjectSerializable {
    init?(response: HTTPURLResponse, representation: Any)
}

extension DataRequest {
    func responseObject<T: ResponseObjectSerializable>(
        queue: DispatchQueue? = nil,
        completionHandler: @escaping (DataResponse<T>) -> Void)
        -> Self
    {
        let responseSerializer = DataResponseSerializer<T> { request, response, data, error in
            guard error == nil else { return .failure(BackendError.network(error: error!)) }

            let jsonResponseSerializer = DataRequest.jsonResponseSerializer(options: .allowFragments)
            let result = jsonResponseSerializer.serializeResponse(request, response, data, nil)

            guard case let .success(jsonObject) = result else {
                return .failure(BackendError.jsonSerialization(error: result.error!))
            }

            guard let response = response, let responseObject = T(response: response, representation: jsonObject) else {
                return .failure(BackendError.objectSerialization(reason: "JSON could not be serialized: \(jsonObject)"))
            }

            return .success(responseObject)
        }

        return response(queue: queue, responseSerializer: responseSerializer, completionHandler: completionHandler)
    }
}

struct User: ResponseObjectSerializable, CustomStringConvertible {
    let username: String
    let name: String

    var description: String {
        return "User: { username: \(username), name: \(name) }"
    }

    init?(response: HTTPURLResponse, representation: Any) {
        guard
            let username = response.url?.lastPathComponent,
            let representation = representation as? [String: Any],
            let name = representation["name"] as? String
        else { return nil }

        self.username = username
        self.name = name
    }
}

Alamofire.request("https://example.com/users/mattt").responseObject { (response: DataResponse<User>) in
    debugPrint(response)

    if let user = response.result.value {
        print("User: { username: \(user.username), name: \(user.name) }")
    }
}
```

同样的方法也可用于处理返回对象集合表示的端点：

```swift
protocol ResponseCollectionSerializable {
    static func collection(from response: HTTPURLResponse, withRepresentation representation: Any) -> [Self]
}

extension ResponseCollectionSerializable where Self: ResponseObjectSerializable {
    static func collection(from response: HTTPURLResponse, withRepresentation representation: Any) -> [Self] {
        var collection: [Self] = []

        if let representation = representation as? [[String: Any]] {
            for itemRepresentation in representation {
                if let item = Self(response: response, representation: itemRepresentation) {
                    collection.append(item)
                }
            }
        }

        return collection
    }
}

extension DataRequest {
    @discardableResult
    func responseCollection<T: ResponseCollectionSerializable>(
        queue: DispatchQueue? = nil,
        completionHandler: @escaping (DataResponse<[T]>) -> Void) -> Self
    {
        let responseSerializer = DataResponseSerializer<[T]> { request, response, data, error in
            guard error == nil else { return .failure(BackendError.network(error: error!)) }

            let jsonSerializer = DataRequest.jsonResponseSerializer(options: .allowFragments)
            let result = jsonSerializer.serializeResponse(request, response, data, nil)

            guard case let .success(jsonObject) = result else {
                return .failure(BackendError.jsonSerialization(error: result.error!))
            }

            guard let response = response else {
                let reason = "Response collection could not be serialized due to nil response."
                return .failure(BackendError.objectSerialization(reason: reason))
            }

            return .success(T.collection(from: response, withRepresentation: jsonObject))
        }

        return response(responseSerializer: responseSerializer, completionHandler: completionHandler)
    }
}

struct User: ResponseObjectSerializable, ResponseCollectionSerializable, CustomStringConvertible {
    let username: String
    let name: String

    var description: String {
        return "User: { username: \(username), name: \(name) }"
    }

    init?(response: HTTPURLResponse, representation: Any) {
        guard
            let username = response.url?.lastPathComponent,
            let representation = representation as? [String: Any],
            let name = representation["name"] as? String
        else { return nil }

        self.username = username
        self.name = name
    }
}

Alamofire.request("https://example.com/users").responseCollection { (response: DataResponse<[User]>) in
    debugPrint(response)

    if let users = response.result.value {
        users.forEach { print("- \($0)") }
    }
}

```

### 安全

在与服务器和Web服务通信时使用安全HTTPS连接是保护敏感数据的重要步骤。默认情况下，Alamofire将使用安全框架提供的Apple内置验证来评估服务器提供的证书链。虽然这可以保证证书链的有效性，但它不会阻止中间人（MITM）攻击或其他潜在漏洞。为了减轻MITM攻击，处理敏感客户数据或财务信息的应用程序应使用由提供的证书或公钥固定`ServerTrustPolicy`。

#### ServerTrustPolicy

该`ServerTrustPolicy`枚举评估一般由提供的服务器信任`URLAuthenticationChallenge`连接到服务器通过安全的HTTPS连接时。

```swift
let serverTrustPolicy = ServerTrustPolicy.pinCertificates(
    certificates: ServerTrustPolicy.certificates(),
    validateCertificateChain: true,
    validateHost: true
)
```

服务器信任评估有许多不同的情况，使您可以完全控制验证过程：

- `performDefaultEvaluation`：使用默认服务器信任评估，同时允许您控制是否验证质询提供的主机。
- `pinCertificates`：使用固定证书来验证服务器信任。如果其中一个固定证书与其中一个服务器证书匹配，则认为服务器信任有效。
- `pinPublicKeys`：使用固定的公钥来验证服务器信任。如果其中一个固定公钥与其中一个服务器证书公钥匹配，则认为服务器信任有效。
- `disableEvaluation`：禁用所有评估，而评估又始终将任何服务器信任视为有效。
- `customEvaluation`：使用关联的闭包来评估服务器信任的有效性，从而使您可以完全控制验证过程。谨慎使用。

#### 服务器信任策略管理器

该`ServerTrustPolicyManager`负责存储服务器的信任策略的内部映射到特定的主机。这允许Alamofire根据不同的服务器信任策略评估每个主机。

```swift
let serverTrustPolicies: [String: ServerTrustPolicy] = [
    "test.example.com": .pinCertificates(
        certificates: ServerTrustPolicy.certificates(),
        validateCertificateChain: true,
        validateHost: true
    ),
    "insecure.expired-apis.com": .disableEvaluation
]

let sessionManager = SessionManager(
    serverTrustPolicyManager: ServerTrustPolicyManager(policies: serverTrustPolicies)
)
```

> 确保保留对新`SessionManager`实例的引用，否则在取消`sessionManager`分配时，您的请求都将被取消。

这些服务器信任策略将导致以下行为：

- ```swift
  test.example.com
  ```

   将始终使用证书链接和证书链并启用主机验证，因此需要满足以下条件才能使TLS握手成功：

  - 证书链必须有效。
  - 证书链必须包含一个固定证书。
  - 挑战主机必须匹配证书链的叶证书中的主机。

- `insecure.expired-apis.com` 永远不会评估证书链，并始终允许TLS握手成功。

- 所有其他主机将使用Apple提供的默认评估。

##### 子类化服务器信任策略管理器

如果您发现自己需要更灵活的服务器信任策略匹配行为（即通配域），则使用您自己的自定义实现继承`ServerTrustPolicyManager`并覆盖该`serverTrustPolicyForHost`方法。

```swift
class CustomServerTrustPolicyManager: ServerTrustPolicyManager {
    override func serverTrustPolicy(forHost host: String) -> ServerTrustPolicy? {
        var policy: ServerTrustPolicy?

        // Implement your custom domain matching behavior...

        return policy
    }
}
```

#### 验证主机

的`.performDefaultEvaluation`，`.pinCertificates`和`.pinPublicKeys`服务器信任的政策都将一个`validateHost`参数。将值设置为`true`将导致服务器信任评估验证证书中的主机名是否与质询的主机名匹配。如果它们不匹配，评估将失败。一个`validateHost`价值`false`仍然会评估整个证书链，但不会验证叶证书的主机名。

> 建议`validateHost`始终`true`在生产环境中进行设置。

#### 验证证书链

固定证书和公钥都可以选择使用`validateCertificateChain`参数验证证书链。通过将此值设置为`true`，除了对固定证书或公钥执行字节相等性检查之外，还将评估完整证书链。值`false`将跳过证书链验证，但仍将执行字节相等性检查。

在某些情况下，禁用证书链验证可能是有意义的。禁用验证的最常见用例是自签名和过期证书。在这两种情况下，评估总是会失败，但字节相等检查仍将确保您收到您希望从服务器获得的证书。

> 建议`validateCertificateChain`始终`true`在生产环境中进行设置。

#### App Transport Security

通过在iOS 9中添加App Transport Security（ATS），使用`ServerTrustPolicyManager`具有多个`ServerTrustPolicy`对象的自定义可能无效。如果你不断看到`CFNetwork SSLHandshake failed (-9806)`错误，你可能遇到了这个问题。除非您在应用程序的plist中配置ATS设置以禁用足够的ATS设置以允许您的应用评估服务器信任，否则Apple的ATS系统会覆盖整个质询系统。

如果您遇到此问题（使用自签名证书的概率很高），您可以通过将以下内容添加到您的问题来解决此问题`Info.plist`。

```swift
<dict>
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSExceptionDomains</key>
        <dict>
            <key>example.com</key>
            <dict>
                <key>NSExceptionAllowsInsecureHTTPLoads</key>
                <true/>
                <key>NSExceptionRequiresForwardSecrecy</key>
                <false/>
                <key>NSIncludesSubdomains</key>
                <true/>
                <!-- Optional: Specify minimum TLS version -->
                <key>NSTemporaryExceptionMinimumTLSVersion</key>
                <string>TLSv1.2</string>
            </dict>
        </dict>
    </dict>
</dict>
```

是否需要设置`NSExceptionRequiresForwardSecrecy`为`NO`取决于您的TLS连接是否使用允许的密码套件。在某些情况下，需要将其设置为`NO`。在`NSExceptionAllowsInsecureHTTPLoads`必须设置为`YES`以允许`SessionDelegate`接受挑战回调。一旦调用挑战回调，`ServerTrustPolicyManager`将接管服务器信任评估。您可能还需要指定`NSTemporaryExceptionMinimumTLSVersion`是否尝试连接到仅支持低于TLS版本的主机`1.2`。

> 建议始终在生产环境中使用有效证书。

#### 将自签名证书与本地网络配合使用

如果您尝试连接到localhost上运行的服务器，并且您使用的是自签名证书，则需要将以下内容添加到您的`Info.plist`。

```swift
<dict>
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsLocalNetworking</key>
        <true/>
    </dict>
</dict>
```

根据[苹果的文档](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW35)，设置`NSAllowsLocalNetworking`以`YES`允许本地资源负载，而无需禁用ATS为您的应用程序的其余部分。

### 网络可达性

该`NetworkReachabilityManager`主机和地址都WWAN和WiFi网络接口的可达性变化监听。

```swift
let manager = NetworkReachabilityManager(host: "www.apple.com")

manager?.listener = { status in
    print("Network Status Changed: \(status)")
}

manager?.startListening()
```

> 请务必记住保留`manager`上面的示例，否则不会报告任何状态更改。此外，不要在`host`字符串中包含该方案，否则可达性将无法正常运行。

使用网络可达性来确定下一步该做什么时，需要记住一些重要的事项。

- 请勿

  使用Reachability确定是否应发送网络请求。

  - 你应该**总是**发送它。

- 恢复可达性后，使用该事件重试失败的网络请求。

  - 即使网络请求可能仍然失败，这是重试它们的好时机。

- 网络可达性状态可用于确定网络请求可能失败的原因。

  - 如果网络请求失败，告诉用户网络请求由于脱机而不是更技术性错误（例如“请求超时”）更有用。

> 建议查看[WWDC 2012 Session 706，“Networking Best Practices”](https://developer.apple.com/videos/play/wwdc2012-706/)以获取更多信息。