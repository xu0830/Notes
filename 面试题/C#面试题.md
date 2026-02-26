# C#

## 什么是CLR

1、负责控制 JIT 编译 IL 代码

2、垃圾回收

JIT 运行时编译

### 什么是托管代码和非托管代码

托管代码：在 CLR 环境下执行的代码

非托管代码： 不在 CLR 控制下执行的代码，如 C++代码 （[DllImport("user32.dll")]）

## 垃圾回收

垃圾回收器是一个持续运行的后台进程，可以清理任何类型的未使用的托管资源

值类型出现在线程栈中，每次调用都有线程栈，调用结束后自动释放

引用类型出现在堆里，全局就一个堆，所以需要垃圾回收

析构函数单独处理，释放非托管资源

### 触发垃圾回收条件

分配对象找不到可用空间 （A a = new A()）

分配量超过CLR初始化设置的阈值

收到物理内存不足的通知

直接调用GC.Collect（）

### GC可以清理非托管对象吗？

不能 

## 解释 CTS 的重要性

CTS Common Type System

确保.NET 支持的不同语言定义的数据类型都能编译成通用数据类型

## 解释CLS的重要性

Common Language Specification

通用语言规范。

遵循规范，可以在语言之间交叉互操作，在  C# 中 使用 VB.NET 代码

不适用区分大小写；不要使用指针；

CTS 处理不同数据类型

CLS 处理语言的不同行为

## 泛型

### 约束

#### 接口、抽象约束

where  T : IInterfaceA

#### 引用类型约束

where  T : class

#### 值类型约束

where  T : struct

#### 无参构造函数

where  T : new()

### 协变、逆变

用于接口委托

协变：

out 标记，代表了输出，只能作为结果返回



逆变:

in 标记，代表输入，只能被参数，不能作为返回值

```c#
//	协变
class Bird {}
class Sparrow : Bird {}
List<Bird> birdList = new List<Sparrow>();`//	编译不通过
IEnumerable<Bird> birdList = new List<Sparrow>();	//编译通过
IEnumberable<out T>

public interface ICustomerListOut<out T>
{
    T Get();
    //	void Show(T t); 编译不通过
}
    
//	逆变
ICustomerListIn<Sparrow> customerList1 = new CustomerListIn<Sparrow>();
ICustomerListIn<Sparrow> customerList2 = new CustomerListIn<Bird>();

public interface ICustomerListIn<in T>
{ 
	//	T Get();	编译不通过
    void Show(T t);
}     

public class CustomerListIn<T> ：ICustomerListIn<T>
{ 
	//	T Get();	编译不通过
    public void Show(T t){}
}
```

## 反射

获取运行时类型信息的方式，程序可以访问、检查或修改它本身状态或行为的一种能力

使用反射动态地创建类型的实例

优点

- 1、反射提高了程序的灵活性和扩展性。
- 2、降低耦合性，提高自适应能力。
- 3、它允许程序创建和控制任何类的对象，无需提前硬编码目标类。

缺点

1. 性能问题
2. 提高复杂度

### 破坏单例

### 泛型实例化

占位符

Type type = assembly.GetType("TestDll.TestType`2");

Type genericType = type.MakeGenericType(new Type[] {type(string), typeof(int)});

## 接口、抽象类 的区别

1. 接口不能有字段，抽象类可以有
2. 接口可以多继承

## 委托

类型

解耦，减少重复代码

多播委托不能异步调用，不能 BeginInvoke

多播委托查看调用链 GetInvocationList()

## 事件

带有 event 的委托实例

限制变量在外部被调用或 直接赋值

表达式目录树代替反射，提高性能

## 多线程

CPU计算速度高=快，执行太快，分时间片，上下文切换（加载环境，计算，保存环境）

微观： 一个核同一时刻只能执行一个线程；

宏观： 多线程并发

同步： 阻塞完成计算后，再进入下一行

异步： 不会等待方法的完成，会直接进入下一行，非阻塞

### 异常处理 

不阻塞情况下，无法捕捉异常

线程里的异常会被吞掉，脱离了try catch 范围

WatiAll 阻塞

### 线程取消

task 外部无法中止，线程是 OS 资源，无法掌控

CancellationTokenSource

```c#
CancellationTokenSource cts = new CancellationTokenSource();
调用cancel（）抛出取消异常
```

##  集合

数组： 内存上连续分配，坐标访问，读取快，增删慢

List： 不定长，内存连续分配，数组内部实现

LinkedList：不连续分配，不能下标访问，只能遍历，查找慢，增删快

适合添加删除频繁

Queue： 链表，先进先出，Enqueue，Dequeue

Stack： 先进后出

Set

HashSet

Hashtable

Dictionary： Hashtable 泛型版本

### 线程安全

ConcurrentQueue	线程安全版本的 Queue

ConcurrentStack  线程安全版本的 Stack

ConcurrentBag  线程安全的对象集合

ConcurrentDictionary  线程安全的Dictionary

BlockingCollection

## 字符串

### 不可变原因

享元模式，多个变量指向同一个字符串，字符串变化了，多个变量都会受到影响

堆的内存是连续分配的，改变长度，会导致大量数据移动

## switch 增强

