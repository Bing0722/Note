# A Basic Staring Point

## Building a Basic Project

构建最基本的cmake项目

1. CMake 最低版本要求
2. CMake 工程名和版本号
3. CMake 工程的源文件

```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject VERSION 1.0.0)
add_executable(MyProject main.cpp)
```

## Specifying the C++ Standard

指定目标c++标准

1. 设置全局c++标准
2. 确保强制使用该版本（不允许降级）

```cmake
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

## Configured Header File

配置头文件

1. 配置头文件
2. 添加头文件的路径

```cmake
configure_file(./myheader.h.in ./myheader.h)
target_include_directories(MyProject PUBLIC ${CMAKE_CURRENT_BINARY_DIR})
```
