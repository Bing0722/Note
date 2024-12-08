### Python模块

#### os模块

​	`os` 模块用于与操作系统进行交互，提供了许多文件和目录操作、进程管理、环境变量获取等功能

```py
# 1.文件和目录相关操作
import os
os.path.abspath(path)		: 获取文件的绝对路径
os.path.basename(path)		: 获取路径中的文件名
os.path.dirname(path)		: 获取路径中的目录
os.path.exists(path)		: 判断路径是否存在
os.path.isfile(path)		: 判断路径是否为文件
os.path.isdir(path)			: 判断路径是否为目录
os.path.getsize(path)		: 获取文件的大小(字节)
# 列出目录内容
os.listdir(path)			: 列出指定目录中的文件和目录名
# 创建和删除目录
os.mkdir(path)				: 创建单一目录
os.makedirs(path)			: 创建多层目录
os.rmdir(path)				: 删除空目录
os.removedirs(path)			: 删除多层空目录
# 文件重命名和删除
os.rename(src, dst)			: 重命名文件或目录
os.remove(path)				: 删除文件

# 2.进程管理
# 执行系统命令
os.system(command)			: 执行系统命令，返回命令的退出状态码
# 获取和设置当前工作目录
os.getcwd()					: 获取当前工作目录
os.chdir(path)				: 修改当前工作目录

# 3.环境变量
# 获取和设置环境变量
os.environ					: 获取环境变量的字典
os.getenv(key, default)		: 获取指定环境变量的值
os.putenv(key, value)		: 设置环境变量
# 清除环境变量
os.unsetenv(key)			: 删除指定的环境变量

# 4.路径操作
# 路径拼接和分割
os.path.join(path, *paths)	: 将多个路径组合成一个路径
os.path.split(path)			: 将路径拆分成目录和文件名两部分
os.path.splitext(path)		: 将路径拆分成文件名和扩展名两部分

# 5.获取系统信息
# 获取当前操作系统
os.name						: 获取当前操作系统的名称(如：posix、nt)
# 获取操作系统详细信息
os.uname()					: 获取操作系统的详细信息，返回一个命名元组(Linux 和 macOS 支持)

# 6.文件权限管理
# 修改文件权限和所有者
os.chmod(path, mode)		: 修改文件的权限
os.chown(path, uid, gid)	: 修改文件的拥有者
# 获取文件描述符
os.open(path, flags)		: 打开文件并返回文件描述符
os.close(fd)				: 关闭文件描述符

# 7.常用文件操作
# 文件复制(需导入 shutil 模块)
shutil.copy(src, dst)		: 复制文件
shutil.copytree(src, dst)	: 复制整个目录树
```

#### sys模块

​	`sys` 模块提供了一些变量和函数，用于与 Python 解释器进行交互

```py
# 1.获取命令行参数
sys.argv 		是一个列表，包含了执行 Python 脚本时传入的命令行参数

# 2.退出程序
sys.exit() 		用于终止 Python 程序，可以带一个整数参数，表示退出状态码。0 表示正常退出，非零表示异常退出

# 3.标准输入、输出和错误流
sys.stdin		标准输入流
sys.stdout		标准输出流
sys.stderr		标准错误流

# 4.获取 Python 版本信息
sys.version 	返回 Python 解释器的版本信息

# 5.获取 Python 解释器的路径
sys.executable 	返回 Python 解释器的可执行文件路径

# 6.获取模块的搜索路径
sys.path 		是一个列表，包含 Python 查找模块的路径

# 7.获取操作系统平台
sys.platform 	返回操作系统的标识符，例如 win32 表示 Windows，linux 表示 Linux

# 8.获取最大递归深度
sys.getrecursionlimit() 获取 Python 解释器的最大递归深度，避免递归调用太深导致栈溢出

# 9.设置最大递归深度
sys.setrecursionlimit(limit) 设置递归调用的最大深度

# 10.获取最大整数值
sys.maxsize 	表示解释器中最大的整数值，可以根据平台和 Python 版本的不同而变化

# 11.获取和设置默认编码
sys.getdefaultencoding() 返回当前默认编码
sys.setdefaultencoding() 设置默认编码(一般不建议直接使用该方法)

# 12.获取 Python 解释器的字节顺序
sys.byteorder 	返回当前平台的字节顺序，大端序为 big，小端序为 little

# 13.获取当前的模块缓存
sys.modules 	是一个字典，记录了所有导入的模块名和模块对象

# 14.获取对象所占内存大小
sys.getsizeof(object) 	返回对象的字节大小

# 15.捕获未处理的异常
可以通过 sys.excepthook 自定义未处理异常的行为
```

