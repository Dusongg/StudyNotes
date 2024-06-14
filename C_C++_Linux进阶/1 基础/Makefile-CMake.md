# [Makefile](https://www.yuque.com/docs/share/8495ea21-9tdb-4e7b-aca2-babd86751e39?#)

## 三要素

![image-20240226113411497](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240226113411497.png)

- 伪目标：`.PHONY`

## 变量

```makefile
CC = gcc
GXX = g++
DEBUG = -g
CFLAGS = $(DEBUG) -Wall -c
EXE = simple
OBJS = main.0 foo.o

INC = -I ./include/
LIB = -L ./lib/

$(EXE): $(OBJS)
	$(CC) $(CFLAGS) -o  $(EXE) $(OBJS)
main.o: main.c
	$(CC) -o $(CFLAGS) main.o -c main.c
foo.o: foo.c
	$(CC) -o $(CFLAGS) foo.o -c foo.c
.PHONY: clean
clean:
	rm $(EXE) $(OBJS)
```

## 自动变量

```makefile
CC = gcc
RM = rm
EXE = simple

#SRCS = main.c foo.c foo2.c
SRCS = $(wildcard *.c)

#OBJS = main.o foo.o foo2.o
OBJS = $(patsubst %.c %.o $(SRCS))

$(EXE): $(OBJS)
	$(CC) -o $@ $^
%.o: %.c
	$(CC) -o $@ -c $^
.PHONY:clean
clean:
	$(RM) $(EXE) $(OBJS)

```

![image-20240226132346155](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240226132346155.png)



# CMake

- **创建`CMakelists.txt`文件**，大小写不影响 

 ```cmake
 # 单个目录实现
 # CMake 最低版本号要求
 cmake_minimum_required (VERSION 2.8)
 # 工程，他不是执行文件名
 PROJECT(Darren)
 # 手动加入文件 ${变量名}} ，比如${SRC_LIST}
 SET(SRC_LIST main.c)
 set(SRC_LIST2 main2.c)
 # MESSAGE和echo类似 
 MESSAGE(STATUS "PROJECT_BINARY_DIR DIR " ${PROJECT_BINARY_DIR})
 MESSAGE(STATUS "PROJECT_SOURCE_DIR DIR " ${PROJECT_SOURCE_DIR})
 
 # 生产执行文件名0voice  0voice2
 ADD_EXECUTABLE(0voice ${SRC_LIST})
 ADD_EXECUTABLE(0voice2 ${SRC_LIST2})
 ```

1. 在当前目录执行cmake，会生成很多文件`cmak_install.cmake`, `Makefile`，`CMakeCache.txt`等

```bash
cmake .
```

2. 创建build目录，执行

```bash
cmake ..
```

3.  