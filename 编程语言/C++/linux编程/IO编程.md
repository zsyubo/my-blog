# 文件描述符

linux中，所有对设备或文件的操作都是通过文件描述符进行的。当打开或创建一个文件时，内核向进程返回一个文件描述符(非负整数)。后续对文件的操作只需要通过文件描述符，内核会记录操作问价你的信息。

**默认文件描述符**

`一个进程启动，默认会打开3个文件：标准输入(0)、标准输出(1)、标准错误(2)`，这些常量定义在`unistd.h`中。

**文件指针可以和文件描述符之间互相转换：**

```c++
// 文件指针-> 文件描述符
int fileno(FILE *stream)
//文件描述符->文件指针
FILE *fdopen(int fd,const char *mode);
```

**打印stdin的文件描述符值**

```c++
cout << fileno(stdin) << endl; //0
cout << fileno(stdout) << endl;//1
cout << fileno(stderr) << endl;//2
打开其他文件 // 3
```

# 打开、创建文件

linux提供open函数来打开或者创建一个文件，返回fd。

```c++
// 引入头文件
#include <fcntl.h>
// 如果函数执行成功，就返回文件描述符，失败返回-1。
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);
```

open有打开数量限制，默认最大打开各位限制为1024

使用命令`ulimit  -a`来查看。

- pathname： (绝对或者相对)路径+文件名称
- flags: 文件打开方式
- mode: 规定该文件的访问权限。  比如 0777， 前面的0是必须的，因为是8进制数

## flags

当有多个时，采用”|“连接

<img src="https://s2.ax1x.com/2020/03/10/8CMPUS.png" style="zoom:50%;" />

最后三个SYNC(同步)选项会降低性能。

| 参数     | 说明                                             |
| -------- | ------------------------------------------------ |
| O_RDONLY | 打开一个只供读取的文件                           |
| O_WRONLY | 打开一个只供写入的文件                           |
| O_RDWR   | 打开一个可供读写的文件                           |
| O_APPEND | 写入的所有数据将被追加到文件的末尾               |
| O_CREAT  | 打开文件，如果文件不存在就建立文件               |
| O_EXCL   | 如果已经置O_CREAT且文件存在，就强制open失败      |
| O_DSYNC  | 每次写入时，等待数据写到磁盘上                   |
| O_TRUNC  | 在打开文件时，将文件的内容清空                   |
| O_RSYNC  | 每次读取时，等待相同部分先写到磁盘上             |
| O_SYNC   | 以同步方式写入文件，强制刷新内核缓冲区到输出文件 |



## mode

只有在创建文件时才会使用此参数。指定文件的访问权限

其实这个综合起来就是777

<img src="https://s2.ax1x.com/2020/03/10/8CMeuq.png" style="zoom:50%;" />

| 参数    | 说明                              |
| ------- | --------------------------------- |
| S_IRUSR | 文件所有者的读权限位              |
| S_IWUSR | 文件所有者的写权限位              |
| S_IXUSR | 文件所有者的执行权限              |
| S_IRWXU | S_IRUSR&#124;S_IWUSR&#124;S_IXUSR |
| S_IRGRP | 文件用户组的读权限位              |
| S_IWGRP | 文件用户组的写权限位              |
| S_IXGRP | 文件用户组的执行权限              |
| S_IRWXG | S_IRGRP&#124;S_IWGRP&#124;S_IXGRP |
| S_IROTH | 文件其他用户的读权限位            |
| S_IWOTH | 文件其他用户的写权限位            |
| S_IXOTH | 文件其他用户的执行权限            |



## 案例

**如果文件不存在就创建并以只读方式打开**

```c++
// 路径也可以为绝对路径  ../xx.txt  上一层 
int fd = open("hell11o.txt",O_CREAT|O_RDWR);
 if(fd>=0){
         cout << fd << endl;
 }else{
       // 打印错误信息
       perror("Error: ");
 }
```

## 打开文件的个数限制

