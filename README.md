# CandidateFeaturesForCSharp9

C# 8.0还未正式发布,在官网它的最新版本还是Preview 5,通往C＃9的漫长道路却已经开始.
前写天收到了活跃在C#一线的`BASSAM ALUGILI`给我分享C# 9.0新特性,我在他文章的基础上进行翻译,希望能对大家有所帮助.

![BassamAlugiliTranslateInvestigatre](https://raw.githubusercontent.com/iblogspost/CandidateFeaturesForCSharp9/Writing/BassamAlugiliTranslateInvestigatre.png)

这是世界上第一篇关于C＃9候选功能的文章。阅读完本文后，您将有希望为将来可能遇到的C＃新特性做好更充分的准备。

这篇文章基于，

- [C# 9.0候选新特性](https://github.com/dotnet/csharplang/milestone/15)

## 原生大小的数字类型

 这次引入一组新类型（nint，nuint，nfloat等）'n'表示`native(原生)`,该特性允许声明一个32位或64位的数据类型,这取决于操作系统的平台类型。

```c#
nint nativeInt = 55; take 4 bytes when I compile in 32 Bit host.    
nint nativeInt = 55; take 8 bytes when I compile in 64 Bit host with x64 compilation settings.  
```

xamarin中已存在类似的概念，

- [xamarin原生类型](https://docs.microsoft.com/en-us/xamarin/cross-platform/macios/native-types-cross-platform)

## Records and Pattern-based With-Expression

 这个功能我等待了很长时间,Records是一种轻量级的不可变类型,它可以是方法,属性,运算符等,它允许我们进行结构的比较, 此外,默认情况下,Records属性是只读的。

Records可以是值类型或引用类型。

 **Example**

```c#
public class Point3D(double X, double Y, double Z);    
public class Demo     
{    
  public void CreatePoint()    
  {    
  var p = new Point3D(1.0, 1.0, 1.0);  
  }  
} 
```

这些代码会被编译器转化如下形式.

```c#
public class Point3D    
{    
private readonly double <X>k__BackingField;    
private readonly double <Y>k__BackingField;    
private readonly double <Z>k__BackingField;    
public double X {get {return <X>k__BackingField;}}    
public double Y{get{return <Y>k__BackingField;}}    
public double Z{get{return <Z>k__BackingField;}}    
    
 public Point3D(double X, double Y, double Z)    
 {    
 <X>k__BackingField = X;    
 <Y>k__BackingField = Y;    
 <Z>k__BackingField = Z;    
 }    
    
 public bool Equals(Point3D value)    
 {    
  return X == value.X && Y == value.Y && Z == value.Z;    
 }    
     
 public override bool Equals(object value)    
 {    
  Point3D value2;    
  return (value2 = (value as Point3D)) != null && Equals(value2);    
 }    
    
 public override int GetHashCode()    
 {    
  return ((1717635750 * -1521134295 +  EqualityComparer<double>.Default.GetHashCode(X)) * -1521134295 + EqualityComparer<double>.Default.GetHashCode(Y)) * -1521134295 +  EqualityComparer<double>.Default.GetHashCode(Z);    
 }    
}    
     
Using Records:    
     
public class Demo    
{    
 public void CreatePoint()    
 {    
 Point3D point3D = new Point3D(1.0, 1.0, 1.0);    
 }    
} 
```

Records迎合了基于表达式形式编程的特性,使得我们可以这样使用它.

```c#
var newPoint3D = Point3D.With(x: 42); 
```

这样我们创建的新Point（new Point3D）就像现有的一个（point3D）一样并把X的值更改为42。

 这个特性于基于`pattern matching`也非常有效,我会在我的下一篇文章中介绍这一点.

 那么我们为什么要使用Records而不是用结构体呢?为了回答这些问题，我引用了了Reddit的一句话：

 “结构体是你必须要有一些约定来实现的东西。你可以让它们不是只读的,你也不用去实现他们的比较逻辑,但如果你不这样做,你将会失去它几乎所有的好处，但编译器不会强制执行这些约束"。

Records类型由是编译器实现，这意味着您必须满足所有这些条件并且不能错误, 因此，它们不仅可以减少重复代码，还可以消除一大堆潜在的错误。

 此外，这个功能在F＃中存在了十多年，其他语言如（Scala，Kotlin）也有类似的概念。

  **F＃**

```F#
type Greeter(name: string) = member this.SayHi() = printfn "Hi, %s" name  
```

**Scala**

```scala
class Greeter(name: String)     
{    
   def SayHi() = println("Hi, " + name)    
}  
```

**Kotlin**

```kotlin
class Greeter(val name: String)     
{    
 fun sayhi()     
 {    
 println("Hi, ${name}");    
 }    
}
```

在没有Records之前,我们要实现类似的功能,C#代码要这么写

**C#**

```c#
public class Greeter
{
 private readonly string _name;
 public Greeter(string name)
 {
 _name = name;
 }
 public void Greet()
 {
  Console.WriteLine($ "Hello, {_name}");
 }
}
```

有了Records之后，我们可以将C＃代码大大地减少了，

```c#
ublic class Greeter(name: string)     
{    
 public void Greet()     
 {    
 Console.WriteLine($ "Hello, {_name}");    
 }    
} 
```



Less code!  =  I love it!

 

## Type Classes



此功能的灵感来自Haskell，它是我最喜欢的功能之一。正如我两年前在我文章中所说，C＃将实现更多的函数式编(FP)程概念，Type Classes就是FP概念之一。在函数式编程中，Type Classes允许您在类型上添加一组操作，但不实现它。由于实现是在其他地方完成的，这是一种多态，它比面向对象编程语言中的class更灵活。

 Type Classes和C＃接口具有相似的用途，但它们的工作方式有所不同，在某些情况下，由于处理固定类型而不是继承层次结构，因此Type Classes更易于使用。

 此这特性最初与“extending everything”功能一起引入，您可以将它们组合在一起，如Mads Torgersen给出的例子所示。

 我引用了官方提案中的一些结论：

“一般来说，”shape“（shape是Type Classes的一个新的关键字）声明非常类似于接口声明，除了以下情况，

- 它可以定义任何类型的成员（包括静态成员）
- 可以通过扩展实现
- 只能在指定的地方当作一种类型使用(作用域)“

Haskell中 Type Classes示例。

```haskell
class Eq a where     
(==) :: a -> a -> Bool    
(/=) :: a -> a -> Bool 
```

“Eq”是类名，而==，/ =是类中的操作。类型“a”是类“Eq”的实例。

 如果我们将上述例子用C#接口实现将会是这样.

```c#
interface Num<A>     
{    
 A Add(A a, A b);    
 A Mult(A a, A b);    
 A Neg(A a);    
}    
     
struct NumInt : Num<int>     
{    
 public int Add(int a, int b) => a + b;     
 public int Mult(int a, int b) => a * b;     
 public int Neg(int a) => -a;    
} 
```

如果我们用Type Classes实现C# 功能会是这样

```c#
shape Num<A>    
{    
 A Add(A a, A b);    
 A Mult(A a, A b);    
 A Neg(A a);    
}    
     
instance NumInt : Num<int>    
{    
 int Add(int a, int b) => a + b;     
 int Mult(int a, int b) => a * b;     
 int Neg(int a) => -a;    
} 
```

通过上面例子,可以看到接口和shape的语法类似 ,那我们再来看看Mads Torgersen给出的例子

> Note：shape不是一种类型。相反，shape的主要目的是用作通用约束，限制类型参数以具有正确的形状，同时允许通用声明的主体使用该形状，

[原始来源](https://github.com/dotnet/csharplang/issues/164)

```c#
public shape SGroup<T>      
{      
 static T operator +(T t1, T t2);      
 static T Zero {get;}       
}
```

这个声明说如果一个类型在T上实现了一个+运算符并且具有0静态属性，那么它可以是一个SGroup <T>。

给int添加静态成员Zero

```c#
public extension IntGroup of int: SGroup<int>    
{    
 public static int Zero => 0;    
}  
```

定义一个AddAll方法

```c#
public static AddAll<T>(T[] ts) where T: SGroup<T> // shape used as constraint    
{    
 var result = T.Zero; // Making use of the shape's Zero property    
 foreach (var t in ts) { result += t; } // Making use of the shape's + operator    
 return result;    
} 
```

让我们用一些整数调用AddAll方法，

```c#
int[] numbers = { 5, 1, 9, 2, 3, 10, 8, 4, 7, 6 };        
WriteLine(AddAll(numbers)); // infers T = int  
```

这就是Type class 的妙处,慢慢消化感受一下??

## Dictionary Literals

 

引入更简单的语法来创建初始化的Dictionary <TKey，TValue>对象，而无需指定Dictionary类型名称或类型参数。使用用于数组类型推断的现有规则推断字典的类型参数。

```c#
// C# 1..8    
var x = new Dictionary <string,int> () { { "foo", 4 }, { "bar", 5 }};   
// C# 9    
var x = ["foo":4, "bar": 5]; 
```

该特性使C＃中的字典工作更简单，并删除冗余代码。此外，值得一提的是，在F＃和Swift等其他编程语言中也使用了类似的字典语法。 

## Params Span <T>

 

允许params语法使用Span <T>这个帮助来实现没有任何堆分配的params参数传递。此功能可以使params方法的使用更加高效。

新的语法如下，

```c#
void Foo(params Span<int> values); 
```



## struct允许使用无参构造函数

 到目前为止，在C＃中不允许在结构体声明中使用无参构造函数,在C＃9中，将删除此限制。

 

**StackOverflow示例**

```c#
public struct Rational    
{    
 private long numerator;    
 private long denominator;    
    
 public Rational(long num, long denom)    
 { /* Todo: Find GCD etc. */ }    
    
 public Rational(long num)    
 {    
  numerator = num;    
  denominator = 1;    
 }    
     
 public Rational() // This is not allowed    
 {    
  numerator = 0;    
  denominator = 1;    
 }    
} 
```

[链接到StackOverflow示例](https://stackoverflow.com/questions/333829/why-cant-i-define-a-default-constructor-for-a-struct-in-net)

其实CLR已经允许值类型数据具有无参构造函数,只是C# 对这个功能进行了限制,在C# 9.0中可能会消除这种限制.

## 固定大小的缓冲区

 这些提供了一种通用且安全的机制，用于向C＃语言声明固定大小的缓冲区。

目前,用户可以在不安全的环境中创建固定大小的缓冲区。但是，这需要用户处理指针，手动执行边界检查，并且只支持一组有限的类型（bool，byte，char，short，int，long，sbyte，ushort，uint，ulong，float和double）。该特性引入后将使固定大小的缓冲区变得安全安全，如下例所示。

 可以通过以下方式声明一个安全的固定大小的缓冲区，

```c#
public fixed DXGI_RGB GammaCurve[1025];  
```

该声明将由编译器转换为内部表示，类似于以下内容，

```c#
[FixedBuffer(typeof(DXGI_RGB), 1024)]    
public ConsoleApp1.<Buffer>e__FixedBuffer_1024<DXGI_RGB> GammaCurve;    
    
// Pack = 0 is the default packing and should result in indexable layout.    
[CompilerGenerated, UnsafeValueType, StructLayout(LayoutKind.Sequential, Pack = 0)]    
struct <Buffer>e__FixedBuffer_1024<T>    
{    
 private T _e0;    
 private T _e1;    
 // _e2 ... _e1023    
 private T _e1024;    
 public ref T this[int index] => ref (uint)index <= 1024u ?    
 ref RefAdd<T>(ref _e0, index):    
 throw new IndexOutOfRange();    
} 
```



## Uft8字符串文字

 它是关于定义一种新的字符串类型UTF8String，它将是，

```c#
System.UTF8String myUTF8string ="Test String";  
```



## Base(T)

 问题:

```c#
interface I1    
{     
    void M(int) { }    
}    
    
interface I2 :I1   
{    
    void M(short) { }    
}    
    
interface I3 : I2 ,I1  
{    
    override void I1.M(int) { }    
}    
    
interface I4 : I3    
{    
    void M2()    
    {    
        base(I3).M(0) // What does this do?    
    }    
} 
```



这里最棘手的部分是，无论`M(short)`和`M(int)`适用于 `M(0)`，但在编译器中查找规则是:如果发现在多个派生接口的适用的成员，我们忽略派生程度较小的接口成员。当进入`I3`查找时我们发现的第一件事是查找`I2.M`，这是适用的，这意味着`I1.M`不会出现在适用成员列表中,这样overrides就没有生效了.

由于我们在上次会议中得出结论，目标类型中*必须*存在实现 ，并且`I2`中`M`是唯一适用的成员，因此`base(I3).M(0)`写入的调用是错误的，因为`I2.M`没有实现`I3`。

**结论**

上次会议的决定得到了肯定，我们得出结论，现有的查找规则不会改变,但是现在也不清楚新的查找规则会是什么样子以及什么时候应用新的规则.

> 在https://github.com/dotnet/csharplang/issues/2337 可以看到有了两种解决方案,可能在C# 8.0中先应用一个临时的解决方案PLAN B,等到c# 9.0 再应用更复杂的解决方案PLAN A. 对于这个特性我还没有研究清楚,所以还不适合妄加评论,有兴趣的可以看看github原文.

更多信息，

- *https://github.com/dotnet/csharplang/issues/2337*
- *https://github.com/dotnet/csharplang/blob/master/meetings/2019/LDM-2019-02-27.md*

## 摘要

 

您已经阅读了第一个C＃9候选特性。正如您所看到的，许多新功能受到其他编程语言或编程范例的启发，而不是自我创新，这些些候特性大部分在在社区中得到了广泛认可。



原文 : <https://www.c-sharpcorner.com/article/candidate-features-for-c-sharp-9/>