源代码模式 ctrl+/
# 一级标题
### 三级标题
高亮： == 高亮 == ；  ==//==
上标：^上标^
下标：~下标~

表格：
| MON | WEN | TES | THU |
| --------|-------|------|------|
|1|2|3|4|



# gdb

g-GNU 

d-debug

排除工程逻辑错误---单步调试

```
//gdb调试示例代码 test.c

int main(void)
{
	for(int i=0;i<5;i++)
	{
		a++;
	}
	print("a = %d",a);
	return 0;
}
```

gdb调试：

编译: `gcc -g test.c`

调试:`gdb ./a.out`

打断点：`b` + 函数/行号

列出代码内容:`l`

执行：`r`   下一步: `n`

打印：`p` +变量

快速执行到下一断点`c`

退出gdb `q`



`gcc -c `只编译不链接；查看语法错误~



### Xshell连接不上Ubuntu.

图示禁用再启用即可。

![image-20230822151955915](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230822151955915.png)













# UC
Unix操作系统使用C语言编程



复制粘贴: ctrl+shift+c/v

## 01 

### 计算机系统分层

==操作系统 OS==：是管理计算机软/硬资源的软件。

![image-20230804195252263](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230804195252263.png)

Windows----shell

Linux----Bash

![image-20230804195831904](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230804195831904.png)











### 环境变量

进程：正在执行的程序；

每个进程都有一个环境变量表，表中条目是”键=值“形式的环境变量。

查看环境变量：env；		echo $Name：输出察看环境变量值

添加环境变量：FOOD=liurouduan。可以直接覆盖；

| 全局环境变量              | 局部环境变量              |
| ------------------------- | ------------------------- |
| env可以查看。             | FOOD=liurouduan,env见不着 |
| 当前shell和其子进程均可见 | 当前shell可见。           |

局部--->全局 :export name

删除全局变量：unset name



==PATH环境变量==：

命令ls.cd...都是二进制可执行程序；在每个地方都能执行~取决于PATH

echo $PATH:显示的是==命令检索==的路径。

whereis ls：查看ls命令位置



==Q:执行a.out为什么要加./？==

​			 -----A: ./表示当前目录； ./a.out表示告诉bash可执行文件在当前目录下，不用按照PATH去检索了。

​					------可以不加./的方法：把当前目录加到PATH————PATH=$PATH:.	在原本路径下==$PATH==多检索一个当前路径==:.==

​					------注意：关闭窗口后再重新查看PATH。会发现:.不见了。(==新的Bash进程不同，每个进程有自己的环境变量==)------解决：家目录下有名为./bashrc的脚本文件，每次bash进程启动前都会执行该脚本文件。若==希望每次配置都有效==，可以将环境变量设置写在脚本文件中。-----配置: cd ~进入家目录---vi .bashrc---添加PATH=$PATH:.

![image-20230804202922622](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230804202922622.png)



每个进程对应一个环境变量表---字符指针数组---==指针数组地址保存在全局变量environ中(二级指针)==

`gcc environ.c -o environ`

`./environ								  //新进程？`

```
extern char** environ 					//导入当前进程的环境变量表
for(char** pp = environ;*pp;pp+1)		//环境变量表最后一个指针是NULL
{
	...
}
```

![image-20230804203923302](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230804203923302.png)

或者main函数的第三个参数,两个printf显示地址，地址值一样。

![image-20230804210359917](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230804210359917.png)



![image-20230804213800563](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230804213800563.png)

extern在多文件编译中用别的文件定义的全局变量







## 02 库文件

==相关功能实现(二级制文件)整合在一起便于使用的==。

###  静态库 libxxx.a

![image-20230804212158032](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230804212158032.png)

以数学库为例生成静态库： 

![image-20230804215635365](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230804215635365.png)

```
#libmath.a
#计算功能：加法、减法 add,sub --- calc.h,calc.c --> calc.o
#显示功能 show --- show.c show.h --> show.o
```

```
#calc.h

#ifndef __CALC_H__
#define __CALC_H__
//加法计算 add
int add(int x,int y);
//减法计算 sub
int sub(int x,int y);
#endif

#calc.c

#include"calc.h"
//加法函数
int add(int a,int b)
{
	return a+b;
}
//减法函数
int sub(int a,int b)
{
	return a-b;
}
```

```
#show.h

#ifndef __SHOW_H__
#define __SHOW_H__
#include<stdio.h>
//显示 1+2 = 3;
void show(int l,char op,int r,int res);
#endif

#show.c

#include"show.h"
void show(int l,char op,int r,int res)
{
	printf("%d %c %d = %d\n",l,op,r,res);
}
```

生成.o文件(二进制文件)----- `gcc -c`-------生成`calc.o show.o`-------- `ar -r libmath.a *.o ` 			//*通配符号，表示任意.o后缀文件

```
#main.c

#include<stdio.h>
#include"calc.h"
#include"show.h"
#include

int main(void)
{
	int a = 123,b = 456;
	int res = add(a,b);
	show(a,'+',b,res);
	show(a,'-',b,sub(a,b));
	return 0;
}
```

==编译时拿库一起编译== `gcc main.c libmath.a`		// 查看库中函数   `nm libmath.a`

为了避免大量的头文件声明，可以定义==接口文件== `math.h`

```
#math.h 

#ifndef __MATH_H__
#define __MATH_H__

#include"calc.h"
#include"show.h"

#endif
```



==Q:如果库和调用模块不在一块：==

`mv libmath.a ..`			//把库放上层目录

![image-20230804221448488](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230804221448488.png)

==A:gcc编译器需要知道库路径。==

`gcc main.c -lmath -L..`				//库名不带前后缀。这里库在上层目录。

#利用环境变量链接库。

```
echo $LIBRARY_PATH						//查看库环境变量
LIBRARY_PATH=$LIBRARY_PATH:..			//在原有环境变量下，追加库位置。 ..上层目录。
export LIBRARY_PATH						//直接写环境变量是局部的，无法被gcc找到。需要改为全局环境变量。 
gcc main.c -lmath
```

![image-20230804221547786](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230804221547786.png)





### 动态库 libxxx.so

动态库不会复制功能实现；链接动态库时到库里去寻找对应功能。动态库可以被多个进程(a/b/c.out)共同使用。



![image-20230805101045691](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805101045691.png)

创建动态库

```
mkdir shared
cd shared/
cp ../static/*.h .
cp ../static/*.c .						//所有.c .h文件复制到当前目录。

gcc -c -fpic calc.c
gcc -c -fpic show.c 					//编译为目标文件  -fpic位置无关码(position independent code) 不同进程找到动态库都以相对地址来找；

gcc -shared calc.o show.o -o libmath.so //打包为动态库

echo $LD_LIBRARY_PATH
LD_LIBRARY_PATH = $LD_LIBRARY_PATH:.
export LD_LIBRARY_PATH

gcc main.c libmath.so
```

Q:==执行`gcc main.c libmath.so`时会找不到路径==,链接器负责找库。

A:需要配置环境变量LD_LIBRARY_PATH。



### ==一些环境变量总结==

| 环境变量 |作用|
|---------|------------------|
| PATH | bash找命令 |
|LIBRARY_PATH |GCC编译阶段找库|
|LD_LIBRARY_PATH|链接器在a.out执行阶段找库 |



静态库：大、快、不依赖于库存在；

动态库：小、慢、依赖库存在。----------有利于产品更新迭代





### 动态库动态加载

![image-20230805104220593](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805104220593.png)

==用的时候加载，不用的时候就把库卸载==，提高内存效率----------动态库动态加载---------对应功能库 `dlfcn`

```
#include<dlfcn.h>
-ldl
```

==dlopen,载入共享库==

![image-20230805104713227](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805104713227.png)

==dlsym,找到具体函数功能==![image-20230805105343461](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805105343461.png)

==dlclose,从内存卸载库==     只有没有进程用这个库时，才会真正卸载。但一旦用了，handle就不代表这个动态库了。

![image-20230805110352549](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805110352549.png)

==dlerror,获取在加载、使用动态库时发生的错误==

![image-20230805110644772](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805110644772.png)

```
#load.c 动态加载

#include<stdio.h>
#include<dlfcn.h>

int main(void)
{
	//动态库载入内存
	void* handle = dlopen("./shared/libmath.so",RTLD_NOW);
	
	//dlopen失败
	if(handle == NULL)
	{
		fprintf(stderr,"dlopen:%s\n",dlerror());				
		return -1;
	}
	//获取动态库中函数地址
	int (*add_m)(int,int) = dlsym(handle,"add");			//强制类型转换，add_m为函数指针类型， 参数(int,int),返回int
    
    if(add_m == NULL)
    {
   		fprintf(stderr,"dlsym:%s\n",dlerror());				
		return -1; 	
    }
    
    int (*sub_m)(int,int) = dlsym(handle,"sub");		
    
    if(sub_m == NULL)
    {
   		fprintf(stderr,"dlsym:%s\n",dlerror());				
		return -1; 	
    }
    
	void (*show_m)(int,char,int,int) = dlsym(hanle,"show");
    
    if(show_m == NULL)
    {
    	fprintf(stderr,"dlsym:%s\n",dlerror());				
		return -1; 
    }
	
	//使用函数
	int a=123,b=456;
	show_m(a,'+',b,add_m(a,b));
	show_m(a,'-',b,sub_m(a,b));
	//卸载动态库
	
	if(dlcose(handle))
	{
		fprintf(stderr,"dlcose:%s\n",dlerror());
		return -1;
	}
	return 0;
}


$gcc load.c -ldl -o load					//链接动态库
$./load
```

ps.

对指针： 由里到外，从右到左，括号优先   

 const int * p----p是指针，指向const int类型       int (*p)(int,int)---- p是指针，带参数是函数指针；返回值int。

int* p[]----p是数组，每个元素是int*类型，指针数组。

int (*p)[] ---- p是指针，指针指向整形数组  

```
printf()		输出先到输出缓冲区，再到显示器。要满足\n或缓冲区满才在显示器中显示。
fprintf()		向文件中输出
stdin	默认键盘
stdout  默认显示器，标准输出，经过缓冲区。
stderr  默认显示器，标准错误，不经过缓冲区。第一时间显示错误。
```

 

### 错误处理

出错了错误码会存在全局变量errno中。

![image-20230805140822214](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805140822214.png)

![image-20230805141024714](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805141024714.png)

![image-20230805141152750](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805141152750.png)





## 03 内存

![image-20230805142413744](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805142413744.png)

a.out(磁盘中)按某方式存在内存条(电介质)中。然后CPU从内存条读取数据，执行(更快)

内存不够时，会将部分不用数据暂时放在磁盘中(==页面换出==)。需要用再从磁盘读取(==页面换入==)。

Linux下，把磁盘中暂存内存条数据的地方叫做==交换分区==。(Win下叫==虚拟内存==)



我们打印一个变量的地址时得到的不是其物理地址，而是与其物理地址对应的虚拟地址。

![image-20230805143438634](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805143438634.png)

以32位系统为例---==物理内存大小与虚拟地址范围无关==，虚拟地址范围与系统位数有关；

要用的虚拟地址才对应物理地址；其他虚拟地址实际是不存在的，不对应物理地址。

![image-20230805145700295](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805145700295.png)==0x00000000-0xffffffff----4G,其中低3G空间一般是用户数据；高1G空间为内核数据==。

![image-20230805150322188](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805150322188.png)

![image-20230805151120582](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805151120582.png)

```
#虚拟地址空间布局 map.c
#include<stdio.h>
#include<stdlib.h>			//malloc

const int const_global = 1;
int init_global = 2;
int uninit_global

int main(int argc,char* argv[], char* envp[])
{
	const static int const_static = 3;
	static int init_static = 2;
	static int unint_static;
	const int const_local = 5;				//常局部变量
	int local;								//局部变量。
	int* heap = malloc(sizeof(int));		//堆变量
	*heap = 6;
	char* string = "abc";					//字面值常量
	
	printf("--------参数和环境变量----------");
	printf("		命令行参数:%p\n	 ",argv);
	printf("		环境变量：%p\n	 ",envp);
	printf("--------栈区------------------\n");
	printf("		常局部变量：%p\n",&const_local);
	printf("		局部变量：%p\n",&local);
	printf("--------堆区------------------\n");
	printf("		堆变量：%p\n",heap);
	printf("--------BSS区------------------\n");
	printf("		未初始化全局变量：%p\n",&uninit_global);
	printf("		未初始化静态局部变量：%p\n",&uninit_static);
	printf("--------数据区------------------\n");	
    printf("		初始化全局变量：%p\n",&init_global);
	printf("		初始化静态局部变量：%p\n",&init_static);
	printf("--------只读常量区------------------\n");	
	printf("		函数：%p\n",main);
	printf("		常全局变量：%p\n",&const_global);
	printf("		常静态变量：%p\n",&const_static);	
	printf("		字面值量：%p\n",string);	
	printf("----------------------------------\n");		
	return 0;
}
```



### 内存壁垒

![image-20230805154811831](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805154811831.png)

不同进程相同的虚拟地址对应的不是一个物理地址。



![image-20230805155449629](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805155449629.png)