#### time模块

​	`time` 模块提供了与时间相关的操作，支持时间戳、延时、格式化、计算时间差等功能

```py
# 1.获取当前时间戳
time.time()		返回当前时间的时间戳(自1970年1月1日以来的秒数)，类型为 float，精确到小数点后6位

# 2.延时操作
time.sleep(seconds)		使程序暂停执行指定的秒数（seconds 可以是浮点数），常用于控制程序运行节奏或定时任务

# 3.获取当前时间的结构化时间
time.localtime([secs])	将时间戳(secs)转换为本地时间的结构化时间(time.struct_time)，不传入参数则获取当前时间
time.gmtime([secs])		将时间戳转换为 UTC(协调世界时)的结构化时间

# 4.格式化时间
time.strftime(format, [t])	将结构化时间 t 格式化为字符串，不传入 t 则使用当前时间。format 指定输出格式
time.strptime(string, format)	将时间字符串 string 按照指定的 format 解析为结构化时间
# 常用格式化代码:
	%Y	# 年份
	%m	# 月份
	%d	# 日期
	%H	# 小时
	%M	# 分钟
	%S	# 秒数
    
# 5.获取当前时间的可读字符串
time.ctime([secs])	将时间戳转换为本地时间的字符串表示，不传入参数则获取当前时间

# 6.将结构化时间转换为时间戳
time.mktime(t)	将本地时间的结构化时间 t 转换为时间戳

# 7.计算程序运行时间
time.perf_counter()	返回高精度计时器值，适合测量代码运行时间(不随系统时钟变化)
time.process_time()	返回当前进程消耗的 CPU 时间，不包括等待时间

# 8.获取结构化时间的信息
结构化时间 time.struct_time 包含 9 个属性，可以通过以下方式访问
t = time.localtime()
print(t.tm_year)    # 年份
print(t.tm_mon)     # 月份
print(t.tm_mday)    # 日期
print(t.tm_hour)    # 小时
print(t.tm_min)     # 分钟
print(t.tm_sec)     # 秒
print(t.tm_wday)    # 星期几（0 表示星期一）
print(t.tm_yday)    # 一年中的第几天
print(t.tm_isdst)   # 是否为夏令时（-1, 0, 1）
```

#### datetime模块

​	`datetime` 模块在 Python 中用于处理日期和时间，提供比 `time` 模块更丰富的功能，如支持日期和时间的计算、时区管理、格式化等

```py
# 1.获取当前日期和时间
datetime.datetime.now()		返回当前本地日期和时间
datetime.datetime.utcnow()	返回当前 UTC 日期和时间# 被弃用使用datetime.now(timezone.utc)

# 2.创建指定日期和时间的对象
datetime(year, month, day, hour=0, minute=0, second=0, microsecond=0)	可以指定日期和时间创建 datetime 对象

# 3.获取当前日期
datetime.date.today()		返回当前日期，不包含时间

# 4.获取当前时间
datetime.datetime.now().time()	返回当前时间，不包含日期

# 5.日期和时间的格式化
datetime.strftime(format)	将 datetime 对象格式化为字符串。format 是指定格式的字符串
datetime.strptime(date_string, format)	将字符串解析为 datetime 对象

# 6.日期和时间的计算
datetime 		支持日期和时间的加减运算，通过 timedelta 对象来表示时间差

# 7.访问日期和时间的属性
可以通过以下属性访问 datetime 对象的年、月、日、时、分、秒等
print("年:", current_time.year)
print("月:", current_time.month)
print("日:", current_time.day)
print("小时:", current_time.hour)
print("分钟:", current_time.minute)
print("秒:", current_time.second)

# 8.日期和时间的比较
datetime 对象支持比较运算符，如 <, <=, >, >=, ==, !=，可用于判断两个日期时间的先后关系

# 9.处理时区
datetime 模块支持时区，通过 pytz 第三方库可以更方便地进行时区操作
```

