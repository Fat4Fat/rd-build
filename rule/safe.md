# 安全规范

安全编码定义了一套可以集成到开发生命周期中的通用软件安全编码规范。采用这些规范将减少最为常见的安全漏洞，也可以作为在代码review时安全checklist名单.



## 一. 输入验证

认为所有客户端传输的数据皆不可信，验证所有来自客户端的数据，包括:

1. 所有请求参数
2. URL
3. HTTP 头信息(比如:cookie 名字和数据值)
4. 环境信息
5. 上传文件
6. 验证正确的数据类型，数据范围，数据长度，数据格式

**具体注意点：**

1.尽可能采用白名单和正则表达形式，验证所有的输入

2.丢弃任何没有通过输入验证的数据，不允许在程序里直接使用$_SERVER, $_GET, $_POST, $_COOKIE, $_FILES, $_ENV, $REQUEST获取外部数据

3.不建议只使用转义，使用转义必须有其他控制手段范围控制等，转义函数可以参考htmlentities，htmlspecialchars

4.如果任何潜在的危险字符必须被作为输入，请确保您执行了额外的控制，比如:输入转义、输出编码、特定的安全 API、以及在应用程序中使用的原因。部分常见的危险字符包括:< > " ' % ( ) & + \ \' \" 

**文件相关：**

文件名中不能包含二进制数据，否则可能引起问题。同时需要限定文件名的长度。虽然Unix系统几乎可以在文件名设定中使用任何符号，但是应当尽量使用 - 和 _ 避免使用其他字符。一些系统允许Unicode多字节编码的文件名，但是尽量避免，应当使用ASCII的字符。如果用户不需要知道存储的文件名。建议使用md5或者其他方式重命名文件名指定文件后缀。

实例：

## **二. 访问控制**

1. 除了那些特定设为“公开”的内容以外,对所有的网页和资源要求身份验证。
2. 对资源获取的时候都需要判断是不是该用户能操作和获取的。避免越权和全面遍历。
3. 对操作和浏览尽量小的权限，提升权限的时候需要进一步验证。
4. 对于有流程限制的操作，注意限制跨步提交。

**当设计外网访问API接口时应注意：**

1. 防止请求内容篡改和伪造
2. 防止请求内容被重放攻击
3. 确保请求是由应用方所发起
4. 尽可能使用HTTPS传输

**当设计JS或者客户端请求接口时应注意：**

1. 校验请求refer，不仅有域限制，而且要严格限制请求发起地址
2. 关键数据变更可以使用验证token，可以考虑验证码
3. 在关键位置要做防止机器破解控制，使用频率控制器，限制尝试次数，使用验证码区别人和机，比如：注册登录
4. 在做HEADER跳转时，url的地址不能是外界传入，尽量控制是本域跳转。如果必须用户传入跳转地址，添加一个用户不可控的随机值参数，如果验证成功则直接跳转，验证失败进行跳转提示(参考google)

**管理系统等内部系统，务必处于内网或者VPN保护下。禁止公网一切访问。**

**在后台系统用户授权上，建议采用双因子认证，比如google二步认证。**

## **三. 数据保护**

1. 对隐私和主要机密信息，在存储设备上应该做加密处理，用来验证的信息使用散列函数求值存储（比如密码），用来获取的信息使用对称加密算法AES处理（比如手机号）
2. 为防范对随机数据的猜测攻击,应当使用加密模块中已验证的随机数生成器生成所有的随机数、随机文件名、随机 GUID 和随机字符串
3. 不要在客户端上以明文形式或其他非加密安全模式保存密码、连接字符串或其他敏感信息。 这包括嵌入在不安全的形式中:MS viewstate、Adobe flash 或者已编译的代码。
4. 对数据采用一定的措施降低泄露后威胁程度，比如（给身份证图片添加无法擦除水印，避免用于其他用途）
5. 包含敏感信息的数据传输的采用TLS
6. 当链接到外部站点时,过滤来自 HTTP referer 中包含敏感信息的参数（比如sessionid）
7. 限制只有授权的用户才能访问文件或其他资源，比如（用户自身上传的非公开的图片）

## **四. 服务禁用或受限**

1. 线上WEB服务器禁止直接调用本地命令，如果需要执行单独申请。
2. 错误日志及调试信息，禁止线上打印输出一切无关信息，关闭错误直接输出。禁止线上直接写本地文件。
3. 线上数据库只提供select,update,delete,insert功能，默认禁止建表，删表，清空表操作。
4. 禁止直接本机出公网，必须通过统一出口网关。需要找运维添加白名单。