内核数据在内存中只有一份，被多进程轮转使用(单核）。





### 段错误

对虚拟内存的越权访问。

1.向没有对应物理地址的虚拟地址写入数据。

2.对只读内存做写操作(代码区)

```
int main(void)
{
	int *p = (int*)0x12345678;
	*p = 1;						//error1.
	
	const int i = 1;			//栈区
	*(int*)&i = 2;				//const int*-->int *再赋值，可行；
	
	const static int j = 1;		//代码区，
	*(int*)&j = 2;				//error2.
	
	return 0；
}
```



## 04 内存管理

`mmap()`   malloc()的底层。

1.在物理内存上分配存储区；

2.建立物理与虚拟地址的映射。

页 = 4096字节；

![image-20230805161715841](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805161715841.png)

![image-20230805162205472](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805162205472.png)

`munmap` 解映射

![image-20230805162455235](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805162455235.png)

```
#mmap.c

#include<stdio.h>
#include<string.h>
#include<sys/mman.h>

int main(void)
{
	//内存映射
	char* start = (char*) mmap(NULL,8192,PROT_READ | PROT_WRITE,MAP_PRIVATE | MAP_ANONYMOUS,0,0);
	
	if(start == MAP_FAILED)
	{
		perror("mmap");
		return -1;
	}
	//使用
	strcpy(start,"小鸡炖蘑菇");			//写入数据
	printf("%s\n",start);
	//解除映射，也可以部分解除映射，但是需要按页处理。
	
	if(munmap(start,8192) == -1)
	{
		perror("munmap");
		return -1;
	}
	return 0;
}
```



`sbrk()`  底层是mmap

![image-20230806162753207](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806162753207.png)

维护堆尾指针。控制范围虚拟内存边界。超出已映射页的虚拟空间，则会立刻补映射。

![image-20230805165232162](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805165232162.png)

![image-20230805165524318](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230805165524318.png)

返回堆尾指针移动之前的地址。

```
#sbrk()

#include<stdio.h>
#include<unistd.h>

int main(void)
{
	printf("当前堆尾：%p\n",sbrk(0));
	int* p1 = sbrk(4);
	printf("p1 = %p\n",p1);
	*p1 = 123;
	double*p2 = sbrk(8);
	printf("p2 = %p\n",p2);
	*p2 = 4.56;
	printf("%d,%lg\n",*p1,*p2);
	//释放
	sbrk(-(4+8));
	printf("当前堆尾：%p\n",sbrk(0));
	return 0;
}
```



==brk==

![image-20230806163857518](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806163857518.png)



![image-20230806164029256](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806164029256.png)









## 05 文件

### 文件系统

机械硬盘：磁介质存储介质；

磁道~扇区  ~柱面 ~分区

固态硬盘：类似U盘



文件在磁盘上以  i节点 + 数据块 形式表示。

i节点中存文件属性以及数据块索引；数据块中存的是文件数据。————删除文件实际上是删除了数据块索引；数据块不再被该文件占用。

![image-20230806165659952](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806165659952.png)

ls -i可以查看文件i节点号；i节点映射表是为了方便找到i节点物理地址。

![image-20230806170519950](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806170519950.png)





### 文件类型

==普通文件==：不同进程可以同时读写同一文件。文件包含元数据(i节点中)和内容数据(数据块)中。

==目录文件==：数据块存储文件名与i节点的映射。每一个这样的映射叫==硬链接==。==目录文件的i节点在上一级目录找...==最终追溯到根目录，根目录的i节点是固定已知的。

P.S. 

==.和..==也作为映射，分别存本级目录i节点和上级目录i节点。

软链接：

硬链接：名与节点号的对应关系。1个文件只对应1个 i 节点；但1个 i 节点可以对应多个文件名

软链接：文件。创建符号链接：`ln -s abc.txt xyz.txt`  xyz中存的是abc的完整访问路径；类似xyz是abc的快捷方式。



==特殊文件==

![image-20230806172322072](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806172322072.png)





### 文件打开与关闭



==open()==，返回int 类型文件描述符fd。

![image-20230806172616616](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806172616616.png)

![image-20230806173150380](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806173150380.png)

```
#include<stdio.h>
#include<unistd.h> 			//close()
#include<fcntl.h>			//open()

int main(void)
{
	//打开
	int fd = open("./open.txt",O_RDWR | O_CREAT | O_TRUNC,0777);			//0xxx八进制,注意后面要考虑权限掩码umask ----。
	if(fd == -1)
	{
		perror("open");
        return -1;
	}
	printf("fd = %d\n",fd);
	
	//关闭
	if(close(fd) == -1)
	{
		perror("close");
		return -1;
	}
	return 0;
}
```

![image-20230806173926973](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806173926973.png)



### 文件的内核结构

将磁盘中i节点数据拷贝到内存v节点上。再用到文件相关数据时，提高效率。还有一个包含文件相关属性的文件表项。

文件关闭时删除这两个结构体。

![image-20230806194511276](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806194511276.png)



文件状态：可读 可写 ....

所有文件表项关闭了，才会释放v节点。

![image-20230806194828026](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806194828026.png)



### 文件描述符

进程表：内核维护的进程信息；

文件描述符表：元素内容为指针，指针指向文件表项；元素索引就是文件描述符。

![image-20230806195326815](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806195326815.png)

0：标准输入文件符(键盘)

1：标准输出 printf() ---> 向文件描述符号1输出 ---> 默认显示器

2：标准错误

重定向后，可以让1对应其他文件；那么再写入数据，对应的文件也就不是显示器了。```

`./a.out > hello.txt	`   shell中的输出重定向；



==文件描述符复制 dup()==

复制文献描述符对应的元素内容(文件表项)。复制到最小未使用项 `dup(3)`

![image-20230806215131726](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806215131726.png)

oldfd与newfd对应同一个文件表项。

要关闭文件表项，必须关闭所有对应的文件描述符。

`dup2`将文件描述符内容复制到指定文件描述符；目标如果已经被占用，则先删除；



### 文件的读写

==write:==

![image-20230806204304653](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806204304653.png)

==read:==

![image-20230806204442755](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806204442755.png)

```
#write

#include<stdio.h>
#include<fcntl.h>
#include<unistd.h>

int main(void)
{
	//打开文件
	int fd = open("./shared.txt",O_WRONLY | O_CREAT | O_TRUNC,0664);
	if(fd==-1)
	{
		perror("open");
		return -1;
	}
	//写入数据
	char* buf = "hello world";
	ssize_t size = write(fd,buf,strlen(buf));
	if(size == -1)
	{
		perror("write");
		return -1;
	}
	printf("实际写入字节数为%ld\n",size);
	//关闭文件
	close(fd);
	return 0;
}
```

```
#read

#include<stdio.h>
#include<fcntl.h>
#include<unistd.h>

int main(void)
{
	//打开
	int fd = open("./shared.txt",O_RDONLY);
	if(fd == -1)
	{
		perror("open");
		return -1;
	}
	//读取
	char buf[128];
	ssize_t size = read(fd,buf,sizeof(buf)-1);
	if(size == -1)
	{
		perror("read");
		return -1;
	}
	printf("%s\n",buf);
	//关闭
	close(fd);
	return 0;
}
```



==lseek==修改文件读写位置；(文件读写位置位于文件表项中)

![image-20230806210453020](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806210453020.png)



==sizeof || strlen==

![image-20230806211935405](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806211935405.png)





![image-20230806212350453](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230806212350453.png)



### 标准I/O和系统I/O

`time`   

标准IO更快；（ fopen/fwrite )：fwrite基于write做了提升。

系统IO：open/write....

 

### 文件锁

![image-20230807095450974](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807095450974.png)

解决文件冲突：文件锁机制----读/写锁

![image-20230807101000841](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807101000841.png)

![image-20230807101135285](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807101135285.png)

![image-20230807101220258](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807101220258.png)

==fcntl==

![image-20230807101324971](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807101324971.png)

![image-20230807102000262](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807102000262.png)

```
#文件锁
#include<stdio.h>
#include<unistd.h>
#include<fcntl.h>
#include<string.h>

int main(int argc,char* argv[])
{
	//./a.out hello
	if(argc < 2)
	{
		fprintf(stderr,"用法:./a.out<字符串>\n");
		return -1;
	}
	//打开文件
	int fd = open("./conf.txt",O_WRONLY | O_CREAT | O_APPEND,0664);
	if(fd==-1)
	{
		perror("open");
		return -1;
	}
	
	//加锁
	struct flock lock;
	lock.l_type = F_WRLCK;
	lock.l_whence = SEEK_SET;
	lock.l_start = 0;
	lock.l_len = 0;
	lock.l_pid = -1;
	
	if(fcntl(fd,F_SETLKW,&lock)==-1)			//阻塞加锁，等到加锁成功。
	{
		perror("fcntl error.");
		return -1;
	}
	
	/*非阻塞加锁；
	while(fcntl(fd,F_SETLKW,&lock)==-1)	
	{
    if(errno == EACCESS || errno == EAGAIN)
    {
    	printf("文件被锁定，加不上锁,先做别的\n");
    	sleep(1);
    }
	else{
		perror("fcntl error.");
		return -1;
	}
	}
	*/
	
	//写入文件
	for(int i=0;i<strlen(argv[1]);i++)
	{
		write(fd,&argv[1][i],sizeof(argv[1][i]));
		sleep(1);
	}
	
	//解锁
	struct flock unlock;
	unlock.l_type = F_UNLCK;
	unlock.l_whence = SEEK_SET;
	unlock.l_start = 0;
	unlock.l_len = 0;
	unlock.l_pid = -1;
	
	if(fcntl(fd,F_SETLK,&unlock)==-1)			//解锁
	{
		perror("fcntl error.");
		return -1;
	}
	
	//关闭文件
	close(fd);
	return 0;
	
}
```



访问测试

![image-20230807105014793](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807105014793.png)

修改文件大小

![image-20230807105714203](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807105714203.png)



### 文件元数据

![image-20230807111105686](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807111105686.png)

stat. 把文件元数据传入buf。

a.out ---> 元数据 struct stat s --->mode_t st_mode ---> 第12bit (B11)为1 ---->设置用户ID。

B8-B0表示权限。

![image-20230807111822811](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807111822811.png)

文件设置了ID位(第12位),则有效用户ID就是设置的用户ID。否则进程有效用户ID就是登录身份。

```
#文件元数据

#include<stdio.h>
#include<string.h>
#include<unistd.h>
#include<sys/stat.h>
//转换类型和权限
char* mtos(mode_t m)
{
	static char s[11];					//-rw-rw-r--
	if(S_ISDIR(m))
	{
		strcpy(s,"d");
	}
	if(S_ISSOCK(m))
	{
		strcpy(s,"s");
	}
	if(S_ISCHR(m))
	{
		strcpy(s,"c");
	}
	if(S_ISBLK(m))
	{
		strcpy(s,"b");
	}
	if(S_ISFIFO(m))
	{
		strcpy(s,"p");
	}
	if(S_ISLINK(m))
	{
		strcpy(s,"l");
	}
	else
	{
		strcpy(s,"-");
	}
	strcat(m & S_IRUSR > "r":"-");
	strcat(m & S_IWUSR > "w":"-");
   	strcat(m & S_IXUSR > "x":"-");
   	
	...组
	...其他			man 2 stat
   	
   	//粘滞位判断
   	...
   	
	return s;
}

int main(int argc,char* argv[])
{
	if(argc < 2)
	{
		fprintf(stderr,"用法:./a.out<文件名>\n");
		return -1;
	}
	struct stat buf;
	if(stat(argv[1],&buf) == -1)
	{
		perror("stat");
		return -1;
	}
	return 0;
}
```





## 06 进程

![image-20230807205658062](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807205658062.png)

### 相关命令

`pstree`显示进程间关系

`ps aux`

`top`	类似任务管理器	ctrl+c退出



### 父子孤尸

![image-20230807210658080](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807210658080.png)



孤儿院进程：

孤儿进程：

父进程先结束，子进程为孤儿进程。

![image-20230807211030148](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807211030148.png)



==僵尸进程==

子进程先于父进程结束；但子进程的资源要由父进程回收；此时父进程未结束、没有回收子进程资源；则子进程为僵尸进程。

![image-20230807211111332](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807211111332.png)

![image-20230807211315929](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807211315929.png)

PPID：自己的父进程标识

```
//进程的ID
#include<stdio.h>
#include<unistd.h>

int main(void)
{
	printf("进程的PID:%d\n",getpid());
	printf("父进程的PID:%d\n",getppid());
	//实际用户id：getuid(),有效用户id：geteuid
	//实际组id：getgid(),有效组id：getegid
	return 0;
}
```





### 创建子进程

==fork==:在父进程中返回子进程PID。在子进程中返回0

![image-20230807212238377](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807212238377.png)

```
#include<stdio.h>
#include<unistd.h>

int main(void)
{	
	printf("%d进程：我是父进程\n",getpid());
	pid_t a = fork();
	if(a==-1)
	{
		perror("fork");
		return -1;
	}
	if(a==0){
		printf("%d进程:锅包肉\n",getpid());
	}
	else{
		printf("%d进程:地三鲜\n",getpid());
	}
	printf("%d进程：铁锅炖大鹅\n",getpid());
	
	return 0;
}
```

fork之后的部分父子各同时执行一遍。



子进程从父进程拷贝除代码区以外的数据；代码区数据是共享的(只读的，不可修改)

子进程也从父进程拷贝文件表项。

![image-20230807215055082](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807215055082.png)

![image-20230807221111815](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807221111815.png)

```
#include<stdio.h>
#include<unistd.h>
#include<fcntl.h>
#include<string.h>

int main(void)
{
	//父进程打开文件
	int fd = open("./ftab.txt",O_WRONLY | O_CREAT | O_TRUNC,0664);
	if(fd == -1)
	{
		perror("open");
		return -1;
	}
	//父进程向文件写入数据 Hello
	char *buf = "Hello world";
	if(write(fd,buf,strlen(buf))==-1){
		perror("write");
		return -1;
	}
	//父进程创建子进程
	pid_t pid == fork();
	if(pid == -1)
	{
		perror("fork");
		return -1;
	}
	//子进程代码 改文件读写位置
	if(pid == 0){
		if(lseek(fd,-6,SEEK_END)==-1)
		{
			perror("lseek");
			return -1;
		}
		close(fd);			//子进程关
		return 0;
	}
	sleep(1);
	buf = "Linux";
	//父进程代码 再写入Linux.
	if(write(fd,buf,strlen(buf))==-1){
		perror("write");
		return -1;
	}
	close(fd);
	return 0;
}
```



### 进程的终止

正常终止:

==1.main函数返回==

main() return值给进程的父进程。父进程

==2.exit==

![image-20230807230651431](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807230651431.png)

==3.atexit()==

注册退出处理函数。

```
void doit(void)
{
	....
}

atexit(doit);			//正常结束都会调用doit，遗言函数。
```



==4.on_exit()==

接受函数地址、串地址，传给内核。 函数status是正常结束的返回值。

![image-20230807232351549](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230807232351549.png)























异常终止:



人为异常终止：

ctrl+c	//	ctrl+\





### 回收子进程

![image-20230808203131074](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230808203131074.png)

![image-20230808203659214](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230808203659214.png)

![image-20230808203807761](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230808203807761.png)

![image-20230808204113002](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230808204113002.png)

```
#回收子进程

#include<stdio.h>
#include<unistd.h>
#include<sys/wait.h>

int main(void)
{
	pid_t pid = fork();
	if(pid == -1)
	{
		perror("fork");
		return -1;
	}
	if(pid == 0)
	{
		printf("%d进程:我是子进程\n",getpid());
		sleep(5);
		return 0;
	}
	int s;										      //保存回收子进程的终止状态
	pid_t childpid = wait(&s);
	if(childpid == -1)
	{
		perror("wait");
		return -1;
	}
	printf("%d进程:我是父进程，回收了%d进程的僵尸。"，getpid(),childpid);
	
	if(WIFEXITED(s))
	{
		printf("正常终止:%d\n",WEXITSTATUS(s));			//正常终止:0.
	}
	else
	{
		printf("异常终止:%d\n",WTERMSIG(s));
	}
	return 0;
}
```



回收多个子进程的死循环例：

```
for(;;)
{
	int s;
	pid_t childpid = wait(&s);
	if(childpid == -1)
	{
		if(errno == ECHILD)
		{
			printf("没有子进程回收\n");				//wait函数返回-1,errno置ECHILD。
			break;
		}
		else
		{
			perror("wait");
			return -1;
		}
	}
}
```





![](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230808211845912.png)

![image-20230808212032831](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230808212032831.png)

以非阻塞模式进行waitpid，但是子进程没死，则返回0.



### 创建新进程

![image-20230808220751138](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230808220751138.png)







execl

```
//变身. new.c
#include<stdio.h>
#include<unistd.h>

int main(int argc,char* argv[],char* envp[])
{
	printf("PID: %d\n",getpid());
	printf("命令行参数内容\n");
	for(char** pp = argv;*pp;pp++)
	{
		printf("%s\n",*pp);
	}
	printf("环境变量\n");
	for(char** pp=envp;*pp;pp++)
	{
		printf("%s\n",*pp);
	}
	printf("---------------------\n");
	
	
	return 0;
}

gcc new.c -o new
```

```
#新进程创建 execl.c

#include<stdio.h>
#include<unistd.h>

int main(void)
{
	printf("%d进程:我要变身了\n",getpid());
	
	if(execl("./new","new",hello,NULL) == -1)				//后面几项命令行参数不是必须的；
	{
		perror("execl");
		return -1;
	}
	printf("进程:%d变身完成\n",getpid());						//这句话属于旧进程。execl后程序就开始执行new程序了。
	return 0;
}
```

![image-20230809103956324](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809103956324.png)

![image-20230809104050617](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809104050617.png)



![image-20230809105049618](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809105049618.png)

```
#forkexec.c

#include<stdio.h>
#include<unistd.h>
#include<sys/wait.h>

int main(void)
{
	//创建子进程
	pid_t pid = fork();
	if(pid == -1)
	{
		perror("fork");
		return -1;
	}
	//子进程代码
	if(pid == 0)
	{
		if(execl("./new","new","hello",NULL) == -1)
		{
			perror("execl");
			return -1;										//变身失败，return-1.变身成功，执行./new也不需要再写return.
		}
		//return 0;
	}
	
	int s;					//终止状态
	if(waitpid(pid,&s,0) == -1)
	{
		perror("waitpid");
		return -1;
	}
	if(WIFEXITED(s))
	{
		printf("正常终止:%d\n",WEXITSTATUS(s));
	}
	else
	{
		printf("异常终止:%d\n",WTERMSIG(s));
	}
	
	//子进程2.
	pid = fork();
	if(pid == -1)
	{
		perror("fork");
		return -1;
	}
	if(pid == 0)
	{
		if(execl("/bin/ls","ls","-l","--color=auto",NULL)==-1)
		{
			perror("execl");
			return -1;
		}
	}
	if(waitpid(pid,&s,0) == -1)
	{
		perror("waitpid");
		return -1;
	}
	if(WIFEXITED(s))
	{
		printf("正常终止:%d\n",WEXITSTATUS(s));
	}
	else
	{
		printf("异常终止:%d\n",WTERMSIG(s));
	}	
	
	return 0;
}
```





==system==

![image-20230809111022614](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809111022614.png)

system 类似 fork+exec.

```
#system.c

#include<stdio.h>
#include<sys/wait.h>
#include<stdlib.h>

int main(void)
{
	int s = system("./new hello 123");
	if(WIFEXITED(s))
	{
		printf("正常结束:%d\n",WEXITSTATUS(s));
	}
	else
	{
		printf("异常终止:%d\n",WTERMSIG(s));
	}
}
```







## 07 信号

提供异步事件处理机制的软件中断。

(不可预知什么时候到来、)

![image-20230809113831210](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809113831210.png)

![image-20230809114210677](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809114210677.png)

信号本质就是整数；

`kill -l`查看所有信号

共62个信号，前31个为不可靠信号；后31个可靠信号。

```
//转储文件

#include<stdio.h>

int main(void)
{
	int *p = NULL;
	*p = 12;							//段错误
	return 0;	
}
```



### 信号处理

![image-20230809115644689](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809115644689.png)

![image-20230809115906800](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809115906800.png)

==返回值==

![image-20230809132959144](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809132959144.png)

```
typedef void(*sighandeler_t)(int);
typedef int* p;
p --> int*;
sighandler_t --> void(*)(int);
```

```
//信号处理
#include<stdio.h>
#include<unistd.h>
#include<signal.h>

typedef void(*sighander_t)(int);

//信号处理函数
void sigfun(int signum)
{
	printf("%d进程:捕获到%d信号\n",getpid(),signum);		
}

int main(void)
{
	//忽略2号信号			Ctrl+C 2号信号无法终止死循环了; ctrl+\ 3号
	sighander_t ret = signal(SIGINT,SIG_IGN);
	if(ret == SIG_ERR)
	{
		perror("signal");
		return -1;
	}
	printf("ret = %p\n",ret);				//返回==该信号==的上一次处理方式——————之前没处理过2号信号，返回NULL
	//捕获2号信号
	ret = signal(SIGINT,sigfun);
	if(ret == SIG_ERR)
	{
		perror("signal");
		return -1;
	}
	printf("ret = %p\n",ret);				//返回上一次处理方式——————返回1.忽略方式
	for(;;);
	return 0;
}

```



### 太平间信号 

17号信号,默认忽略。

正常结束：exit/return 

冲刷IO

....

_exit：1.关文件描述符	2.托孤(子进程给孤儿院进程)    3.向父进程发17号信号

==想法==

捕获17号信号，回收子进程尸体。//体会与waitpid的区别；

![image-20230809134756210](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809134756210.png)

![image-20230809135316571](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809135316571.png)

```
//太平间信号
#include<stdio.h>
#include<unistd.h>
#include<signal.h>
#include<sys/wait.h>

//信号处理函数，收尸子进程。
void sigchild(int signum)
{
	printf("%d进程:捕获到%d信号\n",getpid(),signum);
	pid_t pid = wait(NULL);
	if(pid == -1)
	{
		perror("wait");
		return;
	}
	printf("%d进程:回收了%d进程的僵尸",getpid(),pid);
}
int main(void)
{
	//捕获17号信号
	if(signal(SIGCHLD,sigchild) == SIG_ERR)
	{
		perror("signal");
		return -1;
	}
	//创建5个子进程
	for(int i=0;i<5;i++)
	{
		pid_t pid = fork();
		if(pid == -1)
		{
			perror("fork");
			return -1;
		}
		if(pid == 0)
		{
			printf("%d进程:我是子进程\n",getpid());
			sleep(i+1);
			return 0;
		}
	}
	//父进程代码
	for(;;);
	
	return 0;
}
```

P.S.太平间信号属于不可靠信号；会丢失。

==在信号处理函数执行期间，若有多个相同信号再次到来，则只保留一个，其余丢弃。==

解决：再一次信号处理函数中，尽可能多的收尸。

```
//阻塞方式
void sigchild(int signum)
{
	sleep(3)
	printf("%d进程:捕获到%d信号\n",getpid(),signum);
	pid_t pid = wait(NULL);
	for(;;)							//在一次信号处理函数中，尽可能多的收尸。
	{
		if(pid == -1)
		{
			if(errno = ECHILD)
			{
				printf("%d进程:没有子进程\n",getpid());
				break;
			}
			else
			{
				perror("wait");
				return;
			}
		}
		else
		printf("%d进程:回收了%d进程的僵尸",getpid(),pid);
	}
}
```

![image-20230809154956312](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809154956312.png)

收到17号信号；处理子程序处理5个子进程；后面还会同时受到4个17号信号；但是只保留一个，所以捕获到17号信号，但是没有子进程了，所以输出没有子进程。

```
//非阻塞方式

void sigchild(int signum)
{
	for(;;)
	{
		pid_t pid = waitpid(-1,NULL,WNOHANG);
		if(pid == -1)
		{
			if(errno == ECHILD)
			{
				printf("%d进程:没有子进程了\n",getpid());
				break;
			}
			else
			{
				perror("waitpid");
				return;			
			}
		}
			else if(pid == 0)
			{
				printf("%d进程:子进程正在运行,没法收\n",getpid());
				break;				
			}
			else
			{
				printf("%d进程:回收了%d进程的僵尸\n",getpid());
			}
			
		}
	}
}
```





### 信号处理的继承与恢复

```
//子进程是否会继承父进程的信号处理方式

