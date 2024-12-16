# Adding a Library

## Creating a Library

创建一个库

1. 在子目录的CMakeLists.txt中添加库
2. 在顶层 CMakeLists.txt 中添加子目录的库
3. 在顶层 CMakeLists.txt 中链接该库
4. 在顶层 CMakeLists.txt 中指定该库所在的目录

```cmake
-- ./src/CMakeLists.txt
add_library(func func.cpp)

-- ./CMakeLists.txt
add_subdirectory(src)
target_link_libraries(project PUBLIC func)
target_include_directories(project PUBLIC "${PROJECT_SOURCE_DIR}/src")
```

## Adding an Option

添加选项

1. 在子目录的CMakeLists.txt 添加选项
2. 然后添加判断是否使用可选项
3. 源代码中或许也要修改

```cmake
-- ./src/CMakeLists.txt
option(USE_OPTION "Use option" ON)

if(USE_OPTION)
  target_compile_definitions(project PUBLIC USE_OPTION)
  add_library(funclib STATIC sqrt.cpp)
  target_link_libraries(func PRIVATE funclib)
endif()

add_library(func func.cpp)

```

- 当可选项为 ON 的时候 会将库中的源文件添加到目标
