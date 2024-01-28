# Lab: Xv6 and Unix utilities

## 1.Boot xv6

* 首先使用命令`git checkout util`确保切换到`util`分支进行实验。

* 使用命令`make qemu`可以创建并且运行xv6，运行结果如下：

  ```
  tao@ubuntu:~/Desktop/qemu-5.1.0/xv6-labs-2020$ make qemu
  qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0
  
  xv6 kernel is booting
  
  hart 2 starting
  hart 1 starting
  init: starting sh
  ```

* 输入命令`ls`的结果如下：

  ```
  $ ls
  .              1 1 1024
  ..             1 1 1024
  README         2 2 2059
  xargstest.sh   2 3 93
  cat            2 4 24000
  echo           2 5 22832
  forktest       2 6 13200
  grep           2 7 27352
  init           2 8 23928
  kill           2 9 22800
  ln             2 10 22752
  ls             2 11 26240
  mkdir          2 12 22904
  rm             2 13 22888
  sh             2 14 41784
  stressfs       2 15 23904
  usertests      2 16 147544
  grind          2 17 38016
  wc             2 18 25144
  zombie         2 19 22304
  console        3 20 0
  ```

* 要退出qemu，输入：`Ctrl+a+x`即可。

## 2.Sleep

### 1.实验目的

* 使用***kernel/sysproc.c***中的`sleep`进行用户层面的`sleep`功能。
* 如果`sleep`后面没有参数，打印错误信息。
* 在xv6中执行`sleep`指令，阻塞输入的参数所代表的时间。

### 2.实验代码

```c
#include "kernel/types.h"
#include "user/user.h"

int main(int argc,char const *argv[])
{
	//如果输入的命令大于2则报错
	if (argc != 2){
		printf("usage: sleep+time\n");
		exit(1);
	}
	sleep(atoi(argv[1]));
	exit(0);
}
```

### 3.注意事项

* 该实验主要是通过阅读代码理解`sleep`的系统调用，然后在用户层面进行执行。

* 理解`main`函数中的参数代表的意思，以及参数是如何传递的。

> `int main(int args,const char *argv[]);`
>
> * `args`代表的是命令的数量，例如输入`make qemu`时，`args=2`。
> * `argv`是一个数组，存储了所输入的命令，大小即为`args`。

## 3.pingpong

### 1.实验目的

* 该实验主要是利用管道pipe进行父子进程之间的通信。
* 父进程应该向子进程发送一个字节;子进程应该打印“`<pid>: received ping`”，其中`<pid>`是进程ID，并在管道中写入字节发送给父进程，然后退出;父级应该从读取从子进程而来的字节，打印“`<pid>: received pong`”，然后退出。

### 2.实验代码

```c
#include "kernel/types.h"
#include "user/user.h"

#define R 0 //pipe的写端
#define W 1 //pipe的读端

int main(int argc,char const *argv[])
{
    char buf = 'A'; //传送的字节
    int p2c[2]; //父到子的管道
    int c2p[2]; //子到父的管道
    pipe(p2c);
    pipe(c2p);

    int pid = fork(); //创建子进程
    int exit_status = 0;
    //如果创建失败,打印错误信息并关闭管道
    if (pid < 0){
        printf("fork() ERROR!\n");
        close(p2c[R]);
        close(p2c[W]);
        close(c2p[R]);
        close(c2p[W]);
        exit(1);
    }
    //如果是子进程
    else if (pid == 0){
        close(p2c[W]);
        close(c2p[R]);  

        //子进程读取父进程      
        if (read(p2c[R],&buf,sizeof(char)) != sizeof(char)){
            printf("Child read() ERROR!\n");
            exit_status = 1;
        }
        else {
            printf("<%d>: received ping\n",getpid());
        }

        //子进程向父进程写
        if (write(c2p[W],&buf,sizeof(char)) != sizeof(char)){
            printf("Child write() ERROR!\n");
            exit_status = 1;
        }

        close(p2c[R]);
        close(c2p[W]);
        exit(exit_status);
    }
    //如果是父进程
    else {
        close(c2p[W]);
        close(p2c[R]);

        if (write(p2c[W],&buf,sizeof(char)) != sizeof(char)){
            printf("Parent write() ERROR!\n");
            exit_status = 1; //标记出错
        }

        if (read(c2p[R],&buf,sizeof(char)) != sizeof(char)){
            printf("Parent read() ERROR!\n");
            exit_status = 1; //标记出错
        }        
        else {
            printf("<%d>: received ping\n",getpid());
        }
        close(c2p[R]);
        close(p2c[W]);
    
        exit(exit_status);
    }
}
```

  ### 3.注意事项

