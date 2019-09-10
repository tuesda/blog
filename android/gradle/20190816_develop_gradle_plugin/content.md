## 开发 Gradle 插件

Android Studio 中使用 Gradle 来执行 Android 项目的编译过程，主要的 Gradle 脚本有：

- `settings.gradle`
- 根目录的  `build.gradle`
- module 目录的 `build.gradle`

Gradle 是一门基于插件的语言，脚本中一般通过插件应用特定的编译功能，例如 module 下的 `build.gradle` 中：

``` groovy
apply plugin: 'com.android.application'
```

应用了一个名叫 `com.android.application` 的插件，从名字可以看出来这是针对 Android 项目的，里面包含了和 Android 编译相关的一些逻辑。本文将会讲解如何创建一个自己的 Gradle 插件以及如何使用它，首先我们先简单了解下 Gradle。

### Gradle

- Gradle 是什么？
    - 用来控制编译过程的编译系统
    - Gradle 编译过程主要分三大步
        - 「初始化」确定都有哪些 project
        - 「配置」所有 project 的编译脚本被执行，创建所有需要的 task
        - 「执行」执行第二步创建的 task
- Gradle 插件是什么？
    - AGP：Android Gradle Plugin，Google 开发的专门用于控制 Android 项目编译过程的 Gradle 插件

- 为什么要开发 Gradle 插件？

- 怎么开发 Gradle 插件？
    - 用什么 IDE ？
    - 用什么语言？