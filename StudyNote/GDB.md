## GDB
GDB (GNU Debugger) 是一款开源的、跨平台的、功能强大的程序调试器。它可以用来调试多种程序，包括 C、C++、Java、Python、Go 等。GDB 可以帮助我们快速定位和解决程序中的错误，并帮助我们理解程序的运行机制。

### 1.编译时启用调试信息
在使用GDB调试之前，你需要确保在编译C++代码时包含调试信息。可以通过-g编译选项来生成调试信息
```bash
g++ -g main.cpp -o main
```

### 2.启动GDB
启动GDB后，我们需要告诉它我们要调试的程序的名称和可执行文件。
```bash
gdb ./main
```

### 3.常用GDB命令
#### 3.1 运行程序
```bash
run
```
- `run`命令会运行程序，直到遇到断点或程序结束。

#### 3.2 设置断点
```bash
break [位置]
```
- `break`(或`b`)命令可以设置断点，当程序运行到断点处时，GDB会暂停运行，等待我们输入命令。
  
位置可以是函数名、行号、文件名和函数+行号的组合。
- 在函数处设置断点：
```bash
break main
break MyClass::myMethod
```

- 在某一行设置断点：
```bash
break 123               # 断点在第123行
break MyClass.cpp:123   # 断点在MyClass.cpp的第123行
```

#### 3.3 列出断点
```bash
info breakpoints
```
- `info breakpoints`(或`i b`)命令可以列出当前设置的所有断点。
  
#### 3.4 删除断点
```bash
delete [编号]
```
- `delete`(或`d`)命令可以删除指定编号的断点。

#### 3.5 单步执行
```bash
step
next
```
- `step`(或`s`)命令可以单步执行程序，遇到函数调用时，会进入函数内部。
- `next`(或`n`)命令与`step`命令类似，但遇到函数调用时，不会进入函数内部。

#### 3.6 继续运行
```bash
continue
```
- `continue`(或`c`)命令可以继续运行程序，直到遇到下一个断点或程序结束。

#### 3.7 查看变量
```bash
print [变量名]
```
- `print`(或`p`)命令可以查看变量的值。

#### 3.8 查看当前栈帧
```bash
backtrace
```
- `backtrace`(或`bt`)命令可以查看当前栈帧。
  
#### 3.9 切换栈帧
```bash
frame [编号]
```
- `frame`(或`f`)命令可以切换到指定编号的栈帧。

#### 3.10 查看局部变量
```bash
info locals
```
- `info locals`(或`i l`)命令可以查看当前栈帧的局部变量。

#### 3.11 条件断点
```bash
break [位置] if [条件]
```
- `break if [条件]`命令可以设置条件断点，只有满足条件时，程序才会暂停运行。

#### 3.12 观察变量变化
```bash
watch [变量名]
```
- `watch`(或`w`)命令可以观察变量的值的变化。

#### 3.13 跳过代码
```bash
jump [位置]
```
- `jump`(或`j`)命令可以跳过指定位置的代码。

#### 3.14 退出GDB
```bash
quit
```
- `quit`(或`q`)命令可以退出GDB。

### 4.调试多线程程序
GDB可以调试多线程程序，但需要使用`-pthread`编译选项。
```bash
g++ -g -pthread main.cpp -o main
```

- 查看所有线程：
```bash
info threads
```
- 切换线程：
```bash
thread [编号]
```

### 5.调试核心存储文件


### 6.GDB的TUI模式

### 7.断点命令与自动化操作
- 在断点处自动执行某些操作(如打印变量或继续执行):
```bash
break myfile.cpp:15
commands  # 输入命令来定义在断点处自动执行的操作
print x   # 打印变量 x
continue  # 继续执行
end
```

### 8.调试技巧
- 调试优化后的代码：编译优化后的代码时，可能会让调试变得更困难。因此，如果需要调试，建议先用 -O0 关闭编译器优化
```bash
g++ -g -O0 main.cpp -o main
```

- 反汇编代码:查看某个函数的汇编代码，有时在调试优化后的程序或调试低层次问题时非常有用。
```bash
disas function_name
```

- 跳过函数:有时我们只想跳过某个函数，而不想进入函数内部。这时，可以用 `finish` 命令跳过函数，并继续执行程序。
```bash
finish
```
内存工具有哪些？
- 内存分析工具：Valgrind、AddressSanitizer、LeakSanitizer等。