#include<stdio.h>
#include<unistd.h>
#include<signal.h>
#include<sys/wait.h>
//信号处理函数
void sigfun(int signum)
{
	printf("%d进程:捕获到%d信号\n",getpid(),signum);
}

int main(void)
{
	//忽略2号信号
	if(signal(SIGINT,SIG_IGN) == SIG_ERR)
	{
		perror("signal");
		return -1;
	}
	//捕获3号信号
	if(signal(SIGQUIT,sigfun) == SIG_ERR)
	{
		perror("signal");
		return -1;
	}
	//创建子进程
	pid_t pid = fork();
	if(pid == -1)
	{
		perror("fork");
		return -1;
	}
	if(pid == 0)
	{
		printf("%d进程:我是子进程\n",getpid());
		sleep(20);
		return 0;
	}
	//父进程代码
	if(wait(NULL) == -1)
	{
		perror("wait");
		return -1;
	}
	printf("%d进程:回收了子进程的僵尸\n",getpid());
	return 0;
}
```

`kill -2 17095`    给17095进程发送2号信号与3号信号。

==父进程忽略/捕获什么信号，子进程同样忽略/捕获该信号==；



exec变身

```
//变身对象
#include<stdio.h>
#include<unistd.h>

int main(void)
{
	printf("PID:%d\n",getpid());
	for(;;);
	return 0;
}
```

```
//新进程是否会继承旧进程处理方式
#include<stdio.h>
#include<unistd.h>
#include<signal.h>

//信号处理函数
void sigfun(int signum)
{
	printf("%d进程:捕获到%d信号\n"，getpid(),signum);
}

int main(void)
{
//忽略2号信号
	if(signal(SIGINT,SIG_IGN) == SIG_ERR)
	{
		perror("signal");
		return -1;
	}
//捕获3号信号
	if(signal(SIGQUIT,sig_fun) == SIG_ERR)
	{
		perror("signal");
		return -1;	
	}
//变身为new
	if(execl("./new","new",NULL) == -1)
	{
		perror("execl");
		return -1;
	}
	return 0;
}
```

==变身之前忽略谁，变身之后仍然忽略；变身之前捕获谁，变身后恢复默认处理==

原因：依赖于信号处理函数；

![image-20230809163737462](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809163737462.png)



### 发送信号

![image-20230809163836230](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809163836230.png)

![image-20230809164117189](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809164117189.png)

`kill 0`验证进程存在；

```
//kill.c

#include<stdio.h>
#include<unistd.h>
#include<signal.h>
#include<stdlib.h>
#include<errno.h>
void sigfun(int signum)
{
	printf("%d进程:捕获到%d信号\n",getpid(),signum);
}
//判断进程存在函数
int isexist(pid_t pid)
{
	if(kill(pid,0) == -1)
	{
		if(errno == ESRCH)
		{	//失败因为进程不存在，返回1
			return 1;
		}
		else
		{
			perror("kill");
			exit(-1);
		}
	}
	return 0;	//kill成功，返回0
}
int main(void)
{
	//创建子进程
	pid_t pid = fork();
	if(pid == -1)
	{
		perror("fork");
		return -1;
	}
	//子进程对2号进行捕获
	if(pid == 0)
	{
		printf("%d进程:我是子进程\n",getpid());
		/*if(signal(SIGINT,sigfun) == SIG_ERR)
		{
			perror("signal");
			return -1;
		}*/
		sleep(10);
	}
	//父进程向子进程发送2号信号
	printf("%d进程:我要向子进程发送2号信号\n",getpid());
	getchar();							//控制程序执行，什么时候敲回车，什么时候进行kill.
	if(kill(pid,SIGINT) == -1)
	{
		perror("kill");
		return -1;
	}
	getchar();
	printf("子进程%s\n",isexist(pid)? "不存在":"存在");
	
	//收尸，子进程因为收到2号信号成为僵尸。
	if(wait(NULL) == -1)
	{
		perror("wait");
		return -1;
	}
	
	return 0;
}
```



### 暂停睡眠与闹钟

暂停

![image-20230809171839142](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809171839142.png)

```
//pause.c

#include<stdio.h>
#include<unistd.h>
#include<signal.h>
#include<sys/wait.h>
//信号处理函数
void sigfun(int signum)
{
	printf("%d进程:%d信号处理开始\n",getpid(),signum);
	sleep(3);
	printf("%d进程:%d信号处理结束\n",getpid(),signum);
}
int main(void)
{
//创建子进程；
	pid_t pid = fork();
	if(pid == -1)
	{
		perror("fork");
		return -1;
	}
//子进程pause
	if(pid == 0)
	{
	//子进程捕获2号信号，苏醒；
		if(signal(SIGINT,sigfun) == SIG_ERR)
		{
			perror("signal");
			return -1;
		}
		printf("%d进程:我是子进程，我pause\n",getpid());
		int res = pause();
		printf("%d进程:pause函数返回%d\n",getpid(),res);
		return 0;
	}
//父进程发信号打断子进程pause。
	printf("%d进程:我是父进程，给子进程发2号信号。\n",getpid());
	getchar();
	if(kill(pid,SIGINT) == -1)
	{
		perror("kill");
		return -1;
	}
	//收尸。
	if(wait(NULL) == -1)
	{
		perror("wait");
		return -1;
	}
	return 0;
}
```

==信号处理函数结束了以后pause才返回==



![image-20230809173659069](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230809173659069.png)

信号打断sleep时间，sleep返回sleep剩余时间

```
#include<stdio.h>
#include<unistd.h>
#include<signal.h>
#include<sys/wait.h>
//信号处理函数
void sigfun(int signum)
{
	printf("%d进程:%d信号处理开始\n",getpid(),signum);
	sleep(3);
	printf("%d进程:%d信号处理结束\n",getpid(),signum);
}
int main(void)
{
//创建子进程；
	pid_t pid = fork();
	if(pid == -1)
	{
		perror("fork");
		return -1;
	}
//子进程pause
	if(pid == 0)
	{
	//子进程捕获2号信号，苏醒；
		if(signal(SIGINT,sigfun) == SIG_ERR)
		{
			perror("signal");
			return -1;
		}
		printf("%d进程:我是子进程，我要睡眠\n",getpid());
		int res = sleep(10);
		printf("%d进程:sleep函数返回%d\n",getpid(),res);
		return 0;
	}
//父进程发信号打断子进程pause。
	printf("%d进程:我是父进程，给子进程发2号信号。\n",getpid());
	getchar();
	if(kill(pid,SIGINT) == -1)
	{
		perror("kill");
		return -1;
	}
	//收尸。
	if(wait(NULL) == -1)
	{
		perror("wait");
		return -1;
	}
	return 0;
}
```





闹钟，alarm(p)     p秒后发送14号信号

闹钟重置：两次alarm调用；第二次alarm返回上一个闹钟剩余时间。

```
#include<stdio.h>
#include<unistd.h>
#include<signal.h>
//信号处理函数
void sigfun(int signum)
{
	printf("%d进程:捕获到%d信号\n",getpid(),signum);
}
int main(void)
{
	//捕获14号信号
	if(signal(SIGALRM,sigfun) == SIG_ERR)
	{
		perror("signal");
		return -1;
	}
	//设置闹钟
	printf("alarm(10)返回%d\n",alarm(10));
	getchar();
	printf("alarm(5)返回%d\n",alarm(5));
	for(;;);
	return 0;
}
```





### 信号集 信号屏蔽

sigset_t 信号集      128字节 存储区~ 1024bit位。

![image-20230810141453584](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810141453584.png)

信号集操作相关函数

![image-20230810141528133](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810141528133.png)

![image-20230810141641351](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810141641351.png)

![image-20230810141841823](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810141841823.png)

![image-20230810141932955](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810141932955.png)

![image-20230810141942160](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810141942160.png)











信号屏蔽

递送亦即送达；

**![image-20230810142754820](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810142754820.png)**



信号掩码中有什么信号，就会阻拦什么信号递送。


![image-20230810143311510](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810143311510.png)

![image-20230810143326659](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810143326659.png)





![image-20230810143650242](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810143650242.png)

![image-20230810143658176](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810143658176.png)

设置信号掩码；

![image-20230810143907781](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810143907781.png)

获取被屏蔽的信号：sigset是输出型参数。

![image-20230810144126320](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810144126320.png)





![image-20230810144346223](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810144346223.png)

```
//对信号进行屏蔽处理
#include<stdio.h>
#include<unistd.h>
#include<signal.h>
//信号处理函数
void sigfun(int signum)
{
	printf("%d进程:捕获到%d信号",getpid(),signum);
}
//假装更新数据库
void updatedb(void)
{
	for(int i=0;i<5;i++)
	{
		printf("正在更新第%d条数据...\n",i+1);
		sleep(1);
	}
}

int main(void)
{
	//对2号信号捕获处理
	int signum = SIGINT;
	if(signal(signum,sigfun) == SIG_ERR)
	{
		perror("signal");
		return -1;
	}
	//父进程屏蔽2号信号
	printf("%d进程:屏蔽%d信号",getpid(),signum);
	sigset_t sigset;							//信号集变量
	sigempty(&sigset);							//清空信号集
	sigaddset(&sigset,signum);					//添加2号信号到信号集	
	sigset_t oldset;							//放以前的信号掩码
	if(sigprocmask(SIG_SETMASK,&sigset,&oldset) == -1)
	{
		perror("sigprocmask");
		return -1;
	}
	//父创建子
	pid = fork();
	if(pid == -1)
	{
		perror("fork");
		return -1;
	}
	if(pid == 0)
	{
	//子进程代码
		for(int i=0;i<5;i++)
		{
	//子进程向父进程发送2号信号
			printf("%d进程:我要向父进程发送%d信号",getpid(),signum);
			kill(getppid(),signum);
		}
		return 0;
	}
	//父进程假装更新数据库
	updatedb();
	//父进程解除对2号信号屏蔽
	printf("%d进程:解除对%d信号的屏蔽\n",getpid(),signum);
	if(sigprocmask(SIG_SETMASK,&oldset,NULL) == -1)
	{
		perror("sigprocmask");
		return -1;
	}
	for(;;);
	return 0;
}
```





## 08 进程间通信 IPC



不同进程之间存在内存壁垒，不能通过虚拟地址实现数据通信-----------需要进程间通信IPC。

![image-20230810152018321](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810152018321.png)

------------------------------------

专门手段

![image-20230810152232592](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810152232592.png)

![image-20230810152251114](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810152251114.png)



### 有名管道 FIFO

![image-20230810153042187](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810153042187.png)

有名管道文件只是传数据；是内核维护的==缓冲区==。

![image-20230810153738063](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810153738063.png)

```
//terminal1 写入fifo
$mkfifo myfifo
$echo hello >myfifo			//重新定向

