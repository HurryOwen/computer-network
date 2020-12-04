# 数据类型与编码
不同于 TCP 和 UDP 只要把数据发送到对方就算完成了任务，HTTP 还要告诉上层应用是什么数据。

HTTP 通过 MIME 标准来标记 body 的数据类型。

**MIME type**：

- text：文本格式的可读数据。比如：超文本 text/html、纯文本 text/plain、样式表 text/css。
- image：图像文件，image/gif，image/jpeg，image/png。
- audio/video：音频和视频数据，如 audio/mpeg，video/mp4
- application：数据格式不确定，可能是文本也可能是二进制，必须由上层应用来解释。常见的有：application/json、application/javascript、application/pdf


但是仅有 MIME type 还不够，因为 HTTP 在传输时为了节约带宽，还会压缩数据。因此还需要 “Ecoding type”，告诉接收数据用的是什么编码格式，以正确解压缩还原出数据。

**Ecoding type**：
- gzip: GNU zip 压缩格式，也是互联网上最流行的压缩格式；
- deflate：zlib(deflate) 压缩格式，流行程度仅次于 gzip；
- br：一种专门为 HTTP 优化的新压缩算法。

# 数据类型使用的头字段
有了MIME type 和 Encoding type，无论是浏览器还是服务器就都可以轻松识别出 body 的类型，也就能够正确处理数据了。

HTTP 协议为此定义了两个 Accept 请求头字段和两个 Content 实体头字段，用于客户端和服务器进行“**内容协商**”。也就是说，客户端用 Accept 头告诉服务器希望接收什么样的数据，而服务器用 Content 头告诉客户端实际发送了什么样的数据。

