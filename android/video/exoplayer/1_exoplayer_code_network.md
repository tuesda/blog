# Exoplayer源码阅读之网络请求

Exoplayer 是一款很强大的音视频播放器类库，除了可以播放手机本地的资源，也可以播放网络上的。要播放网络上的音视频，肯定少不了网络请求相关的组件，这篇文章主要来分析 Exoplayer 中和网络请求相关的类。

如果你要播放一个网络上的mp3链接，你需要构建一个代表这个链接的 `MediaSource`

```kotlin
val dataSourceFactory = DefaultHttpDataSourceFactory(context, "<userAgent>")
ProgresssiveMediaSource.Factory(dataSourceFactory)
  .create(Uri.parse("https://xxxxx.mp3"))
```

根据这几个类的名字很容易发现 `DefaultHttpDataSourceFactory` 是否则 http 网络请求的，而 DataSource 也进一步说明这个类负责数据来源，也就是要播放的mp3资源。



进入 DefaultHttpDataSourceFactory 类可以发现这个类的作用是创建一个 DefaultHttpDataSource, 我们继续进入这个类发现它实现了 HttpDataSource 的接口，而 HttpDataSource 继承了 DataSource 接口。很明显，DataSource 接口负责获取数据，而 HttpDataSource 接口负责和网络相关的数据。



## DataSource

先看看 `DataSource` 接口：



```java
public interface DataSource {  
  
  long open(DataSpec dataSpec) throws IOException;
  
  int read(byte[] buffer, int offset, int readLength) throws IOException;
  
  void close() throws IOException;
  
  // 忽略掉其它不重要的方法
}
```

可以看到 DataSource 主要的方法是 `open()` `read()` 和 `close()`

- open() 打开媒体文件准备从中读取数据，dataSpec 表示需要打开的数据源
- read() 读取媒体数据到 buffer 中，读取的大小最多不超过 readLength，实际读取的大小作为返回值返回
- close() 关闭之前打开的媒体文件



## HttpDataSource

再看看 `HttpDataSource` 接口：



```java
public interface HttpDataSource extends DataSource {
	// 忽略和 DataSource 相同的方法
  
  void setRequestProperty(String name, String value);
  
  void clearRequestProperty(String name);
  
  void clearAllRequestProperties();
  
  int getResponseCode();
}
```

- setRequestProperty: 设置 http 请求 header
- clearRequestProperty: 删除某个指定的 http 请求 header
- clearAllRequestProperties: 删除所有的 http 请求 header
- getResponseCode: 获取 http 请求返回状态码

可以发现，HttpDataSource 新增了 http 请求相关的操作，这和它的名字很符合。



Exoplayer 中 HttpDataSource 接口有三个实现类：

- DefaultHttpDataSource 使用 HttpURLConnection 实现网络请求
- OkHttpDataSource 使用 OkHttp 实现网络请求
- CronetDataSource 使用 Cronet 实现网络请求（Cronet：Chrome 网络栈，支持 http1 http2)



我们以 DefaultHttpDataSource 为例看一下网络请求的流程，另外两个逻辑也差不多，读者感兴趣的话可以直接看它们的源码。



## DefaultHttpDataSource

我们之前讲过 DataSource 主要的方法是: `open` 、`read` 和 `close`，那就先来看下这三个方法的实现。



### open

open 做的事情主要有这几项：

- 创建 connection
- 获取 responseCode 并判断有效性
- 判断 contentType 有效性
- 获取 bytesToSkip / bytesToRead
- 获取 inputStream

#### 创建 connection

通过 `URL.openConnection()` 创建一个 HttpURLConnection 实例，然后将相关属性配置在这个 connection 上，这些属性有

- connect 超时值
- read 超时值
- 请求 header，例如 `User-Agent` `Accept-Encoding`
- 如果指定了请求文件大小，还要指定 `Range` header
- request method
- request body

如果 `allowCrossProtocolRedirects` 指定为 true，还需要手动进行重定向，因为 HttpURLConnection 不支持自动跨协议的重定向，如果不是跨协议的，直接通过 `HttpURLConnection.setInstanceFollowRedirects(true)` 配置就可以了。

