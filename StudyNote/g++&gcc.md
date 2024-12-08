## g++/gcc

- g++是GNU C++编译器，是GNU项目下的C++编译器。
- gcc是GNU C编译器，是GNU项目下的C语言编译器。
  
  两者的区别：
  
  - 语言：g++支持C++语言，gcc只支持C语言。
  - 功能：g++支持更多的C++特性，如模板、异常处理等，gcc只支持基本的C语言特性。
  - 性能：g++编译速度更快，gcc编译速度更慢。

  一般来说，如果需要编译C++程序，则使用g++；如果只需要编译C程序，则使用gcc。

### 基本用法

#### 编译：

```bash
g++ -o output_file src.cpp
```
- `g++`：编译器
- `-o output_file`：指定输出文件名
- `src.cpp`：源文件名
  
#### 编译多个源文件：

```bash
g++ -o output_file src1.cpp src2.cpp src3.cpp
```

#### 分阶段编译：

```bash
g++ -c src.cpp
g++ -o output_file src.o
```

#### 常用编译选项：

- `-Wall`：显示所有警告信息。
- `-g`：生成调试信息。
- `-O`：优化编译。
- `-std=c++11`：使用C++11标准。

#### 编译静态库和动态库：

创建静态库：
```bash
g++ -c utils.cpp    
ar rcs libutils.a utils.o

# 在编译时链接静态库
g++ -o main main.cpp -L. -lutils
```  

创建动态库：
```bash
g++ -fPIC -c utils.cpp
g++ -shared -o libutils.so utils.o

# 在编译时链接动态库
g++ -o main main.cpp -L. -lutils 
```
### 常用的其他选项
- `-I`：指定头文件搜索路径。
- `-D`：定义宏。
- `-l`：链接库。
- `-L`：指定库搜索路径。
- `-E`：预处理。
- `-c`：只编译，不汇编。
- `-S`：只汇编，不编译。
- `-o`：指定输出文件名。
- `-v`：显示详细信息。
- `-w`：抑制所有警告信息。