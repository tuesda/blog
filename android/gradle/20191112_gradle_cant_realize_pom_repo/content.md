## Gradle 不能识别依赖组件的 pom 文件中的 repository 标签

Maven 仓库中的组件，会有一个 pom 文件和组件位于同一个目录，用来声明组件的一些基本信息。常见的有这个组件的依赖信息，当别的组件依赖这个组件时会自动依赖 pom 文件中包含的依赖组件。除了依赖组件信息，还有仓库信息，用来表示这些依赖组件可能存在的仓库。但在 Gradle 不支持仓库信息，必须在使用组件的地方手动加上组件依赖的仓库信息。

> https://stackoverflow.com/a/55277588

```
There are actually good reasons for the behavior of Gradle:

Gradle places a lot of importance on the source of a dependency. For Gradle, org:foo:1.0 from Maven Central and org:foo:1.0 from JCenter are considered different things. This is why repository ordering and filtering matters.

Repository hijacking has been demonstrated as an attack vector and so having a system that effectively, transitively and transparently, allows dependencies to be downloaded from any repository is not safe.

Because of these reasons, this is a change that is unlikely to happen.

There are however options for your plugin:

Shadow that exotic dependency into your own plugin

Motivate the owner of that library to publish it to a well known repository, or in your plugin's repository

Configure your plugin's repository to also mirror that other repository
```

解决办法：

创建 Gradle 插件，在插件中自动配置 repositories 信息。