![内容协商](https://static001.geekbang.org/resource/image/51/b9/5191bce1329efa157a6cc37ab9e789b9.png)

**Accept** 字段标记的是客户端可理解的 MIME type，可以用“,”做分隔符列出多个类型，让服务器有更多的选择余地，例如下面的这个头：
```http
Accept: text/html,application/xml,image/webp,image/png
```
相应的，服务器会在响应报文里用头字段 **Content-Type** 告诉实体数据的真实类型：
```http
Content-Type: text/html
Content-Type: image/png
```
**Accept-Encoding** 字段标记的是客户端支持的压缩格式，例如上面说的 gzip、deflate 等，同样也可以用“,”列出多个。

服务器可以选择其中一种来压缩数据，实际使用的压缩格式放在响应头字段 **Content-Encoding** 里。
```http
Accept-Encoding: gzip, deflate, br
```
```http
Content-Encoding: gzip
```
*这两个字段是可以省略的，如果请求报文里没有 Accept-Encoding 字段，就表示客户端不支持压缩数据；如果响应报文里没有 Content-Encoding 字段，就表示响应数据没有被压缩。*

# 语言类型和编码
## 语言类型
“**语言类型**”就是人类使用的自然语言，比如en 表示任意的英语，en-US 表示美式英语，en-GB 表示英式英语，而 zh-CN 就表示我们最常使用的汉语。
## Unicode
**字符集**，比如英语世界的 ASCII 字符集，汉语世界用的 GBK、BIG5 字符集，日语世界用的 Shift_JIS 字符集等。同样一段文字，用一种编码显示正常，换成另一种编码后就变成乱码。


于是出现了统一的字符集 Unicode，它将所有符号都纳入其中，并给每一个符号都赋予一个独一无二的二进制代码。

**问题**：
- 如何才能区分 Unicode 和 ASCII ？计算机怎么知道三个字节表示一个符号，而不是分别表示三个符号呢？
- 如果 Unicode 统一规定，每个符号用三个或四个字节表示，那么每个英文字母前都必须由二到三个字节是`0`，这对存储来说是极大的浪费。

## UTF-8
UTF-8 是一种**字符编码**，字符编码是对字符集的实现方式。所以 **UTF-8 是 Unicode 的实现方式之一。**其他实现方式还包括 UTF-16、UTF-32。

UTF-8 最大的特点是，它是一种变长的编码方式，可以用 1 ~ 4 个字节表示一个符号，根据不同的符号而变化字节长度。

UTF-8 的编码规则很简单，只有二条：
- 对于单字节的符号，字节的第一位设为 `0`,后面 7 位为这个符号的 Unicode 编码。因此对于英文字母，UTF-8 编码和 ASCII 码是相同的。
- 对于 `n` 字节的符号（`n>1`），第一个字节的前 `n` 位都设为 `1`，第 `n+1` 位设为 `0`，后面字节前两位一律设为 `10`。剩下的没有提及的二进制位，全部为这个字符的 Unicode 码。

| Unicode 符号范围（16进制）|UTF-8编码方式（二进制）|
|--|--|
|0000 0000 - 0000 007F|0xxxxxxx|
|0000 0080 - 0000 07FF|110xxxxx 10xxxxxx|
|0000 0800 - 0000 FFFF|1110xxxx 10xxxxxx 10xxxxxx|
|0001 0000 - 0010 FFFF|11110xxx 10xxxxxx 10xxxxxx 10xxxxxx|

跟据上表，解读 UTF-8 编码非常简单。如果一个字节的第一位是 `0`，则这个字节单独就是一个字符；如果第一位是 `1`，则连续有多少个 `1`，就表示当前字符占用多少个字节。

# 语言类型使用的头字段
同样的，HTTP 协议也使用 Accept 请求头字段和 Content 实体头字段，用于客户端和服务器就语言与编码进行“内容协商”。

**Accept-Language** 字段标记了客户端可理解的自然语言，也允许用“,”做分隔符列出多个类型，例如：
```http
Accept-Language: zh-CN, zh, en
```
相应的，服务器应该在响应报文里用头字段 **Content-Language** 告诉客户端实体数据使用的实际语言类型：
```http
Content-Language: zh-CN
```
字符集在 HTTP 里使用的请求头字段是 **Accept-Charset**，但响应头里却没有对应的 Content-Charset，而是在 Content-Type 字段的数据类型后面用“charset=xxx”来表示，这点需要特别注意。
```http
Accept-Charset: gbk, utf-8
Content-Type: text/html; charset=utf-8
```
不过现在的浏览器都支持多种字符集，通常不会发送 Accept-Charset，而服务器也不会发送 Content-Language，因为使用的语言完全可以由字符集推断出来，所以在请求头里一般只会有 Accept-Language 字段，响应头里只会有 Content-Type 字段。
![语言内容协商](https://static001.geekbang.org/resource/image/0e/10/0e9bcd6922fa8908bdba79d98ae5fa10.png)

## 内容协商的质量值
在 HTTP 协议里用 Accept、Accept-Encoding、Accept-Language 等请求头字段进行内容协商的时候，还可以用一种特殊的“q”参数表示权重来设定优先级，这里的“q”是“quality factor”的意思。

权重的最大值是 1，最小值是 0.01，默认值是 1，如果值是 0 就表示拒绝。具体的形式是在数据类型或语言代码后面加一个“;”，然后是“q=value”
```http
Accept: text/html,application/xml;q=0.9,*/*;q=0.8
```
它表示浏览器最希望使用的是 HTML 文件，权重是 1，其次是 XML 文件，权重是 0.9，最后是任意数据类型，权重是 0.8。

服务器收到请求头后，就会计算权重，再根据自己的实际情况优先输出 HTML 或者 XML。

## 内容协商的结果
内容协商的过程是不透明的，每个 Web 服务器使用的算法都不一样。但有的时候，服务器会在响应头里多加一个 **Vary** 字段，记录服务器在内容协商时参考的请求头字段，给出一点信息，例如：
```http
Vary: Accept-Encoding,User-Agent,Accept
```
这个 Vary 字段表示服务器依据了 Accept-Encoding、User-Agent 和 Accept 这三个头字段，然后决定了发回的响应报文。

# 总结
![](https://static001.geekbang.org/resource/image/b2/58/b2118315a977969ddfcc7ab9d26cb358.png)
