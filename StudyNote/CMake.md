# CMake学习

CMake 是一个开源的跨平台构建系统，主要用于管理和组织软件项目的编译过程。它通过平台无关的构建脚本（即 CMakeLists.txt 文件）来生成本地构建系统（如 Makefiles 或 Visual Studio 项目文件）。

## CMake 的核心概念

>CMakeLists.txt: CMake 使用的脚本文件。每个目录下都应有一个 CMakeLists.txt 文件，用于定义该目录的构建规则。
>目标（Targets）: 主要包括可执行文件和库，CMake 可以定义多种目标，如 add_executable 和 add_library。
>构建类型（Build Types）: 如 Debug、Release 等。通过 CMAKE_BUILD_TYPE 指定，影响编译器选项。
>配置步骤: CMake 通过 cmake 命令来生成项目的构建文件，例如 Makefile 或 IDE 项目。
>生成步骤: 一旦配置成功，通过构建系统（如 make 或 Visual Studio）执行编译。

### CMake 基础命令

#### 1.项目配置

```bash
# CMakeLists.txt 文件开头一般是定义项目的名称和最低 CMake 版本要求：
cmake_minimum_required(VERSION 3.10)
project(MyProject)
```

#### 定义可执行文件

```bash
# 使用 add_executable 定义要构建的可执行文件。参数是可执行文件的名称和源文件列表：
add_executable(MyExecutable main.cpp)
```

#### 定义库

```bash
# 可以定义静态库或共享库，分别使用 add_library：
# 定义静态库
add_library(MyStaticLib STATIC lib.cpp)

# 定义共享库
add_library(MySharedLib SHARED lib.cpp)
```

#### 设置编译选项

```bash
# 通过 target_compile_options 或 set 命令为目标设置编译器选项：
target_compile_options(MyExecutable PRIVATE -Wall -O2)

# 或者使用 set 来设置全局编译选项：
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O2")
```

#### 指定构建类型

```bash
# 可以在命令行或者 CMakeLists.txt 中指定构建类型：
cmake -DCMAKE_BUILD_TYPE=Release ..
set(CMAKE_BUILD_TYPE Debug)
```

#### 构建和安装项目

生成构建文件：

```bash
mkdir build
cd build
cmake ..
```

- `cmake ..`:在当前目录下生成构建文件。
- `mkdir build`:在当前目录下创建一个 build 目录。

编译项目：

```bash
cmake --build .
```

- `cmake --build .`:在当前目录下使用生成的构建文件 (如 Makefile) 编译项目。

安装项目：

```bash
cmake --install .
```

- `cmake --install .`:将编译好的文件安装到系统中。

### 2.CMake 语法

#### 2.1 message 命令

```bash
message("Hello, world!")
message(Hello, world!)
```

#### 2.2 set 命令

```bash
set(<variable> <value>... [CACHE <type> <docstring> [FORCE]])
<variable>: 要设置的变量名。
<value>: 变量值，可以是单个值或多个值。
[CACHE]: 可选参数，用于将变量存储在缓存中，便于在不同的 CMake 调用之间保持值。
[FORCE]: 可选参数，强制覆盖缓存中已有的变量值。

set(VAR "value")
set(VAR value)
set(VAR ${value})
set(VAR ${value} CACHE STRING "description")
```

#### list 命令

````bash
list(<command> <listname> <args>...)

# 向列表中添加元素
list(APPEND MY_LIST "item1" "item2" "item3")

# 获取列表长度
list(LENGTH MY_LIST LENGTH_VAR)
message(STATUS "List length: ${LENGTH_VAR}")

# 获取列表中索引为的元素
list(GET MY_LIST 0 FIRST_ITEM)
message(STATUS "First item: ${FIRST_ITEM}")

# 从列表删除某项
list(REMOVE_ITEM MY_LIST "item2")

# 对列表进行排序
list(SORT MY_LIST)

// 查找某个元素
list(FIND MY_LIST "item1" INDEX)
message(STATUS "Index of item1: ${INDEX}")
````

#### if 语句

```bash
if(<condition>)
    # 条件成立时执行的命令
elseif(<condition>)
    # 另一个条件成立时执行的命令
else()
    # 上述条件都不成立时执行的命令
endif()

```
