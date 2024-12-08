### std::chrono

#### 1.时间间隔duration

`duration`表示一段时间间隔，用来记录时间长度，可以表示几秒、几分钟、几个小时的时间间隔。`duration`的原型如下
```c++
// 定义于头文件 <chrono>
template<
    class Rep,
    class Period = std::ratio<1>
> class duration;
```
- **Rep**: 这是一个数值类型，表示时钟数(周期)的类型(默认为整形)。若 `Rep` 是浮点数，则 `duration` 能使用小数描述时钟周期的数目
- **Period**: 表示时钟的周期，它的原型如下
```c++
// 定义于头文件 <ratio>
template<
    std::intmax_t Num,
    std::intmax_t Denom = 1
> class ratio;
```
`ratio`类表示每个时钟周期的秒数，其中第一个模板参数`Num`代表分子，`Denom`代表分母，该分母值默认为`1`，因此，`ratio`代表的是一个分子除以分母的数值

为了方便使用，在标准库中定义了一些常用的时间间隔，比如:时、分、秒、毫秒、微秒、纳秒，它们都位于`chrono`命名空间下，定义如下:
```c++
std::chrono::nanoseconds    /* 纳秒 */
std::chrono::microseconds   /* 微秒 */
std::chrono::milliseconds   /* 毫秒 */
std::chrono::seconds        /* 秒 */
std::chrono::minutes        /* 分钟 */
std::chrono::hours          /* 小时 */
```

#### 2.时间点time_point
`chrono`库中提供了一个表示时间点的类`time_point`，该类的定义如下
```c++
// 定义于头文件 <chrono>
template<
    class Clock,
    class Duration = typename Clock::duration
> class time_point;
```
它被实现成如同存储一个 `Duration` 类型的自`clock` 的纪元起始开始的时间间隔的值，通过这个类最终可以得到时间中的某一个时间点。
- **clock** :此时间点在此时钟上计量
- **Duration** :用于计量从纪元起时间的`std::chrono::duration` 类型

#### 3.时钟clocks
`chrono`库中提供了获取当前的系统时间的时钟类，包含的时钟一共有三种:
- **system_clock** :系统的时钟，系统的时钟可以修改，甚至可以网络对时，因此使用系统时间计算时间差可能不准
- **steady_c1ock** :是固定的时钟，相当于秒表。开始计时后，时间只会增长并且不能修改，适合用于记录程序耗时
- **high_resolution_clock** :和时钟类`steady_clock` 是等价的(是它的别名)。

在这些时钟类的内部有`time_point`、`duration`、`Rep`、`Period`等信息，基于这些信息来获取当前时间，以及实现`time_t`和`time_point`之间的相互转换。

|时钟类成员类型|描述|
|------------|---|
|rep|表示时钟周期次数的有符号算术类型|
|period|表示时钟计次周期的 std::ratio 类型|
|duration|时间间隔，可以表示负时长|
|time_point|表示在当前时钟里边记录的时间点|

#### 4.转换函数
1. **duration_cast**
`duration_cast` 是 `chrono` 库提供的一个模板函数，这个函数不属于 `duration` 类。通过这个函数可以对 `duration` 类对象内部的时钟周期 `Period` ，和周期次数的类型 `Rep` 进行修改，该函数原型如下:
```c++
template <class ToDuration, class Rep, class Period>
  constexpr ToDuration duration_cast (const duration<Rep,Period>& dtn);
```
- 如果是对时钟周期进行转换: 源时钟周期必须能够整除目的时钟周期(比如:小时到分钟)
- 如果是对时钟周期次数的类型进行转换: 低等类型默认可以向高等类型进行转换(比如:`int` 转 `double`)
- 如果时钟周期和时钟周期次数类型都变了，根据第二点进行推导(也就是看时间周期次数类型)
- 以上条件都不满足，那么就需要使用 duration_cast 进行显示转换,

2. **time_point_cast**
`time_point_cast` 也是 `chrono` 库提供的一个模板函数，这个函数不属于 `time_point` 类。函数的作用是对时间点进行转换，因为不同的时间点对象内部的时钟周期 `Period` ，和周期次数的类型 `Rep` 可能也是不同的，一般情况下它们之间可以进行隐式类型转换，也可以通过该函数显示的进行转换，函数原型如下:
```c++
template <class ToDuration, class Clock, class Duration>
time_point<Clock, ToDuration> time_point_cast(const time_point<Clock, Duration> &t);
```
> 注意事项：关于时间点的转换如果没有没有精度的损失可以直接进行隐式类型转换，如果会损失精度只能通过显示类型转换，也就是调用time_point_cast函数来完成该操作。