//terminal2 读出fifo
$cat myfifo

//查看文件大小 $ls -l
```



![image-20230810154118789](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810154118789.png)

```
//wfifo.c
//rfifo.c
```

```
#wfifo.c
#include<stdio.h>
#include<unistd.h>
#include<string.h>
#include<fcntl.h>
#include<sys/stat.h>

int main(void)
{
	//创建有名管道文件
	printf("%d进程:创建有名管道文件\n",getpid());
	if(mkfifo("./fifo",0664) == -1)
	{
		perror("mkfifo");
		return -1;
	}
	//打开
	printf("%d进程:打开有名管道文件\n",getpid());
	int fd = open("./fifo",O_WRONLY);
	if(fd == -1)
	{
		perror("open");
		return -1;
	}
	//写入
	printf("%d进程:写入\n",getpid());
	for(;;)
	{
		//通过键盘获取要写入的数据；
		char buf[128];
		fgets(buf,sizeof(buf),stdin);					//gets会把"!"与回车一起输入。
		//指定退出条件,输入"!"退出循环
		if(strcmp(buf,"!\n") == 0)
		{
			break;
		}
		//将数据写入管道文件
		if(write(fd,buf,strlen(buf)) == -1)
		{
			perror("write");
			return -1;
		}
	}
	
	//关闭
	printf("%d进程:关闭有名管道文件\n",getpid());
	close(fd);
	//删除
	printf("%d进程:删除有名管道文件\n",getpid());
	unlink("./fifo");
    return 0;
}
```

```
//rfifo.c
#include<stdio.h>
#include<unistd.h>
#include<fcntl.h>

int main(void)
{
	//打开
	printf("%d进程:打开有名管道文件\n",getpid());
	int fd = open("./fifo",O_RDONLY);
	if(fd == -1)
	{
		perror("open");
		return -1;
	}	
	//读取
    printf("%d进程:读取有名管道文件\n",getpid());
	for(;;)
	{
		char buf[128] = {};
		ssize_t size = read(fd,buf,sizeof(buf)-1);			//read函数返回0,表示对端关闭管道了。
		if(size == -1)
		{
			perror("read");
			return -1;
		}
		if(size == 0)
		{
			printf("%d进程:对方关闭管道文件\n",getpid());
			break;
		}
		printf("%s",buf);
	}	
	//关闭
	printf("%d进程:关闭有名管道\n",getpid());
	close(fd);
	printf("%d进程:结束\n",getpid());
	return 0;
}
```







### 无名管道 pipe

![image-20230810162459842](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810162459842.png)

```
#include<unistd.h>
//创建无名管道
int pipefd[2];
pipe(pipefd);				//创建出无名管道后，读端和写端描述符会分别存到数组中。

//写入数据
write(pipefd[1],buf,strlen(buf));

//读取数据
read(pipefd[0],buf,sizeof(buf)-1);
```



![image-20230810162730190](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810162730190.png)

![image-20230810162745942](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810162745942.png)

子进程复制了父进程的文件和文件描述符表，从而可以在父子进程间创建无名管道通信。

```
//pipe.c
#include<stdio.h>
#include<unistd.h>
#include<string.h>
#include<sys/wait.h>

int main(void)
{
//父进程创建无名管道
	printf("%d进程:创建无名管道\n",getpid());
	int fd[2];
	if(pipe(fd) == -1)
	{
		perror("pipe");
		return -1;
	}
	printf("fd[0]=%d\n",fd[0]);
	printf("fd[1]=%d\n",fd[1]);

//父进程创建子进程
	pid_t pid = fork();
	if(pid == -1)
	{
		perror("fork");
		return -1;
	}
//先写子进程。否则子进程也会执行父进程部分代码。
	if(pid == 0)
	{
		printf("%d进程:关闭写端\n",getpid());
		close(fd[1]);
		printf("%d进程:接受数据\n",getpid());
		for(;;)
		{
			char buf[128] = {};
			ssize_t size = read(fd[0],buf,sizeof(buf)-1);
			if(size == -1)
			{
				perror("read");
				return -1;
			}
			if(size == 0)
			{
				printf("%d进程:父进程关闭写端\n",getpid());
				break;
			}
			printf("----> %s",buf);
		}
		printf("%d进程:关闭读端\n",getpid());
		close(fd[0]);
		printf("%d进程:读取完成\n",getpid());
		return 0;
	}
	
	//父进程发送；子进程接受
	printf("%d进程:关闭读端\n",getpid());
	close(fd[0]);
	printf("%d进程:发送数据\n",getpid());	
	for(;;)
	{
		char buf[128] = {};
		fgets(buf,sizeof(buf),stdin);
		if(strcmp(buf,"!\n") == 0)
		{
			break;
		}

		if(write(fd[1],buf,strlen(buf)) == -1)
		{
			perror("write");
			return -1;
		}
	}
	printf("%d进程:关闭写端\n",getpid());
	close(fd[1]);
	
	//父进程收尸
	if(wait(NULL) == -1)
	{
		perror("wait");
		return -1;
	}
	printf("%d进程:结束\n",getpid());

}
```



![image-20230810171703538](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810171703538.png)

![image-20230810172130245](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810172130245.png)









### 管道符号



分屏显示`ls -l /etc | more`	"|"就是管道符号。链接两个命令；

==让前面命令输出作为后面命令输入==

a进程复制文件描述符，并且把写端描述符从4复制到1；

a创建b进程，a创建管道；

b复制文件描述符，把读端从3复制到0

a编程ifconfig;b变身grep

所以a输出相当于在向管道写；b输入相当于读。

![image-20230810172833845](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810172833845.png)





### IPC对象

不同于文件描述符；IPC对象编号是顺序递增的。![image-20230810233549487](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810233549487.png)

![image-20230810233803936](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810233803936.png)



==合成键：==

![image-20230810234125961](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810234125961.png)

通过路径找i节点号、设备ID；

![image-20230810234316183](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810234316183.png)



### 共享内存

![image-20230810234508909](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810234508909.png)

![image-20230810234648267](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810234648267.png)



一般：  IPC_CREAT | IPC_EXCL， 往往只想新建共享内存。

==新建共享内存==

![image-20230810234832015](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810234832015.png)



==加载==

![image-20230810235129277](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810235129277.png)

shmat函数:将给定共享内存映射到调用进程的虚拟内存空间；返回映射区域起始地址。同时系统内核中共享内存对象加载计数加1。

调用进程得到映射区域起始地址后即可访问。



==解除映射==![image-20230810235443855](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810235443855.png)

==销毁==

若不手动销毁共享内存，则其会一直存在。

![image-20230810235540044](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810235540044.png)

`man 2 shmctl`



![image-20230810235908730](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230810235908730.png)

```
//touch wshm.c
//touch rshm.c
```

```
//写入
#include<stdio.h>
#include<string.h>
#include<unistd.h>
#include<sys/shm.h>

int main(void)
{	
	//合成键
	printf("%d进程:合成键\n",getpid());
	key_t key = ftok(".",100);						//当前目录；
	if(key == -1)
	{
		perror("ftok");
		return -1;
	}
	//创建共享内存
	printf("%d进程:创建共享内存\n",getpid());	
	int shmid = shmget(key,4096,IPC_CREAT | IPC_EXCL | 0664);
	if(shmid == -1)
	{
		perror("shmget");
		return -1;
	}
	//加载共享内存
	printf("%d进程:加载共享内存\n",getpid());		
	char* start = shmat(shmid,NULL,0);				//返回值从void*-->char*
	if(start == (void*)-1)							//-1转化成void*,作比较
	{
		perror("shmat");
		return -1;
	}
	//写入数据
	printf("%d进程:写入数据\n",getpid());	
	sprintf(start,"key=0x%x,shmid=%d,pid=%d\n",key,shmid,getpid());									//向指定位置写入；
	//卸载共享内存
	printf("%d进程:卸载共享内存\n",getpid());
	if(shmdt(start) == -1)
	{
		perror("shmdt");
		return -1;
	}
	//删除共享内存
	getchar();
	printf("%d进程:删除共享内存\n",getpid());		
	if(shmctl(shmid,IPC_RMID,NULL)==-1)
	{
		perror("shmctl");
		return -1;
	}
	printf("%d进程:结束\n",getpid());
	return 0;
}
```

```
//读取共享内存 rshm.c
#include<stdio.h>
#include<unistd.h>
#include<sys/shm.h>

int main(void)
{
	//合成键.	ftok		键与wshm.c目录一致。
	printf("%d进程:合成键\n");
	key_t key = ftok(".",100);
	if(key==-1)
	{
		perror("ftok");
		return -1;
	}
	//获取	shmget
	printf("%进程:获取共享内存\n");
	int shmid = shmget(key,0,0);				//获取共享内存，给0。
	if(shmid == -1)
	{
		perror("shmget");
		return -1;
	}
	//加载	shmat
	printf("%d进程:加载共享内存\n");
	char* start = shmat(shmid,NULL,0);
	if(start == (void*)-1)
	{
		perror("shmat");
		return -1;
	}
	//使用
	printf("%d进程:读取数据\n",getpid());
	getchar();
	printf("%s\n",start);
	//卸载	shmdt
	printf("%d进程:卸载共享内存\n");
	if(shmdt(start) == -1)
	{
		perror("shmdt");
		return -1;
	}	
	printf("%d进程:任务完成\n",getpid());
	return 0;
}
```

`$ipcs` 查看共享内存信息；

![image-20230811103420769](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811103420769.png)





###  消息队列

![image-20230811103655803](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811103655803.png)

创建消息队列		==msgget==

进程A通过相关函数向链表插入消息；----发送 ==msgsnd==

进程B通过相关函数取出链表节点；----接收 ==msgrcv==

![image-20230811104427117](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811104427117.png)

![image-20230811104729030](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811104729030.png)





![image-20230811104502835](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811104502835.png)



-----------

msgget( )

![image-20230811104759442](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811104759442.png)

msgsnd()

默认阻塞方式。

![image-20230811104923269](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811104923269.png)

![image-20230811110416739](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811110416739.png)

![image-20230811110246787](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811110246787.png)







msgrcv

![image-20230811110505692](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811110505692.png)

![image-20230811110517232](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811110517232.png)

![image-20230811110657834](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811110657834.png)

![image-20230811111030413](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811111030413.png)

`int size = msgrcv(msgid,&msgbuf,sizeof(msgbuf.data)-1,123,0);`  消息队列id、存储区地址、存储数据大小、消息类型、默认阻塞方式（errno = EIDRM.）(IPC_NOWAIT-非阻塞方式---没收到消息errno = ENOMSG)

非阻塞方式对方销毁消息队列：errno 不会等于EIDRM。







==销毁消息队列==

![image-20230811111117843](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811111117843.png)

![image-20230811111125603](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811111125603.png)

```
//wmsg.c
#include<stdio.h>
#include<string.h>
#include<unistd.h>
#include<sys/msg.h>

int main(void)
{
	//合成键
	printf("%d进程:合成键\n",getpid());
	key_t key = ftok(".",111);
	if(key == -1)
	{
		perror("ftok");
		return -1;
	}
	//创建消息队列
	printf("%d进程:创建消息队列\n",getpid());
	int msgid = msgget(key,IPC_CREAT | IPC_EXCL | 0664);
	if(msgid == -1)
	{
		perror("msgget");
		return -1;
	}
	//发送消息 msgsnd
	printf("%d进程:发送消息\n",getpid());
	for(;;)
	{
		//通过键盘获取要发送的数据
		struct
		{
			long type;						//消息类型
			char data[128];					//消息内容
		} msgbuf = {123,""};
		fgets(msgbuf.data,sizeof(msgbuf.data),stdin);
		if(strcmp(msgbuf.data,"!\n")==0)
		{
			break;
		}
		if(msgsnd(msgid,&msgbuf,strlen(msgbuf.data),0) == -1)
		{
			perror("msgsnd");
			return -1;
		}
	} 
	//销毁消息队列
	printf("%d进程:销毁消息队列\n",getpid());	
	if(msgctl(msgid,IPC_RMID,NULL)==-1)
	{
		perror("msgctl");
		return -1;
	}
	printf("%d进程:完成发送\n",getpid());
	return 0;
}
```

```
//rmsg.c
#include<stdio.h>
#include<unistd.h>
#include<string.h>
#include<sys/msg.h>
#include<errno.h>

int main(void)
{
	//合成键
	printf("%d进程:合成键\n",getpid());
	key_t key = ftok(".",111);
	if(key == -1)
	{
		perror("ftok");
		return -1;
	}
	//获取消息队列 msgget
	printf("%d进程:获取消息队列\n",getpid());
	int msgid = msgget(key,0);
	if(msgid == -1)
	{
		perror("msgget");
		return -1;
	}
	//接收消息	msgrcv
	printf("%d进程:接收消息\n",getpid());
	for(;;)
	{
		struct
		{
			long type;
			char data[128];
		} msgbuf =  {};

		int size = msgrcv(msgid,&msgbuf,sizeof(msgbuf.data)-1,123,0);	//接收消息内容不能比缓冲区msgbuf.data还大；
		if(size == -1)
		{
            if(errno == EIDRM)
            {
                printf("%d进程:对方销毁了消息队列\n",getpid());			//当进程等待接收消息，消息队列被移除了,msgrcv返回EIDRM。
                break;
            }else
            {
                perror("msgrcv");
                return -1;		
            }
        } 
     	printf("%ld>%s",msgbuf.type,msgbuf.data);
	}
	printf("%d接收完成\n",getpid());	
	return 0;
}
```

`ipcs`可查看消息队列。

![image-20230811120918785](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811120918785.png)

`ipcrm -q +消息队列ID`





## 09 网络



![image-20230811155316148](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811155316148.png)

`查看网络信息 ifconfig (Linux)    ipconfig(windows)`



![image-20230811162507168](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811162507168.png)



### 创建套接字 socket



![image-20230811162925605](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811162925605.png)

![image-20230811163004563](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811163004563.png)





### 端序 转换函数

![image-20230811164309407](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811164309407.png)

内存区首地址一般是低地址。

![image-20230811164910058](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230811164910058.png)



### TCP

三次握手，四次分手

P93.



![image-20230812092557269](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230812092557269.png)

![image-20230812093440045](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230812093440045.png)







### 函数_TCP

==//写入、关闭、读取==

```c
#include<unistd.h>
int close(int fd);			//成功返回0
ssize_t write(int fd,const void* buf,size_t nbytes);
ssize_t read(int fd,void* buf,size_t nbytes);
```

==//打开文件==

```c
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>

