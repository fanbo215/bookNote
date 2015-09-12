# network & server
## http protocol
### 概念
* 应用层的面向对象的协议，目前版本号1.1
* 基于请求和响应模式的，无状态的应用层协议，常基于tcp的连接方式

### URI
* 通过URI（uniform resource identifiers）来访问资源。

> http://myname:mypasss@www.vimer.cn:80/mydir/myfile.html?myvar=myvalue#myfrag

| URI部分        | 意义             |
| :------------- | :--------------- |
| myname         | 用户名（可选）   |
| mypass         | 密码（可选）     |
| www.vimer.cn   | 主机网路地址     |
| 80             | 端口号（可选）   |
| /mydir/myfile.html | 资源字符串（可选） |
| myvar=myvalue  | 查询字符串（可选）|
| myfrag         | 锚点             |

* 协议名用“://"结束，用户名和密码以“：”分隔，以“@”结束，端口号与主机网路地址以“：”分隔，资源路径与查询字符串以“？”分隔，锚点以“＃”开头。
* 只有协议名称，主机网路地址和资源路径是必须包含再URI里的。

### http请求与响应
* http协议的交互主要由请求与响应组成，请求是指客户端发起向web服务器请求资源的消息，而响应是web服务器根据客户端的请求会送给客户端的资源信息。
* 发送请求信息（request message）包括以下几个部分：
  * 请求行，如 GET /images/logo.gif HTTP/1.1，表示从/images目录下请求logo.gif这个文件。
  * 请求头，如Accept-Language:en
  * 空行
  * 可选消息体
* 在HTTP/1.1协议中，所有的请求头，除Host外，都是可选的

#### 请求方法（request method）
* 在HTTP/1.1协议中共定义了八种方法（动作），来表明request－URI指定的资源的不同操作方式
* OPTIONS
  * 返回服务器针对特定资源所支持的HTTP请求方法。
* GET
  * 向特定的资源发出请求。该方法不应当被用于产生“副作用”的操作。
* POST
  * 向指定资源提交数据进行处理请求（例如提交表单或上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和／或已有资源的修改
* PUT
  * 向指定资源位置上传其最新内容
* DELETE
  * 删除指定资源
* TRACE
  * 回显服务器收到的请求
* CONNECT
  * 预留给能够将连接改为管道方式的代理服务器
* 一个GET请求的实例如下：
<p align="center">
  ![http get](/resource/network/getRequest.bmp)
</p>

#### 响应
* 一个HTTP响应实例如下：
<p align="center">
  ![http responst](/resource/network/httpResponse.bmp)
</p>
* 每个响应由HTTP协议头和web内容构成。web服务器收到一个请求，就会立刻解释请求中所用到的方法，并开始处理应答。服务器响应消息也包含头字段形式的协议头。
* 其格式为：

> Status-Line*

> （（general-header)|response-header|entity-header)CRLF)

> CRLF

> [message-body]

* 响应消息的第一行是状态行（Status－Line），由协议版本及数字状态码和相关的文本短语组成，各部分间用空格符隔开

##### HTTP响应状态码

| 状态码 |                            定义 |
| :----- | :------------------------------ |
| 1xx    | 报告 (接收到请求，继续进程)    |
| 2xx    | 成功 (步骤成功接收，被理解，并被接受) |
| 3xx    | 重定向 (为了完成请求,必须采取进一步措施) |
| 4xx    | 客户端出错 (请求包括错的顺序或不能完成) |
| 5xx    | 服务器出错 (服务器无法完成显然有效的请求) |

* 下面列举了为HTTP/1.1定义的状态码值，和对应的原因短语（Reason-Phrase）的例子。

##### 客户端错误

> “100″ : Continue  继续

> “101″ : witching Protocols  交换协议

#####  成功

> “200″ : OK

> “201″ : Created 已创建

> “202″ : Accepted 接收

> “203″ : Non-Authoritative Information 非认证信息

> “204″ : No Content 无内容

> “205″ : Reset Content 重置内容

> “206″ : Partial Content 部分内容

#####  重定向

> “300″ : Multiple Choices 多路选择

> “301″ : Moved Permanently  永久转移

> “302″ : Found 暂时转移

> “303″ : See Other 参见其它

> “304″ : Not Modified 未修改

> “305″ : Use Proxy 使用代理

> “307″ : Temporary Redirect

##### 客户方错误

> “400″ : Bad Request 错误请求

> “401″ : Unauthorized 未认证

> “402″ : Payment Required 需要付费

> “403″ : Forbidden 禁止

> “404″ : Not Found 未找到

> “405″ : Method Not Allowed 方法不允许

> “406″ : Not Acceptable 不接受

> “407″ : Proxy Authentication Required 需要代理认证

> “408″ : Request Time-out 请求超时

> “409″ : Conflict 冲突

> “410″ : Gone 失败

> “411″ : Length Required 需要长度

> “412″ : Precondition Failed 条件失败

> “413″ : Request Entity Too Large 请求实体太大

> “414″ : Request-URI Too Large 请求URI太长

> “415″ : Unsupported Media Type 不支持媒体类型

> “416″ : Requested range not satisfiable

> “417″ : Expectation Failed

##### 服务器错误

> “500″ : Internal Server Error 服务器内部错误

> “501″ : Not Implemented 未实现

> “502″ : Bad Gateway 网关失败

> “503″ : Service Unavailable

> “504″ : Gateway Time-out 网关超时

> “505″ : HTTP Version not supported  HTTP版本不支持

#### HTTP状态码是可扩展的。


## AFNetworking
1. 如何上传图片文件

```
- (void)uploadImage:(id)image
            success:(void (^)(AFHTTPRequestOperation *operation, id responseObject))success
            failure:(void (^)(AFHTTPRequestOperation *operation, NSError *error))failure {
    NSData *imageData = UIImageJPEGRepresentation(image, 0.95);
    
    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
    formatter.dateFormat = @"yyyyMMddHHmmss";
    NSString *str = [formatter stringFromDate:[NSDate date]];
    NSString *fileName = [NSString stringWithFormat:@"%@.jpg", str];
    NSString *URLString = [NSString stringWithFormat:@"%@/images/upload", kISBaseURL];

    NSMutableURLRequest *request = [self.manager.requestSerializer multipartFormRequestWithMethod:@"POST" URLString:URLString parameters:@{@"uid": [ISUserInfoManager shareInstance].currentUser.uid}  constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
        [formData appendPartWithFileData:imageData name:@"userimg" fileName:fileName mimeType:@"image/jpeg"];
    } error:nil];
    AFHTTPRequestOperation *operation = [[AFHTTPRequestOperation alloc] initWithRequest:request];
    [operation setCompletionBlockWithSuccess:^(AFHTTPRequestOperation *operation, id responseObject) {
        if (success) {
            success(operation, operation.responseString);
        }
    } failure:failure];
    [operation start];
}
```
