---
layout: post
title: "URL Session 的生命周期（译）"
description: "Life Cycle of a URL Session"
category: Foundation
tags: [network, translation]
imagefeature: codeplace.jpg
comments: false
mathjax: null
featured: false
published: false
---

NSURLSession出来挺长时间了，试用了下确实比NSURLConnection有优势。
为了进步深入理解其运行原理，更好的发挥其优势，最近仔细的看了下其相关文档。其中关于Session生命周期这部分，感觉对于初学者有莫大的帮助，可以少走许多弯路，故有此一译。

> [Apple文档原文地址](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW1)
> 另：根据个人理解，语言描述和语序略有改动，如有谬误，抻头以待，敬请斧正。

使用NSURLSession有两种方式：**系统自带的Delegate**和**自定义Delegate**。<br/>一般来说，必须使用自定义代理的情况如下：

- app没有运行情况下的，后台下载/上传数据操作；
- 执行自定义的身份验证；
- 执行自定义的SSL证书验证；
- 决定将服务器返回的数据下载到磁盘或者基于的MIME等标准进行显示。
- 以body数据流方式上传数据（而非NSData方式）；
- 通过编程限制缓存；
- 通过编程控制重定向。

如果没有以上需求，完全可以使用系统提供的代理来使用NSURLSession。 <br/>根据你选择的技术，从一下项目选择合适的一项：

1. **系统代理中URLSession 的生命周期。**提供了一个轻量级的方式创建和使用一个URL Session。 即使你打算自己编写代理，也应该阅读这些内容。它给完整的描述了代码中必须做的配置和如何使用配置对象。
2. **自定义代理中URLSession 的生命周期。**提供了一个完整的展现了URL会话过程中的每一步操作 。阅读本章节，能帮助你理解Session与代理的互动。尤其是对每一个代理方法调用的说明。

---

### 系统代理中URLSession 的生命周期

如果在使用NSURLSession时没有提供Delegate，那么系统代理将接手处理许多具体的事务内容。以下是使用系统代理情况下使用NSURLSession时，以及必须经历的方法调用和必须处理的delegate方法：

1. **创建Session Configuration（对象）。**对于后台Session，配置对象必须包含一个唯一的标识，存储这个标识，以备在App从崩溃、终止、中断中恢复时使用。
2. **创建Session（对象），**指定一个Configuration对象，并且设置delegate为nil。
3. **在Session中创建Task对象**，每个task对象代表一个资源请求。
    - 每个task创建之初都处于挂起状态，直到调用task的`resume`方法，它才开始下载指定的资源。
    - 根据不同需求，选用`NSURLSessionTask`的相应子类（`NSURLSessionDataTask`、 `NSURLSessionUploadTask`、`NSURLSessionDownloadTask`）创建task对象。这些task对象类似于NSURLConnection对象，但是提供更多的控制操作和统一的Delegate模型。
    - 尽管你可以在一个Session中添加多个Task（通常情况也应该如此做），但是为了简单阐述URL Session的生命周期，后面以只添加一个task为例来说明。
    - **注意**:在使用NSURLSession时如果使用系统代理（不提供自定义Delegate），在NSURLSession中创建task时必须使用中含有completionHandler:的方法，否则无法处理返回的数据。

4. 在下载task中, 如用户在app正与server进行数据传输时暂停task,那么，调用 `cancelByProducingResumeData:` 方法来取消该task。
然后，当用户需要恢复下载时，将上述方法中参数（block）中返回的resumeData传送给 `downloadTaskWithResumeData:` 或者 `downloadTaskWithResumeData:completionHandler:` 方法来恢复下载。
5. 当一个task完成, NSURLSession对象将调用对应task的 `completion handler(Block)`。
    - **注意**: NSURLSession不会通过error参数报告服务器错误。通过error参数只能接收客户端错误，例如不能解析域名或者不能连接服务器等。error codes的具体描述在[URL Loading System Error Codes](URL Loading System Error Codes)文档中。
6. 当一个Session不在需要时，通过调用 `invalidateAndCancel`(立即取消未完成的task，session失效)或者 `finishTasksAndInvalidate`(等待未完成task完成之后session失效)。

---

### 自定义DelegateURLSession 的生命周期

如果想通NSURLSession进行后台下载、上传，或者以自定义方式处理认证、缓存操作。那么，必须提供自定义Delegate（一个`至少`实现了 `NSURLSessionDelegate` 协议的Session代理，一个或者多个`至少`实现了 `NSURLSessionTaskDelegate``NSURLSessionDataDelegate`的Task代理）。来达到相关目的：
- 当使用Download task时，NSURLSession对象使用Delegate为应用程序提供已下载文件的URL。
所有的后台下载和上传都需要Delegates支持。这些Delegates必须实现所有 `NSURLSessionDownloadDelegate` 协议中的委托方法。

