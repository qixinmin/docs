##Learn from offical website
##请注意内存开销

1) Enums的内存消耗通常是static constants的2倍。你应该尽量避免在Android上使用enums。
2) 不要做冗余的工作
例如：
  如果你需要返回一个String对象，并且你知道它最终会需要连接到一个StringBuffer，请修改你的函数实现方式，避免直接进行连接操作，应该采用创建一个临时对象来做字符串的拼接这个操作。
  当从已经存在的数据集中抽取出String的时候，尝试返回原数据的substring对象，而不是创建一个重复的对象。使用substring的方式，你将会得到一个新的String对象，但是这个string对象是和原string共享内部char[]空间的。

3) 尽量避免执行过多的内存分配操作

###选择Static而不是Virtual
java类似c++， 每个java类都是继承自object类， 对对象的内容访问需要代价的，如果对类直接操作会比对对象操作稍微快些
如果你不需要访问一个对象的值，请保证这个方法是static类型的，这样方法调用将快15%-20%。这是一个好的习惯，因为你可以从方法声明中得知调用无法改变这个对象的状态。

###常量声明为Static Final
考虑下面这种声明的方式
```
static int intVal = 42;
static String strVal = "Hello, world!";
```
编译器会使用一个初始化类的函数，然后当类第一次被使用的时候执行。这个函数将42存入intVal，还从class文件的常量表中提取了strVal的引用。当之后使用intVal或strVal的时候，他们会直接被查询到。

我们可以用final关键字来优化：
```
static final int intVal = 42;
static final String strVal = "Hello, world!";
```
这时再也不需要上面的方法了，因为final声明的常量进入了静态dex文件的域初始化部分。调用intVal的代码会直接使用42，**调用strVal的代码也会使用一个相对廉价的“字符串常量”指令，而不是查表**。

Notes：这个优化方法只对原始类型和String类型有效，而不是任意引用类型。不过，在必要时使用static final是个很好的习惯。

###避免内部的Getters/Setters
像C++等native language，通常使用getters(i = getCount())而不是直接访问变量(i = mCount)。这是编写C++的一种优秀习惯，而且通常也被其他面向对象的语言所采用，例如C#与Java，因为编译器通常会做inline访问，而且你需要限制或者调试变量，你可以在任何时候在getter/setter里面添加代码。

然而，在Android上，这不是一个好的写法。虚函数的调用比起直接访问变量要耗费更多。在面向对象编程中，将getter和setting暴露给公用接口是合理的，但在类内部应该仅仅使用域直接访问。

在没有JIT(Just In Time Compiler)时，直接访问变量的速度是调用getter的3倍。有JIT时，直接访问变量的速度是通过getter访问的7倍。

请注意，如果你使用**ProGuard**，你可以获得同样的效果，**因为ProGuard可以为你inline accessors.**

###使用增强的For循环
增强的For循环（也被称为 for-each 循环）可以被用在实现了 Iterable 接口的 collections 以及数组上。使用collection的时候，Iterator会被分配，用于for-each调用hasNext()和next()方法。使用ArrayList时，手写的计数式for循环会快3倍（不管有没有JIT），但是对于其他collection，增强的for-each循环写法会和迭代器写法的效率一样。

请比较下面三种循环的方法：
```
static class Foo {
    int mSplat;
}

Foo[] mArray = ...

public void zero() {
    int sum = 0;
    for (int i = 0; i < mArray.length; ++i) {
        sum += mArray[i].mSplat;
    }
}

public void one() {
    int sum = 0;
    Foo[] localArray = mArray;
    int len = localArray.length;

    for (int i = 0; i < len; ++i) {
        sum += localArray[i].mSplat;
    }
}

public void two() {
    int sum = 0;
    for (Foo a : mArray) {
        sum += a.mSplat;
    }
}
```
zero()是最慢的，因为JIT没有办法对它进行优化。
one()稍微快些。
two() 在没有做JIT时是最快的，可是如果经过JIT之后，与方法one()是差不多一样快的。它使用了增强的循环方法for-each。
**所以请尽量使用for-each的方法，但是对于ArrayList，请使用方法one()。传统类型的for循环**

##Using tools
