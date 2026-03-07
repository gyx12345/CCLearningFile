# C++ learning & project

# 编译器相关

**g++ -Ox 用于控制编译优化级别，其中x决定了编译器在生成代码时候进行多大程度的速度或空间优化。**

### 📊 核心优化级别详解

最常见和标准的级别如下表所示：

| 优化级别                   | 命令示例           | 含义与适用场景                                               |
| :------------------------- | :----------------- | :----------------------------------------------------------- |
| **`-O0`** (字母O后跟数字0) | `g++ -O0 main.cpp` | **默认级别，不进行任何优化**。编译最快，生成代码最易于调试，通常用于**开发调试阶段**。 |
| **`-O1`**                  | `g++ -O1 main.cpp` | **基础优化**。在保证编译速度的同时，尝试减少代码体积并提升执行速度。适用于追求快速构建和一定优化的场景。 |
| **`-O2`**                  | `g++ -O2 main.cpp` | **全面优化**（推荐级别）。启用绝大多数安全的优化选项，**大幅提升生成代码的执行速度**，是**发布版本的常用选择**。 |
| **`-O3`**                  | `g++ -O3 main.cpp` | **激进优化**。在 `-O2` 基础上，启用更激进、可能增加代码体积的优化（如循环展开）。适用于对计算性能要求极高的科学计算等场景。 |
| **-Os**                    | g++ -Os main.cpp   | 优化代码体积。在-O2的基础上，优先减少生成的可执行文件大小    |
| **-Og**                    | g++ -Og main.cpp   | **优化调试体验**。在提供 `-O1` 级别基本优化的同时，**最大限度地保留调试信息**，确保你可以舒适地进行源代码级调试。是 **`-O0` 的绝佳替代品**。 |

**注**：更高的优化级别（如 `-O2`, `-O3`）虽然能提升性能，但会**大幅延长编译时间**，且可能使**调试变得困难**（因为生成的机器码与源代码的行对应关系会被打乱）。对于你的 `atomic` 多线程项目，尤其要注意高优化级别可能对内存访问顺序产生的重排影响。







# 模板（TMP Template Metaprogramming）

## 引用折叠

### 规则

<img src="C:\Users\24144\AppData\Roaming\Typora\typora-user-images\image-20251221194924821.png" alt="image-20251221194924821" style="zoom:67%;" />

```c++
//情况1/2
typedef int& LIntRef;
LIntRef& ref; //int& & ref  --->引用折叠为 int&
LIntRef&& ref; //int& && ref  --->引用折叠为int&
//情况3/4
typedef int&& RIntRef;
RIntRef& ref;//int&& & ref--->折叠为int&
RIntRef&& ref;//int&& && ref--->折叠为int&&

```

### 完美转发

```c++
template<typename T>
constexpr T&& forward(std::remove_reference<T>::type& t)noexcept{//如果T是int& 那么static_cast<int& &&>(param) -> int&。
    return static_cast<T&&>(t);
}

template<typename T>
constexpr T&& forward(std::remove_reference<T>::type&& t)noexcept{//如果T是int&& 那么static_cast<int&& &&>(param) -> int&&
    return static_cast<T&&>(t);
}

void process(int& lval)
{
    std::cout << "lval process" << std::endl;
}

void process(int&& rval)
{
    std::cout << "rval process" << std::endl;
}

template<typename T>
void wrapper(T&& param)//param是一个万能引用
{
    process( my_forward<T>( param ) );//完美转发
}

//萃取 1.0 c++11前 萃取模板

int main()
{
    int a = 10;
    wrapper(a);//传入的是左值  T推导为int&  param的
    wrapper(20);//传入的是右值  T推导为int  param的类型是int&&

    return 0;
}
```





## 类型 萃取 （Trait）

用于在编译期间从复杂的类型如（函数、类、模板）中提取特定信息（如返回类型、参数类型、成员类型等）。

说白了就是利用编译期的功能，获取一些特定信息。

```c++

```

### std::invoke_result_t<F,Args...> 

```c++
//C++17 标准引入。 推导F的返回值，由于重载，要确定一个函数需要函数名以及参数Args
std::invoke_result_t<F,Args...>//F是函数类型， f是函数类型构造出的对象
    
int add(int a,int b){
    return a+b;
}

//推断函数add的返回类型：



```



# constexpr

用以说明变量： 必须要在编译期间确定值的常量

用以说明函数：编译期间可执行的函数

作用：

​    	性能提升：在编译期间进行操作。

​		编译期间安全（如果表达式无法在编译期间计算，直接编译报错，避免运行时错误）。

​		支持编译器编程（结合模板、类型萃取）

### 与const的区别

const在语义上：const 只读语义。用于修饰变量、成员函数或指针，表示“**不可修改**”。它主要强调**运行时只读**，但**不一定在编译时就能确定值**。

