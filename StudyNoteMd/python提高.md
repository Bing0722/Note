#### 文件操作

Python **open()** 方法用于打开一个文件，并返回文件对象。

完整的语法：

```py
open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)
```

参数说明:

- **file: **必需，文件路径（相对或者绝对路径）
- **mode:** 可选，文件打开模式
- **buffering:** 设置缓冲
- **encoding:** 一般使用utf8
- **errors:** 报错级别
- **newline:** 区分换行符
- **closefd:** 传入的file参数类型
- **opener:** 设置自定义开启器，开启器的返回值必须是一个打开的文件描述符

mode参数有：

```py
"""
t	: 文本模式 (默认)
x	: 写模式，新建一个文件，如果该文件已存在则会报错
b	: 二进制模式
+	: 打开一个文件进行更新(可读可写)
U	: 通用换行模式（Python 3 不支持）
r	: 以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式
rb	: 以二进制格式打开一个文件用于只读 文件指针将会放在文件的开头。这是默认模式。一般用于非文本文件如图片等
r+	: 打开一个文件用于读写。文件指针将会放在文件的开头
rb+ : 以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。一般用于非文本文件如图片等
w	: 打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件
wb	: 以二进制格式打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。一般用于非文本文件如图片等
w+	: 打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
wb+	: 以二进制格式打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。一般用于非文本文件如图片等
a	: 打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
ab	: 以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
a+	: 打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。
ab+	: 以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写。
"""
```

##### file对象的操作

```py
file = open("xxxx")
file.close()	# 关闭文件
file.flush()	# 刷新文件内部缓冲 直接把内部缓冲区的数据立刻写入文件 而不是被动的等待输出缓冲区写入
file.read()		# 从文件读取指定的字节数，如果未给定或为负则读取所有
file.readline()	# 读取整行，包括 "\n" 字符
file.readlines()# 读取所有行并返回列表，若给定sizeint>0，返回总和大约为sizeint字节的行, 实际读取值可能比 sizeint 较大, 因为需要填充缓冲区
file.readable()	# 判断是否可读
file.write()	# 将字符串写入文件，返回的是写入的字符长度
file.writelines()	# 向文件写入一个序列字符串列表，如果需要换行则要自己加入每行的换行符
file.tell()		# 返回文件当前位置
file.seek()		# 移动文件读取指针到指定位置
file.truncate()	# 从文件的首行首字符开始截断 截断文件为 size 个字符 无 size 表示从当前位置截断
file.isatty()	# 如果文件连接到一个终端设备返回 True，否则返回 False
file.fileno()	# 返回一个整型的文件描述符(file descriptor FD 整型), 可以用在如os模块的read方法等一些底层操作上
```

##### 文件的使用

```py
"""
with 结合open使用 可以帮助我们自动释放资源
"""
with open("xxxx", 'rb') as stream
	pass
```

#### OS模块(文件相关)

```py
"""
OS模块中的文件相关
"""
from os import path
path.abspath('file.txt')	# 获取文件的绝对路径
path.exists('file.txt')		# 判断文件或目录是否存在
path.isfile('file.txt')		# 判断是否是文件
path.isdir('directory')		# 判断是否是目录
path.getsize('file.txt')	# 获取文件大小，单位字节
path.getctime('file.txt')	# 获取文件创建时间
path.getatime('file.txt')	# 获取文件最后访问时间
path.getmtime('file.txt')	# 获取文件最后修改时间
path.basename('path/to/file.txt') 	# 获取文件名
path.splitext('file.txt')	# 获取文件扩展名
path.dirname('/path/to/file.txt')	# 获取文件的父目录

# os操作
os.getcwd()					# 获取当前工作目录
os.chdir('/path/to/directory')  	# 改变当前工作目录
os.listdir('/path/to/directory')  	# 返回一个列表，包含指定目录中的文件和子目录名
os.rename('old_name.txt', 'new_name.txt')  # 重命名文件
os.remove('file.txt')  		# 删除指定的文件
os.mkdir('new_directory')  	# 创建一个单一目录
os.makedirs('path/to/directory', exist_ok=True)  # 创建多层目录，exist_ok=True表示如果目录已存在，不抛出异常
os.rmdir('empty_directory') # 删除一个空目录
os.stat('file.txt')			# 查看文件的权限信息
os.chmod('file.txt', 0o777) # 改变文件的权限为可读、可写、可执行
os.getlogin()				# 获取当前用户的用户名

# os 模块本身不提供复制文件的功能，但可以使用 shutil 模块来复制文件
import shutil
shutil.copy('file.txt', 'copy_of_file.txt')  	# 复制文件
shutil.rmtree('non_empty_directory')  			# 删除非空目录
```

#### 异常处理

```py
"""
格式：
try:
	可能出现异常的代码
except [报错类型] [as err]:
	如果有异常执行的代码
	[print(err)]
[finally:
	无论是否存在异常都会被执行的代码]


抛出异常：
raise Exception("xxx")
"""
```

#### 生成器

```py
"""
生成器
定义生成器方式：
1. 通过列表推导式
	g = (x*3 for x in range(20))	# --> gentrator
2. 函数+ yield	
	def func():
		...
		yield
	g = func()

生成元素：
1. next(generator)	--> 每次调用都会产生一个新的元素 如果元素产生完毕 再次调用则会产生异常
2. 生成器自己的方法：
	g.__next__()
	g.send(value)
	
应用：协程
"""
```

#### 迭代器

```py
"""
迭代器是一个可以记住遍历的位置的对象。

迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退。

迭代器有两个基本的方法：iter() 和 next()。

借助iter转换成迭代器
"""
```

#### 面向对象

```py
# 类的定义
class Myclass：
	pass

# 类中方法：普通方法，类方法，静态方法，魔术方法
# 普通方法：
class Myclass:
    def func(self):
        pass

# 魔术方法
# (重点)
# __init__:初始化魔术方法
# 触发时机：初始化对象时触发(不是实例化触发，但是和实例化在同一个操作中)
class Myclass:
    def __init__(self):
        pass

# (了解)
# __new__:实例化的魔术方法
# 触发时机：在实例化时触发 
# __del__:作用 没有指针引用的时候会调用，99%都不需要重写
# __call__:作用：想不想将对象当成函数用

# 类方法
# 特点：
# 1. 定义需要依赖装饰器@classmethod
# 2. 类方法中参数不是对象，而是类
# 3. 类方法中可以使用类属性
# 4. 类方法不能使用普通方法
# 类方法的作用：因为只能访问类属性和类方法，所以可以在对象创建之前，如果需要完成一些动作(功能)
class Myclass:
	@classmethod
	def func(cls):
        pass
 
# 静态方法 类似于类方法
# 1. 需要装饰器@staticmethod
# 2. 静态方法无序传递参数(cls, self)
# 3. 也只能访问类的属性和方法，对象的是无法访问的
# 4. 加载时机同类
class Myclass:
    @staticmethod
	def func():
        pass

# 私有化
# 封装：1. 私有化属性 2. 定义公有set和get方法
# __属性: 就是将属性私有化
```

#### 继承

```py
class Parent:
    pass

class Chird(Parent):
    pass
	super().__init__()	# super() 调用父类的对象
```

#### 多态

```py
```

#### 单例模式

```py

```

