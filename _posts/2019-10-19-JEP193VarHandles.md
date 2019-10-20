# JEP193: Variable Handles
    Author	Doug Lea
    Owner	Paul Sandoz
    Type	Feature
    Scope	SE
    Status	Closed / Delivered
    Release	9
    Component	core-libs / java.lang
    Discussion	core dash libs dash dev at openjdk dot java dot net
    Effort	M
    Duration	L
    Relates to	JEP 266: More Concurrency Updates
    Reviewed by	Dave Dice, Paul Sandoz
    Endorsed by	Brian Goetz
    Created	2014/01/06 20:00
    Updated	2017/08/17 16:45
    Issue	8046183

## 开篇语
定义一个标准方法来调用同java.util.concurrent.atomic和sun.misc.Unsafe等价的各种对象字段和数组元素，
一组用于细粒度控制内存序的内存屏障，以及一种标准的可达性屏障操作，以确保引用的对象仍然是强可达的。

## 目标
目标要求如下：  
   
* 安全的。不能将JVM处于一个损坏的内存状态。例如，对象的字段只能被更新为可以转换为该字段实例的对象，
或者数组元素只能使用数组范围内的角标访问。 
      
* 完整性。除了Final常量字段和Final对象外，对对象字段的访问必须遵循与getfield和putfield字节码相同的访问规则
（注：这种安全性和完整性访问规则也适用于对字段进行读写访问的MethodHandles）。  
     
* 性能。性能特征必须和sun.misc.Unsafe操作一样（具体地说，生成的汇编代码应该是几乎完全相同的模块，不能折叠起来）。       
* 可用性。API必须要比sun.misc.Unsafe的API更好。  
   
以上这些要求是可取的，并不是强制要求的，就像java.u til.concurrent.atomic 的API一样。      

## 动机
随着并发和并行编程在Java中持续扩展，开发者对不能使用Java构造函数管理单个类上的字段的原子或有序操作，
感到越来越沮丧。例如，对一个计数值段进行原子递增。到目前为止，实现这些效果的唯一方法还是使用单独的AtomicInteger
（增加了空间开销和额外的并发问题来管理间接寻址），或者，在某些情况下，使用原子性的字段更新程序
（经常遇到比操作本身还要多的开销），或者使用不安全的（不可携带也不受支持）JVM内部的sun.misc.Unsafe API。
内部函数运行的更快，所以它们已经被广泛地使用，这不利用安全性和可移植性。    

如果没有这个JEP，随着原子API的扩展以覆盖额外的访问一致性策略（与最近的C++内存模型对齐），
作为Java内存模型修订的一部分，这些问题预计会变得更糟。       

## 说明   
一个变量句柄就是一个变量的类型化引用，它支持在多种访问模式下对变量的读写访问。支持的变量类型包括实例字段，
静态字段和数组元素正在考虑增加其他变量类型如数组视图，将字节数组或字符数组作为一个长的数组查看，以及使用ByteBuffers
描述的堆外区域中的位置。

变量句柄需要库函数增强，JVM增强，还有编译器的支持。另外，它需要对Java语言规范和JVM规范作少量的更新。少量语言增强，
还考虑了编译时类型检查和少量补充语法。预期得到的规范将以自然的方式扩展到附加的基本类型或类似数组的类型，如果它们
被添加到Java中。不过，这不是一种用于控制多个变量访问和更新的通用事务机制。描述和实现这种结构的替代形式，
可能会在这个JEP过程中探索，也可能成为后续JEPs的主题。