linux默认限制只能最多打开1024个，可以通过`ulimit -a`来查看限制，当然也可以修改限制。

# 创建文件(旧)

早期linux为了兼容性，提供了一个专门创建文件的系统调用，即create函数。

```
int creat(const char *pathname, mode_t mode);
```

相当于open函数的

```
int fd=open(file,O_WRONLY | O_CREAT | O_TRUNC, mode);
```

# 关闭文件

当文件不再使用过的时候，需要把它关系，使用close函数来关闭文件

```c++
#include <unistd.h>
int close(int fd);
```

关闭成功，返回文件描述符，否则返回`-1`。我们开发中，如果一个文件不使用了，那就需要close，否则打开的文件太多，造成文件描述符耗尽的情况，最大打开65534个。

## 案例

```c++
#include <stdio.h>
#include <iostream>
#include <fcntl.h>
#include <unistd.h>

using namespace std;

int main()
{
    int fd = -1;
    char filename[]="hello.txt";
    fd=open(filename, O_RDWR, S_IRWXU);
    // 判断文件是否成功打开
    if(fd == -1)
    {
       cout  << "fail to open" << filename <<";fd:"<< fd<< endl;
    }else{
        cout << "open file" << filename <<  "successfully;fd:"<< fd << endl;
    }
    close(fd);
    return 0;
}
```

# 读取

## read 读取文件数据

函数read读取文件中的数据。该函数声明：

```c++
#include <unistd.h>
ssize_t read(int fd, void * buf, size_t count)
```

`fd`: 进行读取的文件描述符

`buf`: 要写入文件内容或读出文件内容的内存地址

`count`: 要读取的字节数

如果成功读取了数据，就返回所读取的`字节数`，如果读取出现错误会返回`-1`，如果读到了文件末尾，那么返回0。

```c++
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <unistd.h>

int main(int argc, char** argv)
{
    int fd = open("hello.txt",O_RDWR);
    if(fd == -1)
    {
        // perror可以查看错误信息
        // 打印错误：如 ----->open1: No such file or directory
        perror("open1：");
        return -1;
    }
    int read_max_size = 200;
    char buf[100];
	int read_size; // 读取到的字节数
    while((read_size=read(fd, buf, read_max_size)) > 0)
    {
        printf("%s", buf);
    }
    // 如果读取到的为-1，那么证明有错误
    if(read_size == -1)
    {
        // EINTR 被信号中断了
        if(errno == EINTR)
        {
            printf("被信号中断了");
            return -1;
        }
    }
}
```

常见错误

| 错误值 | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| EAGAIN | 使用O_NONBLOCK标志，指定了非阻塞输入输出，但当前没有数据可读 |
| EBADF  | fd不是一个合法的文件描述符，或者不是为了读操作而打开         |
| EINTR  | 在读取到数据前被信号中断                                     |
| EINVAL | fd所指向的对象不适合读，或者是文件打开时指定了O_DIRECT标志   |
| EISDIR | fd指向一个目录。                                             |

# 写数据

可以使用write函数将数据写入已打开的文件内。

```c++
#include <unistd.h>
ssize_t write (int fd, const void * buf, size_t count );
```

此函数会将buf所指向内存中的count个字节写入fd所指向的文件内。

- buf：指向一个缓冲区，表示要写的数据
- count：要写数据的长度，单位是字节。
- fd: 文件描述符

如果函数执行成功，`返回实际写入数据的字节数`。当有错误发生则返回-1，错误代码可以使用errno查看。

常见的错误有：

- EINTR 此调用被信号中断
- EBADF：参数fd是非有效的文件描述符，或该文件已经关闭。
- EINVAL：fd所有指向的对象不适合写，或者文件打开指定了O_DIRECT标志
- ENOSPC： fd指向的文件所在设备无可用空间。
- EPIPE: fd连接到一个管道，或者套接字的读方向一端已经关闭。。

### 案例