constexpr在语义上：编译期常量语义。表示“**可以在编译期求值**”。它要求变量或函数满足严格的编译期常量性条件，常用于数组大小、模板参数、`case` 标签等必须编译时常量的地方。



## static_assert(常量表达式，“可选错误信息”)

用于在编译期间检查一个常量表达式是否为true。







# 设计模式



## singleton







# 关键字

## #pragma once

非标准编译器指令，告诉预处理器：本文件在单次编译中应该被包含一次。

三、核心区别对比表

| 对比项               | `#ifndef`                                  | `#pragma once`                           |
| :------------------- | :----------------------------------------- | :--------------------------------------- |
| **标准性**           | C/C++ 标准，所有编译器支持                 | 非标准，但主流编译器普遍支持             |
| **宏名唯一性**       | 需要开发者保证，有冲突风险                 | 无需宏名，无冲突风险                     |
| **可移植性**         | 最佳                                       | 受限，可能不支持老旧/小众编译器          |
| **实现原理**         | 基于宏定义                                 | 基于文件系统路径/标识                    |
| **处理重复包含**     | 仍需打开文件直到发现宏已定义（现代有优化） | 可直接跳过，无需打开文件                 |
| **对符号链接等处理** | 不受影响，只要宏名相同即跳过               | 可能因路径不同而误判，但现代编译器已改进 |
| **代码简洁性**       | 需要额外几行代码，需手动维护宏名           | 单行指令，简洁明了                       |



## call_once

可以用在单例中表示只调用一次。

- **初始化复杂且可能失败**：例如需要打开文件、连接数据库，失败后希望程序能重试。
- **需要延迟初始化且初始化依赖外部参数**（但单例通常无参，可通过全局状态间接实现）。
- **需要更精细的生命周期控制**，比如在程序结束前手动销毁并重新创建（不常见）。
- **某些旧编译器不支持线程安全的静态局部变量**（但 C++11 后基本都支持）。

## =default





## =delete

告诉编译器，不要补全某些构造函数



## explicit

禁止隐式转换

这里的隐式包括：调用

```c++
    class lock_guard
    {
    public:
      typedef _Mutex mutex_type;

      [[__nodiscard__]]
      explicit lock_guard(mutex_type& __m) : _M_device(__m)
          
          
    }
```

## noexcept

noexcept关键字修饰函数 告诉编译器：这个函数不会抛出异常，**让编译器有更多的优化空间**。如果发生了异常应该立即调用std::terminate()终止程序。

### 必须使用noexcept的场景：

| 场景                               | 原因与示例                                                   |
| :--------------------------------- | :----------------------------------------------------------- |
| **1. 移动构造函数/移动赋值运算符** | 标准库容器（`vector`, `deque` 等）在重新分配内存时，**会根据 `noexcept` 决定使用移动还是拷贝**。如果移动操作可能抛出异常，容器会保守地选择更安全的拷贝操作，导致性能倒退。 |
| **2. 析构函数**                    | C++ 标准规定，**析构函数默认就是 `noexcept` 的**。如果你的析构函数可能抛出，必须显式声明为 `noexcept(false)`，但这被认为是极其糟糕的设计，因为析构函数中的异常极难安全处理。 |
| **3. 内存释放/交换函数**           | 如 `operator delete`, `swap` 函数。这些是基础操作，失败时应立即终止。`swap` 常用于提供强异常安全保证，如果它可能失败，这种保证就失效了。 |
| **4. 标准库的回调/自定义点**       | 如自定义的 `std::hash` 特化、分配器（`allocator::deallocate`）、谓词（Predicates）、比较函数（Comparators）等。标准库假设它们不会抛出。 |

### 推荐使用noexcept的场景

### ✅ **推荐使用 `noexcept` 的场景**

在这些情况下，使用 `noexcept` 能带来好处，但不是强制要求：

| 场景                              | 原因与示例                                                   |
| :-------------------------------- | :----------------------------------------------------------- |
| **1. 简单、显然不会失败的函数**   | 如 getter、setter、数学计算（已知输入范围时）。这给编译器更多优化空间。`int getValue() const noexcept { return value_; }` |
| **2. 性能至关重要的底层代码**     | 允许编译器生成更精简的代码（无需准备栈展开信息）。在紧密循环中调用时可能有微优化效果。 |
| **3. 明确失败不属于“异常”的函数** | 函数通过错误码、`std::error_code` 或返回状态来报告失败，而不是异常。`void log(const std::string& msg) noexcept; // 失败就写日志，不影响程序` |



### 禁止或避免使用noexcept的场景