变量句柄由单个抽象类模型封装，java.lang.invoke.VarHandle。其中，每个变量的访问通过[signature-polymorphic](http://docs.oracle.com/javase/8/docs/api/java/lang/invoke/MethodHandle.html#sigpoly)方法。

访问模式集合表示一个最小的可行集，并被设计为与C/C++ 11原子兼容，而不依赖于Java内存模型的修订更新。如果需要，
还会添加其他访问模式。某些访问模式可能不适用于某些变量类型，如果是的话，在关联的VarHandle实例上调用的话会抛出
UnSupportedOperationException异常。

访问模式分为以下几种类别：

1. 读访问模式，如读取一个具有volatile 内存序字段。

2. 写访问模式，如更新一个具有release 内存序字段。

3. 原子更新访问模式，如对一个变量执行CAS，变量的读写都具有volatile 内存序效果。

4. 数值原子更新访问模式，如get-and-add，写操作时具有plain 内存序效果，读操作时具有acquire 内存序效果。

5. 按位原子更新访问模式，如get-and-bitwise-and，在写操作时具有release 内存序效果，读操作时具有plain 内存序效果。

后面三种类型通常被称为 读-修改-写 模式。

访问模式方法的多态特性使得变量句柄仅使用一个抽象类就可以支持多种变量种类和变量类型。
这避免了变量类型和类型特定类的爆炸。此外，尽管访问模式的多态方法会被声明为对象的可变参数数组，这样的多态特性确保
不会有基本类型参数的装箱，也不会将参数打包到数组中。这是使得HotSpot解释器和C1/C2编译器在运行时具有可预测的行为和性能。

创建VarHandle实例的方法和生成访问相似变量类型的MethodHandle实例的方法位于同一区域。

创建对象实例或者静态变量类型的VarHandle实例方法位于java.lang.invoke.MethodHandles.Lookup，是通过在关联的接收类
中查找字段的过程中创建的。例如，以下在一个接受类Foo上查找并获取名称为 i，类型为int 的VarHandle变量，
可能会执行以下操作：
```
class Foo {
    int i;

    ...
}

...

class Bar {
    static final VarHandle VH_FOO_FIELD_I;

    static {
        try {
            VH_FOO_FIELD_I = MethodHandles.lookup().
                in(Foo.class).
                findVarHandle(Foo.class, "i", int.class);
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}

```

访问变量的VarHandle查找过程，将在生成和返回VarHandle之前，执行的访问控制检查（代表lookup类）与向上查找
MethodHandle执行的访问控制检查完全相同，MethodHandle为同一字段提供读写访问权限
（参见 MethodHandles.Lookup类中{Static},{Getter,Setter}方法）。

在以下条件下调用访问模式方法将会抛出UnsupportedOperationException异常：

* 一个VarHandle对一个Final变量的写访问模式方法。

* 对一个引用变量类型或者一个非数值类型（如boolean）的基于数值的访问模式方法（getAndAdd 和 AddAndGet）。

* 对一个引用变量类型或者float和double类型（后一种限制可能会在以后的修订中取消）的基于位的访问模式方法。

* 不需要将变量标记为volatile，关联的VarHandle才能访问。事实上，volatile修饰符，如果存在，将会被忽略。
这与java.util.concurrent.atomic.Atomic{Int,Long,引用类型}更新程序的行为不同,后者必须将相应的字段标记为volatile。
在某些情况下，这可能过于严格，因为已知某些volatile访问并不总是必需的。

* 为基于数组的变量类型创建VarHandle实例的方法位于java.lang.invoke.MethodHandles（参见在MethodHandles类的数组元素的{Getter,Setter}方法）。
例如，int数组的VarHandle可能创建如下：

``
VarHandle intArrayHandle = MethodHandles.arrayElementVarHandle(int[].class);
``

当在以下条件下调用时，访问模式方法将会抛出UnSupportedOperationException异常：

* 数组组件引用变量类型或者非数值类型（boolean）的基于数值的访问模式方法（getAndAdd和addAndGet）。

* 引用变量类型或float和double类型（后一种限制可能会在以后的修订中取消）的基于位的访问模式方法。

变量种类中的变量类型（实例变量，静态变量和数组元素）支持所有基本类型和引用类型。其它变量种类可以支持所有这些变量
类型或者这些变量类型的一部分。

为基于数组视图的变量类型创建VarHandle实例的方法也位于java.lang.invoke.MethodHandles中。例如，将一个字节数组
视为一个未对齐的Long数组的VarHandle可以创建如下：

``
VarHandle longArrayViewHandle = MethodHandles.byteArrayViewVarHandle(
        long[].class, java.nio.ByteOrder.BIG_ENDIAN);
``

尽管使用java.nio.ByteBuffer也可以实现类似的机制，它要求创建一个ByteBuffer实例来包装字节数组。由于escape分析的
脆弱性以及访问必须经过ByyteBuffer实例，这并不总是能保证可靠的性能。在未对齐访问的情况下，除了普通访问模式外，
所有方法都将抛出IllegalStateException 异常。用于对齐访问的某些volatile操作，取决于变量类型。这样的VarHandle
实例将可用于将数组访问矢量化。

访问模式方法的参数数量，参数类型和返回类型由变量类型，访问模式的变量类型及其特点控制。VarHandle的创建方法（如前所述）
将记录这些需求。例如，先前查找的VH_FOO_FIELD_I变量上的compareAndSet将要求3个参数，一个作为接收者的Foo实例参数，
另外两个int类型的参数，一个作为期望值，一个作为实际值：

``
Foo f = ...
boolean r = VH_FOO_FIELD_I.compareAndSet(f, 0, 1);
``

相反，一个getAndSet要求2个参数，一个接收实例Foo和一个作为待更新值的int类型参数：

``
int o = (int) VH_FOO_FIELD_I.getAndSet(f, 2);
``