```c++
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <unistd.h>

const int read_max_size = 1024;

// 第一个参数为执行的命令，如：./write  x1 x2, 第一个参数 ./write，第二个参数 x1, 第三个参数 x2
int main(int argc, char** argv) {

    char buf[1024];

    if (argc < 3) {
        printf("args is error, args size is %d \n", argc);
        return -1;
    }
    // printf("arg1: %s, arg2: %s \n", argv[1], argv[2]);
    int rfd = open(argv[1], O_RDONLY);
    // 此处权限是必须的
    int wfd = open(argv[2], O_WRONLY | O_CREAT | O_EXCL, S_IRWXU|S_IRWXG);
    if (rfd < 1 || wfd < 1) {
        perror("open error");
        return -1;
    }

    int read_size = 0;
    while ((read_size = read(rfd, buf, read_max_size)) > 0) {
        if (read_size == -1) {
            perror("read error");
            return -1;
        }
        // 写数据
        int res = write(wfd, buf, read_size);
        if(res == -1){
            perror("write error");
            return -1;
        }
        if(res != read_size){
            perror("write size error");
            return -1;
        }
    }
}

```



# 文件偏移量

很多时候我们不只是从头写或者从末尾追加，也需要从文件中的某个文件开始读写。所以有了设定文件偏移量的函数，`文件偏移量指的是当前文件操作位置相对于文件开始位置的偏移。`

open函数如果没有指定`O_APPEND`参数，那么文件的偏移量为0，指定了，那么文件偏移量与文件的长度相等。当然也可以通过`lseek`函数设定文件偏移量。

```c++
#include <unistd.h>
off_t lseek (int fd, off_t offset, int whence );		
```

lseek函数执行成功，返回心得文件偏移量值(相对文件开头的0)，`如果失败，就返回-1`。当然我们也可以传 负值，`所以判断是否操作成功，要使用是否等于-1来判断。`

`off_t`为偏移量，`whence`为操作模式，具体有3中操作模式：

- SEEK_SET: offset为相对文件开始开始处的值.
- SEEK_CUR: offset为相对当前位置的值。
- SEEK_END: offset为相对文件结尾的值。



# 文件访问权限

因为Linux严格的权限，所以我们在打开文件前，一般都需要先判断是否有这个权限。

```c++
int access(const char *pathname, int mode);
```

mode: 多个权限可以使用`|`连接

- R_OK 判断文件是否有读权限
- W_OK 判断文件是否有写权限
- X_OK 判断文件是否有可执行权限
- F_OK 判断文件是否存在

**`对文件的测试成功返回0，只要有一个权限不符合，就返回-1。`**

## 案列

```c++
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <unistd.h>

int main(int argc, char** argv) {
    const char *path = "h1.txt";
    int access_flag = access(path, R_OK|F_OK);
    if(access_flag < 0){
        printf("auth is error");
        return -1;
    }
    return 0;
}
```

# 修改文件属性

当文件被打开之后，进程会获取一个文件描述符，文件描述符包含了文件描述符标志以及当前进程对文件的访问权限等信息标志位。

```c++
int fcntl(int fd, int cmd);
int fcntl(int fd, int cmd, long arg);
```

cmd指定了函数的操作，常见的有

| cmd     | 功能                         |
| ------- | ---------------------------- |
| F_GETFL | 获取文件描述符对应的文件标志 |
| F_SETFL | 设置文件描述符对应的文件标志 |

# 文件夹操作

## 创建文件夹

```c++
#include <sys/stat.h>
#include <sys/types.h>
int mkdir(const char *pathname, mode_t mode);
```

- pathname： 路径
- mode  权限

返回`-1`则代表创建失败。如果目录有文件就会删除失败。需要进行递归删除。

### 案例

```c++
#include <sys/stat.h>
#include <sys/types.h>
#include <stdio.h>
#include <errno.h>

int main(){
	int ret = mkdir("xx/xx");
	// 创建失败
	if(ret == -1){
		// 文件夹已存在
		if( errno == EEXIST){
		
		}
	}
}
```