- Delegates可以处理特定的身份验证。
- Delegates提供body数据流方式上传数据到服务器。
- Delegates能够决定HTTP是否重定向.
- NSURLSession对象通过delegate为app提供每个数据传输的状态。Data task的Delegates Data task delegates 将会收到初始化调用（以便能够转成下载请求）和一系列并发调用（以便提供从服务获取的一些数据）。
- Delegates是NSURLSession对象通知App一个数据传输已经完成的一种方式。

如果你在一个(需要执行后台任务的)URL Session使用自定义Delegate，这个URL Session完整的生命周期将更加复杂。以下是自定义Delegate必须实现的Delegate方法和方法调用序列:

1. **创建一个Session Configuration。**对于后台Session，配置对象必须包含一个唯一的标识，存储这个标识，以备在App从崩溃、终止、中断中恢复时使用。
2. **创建一个Session**，指定一个Configuration对象，设置一个Delegate（可选）。
3. **在Session中创建Task对象**，每个task对象代表一个资源请求。
    - 每个task创建之初都处于挂起状态，直到调用task的`resume`方法，它才开始下载指定的资源。
    - 根据不同需求，选用`NSURLSessionTask`的相应子类（`NSURLSessionDataTask`、 `NSURLSessionUploadTask`、`NSURLSessionDownloadTask`）创建task对象。这些task对象类似于NSURLConnection对象，但是提供更多的控制操作和统一的Delegate模型。
    - 尽管你可以在一个Session中添加多个Task（通常情况也应该如此做），但是为了简单阐述URL Session的生命周期，后面以只添加一个task为例来说明。
4. 如果远程服务器返回一个状态码，表明需要身份验证，如果验证需要一个连接级任务(如SSL客户证书)，NSURLSession将调用身份验证任务的Delegate方法。
    - session-level的任务（NSURLAuthenticationMethodNTLM、NSURLAuthenticationMethodNegotiate、NSURLAuthenticationMethodClientCertificate、NSURLAuthenticationMethodServerTrust）：NSURLSession对象将调用**Session Delegate**的 `URLSession:didReceiveChallenge:completionHandler: `方法。如果没有提供Session Delegate那么NSURLSession将调用**Task Delegate**的`URLSession:task:didReceiveChallenge:completionHandler:`方法来处理此任务。
    - non-session-level的任务(除上述之外所有的情况)：NSURLSession对象将调用**Session Delegate**的 `URLSession:task:didReceiveChallenge:completionHandler:` 方法来处理任务。 如果提供了Session Delegate且需要处理此认证，那么必须同时在task级别上处理认证或者提供并且显式的调用一个task级的处理器。且Session Delegate的 `URLSession:didReceiveChallenge:completionHandler:` 方法在non-session-level任务中将不会被调用。
    - **注意**：Kerberos身份验证是透明处理的。
如果在上传任务中身份验证失败，如果一个任务是从数据流中提供的数据，NSURLSession对象调用Delegate的 `URLSession:task:needNewBodyStream:` 方法。Delegate必须提供一个新的NSInputStream对象为新请求提供body数据。
更多信息关于如何实现NSURLSession认证Delegate方法，请阅读[ Authentication Challenges and TLS Chain Validation](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507-SW1)。

5. 在接收到一个HTTP重定向response时，NSURLSession对象将调用Delegate的 `URLSession:task:willPerformHTTPRedirection:newRequest:completionHandler: `  方法。
在该Delegate方法中调用参数中的completionHandler（一个以NSURLRequest为参数的Block）来处理重定向。该block其中的NSURLRequest参数可以是delegate方法中通过参数传入的newRequest(同意此重定向)；也可以另外创建一个新的NSURLRequest对象(用于重定向到一个其他的URL)；也可以是nil(作为一个有效的response来处理response body和返回的结果，即拒绝重定向)。
    - 如果同意重定向之后，回到步骤4(身份验证任务处理)。
    - 如果Delegate未实现这个方法，追踪至重定向的最大数量。
