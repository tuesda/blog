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

