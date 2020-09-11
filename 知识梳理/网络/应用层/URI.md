# URI、URL、URN、Data URI、Object URL

URI（Uniform Resource Identifier）：统一资源定位符，用于标识互联网的资源。比如标识一本电子书、一张图片、一段音视频。

URI 格式如下所示

```text
URI = scheme:[//authority]path[?query][#fragment]

authority = [userinfo@]host[:port]
```

- scheme:：协议，例如 `http、ftp`
- authority：用户认证，例如 `root@localhost:443`
- path：路径
- query：查询参数
- fragment：片段标识符，hash 参数

URI 具有两种形式，目前最普遍的形式 URL，不咋常见的形式 URN。URL、URN、UFC（未广泛采用） 一起构成了 Internet 的三大信息体系结构。

![](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c3/URI_Euler_Diagram_no_lone_URIs.svg/1200px-URI_Euler_Diagram_no_lone_URIs.svg.png)

## URL

URL（Uniform Resource Locator）：统一资源定位符，用于标识互联网资源的地址。URL 比 URI 更加的具体，不仅标识资源，同时定义了如何获取资源（获取资源的路径）。

> A URL is special type of identifier that also tells you how to access it

URL 通过特定的访问协议和路径指定其位置来标识资源，例如 FTP、HTTP 协议

每个 URL 都符合标准的 URI 格式，URL [Internet 通用语法（rfc1738）](https://tools.ietf.org/html/rfc1738#section-3.1)如下所示

```text
//<user>:<password>@<host>:<port>/<url-path>
```

```text
// 协议
ftp                     File Transfer protocol
http                    Hypertext Transfer Protocol
gopher                  The Gopher protocol
mailto                  Electronic mail address
news                    USENET news
nntp                    USENET news using NNTP access
telnet                  Reference to interactive sessions
wais                    Wide Area Information Servers
file                    Host-specific file names
prospero                Prospero Directory Service
```

例如常用的 URL 地址 "http://www.baidu.com"

## URN

URN（Uniform Resource Name）：统一资源名称，以名字的方式标记互联网资源。

URN 被认为是在定义的名称空间内分配的持久性，位置无关的标识符，通常是由负责命名空间的机构负责的，因此即使它们标识的资源不再存在或变得不可用，它们也可以在全局范围内保持长期唯一性

URN 语法如下所示

```text
urn:<NID>:<NSS>
```

- urn ：协议，大小写不敏感
- NID ：是空间命名的标识，它是一个 _命名空间特定_ 字符串，决定了解释 NSS 的语法
- NSS ：特定于名称空间的字符串，NSS 可能包含 ASCII 字母和数字，以及许多标点符号和特殊字符

例如 "urn:isbn:0451450523"

## Data URI

Data URI：以 `data:` 为协议的 URI。其实对于 Data URI，平时经常称呼为 "Data URL"，`data:` 协议其实不满足于 _URL 的 Internet 通用语法_，在 URL RFC 中也没有指出存在 `data:` 特殊语法，所以称呼为 `Data URL` 不是很合适，`Data URI` 比较合适

Data URI 语法如下所示

```text
data:[<media type>][;base64],<data>
```

- data: ：协议
- media type ：[MIME](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types)，默认为 text/plain;charset=US-ASCII
- base64 ：base64 编码，如果不指出 base64 编码，则按照 URI 编码
- data ：Data URI 承载内容，如果数据是文本类型，你可以直接将文本嵌入 (根据文档类型，使用合适的实体字符或转义字符)，如果是二进制数据，你可以将数据进行 base64 编码之后再进行嵌入

### Data URI 优缺点

**优点**:

将数据嵌入文件内部，可以减少一次 HTTP 请求。比如我们经常配置 webpack 的 url-loader 和 file-loader 来处理图片

**缺点**:

1. 文件体积增大

   Data URI 如果使用 Base64 编码，将会比原文件大 1.37 倍+814 字节（头部）

   > Thus, the actual length of MIME-compliant Base64-encoded binary data is usually about 137% of the original data length, though for very short messages the overhead can be much higher due to the overhead of the headers. Very roughly, the final size of Base64-encoded binary data is equal to 1.37 times the original data size + 814 bytes (for headers)

2. _阻塞渲染_

   拿图片的 Data URI 来说，如果是外链图片的话，图片加载时可以不阻塞页面继续解析、渲染；但 Data URI 会阻塞页面继续解析和渲染

3. 图片无法使用 HTTP 缓存

   Data URI 无法使用独立的 HTTP 缓存，但是文件是可以缓存的，例如 HTML 文件缓存了，那么同时也就缓存了 Data URI

4. Data URI 解析较慢，尤其在移动端

   Data URI 内容必须由浏览器解码回其原始形式，此操作额外消耗性能（CPU、电量），浏览器每次页面渲染时必须解码图像；而二进制图像不需要额外的解码步骤

5. 每个浏览器对 Data URL 支持情况不一样

   其实这个不咋重要，IE8 以下早就该被淘汰了

### Data URI 应用

1. 存储图片，以减少 HTTP 请求
2. 本地读取文件，做图片预览
3. 前端文件下载

```javascript
// index.html，图片预览
<input type="file" name="file" id="file">

<script>
  document.getElementById('file').addEventListener('change', ev => {
    const url = URL.createObjectURL(ev.target.files[ 0 ]);

    const fileReader = new FileReader();
    fileReader.readAsDataURL(url);
    fileReader.addEventListener('load', () => {
      const img = document.createElement('img');
      img.src = url;
      img.addEventListener('load', () => {
        document.body.appendChild(img);
      });
    });
  });
</script>
```

## Blob URL、Object URL

Object URL：允许 Blog、File、MediaSource 做为 URL 源，可以用于导航（例如 img src）或者本地下载，这些 URL 只能在浏览器内部生成，且只能在当前会话中使用，

Object URL 方法如下

```typescript
URL.createObjectURL(object);
URL.revokeObjectURL(objectURL);
```

Object URL 也可以做本地图片预览、前端文件下载

```javascript
// index.html，图片预览
<input type="file" name="file" id="file">

<script>
  document.getElementById('file').addEventListener('change', ev => {
    const url = URL.createObjectURL(ev.target.files[ 0 ]);

    const img = document.createElement('img');
    img.src = url;
    img.addEventListener('load', () => {
      document.body.appendChild(img);
    });
  });
</script>
```

既然已经存在了 Data URI ，那么 Object URL 有什么优势

1. 体积更小：Blob URL 使用二进制；Data URI 为 Base64、URI 编码
2. 速度更快：Blob 只是字节序列，浏览器将其识别为字节流；Data URI 具有解码开销
3. Blob URL 加载文件为同步；FileReader 加载文件为 Data URI 为异步

留一个面试题，纯前端如何将 `{ name: "我", age: 12 }` 下载为一个 user.json 文件

[URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier)、[URL](https://en.wikipedia.org/wiki/URL)、[URN](https://en.wikipedia.org/wiki/Uniform_Resource_Name)  
[URL RFC](https://tools.ietf.org/html/rfc1738)  
[Perfomance Analysis #1: Data URIs](https://raphamorim.io/perfomance-analysis-data-uri-approach/)  
[On Mobile, Data URIs are 6x Slower than Source Linking](https://blog.catchpoint.com/2013/08/21/on-mobile-data-uris-are-6x-slower-than-source-linking/)  
[Base64](https://en.wikipedia.org/wiki/Base64#MIME)  
[Blob-URL](https://w3c.github.io/FileAPI/#blob-url)
