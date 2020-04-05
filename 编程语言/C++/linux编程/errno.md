# 什么是errno

errno 是记录系统的最后一次错误代码。代码是一个int型的值，在errno.h中定义。查看错误代码errno是调试程序的一个重要方法。当linux C api函数发生异常时,一般会将errno变量(需include errno.h)赋一个整数值,不同的值表示不同的含义,可以通过查看该值推测出错的原因。在实际编程中用这一招解决了不少原本看来莫名其妙的问题，特别是在进行系统函数调用时。

errno 错误值对应主要在于/usr/include/asm-generic/errno.h及errno-base.h

# 打印错误信息

**perror**

const char *s: strerror(errno) //提示符：发生系统错误的原因

**strerror**

字符串显示错误信息，

```
#include <string.h>  
char *strerror(int errnum);
```

返回错误码字符串信息

# 使用案例

```C++
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <fcntl.h>

int main(void)
{
    int   fd;
    extern int errno;

    if((fd = open("/dev/dsp",O_WRONLY)) < 0)
    {
        printf("errno=%d\n",errno);
        char * mesg = strerror(errno);
        printf("Mesg:%s\n",mesg);
    }
    return 0;
}
```



