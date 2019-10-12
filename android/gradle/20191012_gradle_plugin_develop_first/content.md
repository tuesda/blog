## 如何从零开发一个 gradle 插件（一）



相信开发过 Android 应用的人都明白 gradle 的重要性，作为 Android 官方默认的构建工具，Android 开发者在日常开发中免不了和 gradle 打交道。而 gradle 的大部分功能都是通过插件扩展的，像我们最常用的插件就是 Android 官方插件 `com.android.application`，用来做一些和 Android 相关的配置。

那如果我们想自己开发一个 gradle 插件该怎么做呢？这里我打算用两篇文章给大家分享下，这篇文章主要介绍下 gradle 插件的相关概念，下一片文章结合一个例子展示如何一步步写一个 gradle 插件。

要知道 gradle 插件是做什么的，先要搞清楚 gradle 是什么。

### gradle

简单说 gradle 是一种构建工具，用来控制代码的编译、构建、打包等过程，有点像 C/C++ 项目中的 Make 工具。gradle 执行一次 build 总共可以分为三个步骤：

- 「initialization 初始化」
    执行 `settings.gradle` 脚本文件，确定当前项目中包含哪些子项目，Android 项目在这个阶段确定项目中有哪些 module。
- 「configure 配置」
    执行项目中的 `build.gradle` 脚本文件，创建所有需要创建的 Task。
- 「execute 执行」
    执行指定的 Task。

看来 gradle 要做的事情最终都转嫁给了 Task 来执行，让我们来看看 Task 是什么。

### Task

我们常见的编程语言的基本运行单元是代码块或者方法，按照调用的先后顺序执行。但 gradle 不同，它的工作都由一个个 Task 来执行。Task 可以指定它所依赖的 Task，或者它要在另一个 Task 之前或者之后运行。

将处理逻辑分在不同的 Task 中有这么两个好处：

- 业务解耦，有利于维护和提高代码健壮性。
- 增量编译，当 Task 的输入/输出没有变化时，不用再次运行，直接复用。

那如何创建一个 Task 呢？常见的方式有两种：

- 在 `build.gradle` 中直接创建。
- 通过插件创建。

`build.gradle` 中一般创建功能简单的 Task，逻辑复杂的 Task 通常由插件创建，否则会使得 `build.gradle` 文件臃肿不堪。可见插件也是 gradle 一个重要组成部分，我们再来看看插件是做什么的。

### 插件

gradle 核心的逻辑比较简单，丰富的构建功能都是通过插件的方式扩展的。比如 Android 的构建逻辑肯定不是 gradle 官方代码自带的，而是 Android 写了对应的 gradle 插件来实现。这个特性保证了 gradle 功能的灵活性，比如要支持 C 语言的编译，只需要写对应的插件就可以了。

作为 Android 开发者，最常见的两个插件分别是：

- `com.android.application`
- `com.android.library`

通常在 `build.gradle` 文件中这么使用插件：

``` groovy
apply plugin: 'com.android.application'
```

这表示所在的 module 是一个 app module，而使用 `com.android.library` 插件表示所在 module 是一个 library module，两种插件分别会对该 module 做不同的配置。至于是如何配置的，我们这里先不展开，后面讲到如何写一个插件时会涉及到。这里可以先理解为：应用一个插件时，相当于执行了一串代码块。

到这里我们明白了 gradle 插件是什么？那为什么需要开发 gradle 插件呢？感觉和应用开发也没多大关系。

### 为什么要开发 gradle 插件？

作为一个 Android 开发者，可能觉得我只要做出酷炫的界面，每个界面无缝切换、不卡顿就行了，为什么要费劲学 gradle 开发呢？

从我自己的经验来说，学习开发 gradle 插件有这么几个好处：

- 将一组 gradle 操作封装在一个插件中，有助于代码复用和避免 `build.gradle` 文件臃肿。
- 第三方库将自己的 gradle 操作封装在插件中，以依赖包的方式提供给别的项目使用。
- 插件可以使用 java/kotlin 来写，为不熟悉 gradle 代码的开发者降低开发难度。
- 增进对 Android 构建过程的理解，有助于学习 Android 热布丁、插件化等技术。

现在我们明白了 gradle 插件是什么，以及为什么要创建 gradle 插件，接下来让我们点击这里查看第二篇文章来看看如何一步步创建一个 gradle 插件。