int open(const char* path,int flag);
//模式
O_CREAT		O_TRUNC		O_APPEND		O_RDONLY		O_WRONLY		O_RDWR
```

==//socket相关函数==

```c
#include<sys/socket.h>
int socket(int domain,int type,int protocol);
//domain---协议族信息，一般PF_INET // PF_INET6
//type---套接字类型 TCP::SOCK_STREAM // UDP::SOCK_DGRAM; 
//protocol---0号协议
int bind(int socket,struct sockaddr* myaddr,socklen_t addrlen);
int listen(int sockfd,int backlog);			//success-0 error-1
//sockfd---套接字描述符
//backlog---链接请求等待队列长度；最多使backlog个连接请求进入队列；
int accept(int sockfd,struct sockaddr* addr,socklen_t* addrlen);
//受理接入成功时返回创建的套接字描述符(服务器端套接字要做门卫) 失败-1
//addr:保存连接成功的客户端地址信息；
//addr_len：addr长度变量的地址。
//**************************************************************
int connect(int sockfd,struct sockaddr* servaddr,socklen_t addrlen);
//sockfd---文件描述符
//servaddr---保存目标服务器端地址信息
//addrlen---传给servaddr的地址变量字节长度
```

==//IPv4结构体：==

```c
struct sockaddr_in
{
	sa_family_t	sin_family;			//地址族
	uint16_t sin_port;				//16位端口号
	struct in_addr sin_addr;		//32位IP地址
	char sin_zero[8];				//不用
};
struct in_addr
{
	in_addr_t s_addr;				//32位IPv4地址。uint32_t
};

//初始化操作	memset()
#include<string.h>
struct sockaddr_in serv_addr;
memset(&serv_addr,0,sizeof(serv_addr));			//string.h

//bind时注意类型转换
bind(serv_sock,(struct sockaddr*)&serv_addr,sizeof(serv_addr));
```

==//网络字节序==

0x12345678

0x12最高位字节；0x78最低位字节。

大端序先保存高位字节；0x12--->低位地址，从低位地址数据开始传输。

小端序先保存低位字节；0x78--->低位地址

//统一为大端序传输。

```C
#include<arpa/inet.h>
unsigned short htons(unsigned short);
unsigned long htonl(unsigned long);
in_addr_t inet_addr(const char* string);		//字符串类型IP地址转换为网络整数型数据
//成功返回32位大端序整形，失败返回INADDR_NONE。
int inet_aton(const char* string,struct in_addr* addr);	
//类似inet_addr,但是这个函数转换后自动填入结构体。
//调用：
char* addr = "127.232.124.79";
struct sockaddr_in addr_inet;
if(inet_aton(addr,&addr_inet.sin_addr) == -1) perror(...);

char* inet_ntoa(struct in_addr adr);		//与inet_aton相反
```



==网络地址初始化操作==_服务器

```c
#include<string.h>
#include<sys/socket.h>
#include<arpa/inet.h>

struct sockaddr_in addr;
memset(&addr,0,sizeof(addr));				//string.h
char* serv_ip = "211.217.168.13";
char* serv_port = "9190";
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = inet_addr(serv_ip);
addr.sin_port = htons(atoi(serv_port));		//atoi:字符串-->int. 
```

```c
int main(int argc,char* argv[])						//input PORT
{
    struct sockaddr_in serv_addr;
    struct sockaddr_in client_addr;
	char* serv_port = "9190";
	memset(&addr,0,sizeof(addr));
	serv_addr.sin_family = AF_INET;
	serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);		
    //INADDR_ANY自动获取运行服务器端的IP地址。
	serv_addr.sin_port = htons(atoi(serv_port));
    
    //bind --serv_addr;	分配
    //listen
    //accept --client_addr; 保存
}
```

==网络地址初始化操作==_客户端

```c
int main(int argc,char* argv[])						//input IP,PORT
{
	..
	struct sockaddr_in serv_addr;
	memset(&serv_addr,0,sizeof(addr));				//string.h
	serv_addr.sin_family = AF_INET;
	serv_addr.sin_addr.s_addr = inet_addr(argv[1]);
	serv_addr.sin_port = htons(atoi(argv[2]));		//atoi:字符串-->int. 
    
    //connect.. --serv_addr
    ..
}
```



### 函数_UDP 

#### sendto && recvfrom

UDP不会保持连接(不用listen-accept；connect)，每次传输数据都要附加地址信息。

UDP存在数据边界，发多少次要收多少次。

```c++
#include<sys/socket.h>
ssize_t sendto(int sock,void* buff,size_t nbytes,
			   int flags,struct sockaddr* to,socklen_t addrlen);
//sockfd
//buff---保存待传输数据的缓冲地址值
//nbytes---待传输数据长度,字节
//flags---0
//to---存有目标地址信息的结构体变量
//addrlen---to变量长度。
//***sendto函数可以给相应套接字分配IP和PORT。(首次调用sendto，之前未分配)--UDP客户端不需要额外地址分配过程

ssize_t recvfrom(int sock,void* buff,size_t nbytes,
				 int flags,struct sockaddr* from,socklen_t *addrlen);
//sockfd
//buff---保存待传输数据的缓冲地址值
//nbytes---待传输数据长度,字节
//flags---0(可选参数，没有为0)
//from---存有发射端地址信息的结构体变量地址值
//addrlen---from变量长度的变量地址值。				 
```

结合connect,可以将默认的未连接UDP改为已连接UDP，与同一主机长时间通信将提升效率。(P112)

#### C7-断开套接字连接

半关闭 

![image-20230817223006526](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817223006526.png)

```c++
#include<sys/socket.h>
int shutdown(int sock,int howto);							//成功0 失败-1
//howto:	SHUT_RD 断开输入 ||  SHUT_WR 断开输出  ||  SHUT_RDWR 同时断开I/O流
```



### 域名及网络地址

域名<-------DNS-------->网络地址

222.122.195.5 <-------DNS--------> www.naver.com

![image-20230817223840944](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817223840944.png)

#### 域名<-->IP函数

```c++
#include <netdb.h>
//域名---->IP
struct hostent * gethostbyname(const char* hostname);					//传递字符串格式域名获取IP地址,失败返回NULL指针
struct hostent
{
    char* h_name;
    char** h_aliases;
    int h_addrtype;					//地址族信息。IP_V4~AF_INET
    int h_length;					//IP地址长度   V4~4  V6~16
    char** h_addr_list;				//整数形式保存域名对应的IP地址。
}
//IP ---->域名
struct hostent * gethostbyaddr(const char* addr,socklen_t len,int family);
//addr: 含有IP地址信息的in_addr 结构体指针。(char*)sockaddr_in.sin_addr
//len: 字节数  IP_V4~4 V6~16
//family: 地址族信息 IP_V4~AF_INET,V6~AF_INET
```

![image-20230817225139221](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817225139221.png)

### C9 套接字多种可选项

查看套接字信息 getsockopt/setsockopt

IO缓冲信息

```c++
#include<sys/socket.h>
int getsockopt(int sock,int level,int optname,void* optval,socklen_t *optlen);			//成功0
int setsockopt(int sock,int level,int optname,const void* optval,socklen_t optlen);
//sock: 套接字文件描述符
//level: 可选项协议层		P141
//optname: 可选项名
//optval: 保存查看结果缓冲地址
//optlen: 向optval传递的可选项信息字节

//----------IO缓冲-------------
//----SO_SNDBUF,SO_RCVBUF-----
//--输入输出缓冲大小相关可选项----
//get_buf
int snd_buf;
socklen_t len = sizeof(snd_buf);
state = getsockopt(sock,SOL_SOCKET,SO_SNDBUF,(void*)&snd_buf,&len);
//set_buf
int snd_buf=1024*3;
state = setsockopt(sock,SOL_SOCKET,SO_SNDBUF,(void*)&snd_buf,sizeof(snd_buf));

```

#### 地址分配错误 Time_Wait

![image-20230818104804296](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230818104804296.png)

先发送FIN消息的主机会经过Time-wait状态。套接字处于Time-wait状态中时端口处于占用状态：

对于服务器，处于Time-wait状态占用了端口，从而导致bind失败。

对于客户端，每次运行程序都会重新分配端口号，不需要关注Time-wait状态。



==Time-wait的作用是防止FIN的应答ACK传递失败。==

-->地址再分配

```
getsockopt(serv_sock,SOL_SOCKET,SO_REUSEADDR,(void*)&option,optlen);
//允许端口复用
int on = 1;
if(setsockopt(serv_sock,SOL_SOCKET,SO_REUSEADDR,&on,sizeof(on)))
{
	perror("setsockopt");
	return -1;
}

```













### C10 进程及应用

```shell
//后台管理
root@my_linux:/tcpip# ./zombie &
root@my_linux:/tcpip# ps au			//这样无需打开新的终端了，显示所有进程信息。
```

#### wait && waitpid

调用wait函数时，若没有已经终止的子进程，则程序将==阻塞==直到子进程终止。

```c
#include <sys/wait.h>
pid_t wait(int* statloc);			//成功返回终止的子进程ID。失败-1。

//调用wait时若已有子进程终止。那么子进程终止时传递的返回值(exit(参数),main-return)将保存到wait参数所指内存空间。
//WIFEXITED: 子进程正常终止返回true.
//WEXITSTATUS: 返回子进程返回值。
wait(&status);
if(WIFEXITED(status))
{
	puts("Normal...");
	printf("Child return num:%d",WEXITSTATUS(status));
}
```

waitpid--不阻塞

```c
#include <sys/wait.h>
pid_t waitpid(pid_t pid,int* statloc,int options);			//成功返回终止的子进程ID或0。失败-1。
//pid 目标子进程id。 传递-1表示任意子进程
//options 传递常量WNOHANG。没有终止的子进程不进入阻塞状态，而是返回0退出函数。

//父进程收尸代码
int status;
while(!waitpid(-1,&status,WNOHANG))
{
	sleep(3);
	puts("sleep 3 sec.");
}
if(WIFEXITED(status))
	printf("Child return num:%d",WEXITSTATUS(status));
```

#### 信号注册--signal

信号注册函数signal。(先注册，真发生特殊情况再调用。)

```c
#include <signal.h>
void* signal(int signo, (void *)func(int)  ) (int);
//signo:特殊情况常数： SIGALRM--到了调用alarm函数注册时间	SIGINT:输入CTRL+C		SIGCHLD:子进程终止
//(void *)func(int): 要调用的函数。
//返回函数指针，参数int类型。
//-------------示例：
//输入CTRL+C时调用keycontrol函数。
signal(SIGINT,keycontrol);
```

alarm函数

x时间后返回SIGNALRM。传0取消预约。

```c
#include<unistd.h>
unsigned int alarm(unsigned int seconds);
```

//更好的信号注册方式 sigaction

```c
struct sigaction act
act.sa_handler = read_childproc;		//注册函数read...
sigemptyset(&act.sa_mask);
act.sa_flags = 0;						//初始化
state = sigaction(SIGCHLD,&act,0);
```





### C12 I/O复用

每个学生对应一个老师，新来一个学生就新添一个老师------多进程服务器端

多个学生，一个老师。学生提问先举手，老师再回答------I/O复用服务器端------确认举手套接字(收到数据)、通过举手套接字接收数据。

#### select

将多个文件描述符集中到一起统一监视。有以下==事件==:

1.是否存在套接字接收数据?	(有人举手？)

2.哪些套接字无需阻塞传输数据?

3.哪些异常套接字？

**调用方法与顺序**

1.设置文件描述符、监视范围、设置超时

2.调用select函数

3.查看调用结果

```c
#include<sys/select.h>
#include<sys/time.h>
//1.设置文件描述符。集中要监视的文件描述符到fd_set数组。数组为1表示该位对应的文件描述符是监视对象。
//操作fd_set通过宏完成。
FD_ZERO(fd_set* fdset): 所有位初始化为0.
FD_SET(int fd,fd_set* fdset):从参数fdset指向的变量中注册文件描述符fd的信息。
FD_CLR(int fd,fd_set* fdset):从参数fdset指向的变量中去除文件描述符fd的信息。
FD_ISSET(int fd,fd_set* fdset):如果fdset指向的变量中包括文件描述符fd信息，返回'真'
    
//select---设置监视范围及超时
int select(int maxfd,fd_set* readset,fd_set* writeset,fd_set* exceptset,const struct timeval* timeout);	
//maxfd:监视对象文件描述符数量
//readset:将事件类型1的文件描述符注册到fd_set。并且传递其地址值；		--监视有人举手？
//writeset:将事件类型2的文件描述符注册到fd_set。并且传递其地址值；		--监视无需阻塞？
//exceptset:将事件类型3的文件描述符注册到fd_set。并且传递其地址值；	
//timeout:	超时信息，防止无限阻塞。不设置超时填入NULL
//------------返回值：超时0,因为发生关注事件返回时,返回对应事件文件描述符数量。
```

```c
//3.查看调用结果。select返回>0的整数说明对应数量的文件描述符发生了变化。
//文件描述符发生了变化:监视的文件描述符发生了相应的监视事件。~readset中存在需要读数据的描述符~文件描述符发生变化。
```

#### 多种I/O函数

##### send&recv

```c
#include <sys/socket.h>
ssize_t send(int sockfd,const void* buf,size_t nbytes,int flags);		//成功返回发送的字节数，失败-1.
//sockfd:套接字文件描述符
//buf:保存待传输数据的缓冲地址
//nbytes:待传输字节数
//flags:传输数据时指定的可选项信息
ssize_t recv(int sockfd,const void* buf,size_t nbytes,int flags);
//sockfd:套接字文件描述符
//buf:保存接收数据的缓冲地址
//nbytes:可接受最大字节数
//flags:接收数据时指定的可选项信息
//-----------flags---可选项信息
MSG_OOB:发送紧急消息。(敦促~不影响TCP保持传输顺序的特性)
P212
```

##### writev&readv

对数据整合以及发送。~多个缓冲区发送，接收 

```c
#include<sys/uio.h>
ssize_t writev(int filedes,const struct iovec* iov,int iovcnt);			//成功返回发送的字节数
ssize_t readv(int filedes,const struct iovec* iov,int iovcnt);			//成功返回发送的字节数
//filedes:套接字、文件、标准输出描述符
//结构体iovec地址；包含待发送数据位置(数据保存位置)和大小。
//iovcnt:第二个参数数组长度
struct iovec vec[2];
char buf1[] = "ABCDEFG";
char buf2[] = "1234567";
vec[0].iov_base = buf1;
vec[0].iov_len = 3;
vec[1].iov_base = buf2;
vec[1].iov_len = 4;
int str_len;
str_len = writev(1,vec,2);												//1:向控制台输出数据。
```







### C13 多播与广播

多播基于UDP。

多播组：D类IP地址(224.0.0.0 ~ 239.255.255.255)

为了传递多播数据包，必须设置TTL。结果一个路由器TTL就减1。

```c
//设置TTL--设置套接字可选项---IPPROTO_IP协议层--IP_MULTICAST_TTL可选项。
int send_sock;
int time_live = 64;
...
send_sock = socket(PF_INET,SOCK_DGRAM,0);
setsockopt(send_sock,IPPROTO_IP,IP_MULTICAST_TTL,(void*)&time_live，sizeof(time_live));

//加入多播组----IPPROTO_IP协议层---IP_ADD_MEMBERSHIP
int send_sock;
struct ip_mreq join_adr;
...
send_sock = socket(PF_INET,SOCK_DGRAM,0);
....
join_adr.imr_multiaddr.s_addr = "多播组地址信息";
join_adr.imr_interface.s_addr = "加入多播组的主机地址信息";
setsockopt(recv_sock,IPPROTO_IP,IP_ADD_MEMBERSHIP,(void*)&join_adr,sizeof(join_adr));

//ip_mreq结构体
struct ip_mreq
{
    struct in_addr imr_multiaddr;	//组IP
    struct in_addr imr_interface;   //加入该组的套接字主机地址，INADDR_ANY。
}
```



广播

广播类似多播，但只能向一个网络中的主机传输数据。P237。





### C15 使用标准I/O函数

#### fdopen&&fileno

文件描述符<--->FILE指针

```c
#include<stdio.h>
FILE * fdopen(int fildes,const char* mode);
//fildes: 需要转换的文件描述符
//mode:	需要创建的结构体指针模式信息

int fileno(FILE * stream);			

