## configure、 make、 make install 背后的原理

原文：<https://thoughtbot.com/blog/the-magic-behind-configure-make-make-install>

如果你之前使用过 Unix 系设备开发，你可能使用过下面这几行命令来安装软件：

``` shell
./configure
make
make install
```

我使用过很多次，但在我刚开始使用 Linux 的时候并不知道这几行命令的意思，只知道安装软件的时候在命令行输入这几行命令就行了。

最近我在开发一个 Unix 工具，所以想弄明白这个标准化安装命令背后的原理。不止 Unix 用户对这几行命令很熟悉，如果要开发一款针对 Homebrew、 Linux 或者 BSD 包管理器的应用，这也是一个很重要的知识点。接下来让我们深入 Unix 去搞清楚这几行命令的作用。

### 做了什么

整个过程分为三步：

1. 配置

   `configure` 脚本负责在你使用的系统上准备好软件的构建环境。确保接下来的构建和安装过程所需要的依赖准备好，并且搞清楚使用这些依赖需要的东西。

   Unix 程序一般是用 C 语言写的，所以我们通常需要一个 C 编译器去构建它们。在这个例子中 `configure` 要做的就是确保系统中有 C 编译器，并确定它的名字和路径。

2. 构建

   当 `configure` 配置完毕后，可以使用 `make` 命令执行构建。这个过程会执行在 `Makefile` 文件中定义的一系列任务将软件源代码编译成可执行文件。

   你下载的源码包一般没有一个最终的 `Makefile` 文件，一般是一个模版文件 `Makefile.in` 文件，然后 `configure` 根据系统的参数生成一个定制化的 `Makefile` 文件。

3. 安装

   现在软件已经被构建好并且可以执行，接下来要做的就是将可执行文件复制到最终的路径。`make install` 命令就是将可执行文件、第三方依赖包和文档复制到正确的路径。

   这通常意味着，可执行文件被复制到某个 `PATH` 包含的路径，程序的调用文档被复制到某个 `MANPATH` 包含的路径，还有程序依赖的文件也会被存放在合适的路径。

   因为安装这一步也是被定义在 `Makefile` 中，所以程序安装的路径可以通过 `configure` 命令的参数指定，或者 `configure` 通过系统参数决定。

   如果要将可执行文件安装在系统路径，执行这步需要赋予相应的权限，一般是通过 sudo。

### 这些脚本是怎么产生的

安装过程简单说就是 `configure` 脚本根据系统信息将 `Makefile.in` 模版文件转换为 `Makefile`文件，但是 `configure` 和 `Makefile.in` 文件是怎么产生的呢？

如果你曾经试着打开 `configure` 或者 `Makefile.in` 文件，你会发现超长而且复杂的 shell 脚本语言。有时候这些脚本代码比它们要安装的程序源代码还要长。

如果想手动创建一个这样的 `configure` 脚本文件是非常可怕的，好消息是这些脚本是通过代码生成的。

通过这种方式构建的软件通常是通过一个叫做 `autotools` 的工具集打包的。这个工具集包含 `autoconf` 、`automake` 等工具，所有的这些工具使得维护软件生命周期变得很容易。最终用户不需要了解这些工具，但却可以让软件在不同的 Unix 系统上的安装步骤变得简单。

### Hello world

我们以一个 Hello world 的简单 C 程序为例，来看看如何使用 `autotools` 打包。

下面是程序源码，源代码文件命名：`main.c`

``` c
#include <stdio.h>

int main(int argc, char* argv[]) {
  printf("Hello world\n");
  return 0;
}
```

### 创建 configure 脚本

我们不直接写 `configure` 脚本文件，而是通过创建一个描述文件 `configure.ac` 来描述 configure 需要做的事情。`configure.ac` 使用 m4sh 写，m4sh 是 `m4` 宏命令和 shell 脚本的组合。

第一个用到的宏命令是 `AC_INIT`，这个命令会初始化 autoconf 并配置一些关于软件的基本信息。下面这行代码表示，软件名是 `helloworld`，版本是 `0.1`，维护作者是  `george@thoughtbot.com`：

``` m4sh
AC_INIT([helloworld], [0.1], [george@thoughtbot.com])
```

因为这个项目需要用到 `automake`，所以我们要用下面这个命令来初始化它：

``` m4sh
AM_INIT_AUTOMAKE
```

接下来，我们需要告诉 `autoconf` configure 脚本需要的依赖。在这个例子中，configure 需要的只是 C 编译器，我们可以用下面这个宏命令来设置：

``` m4sh
AC_PROG_CC
```

如果我们需要别的依赖，可以使用别的 `m4` 宏命令来设置；例如 `AC_PATH_PROG` 表示在 `PATH` 上搜索一个特定的程序。

此时我们已经列出了所有的依赖，我们可以使用它们。前面有提到， `configure` 脚本会根据系统的信息和 `Makefile.in` 文件生成 `Makefile` 文件。	

下面这个宏命令 `AC_CONFIG_FILES` 表示让 autoconf 配置 configure 脚本找到 `Makefile.in` 文件，并将文件内的占位符用对应的值替换，例如将 `@PACKAGE_VERSION@` 替换成 `0.1`，然后将结果写在 `Makefile` 文件中。

``` m4sh
AC_CONFIG_FILES([Makefile])
```

最后，当我们把所有配置信息都告诉 autoconf 后，可以使用 `AC_OUTPUT` 命令去输出脚本：

``` m4sh
AC_OUTPUT
```

下面这段代码是 `configure.ac` 中的所有代码，相比 4737 行的 `configure` 脚本文件，这些代码好懂多了

``` m4sh
AC_INIT([helloworld], [0.1], [george@thoughtbot.com])
AM_INIT_AUTOMAKE
AC_PROG_CC
AC_CONFIG_FILES([Makefile])
AC_OUTPUT
```

还差一点我们就可以发布软件了，`configure` 脚本需要一个 `Makefile.in` 文件，将系统相关信息填充进去后，生成最终的 `Makefile` 文件。

### 创建 Makefile 文件 

