Q:   
android ndkr16 编译 ffmpeg4.0  
stdlib.h: No such file or directory

A:  
出现这个错误是因为使用最新版的NDK造成的，最新版的NDk将头文件和库文件进行了分离，我们指定的sysroot文件夹下只有库文件，而头文件放在了NDK目录下的sysroot内，只需在--extra-cflags中添加 "-isysroot $NDK/sysroot" 即可，还有有关汇编的头文件也进行了分离，需要根据目标平台进行指定 "-I$NDK/sysroot/usr/include/arm-linux-androideabi"，将 "arm-linux-androideabi" 改为需要的平台就可以，终于可以顺利的进行编译了  
[ffmpeg使用NDK编译时遇到的一些坑](https://blog.csdn.net/luo0xue/article/details/80048847)

Q:  
error: expected identifier or '(' before numeric 
constant  
                     int B0 = 0, B1 = 0;
                     
A:  

需要将libavcodec/aaccoder.c里面的B0定义改一下，我是修改为b0，之后make ，编译成功;

这次编译花了我一整天的时间，遇到很多坑，这个问题让我比较纠结的，特别记录一下。


[ndk 编译 FFmpeg遇到的一个坑，附上解决方法](https://blog.csdn.net/hyjwan/article/details/80384916)

```
在module-default.sh增加：
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-linux-perf"
就可以编译通过了。
原因是：
libavcodec/hevc_mvs.c最终会include libavutil/timer.h文件
#if CONFIG_LINUX_PERF
#ifndef _GNU_SOURCE
#define _GNU_SOURCE
#endif
#include <unistd.h> // read(3)
#include <sys/ioctl.h>
#include <asm/unistd.h>
#include <linux/perf_event.h>
#endif
在sys/ioctl.h中最终会include asm/termbits .h文件
#define B0 0000000
所以在用B0作为变量或参数的地方，一般都会报错。
```
[ffmpeg编译错误](https://github.com/Bilibili/ijkplayer/issues/4093)