| 场景                              | 原因与后果                                                   |
| :-------------------------------- | :----------------------------------------------------------- |
| **1. 函数实现可能抛出异常**       | 这是最危险的误用。如果你不确定函数及它调用的所有函数是否可能抛出，**宁可不用 `noexcept`**。误用会导致程序在应该恢复时直接崩溃。 |
| **2. 虚函数**                     | 如果基类的虚函数声明为 `noexcept`，那么所有覆盖它的派生类函数也自动是 `noexcept`（或更严格）。这限制了派生类的实现灵活性，除非你确定整个继承体系都不会抛出。 |
| **3. 库的公共接口（如果不确定）** | 对于提供给他人使用的库，除非接口契约明确规定了“永不抛出”，否则保留抛出异常的权利是为未来实现留出灵活性。 |







## std::future< T >  && std::function

## 一、基本概念对比

| 特性         | `std::future<T>`               | `std::function<ReturnType(Args...)>`   |
| :----------- | :----------------------------- | :------------------------------------- |
| **本质**     | 异步操作的**结果容器**         | 可调用对象的**包装器**                 |
| **存储内容** | 异步计算的结果值               | 函数、lambda、成员函数等可调用对象     |
| **主要操作** | `get()`, `wait()`, `valid()`   | `operator()`, `target()`, `bool()`转换 |
| **线程安全** | 通常线程安全（但具体实现依赖） | 不是线程安全的                         |
| **生命周期** | 与异步任务绑定                 | 独立的可调用对象容器                   |



## std::packaged_task<ReturnType(Args)> task(caller);

### 使用场景：

1、适用于需要分离任务创建和执行场景

2、线程池或任务调度系统

3、需要延迟执行但提前获取future的场景

### 不适合使用 `std::packaged_task` 的情况：

1. **简单异步调用**：使用 `std::async` 更简单
2. **只需要单向通信**：考虑使用 `std::promise` 和 `std::future`
3. **轻量级回调**：使用 `std::function` 或 lambda 更合适
4. **需要定期执行**：考虑使用定时器或事件循环



`std::packaged_task` 是 C++ 并发编程中的强大工具，特别适合构建复杂的异步任务系统。它提供了任务和结果之间的明确关联，同时给予开发者对执行时机的完全控制。



```c++
//future中的T表示，异步结果要返回的值的类型。
int a=10;
//这里int 是要返回的类型
std::future<int> fut=std::async(std::launch::async,[a](){ return a;})

 //函数包装器
//std::function<返回值类型(参数列表)>func    
    
std::packaged_task<返回值类型(参数列表)> task(可调用对象);
```



## std::promise

promise允许手动控制异步操作的结果（值或异常）。promise将结果的生产与消费完全分离

**使用`std::promise`当：**

1. 需要在**特定时刻**手动设置结果
2. 需要**跨线程传递异常**
3. 实现**自定义的异步模式**（如超时、取消）
4. 构建**复杂的事件驱动系统**
5. 需要**多个消费者**共享结果

**避免使用`std::promise`当：**

1. 简单的异步任务（用`std::async`）
2. 任务队列（用`std::packaged_task`）
3. 性能极敏感的场景（考虑更低级原语）

## 📊 `promise` vs `async` vs `packaged_task` 对比

| 特性         | `std::promise`           | `std::async` | `std::packaged_task` |
| :----------- | :----------------------- | :----------- | :------------------- |
| **控制粒度** | 完全手动控制             | 自动调度     | 半自动（包装函数）   |
| **结果设置** | 任意位置、任意线程       | 函数返回时   | 函数调用时           |
| **灵活性**   | 最高                     | 最低         | 中等                 |
| **异常处理** | 手动`set_exception`      | 自动传播     | 自动传播             |
| **适用场景** | 复杂异步流程、自定义同步 | 简单异步任务 | 任务队列、线程池     |





# RAII进一步理解

## 作用域 & 生命周期

RAII：资源定义即初始化，资源生命周期结束即析构。

比如stl 或其他对象，构造后，发生异常退出，没有RAII机制，那么资源如何释放？

c++中内置类型，未初始化变量存储在.BSS段。有自定义构造函数的类实例化的对象存储在数据段。

```c++
//需要注意的是 作用域与生命周期的区别
#include <iostream>
#include <

static int glob;//全局静态未初始化变量：生命周期直到进程终止，作用域是本文件

void func(){
    static int localV;//局部静态变量。生命周期是进程终止，作用域是本函数调用
}

class A{
 public:
    A(int a):m_a(a){}
 private:
    int m_a;
};

int main()
{
 	std::shared_ptr<A> sp{make_shared<A>(10)};
    
    std::assert();//在这里异常终止，根据RAII 生命周期结束，于是调用析构函数
    
    return 0;
}

```



<img src="C:\Users\24144\AppData\Roaming\Typora\typora-user-images\image-20260117150634658.png" alt="image-20260117150634658" style="zoom:67%;" />



# shared_ptr & weak_ptr & unique_ptr

