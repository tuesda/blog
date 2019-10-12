## 如何从零开发一个 gradle 插件（二）



上一篇我们介绍了 gradle 插件相关的概念，没看的同学可以点击这里查看，这一篇让我们结合例子来看看如何一步步创建一个 gradle 插件，并在 Android 项目中使用。

### IDE

工欲善其事必先利其器，要开发 gradle 插件得先决定用什么开发环境。我自己使用过 Intellij IDEA 和 AndroidStudio 来开发，因为 AndroidStudio 是基于 Intellij 改的，所以区别不是很大。

但就我个人的偏好来说更偏向于 Intellij，因为 AndroidStudio 是专门用来开发 Android 的，有很多配套的调试按钮，如果用来开发一个纯的 gradle 插件项目就会很别扭，所以我的判断原则是：

- 如果是一个纯的 gradle 插件项目，用 Intellij IDEA。
- 如果是 Android 项目中内嵌的 gradle 插件项目，比如一个 module 用来放插件的代码，用 AndroidStudio。

### 语言

gradle 插件比较主流的开发语言是 groovy，因为 gradle 就是用 groovy 写的。通过查看第三方写的 gradle 插件代码发现大部分也是用 groovy 写的。

除了 groovy 还可以用来开发 gradle 插件的语言有：

- java
- kotlin
- 其它基于 jvm 语言

因为 groovy 是基于 jvm 的，所以同样基于 jvm 的也可以用来写 gradle 插件。但 java 和 kotlin 是静态语言，意味着需要不停地判断类型和强转，相比之下，groovy 作为动态语言就没有这个问题。所以本文后续的插件代码都是 groovy 写的。

具体开发之前让我们先来看看要开发的 gradle 插件要满足什么需求。

### 需求描述

实现一个插件，名字叫 `io.helloworld.myplugin`，需要实现下面的功能：

- 进行 Android 自动配置
	- minSdkVersion 设置为 21
	- targetSdkVersion 设置为 26
	- compileSdkVersion 设置为 29
	- buildToolsVersion 设置为 ‘29.0.2’
- 注册一个 Task，名字叫 `myTask`。

### 具体步骤

- 打开 Intellij IDEA，按 file -> new -> project 创建一个新项目。在 「new project」对话框左边选择 gradle，右边的 「Additional Libraries and Frameworks」只选择 Groovy：

	<img src="http://tva1.sinaimg.cn/large/007X8olVly1g7ve7gm6dfj30yt0u0tdi.jpg" width="50%" />

- 下一个界面选择代表你这个插件包的 groupId 和 ArtifactId，例如 `io.helloworld.robot` 和 `gradletools`，版本号可以保持默认不变 `1.0-SNAPSHOT`

	> groupId、artifactId 和 version 构成了我们这个插件包的标识 Id，另一个项目如果想使用我们这个插件时，就需要通过这个 Id 来声明依赖。

	<img src="http://tva1.sinaimg.cn/large/007X8olVly1g7ve7h4fhzj30xq0u0jtm.jpg" width="50%" />

- 然后点 「next」-> 「finish」，项目结构大概像下面这样：

	<img src="http://tva1.sinaimg.cn/large/007X8olVly1g7ve7hhwsnj30mu0lidhc.jpg" width="50%" />

- 因为要用到 gradle 的依赖，所以需要在项目 build.gradle 脚本文件中加一条 gradle 的依赖：

	``` groovy

	dependencies {
		implementation gradleApi()
		...
	}
	```

	> 创建项目时，IDE 会默认添加上 groovy 的依赖：
	> ``` groovy
	> implementation 'org.codehaus.groovy:groovy-all:2.3.11'
	> ```
	> 为了防止出现 groovy 版本不一致的问题，这里的依赖最好换成下面这样，让 gradle 自动去获取本地的 groovy 版本。
	> ``` groovy
	> implementation localGroovy()
	> ```
	
	然后点一下 gradle 的同步按钮，保证 IDE 能引用到对应的类。

	<img src="http://tva1.sinaimg.cn/large/007X8olVly1g7ve7hpoi9j30k809yjrt.jpg" width="50%" />

- 在 groovy 文件夹下创建包 `io.helloworld.robot`，和前面设置的 groupId 保持一致。在这个包下创建一个 groovy 类 `MyPlugin`，代码如下：

	``` groovy
	package io.helloworld.robot

	import org.gradle.api.Plugin
	import org.gradle.api.Project

	class MyPlugin implements Plugin<Project> {
		@Override
		void apply(Project project) {
			println("beginning of plugin 'MyPlugin'")
		}
	}
	```

	可以看到 `MyPlugin` 实现了 `Plugin<Project>` 的接口，而这个接口有一个方法 `apply(Project project)`，猜测当我们这个插件在 `build.gradle` 中应用时，这个方法会被调用，我们在这个方法中加上一行打印调试信息的代码。

	> 这里创建的插件是给 `build.gradle` 用的，所以 `Plugin` 的泛型类型是 `Project`，如果给 `settings.gradle` 用的话，泛型类型应该是 `Settings`。

	现在我们的插件已经创建好了（虽然很简陋），那怎样才能让 Android 项目使用到它呢？还需要再做两件事：创建插件的声明文件和将包含插件的软件包发布到 maven 仓库。

- 创建插件的声明文件

	需要在 resources 文件夹下创建文件夹 META-INF/gradle-plugins，所有插件的声明文件都要放在这个目录下：

	<img src="http://tva1.sinaimg.cn/large/007X8olVly1g7ve7i44kdj30jk04ct98.jpg" width="50%" />

	插件的声明文件都是后缀为 `properties` 的配置文件，文件名就是插件的名字，例如这里的插件名字就是 `io.helloworld.myplugin`。在文件中通过 `implementation-class` 的值关联到上面创建的插件类 `MyPlugin`：

	```
	implementation-class=io.helloworld.robot.MyPlugin
	```