#### random模块

​	`random` 模块用于生成随机数，并提供生成随机整数、随机浮点数、从序列中随机选择元素、随机打乱序列等功能。它主要用于随机选择、随机抽样、数据加密等用途

```py
# 1.生成随机整数
random.randint(a, b)	# 生成范围 [a, b] 之间的一个随机整数，包括 a 和 b
random.randrange(start, stop[, step])	# 在 range(start, stop, step) 序列中随机选择一个元素

# 2.生成随机浮点数
random.random()			# 生成一个 [0.0, 1.0) 之间的随机浮点数
random.uniform(a, b)	# 生成 [a, b] 之间的随机浮点数

# 3.从序列中随机选择
random.choice(seq)		# 从非空序列 seq 中随机选择一个元素
random.choices(population, weights=None, k=1)	# 从 population 中随机选择 k 个元素，允许重复选择，weights 可以指定每个元素被选中的权重

# 4.从序列中随机抽样
random.sample(population, k)	# 从 population 中随机选择 k 个不同的元素，不能重复

# 5.打乱序列
random.shuffle(x)		# 将序列 x 中的元素随机打乱，直接修改原序列

# 6.生成随机比特数和分布
random.getrandbits(k)	# 生成一个 k 位的随机整数

# 7.随机分布
random 模块提供了多种随机分布函数，如正态分布、指数分布等，常用于科学计算、统计分析等场景
	random.gauss(mu, sigma)		# 生成符合正态分布的随机浮点数，mu 是均值，sigma 是标准差。
	random.expovariate(lambd)	# 生成符合指数分布的随机浮点数，lambd 是分布的λ（λ是平均间隔	   时间的倒数）。
	random.uniform(a, b)		# 均匀分布生成的浮点数，前面已介绍。
	random.triangular(low, high, mode)	# 生成符合三角分布的随机浮点数，low 和 high 是范		围，mode 是众数位置。

# 8.随机种子
random.seed(a=None)			# 初始化随机数生成器种子，可设定为任意哈希值（如整数、字符串等），使生成的随机序列可重复。通常用于调试或测试
```

#### hashlib

​	`hashlib` 模块提供了常用的加密哈希函数，例如 `SHA-1`, `SHA-256`, `MD5` 等，用于生成不可逆的哈希值。这在密码存储、数据完整性验证等应用中非常常见

```py
# 1.常用哈希算法
# hashlib 提供了以下常用哈希算法：
MD5：md5()，生成 128 位哈希值，常用于校验和，但由于安全性较弱，适合非安全需求。
SHA-1：sha1()，生成 160 位哈希值，安全性较高但已被破解，不适合安全需求。
SHA-256：sha256()，生成 256 位哈希值，安全性更高，适用于密码和数据完整性。
SHA-512：sha512()，生成 512 位哈希值，适用于需要更高安全性的场景。

# 2.生成哈希值
# 每种哈希函数都支持以下方法：
update(data)	# 将 data 加入哈希计算，可多次调用。
hexdigest()		# 返回十六进制表示的哈希值（字符串格式）。
digest()		# 返回二进制表示的哈希值（字节格式）。

import hashlib
# 使用 SHA-256 生成哈希值
data = "Hello, World!".encode('utf-8')
hash_object = hashlib.sha256(data)
hex_dig = hash_object.hexdigest()
print("SHA-256 哈希值:", hex_dig)  # 输出：SHA-256 的十六进制哈希值

# 3.MD5 哈希值
# MD5 常用于非安全需求，如校验文件内容的完整性
md5_hash = hashlib.md5()
md5_hash.update(b"Example data")
print("MD5 哈希值:", md5_hash.hexdigest())

# 4.SHA-1 哈希值
# SHA-1 安全性高于 MD5，但不适合密码保护
sha1_hash = hashlib.sha1(b"Example data")
print("SHA-1 哈希值:", sha1_hash.hexdigest())

# 5.文件哈希值计算
# 计算文件的哈希值可以检测文件的完整性，防止传输过程中的数据篡改。可以逐步读取文件并更新哈希对象，以减少内存消耗
def hash_file(filepath, algorithm='sha256'):
    hash_obj = getattr(hashlib, algorithm)()
    with open(filepath, 'rb') as f:
        for chunk in iter(lambda: f.read(4096), b""):
            hash_obj.update(chunk)
    return hash_obj.hexdigest()

print("文件的 SHA-256 哈希值:", hash_file("example.txt"))

# 6.比较哈希值
# 验证数据完整性或密码时，直接将输入的哈希值与存储的哈希值进行比较
def hash_password(password):
    return hashlib.sha256(password.encode()).hexdigest()

def verify_password(stored_hash, password):
    return stored_hash == hash_password(password)

# 示例：生成并验证密码哈希
hashed_pw = hash_password("my_secure_password")
print("验证结果:", verify_password(hashed_pw, "my_secure_password"))  # 输出：True

# 7.HMAC：基于密钥的消息认证码
# 如果需要额外的安全性，可以使用 hmac 模块，该模块结合密钥和哈希算法生成 HMAC（适用于对称密钥加密）
```