```c#
var list = new List<object>(){1, "2"};
foreach(var item in list)
{
    switch(item)
    {
        //	当前 item 是 int 类型
        case int val:
            break;
        case string str when int.TryParse(str, out int ival):
            break;
        default:
            break;
    }
}
```

## 锁机制

### lock 关键字

语法糖，底层是用 Monitor 实现

基于 Monitor.Enter 和 Monitor.Exit

支持 可重入（同一线程可多次获取锁）

不支持超时设置

#### 建议

使用专用对象作为锁（避免锁字符串或 this）

保持锁内代码简短（减少锁持有时间）

```
private readonly object _lockObj = new object();
private int _sharedValue;

public void ThreadSafeMethod()
{
    lock (_lockObj) // 进入临界区
    {
        // 安全访问共享资源
        _sharedValue++;
    } // 退出临界区
}
```  

### Monitor

支持 TryEnter 带超时

可配合 Wait/Pulse 实现条件变量

```
object lockObj = new object();
bool lockTaken = false;

try
{
    Monitor.Enter(lockObj, ref lockTaken); // 更灵活的获取方式
    // 临界区代码
}
finally
{
    if (lockTaken) Monitor.Exit(lockObj);
}
```

### ReaderWriterLockSlim

读写分离

读多写少的并发场景

比简单 lock 性能更高

```
private readonly ReaderWriterLockSlim _rwLock = new ReaderWriterLockSlim();

public void ReadOperation()
{
    _rwLock.EnterReadLock();
    try
    {
        // 多个读取者可以同时进入
    }
    finally
    {
        _rwLock.ExitReadLock();
    }
}

public void WriteOperation()
{
    _rwLock.EnterWriteLock();
    try
    {
        // 一次只允许一个写入者
    }
    finally
    {
        _rwLock.ExitWriteLock();
    }
}
```

### SpinLock

自旋锁

不进行线程上下文切换

适合极短时间的锁保护（<1μs）

不可重入（同一线程重复获取会导致死锁）

```
private SpinLock _spinLock = new SpinLock();

public void SpinLockExample()
{
    bool lockTaken = false;
    try
    {
        _spinLock.Enter(ref lockTaken);
        // 短时间临界区
    }
    finally
    {
        if (lockTaken) _spinLock.Exit();
    }
}
```

### Interlocked

原子操作

Increment / Decrement

Add / Exchange

CompareExchange（CAS操作）

```
private int _counter;

public void Increment()
{
    Interlocked.Increment(ref _counter);
}

public int Read()
{
    return Interlocked.CompareExchange(ref _counter, 0, 0);
}
```

### Semaphore / SemaphoreSlim

```
// 允许最多5个线程同时访问
private SemaphoreSlim _semaphore = new SemaphoreSlim(5, 5);

public async Task AccessResource()
{
    await _semaphore.WaitAsync(); // 异步等待
    try
    {
        // 受保护的资源访问
    }
    finally
    {
        _semaphore.Release();
    }
}
```

### Mutex

跨进程锁

```
// 全局命名的Mutex（跨进程可用）
using var mutex = new Mutex(true, "Global\\MyAppMutex", out bool createdNew);

if (!createdNew)
{
    Console.WriteLine("另一个实例正在运行");
    return;
}

try
{
    // 应用程序代码
}
finally
{
    mutex.ReleaseMutex();
}
```

### AsyncLock 模式

```
public class AsyncLock
{
    private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1, 1);
    
    public async Task<IDisposable> LockAsync()
    {
        await _semaphore.WaitAsync();
        return new LockReleaser(_semaphore);
    }
    
    private struct LockReleaser : IDisposable
    {
        private readonly SemaphoreSlim _semaphore;
        public LockReleaser(SemaphoreSlim semaphore) => _semaphore = semaphore;
        public void Dispose() => _semaphore.Release();
    }
}

// 使用示例
private readonly AsyncLock _asyncLock = new AsyncLock();

public async Task ThreadSafeOperation()
{
    using (await _asyncLock.LockAsync())
    {
        // 异步临界区
        await Task.Delay(100);
    }
}
```

### Concurrent 集合 / Immutable 集合

Immutable 集合 不可变性

```
// 使用Immutable集合
private ImmutableDictionary<int, string> _safeDict = 
    ImmutableDictionary<int, string>.Empty;

public void AddItem(int key, string value)
{
    ImmutableInterlocked.Update(ref _safeDict, 
        (dict) => dict.Add(key, value));
}

// 使用Concurrent集合
private ConcurrentDictionary<int, string> _concurrentDict = new();

public void ConcurrentAdd(int key, string value)
{
    _concurrentDict.TryAdd(key, value);
}
```

## 锁场景

|     场景     |            推荐机制            |
| ----------- | ------------------------------ |
| 一般同步需求 | lock                           |
| 读多写少     | ReaderWriterLockSlim           |
| 极短临界区   | SpinLock                       |
| 跨进程同步   | Mutex                          |
| 资源池控制   | Semaphore/SemaphoreSlim        |
| 原子操作     | Interlocked                    |
| 异步环境     | AsyncLock 模式                 |
| 高性能无锁   | Concurrent 集合/Immutable 集合 |

	
	 
	
	
	
	
	
	
	