#### 获取 responseCode 并判断有效性

获取到 responseCode 后进行判断，如果不在 `200..299` 这个范围内就认为请求错误，抛出一个 `InvalidResponseCodeException` 的错误表示请求失败，调用放则可以根据这个异常进行后续处理。这个异常中也会带上请求返回的 headers 和 responseMessage，为错误处理提供一些线索。

#### 判断 ContentType 有效性

DefaultHttpDataSource 有一个方法 `void setContentTypePredicate(@Nullable Predicate<String> contentTypePredicate)` 用来指定哪些 contentType 可以处理。这个 contentTypePredicate 就是在这里用到的，如果发现当前请求返回的 contentType 不被接受，则抛出一个 `InvalidContentTypeException` 表示 contentType 异常。

#### 获取 bytesToSkip/bytesToRead

bytesToSkip 表示需要跳过的字节数，为什么需要跳过一些字节数而不是从第0个字节开始读呢？

当我们从某个资源文件的中间请求时，此时表示资源文件的 `dataSpec` 中的 `position != 0` ，会在 request header 中设置 `Range` 字段，表示需要读取的范围例如 `Range:bytes=1500-4000/6000`。正常情况下返回的 responseCode 应该是 `206(Partial Content)`，表示返回部分内容即我们要的第 1500 字节到4000字节，但有的内容服务器不支持部分内容请求，将整个内容返回过来，此时 responseCode 是 `200`。

因此当我们发现，`dataSpec.position != 0 && responseCode == 200` 时，就需要计算 bytesToSkip，并在读取内容时考虑在内，这里的 `bytesToSkip = dataSpec.position`。

bytesToRead 表示需要读取的内容字节长度，它的计算方式如下: 

``` java
boolean isCompressed = isCompressed(connection);
if (!isCompressed) {
  if (dataSpec.length != C.LENGTH_UNSET) {
    bytesToRead = dataSpec.length;
  } else {
    long contentLength = getContentLength(connection);
    bytesToRead = contentLength != C.LENGTH_UNSET ? (contentLength - bytesToSkip)
        : C.LENGTH_UNSET;
  }
} else {
  // Gzip is enabled. If the server opts to use gzip then the content length in the response
  // will be that of the compressed data, which isn't what we want. Always use the dataSpec
  // length in this case.
  bytesToRead = dataSpec.length;
}
```
可以简单分为这几种情况：

- 当内容没有压缩时，并且 dataSpec 没有指定请求的长度，此时根据 response headers 计算内容长度：
	- response headers 计算的结果如果有效？减去 bytesToSkip，得到的就是 bytesToRead
	- 否则 `bytesToRead = C.LENGTH_UNSET`
- 其余情况 `bytesToRead = dataSpec.length`

#### 获取 inputStream

通过 `HttpURLConnection.getInputStream()` 即可获取到 inputStream，并将它记为成员变量，以供在 read 方法读取内容，如果返回的内容是被压缩的，需要再通过 `GZIPInputStream` 包一下刚返回的 inputStream：

``` java
inputStream = new GZIPInputStream(inputStream);
```

### read

相比 open 方法 read 方法简单很多，就包含两个步骤，skip 和 read，分别由 skipInternal 和 readInternal 两个方法负责。

#### skipInternal

上面 open 方法里计算的 bytesToSkip 就是在这里用的，如果 bytesToSkip == 0，这里就什么都不用做，否则需要跳过指定数目的字节。跳过的原理也很简单，创建一个长度 4096 的 byte 数组，不停得将 inputStream 中的内容写到这个数组中，直到达到 bytesToSkip 的值。

#### readInternal

readInternal 的方法签名如下

``` java
int readInternal(byte[] buffer, int offset, int readLength)
```

- buffer 用来存储读到的内容
- offset 表示 buffer 中存储的起始坐标
- readLength 最多希望读取的字节长度
- 返回实际读取的字节长度

这个方法也比较简单，核心代码就这一行：

``` java
int read = inputStream.read(buff, offset, readLength);
```

重点是读取前对 readLength 的调整，以及读取之后判断是否到了文件末尾。

### close

close 方法的实现最简单：`inputStream.close()`，以及状态变量的重置和释放。