weak_ptr是未来解决C++中的循环引用问题

```c++
class A{
  public:
  	std::shared_ptr<B> b_sp; 
};

class B{
  public:
  	std::shared_ptr<A> a_sp;  
};

int main()
{
    std::shared_ptr<A> a{std::make_shared<A>()};
    auto b=std::make_shared<B>();
    
    a->b_sp=b;
    b->a_sp=a;//存在循环引用
    return 0;
}



class A{
  public:
  	std::shared_ptr<B> b_sp; 
};

class B{
  public:
  	std::weak_ptr<A> a_sp;  
};

int main()
{
    std::shared_ptr<A> a{std::make_shared<A>()};
    auto b=std::make_shared<B>();
    
    a->b_sp=b;
    b->a_sp=a;//存在循环引用
    return 0;
}

```



## std::atomic<>

### 内存序

1、分为编译器/CPU重排之后，真实发生的读写顺序

2、另一个线程观察到的读写先后顺序（可见性顺序）。（**多cpu核或超线程**（多逻辑核心）写缓存cache，再使得cache line 一致性，两个步骤。容易发生不一致性）



### 原子操作的原理：

```c++
main:
.LFB2335:
        .cfi_startproc
        endbr64
        sub     rsp, 24
        .cfi_def_cfa_offset 32
        mov     edx, 1
        mov     rax, QWORD PTR fs:40
        mov     QWORD PTR 8[rsp], rax
        xor     eax, eax
        mov     BYTE PTR 6[rsp], 0
        mov     BYTE PTR 7[rsp], 0
        lock cmpxchg    BYTE PTR 6[rsp], dl  #原子CAS操作！
        sete    dl
        je      .L2
        mov     BYTE PTR 7[rsp], al
.L2:
        xor     edx, 1
        movzx   eax, dl
        mov     rdx, QWORD PTR 8[rsp]
        sub     rdx, QWORD PTR fs:40
        jne     .L6
        add     rsp, 24
        .cfi_remember_state
        .cfi_def_cfa_offset 8
        ret
.L6:
        .cfi_restore_state
        call    __stack_chk_fail@PLT
        .cfi_endproc
```

lock cmpxchg    BYTE PTR 6[rsp], dl

lock前缀：确保操作在多核处理器上的原子性。

cmpxchg：比较并交换指令。

BYTE PTR 6[rsp]：内存地址（stop_变量在栈上的位置）。

dl：要写入的新值。



**工作流程**（原子操作）：

1. 比较原子变量的当前值与 `expected` 值

2. 如果相等：将原子变量设置为 `desired` 值，返回 `true`

3. 如果不相等：用原子变量的当前值**更新** `expected`，返回 `false`

   

1. **`compare_exchange_strong(expected,true,std::memory_order_acq_rel)`**：原子地检查并修改`stop_`标志，确保只有一个线程能成功将其从`false`改为`true`
2. **`std::memory_order_acq_rel`**：
   - 成功时：确保**之前的所有操作**在`stop_=true`对其他线程可见**之前**完成
   - 失败时：获取当前值，建立必要的内存同步
3. **典型应用**：实现线程安全的单次触发、锁、状态机等

```c++
std::atomic<bool> stop;
bool expect=false;
stop.compare_exchange_strong(expect,true,std::memory_order_acq_rel);
```

### 内存序 & 无锁编程

C++根据使用场景 提供了6中内存顺序

```c++
1、memory_order_relaxed  //最宽松
只保证操作的原子性
不保证内存访问顺序
不提供线程同步
性能最好，适合计数器
std::atomic<FreeNode*>head=nullptr;
void push(){
head.load(std::memory_order_relaxed);//memory_order_relaxed是为了读取值时候head不会被其他线程中断。
}

2、memory_order_consume //废弃 deprecate 用memory_order_acquire
    
3、std::load(memory_order_acquire)
用于加载操作
确保该操作之后的所有读写不会被重排到该操作之前。
4、std::store(value,std::memory_order_release)
用于存储操作
确保该操作之前的所有读写不会被重排到该操作之后
与memory_order_acquire配对
    
5、memory_order_acq_rel
用于读-修改-写操作。(Rread modify write)
同时具有acquire和release语义。
操作前有release语义，操作后有acquire语义
    
6、memory_order_seq_cst
最强的内存序保证
所有线程看到的操作顺序一致
性能最差
如果不指定内存序，默认使用此顺序
    
```

| 特性         | `release`         | `acquire`        |
| :----------- | :---------------- | :--------------- |
| **使用操作** | 存储（store）操作 | 加载（load）操作 |
| **作用方向** | 向后的单向屏障    | 向前的单向屏障   |
| **主要目的** | 发布/释放数据     | 获取/消费数据    |
| **类比**     | 生产者打包发货    | 消费者收货检查   |
| **屏障位置** | 在关键操作之后    | 在关键操作之前   |

