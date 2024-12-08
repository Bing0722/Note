## CMake

### 常用 CMake 模式
```bash
cmake_minimum_required(VERSION 3.10)    # 设置 CMake 最低版本
project(MyProject)                      # 设置项目名称

set(PROJECT_NAME "MyProject")           # 设置项目名称

set(CMAKE_CXX_STANDARD 11)              # 设置 C++ 语言标准
set(CMAKE_CXX_STANDARD_REQUIRED ON)     # 设置 C++ 语言标准是否必须

find_package(OpenSSL REQUIRED)           # 查找 OpenSSL 库

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Werror")  # 设置编译选项

include_directories(${PROJECT_SOURCE_DIR}/include)  # 设置头文件搜索路径

aux_source_directory(${PROJECT_SOURCE_DIR}/src SRCS_LIST)   # 设置源文件搜索路径

add_executable(${PROJECT_NAME} ${SRCS_LIST})       # 添加可执行文件

if(OPENSSL_FOUND)                             # 链接 OpenSSL 库
    target_include_directories(${PROJECT_NAME} PRIVATE ${OPENSSL_INCLUDE_DIR})    # 设置头文件搜索路径
    target_link_libraries(${PROJECT_NAME} PRIVATE ${OPENSSL_LIBRARIES})    # 链接 OpenSSL 库
else()
    message(FATAL_ERROR "OpenSSL not found")        # 未找到 OpenSSL 库时报错
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES    # 设置可执行文件属性
    RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)        # 设置可执行文件输出路径
```
