# C#



## 关键字

### this

1、代表当前类的实例对象

2、串联构造函数，this( )

3、为原始类型扩展方法



进程堆 （进程唯一）

线程栈（每个线程一个）

任何引用类型的值都在堆里

引用类型的值类型保存在堆中



继承方法调用

BaseClass baseClass = new DerivedClass();

baseClass.CallMethod();

普通方法编译时决定，调用父类方法

抽象/虚方法运行时决定，调用子类方法



### is

模式匹配

```c#
static bool IsFirstSummerMonday(DateTime date) => date is { Month: 6, Day: <=7, DayOfWeek: DayOfWeek.Monday };
```

将表达式与 `null` 匹配时，编译器保证不会调用用户重载的 `==` 或 `!=` 运算符

```c#
if (input is null)
{
    return;
}

```

C# 9.0c

否定模式 

```c#
if (result is not null)
{
    Console.WriteLine(result.ToString());
}
```



### as

不能操作基元类型

如果类型之间都上溯到了某个共同的基类，

那么根据此基类进行的转型（即基类转型为子类本身），应该使用 as

子类与子类之间的转型，则应该提供转换提示符，以便进行强制类型转换

#### 强制类型转换

```c#
public class MyType
{

}

public class SecondType
{
    public static explicit operator MyType(SecondType secondType)
    {
    	return new MyType();
    }
}

public static class Factory
{
    public static object GetObject()
    {
        return new SecondType();
    }
}

class Program
{
    public static void Main(string[] args)
    {
        var o = Factory.GetObject();
        
		/*	抛出异常 InvalidCastException， 用户定义的类型转换逻辑针对的是源对象的编译期类型，而不是			   实际类型
  			编译期类型是 object，编译期会把它当成 object 看待，不考虑其在运行期的类型，不理会自定义的			转换逻辑
  			编译期考虑的是对象 o 的编译期类型与目标类型 MyType 之间有没有转换逻辑
  			需要先把 o 转换为 SecondType: 
        */
        
        //SecondType st = o as SecondType;
        //MyType mt = (MyType)st;
        MyType mt = (MyType)o;   
    }
}
```

### const

编译器变量，不能用 static 修饰，天然是 static，编译后直接用实际值替代

只能修饰基元类型，枚举类型货字符串

### readonly

运行时常量，运行时第一次复制后不可改变

值类型，值本身不可改变

<font color='red'>引用类型，引用本身（指针）不可改变</font>

可修饰的数据类型无限制

### lock

语法糖，等价于

```c#
Monitor.Enter(obj);//必须是引用类型，将引用堆上的block改成1，标识占据，0才能改，1等待
try
{ 
	//	动作代码
}
finally
{
    Monitor.Exit(obj)
}    

```

lock 同一个变量，互斥

lock 不同变量，并发

lock(this) 

避免使用this，外部可能使用这个实例，发生冲突

在一个线程锁定对象之后导致整个对象无法被其他线程访问

