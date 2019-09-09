## gradle and android plugin

>  https://developer.android.com/studio/build

Android Studio 使用 Gradle 编译 Apk，AGP(Android Gradle Plugin) 是专门针对 Android 编译的 Gradle 插件。

Gradle 和 AGP 不依赖 Android Studio，可在命令行中运行。

Android 的编译代码和源代码隔离，这篇文章会讲到 Apk 的编译过程以及如何配置。

### 编译过程

1. 编译器将源代码编译成 dex 文件和 resources 文件。
2. APK Packager 将 dex 文件和 resources 文件打包成 Apk，打包时会根据配置分别使用 release keystore 或 debug keystore 生成 release 或 debug 版本。
3. zipalign 工具会对 apk 进行瘦身，从而降低 apk 内存占用。

### 编译配置

1. BuildType

    用来表示当前开发阶段，默认有 release / debug

2. Product Flavors

    发布版本，比如付费版和免费版，不同的版本可以使用不同的代码和资源文件，默认不启用

3. Build Variants

    build variant = buildType + productFlavor

4. Manifest Entries

    自定义变量，可以根据 buildVariant 变化，在 AndroidManifest.xml 中使用

5. Dependencies

    两种依赖方式：本地文件系统和远端网络

6. Signing

    debug 可以自动签名，release 必须手动置顶签名配置

7. ProGuard

    指定自定义混淆规则

8. Multiple APK Support

    可以针对不同屏幕和 ABI 编译不同的 apk。



### 编译配置文件

`*.gradle` 使用的 groovy 语言的 DSL 编写，groovy 是针对 JVM 的动态语言。

####  `settings.gradle`

在根目录下，用来指定需要包含哪些 module

``` groovy
include ':app'
```

#### 根目录的 `build.gradle`

1. 用来配置适用所有「模块 module」的规则。

2. 配置所有模块共享的变量

    ``` groovy
    ext {
        compileSdkVersion = 28
    }
    ```

    在 module 的 build.gradle 中这样引用：

    ``` groovy
    rootProject.ext.compileSdkVersion
    ```

    但为了解耦和增加编译速度，尽量避免多个模块共享变量

#### 模块级的 `build.gradle`

针对模块的编译配置，配置 buildType、productFlavor、buildVariants等，可以覆盖根目录下 `build.gradle` 的配置。

#### Gradle 属性文件 `*.properties`

1. `gradle.properties`

    项目级别的属性配置，例如 gradle daemon 的 max heap size.

2. `local.properties`

    1. `ndk.dir` 弃用，必须在 `sdk.dir` 的 ndk 文件夹内
    2. `sdk.dir` 指向 sdk 路径
    3. `cmake.dir` 指向 cmake 路径
    4. `ndk.symlinkdir` ndk 路径的简短版

#### SourceSets

- `src/main`
- `src/buildType`
- `src/productFlavor`
- `src/productFlavorBuildType`

"fullDebug" variant 的 apk 会使用下面几个文件夹下的代码和文件：

- `src/fullDebug`
- `src/debug`
- `src/full`
- `src/main`

优先级

```
build variant > build type > product flavor > main source set > library dependencies
```



