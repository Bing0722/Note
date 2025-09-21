# Adding Usage Requirements for a Library

## Adding Usage Requirements for a Library

添加库的使用要求

1. 在子目录 CMakeLists.txt中添加包含当前目录 使用INTERFACE关键字

```cmake
-- ./src/CMakeLists.txt
target_include_directories(func INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

## Setting the C++ Standard with Interface Libraries

使用接口库设定c++标准

1. 删去原有的set设定的c++标准
2. 在顶层 CMakeLiosts.txt中添加INTERFACE库的定义
3. 在INTERFACE库中添加c++标准
4. 在子目录中也要链接到新的INTERFACE库

```cmake
-- ./CMakeLists.txt
add_library(flags INTERFACE)
target_compile_features(flags INTERFACE cxx_std_11)

-- ./src/CMakeLists.txt
target_link_libraries(func INTERFACE flags)
```