//使用fdopen后返回FILE指针，可以直接用fputs。
fp = fdopen(fd,"w");
fputs("Network C programming \n",fp);
```





### C17 Linux-epoll

epoll优点:

1.不用编写以监视状态变化为目的的针对所有文件描述符的循环语句。

2.epoll_wait不需要每次传递监视对象信息。

```c
#include<sys/epoll.h>
//epoll_creat:	创建保存epoll文件描述符的空间
int epoll_creat(int size);											//成功时返回epoll文件描述符，失败-1.
//epoll_ctl:		向空间注册并注销文件描述符
int epoll_ctl(int epfd,int op,int fd,struct epoll_event * event);	//成功0，失败-1.
//---epfd: 注册监视对象的epoll例程的文件描述符。
//---op: 指定监视对象的添加、删除或更改。EPOLL_CTL_ADD   EPOLL_CTL_DEL   EPOLL_CTL_MOD
//---fd: 需要注册的监视对象文件描述符。
//---event: 监视对象的事件类型。
struct epoll_event
{
	__uint32_t events;
	epoll_data_t data;
}
//声明足够大的epoll_event结构体数组后，传递给epoll_wait函数，发生变化的文件描述符信息将被填入该数组。
typedef union epoll_data
{
	void* ptr;
	int fd;
	__uint32_t u32;
	__uint64_t u64;
}epoll_data_t;

epoll_event中的成员events中可以保存的常量及其所指的事件类型：
EPOLLIN：需要读取数据的情况
EPOLLOUT：输出缓冲为空，可以立即发送数据的情况
EPOLLPRI：收到OOB数据的情况
EPOLLDHUP：断开连接或半关闭的情况，这在边缘触发方式下非常有用
EPOLLERR：发生错误的情况
EPOLLET：以边缘触发的方式得到事件通知
EPOLLONESHOT：发生一次事件后，相应文件描述符不再收到事件通知。因此需要向epoll_ctl函数的第二个参数传递EPOLL_CTL_MOD，再次设置事件

e.g. epoll_ctl(A,EPOLL_CTL_ADD,B,C)---例程A中注册文件描述符B，主要目的是监视参数C的事件。
e.g. epoll_ctl(A,EPOLL_CTL_DEL,B,NULL)


//epoll_wait:		等待文件描述符发生变化
int epoll_wait(int epfd,struct epoll_event * events,int maxevents,int time_out);
//---epfd: 表示事件发生监视范围的epoll例程的文件描述符。
//---events: 保存发生事件的文件描述符集合的结构体地址。
//---maxevents: 第二个参数可以保存的最大时间数。
//---event: 单位1/1000秒的等待时间。-1时一直等待。
```

```c
//echo_epollserv.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#include<sys/epoll.h>

#define BUF_SIZE 100
#define EPOLL_SIZE 50

int main(int argc,char * argv[])
{
	int serv_sock,clnt_sock;
    struct sockaddr_in serv_adr,clnt_adr;
    socklen_t adr_sz;
    int str_len,i;
    char buf[BUF_SIZE];
    
    struct epoll_event *ep_events;
    struct epoll_event event;
    int epfd,event_cnt;
    
    if(argc!=2)
    {
        printf("Usage: %s<port>\n",argv[0]);
        exit(1);
    }
    serv_sock = socket(PF_INET,SOCK_STREAM,0);
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));
    
    if(bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr))==-1)
        perror("bind");
    if(listen(serv_sock,5)==-1)
        perror("listen");
    
    //epoll
    epfd = epoll_create(EPOLL_SIZE);
    ep_events = malloc(sizeof(struct epoll_event)* EPOLL_SIZE);
    
    event.events = EPOLLIN;										//检测事件类型:有数据输入。
    event.data.fd = serv_sock;
    epoll_ctl(epfd,EPOLL_CTL_ADD,serv_sock,&event);
    
    while(1)
    {
		event_cnt = epoll_wait(epfd,ep_events,EPOLL_SIZE,-1);
        if(event_cnt == -1)
        {
			puts("epoll_wait() error.");
            break;
        }
        
        for(i=0;i<event_cnt;i++)
        {
			if(ep_events[i].data.fd == serv_sock)				//当前文件描述符是服务端套接字；找客户端文件描述符。
            {
				adr_sz = sizeof(clnt_adr);
                clnt_sock = accept(serv_sock,(struct sockaddr*)&clnt_adr,&adr_sz);
                event.events = EPOLLIN;
                event.data.fd = clnt_sock;
                epoll_ctl(epfd,EPOLL_CTL_ADD,clnt_sock,&event);
                printf("connect client:%d\n",clnt_sock);
            }
            else
            {
				str_len = read(ep_events[i].data.fd,buf,BUF_SIZE);
            	if(str_len ==0)
                {
					epoll_ctl(epfd,EPOLL_CTL_DEL,ep_events[i].data.fd,NULL);
                    close(ep_events[i].data.fd);
                    printf("Closed client:%d\n",ep_events[i].data.fd);
                }
                else
                {
					write(ep_events[i].data.fd,buf,str_len);
                }
            }
        }
    }
        close(serv_sock);
        close(epfd);
        return 0;
}
```









### C18 多线程服务器端实现

#### 线程的创建和运行

```c
#include<pthread.h>
int pthread_create(pthread_t* restrict thread,
const pthread_attr_t* restrict attr,void*(*start_routine)(void*),void* restrict arg);
//成功0,失败其他值
//----thread	保存新创建线程ID的变量地址值
//----attr		传递线程属性。传递NULL时，创建默认属性线程。
//----start_routine	相当于线程main函数，在单独执行流中执行的函数地址。
//----arg		通过第三个参数传递调用函数时，包含传递参数信息的变量地址值。(thread_param)
```



```c
#include<stdio.h>
#include<pthread.h>
#include<unistd.h>
void* thread_main(void* arg);

int main(int argc,char* argv[])
{
    pthread_t t_id;
    int thread_param = 5;
    
    if(pthread_create(&t_id,NULL,thread_main,(void*)&thread_param)!=0)
    {
        puts("pthread_create() error");
        return -1;
    }
    sleep(10);
    puts("end of main");
    return 0;
}

void* thread_main(void* arg)
{
    int i;
    int cnt = *((int*)arg);
    for(i=0;i<cnt;i++)
    {
		sleep(1);
        puts("running thread");
    }
	return NULL;
}
```

`gcc thread1.c  -o tr1 -lpthread`		连接线程库。

#### 函数控制线程执行流

```c
#include<pthread.h>
int pthread_join(pthread_t thread,void** status);		//成功0,失败其他
//thread 线程ID。该参数ID的线程终止后才会从该函数返回。
//status 保存线程的main函数返回值的指针变量地址值。
```

```c
//thread2.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<pthread.h>
#include<unistd.h>

void* thread_main(void *arg);

int main(int argc,char* argv[])
{
	pthread_t t_id;
	int thread_param = 5;
	void* thr_ret;
	
    //创建线程
	if(pthread_create(&t_id,NULL,thread_main,(void*)&thread_param)!=0)
	{
		puts("pthread_create() error");
		return -1;
	}
	//等待线程终止。
	if(pthread_join(t_id,&thr_ret)!=0)				//thr_ret--->thread_main返回值。 
	{
		puts("pthread_join() error");
		return -1;		
	}
	
	printf("Thread return message: %s\n",(char*)thr_ret);
	free(thr_ret);
	return 0;
}

void* thread_main(void *arg)
{
	int i;
    int cnt = *((int*)arg);
    char* msg = (char*)malloc(sizeof(char)*50);
    strcpy(msg,"Hello,I'am thread~\n");
    
    for(i=0;i<cnt;i++)
    {
        sleep(1);
        puts("running thread");
    }
    return (void*)msg;
}
```

#### 同时运行多个线程

多个线程同时调用函数时可能产生问题----临界区。

针对上述问题，函数可以分为线程安全函数和非线程安全函数。

为了调用线程安全函数，在编译线程相关代码时需要添加宏:

`gcc -D_REENTRANT mythread.c -o mthread -lpthread`



#### 工作线程模型

#示例:两个线程，一个计算1-5的和，另一个计算6-10的和。main函数负责输出运算结果。

![image-20230822103640770](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230822103640770.png)

```c
#include<stdio.h>
#include<pthread.h>
void * thread_summation(void* arg);
int sum = 0;

int main(int argc,char* argv[])
{
	pthread_t id_t1,id_t2;
    int range1[] = {1,5};
    int range2[] = {6,10};
    //创建线程
    pthread_create(&id_t1,NULL,thread_summation,(void*)range1);
    pthread_create(&id_t2,NULL,thread_summation,(void*)range2);
   	//等待
    pthread_join(id_t1,NULL);
    pthread_join(id_t2,NULL);
    printf("result:%d \n",sum);
    return 0;
}
  
void * thread_summation(void* arg)
{
    int start = ((int*)arg)[0];
    int end = ((int*)arg)[1];
    while(start<=end)
    {
		sum+= start;
        start++;
    }
    return NULL;
}
```

上面的示例存在临界区相关问题。下面增加发生临界区错误可能性：

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<pthread.h>
#define NUM_THREAD 100
void* thread_inc(void* arg);
void* thread_des(void* arg);
long long num = 0;

int main(int argc,char* argv[])
{
	pthread_t thread_id[NUM_THREAD];
    int i;
    printf("sizeof long long:%d \n",(int)sizeof(long long));
    for(i=0;i<NUM_THREAD;i++)
    {
		if(i%2)
            pthread_create(&(thread_id[i]),NULL,thread_inc,NULL);
        else
            pthread_create(&(thread_id[i]),NULL,thread_des,NULL);
    }
    for(i=0;i<NUM_THREAD;i++)
    {
		pthread_join(thread_id[i],NULL);
    }
    printf("result:%lld\n",num);
}
void* thread_inc(void* arg)
{
	int i;
    for(i=0;i<50000000;i++)
        num+=1;						//临界区
    return NULL;
}
void* thread_des(void* arg)
{
	int i;
    for(i=0;i<50000000;i++)
        num-=1;						//临界区
    return NULL;
}
```

结果发现上面的代码运行结果result != 0。

原因：多线程访问同一个内存--->==不同步==。



![image-20230822112507424](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230822112507424.png)





#### 线程同步

1.解决同时访问同一内存可能带来的问题；

2.指定访问同一内存空间线程执行顺序的情况。 

##### 1.互斥量(上锁)

互斥量的创建及销毁

```c
#include<pthread.h>
int pthread_mutex_init(pthread_mutex_t * mutex,const pthread_mutexattr_t* attr);
int pthread_mutex_destroy(pthread_mutex_t * mutex);									//成功0,失败其他
//mutex：创建时:传递保存互斥量的变量地址；销毁时:传递需要销毁的互斥量变量地址。
//attr:即将创建的互斥量属性。没有特别属性为NULL。

//创建pthread_mutex_t型变量
pthread_mutex_t mutex;
```

利用互斥量锁住或释放临界区时使用的函数:

```c
#include<pthread.h>
int pthread_mutex_lock(pthread_mutex_t * mutex);	//若发现其他线程已经进入临界区，当前线程将一直处于阻塞状态。
//临界区开始
...
//临界区结束
int pthread_mutex_unlock(pthread_mutex_t * mutex);	//线程退出临界区而不解锁---死锁。								
//成功0,失败其他
```

```c
//mutex.c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<pthread.h>
#define NUM_THREAD 100
void * thread_inc(void* arg);
void * thread_des(void* arg);
long long num = 0;
pthread_mutex_t mutex;

int main(int argc,char* argv[])
{
	pthread_t thread_id[NUM_THREAD];
    int i;
    //创建锁
    pthread_mutex_init(&mutex,NULL);
    for(i=0;i<NUM_THREAD;i++)
    {
		if(i%2)
            pthread_create(&(thread_id[i]),NULL,thread_inc,NULL);
        else
        	pthread_create(&(thread_id[i]),NULL,thread_des,NULL);
    }
    //线程控制(等待线程终止)
    for(i=0;i<NUM_THREAD;i++)
        pthread_join(thread_id[i],NULL);
    
    printf("result:%lld \n",num);
    //销毁锁
    pthread_mutex_destroy(&mutex);
    return 0;
}
void* thread_inc(void* arg)
{
	int i;
    //上锁
    pthread_mutex_lock(&mutex);
    for(i=0;i<50000000;i++)
        num+=1;
    pthread_mutex_unlock(&mutex);
    return NULL;
}
void* thread_des(void* arg)
{
  	int i;

    for(i=0;i<50000000;i++)
    {
       	pthread_mutex_lock(&mutex);						//多调用了49,999,999次lock/unclock。执行时间很长。
        num-=1;
    	pthread_mutex_unlock(&mutex);       
    }

    return NULL;  
}
```



##### 2.信号量

P305



#### 线程销毁

1.pthread_join					等待线程终止并引导线程销毁(终止前阻塞)

2.pthread_detach			  引导线程销毁，不会阻塞。

```c
#include<pthread.h>
int pthread_detach(pthread_t thread);						//成功0,失败其他。
```



#### 多线程公共聊天室

客户端发送给服务器端的数据回声送回客户端显示。

##### 多线程并发服务器端

注意访问(==修改==)全局变量的代码将构成临界区。



多进程并发服务器端
全局变量:客户端套接字集合、客户端套接字数量、互斥锁。 
Main():创建套接字；
当有客户端接入时，添加客户端套接字集合、调用handle_clnt函数。
Handle_clnt():删除客户端套接字集合的客户端套接字。回声发送接收到的数据。Send_msg.
Send_msg():发送数据。访问套接字集合中的具体套接字。

```c
//chat_server.c
#include<stdio.h>
#include <stdlib.h>
#include<unistd.h>
#include<sys/socket.h>
#include<pthread.h>
#include<arpa/inet.h>
#include<string.h>

#define BUF_SIZE 100
#define MAX_CLNT 256
void* handle_clnt(void* arg);
void send_msg(char* msg,int len);

int clnt_cnt = 0;
int clnt_socks[MAX_CLNT];
pthread_mutex_t mutx;

int main(int argc,char* argv[])
{
    int serv_sock,clnt_sock;
    struct sockaddr_in serv_adr,clnt_adr;
    int clnt_adr_sz;
    pthread_t t_id;
    if(argc != 2)
    {
        printf("Usage:%s <port>",argv[0]);
        exit(1);
    }
    
    pthread_mutex_init(&mutx,NULL);
    serv_sock = socket(PF_INET,SOCK_STREAM,0);
    
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));
    
    if(bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr)) == -1)
        perror("bind");
    if(listen(serv_sock,5)==-1)
        perror("listen");
    
    while(1)
    {
        clnt_adr_sz = sizeof(clnt_adr);
        clnt_sock = accept(serv_sock,(struct sockaddr*)&clnt_adr,&clnt_adr_sz);
        
        pthread_mutex_lock(&mutx);
        clnt_socks[clnt_cnt++] = clnt_sock;									//访问全局变量clnt_socks
        pthread_mutex_unlock(&mutx);
        
        pthread_create(&t_id,NULL,handle_clnt,(void*)&clnt_sock);
        pthread_detach(t_id);
        printf("Connected client IP:%s \n",inet_ntoa(clnt_adr.sin_addr));        
    }
    close(serv_sock);
    return 0;
}

void* handle_clnt(void* arg)												//维护clnt表
{
	int clnt_sock = *((int*)arg);
    int str_len=0,i;
    char msg[BUF_SIZE];
    
    while((str_len = read(clnt_sock,msg,sizeof(msg))) != 0)
        send_msg(msg,str_len);
    pthread_mutex_lock(&mutx);
    for(i=0;i<clnt_cnt;i++)													//访问全局变量，删除套接字。
    {
        if(clnt_sock == clnt_socks[i])
        {
            while(i++ < clnt_cnt - 1)
				clnt_socks[i] = clnt_socks[i+1];
            break;
		}
    }
    clnt_cnt--;
    pthread_mutex_unlock(&mutx);
    close(clnt_sock);
    return NULL;
}

void send_msg(char* msg,int len)											//发消息。
{
	int i;
    pthread_mutex_lock(&mutx);
    for(i=0;i<clnt_cnt;i++)													//访问全局变量
        write(clnt_socks[i],msg,len);
    pthread_mutex_unlock(&mutx);
}
```



