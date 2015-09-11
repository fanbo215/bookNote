# network & server
## http protocol
### 概念
* 应用层的面向对象的协议，目前版本号1.1
* 基于请求和响应模式的，无状态的应用层协议，常基于tcp的连接方式

### URL
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
http请求与响应
http协议的交互主要有请求与响应组成，请求是指客户端发起向web服务器请求资源的消息，而响应是web服务器根据客户端的请求会送给客户端的资源信息。
发送请求信息（request message）包括以下几个部分：
请求行，如 GET /images/logo.gif HTTP/1.1，表示从/images目录下请求logo.gif这个文件。
请求头，如Accept-Language:en
空行
可选消息体
在HTTP/1.1协议中，所有的请求头，除Host外，都是可选的
请求方法（request method）
在HTTP/1.1协议中共定义了八种方法（动作），来表明request－URI指定的资源的不同操作方式
OPTIONS
返回服务器针对特定资源所支持的HTTP请求方法。
GET
向特定的资源发出请求。该方法不应当被用于产生“副作用”的操作。
POST
向指定资源提交数据进行处理请求（例如提交表单或上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和／或已有资源的修改
PUT
向指定资源位置上传其最新内容
DELETE删除指定资源
TRACE
回显服务器收到的请求
CONNECT
预留给能够将连接改为管道方式的代理服务器


## AFNetworking
1. 如何上传图片
2. 如何上传文件

