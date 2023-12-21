#### 小项目可使用的makefile写法

~~~makefile
all:server client    //目标文件server和client

server:server.o wrap.o  //server的依赖为server.o和wrap.o
  gcc server.o wrap.o -o server -Wall   //利用gcc编译器生成可执行文件server
  
client:client.o wrap.o  //client的依赖为client.o和wrap.o
  gcc client.o wrap.o -o client -Wall  //利用gcc编译器生成可执行文件client

server.o:server.c wrap.h  //server.o的依赖为server.c和wrap.h
  gcc -c server.c -o server.o -Wall  //利用gcc编译器生成中间目标文件server.o

client.o:client.c wrap.h  //同上
  gcc -c client.c -o client.o -Wall

wrap.o:wrap.c wrap.h   //同上
  gcc -c wrap.c -o wrap.o -Wall
~~~

#### 大项目使用的makefile写法

~~~makefile
target1=server        //服务器运行目标文件server
target2=client        //客户端目标文件client
src=$(wildcard *.c)   //用wildcard函数找到所有.c文件，server.c、client.c、wrap.c
deps=$(wildcard *.h)   //用wildcard函数找到所有的.h文件，wrap.h
obj=$(patsubst %.c,%.o,$(src))    //用patsubst函数将所有.c文件替换成.o文件

all:$(target1) $(target2)    //目标文件server和client，多个目标文件一定形成此依赖关系

$(target1):server.o wrap.o  //server的依赖为server.o和wrap.o
  gcc $^ -o $@ -Wall   //利用gcc编译器生成可执行文件server
  
$(target2):client.o wrap.o  //client的依赖为client.o和wrap.o
  gcc $^ -o $@ -Wall  //利用gcc编译器生成可执行文件client

%.o:%.c $(deps)  //任意一个.o中间目标文件的依赖是其对应的.c文件，如：client.o的依赖为client.c
  gcc -c $< -o $@ -Wall   //根据目标文件编译的需求依次将依赖编译成对应的中间目标文件

.PHONY:clean  //伪文件，需要在终端输入make clean才会调用
clean:
  -rm -rf $(target1) $(target2) $(obj)   //删除所有的目标文件以及中间目标文件，用于重新编译。
~~~

~~~markdown
 1、$<     第一个依赖文件，若有多个目标文件，依次编译依赖文件
 2、$^     所有依赖文件的集合
 3、$@     目标文件的集合
~~~