##### 多线程并发客户端

分为读线程和写线程。不涉及临界区域，没有锁。

```
sprintf(字符串地址,"%s %s",串1，串2);	打印到字符串中。
```

```c
//chat_client.c
#include<stdio.h>
#include<unistd.h>
#include<string.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#include<pthread.h>
#include<stdlib.h>

#define BUF_SIZE 100
#define NAME_SIZE 20

void* send_msg(void* arg);
void* recv_msg(void* arg);

char name[NAME_SIZE] = "[DEFAULT]";
char msg[BUF_SIZE];

int main(int argc,char* argv[])
{
	int sock;
    struct sockaddr_in serv_addr;
    pthread_t snd_thread,rcv_thread;
    void* thread_return;
    if(argc!=4)
    {
		printf("Usage:%s <IP><Port><Name>\n",argv[0]);        
        exit(1);
    }
    
	sprintf(name,"[%s]",argv[3]);							//将argv[3]打印到字符串中。
    
    sock = socket(PF_INET,SOCK_STREAM,0);
    
    memset(&serv_addr,0,sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = inet_addr(argv[1]);
    serv_addr.sin_port = htons(atoi(argv[2]));
    
    if(connect(sock,(struct sockaddr*)&serv_addr,sizeof(serv_addr))==-1)
        perror("connect");
    //读线程和写线程。
    pthread_create(&snd_thread,NULL,send_msg,(void*)&sock);
    pthread_create(&rcv_thread,NULL,recv_msg,(void*)&sock);
    pthread_join(snd_thread,&thread_return);
    pthread_join(rcv_thread,&thread_return);
    close(sock);
    return 0;
}

void* send_msg(void* arg)
{
    int sock=*((int*)arg);
    char name_msg[NAME_SIZE + BUF_SIZE];
    while(1)
    {
		fgets(msg,BUF_SIZE,stdin);
        if(!strcmp(msg,"q\n")||!strcmp(msg,"Q\n"))
        {
			close(sock);
            exit(0);
        }
    	sprintf(name_msg,"%s %s",name,msg);					//拼接字符串存入
        write(sock,name_msg,strlen(name_msg));
    }
    return NULL;
}

void* recv_msg(void* arg)
{
    int sock = *((int*)arg);
    char name_msg[NAME_SIZE + BUF_SIZE];
    int str_len;
    while(1)
    {
        str_len = read(sock,name_msg,NAME_SIZE+BUF_SIZE - 1);
        if(str_len == -1)
            return (void*)-1;
        name_msg[str_len] = 0;								//数组最后一位填0。
        fputs(name_msg,stdout);
    }
    return NULL;
}
```















### C24 Web服务器端(HTTP)

web服务器端:基于HTTP协议，将网页对应文件传输给客户端的服务器端?？ 不理解

HTTP协议:以超文本传输为目的的应用层协议。基于TCP/IP。



#### HTTP

##### 无状态的stateless协议

服务器端不会维持客户端状态。

![image-20230823103626253](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230823103626253.png)





==P.S.==

为了弥补HTTP无法保持连接的缺点:web编程通常使用Cookie和Session技术。(保持状态)



##### 请求消息 request

**请求行：**请求信息；典型请求方式:GET:请求数据；POST:传输数据。

​				图示:请求(GET)index.html文件，希望以1.1版本的HTTP协议进行通信。

**消息头**：包含发送请求的(或将要接收响应信息的)浏览器信息、用户认证信息等HTTP消息的附加消息。

**消息体：** 客户端向服务端传输的数据，请求行需要为POST。

![image-20230823104923534](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230823104923534.png)

##### 响应消息 response

**状态行:** 关于客户端请求的处理结果。

​			  200 OK:成功处理了请求！

​			  404 Not Found:请求的文件不存在!

​			  404 Bad Request:请求方式错误，请检查。

**消息头:** 传输数据类型和长度信息。

​			  图示:服务器端名SimpleWebServer，传输数据类型为text/html。长度不超过2048字节。

**消息体:** 文件数据。

![image-20230823105800361](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230823105800361.png)





##### 基于Linux的多线程Web服务器端

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#include<pthread.h>
#include<string.h>

#define BUF_SIZE 1024
#define SMALL_BUF 100

void* request_handler(void* arg);
void send_data(FILE* fp,char* ct,char* file_name);
char* content_type(char* file);
void send_error(FILE* fp);

int main(int argc,char* argv[])
{
	//声明套接字、地址结构体、缓冲区、线程
	int serv_sock,clnt_sock;
	struct sockaddr_in serv_adr,clnt_adr;
	int clnt_adr_size;
    char buf[BUF_SIZE];
    pthread_t t_id;
    
    if(argc != 2)
    {
		printf("Usage: %s <Port> \n",argv[0]);
    }
	//初始化
    serv_sock = socket(PF_INET,SOCK_STREAM,0);
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));
	//bind
    if(bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr))==-1)
    {
		perror("bind");
        return -1;
    }
	//listen
    if(listen(serv_sock,20)==-1)
    {
        perror("listen");
        return -1;
    }
	//accept,创建线程调用request处理函数响应客户端请求。
    while(1)
    {
		clnt_adr_size = sizeof(clnt_adr);
        clnt_sock = accept(serv_sock,(struct sockaddr*)&clnt_adr,&clnt_adr_size);
        printf("Connection Request: %s:%d\n",inet_ntoa(clnt_adr.sin_addr),ntohs(clnt_adr.sin_port));
    	pthread_create(&t_id,NULL,request_handler,&clnt_sock);
    	pthread_detach(t_id);
    }
    close(serv_sock);
    return 0;
}

//request处理函数,收到GET请求后会发送响应消息(调用send_data)

void* request_handler(void* arg)
{
    int clnt_sock = *((int*)arg);
    char req_line[SMALL_BUF];
    
    FILE* clnt_read;
    FILE* clnt_write;
    
    char method[10];
    char ct[15];
    char file_name[30];
    
    clnt_read = fdopen(clnt_sock,"r");
    clnt_write = fdopen(dup(clnt_sock),"w");					//dup,fdopen.
    fgets(req_line,SMALL_BUF,clnt_read);
    if(strstr(req_line,"HTTP/") == NULL)						//strstr,看req_line中有没有http/.没有则send_error.
    {
        send_error(clnt_write);
        fclose(clnt_read);
        fclose(clnt_write);
        return;
    }
    
    strcpy(method,strtok(req_line,"/"));						//char *strtok(char *str, const char *delim)；用delim分割字符串。
    strcpy(file_name,strtok(NULL,"/"));							//file_name存放文件类型字符串
    strcpy(ct,content_type(file_name));							//调用content_type转换文件类型，存入ct.
    
    if(strcmp(method,"GET")!=0)									//只考虑GET。不考虑POST。没收到GET，就send_error.
    {
		send_error(clnt_write);
        fclose(clnt_read);
        fclose(clnt_write);
        return;
    }
    fclose(clnt_read);											//收到GET，向clnt_write写数据。
    send_data(clnt_write,ct,file_name);
}

void send_data(FILE* fp,char* ct,char* file_name)
{
	char protocol[] = "HTTP/1.0 200 OK\r\n";
    char server[] = "Server:Linux web server \r\n";
    char cnt_len[] = "Content-length:2048 \r\n";
    
    char cnt_type[SMALL_BUF];
    char buf[BUF_SIZE];
    
    FILE* send_file;
	
    sprintf(cnt_type,"Content-type:%s\r\n\r\n",ct);
	send_file = fopen(file_name,"r");
    if(send_file == NULL)
    {
		send_error(fp);
        return;
    }
    //传输头信息
    fputs(protocol,fp);
    fputs(server,fp);
    fputs(cnt_len,fp);
    fputs(cnt_type,fp);
	//传输请求数据
    while(fgets(buf,BUF_SIZE,send_file)!=NULL)
    {
        fputs(buf,fp);
        fflush(fp);
    }
    fflush(fp);
    fclose(fp);
}

char* content_type(char* file)
{
    char extension[SMALL_BUF];
    char file_name[SMALL_BUF];
    strcpy(file_name,file);
    strtok(file_name,".");
    strcpy(extension,strtok(NULL,"."));
    if(!strcmp(extension,"html") || !strcmp(extension,"htm"))
        return "text/html";
    else
        return "text/plain";
}

void send_error(FILE* fp)
{
    char protocol[] = "HTTP/1.0 400 Bad Request\r\n";
    char server[] = "Server:Linux Web Server \r\n";
    char cnt_len[] = "Content-length:2048 \r\n";
    char cnt_type[] = "Content-type:text/html\r\n\r\n";
    char content[] = "<html><head><title>NETWORK</title></head>"
        			 "<body><font size=+5><br>发生错误!查看请求文件名和请求方式!"
        			 "</font></body></html>";
    fputs(protocol,fp);
    fputs(server,fp);
    fputs(cnt_len,fp);
    fputs(cnt_type,fp);
    fflush(fp);
}
```

第一次调用strtok()，传入的参数str是要被分割的字符串{aaa - bbb -ccc}，而成功后返回的是第一个子字符串{aaa}；

而第二次调用strtok的时候，传入的参数应该为NULL，使得该函数默认使用上一次未分割完的字符串继续分割 ，就从上一次分割的位置{aaa-}作为本次分割的起始位置，直到分割结束。

```c
#include <stdio.h>
FILE *fopen(const char *filename, const char *mode);		//fopen函数返回新打开文件的文件指针；如果此文件不能打开，则返回NULL指针
filename: 要打开的文件名字符串
mode: 访问文件的模式
```

![image-20230823115909608](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230823115909608.png)





### Web服务器项目

浏览器作为客户端；



![image-20230824102846387](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230824102846387.png)





#### 概要:

main启动服务器模块。服务器模块主要负责接收请求开线程。

线程模块借助通信模块，接收客户端发送的HTTP请求。

线程模块借助HTTP模块，解析HTTP请求。

线程模块借助资源模块，判断文件。

线程模块借助HTTP模块，构造响应。

线程模块借助通信模块，发送响应。



#### 详细设计

http:					http.h					http.c

通信模块:		socket.h				socket.c

资源模块:	resource.h			resource.c

​							mime.h

信号模块:		signals.h			   signals.c

线程模块:			client.h				client.c

服务器模块:		server.h			 server.c

主模块:											 main.c



#### http模块:

解析HTTP请求；

构造HTTP响应。

```c
//http.h
#ifndef __HTTP_H__
#define __HTTP_H__

#include<limits.h>
#include<sys/types.h>			//表示最大路径的宏


//--解析得到的数据存在结构体中；
typedef struct httpRequest
{
    char method[32];			//get/post
    char path[PATH_MAX + 1];	//路径
    char protocol[32];			//协议
    char connection[32];		//链接状态
}HTTP_REQUEST;

//--构造响应头时需要的结构体
typedef struct httpRespond
{
    char protocol[32];			//协议版本
    int status;					//状态码
    char desc[128];				//状态描述
    char type[64];				//类型 imag...
    off_t length;				//off_t表示文件大小
	char connection[32];
}HTTP_RESPONSE;

//************************************http请求解析

//--输入参数http请求。输入req表示请求字符串；输出型参数http请求结构体。
int parseRequest(const char* req,HTTP_REQUEST* hreq);			

//************************************http响应构造

//--输入结构体地址(相关信息存储区)。输出型参数为char* 指针
inr constructHead(const HTTP_RESPONSE* hres,char* head);

#endif
```

==请求==

![image-20230824110551311](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230824110551311.png)

==响应==

![image-20230824113403928](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230824113403928.png)

```c
//http.c
#include<unistd.h>
#include<sys/syscall.h>					//
#include<stdio.h>
#define __USE_GNU						// 为了用strcasestr，定义宏，string.h才有这个函数。
#include<string.h>
#include<time.h>						//
#include"http.h"

//************************************http请求解析
int parseRequest(const char* req,HTTP_REQUEST* hreq)
{
    //把上图的请求信息*req存入结构体。//strtok,sscanf(从字符串中读取数据存入变量。各串空格间隔)
    sscanf(req,"%s%s%s",hreq->method,hreq->path,hreq->protocol);
    //strstr(在大串中找小串，返回小串第一个字符地址。) strcasestr不区分大小写。
    char* connection = strcasestr(req,"connection");
    if(connection)
    {
        //********%*s 虚读。	connection定位的是connection第一个字符c，我们想要的是keep-alive。
        sscanf(connection,"%*s%s",hreq->connection);
    }
    //syscall(SYS_gettid),见后面注释。获取tid。
    printf("%d.%ld > [%s][%s][%s][%s]\n",getpid(),syscall(SYS_gettid),hreq->method,hreq->path,hreq->protocol,hreq->connection);
    
    //判断请求方法。
    if(strcasecmp(hreq->method,"get"))								//一样返回0，不一样非0
    {
        printf("%d.%ld > 无效方法\n",getpid(),syscall(SYS_gettid));
        return -1;
    }
    
    //判断协议版本。
    if(strcasecmp(hreq->protocol,"http/1.0") && strcasecmp(hreq->protocol,"http/1.1"))
    {
        printf("%d.%ld > 无效协议\n",getpid(),syscall(SYS_gettid));
        return -1;
    }    
    return 0;
}

//************************************http响应构造
int constructHead(const HTTP_RESPONSE* hres,char* head)
{
    char dateTime[32];
    time_t now = time(NULL);
    //strftime  man 3 strftime
    //time_t 转换为 图示时间。gmtime表示将time_t时间转化为结构体。(strftime函数需要结构体)
    strftime(dateTime,sizeof(dateTime),"%a %d %b %Y %T",gmtime(&now));	
    
    //字符串存入head。
    sprintf(head,"%s %d %s\r\n"
            "Server: Tencent1.0\r\n"
            "Date:%s\r\n"
            "Content-Type:%s\r\n"
            "Content-Length:%ld\r\n"
    		"Connection:%s\r\n\r\n",
            hres->protocol,hres->status,hres->desc,
            dateTime,
            hres->type,
            hres->length,
            hres->connection);
        	
    return 0;
}
```

```
syscall(SYS_gettid)	   获取真实线程id标识 tid。
在linux下每一个进程都一个进程id，类型pid_t，可以由getpid（）获取
POSIX线程也有线程id，类型pthread_t，可以由pthread_self（）获取，线程id由线程库维护。
但是各个进程独立，所以会有不同进程中线程号相同节的情况。那么这样就会存在一个问题，我的进程p1中的线程pt1要与进程p2中的线程pt2通信怎么办，进程id不可以，线程id又可能重复，所以这里会有一个真实的线程id唯一标识，tid。

\r\n 回车换行
```





#### 通信模块

----一次

创建套接字

组织地址结构

绑定

监听

----多次

接收连接请求---接一个请求，创建一个线程。

接收HTTP请求

发送HTTP响应

```c
//socket.h
#ifndef __SOCKET_H__
#define __SOCKET_H__
//初始化套接字
int initSocket(short port);

//接收客户端连接请求	;服务器端全局套接字，所以不传入数据，返回客户端套接字
int acceptClient(void);

//接收HTTP请求。接收哪一个套接字,返回存储区地址。
char* recvRequest(int socketfd);

//发送HTTP响应。 响应头  响应体。
//发响应头 HTTP模块构建的；
int sendHead(int socketfd,const char* head);
//发响应体 请求中就包括资源路径。
int sendBody(int socketfd,const char* path);