在使用lock的时候，被lock的对象(locker)一定要是引用类型的，如果是值类型，将导致每次lock的时候都会将该对象装箱为一个新的引用对象(事实上如果使用值类型，c#编译器在编译时会给出个错误）

lock 不要锁字符串（字符串不可变性）

```c#
string lockStr = "test";
string lockStr1 = "test";
lock(lockStr)
{
}
lock(lockStr1 )
{
}
```

泛型 lock

### new

初始化

```c#
Program g = new();
```

显式隐藏从基类继承的成员

### virtual

用于修改方法、属性、索引器或事件声明，使它们可以在派生类中被重写

### record

c# 9.0

定义一个引用类型，提供用于封装数据的内置功能，可创建具有不可变属性的记录类型

### Not

## foreach

自动将代码置于 try-finally 块中

若类型实现了 IDisposable 接口，循环结束后自动调用 Dispose（）

不支持循环时对集合进行增删操作

迭代器内部维护了一个对集合版本的控制

对集合的增删操作会使集合版本加一

MoveNext 方法内部会进行集合版本检测

for循化、索引器、foreach循环、迭代器，都是对内部数组的访问



## 运算符

### ！

#### null 包容

https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/operators/null-forgiving

C# 8.0 及更高版本中可用

声明可为空的引用类型的表达式 x 不为 null， 抑制编译器不提示警告 Warning CS8602: Dereference of a possibly null reference

[NotNullWhen]

### @

1、使 C# 关键字用作标识符

2、指示将原义解释字符串

3、使编译器在命名冲突的情况下区分两种属性

```c#
public class Info : Attribute {}

public class InfoAttribute : Attribute {}
```



### ?? 

如果左操作数的值不为 `null`，则 null 合并运算符 `??` 返回该值；否则，它会计算右操作数并返回其结果

如果左操作数的计算结果为非 null，则 `??` 运算符不会计算其右操作数

### ??=

空合并赋值运算符 

C# 8.0 及更高版本中可使用

仅在左侧操作数的求值结果为 `null` 时，才将其右侧操作数的值赋值给左操作数。 如果左操作数的计算结果为非 null，则 `??=` 运算符不会计算其右操作数

### ::

使用 [using 别名指令](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/using-directive)创建的命名空间别名：

```c#
using forwinforms = System.Drawing;
using forwpf = System.Windows;

public class Converters
{
    public static forwpf::Point Convert(forwinforms::Point point) => new forwpf::Point(point.X, point.Y);
}
```

## with表达式

 C# 9.0 及更高版本中可用

使用修改的特定属性和字段生成其[记录](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/record)操作数的副本

```c#
using System;
public class WithExpressionBasicExample
{
    public record NamedPoint(string Name, int X, int Y);
    public static void Main()
    {
        var p1 = new NamedPoint("A", 0, 0);
        Console.WriteLine($"{nameof(p1)}: {p1}");  // output: p1: NamedPoint { Name = A, X = 0, Y = 0 }

        var p2 = p1 with { Name = "B", X = 5 };
        Console.WriteLine($"{nameof(p2)}: {p2}");  // output: p2: NamedPoint { Name = B, X = 5, Y = 0 }

        var p3 = p1 with 
        { 
            Name = "C", 
            Y = 4 
        };
        Console.WriteLine($"{nameof(p3)}: {p3}");  // output: p3: NamedPoint { Name = C, X = 0, Y = 4 }

        Console.WriteLine($"{nameof(p1)}: {p1}");  // output: p1: NamedPoint { Name = A, X = 0, Y = 0 }
    }
}
```


## switch表达式

```c#
public enum Direction
{
    Up,
    Down,
    Right,
    Left
}

public static string ToOrientation(Direction direction) => direction switch
{
	Direction.Up    => "North",
	Direction.Right => "East",
	Direction.Down  => "South",
	Direction.Left  => "West",
   	//	弃元模式
	_ => throw new ArgumentOutOfRangeException(nameof(direction), $"Not expected direction value: {direction}"),
};

```



<font color='red'>[MethodImpl] ? </font>

## string

**引用类型**

值不可变

在内存分配时，使用享元模式，堆里面一个值，一般只出现一次 

同一进程赋值时，相同字符串指向同一内存地址

拼装例外

拼装字符串时不能使用享元模式

= 赋值运算符重载，赋值时先查找当前进程内存中是否存在相同字符串，

若存在则直接将引用指向该内存地址，否则新开辟内存空间，存储新字符串

### 不可变的好处



### 标准数字格式字符串

区域名称

https://docs.microsoft.com/zh-cn/openspecs/windows_protocols/ms-lcid/a9eac961-e77d-41a6-90a5-ce1a8b0cdb9c

zh-CN

zh-SG|en-SG新加坡

 zh-HK|en-HK香港

zh-MO澳门

zh-TW台湾

 en-AU澳大利亚

en-CA加拿大

en-150欧洲

en-DE德国

en-ZA南非

en-GB英国

en-US美国

fr-FR法国

| 格式说明符 | 属性           | 说明                                                         |
| ---------- | -------------- | ------------------------------------------------------------ |
| C或c       | 货币           | 123.456.ToStrng("C", CultureInfo.GetCultureInfo("en-US")) ->$123.46<br /><br />CultureInfo.CurrentCulture = new CultureInfo("en-US", false);<br />string.Format("{0:X}", -1)  -> FF |
| D或d       | 十进制         | 精度说明符：所需数字最小位数，左补零<br />1234 ("D") -> 1234<br />-1234 ("D6") -> -001234 |
| E或e       | 指数（科学型） | 1052.0329112756 ("E", en-US) -> 1.052033E+003                |
| F或f       | 定点           | 1234.567 ("F", en-US) -> 1234.57                             |
| G或g       | 常规           | 更紧凑的定点表示法或科学记数法<br />-123.456 ("G", en-US) -> -123.456 |
| N或n       | 数字           | 1234.567 ("N", en-US) -> 1,234.57                            |
| P或p       | 百分比         | 1 ("P", en-US) -> 100.00 %                                   |
| R或r       | 往返过程       | 123456789.12345678 ("R") -> 123456789.12345678               |
| X或x       | 十六进制       | 只有整型才支持此格式<br />255.ToStirng("X") -> FF       <br />string.Format("{0:X}", -1)  -> FF |

### StringBuilder 

AppendFormat("{0:C1}", 1));

## 通用类型系统

.NET 使用**通用类型系统（CTS）**定义可以在中间语言（IL）中使用的预定义数据类型

## 数据类型

所有类型都继承自 object

进程堆 （进程唯一）

线程栈（每个线程一个）

### 值类型

隐式继承 System.ValueType

内置值类型、用户定义的值类型、枚举

隐式派生自 ValueType

struct 直接派生自 ValueType

声明 int 变量，声明的实际上是 CTS 中 System.Int32 的一个实例

意义

1、确保 IL 上的强制类型

2、实现了不同 .NET 语言的互操作性

