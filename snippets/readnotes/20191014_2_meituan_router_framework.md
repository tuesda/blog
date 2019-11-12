WMRouter: 美团外卖 Android 开源路由框架 阅读笔记

> https://tech.meituan.com/2018/08/23/meituan-waimai-android-open-source-routing-framework.html

文章结构：

- 功能简介

    - URI 分发

        可用于多工程之间页面跳转，动态下发URI链接跳转，特点如下：

        - 支持多scheme、host、path
        - 正则匹配
        - 动态注册或者注解配置
        - 支持全局拦截/局部拦截
        - 支持参数设置、跳转动画设置、自定义startActivity
        - 支持页面exported控制
        - 支持全局/局部降级跳转
        - 支持全局/单次跳转监听
        - 组件化设计

    - ServiceLoader

        基于[SPI]()设计思想，提供了ServiceLoader模块，通过接口调用代码，实现模块解耦，特点如下：

        - 注解自动配置
        - 支持获取所有的接口实现，或根据key获取特定实现
        - 支持获取Class或实例
        - 支持无参构造、Context构造或自定义Factory、Provider构造
        - 支持单例管理
        - 支持方法调用

    - 其它特性：

        - 优化 gradle 插件，加快编译速度
        - 编译时和运行时检查，避免配置错误
        - 编译器自动配置proguard混淆规则
        - 完善调试功能

- 适用场景

    - Native+H5混合开发，页面互相跳转，需要下发运营跳转链接，跳转前进行特定的处理。
    - 管理来自App外的URI跳转，需要先展示欢迎页。
    - 页面跳转前有复杂的判断逻辑或前置操作。
    - 多工程、组件化开发。
    - 页面跳转埋点需求。
    - 线上问题检测，及出现问题的降级操作。
    - A/B测试，动态下发路由。

    > 第 1、2、3条和最后一条有点重复

- 基本概念解释

    - 路由

        通过特定的协议把信息从源地址传输到目的地址的过程。

    - URI

        URI = Uniform Resource Identifier，统一资源标识符

        URI的组成部分主要包括：scheme、host、path和query

    - Android中的Intent跳转

        - 显式跳转

            指定Component（类名）的跳转。

        - 隐式跳转

            不指定Component，通过IntentFilter找到匹配的组件。

- URI跳转

    - 老实现

        使用IntentFilter隐式跳转，这种实现方案的问题：

        - 没有经过欢迎页面，无法进行准备操作。
        - 每个页面都需要准备操作，代码冗余。
        - 容错性较差，无法进行错误检测。

        解决办法：建立一个分发组件，统一处理URI分发。

    - 核心设计思路

        ![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/cbac1e74.png)

        借鉴网络请求机制，每次URI分发看作一个网络请求，被Router逐层分发给不同的UriHandler，每个UriHandler之前都有一个UriInterceptor，做一些预先处理或拦截掉URI分发请求。

    - 页面跳转来源

        - App内Native页面
        - App内Web页面
        - App外通过URI，微信、浏览器
        - 通知中心push

    - UriRequest

        为了简化数据结构，URI分发请求的响应也用UriRequest结构（读者：在维护性上考虑感觉不该偷这个懒）

        UriRequest的结构：context、uri、fields，fields主要有以下这几种情况：

        - Intent的Extra参数，bundle类型
        - 用于startActivityForResult的requestCode，int类型
        - 用于overridePendingTransition方法的页面切换动画资源，int[] 类型
        - 跳转结果的监听器，onCompleteListener 类型

        每次跳转的结果里会有一个ResultCode（类似http里的responseCode）：

        - 200：跳转成功
        - 301：重定向
        - 400：请求错误，Context或URI为空
        - 403：禁止跳转，例如白名单外的http链接
        - 404：找不到目标，
        - 500：发生错误

    - UriHandler

        UriHandler可以嵌套从而逐层处理UriRequest，处理过程可以是异步的，当前UriHandler可以决定处理或者交给下一个UriHandler处理。

    - UriInterceptor

        ​	拦截器，不做最终跳转，可以做一些前置操作，也是异步的：

        - 跳转拦截，返回错误代码，例如 403、400
        - 参数修改，例如http链接加上query参数
        - 各种前置处理，比如登录、定位

        一个UriHandler可以有多个UriInterceptor

    - URI的分发和降级

        ![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/c846871f.png)

        URI分发示意图

        - 可以在RootUriHandler全局降级处理，也可以在每个UriHandler局部降级。

- 平台化与两端复用

    美团外卖的架构：

    ![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/61d0fa17.png)

    - 通信问题

        通过URI请求解决这个问题

        ![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/6a070dbf.png)

        URI分发解决的问题：

        - 运营需求动态下发跳转URI
        - App内不同组件间的页面跳转

        一条最佳实践：和外部多端统一制定的跳转URI使用一个scheme，App内部的页面跳转使用另一个scheme。

    - 复用问题

        **ServiceLoader**

        - 除了通过接口获取所有实现，还可以通过key获取制定实现。
        - 可直接夹在实现类。（读者：不太明白为什么要获取实现类，这和解耦不是矛盾了么，既不必要也不应该）
        - 用Factory和Provider创建对象
        - 单例控制（读者：感觉这个和上一个不值得指出来夸奖）
        - 优化gradle插件提高编译速度。

        ![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/9e35161a.png)

        1. 接口下沉到平台层，实现类实现接口，并声明一个key用来绑定。
        2. 调用方通过接口和key获取实现类实现调用。

    - 依赖注入

        依赖注入就是指底层库调用上层库，而对WMRouter来说主工程、业务库和平台库都是一样的，所以依赖注入也可以用ServiceLoader实现。

    - 编译问题

        主工程和业务库耦合严重导致经常出现无法编译的情况，为了解决这个问题：

        - WMRouter增加了注解支持，解决跳转页面注册的问题
        - ServiceLoader提供方法加载特定接口的所有实现类，来实现主工程和业务库的隔离。（读者：看来这里的编译问题是指之前在主工程里直接创建业务库里接口的实现类插入到ServiceLoader里，感觉这里可以通过配置解决初始化时插入实现类的问题。）

- WMRouter的推广

    WMRouter提供了很多灵活的接口用来实现功能，但对于简单的需求来说配置有点复杂，降低了易用性。所以WMRouter对代码进行了划分：

    - 核心接口
    - 通用实现类

    一方面保留了最大的灵活性，另一方面通过默认实现类降低使用门槛，方便推广。