* 该实验主要考虑管道之间的及时关闭，不然会出错。

## 4.Primes

### 1.实验目的

* 使用`pipe`和`fork`来设置管道。第一个进程将数字2到35输入管道。对于每个素数，您将安排创建一个进程，该进程通过一个管道从其左邻居读取数据，并通过另一个管道向其右邻居写入数据。由于xv6的文件描述符和进程数量有限，因此第一个进程可以在35处停止。
* 素数的筛选过程使用如下方法：

> ```c
> p = get a number from left neighbor
> print p
> loop:
>     n = get a number from left neighbor
>     if (p does not divide n)
>         send n to right neighbor
> p = 从左邻居中获取一个数
> print p
> loop:
>     n = 从左邻居中获取一个数
>     if (n不能被p整除)
>         将n发送给右邻居
> ```
>
> ![p1](https://github.com/Fengtao4428/MIT-6.S081/assets/88192248/2b368f70-f7b4-4ba9-9d90-8f702e53ceab)
>
> * 生成进程可以将数字2、3、4、…、1000输入管道的左端：行中的第一个进程消除2的倍数，第二个进程消除3的倍数，第三个进程消除5的倍数，依此类推。

### 2.实验代码

```c
#include "kernel/types.h"
#include "user/user.h"

#define R 0 //pipe的写端
#define W 1 //pipe的读端

const uint INT_LEN = sizeof(int);
//读取左邻居的第一个数据，并打印
int lpipe_first_data(int lpipe[2],int *first)
{
    if (read(lpipe[R],first,INT_LEN) == INT_LEN){
        printf("prime %d\n",*first);
        return 0;
    }
    return -1;
}
//将不能被first整除的数据传输给右邻居
void dataTransmit(int lpipe[2],int rpipe[2],int first)
{
    int data;
    while (read(lpipe[R],&data,INT_LEN) == INT_LEN){
        //将不能被first整除的数据传给rpipe
        if (data % first !=0){
            write(rpipe[W],&data,INT_LEN);
        }
    }
    close(lpipe[R]);
    close(rpipe[W]);
}
//找出管道中的素数
void primes(int lpipe[2]){
    close(lpipe[W]);
    int first;
    if (lpipe_first_data(lpipe,&first) == 0){
        int p[2];
        pipe(p);
        dataTransmit(lpipe,p,first);

        if (fork() == 0){
        prime(p);
        }
        else {
            close(p[R]);
            wait(0);
        }
    }
    exit(0);
}
//主函数
int main(int argc,const char *argv)
{
    int p[2];
    pipe(p);

    for (int i = 2;i <= 35;i++){
        write(p[W],&i,INT_LEN);
    }
    if (fork() == 0){
        prime(p);
    }
    else {
        close(p[R]);
        close(p[W]);
        wait(0);
    }
    exit(0);
}
```

### 3.注意事项

* 对素数筛选方法的理解，主要是看懂所给的原理图。
* 该程序实际上就是递归。

## 5.find

### 1.实验目的

* 写一个简化版本的UNIX的`find`程序：查找目录树中具有特定名称的所有文件，你的解决方案应该放在`user/find.c`。
* 不要在“`.`”和“`..`”目录中递归。
* 查看***user/ls.c\***文件学习如何读取目录。

### 2.实验代码

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

void find(char *path,const char *filename)
{
    char buf[512],*p;
    int fd;
    struct dirent de;
    struct stat st;

    if ((fd = open(path,0)) < 0){
        fprintf(2,"find: cannot open %s\n",path);
        return;
    }

    if (fstat(fd,&st) < 0){
        fprintf(2,"find: cannot fstat %s\n",path);
        close(fd);
        return;
    }

    //find的第一个参数必须时目录
    if (st.type != T_DIR){
        fprintf(2,"usage: find <DIRECTORY> <filename>\n");
        return;
    }

    if (strlen(path) + 1 + DIRSIZ + 1 > sizeof buf){
        fprintf(2,"find: path too long\n");
        return;
    }
    strcpy(buf,path);
    p = buf + strlen(buf);
    *p++ = '/';
    while (read(fd,&de,sizeof(de)) == sizeof(de)){
        if (de.inum == 0){
            continue;
        }
        memmove(p,de.name,DIRSIZ);
        p[DIRSIZ] = 0;
        if (stat(buf,&st) < 0){
            fprintf(2,"find: cannot stat %s\n",buf);
            continue;
        }
        //不在"."和".."中递归
        if (st.type == T_DIR && strcmp(p,".") != 0 && strcmp(p,"..") != 0){
            find(buf,filename);
        }
        else if (strcmp(filename,p) == 0){
            printf("%s\n",buf);
        }
    }
    close(fd);
}
int main(int argc, char *argv[])
{
  if (argc != 3) {
    fprintf(2, "usage: find <directory> <filename>\n");
    exit(1);
  }
  find(argv[1], argv[2]);
  exit(0);
}
```

### 3.注意事项

* 理解文件的访问。
## 6.xargs

### 1.实验目的

* 编写一个简化版UNIX的`xargs`程序：它从标准输入中按行读取，并且为每一行执行一个命令，将行作为参数提供给命令。你的解决方案应该在**user/xargs.c**

* 例如：

  ```c
  $ echo hello too | xargs echo bye
  bye hello too
  $
  ```

* 其中左边命令执行的结果通过管道输入到右边，xargs中的命令以左边的每行结果为参数进行执行。

### 2.实验代码：

```c
#include "kernel/types.h"
#include "user/user.h"
#include "kernel/param.h"

#define MAX_BUF 512

int main(int argc,char *argv[])
{
    char buf[MAX_BUF] = {0};
    char *xargv[MAXARG] = {0};
    int occupy = 0;
    int end = 0;

    for(int i = 1;i < argc;i++){
        xargv[i-1] = argv[i];
    }

    //从左侧命令读入数据
    while (!(end && occupy == 0)){
        if (!end){
            int remain = MAX_BUF - occupy;
            int readBytes = read(0,buf + occupy,remain);
            if (readBytes < 0){
                fprintf(2,"READ ERROR!\n");
            }
            if (readBytes == 0){
                close(0);
                end = 1;
            }
            occupy += readBytes;
        }
        char *pos = strchr(buf,'\n');
        //逐行分解执行命令
        while (pos){
            char xbuf[MAX_BUF+1] = {0};
            memcpy(xbuf,buf,pos-buf);
            xargv[argc-1] = xbuf;
            //创建子程序进行执行
            int ret = fork();
            if (ret == 0){
                if (!end){
                    close(0);
                }
                if (exec(argv[1],xargv) < 0){
                    fprintf(2,"EXEC ERROR!\n");
                    exit(1);
                }
            }
            //处理已执行的数据
            else {
                memmove(buf,pos+1,occupy-(pos-buf)-1);
                occupy -= pos - buf + 1;
                memset(buf + occupy,0,MAX_BUF - occupy);
                int pid;
                wait(&pid);
                
                pos = strchr(buf,'\n');
            }
        }
    }
    exit(0);
}
```

### 3.注意事项

* 正确理解标准输入流的读取方式。
* 对传入的参数进行理解`argc`和`argv`。