3、所有的数据类型都是对象。可以有方法，属性等

### 引用类型

类、接口、委托

指针类型、接口类型、类、数组、委托、已装箱的值类型

任何引用类型的值都在堆里

引用类型的值类型保存在堆中

## 对象实例化步骤

1、清零存放静态变量的空间

2、执行静态变量的初始化语句

3、执行基类的构造函数

4、执行本类的静态构造函数

5、清零存放实例变量的空间

6、执行实例变量的初始化语句

7、适当执行基类的实例构造函数

8、执行本类的实例构造函数

类级别的初始化只执行一次

## 惰性求值算法

### 优点

少创建对象

### 缺点

导致对象在内存中待的比较久

不知道何时调用 Dispose 方法，无法释放非托管资源

## 修饰符

### 类修饰符

public、 internal、 partial、abstract、 sealed、 static 

### 成员修饰符

public、protected、private、internal、sealed、abstract、virtual、override、readonly、const

## 协变、逆变

## 装箱、拆箱

### 步骤

首先，会为值类型在托管堆中分配内存

除了值类型本身所分配的内存外，内存总量还要加上类型对象指针和同步块索引所占用的内存

将值类型的值复制到新分配的堆内存中

返回已经成为引用类型的对象的地址

## 集合

### 线性集合

#### 存储方式

##### 直接存储

数组、List、string、struct

元素直接通过下标访问

###### 优点

添加元素高效，直接放在元素末尾

###### 缺点

插入元素抵低效，后面的元素要向后自动腾出位置

##### 顺序存储

线性表、队列queue、栈 stack、索引群集（字典类型，双向 LinkedList）

动态扩大缩小

###### 优点

插入删除数据高效

###### 缺点

查找低效

通过对地址的引用来搜索元素，必须遍历所有元素才能找到某个元素

ConcurrentBag<T> 对应 List<T>

ConcurrentDictionary<TKey, TValue> 对应 Dictionary<TKey, TValue>

ConcurrentQueue<T> 对应 Queue<T>

ConcurrentStack<T> 对应 Stack<T>

## 泛型

非泛型类型中的泛型方法并不会在运行时的代码中生成不同的类型

## 闭包

创建内部匿名类 匿名对象
如果匿名方法（Lambda表达式）引用了某个局部变量，编译器就会自动将该引用提升到该闭包对象中，即将for循环中的变量i修改成了引用闭包对象的公共变量i。这样一来，即使代码执行后离开了原局部变量i的作用域（如for循环），包含该闭包对象的作用域也还存在

```c#
static void Main(string[] args)
{       
    List<Action> lists = new List<Action>();

    for (int i = 0; i < 5; i++)
    {
        Action t = () =>
        {
            Console.WriteLine(i.ToString());
        };
        lists.Add(t);
    }
    foreach (Action t in lists)
    {
        t();
    }
}
```

代码等同

```c#
static void Main(string[] args)
{
	List<Action> lists = new List<Action>();
	for ( int i = 0; i < 5; i++)
	{
		TempClass tempClass = new TempClass();
		tempClass.i = i;
		Action t = tempClass.TempFuc;
		lists.Add(t);
    }
	foreach (Action t in lists)
	{
		t();
	}
}
class TempClass
{
	public int i;
	public void TempFuc()
	{
		Console.WriteLine(i.ToString());
	}
}
```

## 栈内存、堆内存

|   特性   |                       栈内存                        |                     堆内存                     |
| -------- | -------------------------------------------------- | ---------------------------------------------- |
| 存储内容 | 值类型（如 int, double, bool）和引用类型的引用地址     | 引用类型的对象实例（如 class, string 的具体内容） |
| 管理方式 | 由系统自动分配和释放（遵循 LIFO 后进先出原则）          | 由垃圾回收器 (GC) 自动回收                       |
| 性能     | 极快。类似于 CPU 寄存器操作，地址连续                  | 较慢。涉及内存查找、分配、管理及 GC 开销           |
| 空间大小 | 有限且固定。较小，超出则会导致 StackOverflowException | 很大。受物理内存和虚拟内存限制，空间灵活           |
| 生命周期 | 随方法调用入栈，方法结束出栈，自动销毁                 | 由 GC 决定，当对象不再被引用时择机回收            |

## struct

### 内存分片

#### 分配在栈（stack）

* 方法局部变量    在方法内直接声明的结构体变量

* 方法参数    传递给方法的非 ref、非 out 的结构体参数

* ref struct 声明为 ref struct 的类型（Span<T>）,强制保证其只能在栈上

#### 分配在堆（heap）

* 引用类型的成员    
如果 struct 是某个 class 的字段，它会随对象实例一起存在堆里

* 数组元素    
struct[] 数组本身是引用类型，数组内的所有结构体数据都存在堆中

* 装箱
将 struct 赋值给 object、接口（interface）或 ValueType 时，会在堆上创建副本

* 静态字段
声明为 static 的结构体字段存在于静态存储区（逻辑上属于堆）