#### 正则表达式

​	Python 的 `re` 模块用于处理正则表达式，可以高效地匹配、查找和操作字符串。正则表达式适合处理复杂的字符串查找和替换任务，比如匹配特定的文本格式或提取信息

```py
# 1.常用方法
re.match(pattern, string)		# 从字符串开头匹配 pattern。
re.search(pattern, string)		# 在整个字符串中搜索 pattern 的第一次匹配。
re.findall(pattern, string)		# 返回所有非重叠匹配结果的列表。
re.finditer(pattern, string)	# 返回一个迭代器，包含所有匹配的 Match 对象。
re.sub(pattern, repl, string, count=0)	# 将字符串中所有匹配 pattern 的部分替换为 repl，count 指定替换的最大次数（默认全部替换）。

# 2.匹配对象 Match
re.match、re.search 等方法返回 Match 对象（匹配成功时），包含匹配的详细信息：
	group()				# 返回整个匹配结果或指定组的子字符串。
	groups()			# 返回所有捕获组的元组。
	start()和end()	   # 返回匹配的起始和结束位置。
	span()				# 返回匹配位置的元组。
    
# 3.基本正则表达式语法
# 字符匹配：
	.	# 匹配除换行符外的任意字符。
    \d	# 匹配数字 [0-9]。
    \D	# 匹配非数字。
    \w	# 匹配字母、数字、下划线 [a-zA-Z0-9_]。
    \W	# 匹配非字母、数字、下划线。
    \s	# 匹配空白字符（空格、制表符、换行符等）。
    \S	# 匹配非空白字符。
# 数量修饰符：
    *	# 匹配前面的字符零次或多次。
    +	# 匹配前面的字符一次或多次。
    ?	# 匹配前面的字符零次或一次。
    {n}	# 匹配前面的字符恰好 n 次。
    {n,}# 匹配前面的字符至少 n 次。
    {n,m}# 匹配前面的字符 n 到 m 次。
# 边界匹配：
    ^	# 匹配字符串的开头。
    $	# 匹配字符串的结尾。
    \b	# 匹配单词边界。
    \B	# 匹配非单词边界。
# 分组：
    ()	# 用于分组或捕获匹配内容。
    |	# 逻辑“或”操作符。

# 4.使用 re.sub 进行替换
re.sub 	# 可以将字符串中匹配的内容替换为指定的字符串

# 5.使用 re.compile 编译模式
可以先编译正则表达式为一个 Pattern 对象，然后多次使用，提高效率

# 6.贪婪与非贪婪匹配
贪婪模式：默认匹配尽可能多的字符。
贪婪模式：默认匹配尽可能多的字符。
```

#### 多线程

​	Python 中的 `threading` 模块是标准库中的多线程工具，允许程序创建和管理多个线程，以提高并发性。`threading` 可以用于执行 I/O 密集型任务，比如文件读写、网络请求等