std::memory_order_release()： 我之前的写现在可以让别人看到了

std::memory_order_acquire()：我既然看到了，那我也必须看到你之前写的所有东西

acquire & release 等原子操作比mutex轻。

```c++
// release 屏障方向：阻止之前操作移到之后
// ↓ 之前的所有操作不能越过这个点向后移动 ↓
x = 1;                      // 操作A
y = 2;                      // 操作B  
atom.store(val, std::memory_order_release);  // 释放点
z = 3;                      

// acquire 屏障方向：阻止之后操作移到之前
// ↓ 之后的所有操作不能越过这个点向前移动 ↓
int val = atom.load(std::memory_order_acquire);  // 获取点
a = data1;                   // 操作D（不会移到load前面）
b = data2;                   // 操作E（不会移到load前面）

```



### 方法

1、存储写操作

```c++
void store(T desire,memory_order order=memory_order_seq_cst) noexcept;

```

2、加载读操作

```c++
void load();
```

3、比较交换

```c++
swap();
```

4、等待/通知 （C++20新增）

```c++

```



5、线程间屏障

```c++
std::atomic_thread_fence(std::memory_order_seq_cst);//全屏障

std::atomic_thread_fence(std::memory_order_acquire);//获取屏障

std::atomic_thread_fence(std::memory_order_release);//释放屏障

```

6、fetch_sub(1)

```c++
fetch_sub(1) 的原子执行过程：
1. 读取当前值 → Read
2. 计算新值（当前值 - 1） → Modify  
3. 写入新值 → Write
4. 返回读取的旧值。如果返回新值那么可能有值会被略过。
整个过程必须是原子的！不可被其他线程中断。
```



## std::this_thread::yield()

功能：提示函数，向操作系统调度器发出信号。主动让出CPU，运行其他线程被调度运行。

使用场景：

1、自旋锁：在尝试获取锁时，如果锁被占用，可以调用yield()让出CPU，避免忙等待消耗CPU。

2、忙等待：当等待某个条件时，如果不希望线程一直占用CPU，可以使用yield()让出CPU。

3、协作多任务系统中，线程让出CPU，以便其他线程运行。



与sleep_for的区别：

**yield() 只是让出当前时间片，线程可能立即被再次调度。**

sleep_for会让线程休眠一段指定时间，这段时间内线程不会被再次调用。


- 过度使用`yield()`可能会导致线程切换频繁，增加系统开销。
- 在某些情况下，使用条件变量或事件驱动模式可能比忙等待更高效。



### 对比表格

| 特性         | `yield()`  | `sleep_for()` | `sleep_until()`  |
| :----------- | :--------- | :------------ | :--------------- |
| **目的**     | 让出时间片 | 暂停指定时间  | 暂停到指定时间点 |
| **精度**     | 调度器决定 | 相对较精确    | 绝对精确         |
| **唤醒**     | 立即可调度 | 定时唤醒      | 定时唤醒         |
| **CPU占用**  | 几乎为0    | 0             | 0                |
| **使用场景** | 忙等待优化 | 定时任务      | 精确时间控制     |



## alignas(64) 

需要区分**结构体内占位置大小**与**结构体对齐**

```c++
对齐要求（Alignment）
1、不加 alignas：结构体的对齐由其成员的最大自然对齐决定。
2、加 alignas(64)：强制要求整个结构体对象的起始地址必须是 64 字节的倍数。
结构体大小（size）    
不加alignas: 大小等于各成员大小和加上可能的内部填充。
加 alignas(64)：在满足了所有成员对齐后，如果结构体整体大小不是 64 的倍数，编译器会在末尾追加填充（padding）直到大小是 64 的倍数。确保在数组中，每个元素的起始地址仍满足64对齐
```

C++并发编程的重要工具之一，特别适用于需要极致性能的应用程序。

64字节是大多数X86/X64 CPU的缓存行的大小。空间换时间优化策略。

alignas确保结构体在内存中的地址是对齐的。

使用场景：

​	高频并发访问的数据结构。

​	多核CPU下的分片算法。

​	性能关键服务器应用

​	金融交易、游戏服务器等低延迟系统。

```c++
struct alignas(64) ShardQueue {
    std::mutex mtx;        // 互斥锁，通常4-8字节
    std::deque<Task> dq;   // 双端队列，通常24-40字节（取决于实现）
};

// 使用示例：
std::vector<ShardQueue> queues(4);  // 4个分片队列


// ❌ 没有对齐的情况（可能出问题）：
struct BadQueue {
    std::mutex mtx;
    std::deque<Task> dq;
};

BadQueue queues[4];  // 4个队列在内存中可能紧挨着

// 假设：
// CPU0 频繁访问 queues[0].mtx（写锁/解锁）
// CPU1 频繁访问 queues[1].mtx
// 如果它们在同一个64字节缓存行中...
// 一个CPU的修改会导致另一个CPU的缓存行无效化！
```



