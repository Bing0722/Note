## Makefile

Makefile 是 GNU Make 工具使用的配置文件，主要用于编译和构建项目，尤其是在 C/C++ 等语言的项目中非常常见。通过 Makefile，你可以定义规则来简化编译过程，管理依赖关系，并加速开发。

### 1.Makefile 基本结构

```makefile
target : dependencies
    command
```
- `target`：目标文件或目标操作。
- `dependencies`：目标所依赖的文件或操作。
- command：执行的命令。 

### 2.Makefile 示例

```makefile
# 生成目标程序 myprogram
myprogram: main.o utils.o
    gcc -o myprogram main.o utils.o

# 编译 main.c 生成 main.o
main.o: main.c
    gcc -c main.c

# 编译 utils.c 生成 utils.o
utils.o: utils.c
    gcc -c utils.c

# 清理编译生成的文件
clean:
    rm -f *.o myprogram
```

###  3.使用 Makefile

- 在包含上述内容的 Makefile 所在目录下，执行 make 命令

```bash
make
```

### 4.Makefile 常用变量
```makefile
CC = gcc                # 编译器
CFLAGS = -Wall -g       # 编译选项
TARGET = myprogram      # 目标可执行文件
OBJS = main.o utils.o   # 目标文件列表

# 使用变量
$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

%.o: %.c
    $(CC) $(CFLAGS) -c $<

clean:
    rm -f $(OBJS) $(TARGET)
```
- $(CC)：表示使用变量 CC 的值（gcc）。
- $(CFLAGS)：编译选项变量。
- $(OBJS)：目标文件集合。
- $(TARGET)：要生成的目标文件。
- $<：依赖文件中第一个文件的名字

### 5.自动变量
- `$@`：代表目标文件名。
- `$^`：代表所有依赖文件名。
- `$<`：代表第一个依赖文件名。
- `$?`：代表比目标文件更新的依赖文件名。
- `$*`：代表目标文件名的前缀。

### 6.通配符和模式匹配

通配符示例：
```makefile
SRCS = $(wildcard *.c)   # 所有 .c 文件
OBJS = $(patsubst %.c,%.o,$(SRCS))   # 对所有 .c 文件替换成 .o
```
- `$(wildcard *.c)`：匹配当前目录下所有 .c 文件。
- `$(patsubst %.c,%.o,$(SRCS))`：使用模式匹配，将所有 .c 文件替换成 .o。

### 7.伪目标
```makefile
.PHONY: clean all

all: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) -o $(TARGET) $(OBJS)

%.o: %.c
    $(CC) $(CFLAGS) -c $<

clean:
    rm -f $(OBJS) $(TARGET)
```
- `.PHONY`：伪目标，不代表文件，仅仅表示一个动作。
- `all`：目标文件，表示编译所有文件。
- `$(TARGET)`：依赖目标文件，表示编译目标文件。
- `clean`：清理目标文件。
- `$(OBJS)`：目标文件集合。
- `$(CC) -o $(TARGET) $(OBJS)`：编译命令。

### 8.条件判断

```makefile
ifeq ($(OS),Windows_NT)
    RM = del /f /q
else
    RM = rm -f
endif

clean:
    $(RM) *.o hello.exe
```
- `ifeq ($(OS),Windows_NT)`：判断操作系统是否为 Windows。
- `RM = del /f /q`：设置删除命令。
- `else`：条件结束。

### 9.函数

Makefile 提供了一些内置函数来进行字符串操作、路径处理等。

- `$(wildcard pattern)`：返回符合模式的文件列表。
- `$(patsubst pattern,replacement,text)`：对文本中的模式进行替换。
- `$(addprefix prefix,names)`：为每个名字添加前缀。
- `$(addsuffix suffix,names)`：为每个名字添加后缀。

示例：
```makefile
SRCS = $(wildcard src/*.c)
OBJS = $(patsubst %.c,%.o,$(SRCS))

all: $(OBJS)
    gcc -o myprogram $(OBJS)
```

### 10.包含其他 Makefile
```makefile
include config.mk
```
- `include config.mk`：包含其他 Makefile。

### 11.分段编译(递归 Makfile)

```makefile
SUBDIRS = src lib

all: $(SUBDIRS)

$(SUBDIRS):
    $(MAKE) -C $@

clean:
    for dir in $(SUBDIRS); do $(MAKE) -C $$dir clean; done
```
- `SUBDIRS`：子目录列表。
- `$(SUBDIRS)`：递归编译所有子目录。
- `$(MAKE) -C $@`：进入子目录，并执行 make 命令。
- `clean`：清理所有子目录的目标文件。

### 12.Makefile 最佳实践

- 使用变量，减少重复代码。
- 使用模式匹配，简化文件名处理。
- 使用函数，简化字符串处理。
- 使用伪目标，简化命令。
- 使用分段编译，提高编译速度。

### 示例
```makefile
# Makefile for a multi-source C project

# 编译器和选项
CC = gcc
CFLAGS = -Wall -g

# 目标可执行文件
TARGET = myprogram

# 源文件
SRCS = main.c utils.c math.c

# 目标文件
OBJS = $(SRCS:.c=.o)

# 链接目标
$(TARGET): $(OBJS)
    $(CC) $(CFLAGS) -o $@ $(OBJS)

# 编译规则
%.o: %.c
    $(CC) $(CFLAGS) -c $<

# 清理规则
clean:
    rm -f $(OBJS) $(TARGET)

.PHONY: clean
```