## 删除文件夹

```c++
#include <unistd.h>
int rmdir(const char *pathname);
```

pathname: 删除路径

返回`-1`则代表删除失败。

## 打开文件夹

```c++
#include <sys/types.h>
#include <dirent.h>
DIR *opendir(const char *name);
DIR *fdopendir(int fd);
```

打开文件夹，如果打开失败则会返回NULL

## readdir读文件夹  

单独使用`opendir`是无意义的，需要使用`readdir  `配合使用

```c++
#include <dirent.h>
struct dirent *readdir(DIR *dirp);
struct dirent {
ino_t d_ino; /* 此目录进入点的inode */
off_t d_off; /* 目录文件开头至此目录进入点的位移 */
unsigned short d_reclen; /* length of this record */
unsigned char d_type; /* 所指的文件类型*/
char d_name[256]; /* 文件名 */
};
```

### 案例

```c++
DIR* dir = opendir("/home/zsyubo/Documents/c++/io/sss");
// 先判断是否为空
    if(dir == NULL){
        printf("flodir open fire");
        perror("error");
        return 0;
    }
   struct  dirent *ptr;
    int i;
    while((ptr = readdir(dir)) != NULL){
        printf("d_name: %s\n", ptr->d_name);
    }
	// 关闭
    closedir(dir);
```

## rewinddir 重置读取目录的位置  

```c++
#include <sys/types.h>
#include <dirent.h>
void rewinddir(DIR *dirp);
```

就是吧`readdir`的读取位置重置

## telldir   记录dir流的当前读取位置

```c++
#include <dirent.h>
long telldir(DIR *dirp);
```



### seekdir  设置dir流当前的读取位置

```c++
void seekdir(DIR *dirp, long offset);
```

## closedir 关闭dir流

```c++
#include <sys/types.h>
#include <dirent.h>
int closedir(DIR *dirp);
```

## 案例

遍历目录，并打印目录下的文件夹名

```c++
int main(int argc, char** argv) {
    char* path ="/home/zsyubo/Documents/c++/io/";
    DIR* dir = opendir(path);
    if(dir == NULL){
        perror("error");
        return 0;
    }

   struct  dirent *ptr;
    while((ptr = readdir(dir)) != NULL){
        // 拼接完整目录
        char name[100];
        sprintf(name, "%s/%s", path, ptr->d_name);

        // 如果是 . 或者.. 则跳过
        if (strcmp(ptr->d_name, ".") == 0 || strcmp(ptr->d_name, "..") == 0)
            continue;
		// 使用stat配合判断是否是文件夹
        struct stat stbuf;
        if (stat(name, &stbuf) == -1) {
            perror("stat");
        }
        if (S_ISDIR(stbuf.st_mode)){
            printf("isdir: %s \n", ptr->d_name);
        }
    }
    closedir(dir);
}
```

# 获取文件状态

很多时候我们要经常用到文件的一些状态信息，比如修改时间、大小等。

```c++
#include <sys/stat.h>
int stat(const char *path,struct stat *buf);
int fstat(int filedes, struct stat *buf);
int lstat(const char *path, struct stat *buf);
```

参数path是文件路径，filedes是文件描述符，buf为指向struct stat结构的指针，获得状态从这个参数中返回，函数执行成功返回0，失败返回-1。

stat是一个结构体。stat和lstat的区别是，当文件是一个符号链接时，lstat返回的是改符号链接本身的信息。