# std::mutex

std::mutex既不能拷贝也不能移动。

不能拷贝原因简单：因为锁只有同一个才能互斥。

不能移动：1、其他线程可能通过这个地址来访问，如果移动会导致地址失效。

​					2、所有权模糊



## 📊 与其他资源管理类的对比

| 类型                                    | 可否拷贝 | 可否移动 | 原因                                                         |
| :-------------------------------------- | :------- | :------- | :----------------------------------------------------------- |
| `std::mutex`, `std::condition_variable` | ❌ 否     | ❌ 否     | **身份标识**，地址必须稳定，多个线程依赖此地址访问同一同步对象。 |
| `std::thread`                           | ❌ 否     | ✅ 是     | 代表执行流的所有权。移动操作将执行流的管理权移交，**原对象变为“空”状态**（不关联任何线程），这是明确且安全的。 |
| `std::unique_ptr<T>`                    | ❌ 否     | ✅ 是     | 代表对堆内存的独占所有权。移动移交所有权，**源对象明确变为`nullptr`**。 |
| `std::shared_ptr<T>`                    | ✅ 是     | ✅ 是     | 代表共享所有权，通过引用计数安全管理。拷贝增加计数，移动则移交所有权。 |
| `std::vector<T>`                        | ✅ 是     | ✅ 是     | 纯数据容器，持有数据副本。移动是高效的数据指针转移。         |



## lock_guard<mutex>

```c++
 template<typename _Mutex>
    class lock_guard
    {
    public:
      typedef _Mutex mutex_type;

      [[__nodiscard__]]
      explicit lock_guard(mutex_type& __m) : _M_device(__m)//禁止隐式转换
      { _M_device.lock(); }

      [[__nodiscard__]]
      lock_guard(mutex_type& __m, adopt_lock_t) noexcept : _M_device(__m)
      { } // calling thread owns mutex

      ~lock_guard()
      { _M_device.unlock(); }

      lock_guard(const lock_guard&) = delete;// prevent copying
      lock_guard& operator=(const lock_guard&) = delete;// 禁止拷贝赋值

    private:
      mutex_type&  _M_device;
    };
```



# 无锁编程

## 适用无锁编程的场景

性能瓶颈明确：锁竞争成为系统瓶颈

实时性要求高：不能容忍线程阻塞

有成熟的实现可用：如boost::lockfree::queue

专家级团队：有能力验证正确性

**适合无锁的场景：**

- 简单的计数器、标志位

- 高性能队列（如Disruptor模式）

- 实时系统核心组件

  

## 适用锁编程的场景

开发效率优先：需要快速实现

逻辑复杂：需要保护复杂的数据结构

锁竞争不激烈：锁冲突概率低

团队成员不熟悉无锁编程

- 临界区执行时间长
- 并发度不高
- 需要保护复杂的数据结构
- 代码可读性和可维护性要求高





# TLS与ThreadLocal变量

**TLS：ThreadLocal storage Area**

为什么需要TLS区域：

​		1、栈生命周期太短（调用结束后就stack pop）

​		2、全局区域太共享（许多线程都能读取，需要加锁互斥访问）

## ① `errno`（TLS 诞生的直接原因之一）

```
open(...);
if (errno == EINVAL) { ... }
```

如果 `errno` 是全局变量：

```
线程 A: open() 失败 → errno = EINVAL
线程 B: open() 成功 → errno = 0
线程 A: 读 errno → 0  ❌
```

👉 所以 **`errno` 必须是每线程一份**
 👉 但又必须是“全局可见符号”

📌 现实：`errno` 就是 TLS

------

## ② 线程运行时上下文

比如：

- 线程 id / worker id
- 当前调度器
- 当前线程的随机数引擎
- per-thread cache

这些都：

- 不属于某一次函数
- 不该被别的线程看到
- 不想加锁



<img src="C:\Users\24144\AppData\Roaming\Typora\typora-user-images\image-20260127224427928.png" alt="image-20260127224427928" style="zoom:60%;" />



## TLS 存在的区域（.elf静态文件 & 运行时内存）

| 区域          | TLS 数据是否在这里 | 说明                                                 |
| ------------- | ------------------ | ---------------------------------------------------- |
| kernel        | ❌                  | 内核不存 TLS 数据，只在切线程时保存 FS/GS            |
| stack         | ❌                  | TLS 不是栈变量，不随函数调用变化                     |
| heap          | ❌（概念上）        | TLS 可能通过 malloc 获得，但**不属于普通 heap 管理** |
| `.text`       | ❌                  | 代码段                                               |
| `.data`       | ❌（实例不在）      | 只存 TLS **初始化模板**                              |
| `.bss`        | ❌（实例不在）      | 只存 TLS **零初始化模板**                            |
| ELF TLS 段    | ✅（模板）          | `.tdata` / `.tbss`                                   |
| 运行时 TLS 区 | ✅（实例）          | 每线程私有，用户态内存                               |

