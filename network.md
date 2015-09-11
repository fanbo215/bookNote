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

## AFNetworking
1. 如何上传图片
2. 如何上传文件