![](https://s1.ax1x.com/2020/03/15/83TSHK.png)

# 文件锁定

当多个进程同时操作一个文件时，会出现并发异常。linux防止这种情况，提供了文件锁，分为 建议锁和强制索。

**建议锁**是给文件上锁后，只是在文件上放置一个锁的标识，其他进程依然可以操作此文件，并不能阻止其他进程对文件进行操作。

**强制性锁**:给文件上锁后，内核将`阻塞`后来的进程，知道第一个进程将锁解开。

Linux建议使用强制锁，毕竟很多程序比一定按照建议锁的规矩来，为了安全，列如执行open、read、write等操作是，内核会检测是否加了强制索，如果加了强制锁，就会导致文件操作失败。

**fork**产生的子进程不继承进程设置锁的，对于从父进程处继承过来的任一描述符，子进程需要调用fcntl才能获得它自己的锁。

**防止重复启动：**文件锁定也可以用来防止程序重复启动，可以在进程启动时，对/var/run下的.PID文件进行锁定，这样后面的进程就会因为无法对该文件上锁而退出。

```c++
#include <unistd.h>
#include <fcntl.h>
// fd 文件描述符，cmd是操作命令，lock是指向结构体flock的指针。
int fcntl(int fd, int cmd, struct flock *lock);
```

cmd取值如下：

- F_GETLK: 根据lock描述，决定是否上文件锁
- F_SETLK: 设置lock描述的锁。



虽然建议使用强制锁，但是强制索对性能影响很大，所以fuctl默认是建议锁，如果想在linux中使用强制锁，则要在root权限下，通过mount命令用-o mand选项打开该机制。结构体flock定义如下：

![](https://s1.ax1x.com/2020/03/15/88h9mT.png)

l_type:

- F_RDLCK:  共享锁(读取锁)，只读用，多个进程可以同时建立读取锁。
- F_WRLCK: 独占锁(写入锁)，在任何时候只能有一个进程建立写入锁。
- F_UNLCK: 解除锁定。

l_whence取值（unistd.h中定义）：

- SEEK_SET: 文件开始位置。
- SEEK_CUR: 文件当前位置。
- SEEK_END: 文件末尾位置。

l_start 为相对开始偏移量：

- l_len： 加锁的长度，0为到文件末尾
- l_pid: 当前操作文件的进程id号。

`如果函数执行成功，返回0，否则返回-1，可以使用errno查询错误码。`



# 文件和内存映射

就是讲普通文件映射到内存中，普通文件被映射到内存地址空间中，进程可以像访问普通内存一样对文件进行访问。

```c++
void *mmap(void *start, size_t length, int prot, int flags, int fd, off_t offset)
```

start为映射区的起始地址，通常为NULL或0，表示系统自己决定映射到什么地址。

length表示映射数据的长度，即文件需要映射到内存中的数据的大。

offset:  表示映射数据在文件中的起点。

prot: 表示映射区保护方式，取值如下：

- PROT_EXEC: 映射区可被执行。
- PROT_READ: 映射区可读取，
- PROT_WRITE: 映射区可写入。
- PROT_NONE: 映射区不可访问。

flags: 用来制定个映射对象的类型、映射选项和映射页是否可以共享。`它的值可以是一个或者多个位的组合`

- MAP_FIXED: 如果参数start指定需要映射到的地址，而所指定的地址无法成功建立映射，映射会失败。
- MAP_SHARED:  共享映射区域，映射区域允许其他进程共享，对映射区域写入数据将会写入到原来的文件中。
- MAP_RIVATE: 对映射区域进行写入操作会产生一个映射文件的复制，即写入复制，而读操作不会影响此复制。对映射区的修改不会写回原来的文件，即不会影响原来文件的内容。
- MAP_ANONYMOUS: 建立匿名映射。映射区不与任何文件关联，而映射区无法与其他进程共享。
- MA_DENYWRITE: 对文件的写入操作将被禁止，不允许直接对文件进行操作。
- MAP_LOCKED: 将映射区锁定，防止页面被交换出内存。

`flags必须是`MAP_SHARED或者MAP_RIVATE二者之一。`MAP_RIVATE是多个进进程进行写入操作时，会复制一个副本给修改的进程，多个进程之间的副本是不一致的。`

mmap()映射后，让用户程序直接访问设备内存，相比较在用户空间和内核空间互相复制数据，效率更高。mmap映射内存必须是页面大小的整数倍。面向流的设备不能进行mmap。