```c++
thread_local int x=42;
thread_local int y;

.tdata  ---> thread_local //有有初始值的变量（x=42）
.tbss  ---->thread_local //零初始化向量（y）

```

数据要分为.data与.bss 为了节省磁盘、加快程序加载速度、优化内存使用。

### 📊 核心区别一览

| 特性                   | **`.data` 节 (已初始化数据)**                    | **`.bss` 节 (Block Started by Symbol)**                      |
| :--------------------- | :----------------------------------------------- | :----------------------------------------------------------- |
| **内容**               | 所有初始值**非零**的全局/静态变量。              | 初始值为**零**或**未显式初始化**的全局/静态变量。            |
| **在磁盘文件中的大小** | **有实际大小**，存储变量的初始值字节。           | **理论上接近零**，只存储一段长度的描述信息。                 |
| **在内存中的大小**     | 与磁盘中相同，加载时直接映射。                   | 在**加载时或运行时**向操作系统申请相应大小的内存，并**填充为零**。 |
| **主要目的**           | **保存数据**：存储程序启动时必须具备的初始状态。 | **预留空间**：预定一块内存区域，并确保其初始状态为确定值（零）。 |



## thread_local 

thread_local 是C++让我声明并使用TLS的标准语法

### ❌ 误区 1：`thread_local` 是 OS 提供的

❌ 错
 ✔ 是 **语言级关键字**，OS 只是提供“每线程存储”的基础设施

------

### ❌ 误区 2：TLS 和栈是一回事

❌ 错
 ✔ TLS 有独立布局，生命周期是线程级

------

### ❌ 误区 3：`static thread_local` 和 `thread_local` 不一样

❌ 错（在函数内几乎等价）
 ✔ `static` 只影响**链接/作用域**，不影响 TLS 语义



## thread_local 使用场景

1、**保存线程上下文信息（thread id，worker编号）**。

​		无锁、性能高、跨函数、跨库访问。

```c++
thread_local int worker_id=-1;
```

2、**errno 错误状态**

​		POSIX标准级需求。如果没有TLS，多线程根本不可用。

​		这是ThreadLocal storage Area被提出的原因。

3、**每线程缓存**

​		**减少malloc/free**

​		减少锁竞争

​		避免 false sharing

常用于：

​	内存分配器、日志缓冲、编解码

​	

```c++
//关于减少malloc/free

thread_local std::string buf;
buf.clear(); //逻辑清空，实际不释放堆内存、不调用free、内存还在
buf.reserve(4096);    // 复用之前申请过的容量

```

##  如果不用这种写法会发生什么？

### ❌ 每次新建 string

```c++
std::string buf;
buf.reserve(4096);
/*
每次调用：
新建对象
很可能每次都 malloc
用完析构 → free
下次再 malloc
👉 频繁 malloc/free
*/
```



# perf火焰图分析系统瓶颈



```bash
# 宏观统计：运行程序并收集关键性能计数器
perf stat -e task-clock,context-switches,cpu-migrations,page-faults ./testperf
# 微观热点：记录性能数据用于生成火焰图 生成perf.data
perf record -F 99 -g --call-graph dwarf ./testperf  2 1
# 生成并查看火焰图 (确保在build目录下)
perf script > out.perf     #生成了这一步可以使用https://ui.perfetto.dev/ 查看分析


~/FlameGraph/stackcollapse-perf.pl out.perf > out.folded
~/FlameGraph/flamegraph.pl out.folded > flamegraph.svg
```



# valgrind 检测内存泄露

asan & kasan 扫描内存泄露
能检测到运行时候的内存情况。虽然最后都有系统回收，那样不是不存在泄露了？------ 我们讨论的是运行时候的内存泄露情况。

```shell
# 编译，添加 -g 选项生成调试信息
gcc -g -o test test.c
# 使用 Valgrind 的内存检查工具运行程序
valgrind --tool=memcheck --leak-check=full ./test
```



# performance分析

## p50 & p99

p是percentage。
eg：p99=500 表示99%的样本不超过500

不超过P99的样本占99%

## 设置CPU亲和性

### 什么是调度漂移

线程不是一启动就永远固定在某个核心上跑。操作系统会根据负载不断调度它：

- 这个线程先在 CPU 1 跑一会儿
- 然后被抢占
- 过一会儿恢复时，可能跑到 CPU 4
- 再过一会儿又被迁移到 CPU 7

这种“运行核心和运行时机不断变化”的现象，就可以理解成调度漂移。