```py
# 1.创建线程
可以使用 threading.Thread 类创建和启动新线程

# 2.使用 Thread 子类创建线程
可以通过创建 Thread 的子类，将逻辑写在 run 方法中

# 3.常用的线程属性和方法
threading.current_thread()	返回当前线程对象。
threading.active_count()	返回当前活动线程的数量。
thread.name					线程名称，默认格式为 Thread-N。
thread.is_alive()			检查线程是否仍在运行。

# 4.线程锁(Lock)
线程锁 (Lock) 用于保证线程安全，防止多个线程同时访问共享资源。可以避免竞争条件，但会降低并发性能。

# 5.线程同步(Event)
Event 是一种用于线程之间的信号传递的同步原语。例如，可以在一个线程设置事件标志后，其他等待的线程才能继续执行

# 6.线程池(concurrent.futures 模块)
使用 concurrent.futures.ThreadPoolExecutor 可以创建线程池，从而管理多个线程的执行。适用于大量短时任务的并行执行

# 7. 定时器(Timer)
Timer 类允许在指定时间后执行一个函数或方法。适用于需要延时执行的任务

# 8. 线程间通信(Queue)
可以使用 queue.Queue 实现线程间的数据交换。Queue 是线程安全的，可以在生产者-消费者模式中使用
```

#### logging日志模块

​	Python 的 `logging` 模块提供了功能全面的日志记录功能，便于在控制台、文件或其他输出渠道中追踪代码执行过程。通过配置日志级别、格式、文件输出等选项，`logging` 可以帮助跟踪程序的运行情况、调试问题，并为未来的分析保留详细的日志记录

```py
# 1.logging.basicConfig() 可以快速设置日志的基本配置，通常用于简单的日志记录。
"""
import logging
logging.basicConfig(level=logging.INFO)  # 设置日志级别
logging.debug("调试信息")       		# 不会显示，因为级别低于 INFO
logging.info("普通信息")        		# 输出 INFO 信息
logging.warning("警告信息")     		# 输出 WARNING 信息
logging.error("错误信息")       		# 输出 ERROR 信息
logging.critical("严重错误信息")  		# 输出 CRITICAL 信息
"""

# 2.日志级别
logging 有五个预定义的日志级别，用于指定日志的严重性：
    DEBUG		调试信息，最低级别，常用于开发。
    INFO		常规信息，用于记录程序的运行状态。
    WARNING		警告信息，表示可能会有问题。
    ERROR		错误信息，表示程序遇到了无法正常运行的问题。
    CRITICAL	严重错误，表示程序可能无法继续运行。
默认情况下，logging 会记录 WARNING 及以上级别的信息

# 3.设置日志格式
logging 模块允许自定义日志格式，常见的日志内容包括时间、日志级别、文件名、行号、消息内容等
"""
import logging
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)
logging.debug("这是调试信息")
"""

# 4.输出日志到文件
可以将日志记录到文件中，以便持久保存或后续分析
'''
import logging

logging.basicConfig(
    filename="app.log",
    filemode="a",  # 追加模式（"w" 覆盖模式）
    level=logging.DEBUG,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

logging.info("将日志写入文件")
'''

# 5. 使用日志器（Logger）和处理器（Handler）
可以创建多个 Logger 实例，每个实例可以分别设置日志级别和输出格式；可以给 Logger 添加多个 Handler，让日志输出到不同的地方(例如，控制台和文件)
"""
import logging

# 创建日志器
logger = logging.getLogger("my_logger")
logger.setLevel(logging.DEBUG)

# 创建控制台处理器
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.WARNING)

# 创建文件处理器
file_handler = logging.FileHandler("my_log.log")
file_handler.setLevel(logging.DEBUG)

# 创建格式器并添加到处理器
formatter = logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s")
console_handler.setFormatter(formatter)
file_handler.setFormatter(formatter)

# 将处理器添加到日志器
logger.addHandler(console_handler)
logger.addHandler(file_handler)

# 记录日志
logger.debug("这是调试信息")
logger.warning("这是警告信息")
"""

# 6.使用 RotatingFileHandler 实现日志轮转
RotatingFileHandler 可以限制日志文件大小并在文件达到指定大小时自动创建新的日志文件，防止日志文件过大
"""
from logging.handlers import RotatingFileHandler

# 创建轮转文件处理器
rotating_handler = RotatingFileHandler("my_log.log", maxBytes=1024, backupCount=3)
rotating_handler.setFormatter(formatter)

logger.addHandler(rotating_handler)

# 记录大量日志以测试轮转效果
for i in range(100):
    logger.info(f"日志条目 {i}")
"""

# 7.配置字典（dictConfig）设置
可以用字典配置日志，这种方式更适合复杂的日志配置。以下示例定义了多个 Logger 和 Handler 配置
"""
import logging
import logging.config

log_config = {
    "version": 1,
    "formatters": {
        "default": {
            "format": "%(asctime)s - %(levelname)s - %(message)s",
        },
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "level": "DEBUG",
            "formatter": "default",
            "stream": "ext://sys.stdout",
        },
        "file": {
            "class": "logging.FileHandler",
            "level": "WARNING",
            "formatter": "default",
            "filename": "app.log",
        },
    },
    "root": {
        "level": "DEBUG",
        "handlers": ["console", "file"],
    },
}

logging.config.dictConfig(log_config)

logging.debug("调试信息")
logging.error("错误信息")
"""
```