6. 通过调用 `downloadTaskWithResumeData:` 或者`downloadTaskWithResumeData:completionHandler:`来创建的下载（或者重新下载）task，NSURLSession对象将调用Delegate的 `URLSession:downloadTask:didResumeAtOffset:expectedTotalBytes:`方法并将此task传入。
7. 对于Data task，NSURLSession对象将调用Delegate的 `URLSession:dataTask:didReceiveResponse:completionHandler:` 方法。在此方法中决定是否将Data task转化为Download task，进而调用completionHandler来继续接收数据或者下载数据。
如果你选择将Data task转换为Download Task，NSURLSession将调用Delegate的 `URLSession:dataTask:didBecomeDownloadTask:` 方法并将新生成的Download task对象作为参数传入。在此调用之后，Delegate将不再接收来自Data task的回调消息，并开始接收Download task的回调消息。
8. 通过调用 `uploadTaskWithStreamedRequest:` 来创建的Upload task，NSURLSession对象将调用Delegate的 `URLSession:task:needNewBodyStream:` 方法以提供body数据。
9. 在body content上传到服务器过程中，Delegate的 `URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:`方法将被定期调用，用以报告上传进度。
10. 在于服务器数据传输期间，Task Delegate将定期接收到回调消息（传输的进度）。对于一个Download task，Session将在数据成功下载到磁盘后调用Delegate的 `URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:` 方法。对于一个Data task，Session将在成功接收到数据之后调用Delegate的 `URLSession:dataTask:didReceiveData:` 方法。
在Download task中，在从服务器传输过程中如果用户要求app暂停下载，那应该通过调用 `cancelByProducingResumeData:` 方法来取消任务。
然后，当用户需要恢复下载时，将上述方法中参数（block）中返回的resumeData传送给 `downloadTaskWithResumeData:` 或者 `downloadTaskWithResumeData:completionHandler:` 方法创建一个新的下载任务用于继续下载，然后转到步骤3 （创建和恢复任务对象）。
11. 在Data task中，NSURLSession对象调用Delegate的 `URLSession:dataTask:willCacheResponse:completionHandler:` 方法。（在此方法中）应该设置决定是否允许缓存。如果不实现此方法，默认行为是使用Session Configuration对象中指定的缓存策略。
12. 如果一个Download task已完成(成功)，那么NSURLSession 对象将以该task和下载文件的临时目录URL为参数，调用Delegate的 `URLSession:downloadTask:didFinishDownloadingToURL:` 方法与临时文件的位置。在此方法结束之前，应该立即读取文件内容或者将临时文件移动到一个可持久化存储的目录中。
13. 当任何task完成后，NSURLSession对象将调用Delegate的`URLSession:task:didCompleteWithError:` 方法，didCompleteWithError:可能是一个NSError对象（任务出错）或者nil （任务成功）。
如果task失败，大多数app应重新请求，直到用户取消下载或者服务器返回表面该请求永远不会成功的错误标识。然而，app不应该直接重新发出请求，而应先使用可达性Api来确定服务器的可访问性，然后在可访问的前提下，再发送新的请求。
如果这个Download task是可以恢复的，NSError对象的userInfo将包含一个 key为`NSURLSessionDownloadTaskResumeData`的键值对。app应该通过此值来调用 `downloadTaskWithResumeData:` 或 `downloadTaskWithResumeData:completionHandler:` 来创建一个新的下载任务，继续现有的下载。
如果是一个不能恢复的task，那么应创建一个新的Download task并重新从头开始下载。
在任一情况下，如果不是服务器错误导致的传输失败，请转到步骤 3 （创建和恢复task对象）。
**注意**：NSURLSession不会通过error参数报告服务器错误。通过error参数只能接收客户端错误，例如不能解析域名或者不能连接服务器等。error codes的具体描述在[URL Loading System Error Codes](URL Loading System Error Codes)文档中。
服务器端错误是通过NSHTTPURLResponse对象中的HTTP 状态代码来报告的。更多的信息，请阅读NSHTTPURLResponse和NSURLResponse的文档。
14. 如果response是multipart编码的，Session可能会再次调用Delegate的 `didReceiveResponse`方法，紧接着是零个或多个 `didReceiveData` 被调用。如果发生这种情况，请转到步骤7（处理 `didReceiveResponse` 调用）。
15. 当你不再需要一个会话时，通过调用 `invalidateAndCancel` （立即取消未完成的任务） 或 `finishTasksAndInvalidate` （在完成未完成的task完成之后） 使Session无效。
无效的Session，在所有未完成的任务已取消或已结束时后, 会给Delegate发送 `URLSession:didBecomeInvalidWithError:` 消息。当该委托方法执行完毕后，Session释放其对Delegate对象的强引用。
**注意**：Session对象保持其对Delegate的强引用，直到显式地调用方法使Session失效。如果在不需要一个Seesion之后并没有将其invalidate的话，将会出现内存泄漏。

如果app取消正在进行的下载，NSURLSession对象将调用该委托的 `URLSession:task:didCompleteWithError:` 方法，从Delegate角度看是出现了一个错误（其实是你自己取消了task，应予以适当处理）。