它主要包含两类事：

#### 1) 时间上的漂移

线程本来连续执行，结果被中途打断：

- 时间片用完
- 被更高优先级线程抢占
- 主机上还有别的任务在跑

于是一次操作本来只要几百纳秒，结果中间被 OS 插了一脚，测出来可能变成几十微秒甚至更高。

#### 2) 空间上的漂移

线程本来在一个核上跑，后来被迁移到另一个核上。
 这会影响：

- cache 命中率
- TLB
- NUMA 局部性
- cache line 所有权转移

所以即使代码没变，测出来的性能也会抖。

绑定CPU核心 避免 调度漂移







# protobuf与android AIDL 序列化的联系：

| **特性**       | **Android 原生 (Parcelable + AIDL)**                       | **Google 生态 (Protobuf + gRPC/Utility)**                    |
| -------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| **设计初衷**   | 专为 Android **手机本地**跨进程通信优化。                  | 专为**全球网络**、异构系统的高性能通信设计。                 |
| **核心协议**   | **AIDL (Android Interface Definition Language)**           | **`.proto` (Protocol Buffer 定义文件)**                      |
| **序列化接口** | 实现 `Parcelable` 接口。                                   | 使用生成的 Proto Message 类。                                |
| **二进制格式** | **TLV-like**, 但侧重于**顺序解析**，依赖两端代码严格一致。 | **严格的严格的 TLV (Tag-Length-Value)**，极其紧凑，且具有自我描述性。 |
| **主要优势**   | **零反射，极速**，直接与 Android 驱动集成。                | **跨语言、跨平台**；**极强的向后/向前兼容性**。              |
| **主要劣势**   | 跨平台维护困难；版本兼容性脆弱（新加字段很难处理）。       | 需要引入 Protobuf 运行时 SDK；APK 体积会略微增加。           |



# yeild & sleep & spin

## 三者的对比

| 行为         | 自旋（spin）                 | yield                            | sleep                              |
| :----------- | :--------------------------- | :------------------------------- | :--------------------------------- |
| **CPU 占用** | 一直占用 CPU，空转           | 暂时让出 CPU，但很快可能又运行   | 不占用 CPU，完全让出               |
| **线程状态** | 运行态（running）            | 运行态 → 就绪态（ready）         | 运行态 → 睡眠态（sleeping）        |
| **响应速度** | 最快（一旦锁释放，立即获取） | 较快（取决于调度器何时重新调度） | 慢（必须等待休眠结束或被唤醒）     |
| **适用场景** | 锁持有极短，多核，竞争不激烈 | 锁持有较短但可能偶尔长，负载高   | 锁持有较长，或希望线程休眠一段时间 |
| **缺点**     | 浪费 CPU 资源                | 可能频繁调度，效果不稳定         | 延迟大，不适合短等待               |



# 零拷贝是传左值引用&右值引用&还是const 左值引用？

## 构造核心原则：看实参是左值还是右值

### 如果有参数需要构造，改调用什么构造：

实参是左值  →  调用拷贝构造
实参是右值  →  调用移动构造

### 那么什么时候需要构造对象呢？

看声明的是值类型还是引用类型

```c++
vector<int> v  = ...;   // 值类型  → 需要构造新对象
vector<int>& v  = ...;  // 左值引用 → 不构造，绑定
vector<int>&& v = ...;  // 右值引用 → 不构造，绑定
```

std::move 返回的是一个右值引用类型。移动构造，才是正真的移动操作。

```c++
//std::move() 返回 的是右值引用 &&  
template<typename _Tp>
    [[__nodiscard__,__gnu__::__always_inline__]]
    constexpr typename std::remove_reference<_Tp>::type&&
    move(_Tp&& __t) noexcept
    { return static_cast<typename std::remove_reference<_Tp>::type&&>(__t); }

```

下面举个例子来分析一下传哪些值

```c++
void processA(std::vector<int> v){
    ...
}

void processB(const std::vector<int> &v){
    ...//传const &，不能更改。
}

void processC(std::vector<int> &v){
    ....//传左值语义不明确，容易被更改了，不知道哪里改的。
}

void processD(std::vector<int> &&v){//这里是右值引用的赋值，没有调用 移动构造函数。
    ...//
}

int main(){
    vector<int> v1;
	processA(std::move(v1));//这里是否要构造对象？需要，被赋值的是值类型。 怎么构造对象？实参是右值类型，调用移动构造函数。
	processB(v1);//这里是否要构造对象？不需要，被赋值的是const 引用类型。
    processC(v1);//这里是否要构造对象？不需要，被赋值的是左值引用类型。
    processD(std::move(v1));//这里是否需要构造对象？不需要，被赋值的是右值引用类型。
}
```

所以结论是 传右值引用（实参）给，常量（形参），减少拷贝，直接移动。