#### resquests模块

​	`requests` 是一个非常流行的 Python 库，用于简化 HTTP 请求的操作。它可以方便地发送 GET、POST、PUT、DELETE 等请求，并支持 HTTP 身份验证、会话、文件上传等功能

```py
# 对象属性
status_code		# 返回 http 的状态码，比如 404 和 200(200 是 OK，404 是 Not Found)
text			# 返回响应的内容，unicode 类型数据
encoding		# 解码 text 的编码方式
apparent_encoding	# 编码方式
content			# 返回响应的内容，以字节为单位

# 异常
raise_for_status()	# 如果发生错误，方法返回一个 HTTPError 对象

ConnectionError		# 网络连接错误异常，如DNS查询失败、拒绝连接
HTTPError			# HTTP错误异常
URLRequird			# URL缺失异常
TooManyRedirects	# 超过最大重定向次数，产生重定向异常
ConnectTimeout		# 连接远程服务器超时异常
Timeout				# 请求URL超时，产生超时异常

# request主要方法
request()			# 向指定的 url 发送指定的请求方法
get()				# 发送 GET 请求到指定 url
head()				# 发送 HEAD 请求到指定 url
post()				# 发送 POST 请求到指定 url
put()				# 发送 PUT 请求到指定 url
patch()				# 发送 PATCH 请求到指定 url
delete()			# 发送 DELETE 请求到指定 url
```

##### 网络爬虫robots协议

​	遵守网络爬虫的协议、



#### BeautifulSoup模块

```py
# 基本元素
Tag					# 标签，最基本的信息元素组织单元，分别用<>和</>标明开头和结尾
Name				# 标签的名字，<p>...</p>的名字是'p'，格式：<tag>.name
Attributrs			# 标签的属性，字典形式组织，格式：<tag>.attrs
NavigableString		# 标签内非属性字符串，<>...</>中字符串，格式：<tag>.string
Comment				# 标签内字符串的注释部分，一种特殊的Comment类型

# 标签树的下行遍历
contents			# 子节点的列表，将<tag>所有儿子节点存入列表
children			# 子节点的迭代类型，与contents类似，用于循环遍历儿子节点
descendants			# 子孙节点的迭代类型，包含所有子孙节点，用于循环遍历

# 标签树的上行遍历
parent				# 节点的父亲标签
parents				# 节点的先辈标签的迭代类型，用于循环遍历先辈节点

# 标签树的平行遍历
next_sibling		# 返回按照HTML文本顺序的下一个平行节点的标签
previous_sibling	# 返回按照HTML文本顺序的上一个平行节点标签
next_siblings		# 迭代类型，返回按照HTML文本顺序的后续所有平行节点标签
previous_siblings	# 迭代类型，返回按照HTML文本顺序的前续所有平行节点标签

# 
prettify			# 更好的格式

# 拓展方法
find_all			# 方法返回所有匹配的元素，返回的是一个列表
find				# 搜索且只返回一个结果，字符串类型，同fnd all参数
find_parents		# 在先辈节点中搜索，返回列表类型，同find all参数
find_parent			# 在先辈节点中返回一个结果，字符串类型，同find参数
find_next_siblings	# 在后续平行节点中搜索，返回列表类型，同find all参数
find_next_sibling	# 在后续平行节点中返回一个结果，字符串类型，同find参数
find_previous_siblings # 在前序平行节点中搜索，返回列表类型，同find all参数
find_previous_sibling  # 在前序平行节点中返回一个结果，字符串类型，同find参数
```