- 发布到 maven 仓库

	因为 maven 仓库不是这篇文章的重点，所以简单起见，这里将包含插件的软件包发布到本地仓库。

	要发布到本地仓库，`build.gradle` 需要加上下面这些代码：

	``` groovy
	apply plugin: 'maven-publish'

	publishing {
		publications {
			mavenPub(MavenPublication) {
				// 这一行表示将 jar 包包含在要发布的组件中
				from components.java
				// 描述性信息
				pom {
					name = 'Robot MyPlugin'
					description = 'This is a test project for gradle plugin.'
					url = 'http://www.helloworld.com'
				}
			}
		}
		repositories {
			maven {
				// 本地 repo 地址，这里写上你们电脑上自己的地址
				url = '/Users/zhanglei/projects/repo'
			}
		}
	}
	```

	这里使用 `maven-publish` 插件来发布到 maven 仓库，其实发布到网络上也是类似的，将上面代码的 `url` 改成网络地址，再加上用户名/密码就可以了。

	改好 `build.gradle` 后点一下 gradle 的同步按钮，在 gradle 窗口的 Tasks 条目下就会看到 publishing 的 Task 分组，其中可以找到「publish」Task：

	<img src="http://tva1.sinaimg.cn/large/007X8olVly1g7ve7iedtvj30ow0laq54.jpg" width="50%" />

	双击执行这个 Task 来发布组件，执行结束后在上述本地仓库的地址下可以看到组件已经发布成功：

	<img src="http://tva1.sinaimg.cn/large/007X8olVly1g7ve7iy8otj30om0iu0ve.jpg" width="50%" />

	OK，所有的准备工作都做完了，现在可以使用插件了。

- 在 Android 项目中使用插件

	因为我们的插件是配置 Android 相关信息，所以要在 application 类型的 module 下使用。在 AndroidStudio 中打开 module 下的 `build.gradle` 文件，加上下面的代码：

	``` groovy
	// 使用插件
	apply plugin: 'io.helloworld.myplugin'
	buildscript {
		repositories {
			// 声明本地 repo 的地址
			maven {
				url '/Users/zhanglei/projects/repo'
			}
		}
		dependencies {
			// 声明依赖我们上面发布到本地 maven 仓库的软件包
			classpath 'io.helloworld.robot:gradletools:1.0-SNAPSHOT'
		}
	}
	```

	因为插件发布在本地仓库中，所以要在 buildscript.repositories 中声明一下仓库地址。然后在 AndroidStudio 中同步下 gradle 就可以看到我们刚才写在 `apply()` 中的调试代码输出的 log：

	<img src="http://tva1.sinaimg.cn/large/007X8olVly1g7ve7jd7euj30uk0fwju2.jpg" width="50%" />

	> AndroidStudio 中同步 gradle 和 Intellij 中稍微有点不一样：点下面这个按钮：

	> <img src="http://tva1.sinaimg.cn/large/007X8olVly1g7ve7jhl6vj30cg04074c.jpg" width="50%" />

	好的，我们现在把开发插件到使用插件这条路调通了，接下来我们在插件中完成上面说的「插件需求」，先来看看第一个：

- 实现插件功能一：进行 Android 自动配置

	> - minSdkVersion 设置为 21
	> - targetSdkVersion 设置为 26
	> - compileSdkVersion 设置为 29
	> - buildToolsVersion 设置为 29.0.2

	在 Intellij 中打开 MyPlugin 类，在 `apply()` 方法中加上下面这段代码：

	``` groovy
	project.extensions.findByName('android').with {
		getProperty('defaultConfig').with {
			setProperty('minSdkVersion', 21)
			setProperty('targetSdkVersion', 26)
		}
		setProperty('compileSdkVersion', 29)
		setProperty('buildToolsVersion', '29.0.2')
	}
	```

	然后发布到本地仓库，然后在 AndroidStudio 中同步下 gradle，这就相当于在 `build.gradle` 中写了：

	``` groovy
	android {
		defaultConfig {
			minSdkVersion 21
			targetSdkVersion 26
		}
		compileSdkVersion 29
		buildToolsVersion '29.0.2'
	}
	```
	
	第一个需求完成了，接下来看一下第二个需求。

- 实现插件功能二：注册一个 Task `myTask`

	打开 `MyPlugin`，在 `apply()` 方法中加上下面这段代码：

	``` groovy
	project.task('myTask') {
		group 'helloworld'
		description 'This is a task named myTask, created in myPlugin'
		doFirst {
			println("run in myTask it $it")
		}
	}
	```

	这里创建了一个名叫 `myTask` 的 Task，并设置了 group、description 和它的运行代码。和上面一样，将插件发布到本地仓库，并同步 Android 项目的 gradle。可以看到 Android 项目中已经存在 `myTask` Task：

	<img src="http://tva1.sinaimg.cn/large/007X8olVly1g7ve7joe5lj30tu0mk0uc.jpg" width="50%" />

到这里一个拥有基本功能的 gradle 插件就开发好了，如果想继续扩展这个插件的功能，就继续在 `apply()` 方法里添加代码。因为 `apply()` 方法的参数是 project，所以这里可以实现 `build.gradle` 中的所有能力。你也可以在这个项目中开发一个新的插件，放一些特殊的 gradle 处理逻辑。

关于 gradle 插件开发就简单介绍到这里，由于我也是边开发边学习，难免有一些不对的地方，欢迎大家评论指正。