//终止套接字
void deinitSocket(void);
#endif
```

```c
//socket.c
#include<unistd.h>
#include<fcntl.h>							//用于打开请求文件
#include<sys/syscall.h>
#include<sys/types.h>
#include<arpa/inet.h>
#include<stdio.h>
#include<stdlib.h>							//malloc
#include<string.h>
#include"socket.h"

static int serv_sock = -1;					//全局变量-侦听套接字；static只可在当前文件使用。
//*********初始化套接字
int initSocket(short port)
{
    printf("%d.%ld > 创建套接字\n",getpid(),syscall(SYS_gettid));
    serv_sock = socket(AF_INET,SOCK_STREAM,0);
    if(serv_sock == -1)
    {
		perror("socket");
        return -1;
    }
    
    printf("%d.%ld > 设置套接字\n",getpid(),syscall(SYS_gettid));			//time-wait下地址再分配
    int on = 1;
    if(setsockopt(serv_sock,SOL_SOCKET,SO_REUSEADDR,&on,sizeof(on)))
    {
        perror("setsockopt");
        return -1;
    }
    
    printf("%d.%ld > 组织地址结构\n",getpid(),syscall(SYS_gettid));
    struct sockaddr_in serv_adr;
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(port);
    
    printf("%d.%ld > 绑定地址结构\n",getpid(),syscall(SYS_gettid));
    if(bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr)) == -1)
    {
		perror("bind");
        return -1;
    }
   
    printf("%d.%ld > 启动侦听\n",getpid(),syscall(SYS_gettid));
    if(listen(serv_sock,1024) == -1)
    {
        perror("listen");
        return -1;
    }
    
	return 0;
}

//***********接受套接字请求
int acceptClient(void)
{
    printf("%d.%ld > 等待客户机连接\n",getpid(),syscall(SYS_gettid));
    struct sockaddr_in clnt_adr;
    socklen_t len = sizeof(clnt_adr);
    int clnt_sock = accept(serv_sock,(struct sockaddr*)&clnt_adr,&len);
    if(clnt_sock == -1)
    {
        perror("accept");
        return -1;
    }
    printf("%d.%ld > 接收到客户端%s:%hu的连接\n",
           getpid(),syscall(SYS_gettid),inet_ntoa(clnt_adr.sin_addr),ntohs(clnt_adr.sin_port));
    return clnt_sock;
}

//***********接受HTTP请求
char* recvRequest(int socketfd)
{
    //准备存储区存储请求---> 多大空间 ---> 用固定大小缓冲区循环接。
    char* req = NULL;					//记录动态分配的存储区首地址。
    ssize_t len = 0;					//记录已经接收的总字节数。
    for(;;)
    {
        char buf[1024] = {};
        ssize_t size = recv(socketfd,buf,sizeof(buf)-1,0);
        if(size == -1)
        {
            perror("recv");
            free(req);					//出错释放存储区。
            return NULL;
        }
        if(size == 0)					//size == 0 意味着客户端关了套接字，但是不意味着已经接受完数据了。********
        {
            printf("%d.%ld > 客户端关闭链接\n",getpid(),syscall(SYS_gettid));
            free(req);
            return NULL;
        }
        req = realloc(req,len+size+1);		//扩大存储区
        memcpy(req+len,buf,size+1);		//拷贝接受数据到存储区中
        len = size + len;				//长度累加。 
        
        //**************什么时候传完了请求? 两个~\r\n\r\n		接收完成。
        if(strstr(req,"r\n\r\n"))
        {
            break;
        }    
    }
    return req;
}

//***********发送HTTP响应头
int sendHead(int socketfd,const char* head)
{
    if(send(socketfd,head,strlen(head),0) == -1)
    {
        perror("send");
        return -1;
    }
    return 0;
}
//***********发送HTTP响应体
int sendBody(int socketfd,const char* path)				//传文件~文件描述符或者路径字符串
{
    //打开文件,只负责打开文件。文件是不是存在看资源模块。
    int fd = open(path,O_RDONLY);
    if(fd == -1)
    {
        perror("open");
        return -1;
    }
    //读文件，发内容~   文件大小不确定；循环读！
    char buf[1024];
    ssize_t len;
    while((len = read(fd,buf,sizeof(buf)))>0)		
    {
		if(send(socketfd,buf,len,0) == -1)
        {
            perror("send");
            return -1;
        }
    }//循环结束两种情况:执行失败-1，读完0
    if(len == -1)
    {
        perror("read");
        return -1;
    }
    close(fd);							//正常结束；
    return 0;
}

//**********结束套接字
void deinitSocket(void)
{
    close(serv_sock);
}          
```

```
%hu: 以 unsigned short格式输出整数
ntohs： network to host short.

```

##### realloc , memcpy

```
realloc--重新分配内存
memcpy---拷贝数据进内存
```



#### 资源模块

判断资源文件是否存在；

判断文件类型。

```c
//resource.h
#ifndef __RESOURCE_H__
#define __RESOURCE_H__
//判断文件是否存在
int searchResource(const char* path);

//识别文件类型,输出型参数文件类型type。
int identifyResource(const char* path,char* type);

#endif
```

```c
//resource.c
#include<unistd.h>
#include<sys/syscall.h>
#include<stdio.h>
#include<string.h>
#include"resource.h"
#include"mime.h"																//---扩展名搜索对应文件

//********判断文件是否存在
int searchResource(const char* path)
{
    return access(path,R_OK);    
}
//********识别文件类型
int identifyResource(const char* path,char* type)
{
    //路径示例 /home/c/error.html;
    //确定扩展名:	在大串中找最后一个点 str r chr
    char* suffix = strrchr(path,'.');
    if(suffix == NULL)
    {
		printf("%d.%ld > 无法获取拓展名\n",getpid(),syscall(SYS_gettid));
        return -1;
    }
    for(int i=0;i<sizeof(s_mine)/sizeof(s_mine[0]);i++)							//s_mine 为头文件 mime.h的扩展名-文件数组
    {
		if(strcasecmp(suffix,s_mine[i].suffix) == 0)
        {
			strcpy(type,s_mine[i].type);
        	return 0;
        }  
    }
    printf("%d.%ld > 不可识别的文件类型\n",getpid(),syscall(SYS_gettid));
    return -1;
}

```

##### access

```
 return access(path,R_OK);	//判断文件是否有读权限---------可读返回1
```



#### 信号模块

忽略大部分信号    只留下2号(ctrl+c)   和   15号信号(进程结束信号)

```c
//signals.h
#ifndef __SIGNALS_H__
#define __SIGNALS_H__

//初始化信号，忽略大部分信号
int initSignals(void);

#endif
```



```c
//signals.c
#include<unistd.h>
#include<stdio.h>
#include<signal.h>
#include<sys/syscall.h>
#include"signals.h"

//初始化信号
int initSignals(void)
{
    printf("%d.%ld > 忽略大部分信号\n",getpid(),syscall(SYS_gettid));
    for(int signum = SIGHUP;signum <= SIGRTMAX;signum++)
    {
        if(signum != SIGINT && signum != SIGTERM)					
        {
            signal(signum,SIG_IGN);								//kill-l查看信号。 9号信号等是无法忽略的。所以不做错误处理。错了就跳过。
        }
    }
        
    return 0;
}
```



#### 线程模块

负责和客户端通信

![image-20230824181718590](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230824181718590.png)

```c
//client.h
#ifndef __CLIENT_H__
#define __CLIENT_H__
//***********线程过程函数---负责和客户端通信~~~在服务器模块接收了连接请求，才需要开线程和客户端通信。
//和谁通信~~~套接字；
//资源在本地存储的路径~~~
//两个参数,只有一个输入~~~~~结构体
typedef struct clientArgs
{
    const char* home;			//本地存储路径
    int socketfd;				//目标套接字
}CA;

void* client(void* arg);

#endif
```

```c
//client.c
#include<unistd.h>
#include<sys/syscall.h>
#include<sts/stat.h>			//获取文件元数据~~~st_size。文件大小
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

#include"http.h"
#include"socket.h"
#include"resource.h"
#include"client.h"

//**********线程过程函数
//CA ca;
//pthread_create(void* arg)
void* client(void* arg)
{
    CA* ca = (CA*)arg;
    printf("%d.%ld > 客户机线程处理开始\n",getpid(),syscall(SYS_gettid));
    for(;;)
    {
        //*******************接收请求。
        printf("%d.%ld > 接受http请求\n",getpid(),syscall(SYS_gettid));
        char* req = recvRequest(ca->socketfd);
        if(req == NULL)
        {
            break;
        }
        printf("%d.%ld > 请求电文:\n%s\n",getpid(),syscall(SYS_gettid),req);
       
        //*******************解析请求。 
        printf("%d.%ld > 解析请求:\n%s\n",getpid(),syscall(SYS_gettid),req);
        HTTP_REQUEST hreq = {};
		if(parseRequest(req,&hreq) == -1)				//关键数据存入hreq结构体。
        {
            free(req);									//请求错误，错误处理在http模块。这里只需要释放存储区
            req = NULL;
            break;
        }
        free(req);										//解析完成了，数据存入hreq了，释放请求电文存储区
    	req = NULL;
  
        //**************路径处理
    	// /c/error.html	请求路径
   	 	// /home/tarena/2208/project/home	存储路径
    	// /home/tarena/2208/project/home/c/error.html	真实路径
    	char root[PATH_MAX + 1];						//存储路径
        char path[PATH_MAX + 1];						//真实路径
        
        strcpy(root,ca->home);							//存储路径存入root
        //如果路径最后一个元素是/,去掉
        if(root[strlen(root) - 1] == '/')
        {
			root[strlen(root) - 1] = '\0';
        }
    	//拼接文件真实路径。
        strcpy(path,root);
        strcat(path,hreq.path);							//freq.path中存的是请求路径。存储路径+请求路径 =  真实路径。
    	
        //如果请求路径是'/',应该将首页文件给对方。
        if(strcmp(hreq.path,"/") == 0)					// /home/tarena/2208/project/home/,则应该在后面拼接index.html。
        {
            strcat(path,"index.html");
        }
        
        //*************构造响应
        //响应结构体
        HTTP_RESPOND hres = {"HTTP/1.1",200,"OK","text/html"};
        //搜索资源
        if(searchResource(path) == -1)
        {
            //没搜到----> 404.html
            hres.status = 404;
            strcpy(hres.desc,"Not Found");
            // /home/tarena/2208/project/home/404.html
            strcpy(path,root);
            strcat(path,"/404.html");
        }
        	//搜到了,但没识别出来 ----> 404.html
        else if(identifyResource(path,hres.type) == -1)
        {
            hres.status = 404;
            strcpy(hres.desc,"Not Found");
            // /home/tarena/2208/project/home/404.html
            strcpy(path,root);
            strcat(path,"/404.html");
        }
        //确定文件长度~~利用文件元数据
        struct stat st;					//用于输出文件元数据
        if(stat(path,&st) == -1)		//stat()查看元数据，存入stat结构体中。
        {
			perror("stat");
            break;
        }
        hres.length = st.st_size;
        //连接状态
        if(strlen(hreq.connection))		//请求中如果有连接状态，则响应连接状态与之一致。
        {
            strcpy(hres.connection,hreq.connection);	
        }								//请求中没有连接状态，看协议。
        else if(strcasecmp(hreq.protocol,"http/1.0") == 0)
        {
          	strcpy(hres.connection,"Close");
        }
        else
        {
			strcpy(hres.connection,"Keep-alive");
        }
        //构造响应
        printf("%d.%ld > 构造响应\n",getpid(),syscall(SYS_gettid));
        char head[1024];
        if(constructHead(&hres,head) == -1)
        {
			break;
        }
        printf("%d.%ld > 响应电文:\n%s\n",getpid(),syscall(SYS_gettid),head);
        
        //***********发送响应头
        if(sendHead(ca->socketfd,head) == -1)
        {
            break;
        }
        //***********发送响应体
        if(sendBody(ca->socketfd,path) == -1)
        {
            break;
        }
        //***********若连接状态为Close，退出循环。
        if(strcasecmp(hres.connection,"Close") == -1)
        {
            break;
        }
	}
    close(ca->socketfd);
    free(ca);							//开线程时准备动态分配ca.这里释放。
    printf("%d.%ld > 客户端线程处理结束\n",getpid(),syscall(SYS_gettid));
    return NULL;
}
```

```
HTTP请求:
GET / HTTP/1.1				路径为/时，表示首页文件或者404文件。
```



#### 服务器模块

接收链接请求开线程

```c
//server.h
#ifndef __SOCKET_H__
#define __SOCKET_H__
//初始化服务器；初始化套接字、初始化信号
int initServer(short port);

//运行服务器:接收连接请求；开线程
//~接收客户端连接请求	;服务器端全局套接字，所以不传入数据，返回客户端套接字
//~int acceptClient(void);
//~void* client(void* arg);输入需要本地存储地址。
int runServer(const char* home);

//终止服务器
void deinitServer(void);
#endif
```



```c
//server.c
#include<unistd.h>
#include<pthread.h>
#include<sys/resource.h>
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include"socket.h"
#include"signals.h"
#include"client.h"
#include"server.h"

//初始化最大文件描述符数目
static int initMaxFiles(void)						//static函数，仅在当前模块使用。不应该写在头文件中，不该被别的调用server.c程序调用。
{
    //资源限制,获取资源限制数据，存在rl中。
    struct rlimit rl;
    if(getrlimit(RLIMIT_NOFILE,&rl) == -1)
    {
        perror("getrlimit");
        return -1;
    }
    //若当前进程最大文件描述符数目<系统极限
    if(rl.rlim_cur < rl.rlim_max)
    {
		rl.rlim_cur = rl.rlim_max;
        if(setrlimit(RLIMIT_NOFILE,&rl) == -1)
        {
            perror("setrlimit");
            return -1;
        }
    }
    return 0;
}


//初始化服务器；初始化套接字、初始化信号
int initServer(short port)
{
    //初始化最大文件描述符数目
    if(initMaxFiles() == -1)
        return -1;
    //初始化信号
    if(initSignals() == -1)
        return -1;
    //初始化套接字
    if(initSocket(port) == -1)
        return -1;
    return 0;
}

//运行服务器:接收连接请求；开线程
int runServer(const char* home)
{
    while(1)
    {
        int socketfd = acceptClient();
        if(socketfd == -1)
        {
			return -1;
        }
        //打开线程，具有分离属性。
        pthread_t tid;
        pthread_attr_t attr;				//线程属性
        pthread_attr_init(&attr);			//初始化
        pthread_attr_setdetachstate(&attr,PTHREAD_CREATE_DETACHED);			//将分离属性设置到attr中
        
        CA* ca = malloc(sizeof(CA));										//动态创建CA结构体，将在线程函数client.c释放。
        ca -> socketfd = socketfd;
        ca -> home = home;
        int error = pthread_create(&tid,&attr,client,ca);
        if(error)
        {
            fprintf(stderr,"pthread_create:%s\n",strerror(error));
            return -1;
        }
    }
    return 0;
}

//终止服务器
void deinitServer(void)
{
    deinitSocket();
}
```



##### main模块

```c
//main.c
#include<stdio.h>
#include<stdlib.h>
#include"server.h"

int main(int argc,char* argv[])
{
    // ./a.out 8080 ../home				默认80 ../home
    //初始化服务器
    if(initServer(argc < 2 ? 80:atoi(argv[1])) == -1)
    {
        return -1;
    }
    //运行服务器
    if(runServer(argc < 3? "../home" : argv[2]) == -1)
    {
        return -1;
    }
    //终结化服务器
    deinitServer();
    return 0;
} 
```



```
编译:所有.o文件、链接线程库。
gcc main.c *.o -o webserver.c -pthread

运行permission denied  -----    $sudo ./webserver
```



#### windows无法访问Linux搭建的web服务器

发现是防火墙未开放相应端口。

Ubuntu:

```
1.查看防火墙状态
sudo ufw status
```

















