[toc]

# 常备应对知识点

## 介绍

面试官好!我叫李华，来自武汉大学计算机学院。华为公司是我们一直非常尊敬的公司，这一次我应聘的是网络研发的工作。从事网络相关工作是我一直的梦想。为此，我在研究生阶段很早就结合岗位的要求进行了准备，包括对各种协议的熟悉，常用算法的熟悉等。在两年的时间里，我也重点对网络相关的课题进行了研究。希望能够加入公司，从事网络相关的这个工作，谢谢！

## Serializable 原理及面试题

### 反序列化对象后，需要调用构造函数重新构造吗

不需要的

数据的来源主要以二进制数据为依据构造的

### 序列化前的对象与序列化后的对象是什么关系？是（“==”还是“equal”？是浅复制，还是深复制）

序列化前与后是两个对象，对象的地址是变化了的，但是调用 euqal的话返回是true，因为是深拷贝原理。

### Android里为什么要设计出Bundle而不是Map结构

主要实现是使用的 Parcel ,因为Bundle用于跨进程至之间传输对象，用来传输数据量小的对象。

### SerialVersionID的作用是什么

主要是做版本控制

### Android中Intent/Bundle的通信原理以及大小限制

最大数据量Count为 15个

默认数据量大小为 1Mb （（1*1024*1024）-（4096*2））

可以突破1Mb的传输为最大数据量大小为 4Mb

### 为什么Intent不能直接在组件间传递对象而要通过序列化机制

startActivity(new Intent())

Intent 需要进行序列化，然后跨进程通讯

activity启动需要与 -》 AMS

### 序列化与持久化的关系和区别是什么？

序列化主要是为了跨进程调用进行通讯，传输对象

持久化主要是为了把数据进行存储下来

### 在java有Serializable的前提下，Android为什么设计出了Parcelable？

java中的序列化方式Serializable效率比较低，主要有以下原因：

- Serializable在序列化过程中会创建大量的临时变量，这样就会造成大量的GC。
- Serializable使用了大量反射，而反射操作耗时。
- Serializable使用了大量的IO操作，也影响了耗时。

所以Android就像重新设计了IPC方式Binder一样，重新设计了一种序列化方式，结合Binder的方式，对上述三点进行了优化，一定程度上提高了序列化和反序列化的效率。

#### Serializable、Parcelable、Json等序列化方式我们该怎么选择？

先说说序列化的用处，主要用在三个方面：

##### 内存数据传输

内存传输方面，主要用Parcelable。一是因为Parcelable在内存传输的效率比Serializable高。二是因为在Android中很多传输数据的方法中，自带了对于Serializable、Parcelable类型的传输方法。比如:

- Bundle.putParcelable,
- Intent putExtra(String name, Parcelable value)

等等吧，基本上对象传输的方法都支持了，所以这也是Parcelable的优势。

##### 数据持久化（本地存储）

如果只针对Serializable和Parcelable两种序列化方式，需要选择Serializable。

首先，Serializable本身就是存储到二进制文件，所以用于持久化比较方便。而Parcelable序列化是在内存中操作，如果进程关闭或者重启的时候，内存中的数据就会消失，那么Parcelable序列化用来持久化就有可能会失败，也就是数据不会连续完整。

而且Parcelable还有一个问题是兼容性，每个Android版本可能内部实现都不一样，知识用于内存中也就是传递数据的话是不影响的，但是如果持久化可能就会有问题了，低版本的数据拿到高版本可能会出现兼容性问题。

但是实际情况，对于Android中的对象本地化存储，一般是以数据库、SP的方式进行保存。

##### 网络传输

而对于网络传输的情况，一般就是使用JSON了。主要有以下几点原因：

- 轻量级，没有多余的数据。
- 与语言无关，所以能兼容所有平台语言。
- 易读性，易解析。

### Parcelable一定比Serializable快吗？

正常情况下，对象在内存中进行传输确实是Parcelable比较快，但是Serializable是有缓存的概念的，有人做了一个比较有趣的实验：

当序列化一个超级大的对象图表（表示通过一个对象，拥有通过某路径能访问到其他很多的对象），并且每个对象有10个以上属性时，并且Serializable实现了writeObject()以及readObject()，在平均每台安卓设备上，Serializable序列化速度大于Parcelable 3.6倍，反序列化速度大于1.6倍。

具体原因就是因为Serilazable的实现方式中，是有缓存的概念的，当一个对象被解析过后，将会缓存在HandleTable中，当下一次解析到同一种类型的对象后，便可以向二进制流中，写入对应的缓存索引即可。但是对于Parcel来说，没有这种概念，每一次的序列化都是独立的，每一个对象，都当作一种新的对象以及新的类型的方式来处理。

具体过程可以看看这篇：<https://juejin.cn/post/6854573218334769166>

### 为什么Java提供了Serializable的序列化方式，而不是直接使用json或者xml？

我觉得是历史遗留问题。

有的人可能会想到各种理由，比如可以标记哪些类可以被序列化。又或者可以通过UID来标示反序列化为同一个对象。等等。

但是我觉得最大的问题还是历史遗留问题，在以前，json还没有成为大家认同的数据结构，所以Java就设计出了Serializable的序列化方式来解决对象持久化和对象传输的问题。然后Java中各种API就会依赖于这种序列化方式，这么些年过去了，Java体系的庞大也造成难以改变这个问题，牵一发而动全身。

为什么我这么说呢？

主要有两点依据：

1. 曾经Oracle Java平台组的架构师说过，删除Java的序列化机制并且提供给用户可以选择的序列化方式（比如json）是他们计划中的一部分，因为Java序列化也造成了很多Java漏洞。具体可以参见文章：<https://www.infoworld.com/article/3275924/oracle-plans-to-dump-risky-java-serialization.html>

2. 因为在Serializable类的介绍注释中，明确说到推荐大家选择JSON 和 GSON库，因为它简洁、易读、高效。

```xml
* <h3>Recommended Alternatives</h3>
* <strong>JSON</strong> is concise, human-readable and efficient. Android
* includes both a {@link android.util.JsonReader streaming API} and a {@link
* org.json.JSONObject tree API} to read and write JSON. Use a binding library
* like <a href="http://code.google.com/p/google-gson/">GSON</a> to read and
* write Java objects directly.
```

## 垃圾回收机制

### JVM 内存模型

JVM内存模型是一种符合计算机内存模型规范，屏蔽了各种硬件和操作系统的访问差异，保证了Java程序在各个平台下对内存的访问都能得到一致性效果的一种机制以及规范，目的主要是为了解决由于多线程通过共享内存进行通讯的时候存在的一些原子性、可见性以及有序性问题。

### JVM 内存结构简单说一下呢

从以下两个图开始讲起,线程私有与线程共享的区域，虚拟机栈是基于线程的，一个Main方法都会运行一个线程，在它的细节里面就会涉及到栈帧的出栈入栈，类似于打手枪，栈里面的每一条数据就是每一个栈帧。本地方法栈就是去执行Native方法，程序计数器就是去存储当前线程所执行字节码的指令。

![JVM虚拟机结构图](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-231923.png)

![JVM栈帧结构图](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-232121.png)

### 什么情况下内存栈溢出

栈溢出就是会无限递归startOverFlow

不断的去创建线程，就导致死机，空间不足，爆出没有内存的异常

### 描述new一个对象的流程

![Java对象创建流程图](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-232317.png)

> 在实例化一个对象的时候，JVM首先会检查目标对象是否已经被加载且初始化，如何没有，JVM则需要做的是立刻去加载目标类，然后调用目标类的构造器完成初始化。目标类的加载是通过类加载器来实现的，主要是把一个类加载到内存里面，然后是初始化的过程，这个步骤主要是对目标类里面的**静态变量、成员变量、静态代码块**进行初始化，当目标类被初始化以后，就可以从常量池里面找到对应的类元信息了，并且目标对象的大小在类加载完成之后就已经确定了，所以这个时候就需要去为新创建的对象根据目标对象的大小在堆内存里面去分配内存空间，内存分配的方式一般有两种：第一种是**指针碰撞**，第二种是**空闲列表**，JVM会根据Java堆内存是否规整来决定内存的分配方法，接下来JVM会把对象里面的普通成员变量初始化为0值，比如说，Int类型初始化为0，String类型初始化为null，这一步操作主要是为了保证实例化对象中实例字段不用初始化就可以直接使用，也就是程序能够直接获取这些字段对应的数据类型的0值，然后JVM还需要对目标对象的对象头做一些设置，比如**对象所属的类元信息，对象的GC分代年龄，HashCode，锁标记**等，完成这些步骤以后对于JVM来讲，新对象的创建工作已经完成，但是对于Java语言来说，对象创建才算刚刚开始，接下来要做的就是要执行目标对象内部生成的 `<init>` 方法，初始化成员变量的值，执行构造块，最后调用目标对象的构造方法，去完成对象的创建，其中，`<init>` 方法是Java文件编译后在字节码文件里面自动生成的，它是一个实例构造器，这个构造器里面会把**构造块，变量初始化，调用父类构造器**等这样的一些操作组织在一起，所以调用 `<init>` 能够去完成这一系列的初始化动作，以上就是我对于这个问题的理解。

### Java对象会不会分配在栈中呢

当然会的

特殊情况，如果符合逃逸分析他会走栈上分配

![对象的分配规则](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-232501.png)

### 如果一个对象是否被回收，有哪些算法，实际虚拟机使用的最多的是什么？

引用计数法，相互引用，如果采用引用计数法的话，你们都有计数，但是没用

大部分虚拟机都是使用的可达性算法

![可达性算法](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-232639.png)

### GC收集算法有哪些？他们的特点是什么？

![垃圾回收器](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-233801.png)

![复制算法](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-233934.png)

![标记清除算法](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-234105.png)

![标记整理算法](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-234211.png)

### JVM中一次玩出个的GC流程是怎样的？对象如何晋级到老年代？

需要结合对象分配原则去讲解

GC的时候肯定伴随着对象的分配，所以对象优先在一等区进行分配，一等区空间不够了，然后对象会进入From区，或者to区，都不够了，才会进入老年代

![对象的分配原则](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-234333.png)

### Java中的集中引用关系，他们的区别是什么？

### final、finally、finalize的区别

final 修饰

类（不能被继承）1.锁定 2.效率

变量(成员变量，局部变量)变成了常量，赋值一次，不能修改了。

finally  无论是否有异常，都会进行释放执行

```java
try{
    
}catch(){
    
}finally{
    
}
```

finalize

finalize基本上不怎么用，不是非常的可靠

### String s = new String（“xxx”）;创建了几个对象？

两个对象

```java
String str1 = "abc";  // 在常量池中
 
String str2 = new String("abc"); // 在堆上
```

当直接赋值时，字符串“abc”会被存储在常量池中，只有1份，此时的赋值操作等于是创建0个或1个对象。如果常量池中已经存在了“abc”，那么不会再创建对象，直接将引用赋值给str1；如果常量池中没有“abc”，那么创建一个对象，并将引用赋值给str1。

那么，通过new String("abc");的形式又是如何呢？答案是1个或2个。

当JVM遇到上述代码时，会先检索常量池中是否存在“abc”，如果不存在“abc”这个字符串，则会先在常量池中创建这个一个字符串。然后再执行new操作，会在堆内存中创建一个存储“abc”的String对象，对象的引用赋值给str2。此过程创建了2个对象。

当然，如果检索常量池时发现已经存在了对应的字符串，那么只会在堆内创建一个新的String对象，此过程只创建了1个对象。

在上述过程中检查常量池是否有相同Unicode的字符串常量时，使用的方法便是String中的intern()方法。

```java
public native String intern();
```

下面通过一个简单的示意图看一下String在内存中的两种存储模式。

![192741326440d4fcf1f950b1efe76f84.jpeg](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2021/images/192741326440d4fcf1f950b1efe76f84.jpeg)

上面的示意图我们可以看到在堆内创建的String对象的char value[]属性指向了常量池中的char value[]。

还是上面的示例，如果我们通过debug模式也能够看到String的char value[]的引用地址。

![e16a5886e5933ae37fefcab69f950263.jpeg](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2021/images/e16a5886e5933ae37fefcab69f950263.jpeg)

图中两个String对象的value值的引用均为{char[3]@1355}，也就是说，虽然是两个对象，但它们的value值均指向常量池中的同一个地址。当然，大家还可以拿一个复杂对象（Person）的字符串属性（name）相同时的debug结果进行比对，结果是一样的。

#### 深入问法

如果面试官说程序的代码只有下面一行，那么会创建几个对象？

```java
new String("abc");
```

答案是2个？

还真不一定。之所以单独列出这个问题是想提醒大家一点：没有直接的赋值操作（str="abc"），并不代表常量池中没有“abc”这个字符串。也就是说衡量创建几个对象、常量池中是否有对应的字符串，不仅仅由你是否创建决定，还要看程序启动时其他类中是否包含该字符串。

#### 升级加码

以下实例我们暂且不考虑常量池中是否已经存在对应字符串的问题，假设都不存在对应的字符串。

以下代码会创建几个对象：

```java
String str = "abc" + "def";
```

上面的问题涉及到字符串常量重载“+”的问题，当一个字符串由多个字符串常量拼接成一个字符串时，它自己也肯定是字符串常量。字符串常量的“+”号连接Java虚拟机会在程序编译期将其优化为连接后的值。

就上面的示例而言，在编译时已经被合并成“abcdef”字符串，因此，只会创建1个对象。并没有创建临时字符串对象abc和def，这样减轻了垃圾收集器的压力。

我们通过javap查看class文件可以看到如下内容。

![da2e53e424750afd4ec63dd5acbc6770.jpeg](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2021/images/da2e53e424750afd4ec63dd5acbc6770.jpeg)

针对上面的问题，我们再次升级一下，下面的代码会创建几个对象？

```java
String str = "abc" + new String("def");
```

创建了4个，5个，还是6个对象？

4个对象的说法：常量池中分别有“abc”和“def”，堆中对象new String("def")和“abcdef”。

这种说法对吗？不完全对，如果说上述代码创建了几个字符串对象，那么可以说是正确的。但上述的代码Java虚拟机在编译的时候同样会优化，会创建一个StringBuilder来进行字符串的拼接，实际效果类似：

```java
String s = new String("def");
new StringBuilder().append("abc").append(s).toString();
```

很显然，多出了一个StringBuilder对象，那就应该是5个对象。

那么创建6个对象是怎么回事呢？有同学可能会想了，StringBuilder最后toString()之后的“abcdef”难道不在常量池存一份吗？

这个还真没有存，我们来看一下这段代码：

```java
@Test
public void testString3() {
    String s1 = "abc";
    String s2 = new String("def");
    String s3 = s1 + s2;
    String s4 = "abcdef";
    System.out.println(s3==s4); // false
}
```

![adc020264de49d273648164d40fed2d0.jpeg](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2021/images/adc020264de49d273648164d40fed2d0.jpeg)

很明显，s3和s4的值相同，但value值的地址并不相同。即便是将s3和s4的位置调整一下，效果也一样。s4很明确是存在于常量池中，那么s3对应的值存储在哪里呢？很显然是在堆对象中。

我们来看一下StringBuilder的toString()方法是如何将拼接的结果转化为字符串的：

```java
@Override
public String toString() {
    // Create a copy, don't share the array
    return new String(value, 0, count);
}
```

很显然，在toString方法中又新创建了一个String对象，而该String对象传递数组的构造方法来创建的：

```java
public String(char value[], int offset, int count) 
```

也就是说，String对象的value值直接指向了一个已经存在的数组，而并没有指向常量池中的字符串。

因此，上面的准确回答应该是创建了4个字符串对象和1个StringBuilder对象。

### Java的四大引用

- 强引用： 普通变量都属于强引用，比如 private Context context;
- 软应用： SoftReference，在发生OOM之前，垃圾回收器会回收SoftReference引用的对象。
- 弱引用： WeakReference，发生GC的时候，垃圾回收器会回收WeakReference中的对象。
- 虚引用： 随时会被回收，没有使用场景。
  怎么理解强引用：

> 强引用对象的回收时机依赖垃圾回收算法，我们常说的可达性分析算法，当Activity销毁的时候，Activity会跟GCRoot断开，至于GCRoot是谁？这里可以大胆猜想，Activity对象的创建是在ActivityThread中，ActivityThread要回调Activity的各个生命周期，肯定是持有Activity引用的，那么这个GCRoot可以认为就是ActivityThread，当Activity 执行onDestroy的时候，ActivityThread 就会断开跟这个Activity的联系，Activity到GCRoot不可达，所以会被垃圾回收器标记为可回收对象。

## 类加载

### 类的生命周期

![类的生命周期](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-234529.png)

类从被加载到JVM中开始，到卸载为止，整个生命周期包括：加载、验证、准备、解析、初始化、使用和卸载七个阶段。

### 类加载阶段

类的加载主要有三步：

- 将class文件字节码内容加载到内存中。
- 并将这些静态数据转换成方法区中的运行时数据结构。
- 在堆中生成一个代表这个类的java.lang.Class对象。

我们编写的java文件会在编译后变成.class文件，类加载器就是负责加载class字节码文件，class文件在文件开头有特定的文件标识，将class文件字节码内容加载到内存中，并将这些内容转换成方法区中的运行时数据结构。并且ClassLoader只负责class文件的加载，至于它是否可以运行，则由执行引擎Execution Engine决定。

简单来说类加载机制就是从文件系统将一系列的 class 文件读入 JVM 内存中为后续程序运行提供资源的动作。

### 类加载器种类

类加载器种类主要有四种：

- BootstrapClassLoader：启动类加载器，使用C++实现
- ExtClassLoader：扩展类加载器，使用Java实现
- AppClassLoader：应用程序类加载器，加载当前应用的classpath的所有类
- UserDefinedClassLoader：用户自定义类加载器

属于依次继承关系，也就是上一级是下一级的父加载器(例外：BootstrapClassLoader无父子关系，属于根加载器)。

### 类加载过程（双亲委派机制）

类加载的过程可以用一句话概括：

先在方法区找class信息，有的话直接调用，没有的话则使用类加载器加载到方法区。

对于类加载器加载过程，就用到了双亲委派机制，具体如下：

当一个类加载器收到了类加载的请求，它不会直接去加载这类，而是先把这个请求委派给父加载器去完成，依次会传递到最上级也就是启动类加载器，然后父加载器会检查是否已经加载过该类，如果没加载过，就会去加载，加载失败才会交给子加载器去加载，一直到最底层，如果都没办法能正确加载，则会跑出ClassNotFoundException异常。

举例：

- 当Application ClassLoader 收到一个类加载请求时，他首先不会自己去尝试加载这个类，而是将这个请求委派给父类加载器Extension ClassLoader去完成。
- 当Extension ClassLoader收到一个类加载请求时，他首先也不会自己去尝试加载这个类，而是将请求委派给父类加载器Bootstrap ClassLoader去完成。
- 如果Bootstrap ClassLoader加载失败(在<JAVA_HOME>\lib中未找到所需类)，就会让Extension ClassLoader尝试加载。
- 如果Extension ClassLoader也加载失败，就会使用Application ClassLoader加载。
- 如果Application ClassLoader也加载失败，就会使用自定义加载器去尝试加载。
- 如果均加载失败，就会抛出ClassNotFoundException异常。

这么设计的原因主要有两点：

- 这种层级关系可以避免类的重复加载。
- 是为了防止危险代码的植入，比如String类，如果在AppClassLoader就直接被加载，就相当于会被篡改了，所以都要经过老大，也就是BootstrapClassLoader进行检查，已经加载过的类就不需要再去加载了。

## 抽象类与接口的区别

| 参数               | 抽象类                                                       | 接口                                                         |
| :----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 默认方法实现       | 它可以有默认的方法实现                                       | Java8之前，接口中不存在方法实现                              |
| 实现方式           | 子类使用**extends**关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现。 | 子类使用关键字**implements**来实现接口。它需要提供接口中所有声明的方法的实现 |
| 构造器             | 抽象类可以有构造器                                           | 接口不能有构造器                                             |
| 与正常Java类的区别 | 除了你不能实例化抽象类之外，它和普通Java类没有任何区别       | 接口是完全不同的类型                                         |
| 访问修饰符         | 抽象方法可以有**public**、**protected**和**default**这些修饰符 | 接口方法默认修饰符是**public**。你不可以使用其它修饰符。     |
| main方法           | 抽象方法可以有main方法并且我们可以运行它                     | 接口没有main方法，因此我们不能运行它。                       |
| 多继承             | 抽象方法可以继承一个类和实现多个接口                         | 接口只可以继承一个或多个其它接口                             |
| 速度               | 它比接口速度要快                                             | 接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法。   |
| 添加新方法         | 如果你往抽象类中添加新的方法，你可以给它提供默认的实现。因此你不需要改变你现在的代码。 | 如果你往接口中添加方法，那么你必须改变实现该接口的类。       |

## Java的原子性、可见性与有序性

**原子性（Atomicity）**：一个操作或者多个操作，要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

**可见性（Visibility）**：可见性是指当一个线程修改了共享变量的值，其他线程能够立即得知这个修改。

**有序性（Ordering）**：一个线程中的所有操作必须按照程序的顺序来执行。

Java语言提供了 **volatile** 和 **synchronized** 两个关键字来保证线程之间操作的有序性，**volatile** 关键字本身就包含了禁止指令重排序的语义，而 **synchronized** 则是由“一个变量在同一个时刻只允许一条线程对其进行lock操作”这条规则获得的，这条规则决定了持有同一个锁的两个同步块只能串行地进入。

参考链接：https://blog.csdn.net/mxw2552261/article/details/105548488

## 线程同步与线程安全

### Synchronized 

- 保证⽅法内部或代码块内部资源（数据）的互斥访问。即同⼀时间、由同⼀个 Monitor 监视的代码，最多只能有⼀个线程在访问
- 保证线程之间对监视资源的数据同步。即，任何线程在获取到 Monitor后的第⼀时间，会先将共享内存中的数据复制到⾃⼰的缓存中；任何线程在释放 Monitor 的第⼀时间，会先将缓存中的数据复制到共享内存中。

#### Synchronize应用场景

Synchronize一般应用于以下几个场景：

- 修饰实例方法，对当前实例对象（this）加锁

```java
    public synchronized void lockMethod() {
        System.out.println("lock method");
    }
```

- 修饰静态方法，对当前类对象（Class对象）加锁

```java
    public static synchronized void lockStaticMethod() {
        System.out.println("lock static method");
    }
```

- 修饰代码块，指定对某个对象进行加锁

```java
    Object object = new Object();

    public void lockObject() {
        synchronized (object) {
            System.out.println("lock object");
        }
    }
```

根据锁的力度来选择使用哪一种，比如使用静态方法上锁，锁的力度是整个Class对象，如果大量线程都在使用Class对象作为锁对象，那么锁的粒度很大。比如`System.out.println()`这种方式底层是对PrintStream上锁，但PrintSteam又是单例的，因此在此代码中如果大量使用`System.out.println()`，性能会受影响。

```java
    /**
     * Prints a String and then terminate the line.  This method behaves as
     * though it invokes {@link #print(String)} and then
     * {@link #println()}.
     *
     * @param x  The {@code String} to be printed.
     */
    public void println(String x) {
        synchronized (this) {
            print(x);
            newLine();
        }
    }
```

#### Synchronize锁的膨胀升级过程

Synchronize在1.6版本之前性能较差，在并发不严重的情况下，因为Synchronize依然对象上锁，每个对象需要维护一个Monitor管理对象，管理对象需要维护一个Mutex互斥量对象。Mutex是由操作系统内部的pthread线程库维护的。上锁需要通过JVM从用户态切换到内核态来调用底层操作系统的指令，这样操作的性能比较差。

> **无锁状态 --> 偏向锁 --> 轻量级锁 --> 重量级锁**

- 对象头中的MarkWord字段（32位）

<table style="text-align:center;vertical-align: middle!important;">
  <tr>
    <th rowspan="2">锁状态</th>
    <th colspan="2">25</th>
    <th rowspan="2">4</th>
    <th>1</th>
    <th>2</th>
  </tr>
  <tr>
    <th>23</th>
    <th>2</th>
    <th>是否偏向锁</th>
    <th>锁标志位</th>
  </tr>
  <tr>
    <td>无锁</td>
    <td colspan="2">hashcode</td>
    <td>age</td>
    <td>0</td>
    <td>01</td>
  </tr>
  <tr>
    <td>偏向锁</td>
    <td>thread</td>
    <td>时间戳</td>
    <td>age</td>
    <td>1</td>
    <td>01</td>
  </tr>
  <tr>
    <td>轻量级锁</td>
    <td colspan="4">指向栈中锁的指针</td>
    <td>00</td>
  </tr>
  <tr>
    <td>重量级锁</td>
    <td colspan="4">指向重量级锁的指针</td>
    <td>10</td>
  </tr>
  <tr>
    <td>GC标记</td>
    <td colspan="4"></td>
    <td>11</td>
  </tr>
</table>

- 对象头中的类型指针（Klass Pointer）

类型指针用于指向元空间当前类的类元信息，比如调用类中的方法，通过类型指针找到元空间中的该类，在找到相应的方法。开启指针压缩后，类型指针只用4个字节存储，否则需要8个字节存储。

##### 膨胀升级

- 无锁状态：当对象锁被创建出来时，在线程获得该对象锁之前，对象处于无锁状态。
- 偏向锁：在大多数情况下，锁不仅不存在多线程中竞争，而且总是由同一线程多次获得，因为为了减少同一线程获取锁（涉及到一些CAS操作，耗时）的代价而引入偏向锁。偏向锁的核心思想是，一旦有线程持有了这个对象，标志位修改为1，就进入偏向模式，同时会把这个线程的ID记录在对象的 `Mark Work`中。当这个线程再次请求锁时，无需再做任何同步操作，即获得锁的过程，这样就省去了大量有关锁申请的操作，从而也就提升程序的性能。对于锁竞争比较激烈的场合，偏向锁就失效了，因为这样的场合极有可能每次申请锁的线程都是不同的，因此这种场合下不应该使用偏向锁，否则会得不偿失，需要注意的是，偏向锁失败后，并不会立即膨胀为重量级锁，而是先升级为轻量级锁。
- 轻量级锁：如果对象是无锁的，JVM会在当前线程中的栈帧中建立一个 `Lock Record（锁记录）` 的空间，用来存放对象的 `Mark Work` 拷贝，然后把 `Lock Record`中的 `owner` 属性指向当前对象。接下来JVM会利用CAS尝试把对象原本的  `Mark Work` 更新回 `Lock Record` 的指针，成功就说明加锁成功，于是改变锁标志位，执行相关同步操作。如果失败了，判断当前对象的 `Mark Work` 是否指向当前线程中的栈帧，如果是就表示当前线程已经持有该对象锁。如果不是，说明当前对象锁被其他线程持有，于是进行自旋。
- 自旋锁：线程通过不断的自旋尝试上锁，为什么要自旋？因为如果线程被频繁挂起，也就意味着系统在用户态与内核态之间频繁的切换。——我们所有的程序都在用户空间运行，进入运行状态也就是（用户态），但是很多操作可能涉及内核运行，比如I/O，我们就会进入内核运行状态（内核态）。通过自旋，让线程再等待时不会挂起。自旋次数默认为10次，可以通过 `--XX:PreBlockSpin` 进行修改。如果自旋失败到达阈值，即将升级为重量级锁。

注意：锁的膨胀升级，只能升不能降，也就是说升级过程不可逆。

### Volatile

- 保证加了 volatile 关键字的字段的操作具有同步性，以及对 long 和 double 的操作的原⼦性（long double 原⼦性这个简单说⼀下就⾏）。因此 volatile 可以看做是简化版的 synchronized。
- volatile 只对基本类型 (byte、char、short、int、long、ﬂoat、double、boolean) 的赋值操作和对象的引⽤赋值操作有效，你要修改 User.name 是不能保证同步的。
- volatile 依然解决不了 ++ 的原⼦性问题。

### ReentrantLock和Synchronized对比

|                     | **ReentrantLock** | **Synchronized**                                             |
| ------------------- | ----------------- | ------------------------------------------------------------ |
| 底层实现            | 通过`AQS`实现     | 通过`JVM`实现，其中`synchronized`又有多个类型的锁，除了重量级锁是通过`monitor`对象(操作系统mutex互斥原语)实现外，其它类型的通过对象头实现。 |
| 是否可重入          | 是                | 是                                                           |
| 公平锁              | 是                | 否                                                           |
| 非公平锁            | 是                | 是                                                           |
| 锁的类型            | 悲观锁、显式锁    | 悲观锁、隐式锁(内置锁)                                       |
| 是否支持中断        | 是                | 否                                                           |
| 是否支持超时等待    | 是                | 否                                                           |
| 是否自动获取/释放锁 | 否                | 是                                                           |

参考链接：https://juejin.cn/post/6844903805683761165

### 线程安全问题的本质

在多个线程访问共同的资源时，在某**⼀个线程**对资源进⾏**写操作的中途**（写⼊已经开始，但还没结束），**其他线程**对这个**写了⼀半的资源**进⾏了**读操作**，或者基于这个**写了⼀半的资源**进⾏了**写操作**，导致出现**数据错误**。

### 锁机制的本质

通过对共享资源进⾏访问限制，让同⼀时间只有⼀个线程可以访问资源，保证了数据的准确性。

### 小结

不论是线程安全问题，还是针对线程安全问题所衍⽣出的锁机制，它们的核⼼都在于共享的资源，⽽不是某个⽅法或者某⼏⾏代码。

## 集合

### HashMap

效率有保障

- hash的查找效率O(1)
- 链表的查找效率O(n)
- 红黑树的查找效率O(log(n))

#### 位移

高低16位同时参加运算，可以让 hash 分布的更加分散

```
1 <<< 8 无符号左移8位
0000 0000 0000 0000 0000 0000 0000 0001
0000 0000 0000 0000 0000 0001 0000 0000

原来：1 * 2^0
左移后：1 * 2^8 

1 >> 8 有符号右移8位
  
0000 0000 0000 1010 0000 0000 0000 0001
0000 0000 0000 0000 0000 1010 0000 0000

右移后把最右边的给挤掉了


1 ^ 2 异或运算 相同为0不同为1

0000 0000 0000 0000 0000 0000 0000 0001
0000 0000 0000 0000 0000 0000 0000 0010

0000 0000 0000 0000 0000 0000 0000 0011
```

#### HashMap 特点

1. 允许key 为 null ，value 为 null
2. key 不可以重复，value 可以重复
3. 内部是以数组+单链表实现的，JDK8 以后加入了红黑树
4. 内部键值对的存储是无序的，而且顺序会在扩容时改变
5. 初始容量为 16，负载因子为 0.75，也就是当容量达到 容量*负载因子 的时候会扩容，一次扩容增加一倍。
6. 内部的键值对要重写 key 对象重写 hashCode 方法、equals 方法。
7. 线程不安全的，如果想要线程安全，有三种方法
   1. Hashtable 不允许key 为 null，内部使用个 Synchronized 关键字实现线程安全
   2. ConcurrentHashMap java 并发包下的一个线程安全的 HashMap
   3. SynchronizedMap 使用 Collections.synchronizedMap(new HashMap<>()); 来获得一个线程安全的 HashMap,SynchronizedMap内部也是使用的 Synchronized 实现线程安全的

##### 扩容的时机

1. size大于等于 容量 * 加载因子
2. 扩容的大小为原来的2倍，左移一位

**注：** 参考 resize()

##### 树化的时机

1.容量大于等于64
2.链表的长度大于等于8

##### 树化的过程

1. 把普通的 Node 转化为 treeNode
2. 调用 treeify 进行树化

**注：** 非常的耗费资源，需要进行左旋与右旋

##### 插入方式

JDK 1.7 为**头插**

1. 头插不需要进行遍历
2. 设计者认为后插入的一般会先查找

数据结构 链表+数组

多线程容易产生循环链表的问题，导致cpu直接卡死 线程不安全

JDK 1.8 为**尾插**

数据结构 链表+数组 加入了红黑树

使用 ConcurrentHashMap 解决线程不安全的问题

#### 底层数据结构?

采用数组+单链表+红黑树来实现的，

数据中存储的是单链表的头节点或者是红黑树节点。当单链表长度大于 8 的时候，进行树化，当树的长度小于 6 的时候，转化成单链表。

#### 什么是 Hash 冲突？如何解决 Hash冲突？

hash 冲突是在拿到 HashMap 的 key 以后调用 hash 方法去计算哈希值，然后拿这个哈希值和哈希桶的长度 -1 取模运算，得到要把这个 Entry 放入哈希桶的位置，如果这个时候哈希桶的这个位置已经有值了，那么就产生了 hash 冲突。

解决办法就是哈希桶的位置已经有值得情况下去比对当前哈希桶存的链表。

1. 如果头节点的 key 和传入的 key 相等，执行覆盖
2. 如果是红黑树，交给红黑树处理
3. 遍历单链表中的节点，找到相同的 key ，进行覆盖。没找到，插入到最后节点，超过 8，进行树化，提高访问速度。

> 取模用与操作（hash & （arrayLength-1））会比较快，所以数组的大小永远是2的N次方， 你随便给一个初始值比如17会转为32。默认第一次放入元素时的初始值是16。

#### 为什么在树化前会优先考虑扩容，而不是直接树化？

这个主要是从效率和空间上来衡量的，

链表占用空间小，但是查询效率低，

红黑树占用空间大，但是查询效率高

在空间和效率上做了一个平衡。

当 hashmap 的 哈希桶长度大于容量*负载因子的时候，就会扩大哈希桶的容量。一次性扩大两倍，永远是 2 的 n 次方，取模的时候速度比较快，扩容的时候会重新安排所有元素的位置，尽量能够保证均匀的分布在不同的哈希桶中。

#### HashMap 与 HashTable 的不同

首先 HashMap 与 HashTable 实现的父类不一样，一个是AbstractMap(HashMap)，一个是Dictionary(HashTable),因为在 HashTable 里面所有的方法都加了内置锁 `synchronized` ，所以说在 HashTable 里面的方法都是同步方法，它是线程安全的，但 HashMap 是线程不安全的，在 HashTable 里面我们计算 `hash` 值是与 HashMap 是不一样的，HashTable 计算 `hash` 值比较简单，它直接是与数组的长度进行取余，而 HashMap 里面是通过异或取 `hash` 值，并通过高位右移进行异或运算得到的一个 `hash` 值。其次是两者的扩容方式一样，HashTable 是每次扩容 `2n+1` 倍，而 HashMap 每次是扩容 `2n` 倍，然后初始化容量方式不同，通过源码我们可以知道，**HashTable 是在构造方法里面进行数组的初始化，初始化的容量是 11，在 HashMap 中，是第一次 `put` 方法的时候会进行初始化数组，初始化容量是 16，，** 而且要求它的**容量是2的次幂，因为要保证计算其下标的效率**，在 HashMap 里面是通过 容量减一 与上 `hash` 值来计算下标值，但是 HashTable 里面是直接对数组长度取余来计算下标值。

注：HashTable key 与 value 值不能为 null，HashMap key 与 value 可以

#### HashMap 源码分析

##### 变量和常量

```java
public class HashMap<K, V> extends AbstractMap<K, V>
        implements Map<K, V>, Cloneable, Serializable {

    // 默认初始容量：16，必须是 2 的整数次方 ， 1 << 4 相当于 1* 2的四次方 = 16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    // 最大容量 2 的 30 次方。
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认的负载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 链表树化边界值
    static final int TREEIFY_THRESHOLD = 8;
    // 红黑树转换链表的阀值 ，小于 6 的时候，红黑树转换为链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 容器可以树化的最小容量，
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 序列化 id
    private static final long serialVersionUID = 362498820763181265L;
    // 加载因子，可以作为决定扩容的一个因子
    final float loadFactor;
    // 哈希数组，也叫做哈希桶
    transient Node<K, V>[] table;
    //缓存的键值对集合 ， keySet() 和 values() 声明在 AbstractMap 中
    transient Set<Map.Entry<K, V>> entrySet;
    // 目前 hashmap 的大小
    transient int size;
    // 当前 HashMap 修改的次数，这个变量用来保证 fail-fast 机制
    transient int modCount;
    // 扩容边界值，threshold = 容量(table.length)  * 负载因子(loadFactor)
    int threshold;
}
```

##### 构造方法

```java
// 创建一个有初始容量和自定义负载因子的 HashMap
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                loadFactor);
    this.loadFactor = loadFactor;
   // 根据初始容量和负载因子计算阈值
    this.threshold = tableSizeFor(initialCapacity);
}
// 创建一个有初始容量的 HashMap 
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
// 构造一个空的 HashMap 
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
// 根据传入的 Map 创建一个新的 HashMap
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

在第四个构造方法里面调用了 putMapEntries ：

##### putMapEntries()方法

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
   // 拿到传入 Map 的大小
    int s = m.size();
    if (s > 0) {
       // 如果 table==null 说明未创建哈希桶
        if (table == null) { // pre-size
           // 因为传入了 s 个键值对，所以需要的容量就是  s / loadFactor ，为了不触发扩容，多加了 1，
            float ft = ((float) s / loadFactor) + 1.0F;
           // 如果 ft 小于最大容量，使用 ft ，否则使用最大容量。
            int t = ((ft < (float) MAXIMUM_CAPACITY) ?
                    (int) ft : MAXIMUM_CAPACITY);
           // 如果容量大于扩容阈值 threshold ，使用tableSizeFor(t)重新计算扩容阈值 threshold 
            if (t > threshold)
                threshold = tableSizeFor(t);
        } else if (s > threshold)
           // 如果已经创建了哈希桶，并且需要的容量大于目前的扩容阈值 threshold ，resize()扩容
            resize();
       // 循环 传入的 map，调用  putVal 方法循环插入新值。
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

##### Node 单链表实现

```java
//单链表的一个节点，实现了Map.Entry 接口
static class Node<K, V> implements Map.Entry<K, V> {   
  // 我们传入的key 的 hash 值    
  final int hash;   
  // 我们传入的key    
  final K key;   
  // 我们传入的value    
  V value;   
  // 当前节点的后置节点   
  Node<K, V> next;   
  
  Node(int hash, K key, V value, Node<K, V> next) {        
    this.hash = hash;        
    this.key = key;        
    this.value = value;       
    this.next = next;    
  }
}
```

hash 值是确定在哈希桶中的位置的，如果 hash 值和 key 相等，说明是 key 值重复，会执行覆盖操作。如果是没有相同的 key ，直接创建新的链表节点，并把最后一个链表节点指向新创建的节点。

value 就是我们设置的 value 值。

next 就是指向单链表的下个节点的。

可以看出，我们传入的键值对基本都是存储在这个 Node 节点里面的(未树化的情况下，树化的话，存储在树节点)。

##### put()方法

```java
public V put(K key, V value) {   
  return putVal(hash(key), key, value, false, true);
}
```

通过 hash() 方法计算出了 key 的 hash 值

```java
static final int hash(Object key) {   
  int h;   
  return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

如果是 null 直接返回 0；

如果不是 null，h = key.hashCode()，就是调用我们传入的 key 对象的 hashCode() 方法，通过 hashCode 方法计算出来 key 对象的 int 类型的散列值，所以我们在把对象作为 key 的时候，一定要重写 hashCode 方法。

##### putVal()方法

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {    
  //哈希桶 tab    
  Node<K, V>[] tab;    
  // 链表的头结点    
  Node<K, V> p;   
  int n, i;    
  // 把当前哈希桶赋值给 tab,如果为空,或者不为空,但是当前哈希桶的长度为 0    
  // 执行 resize() 创建哈希桶,并且把长度赋值给 n   
  if ((tab = table) == null || (n = tab.length) == 0)        
    n = (tab = resize()).length;    
  // 现根据 当前 哈希桶长度- 1 和 传来的 hash 值进行与运算,得到哈希桶下标,然后从哈希桶中根据坐标取出节点,赋值给 p    
  // 如果节点 p 的值为 null.说明当前哈希桶的位置不存在链表节点,不存在 哈希碰撞,则创建一个新的节点,直接放到哈希桶中.    
  if ((p = tab[i = (n - 1) & hash]) == null)       
    tab[i] = newNode(hash, key, value, null);    
  // 这里是发生了哈希碰撞,说明 p 已经成功被赋值且不为 null.   
  else {        
    // 存在相同 key 的节点.       
    Node<K, V> e;       
    // 头结点 p 的 key 值       
    K k;        
    // 如果头结点 p 的 hash值和传来的相等并且 (头节点的 key 和传来的 key 的地址相等  或者 key值不为 null并且但是和传来的 key 值相等        
    // 说明是 key 已存在,并且相同.如果 onlyIfAbsent 为 false ,会覆盖 .如果为true ,则不覆盖        
    if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))            
      e = p;       
    // p 属于红黑树节点        
    else if (p instanceof TreeNode)           
      e = ((TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);      
    //       
    else {       
      // 开启个无限循环    
      for (int binCount = 0; ; ++binCount) {        
        // 如果当前哈希桶位置的头节点的后置节点为 null,也就是不存在后置节点,         
        if ((e = p.next) == null) {             
          // 新建一个 Node节点,然后让头节点的后置节点指向新创建的 Node 节点.  
          p.next = newNode(hash, key, value, null);        
          // 如果 binCount >= 7 ,执行  treeifyBin 进行树化. 
          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
            treeifyBin(tab, hash);                   
          // 终止循环             
          break;             
        }              
        // 这个时候执行过了 e = p.next ,说明是 e 是 头节点 p 的后置节点.    
        // 如果 e 的 hash值和传来的相等并且 (e 节点的 key 和传来的 key 的地址相等  或者 key值不为 null并且但是和传来的 key 值相等      
        // 说明在链表中找到了同hash、key的节点，那么直接退出循环      
        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))     
          break;               
        // p 指向下一个节点.继续下次循环            
        p = e;      
      }     
    }      
    // 如果当前 e 节点不为 null ,说明存在 key 相同的节点.那么直接覆盖原节点.   
    if (e != null) { 
      // existing mapping for key      
      V oldValue = e.value;          
      // 如果onlyIfAbsent 或者 找到的节点的 值为 null ,则覆盖原节点的 value值.    
      if (!onlyIfAbsent || oldValue == null)             
        e.value = value;        
      afterNodeAccess(e);         
      // 返回旧的 value 值.      
      return oldValue;  
    }   
  }    
  // 操作数 +1;  
  ++modCount;  
  // 如果新增后的大小超过了阈值.执行 resize() 扩容.   
  if (++size > threshold)     
    resize();   
  afterNodeInsertion(evict);   
  return null;
}
```

首先在刚开始的时候，先判断当前哈希桶有没有创建，如果没创建，执行 resize() 创建哈希桶

接下来就通过 hash 值和哈希桶长度 -1 做 取模运算，得到哈希桶的坐标。从哈希桶中取出这个位置的 节点赋值给 p。

没有发生 hash 碰撞
如果头节点 p 为 null ，说明没有发生哈希碰撞，直接放入头节点，就结束了。

发生了 hash 碰撞
如果头节点 p 不为 null ，说明当前哈希桶位置已经有值，就发生了 hash 碰撞了。

1. 先判断当前头节点的 key 是否相等，这个会根据 hash 值、key 是否为 null，不为 null 的时候，调用 key 的equals 方法进行比较 key 是否相等。如果为 null ，或者 key 相等，待会就会执行覆盖。这个地方也说明了，key 是可以为 null 的。也验证了 key 是不能重复的，重复就会覆盖(只有onlyIfAbsent 为 false 的时候会覆盖)。
2. 如果头节点属于红黑树节点，进行红黑树相关处理
3. 如果不是，意思就是发生了hash 碰撞，需要遍历单链表的节点解决问题。在循环中判断：
   1. 如果取到的节点的 next 为 null ，说明没有后置节点，直接新建节点，并把当前节点的后置节点指向它。如果达到树化的值，就会去树化。否则跳出循环，插入结束。
   2. 取到的节点的 next 不为 null，如果在单链表中找到了与之 key 值相等的，也跳出循环，去执行覆盖。没找到相等的继续循环，找下一个，一直会在找到的节点的 next 为 null 或者 key 值相等的时候结束循环。
   3. 循环结束，如果当前 e 节点不为 null ,说明存在 key 相同的节点.那么直接覆盖原节点.
4. 最后增加操作数，判断是否需要扩容，如果需要，执行 resize 扩容。

**注：** 在 putVal 方法中有一个参数，onlyIfAbsent 这个参数意思是如果为 true ，在key 值相同的时候，不会覆盖原值。我们调用 put 能覆盖，是因为在 put 方法调用 putVal 的时候传入的是 false，如果不想在 key 重复的时候覆盖，可以使用 putIfAbsent 方法。

##### resize()方法

```java
 final Node<K, V>[] resize() {       
   // 哈希桶大小        
   Node<K, V>[] oldTab = table;   
   // 老的容量.如果没创建 哈希桶,返回0  
   int oldCap = (oldTab == null) ? 0 : oldTab.length;  
   // 老的扩容边界值      
   int oldThr = threshold;  
   // 新的容量,新的扩容边界值.   
   int newCap, newThr = 0;    
   //如果老的容量大于 0 .说明已经创建了 哈希桶  
   if (oldCap > 0) {        
     //  如果超过了最大容量      
     if (oldCap >= MAXIMUM_CAPACITY) {  
       // 扩容边界值就是 Integer 的最大值  
       threshold = Integer.MAX_VALUE;   
       // 返回老的哈希桶,不再扩容          
       return oldTab;             
       //新容量 = 老容量*2 小于最大容量,并且老容量大于等于默认的初始容量   
     } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&  
                oldCap >= DEFAULT_INITIAL_CAPACITY)           
       // 计算新的扩容边界值 *2                
       newThr = oldThr << 1; 
     // double threshold        
     // 如果当前表是空的，但是有阈值。代表是初始化时指定了容量、阈值的情况,直接使用旧的扩容边界值作为新的容量,
   } else if (oldThr > 0) 
     // initial capacity was placed in threshold         
     newCap = oldThr;      
   // 容量为 0,容量未初始化,设置默认容量和扩容边界值   
   else {            
     // zero initial threshold signifies using defaults   
     newCap = DEFAULT_INITIAL_CAPACITY;   
     newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);   
   }     
   // 如果新的扩容边界值为0,对应的是表是空的,但是有扩容边界值的情况.   
   if (newThr == 0) {    
     //根据新表容量 和 加载因子 求出新的阈值   
     float ft = (float) newCap * loadFactor;     
     // 进行越界修复       
     newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ?                    (int) ft : Integer.MAX_VALUE);     
   }      
   // 更新扩容边界值 
   threshold = newThr;       
   @SuppressWarnings({"rawtypes", "unchecked"}) 
   // 根据新的容量创建一个新的 哈希桶     
   Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap]; 
   //更新哈希桶      
   table = newTab;    
   // 遍历老的 哈希桶,然后转移内部元素到新的哈希桶 
   if (oldTab != null) {      
     for (int j = 0; j < oldCap; ++j) {    
       Node<K, V> e;   
       // 取出旧哈希桶中的节点(是一个链表中的头节点) 赋值给 e ,并且不为null   
       if ((e = oldTab[j]) != null) {         
         // 原来桶中的头节点置为 null          
         oldTab[j] = null;    
         // 如果没有下个节点,就是将这个位置没发生哈希碰撞. 
         if (e.next == null)                   
           // 根据原节点的 hash 值和新哈希桶的长度 -1 计算出新 哈希桶中的位置,把 e 放进去       
           newTab[e.hash & (newCap - 1)] = e;        
         // 处理红黑树及诶单       
         else if (e instanceof TreeNode)    
           ((TreeNode<K, V>) e).split(this, newTab, j, oldCap);   
         //这里处理的是发生过哈希碰撞,并且单链表长度不满 8 个的情况,先将其分组,再进行映射.   
         else { 
           // preserve order   
           //低位链表的头结点、尾节点         
           Node<K, V> loHead = null, loTail = null;          
           //高位链表的头结点、尾节点             
           Node<K, V> hiHead = null, hiTail = null;    
           //临时节点 存放e的下一个节点     
           Node<K, V> next;          
           //                      
           do {                  
             //取出下个节点          
             next = e.next;      
             // 等于 0 是低位.,扩容后哈希桶位置不变. 
             if ((e.hash & oldCap) == 0) {           
               if (loTail == null)                      
                 loHead = e;                           
               else                            
                 loTail.next = e;    
               loTail = e;      
               // 不等于 0 是高位.扩容后位置会变化     
             } else {                  
               if (hiTail == null)           
                 hiHead = e;     
               else                
                 hiTail.next = e;  
               hiTail = e;         
             }                    
           } while ((e = next) != null);   
           // 低位的头结点和尾节点.  哈希桶坐标和原坐标一样        
           if (loTail != null) {        
             loTail.next = null;              
             newTab[j] = loHead;          
           }                     
           // 高位的头结点和尾节点.扩容后的 哈希桶坐标 = 原坐标一样 + 扩容的大小(原坐标的大小)     
           if (hiTail != null) {                  
             hiTail.next = null;           
             newTab[j + oldCap] = hiHead;     
           }                 
         }          
       }           
     }    
   }     
   return newTab; 
 }
```

主要分为以下几步：

1. 旧容量大于 0， 如果超过了最大值 MAXIMUM_CAPACITY，不再进行扩容，返回。没有超过最大值的话扩容为 2 倍。
2. 旧容量小于等于 0，如果旧的扩容边界值大于 0 ，把旧的扩容边界值赋值给新的扩容边界值
3. 旧容量小于等于 0，并且旧的扩容边界值小于等于 0 ，也就是没指定，就初始化默认的容量和扩容边界值
4. 如果新的扩容边界值等于 0，根据新容量和新负载因子计算新的扩容边界值
5. 然后根据新的容量进行创建一个新的哈希桶，并进行遍历
6. 取出头节点，如果头节点后置节点为 null ，表明单链表只有一个头节点，没有发生哈希碰撞，直接放入新数组。
7. 如果头节点是属于 树节点，调用 split 去处理，这里不再详细分析。
8. 否则就说明是此位置的单链表发生了哈希碰撞，通过定义loHead、loTail、hiHead、hiTail来讲一个链表拆分成两个独立的链表。注意：如果e的hash值与老表的容量进行与运算为0,则扩容后的索引位置跟老表的索引位置一样。所以loHead-->loTail组成的链表在新Map中的索引位置和老Map中是一样的。如果e的hash值与老表的容量进行与运算为1,则扩容后的索引位置为:老表的索引位置＋oldCap，最后将loHead、loTail、hiHead、hiTail组成的两条链表重新定位到新的Map中即可

##### remove()方法

```java
public V remove(Object key) {  
  Node<K, V> e;   
  // 计算散列码 
  // 根据 key 调用 removeNode  
  return (e = removeNode(hash(key), key, null, false, true)) == null ?    
    null : e.value;}

/** 
 * Implements Map.remove and related methods 
 * 
 * @param hash       hash for key 
 * @param key        the key 
 * @param value      the value to match if matchValue, else ignored 
 * @param matchValue if true only remove if value is equal 
 * @param movable    if false do not move other nodes while removing 
 * @return the node, or null if none 
 */
final Node<K, V> removeNode(int hash, Object key, Object value,      
                            boolean matchValue, boolean movable) {  
  Node<K, V>[] tab;  
  Node<K, V> p;  
  int n, index;  
  // 如果哈希桶不为 null,并且长度大于 0,并且 哈希桶位置的头节点不为 null  
  if ((tab = table) != null && (n = tab.length) > 0 &&(p = tab[index = (n - 1) & hash]) != null) {  
    Node<K, V> node = null, e;  
    K k;    
    V v;  
    // 头节点就是要移除的,赋值给 node  
    if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))      
      node = p;  
    // 头节点不是要移除的,并且有后置节点    
    else if ((e = p.next) != null) {       
      // 树节点,红黑树处理       
      if (p instanceof TreeNode)   
        node = ((TreeNode<K, V>) p).getTreeNode(hash, key);  
      // 单链表处理         
      else {        
        do {           
          // 循环,一直找到 key 值相等的节点跳出循环,否则最后跳出循环     
          if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) { 
            node = e;                 
            break;       
          }              
          p = e;           
        } while ((e = e.next) != null);    
      }     
    }    
    // 如果找到了目标删除节点,执行删除操作,并返回删除的目标节点,否则返回 null   
    if (node != null && (!matchValue || (v = node.value) == value ||    
                         (value != null && value.equals(v)))) {     
      if (node instanceof TreeNode)       
        ((TreeNode<K, V>) node).removeTreeNode(this, tab, movable);      
      else if (node == p)         
        tab[index] = node.next;        
      else            
        p.next = node.next;   
      ++modCount;         
      --size;         
      afterNodeRemoval(node);   
      return node;     
    }  
  }
  return null;
}
```

##### replace() 方法

```java
@Override
public V replace(K key, V value) { 
  Node<K, V> e;    
  // 调用 getNode 通过 key 找到节点,并且不为 null,覆盖原节点值. 
  if ((e = getNode(hash(key), key)) != null) {  
    V oldValue = e.value;      
    e.value = value;      
    afterNodeAccess(e);      
    return oldValue;  
  }  
  return null;
}
```

##### get()方法

```java
public V get(Object key) { 
  Node<K, V> e;     
  // 利用 hash 扰乱 key 的散列值    
  // 调用 getNode 找 key 相等的节点       
  return (e = getNode(hash(key), key)) == null ? null : e.value;  
}    

/**     
* Implements Map.get and related methods     
*     
* @param hash hash for key     
* @param key  the key     
* @return the node, or null if none     
*/    
final Node<K, V> getNode(int hash, Object key) {    
  // 哈希桶      
  Node<K, V>[] tab;  
  // first 头节点和要找的节点 
  Node<K, V> first, e;  
  //哈希桶长度      
  int n;   
  K k;       
  // 如果哈希桶不为 null,并且长度大于 0,并且 哈希桶位置的头节点不为 null      
  if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) { 
    // 如果头节点满足,就返回头节点        
    if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k))))    
      return first;        
    // 如果头节点的后置节点不为 null  
    if ((e = first.next) != null) {      
      // 如果属于红黑树,去红黑树中去查           
      if (first instanceof TreeNode)            
        return ((TreeNode<K, V>) first).getTreeNode(hash, key);  
      // 不属于红黑树,在单链表中查            
      do {          
        // 一直找到 key 值相等的节点.如果没找到并且最后一个节点的后置节点为 null ,说明没有key 值相等的,退出循环,返回 null        
        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))   
          return e;       
      } while ((e = e.next) != null);  
    }     
  }  
  return null; 
}
```

get 方法大概就是这样，还是先计算 hash 值，然后调用 getNode 找节点，找到说明存在，返回节点，最后返回节点的 value 值。

没找到说明不存在，则返回 null；

##### 遍历

```java
//迭代器遍历
Set keys = map.keySet();
Iterator ite = keys.iterator();
while (ite.hasNext()) {   
  Object key = ite.next();  
  key.key;  
  key.value;
}
// entrySet 遍历
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {   
  System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
}
// 在for-each循环中遍历keys或values。
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
//遍历map中的键
for (Integer key : map.keySet()) {  
  System.out.println("Key = " + key); 
  System.out.println("value = " + map.get(key));
}
//遍历map中的值
for (Integer value : map.values()) { 
  System.out.println("Value = " + value);
}
```

#### 你知道 HashMap 吗？了解有多少？

你好，HashMap 是我们日常开发当中几乎每天都要用到的这么一个集合类，它是以**键值对**的形式进行存储，在 JDK1.7 到 JDk1.8 之间 HashMap 的实现略有区别，其中最重要的两个区别在 JDK1.7 的时候 HashMap 采用的数据结构是**数组+链表**，但是到了 JDK1.8 之后数据结构是 **数组+链表+红黑树**，红黑树的引入是为了提高他的查询效率，因为我们的链表的查询，查询时间的复杂度是  `O(n)` ,但是我们的红黑树的是 `O(log(n))` ,还有一点是，在 JDK1.7 之前当我们遇到哈希碰撞需要在链表上添加数据的时候，采用的是**头插法**，但是到了 JDK1.8 之后改用了**尾插法**，嗯，因为头插法在多线程的情况下，会导致一些问题啊，比如说，他会形成**循环链表**啊，耗尽我们 CPU 的性能，恩，为了解决这个问题，在 JDK1.8 之后改用了尾插法，当然，JDK1.7 和 JDK1.8 之间还有很多优化的细节啊，这个我可能记得不大清楚了，比如说他的 hash 算法，进行了一个简化，还有一些其他的东西，我们需要在源码里面才能把它看的更透，当然这个源码我是读过的啊，只是这些东西，啊，可能是记得不太清楚了，接下来呢，我就 JDK1.8 和您聊一聊 HashMap 的一些基本原理，首先啊，我们在创建 HashMap 的时候，阿里规约里面要求我们需要传入一个初始化容量，就是在我们预知的前提下，我们将来可能要插入多少数据的前提下，我们最好传入一个初始化容量，而且这个初始化容量最好是一个二的次幂，我接下来可能和你接着聊，为什么是二的次幂，首先呢，当我们往 HashMap 中 put 值的时候，当 put 第一个值的时候，我们的刚才所说的不管是**数组加链表**还是**数组加链表加红黑树**的数组会不会初始化，按照我们传入的初始化容量，大于等于这个初始化容量的最近的一个二次幂，啊，这么一个值，给我们进行初始化数组，啊，然后初始化之后呢，他会使用他的 key 的 hash 值 **与(&)** 上它的容量，就是我们的这个二的次幂减一算出他的下标，也因为我们的容量都是二的次幂，二的次幂减一之后它所有的低位都是一，高位都是零，和咱们的哈希进  **与(&)** 运算之后他一定能 **与(&)** 出来一个下标在咱们的容量范围之内的一个下标值。因为 **与(&)** 运算在计算机里面效率是非常的高，所以它采取的是 **与(&)** 运算而不是取余运算，取余运算的效率非常的慢，当然呢，在咱们往里面添加数据的时候会产生两个问题，**一个问题就是扩容的问题，一个问题就是树化的问题**，关于扩容的问题，在 HashMap 里面有一个成员变量它叫**加载因子**，当我们 HashMap 的 Size 就是你**插入节点的数量大于等于容量乘以扩容因子，也就是加载因子**，也就是 16 * 0.75，也就是当 Size 大于 12 的时候它就会进行一次扩容，当然，当我们的链表上悬挂的数据节点足够多的时候，它还会进行树化，当然树化其实是一个很耗性能的，那当然树化和扩容都是一个很耗性能的操作，**树化的前提就是我们的链表长度必须大于等于八** ，当然这还不够，在我们的 HashMap 的源码里面还有一个成员变量，它让我们清晰的看到，这个成员变量就是最小的树化的一个容量，意思就是咱们的那个数组，如果数组的容量达不到64，它默认是64，啊，它会优先选择扩容，而不是对咱们的链表进行树化，所以树化它是有两个先决条件的，第一个它要满足的条件就是，我们的容量，数组的容量要大于等于64，第二个就是链表的长度要大于等于8，那么，阿里规约里面要求我们传入初始化容量，其实根本的目的就是在于少扩容，那我我们也可以在阿里规约里面找到这个初始化容量的一些计算方式，我记得应该是，你**将来要存入的数据的数量除以扩容因子然后加一**，应该是这么一个算法，关于 HashMap 我了解的大概就这么多。

### List、Map、Set的区别与联系

List和Set是存储单列数据的集合，Map是存储键值对这样的双列数据的集合；

List中存储的数据是有顺序的，并且值允许重复；

Map中存储的数据是无序的，它的键是不允许重复的，但是值是允许重复的；

Set中存储的数据是无顺序的，并且不允许重复，但元素在集合中的位置是由元素的hashcode决定，即位置是固定的（Set集合是根据hashcode来进行数据存储的，所以位置是固定的，但是这个位置不是用户可以控制的，所以对于用户来说set中的元素还是无序的）。

List接口有三个实现类：
```text
1.1 LinkedList
基于链表实现，链表内存是散列的，增删快，查找慢；

适用场景：适合单线程中，顺序读取，不适合随机读取和随机删除。

1.2 ArrayList
基于数组实现，非线程安全，效率高，增删慢，查找快；

初始容量：10初始容量：10
扩容容量：(原始容量 x 3 ) / 2 + 1。
适用场景：ArrayList 适合单线程，或多线程环境，但 List 只会被单个线程操作；随机查找和遍历，不适合插入和删除。

1.3 Vector
基于数组实现，线程安全，效率低，增删慢，查找慢；

初始容量：10初始容量：10
扩容容量：若扩容系数 > 0，则将容量的值增加“扩容系数”；否则，将容量大小增加一倍。
适用场景：能不用就别用。
```
Map接口有四个实现类：
```text
2.1HashMap
基于hash表的Map接口实现，非线程安全，高效，支持null值和null键；
2.2HashTable
线程安全，低效，不支持null值和null键；
2.3LinkedHashMap
是HashMap的一个子类，保存了记录的插入顺序；
2.4SortMap接口
TreeMap，能够把它保存的记录根据键排序，默认是键值的升序排序
```
Set接口有两个实现类：
```text
3.1 HashSet
底层是由 Hash Map 实现，不允许集合中有重复的值，使用该方式时需要重写 equals()和 hash Code()方法；
3.2 LinkedHashSet
继承于 HashSet，同时又基于 LinkedHashMap 来进行实现，底层使用的是 LinkedHashMap
```
#### 区别
1.List集合中对象按照索引位置排序，可以有重复对象，允许按照对象在集合中的索引位置检索对象，例如通过list.get(i)方法来获取集合中的元素；

2.Map中的每一个元素包含一个键和一个值，成对出现，键对象不可以重复，值对象可以重复；

3.Set集合中的对象不按照特定的方式排序，并且没有重复对象，但它的实现类能对集合中的对象按照特定的方式排序，例如TreeSet类，可以按照默认顺序，也可以通过实现Java.util.Comparator<Type>接口来自定义排序方式。

#### 补充：HashMap和HashTable
HashMap是线程不安全的,HashMap是一个接口,是Map的一个子接口,是将键映射到值得对象,不允许键值重复,允许空键和空值;由于非线程安全,HashMap的效率要较HashTable的效率高一些.

HashTable是线程安全的一个集合,不允许null值作为一个key值或者Value值;

HashTable是sychronize(同步化),多个线程访问时不需要自己为它的方法实现同步,而HashMap在被多个线程访问的时候需要自己为它的方法实现同步;

## Handler

### MessageQueue源码分析

![MessageQueue源码分析](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-234630.png)

### 流程

主线程默认实现了Looper （调用loop.prepare方法 向sThreadLocal中set一个新的looper对象， looper构造方法中又创建了MsgQueue） 手动创建Handler ，调用 sendMessage 或者 post (runable) 发送Message 到 msgQueue ，如果没有Msg 这添加到表头，有数据则判断when时间 循环next 放到合适的 msg的next 后。Looper.loop不断轮训Msg，将msg取出 并分发到Handler 或者 post提交的 Runable 中处理，并重置Msg 状态位。回到主线程中 重写 Handler 的 handlerMessage 回调的msg 进行主线程绘制逻辑。

### 系统为什么提供Handler

这点大家应该都知道一些，就是为了切换线程，主要就是为了解决在子线程无法访问UI的问题。

一种东西被设计出来肯定就有它存在的意义，而 `Handler` 的意义就是切换线程。
作为 `Android` 消息机制的主要成员，它管理着所有与界面有关的消息事件，常见的使用场景有：

- **跨进程之后的界面消息处理**。

比如Activity的启动，就是AMS在进行进程间通信的时候，通过Binder线程 将消息发送给 `ApplicationThread` 的消息处理者 `Handler` ，然后再将消息分发给主线程中去执行。

- **网络交互后切换到主线程进行UI更新**

当子线程网络操作之后，需要切换到主线程进行UI更新。

总之一句话，`Hanlder` 的存在就是为了解决在子线程中无法访问UI的问题。

#### 那么为什么系统不允许在子线程中访问UI呢？

因为Android中的UI控件不是线程安全的，如果多线程访问UI控件那还不乱套了。

##### 那么为什么不给UI控件加锁呢？

- **会降低UI访问的效率**。本身UI控件就是离用户比较近的一个组件，加锁之后自然会发生阻塞，那么UI访问的效率会降低，最终反应到用户端就是这个手机有点卡。

- **太复杂了**。本身UI访问时一个比较简单的操作逻辑，直接创建UI，修改UI即可。如果加锁之后就让这个UI访问的逻辑变得很复杂，没必要

因为加锁会让UI访问的逻辑变得复杂，而且会降低UI访问的效率，阻塞线程执行。

所以，Android设计出了 单线程模型 来处理UI操作，再搭配上Handler，是一个比较合适的解决方案。

#### 子线程访问UI的 崩溃原因 和 解决办法？

崩溃发生在ViewRootImpl类的 `checkThread` 方法中：

```java
 void checkThread() {   
   if (mThread != Thread.currentThread()) {  
     throw new CalledFromWrongThreadException( 
       "Only the original thread that created a view hierarchy can touch its views.");   
   }  
 }  
```

其实就是判断了当前线程 是否是 `ViewRootImpl` 创建时候的线程，如果不是，就会崩溃。

而 `ViewRootImpl` 创建的时机就是界面被绘制的时候，也就是  `onResume` 之后，所以如果在子线程进行UI更新，就会发现当前线程（子线程）和 `View` 创建的线程（主线程）不是同一个线程，发生崩溃。

解决办法有三种：

- 在新建视图的线程进行这个视图的UI更新，主线程创建View，主线程更新View。
- 在 `ViewRootImpl` 创建之前进行子线程的UI更新，比如onCreate方法中进行子线程更新UI。
- 子线程切换到主线程进行UI更新，比如Handler、view.post方法。

#### Handler是怎么获取到当前线程的Looper的

大家应该都知道Looper是绑定到线程上的，他的作用域就是线程，而且不同线程具有不同的Looper，也就是要从不同的线程取出线程中的Looper对象，这里用到的就是ThreadLocal。

假设我们不知道有这个类，如果要完成这样一个需求，从不同的线程获取线程中的Looper，是不是可以采用一个全局对象，比如hashmap，用来存储线程和对应的Looper？

所以需要一个管理Looper的类，但是，线程中并不止这一个要存储和获取的数据，还有可能有其他的需求，也是跟线程所绑定的。所以，我们的系统就设计出了ThreadLocal这种工具类。

ThreadLocal的工作流程是这样的：我们从不同的线程可以访问同一个ThreadLocal的get方法，然后ThreadLocal会从各自的线程中取出一个数组，然后再数组中通过ThreadLocal的索引找出对应的value值。具体逻辑呢，我们还是看看代码，分别是ThreadLocal的get方法和set方法：

```java
public void set(T value) {  
  Thread t = Thread.currentThread();  
  ThreadLocalMap map = getMap(t);   
  if (map != null)     
    map.set(this, value); 
  else      
    createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {   
  return t.threadLocals;
}    

public T get() { 
  Thread t = Thread.currentThread();  
  ThreadLocalMap map = getMap(t);  
  if (map != null) {    
    ThreadLocalMap.Entry e = map.getEntry(this);      
    if (e != null) {      
      @SuppressWarnings("unchecked")    
      T result = (T)e.value;      
      return result;     
    } 
  }  
  return setInitialValue();
}    
```

首先看看set方法，获取到当前线程，然后取出线程中的threadLocals变量，是一个ThreadLocalMap类，然后将当前的ThreadLocal作为key，要设置的值作为value存到这个map中。

get方法就同理了，还是获取到当前线程，然后取出线程中的ThreadLocalMap实例，然后从中取到当前ThreadLocal对应的值。

其实可以看到，操作的对象都是线程中的ThreadLocalMap实例，也就是读写操作都只限制在线程内部，这也就是ThreadLocal故意设计的精妙之处了，他可以在不同的线程进行读写数据而且线程之间互不干扰。

画个图方便理解记忆：

![ThreadLocal对象](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-234749.png)

#### MessageQueue是干嘛呢？用的什么数据结构来存储数据？

看名字应该是个队列结构，队列的特点是什么？**先进先出**，一般在队尾增加数据，在队首进行取数据或者删除数据。

那 `Hanlder` 中的消息似乎也满足这样的特点，先发的消息肯定就会先被处理。但是，`Handler`中还有比较特殊的情况，比如延时消息。

延时消息的存在就让这个队列有些特殊性了，并不能完全保证先进先出，而是需要根据时间来判断，所以 `Android` 中采用了链表的形式来实现这个队列，也方便了数据的插入。

来一起看看消息的发送过程，无论是哪种方法发送消息，都会走到sendMessageDelayed方法

```java
    public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis) {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }

    public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```

`sendMessageDelayed` 方法主要计算了消息需要被处理的时间，如果delayMillis为0，那么消息的处理时间就是当前时间。

然后就是关键方法 `enqueueMessage`。

```java
    boolean enqueueMessage(Message msg, long when) {
        synchronized (this) {
            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (; ; ) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p;
                prev.next = msg;
            }
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```

不懂得地方先不看，只看我们想看的：

- 首先设置了`Message` 的when字段，也就是代表了这个消息的处理时间
- 然后判断当前队列是不是为空，是不是即时消息，是不是执行时间when大于表头的消息时间，满足任意一个，就把当前消息msg插入到表头。
- 否则，就需要遍历这个队列，也就是**链表**，找出when小于某个节点的when，找到后插入。

好了，其他内容暂且不看，总之，插入消息就是通过消息的执行时间，也就是 `when` 字段，来找到合适的位置插入链表。

具体方法就是通过死循环，使用快慢指针p和prev，每次向后移动一格，直到找到某个节点p的when大于我们要插入消息的when字段，则插入到p和prev之间。或者遍历到链表结束，插入到链表结尾。

所以，`MessageQueue` 就是一个用于存储消息、用链表实现的特殊队列结构。

#### 延迟消息是怎么实现的？

总结上述内容，延迟消息的实现主要跟消息的统一存储方法有关，也就是上文说过的 `enqueueMessage` 方法。

无论是即时消息还是延迟消息，都是计算出具体的时间，然后作为消息的 `when` 字段进程赋值。

然后在 `MessageQueue` 中找到合适的位置（安排when小到大排列），并将消息插入到 `MessageQueue` 中。

这样，`MessageQueue` 就是一个按照消息时间排列的一个链表结构。

#### MessageQueue的消息怎么被取出来的？

刚才说过了消息的存储，接下来看看消息的取出，也就是 `queue.next` 方法。

```java
Message next() {
        for (; ; ) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }
            nativePollOnce(ptr, nextPollTimeoutMillis);
            synchronized (this) {
                // Try to retrieve the next message.  Return if found.   
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.            
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.              
                    nextPollTimeoutMillis = -1;
                }
            }
        }
    }
```

奇怪，为什么取消息也是用的死循环呢？

其实死循环就是为了保证一定要返回一条消息，如果没有可用消息，那么就阻塞在这里，一直到有新消息的到来。

其中，`nativePollOnce` 方法就是阻塞方法，`nextPollTimeoutMillis` 参数就是阻塞的时间。

那什么时候会阻塞呢？两种情况：

1. 有消息，但是当前时间小于消息执行时间，也就是代码中的这一句

```java
if (now < msg.when) {   
  nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
}
```

这时候阻塞时间就是消息时间减去当前时间，然后进入下一次循环，阻塞。

2. 没有消息的时候，也就是上述代码的最后一句：

```java
if (msg != null) {
  
}    
else { 
  // No more messages.   
  nextPollTimeoutMillis = -1;   
}
```

`-1` 就代表一直阻塞。

#### 当MessageQueue 没有消息的时候，在干什么，会占用CPU资源吗

MessageQueue 没有消息时，便阻塞在 loop 的 queue.next() 方法这里。具体就是会调用到nativePollOnce方法里，最终调用到epoll_wait()进行阻塞等待。

这时，主线程会进行休眠状态，也就不会消耗CPU资源。当下个消息到达的时候，就会通过pipe管道写入数据然后唤醒主线程进行工作。

这里涉及到阻塞和唤醒的机制叫做 epoll 机制。

##### 先说说文件描述符和I/O多路复用

> 在Linux操作系统中，可以将一切都看作是文件,而文件描述符简称fd，当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符，可以理解为一个索引值。

> I/O多路复用是一种机制，让单个进程可以监视多个文件描述符，一旦某个描述符就绪（一般是读就绪或写就绪），能够通知程序进行相应的读写操作

所以I/O多路复用其实就是一种监听读写的通知机制，而Linux提供的三种 IO 复用方式分别是：select、poll 和 epoll 。而这其中epoll是性能最好的多路I/O就绪通知方法。

所以，这里用到的epoll其实就是一种I/O多路复用方式，用来监控多个文件描述符的I/O事件。通过epoll_wait方法等待I/O事件，如果当前没有可用的事件则阻塞调用线程。

**epoll机制**是一种IO多路复用的机制，具体逻辑就是一个进程可以监视多个描述符，当某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作，这个读写操作是阻塞的。在Android中，会创建一个 `Linux管道(Pipe)` 来处理阻塞和唤醒。

- 当消息队列为空，管道的读端等待管道中有新内容可读，就会通过**epoll机制**进入阻塞状态。
- 当有消息要处理，就会通过管道的写端写入内容，唤醒主线程。

#### Handler 同步屏障机制

通过发送异步消息，在msg.next 中会优先处理异步消息，达到优先级的作用。

#### 同步屏障和异步消息是怎么实现的？

其实在Handler机制中，有三种消息类型：

- **同步消息**。也就是普通的消息。
- **异步消息**。通过setAsynchronous(true)设置的消息。
- **同步屏障消息**。通过postSyncBarrier方法添加的消息，特点是target为空，也就是没有对应的handler。

这三者之间的关系如何呢？

- 正常情况下，同步消息和异步消息都是正常被处理，也就是根据时间when来取消息，处理消息。
- 当遇到同步屏障消息的时候，就开始从消息队列里面去找异步消息，找到了再根据时间决定阻塞还是返回消息。

也就是说同步屏障消息不会被返回，他只是一个标志，一个工具，遇到它就代表要去先行处理异步消息了。

所以同步屏障和异步消息的存在的意义就在于有些消息需要“**加急处理**”。

#### 同步屏障和异步消息有具体的使用场景吗？

使用场景就很多了，比如绘制方法 `scheduleTraversals` 。

```java
void scheduleTraversals() {   
  if (!mTraversalScheduled) {  
    mTraversalScheduled = true;   
    // 同步屏障，阻塞所有的同步消息        
    mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();    
    // 通过 Choreographer 发送绘制任务        
    mChoreographer.postCallback(                 
      Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);   
  }   
}   
Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);   
msg.arg1 = callbackType; 
msg.setAsynchronous(true);  
mHandler.sendMessageAtTime(msg, dueTime);
```

在该方法中加入了同步屏障，后续加入一个异步消息`MSG_DO_SCHEDULE_CALLBACK`，最后会执行到`FrameDisplayEventReceiver`，用于申请`VSYNC`信号。
更多Choreographer相关内容可以看看这篇文章——<https://www.jianshu.com/p/86d00bbdaf60>

#### Message消息被分发之后会怎么处理？消息怎么复用的？

再看看loop方法，在消息被分发之后，也就是执行了`dispatchMessage`方法之后，还偷偷做了一个操作——`recycleUnchecked`。

```java

    public static void loop() {
        for (; ; ) {
            Message msg = queue.next();
            // might block   
            try {
                msg.target.dispatchMessage(msg);
            } msg.recycleUnchecked();
        }
    }

    //Message.java   
    private static Message sPool;
    private static final int MAX_POOL_SIZE = 50;

    void recycleUnchecked() {
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = UID_NONE;
        workSourceUid = UID_NONE;
        when = 0;
        target = null;
        callback = null;
        data = null;
        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }
```

在 `recycleUnchecked` 方法中，释放了所有资源，然后将当前的空消息插入到sPool表头。
这里的 `sPool` 就是一个消息对象池，它也是一个链表结构的消息，最大长度为50。

##### Messaage复用

将使用完的Message清除附带的数据后, 添加到复用池中 ,当我们需要使用它时,直接在复用池中取出对象使用,而不需要重新new创建对象。复用池本质还是Message 为node 的单链表结构。所以推荐使用Message.obation获取 对象。

那么Message又是怎么复用的呢？在Message的实例化方法 `obtain` 中：

```java
  public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0;
                // clear in-use flag   
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }
```

直接复用消息池sPool中的第一条消息，然后sPool指向下一个节点，消息池数量减一。

#### Looper是干嘛呢？怎么获取当前线程的Looper？为什么不直接用Map存储线程和对象呢？

在Handler发送消息之后，消息就被存储到 `MessageQueue` 中，而 `Looper` 就是一个管理消息队列的角色。`Looper` 会从 `MessageQueue` 中不断的查找消息，也就是 `loop` 方法，并将消息交回给 `Handler` 进行处理。

而Looper的获取就是通过ThreadLocal机制:

```java

    static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

    public static @Nullable
    Looper myLooper() {
        return sThreadLocal.get();
    }
```

通过prepare方法创建Looper并且加入到sThreadLocal中，通过myLooper方法从sThreadLocal中获取Looper。

#### ThreadLocal运行机制？这种机制设计的好处？

下面就具体说说 `ThreadLocal` 运行机制。

```java

    //ThreadLocal.java  
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked") T result = (T) e.value;
                return result;
            }
        }
        return setInitialValue();
    }

    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) map.set(this, value);
        else createMap(t, value);
    }
```

从 `ThreadLocal` 类中的get和set方法可以大致看出来，有一个 `ThreadLocalMap` 变量，这个变量存储着键值对形式的数据。

- `key`为this，也就是当前ThreadLocal变量。
- `value`为T，也就是要存储的值。

然后继续看看 `ThreadLocalMap` 哪来的，也就是getMap方法：

```java
//ThreadLocal.java   
ThreadLocalMap getMap(Thread t) {  
  return t.threadLocals;  
}   
//Thread.java  
ThreadLocal.ThreadLocalMap threadLocals = null;
```

原来这个`ThreadLocalMap`变量是存储在线程类Thread中的。

所以`ThreadLocal`的基本机制就搞清楚了：

在每个线程中都有一个threadLocals变量，这个变量存储着ThreadLocal和对应的需要保存的对象。

这样带来的好处就是，在不同的线程，访问同一个ThreadLocal对象，但是能获取到的值却不一样。

挺神奇的是不是，其实就是其内部获取到的Map不同，Map和Thread绑定，所以虽然访问的是同一个`ThreadLocal`对象，但是访问的Map却不是同一个，所以取得值也不一样。

这样做有什么好处呢？为什么不直接用Map存储线程和对象呢？

打个比方：

- `ThreadLocal`就是老师。
- `Thread`就是同学。
- `Looper`（需要的值）就是铅笔。

现在老师买了一批铅笔，然后想把这些铅笔发给同学们，怎么发呢？两种办法：

- 1. 老师把每个铅笔上写好每个同学的名字，放到一个大盒子里面去（map），用的时候就让同学们自己来找。

这种做法就是Map里面存储的是**同学和铅笔**，然后用的时候通过同学来从这个Map里找铅笔。

这种做法就有点像使用一个Map，存储所有的线程和对象，不好的地方就在于会很混乱，每个线程之间有了联系，也容易造成内存泄漏。

- 2. 老师把每个铅笔直接发给每个同学，放到同学的口袋里（map），用的时候每个同学从口袋里面拿出铅笔就可以了。

这种做法就是Map里面存储的是**老师和铅笔**，然后用的时候老师说一声，同学只需要从口袋里拿出来就行了。

很明显这种做法更科学，这也就是 `ThreadLocal` 的做法，因为铅笔本身就是同学自己在用，所以一开始就把铅笔交给同学自己保管是最好的，每个同学之间进行隔离。

#### 还有哪些地方运用到了ThreadLocal机制？

比如：Choreographer。

```java
  public final class Choreographer {
        // Thread local storage for the choreographer.   
        private static final ThreadLocal<Choreographer> sThreadInstance = new ThreadLocal<Choreographer>() {
            @Override
            protected Choreographer initialValue() {
                Looper looper = Looper.myLooper();
                if (looper == null) {
                    throw new IllegalStateException("The current thread must have a looper!");
                }
                Choreographer choreographer = new Choreographer(looper, VSYNC_SOURCE_APP);
                if (looper == Looper.getMainLooper()) {
                    mMainInstance = choreographer;
                }
                return choreographer;
            }
        };
        private static volatile Choreographer mMainInstance;
    }
```

`Choreographer`主要是主线程用的，用于配合 `VSYNC0` 中断信号。
所以这里使用ThreadLocal更多的意义在于完成线程单例的功能。

#### 可以多次创建Looper吗？

Looper的创建是通过`Looper.prepare`方法实现的，而在prepare方法中就判断了，当前线程是否存在Looper对象，如果有，就会直接抛出异常：

```java
    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
```

**所以同一个线程，只能创建一个`Looper`，多次创建会报错。**

#### Looper中的quitAllowed字段是啥？有什么用？

按照字面意思就是是否允许退出，我们看看他都在哪些地方用到了：

```java

    void quit(boolean safe) {
        if (!mQuitAllowed) {
            throw new IllegalStateException("Main thread not allowed to quit.");
        }
        synchronized (this) {
            if (mQuitting) {
                return;
            }
            mQuitting = true;
            if (safe) {
                removeAllFutureMessagesLocked();
            } else {
                removeAllMessagesLocked();
            }
        }
    }
```

哦，就是这个`quit`方法用到了，如果这个字段为`false`，代表不允许退出，就会报错。

但是这个`quit`方法又是干嘛的呢？从来没用过呢。还有这个`safe`又是啥呢？

其实看名字就差不多能了解了，`quit`方法就是退出消息队列，终止消息循环。

- 首先设置了`mQuitting` 字段为true。
- 然后判断是否安全退出，如果安全退出，就执行`removeAllFutureMessagesLocked`方法，它内部的逻辑是清空所有的延迟消息，之前没处理的非延迟消息还是需要取处理，然后设置非延迟消息的下一个节点为空（p.next=null）。
- 如果不是安全退出，就执行`removeAllMessagesLocked`方法，直接清空所有的消息，然后设置消息队列指向空（mMessages = null）

然后看看当调用quit方法之后，消息的发送和处理：

```java
    //消息发送  
    boolean enqueueMessage(Message msg, long when) {
        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }
        }
    }
```

当调用了`quit`方法之后，`mQuitting`为`true`，消息就发不出去了，会报错。

再看看消息的处理，loop和next方法：

```java
    public static void loop() {
        for (; ; ) {
            Message msg = queue.next();
            if (msg == null) {
                // No message indicates that the message queue is quitting.      
                return;
            }
        }
    }
```

很明显，当`mQuitting`为`true`的时候，`next`方法返回`null`，那么`loop`方法中就会退出死循环。

那么这个`quit`方法一般是什么时候使用呢？

- 主线程中，一般情况下肯定不能退出，因为退出后主线程就停止了。所以是当APP需要退出的时候，就会调用quit方法，涉及到的消息是EXIT_APPLICATION，大家可以搜索下。
- 子线程中，如果消息都处理完了，就需要调用quit方法停止消息循环。

#### Looper.loop方法是死循环，为什么不会卡死（ANR）？

为了app不挂掉，就要保证主线程一直运行存在，使用死循环代码阻塞在`msgQueue.next()`中的`nativePollOnce()`方法里 ，主线程就会挂起休眠释放cpu，线程就不会退出。Looper死循环之前，在`ActivityThread.main()`中就会创建一个 `Binder 线程（ApplicationThread）`，接收系统服务AMS发送来的事件。当系统有消息产生（其实系统每 16ms 会发送一个刷新 UI 消息唤醒）会通过`epoll机制` 向`pipe管道`写端写入数据 就会发送消息给 looper 接收到消息后处理事件，保证主线程的一直存活。只有在主线程中处理超时才会让app崩溃 也就是ANR。

- 1. 主线程本身就是需要一直运行的，因为要处理各个View，界面变化。所以需要这个死循环来保证主线程一直执行下去，不会被退出。
- 2. 真正会卡死的操作是在某个消息处理的时候操作时间过长，导致掉帧、ANR，而不是loop方法本身。
- 3. 在主线程以外，会有其他的线程来处理接受其他进程的事件，**比如Binder线程（ApplicationThread）**，会接受AMS发送来的事件
- 4. 在收到跨进程消息后，会交给主线程的 `Hanlder` 再进行消息分发。所以Activity的生命周期都是依靠主线程的`Looper.loop`，当收到不同Message时则采用相应措施，比如收到`msg=H.LAUNCH_ACTIVITY`，则调用`ActivityThread.handleLaunchActivity()`方法，最终执行到onCreate方法。
- 5. 当没有消息的时候，会阻塞在loop的`queue.next()`中的`nativePollOnce()`方法里，此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生。所以死循环也不会特别消耗CPU资源。

#### Message是怎么找到它所属的Handler然后进行分发的？

在loop方法中，找到要处理的`Message`，然后调用了这么一句代码处理消息：

所以是将消息交给了`msg.target` 来处理，那么这个target是啥呢？

找找它的来头：

```java
//Handler    
private boolean enqueueMessage(MessageQueue queue,Message msg,long uptimeMillis) {    
  msg.target = this;             
  return queue.enqueueMessage(msg, uptimeMillis); 
}
```

在使用Hanlder发送消息的时候，会设置`msg.target = this`，所以target就是当初把消息加到消息队列的那个Handler。

#### Handler 的 post(Runnable) 与 sendMessage 有什么区别

Hanlder中主要的发送消息可以分为两种：

- post(Runnable)
- sendMessage

```java
    public final boolean post(@NonNull Runnable r) {
        return sendMessageDelayed(getPostMessage(r), 0);
    }

    private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }
```

通过`post`的源码可知，其实`post`和`sendMessage`的区别就在于：

`post`方法给`Message`设置了一个`callback`。

那么这个callback有什么用呢？我们再转到消息处理的方法`dispatchMessage`中看看：

```java
    public void dispatchMessage(@NonNull Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }

    private static void handleCallback(Message message) {
        message.callback.run();
    }
```

这段代码可以分为三部分看：

- 1. 如果`msg.callback`不为空，也就是通过`post`方法发送消息的时候，会把消息交给这个`msg.callback`进行处理，然后就没有后续了。
- 2. 如果`msg.callback`为空，也就是通过`sendMessage`发送消息的时候，会判断`Handler`当前的`mCallback`是否为空，如果不为空就交给`Handler.Callback.handleMessage`处理。
- 3. 如果`mCallback.handleMessage`返回`true`，则无后续了。
- 4. 如果`mCallback.handleMessage`返回`false`，则调用`handler`类重写的`handleMessage`方法。

**所以`post(Runnable)` 与 `sendMessage`的区别就在于后续消息的处理方式，是交给`msg.callback`还是`Handler.Callback`或者`Handler.handleMessage`。**

#### Handler.Callback.handleMessage 和 Handler.handleMessage 有什么不一样？为什么这么设计？

接着上面的代码说，这两个处理方法的区别在于`Handler.Callback.handleMessage`方法是否返回true：

- 如果为true，则不再执行Handler.handleMessage
- 如果为false，则两个方法都要执行。

那么什么时候有`Callback`，什么时候没有呢？这涉及到两种Hanlder的 创建方式：

```kotlin
val handler1= object : Handler(){  
  override fun handleMessage(msg: Message) {
    super.handleMessage(msg)     
  }   
}  

val handler2 = Handler(object : Handler.Callback {   
  override fun handleMessage(msg: Message): Boolean {    
    return true   
  } 
})
```

常用的方法就是第1种，派生一个Handler的子类并重写handleMessage方法。而第2种就是系统给我们提供了一种不需要派生子类的使用方法，只需要传入一个Callback即可。

#### Handler、Looper、MessageQueue、线程是一一对应关系吗？

- 一个线程只会有一个`Looper`对象，所以线程和`Looper`是一一对应的。
- `MessageQueue`对象是在`new Looper`的时候创建的，所以`Looper`和`MessageQueue`是一一对应的。
- `Handler`的作用只是将消息加到`MessageQueue`中，并后续取出消息后，根据消息的`target`字段分发给当初的那个`handler`，所以`Handler`对于`Looper`是可以多对一的，也就是多个`Hanlder`对象都可以用同一个线程、同一个`Looper`、同一个`MessageQueue`。

总结：`Looper`、`MessageQueue`、线程是一一对应关系，而他们与`Handler`是可以一对多的。

#### ActivityThread中做了哪些关于Handler的工作？（为什么主线程不需要单独创建Looper）

主要做了两件事：

- 1. 在main方法中，创建了主线程的Looper和MessageQueue，并且调用loop方法开启了主线程的消息循环。

```java
    public static void main(String[] args) {
        Looper.prepareMainLooper();
        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }
        Looper.loop();
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

- 2. 创建了一个Handler来进行四大组件的启动停止等事件处理

```java
    final H mH = new H();

    class H extends Handler {
        public static final int BIND_APPLICATION = 110;
        public static final int EXIT_APPLICATION = 111;
        public static final int RECEIVER = 113;
        public static final int CREATE_SERVICE = 114;
        public static final int STOP_SERVICE = 116;
        public static final int BIND_SERVICE = 121;
    }
```

#### IdleHandler是啥？有什么使用场景？

之前说过，当`MessageQueue`没有消息的时候，就会阻塞在next方法中，其实在阻塞之前，`MessageQueue`还会做一件事，就是检查是否存在`IdleHandler`，如果有，就会去执行它的`queueIdle`方法。

```java
   
    private IdleHandler[] mPendingIdleHandlers;

    Message next() {
        int pendingIdleHandlerCount = -1;
        for (; ; ) {
            synchronized (this) {
                //当消息执行完毕，就设置pendingIdleHandlerCount     
                if (pendingIdleHandlerCount < 0 && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                //初始化mPendingIdleHandlers  
                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                //mIdleHandlers转为数组    
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }
            // 遍历数组，处理每个IdleHandler  
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null;
                // release the reference to the handler   
                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }
                //如果queueIdle方法返回false，则处理完就删除这个IdleHandler     
                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }
            // Reset the idle handler count to 0 so we do not run them again.       
            pendingIdleHandlerCount = 0;
        }
    }

```

当没有消息处理的时候，就会去处理这个`mIdleHandlers`集合里面的每个`IdleHandler`对象，并调用其`queueIdle`方法。最后根据`queueIdle`返回值判断是否用完删除当前的`IdleHandler`。
然后看看`IdleHandler`是怎么加进去的：

```java
    Looper.myQueue().addIdleHandler(new IdleHandler() {
        @Override public boolean queueIdle ()
        {
            //做事情
            return false;
        }
    });

    public void addIdleHandler(@NonNull IdleHandler handler) {
        if (handler == null) {
            throw new NullPointerException("Can't add a null IdleHandler");
        }
        synchronized (this) {
            mIdleHandlers.add(handler);
        }
    }
```

ok，综上所述，`IdleHandler`就是当消息队列里面没有当前要处理的消息了，需要堵塞之前，可以做一些空闲任务的处理。

常见的使用场景有：**启动优化**。

我们一般会把一些事件（比如界面view的绘制、赋值）放到`onCreate`方法或者`onResume`方法中。但是这两个方法其实都是在界面绘制之前调用的，也就是说一定程度上这两个方法的耗时会影响到启动时间。

所以我们可以把一些操作放到`IdleHandler`中，也就是界面绘制完成之后才去调用，这样就能减少启动时间了。

但是，这里需要注意下可能会有坑。

如果使用不当，`IdleHandler`会一直不执行，比如在`View的onDraw`方法里面无限制的直接或者间接调用`View的invalidate`方法。

其原因就在于onDraw方法中执行`invalidate`，会添加一个同步屏障消息，在等到异步消息之前，会阻塞在next方法，而等到`FrameDisplayEventReceiver`异步任务之后又会执行`onDraw`方法，从而无限循环。

#### HandlerThread是啥？有什么使用场景？

直接看源码：

```java

    public class HandlerThread extends Thread {
        @Override
        public void run() {
            Looper.prepare();
            synchronized (this) {
                mLooper = Looper.myLooper();
                notifyAll();
            }
            Process.setThreadPriority(mPriority);
            onLooperPrepared();
            Looper.loop();
        }

    }
```

哦，原来如此。`HandlerThread`就是一个封装了Looper的Thread类。

就是为了让我们在子线程里面更方便的使用Handler。

这里的加锁就是为了保证线程安全，获取当前线程的Looper对象，获取成功之后再通过`notifyAll`方法唤醒其他线程，那哪里调用了`wait`方法呢？

```java
     public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }
        // If the thread has been started, wait until the looper has been created.      
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }
```

就是`getLooper`方法，所以wait的意思就是等待Looper创建好，那边创建好之后再通知这边正确返回Looper。

#### IntentService是啥？有什么使用场景？

老规矩，直接看源码：

```java

    public abstract class IntentService extends Service {
        private final class ServiceHandler extends Handler {
            public ServiceHandler(Looper looper) {
                super(looper);
            }

            @Override
            public void handleMessage(Message msg) {
                onHandleIntent((Intent) msg.obj);
                stopSelf(msg.arg1);
            }
        }

        @Override
        public void onCreate() {
            super.onCreate();
            HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
            thread.start();
            mServiceLooper = thread.getLooper();
            mServiceHandler = new ServiceHandler(mServiceLooper);
        }

        @Override
        public void onStart(@Nullable Intent intent, int startId) {
            Message msg = mServiceHandler.obtainMessage();
            msg.arg1 = startId;
            msg.obj = intent;
            mServiceHandler.sendMessage(msg);
        }
    }
```

理一下这个源码：

- 首先，这是一个`Service`
- 并且内部维护了一个`HandlerThread`，也就是有完整的Looper在运行。
- 还维护了一个子线程的`ServiceHandler`。
- 启动`Service`后，会通过`Handler`执行`onHandleIntent`方法。
- 完成任务后，会自动执行`stopSelf`停止当前`Service`。

所以，这就是一个可以在子线程进行耗时任务，并且在任务执行后自动停止的Service。

#### BlockCanary使用过吗？说说原理

`BlockCanary`是一个用来检测应用卡顿耗时的三方库。

上文说过，View的绘制也是通过Handler来执行的，所以如果能知道每次Handler处理消息的时间，就能知道每次绘制的耗时了？那Handler消息的处理时间怎么获取呢？
再去loop方法中找找细节：

```java

    public static void loop() {
        for (; ; ) {
            // This must be in a local variable, in case a UI event sets the logger    
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " + msg.callback + ": " + msg.what);
            }
            msg.target.dispatchMessage(msg);
            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }
        }
    }
```

可以发现，`loop`方法内有一个`Printer`类，在`dispatchMessage`处理消息的前后分别打印了两次日志。

那我们把这个日志类`Printer`替换成我们自己的`Printer`，然后统计两次打印日志的时间不就相当于处理消息的时间了？

```java
Looper.getMainLooper().setMessageLogging(mainLooperPrinter);    
public void setMessageLogging(@Nullable Printer printer) {    
  mLogging = printer;  
}
```

这就是BlockCanary的原理。
具体介绍可以看看作者的说明：<http://blog.zhaiyifan.cn/2016/01/16/BlockCanaryTransparentPerformanceMonitor/>

#### 说说Hanlder内存泄露问题

这也是常常被问的一个问题，`Handler`内存泄露的原因是什么？

**"内部类持有了外部类的引用，也就是Hanlder持有了Activity的引用，从而导致无法被回收呗。"**

其实这样回答是错误的，或者说没回答到点子上。

我们必须找到那个最终的引用者，不会被回收的引用者，其实就是主线程，这条完整引用链应该是这样：

主线程 —> threadlocal —> Looper —> MessageQueue —> Message —> Handler —> Activity

具体分析可以看看我之前写的这篇文章：<https://juejin.cn/post/6909362503898595342>

#### 利用Handler机制设计一个不崩溃的App？

主线程崩溃，其实都是发生在消息的处理内，包括生命周期、界面绘制。

所以如果我们能控制这个过程，并且在发生崩溃后重新开启消息循环，那么主线程就能继续运行。

```java
Handler(Looper.getMainLooper()).post {     
  while (true) {       
    //主线程异常拦截       
    try {                
      Looper.loop()     
    } catch (e: Throwable) {    
    }     
  } 
}
```

还有一些特殊情况处理，比如onCreate内发生崩溃，具体可以看看文章
《能否让APP永不崩溃》<https://juejin.cn/post/6904283635856179214>

## Android 性能优化

> [面试官: 说一下你做过哪些性能优化?](https://juejin.cn/post/6844904105438134286)

主要介绍了一下10点性能优化内容，可供参考

- [1、你对 APP 的启动有过研究吗? 有做过相关的启动优化吗?](https://juejin.cn/post/6844904105438134286#heading-1)
- [2、有做过相关的内存优化吗?](https://juejin.cn/post/6844904105438134286#heading-2)
- [3、你在项目中有没有遇见卡顿问题？是怎么排查卡顿？又是怎么优化的?](https://juejin.cn/post/6844904105438134286#heading-3)
- [4、怎么保证 APP 的稳定运行?](https://juejin.cn/post/6844904105438134286#heading-4)
- [5、说说你在项目中网络优化?](https://juejin.cn/post/6844904105438134286#heading-5)
- [6、你在项目中有用过哪些存储方式? 对它们的性能有过优化吗？](https://juejin.cn/post/6844904105438134286#heading-6)
- [7、你在项目中有做过自定义 View 吗？有对它做过什么优化？](https://juejin.cn/post/6844904105438134286#heading-7)
- [8、你们项目的耗电量怎么样? 有做过优化吗?](https://juejin.cn/post/6844904105438134286#heading-8)
- [9、有做过日志优化吗?](https://juejin.cn/post/6844904105438134286#heading-9)
- [10、你们 APK 有多大？有做过 APK 体积相关的优化吗?](https://juejin.cn/post/6844904105438134286#heading-10)


### 布局优化

布局优化的思想很简单，，就是尽量减少布局文件的层级，这个道理是很浅显的，布局中的层级少了，这就意味着Android绘制时的工作量少了，那么程序的性能自然就高了。

如何进行控制布局优化呢？首先删除布局中吴用的空间和层级，其次有选择地使用性能较低的ViewGroup，比如RelativeLayout，如果布局中既可以使用LinearLayout也可以使用RelativeLayout，那么就采用LinearLayout，这是因为RelativeLayout功能比较复杂，他的布局过程需要花费更多的CPU时间。FrameLayout和LinearLayout一样都是一种简单高效的ViewGroup，因此可以考虑使用他们，但是很多时候单纯的通过一个LinearLayout或者FrameLayout无法实现产品效果，需要通过嵌套的方式来完成。这种情况下还是建议采用RelativeLayout，因为ViewGroup的嵌套就相当于增家e了布局的层级，同样会降低程序的性能。

布局优化的另外一种手段是采用<include>标签、<merge>标签、和ViewStub。<include>标签主要用于布局重用，<merge>标签一般和<include>标签配合使用，它可以降低减少布局的层级，而ViewStub中的布局加载到内存，这提高了程序的初始化效率。

### 绘制优化

绘制优化是指View的onDraw方法要避免执行大量的操作，这主要体现在两个方面。

首先，onDraw中不要创建新的局部对象，这是因为onDraw方法可能会被频繁调用，这样就会一瞬间产生大量的临时对象，这不仅占用了过多的内存而且还会导致系统更加频繁GC，降低了程序的执行效率。

另一方面，onDraw方法中不要做耗时的任务，也不能执行成千上万的循环操作，尽管每次循环都很轻量级，但是大量的循环仍然十分抢占CPU的时间片，这会造成View的绘制过程不流畅。按照Google官方给出的性能优化典范中的标准，View的绘制帧率保证在60fps是最佳的，这就要求每帧的绘制时间不超过16ms（16ms = 1000 / 60）,虽然程序很难保证16ms这个时间，但是尽量降低onDraw方法的复杂度总是切实有效的。

### 内存泄露优化

内存泄露在开发过程中是一个需要重视的问题，但是由于内存泄露问题对开发人员的经验和开发意识有比较高的要求，因此这也是开发人员最容易犯的错误之一。内存泄露的优化分为两个方面，一方面是在开发过程中避免写出有内存泄露的代码，另一方面是通过一些分析工具比如MAT来找出潜在的内存泄露继而解决。

#### 静态变量导致内存泄露

例如静态变量持有当前Activity的对象，从而导致Activity无法释放。

#### 单例模式导致内存泄露

单体模式持有当前的Activity的内部对象，从而导致内存泄露。因为单例模式 的特点是其生命周期和Application保持一致，因此Activity对象无法被及时释放。

#### 属性动画导致内存泄露

Android 3.0 开始，Google提供了属性动画，属性动画中有一类五险循环的动画，如果在Activity中播放此类动画且没有在onDestroy中去停止动画，那么动画会一直播放下去，尽管已经无法在界面上看到动画效果了，并且这个时候又持有了Activity和View的对象，最终Activity无法释放。

#### 响应速度优化和ANR日志分析

相应速度优化的核心思想是避免在主线程中作耗时操作，但是有时候的确有很多的耗时操作，怎么办呢？可以将这些耗时操作放在线程中去执行，即采用异步的方式执行耗时操作。响应速度过慢更多地体现在Activity的启动速度上面，如果在主线程中作太多事情，会导致Activity启动时出现黑屏现象，甚至出现ANR
。Android规定，Activity如果5s之内无法响应屏幕触摸事件或者键盘输入事件就会出现ANR，而BroadcastReceiver如果10s内还未执行也会出现ANR。

如何定位ANR呢？其实当一个进程发生了ANR了以后，系统会在`/data/anr`目录下创建一个文件 `traces.txt`,通过分析这个文件就能定位出ANR的原因

#### ListView和Bitmap优化

##### ListView优化

ListView的优化主页分为三个方面：首先要采用ViewHolder并且避免在getView中执行耗时操作；其次要根据列表滑动状态来控制任务的执行频率，比如当列表快速滑动时显然是不太适合开启大量的异步任务的；最后可以尝试开启硬件加速来使ListView的滑动更加流畅。

注意：ListView的优化策略完全适用GridView。

##### Bitmap优化

Bitmap常用的优化方式是两种：

###### 修改Bitmap.Config

这一点刚才也说过，不同的Conifg代表每个像素不同的占用空间，所以如果我们把默认的ARGB_8888改成RGB_565，那么每个像素占用空间就会由4字节变成2字节了，那么图片所占内存就会减半了。

可能一定程度上会降低图片质量，但是我实际测试看不出什么变化。

###### 修改inSampleSize

inSampleSize，采样率，这个参数是用于图片尺寸压缩的，他会在宽高的维度上每隔inSampleSize个像素进行一次采集，从而达到缩放图片的效果。这种方法只会改变图片大小，不会影响图片质量。

```kotlin
val options=BitmapFactory.Options()   
options.inSampleSize=2
val bitmap = BitmapFactory.decodeResource(resources, R.drawable.test2,options)    img.setImageBitmap(bitmap)
```

实际项目中，我们可以设置一个与目标图像大小相近的inSampleSize，来减少实际使用的内存：

```kotlin
fun getImage(): Bitmap {   
  var options = BitmapFactory.Options()    
  options.inJustDecodeBounds = true     
  BitmapFactory.decodeResource(resources, R.drawable.test2, options)     
  // 计算最佳采样率 
  options.inSampleSize = getImageSampleSize(options.outWidth, options.outHeight)      
  options.inJustDecodeBounds = false     
  return BitmapFactory.decodeResource(resources, R.drawable.test2, options) 
}
```

###### inJustDecodeBounds是什么？

上面的例子大家应该发现了，其中有个inJustDecodeBounds，又设置为true，又设置成false的，总感觉多此一举，那么他到底是干嘛呢？

因为我们要获取图片本身的大小，如果直接decodeResource加载一遍的话，那么就会增加内存了，所以官方提供了这样一个参数inJustDecodeBounds。如果inJustDecodeBounds为ture，那么decode的bitmap为null，也就是不返回实际的bitmap，只把图片的大小信息放到了options的值中。

所以这个参数就是用来获取图片的大小信息的同时不占用内存。

###### Bitmap是什么，怎么存储图片

Bitmap，位图，本质上是一张图片的内容在内存中的表达形式。它将图片的内容看做是由存储数据的有限个像素点组成；每个像素点存储该像素点位置的ARGB值，每个像素点的ARGB值确定下来，这张图片的内容就相应地确定下来。其中，A代表透明度，RGB代表红绿蓝三种颜色通道值。

###### Bitmap内存如何计算

Bitmap一直都是Android中的内存大户，计算大小的方式有三种：

- getRowBytes() 这个在API Level 1添加的，返回的是bitmap一行所占的大小，需要乘以bitmap的高，才能得出btimap的大小
- getByteCount() 这个是在API Level 12添加的，其实是对getRowBytes()乘以高的封装
- getAllocationByteCount() 这个是在API Level 19添加的

这里我将一张图片放到项目的drawable-xxhdpi文件夹中，然后通过方法获取图片所占的内存大小：

```kotlin
var bitmap = BitmapFactory.decodeResource(resources, R.drawable.test)    img.setImageBitmap(bitmap) 
Log.e(TAG,"dpi = ${resources.displayMetrics.densityDpi}")
Log.e(TAG,"size = ${bitmap.allocationByteCount}")
```

打印出来的结果是

```
size=1960000
```

具体是怎么计算的呢？

**图片内存=宽 * 高 * 每个像素所占字节**

这个像素所占字节又和Bitmap.Config有关，Bitmap.Config是个枚举类，用于描述每个像素点的信息，比如：

- ARGB_8888。常用类型，总共32位，4个字节，分别表示透明度和RGB通道。
- RGB_565。16位，2个字节，只能描述RGB通道。

所以我们这里的图片内存计算就得出：

宽700 *高700* 每个像素4字节=1960000

###### Bitmap内存 和drawable目录的关系

首先放一张drawable目录对应的屏幕密度对照表,来自郭霖的博客：

![屏幕密度对照表](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-234916.png)

刚才的案例，我们是把图片放到drawable-xxhdpi文件夹,而drawable-xxhdpi文件夹对应的dpi就是我们测试手机的dpi—480。所以图片的内存就是我们所计算的宽 *高* 每个像素所占字节。

如果我们把图片放到其他的文件夹，比如drawable-hdpi文件夹（对应的dpi是240），会发生什么呢？

再次打印结果：

```
size = 7840000
```

这是因为一张图片的实际占用内存大小计算公式是：

**占用内存 = 宽 * 缩放比例 * 高 * 缩放比例 * 每个像素所占字节**

这个缩放比例就跟屏幕密度DPI有关了：

**缩放比例 = 设备dpi/图片所在目录的dpi**

所以我们这张图片的实际占用内存位：

宽700 *（480/240）* 高700 *（480/240）* 每个像素4字节 = 7840000

###### Bitmap内存复用怎么实现？

如果有个需求，是在同一个imageview中可以加载不同的图片，那我们需要每次都去新建一个Bitmap对象，占用新的内存空间吗？如果我们这样写的话：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {     
  super.onCreate(savedInstanceState)     
  setContentView(R.layout.actvitiy_bitmap)    
  btn1.setOnClickListener {         
    img.setImageBitmap(getBitmap(R.drawable.test)) 
  }    
  btn2.setOnClickListener { 
    img.setImageBitmap(getBitmap(R.drawable.test2))      
  } 
}

fun getBitmap(resId: Int): Bitmap {  
  var options = BitmapFactory.Options()   
  return BitmapFactory.decodeResource(resources, resId, options)
} 
```

这样就会Bitmap就会频繁去申请内存，释放内存，从而导致大量GC，内存抖动。

为了防止这种情况呢，我们就可以用到inBitmap参数，用于Bitmap的内存复用。这样同一块内存空间就可以被多个Bitmap对象复用，从而减少了频繁的GC。

```kotlin
val options by lazy {  
  BitmapFactory.Options()
}
val reuseBitmap by lazy { 
  options.inMutable = true  
  BitmapFactory.decodeResource(resources, R.drawable.test, options)
}
fun getBitmap(resId: Int): Bitmap { 
  options.inMutable = true   
  options.inBitmap = reuseBitmap  
  return BitmapFactory.decodeResource(resources, resId, options)
}
```

这里有几个要注意的点

- inBitmap要和inMutable属性配套使用，否则将无法复用。
- 在Android 4.4之前，只能重用相同大小的 Bitmap 内存区域；4.4之后只要复用内存空间的Bitmap对象大小比inBitmap指向的内存空间要小即可。

所以一般在复用之前，还要判断下，新的Bitmap内存是不是小于可以复用的Bitmap内存，然后才能进行复用。

###### 高清大图加载该怎么处理？

如果是高清大图，那就说明不允许进行图片压缩，比如微博长图，清明上河图。

所以我们就要对图片进行局部显示，这就用到BitmapRegionDecoder属性，主要用于显示图片的某一块矩形区域。

比如我要显示左上角的100 * 100区域：

```kotlin
fun setImagePart() {  
  val inputStream: InputStream = assets.open("test.jpg")  
  val bitmapRegionDecoder: BitmapRegionDecoder = BitmapRegionDecoder.newInstance(inputStream, false) 
  val options = BitmapFactory.Options()   
  val bitmap = bitmapRegionDecoder.decodeRegion(Rect(0, 0, 100, 100), options)  image.setImageBitmap(bitmap)
}
```

实际项目使用中，我们可以根据手势滑动，然后不断更新我们的Rect参数来实现具体的功能即可。

具体实现源码可以参考鸿洋的博客：
<https://blog.csdn.net/lmj623565791/article/details/49300989>

###### 如何跨进程传递大图？

- Bundle直接传递。bundle最常用于Activity间传递，也属于跨进程的一种方式，但是传递的大小有限制，一般为1M。

```kotlin
//intent.put的putExtra方法实质也是通过
bundleintent.putExtra("image",bitmap);
bundle.putParcelable("image",bitmap)
```

Bitmap之所以可以直接传递，是因为其实现了Parcelable接口进行了序列化。而Parcelable的传递原理是利用了Binder机制，将Parcel序列化的数据写入到一个共享内存（缓冲区）中，读取的时候也会从这个缓冲区中去读取字节流，然后再反序列化成对象使用。这个共享内存也就是缓存区有一个大小限制—1M，而且是公用的。所以传图片的话很容易就容易超过这个大小然后报错TransactionTooLargeException。

所以这个方案不可靠。

- 文件传输

将图片保存到文件，然后只传输文件路径，这样肯定是可以的，但是不高效。

- putBinder

这个就是考点了。通过传递binder的方式传递bitmap。

```kotlin
//传递binder
val bundle = Bundle()
bundle.putBinder("bitmap", BitmapBinder(mBitmap))
//接收binder中的bitmap
val imageBinder: BitmapBinder = bundle.getBinder("bitmap") as BitmapBinderval 
bitmap: Bitmap? = imageBinder.getBitmap()
//Binder子类
class BitmapBinder :Binder(){ 
  private var bitmap: Bitmap? = null   
 
  fun ImageBinder(bitmap: Bitmap?) {   
    this.bitmap = bitmap   
  }   

  fun getBitmap(): Bitmap? {
    return bitmap
  }
}
```

- 为什么用putBinder就没有大小限制了呢？
  - 因为putBinder中传递的其实是一个文件描述符fd，文件本身被放到一个共享内存中，然后获取到这个fd之后，只需要从共享内存中取出Bitmap数据即可，这样传输就很高效了。
  - 而用Intent/bundle直接传输的时候，会禁用文件描述符fd，只能在parcel的缓存区中分配空间来保存数据，所以无法突破1M的大小限制。

### 线程优化

线程优化的思想是采用线程池，避免程序中存在大量的Thread。线程池可以重用内部的线程，从而避免线程的创建和销毁所带来的性能开销，同时线程池还能有效地控制线程池的最大并发数，避免大量的线程因互相抢占系统资源从而导致阻塞现象发生。因此实际开发中，我们要尽量采用线程池，而不是每次对创建一个Thread对象。

### 启动优化

#### 冷启动、温启动、热启动

> 冷启动。冷启动指的是该应用程序在此之前没有被创建，发生在应用程序首次启动或者自上次被终止后的再次启动。简单的说就是app进程还没有，需要创建app的进程启动app。

比如开机后，点击屏幕的app图标启动应用。

冷启动的过程主要分为两步：

1）系统任务。加载并启动应用程序；显示应用程序的空白启动窗口；创建APP进程

2）APP进程任务。启动主线程；创建Activity；加载布局；屏幕布局；绘制屏幕

其实这不就是APP的启动流程嘛？所以冷启动是会完整走完一个启动流程的，从系统到进程。

> 温启动。温启动指的是App进程存在，但Activity可能因为内存不足被回收，这时候启动App不需要重新创建进程，只需要执行APP进程中的一些任务，比如创建Activity。

比如返回主页后，又继续使用其他的APP，时间久了或者打开的应用多了，之前应用的Activity有可能被回收了，但是进程还在。
所以温启动相当于执行了冷启动的第二过程，也就是APP进程任务，需要重新启动线程，Activity等。

> 热启动。热启动就是App进程存在，并且Activity对象仍然存在内存中没有被回收。

比如app被切到后台，再次启动app的过程。

所以热启动的开销最少，这个过程只会把Activity从后台展示到前台，无需初始化，布局绘制等工作。

#### 启动优化我们可以介入的优化点

所以三种启动方式中，冷启动经历的时间最长，也是走完了最完整的启动流程，所以我们再次分析下冷启动的启动流程，看看有哪些可以优化的点：

- Launcher startActivity
- AMS startActivity
- Zygote fork 进程
- ActivityThread main()
- ActivityThread attach
- handleBindApplication
- attachBaseContext
- Application attach
- installContentProviders
- Application onCreate
- Looper.loop
- Activity onCreate，onResume

纵观整个流程，其实我们能动的地方不多，无非就是Application的attach，onCreate方法，Activity的onCreate，onResume方法，这些方法也就是我们的优化点。

#### 启动优化方案总结

最后再和大家回顾下今天说到的启动优化方案：

- 消除启动时的白屏/黑屏。windowBackground。
- 第三方库懒加载/异步加载。线程池，启动器。
- 预创建Activity。对象预创建。
- 预加载数据。
- Multidex预加载优化。5.0以下多dex情况。
- Webview启动优化。预创建，缓存池，静态资源。
- 避免布局嵌套。多层嵌套。

为了方便记忆，我再整理成以下三类，分别是Application、Activity、UI：

- Application 三方库，Multidex。
- Activity 预创建类，预加载数据。
- UI方面 windowBackground，布局嵌套，webview。

具体说明可以看往期文章 Android启动优化全解析
<https://mp.weixin.qq.com/s?__biz=MzU0MTYwMTIzMw==&mid=2247484835&idx=1&sn=f62c8fb5c9a461987f5bb5446bc5a19b&scene=21#wechat_redirect>

##### 障眼法之闪屏页

为了消除启动时的白屏/黑屏，可以通过设置android:windowBackground，让人感觉一点击icon就启动完毕了的感觉。

```xml
<activity android:name=".ui.activity.启动activity"    
          android:theme="@style/MyAppTheme"   
          android:screenOrientation="portrait">    
  <intent-filter>           
    <action android:name="android.intent.action.MAIN" />      
    <category android:name="android.intent.category.LAUNCHER" />      
  </intent-filter>
</activity>

<style name="MyAppTheme" parent="Theme.AppCompat.NoActionBar">  
  <item name="android:windowBackground">@drawable/logo</item>
</style>
```

##### 预创建Activity

对象第一次创建的时候，java虚拟机首先检查类对应的Class 对象是否已经加载。如果没有加载，jvm会根据类名查找.class文件，将其Class对象载入。同一个类第二次new的时候就不需要加载类对象，而是直接实例化，创建时间就缩短了。

##### 第三方库懒加载

很多第三方开源库都说在Application中进行初始化，所以可以把一些不是需要启动就初始化的三方库的初始化放到后面，按需初始化，这样就能让Application变得更轻。

##### WebView启动优化

webview第一次启动会非常耗时,看下面的关于WebView优化

##### 线程优化

线程是程序运行的基本单位，线程的频繁创建是耗性能的，所以大家应该都会用线程池。单个cpu情况下，即使是开多个线程，同时也只有一个线程可以工作，所以线程池的大小要根据cpu个数来确定。

#### 分析启动耗时的方法

Systrace + 函数插桩

也就是通过在方法的入口和出口加入统计代码，从而统计方法耗时

```java
class Trace {
    public static void i(String tag) {
        android.os.Trace.beginSection(tag);
    }

    public static void o() {
        android.os.Trace.endSection();
    }
}

    void test() {
        Trace.i("test");
        System.out.println("doSomething");
        Trace.o();
    }
```

BlockCanary BlockCanary 可以监听主线程耗时的方法，就是在主线程消息循环打出日志的地入手, 当一个消息操作时间超过阀值后, 记录系统各种资源的状态, 并展示出来。所以我们将阈值设置低一点，这样的话如果一个方法执行时间超过200毫秒，获取堆栈信息。

而记录时间的方法我们之前也说过，就是通过looper()方法中循环去从MessageQueue中去取msg的时候，在dispatchMessage方法前后会有logging日志打印，所以只需要自定义一个Printer，重写println(String x)方法即可实现耗时统计了。

#### 打包优化

Analyze APK 后可以发现代码 和 资源其实是 app包的主要内存。

- res 文件夹下 分辨率下的图片 国内基本提供 xxhdpi 或者 xhdpi 即可，android 会分析手机分辨率到对应分辨率文件夹下加载资源。
- res中的 png 图片 都可以转为 webg 或者 svg格式的 ，如果不能转 则可以通过 png压缩在减少内存。
- 通过在 build.gradle 中配置 minifyEnabled true（混淆）shrinkResources true 。（移除无用资源）
- Assests 中的 mp4 /3 可以在需要使用的时候从服务器上下载下来，字体文件 使用字体提取工具FontZip 删除不用的文字格式，毕竟几千个中文app中怎么可能都使用。
- lib 包如果 适配机型大多为高通 RAM ，可以单独引用abiFilters "armeabi-v7a"。
- build文件中 resConfigs "zh" 剔除掉 官方中或者第三方库中的 外国文字资源。

### 优化WebView内存泄露

WebView的内存泄露主要是因为在页面销毁后，WebView的资源无法马上释放所导致的。现在主流的是两种方法：

1）不在xml布局中添加webview标签，采用在代码中new出来的方式，并在页面销毁的时候去释放webview资源

```java
//addviewprivate 
WeakReference<BaseWebActivity> webActivityReference = new WeakReference<BaseWebActivity>(this);
mWebView = new BridgeWebView(webActivityReference .get());
webview_container.addView(mWebView);
//销毁View
Parent parent = mWebView.getParent();
if (parent != null) {  
  ((ViewGroup) parent).removeView(mWebView);
}
mWebView.stopLoading();
mWebView.getSettings().setJavaScriptEnabled(false);
mWebView.clearHistory();
mWebView.clearView();
mWebView.removeAllViews();
mWebView.destroy()；
mWebView=null；
```

2）另起一个进程加载webview，页面销毁后干掉这个进程。但是这个方法的麻烦之处就在于进程间通信。

使用方法很简单，xml文件中写出进程名即可，销毁的时候调用System.exit(0)

```xml
<activity android:name=".WebActivity"   
android:process=":remoteweb"/>

System.exit(0)   
```

#### webView还有哪些可以优化的地方

- 提前初始化或者使用全局WebView。首次初始化WebView会比第二次初始化慢很多。初始化后，即使WebView已释放，但一些多WebView共用的全局服务/资源对想仍未释放，而第二次初始化不需要生成，因此初始化变快。
- DNS采用和客户端API相同的域名，DNS解析也是耗时比较多的部分，所以用客户端API相同的域名因为其DNS会被缓存，所以打开webView的时候就不会再耗时在DNS上了
- 对于JS的优化，尽量不要用偏重的框架，比如React。其次是高性能要求页面还是需要后端渲染。最后就是app中的网页框架要统一，这样就可以对js进行缓存和复用。

这里有美团团队的总结方案，如下：

- WebView初始化慢，可以在初始化同时先请求数据，让后端和网络不要闲着。
- 后端处理慢，可以让服务器分trunk输出，在后端计算的同时前端也加载网络静态资源。
- 脚本执行慢，就让脚本在最后运行，不阻塞页面解析。
- 同时，合理的预加载、预缓存可以让加载速度的瓶颈更小。
- WebView初始化慢，就随时初始化好一个WebView待用。
- DNS和链接慢，想办法复用客户端使用的域名和链接。
- 脚本执行慢，可以把框架代码拆分出来，在请求页面之前就执行好。

## 简述Android系统启动流程

主要考察点在：

1. Android有哪些主要的系统进程？
2. 这些系统进程是怎么启动的？
3. 进程启动之后主要做了些什么事情？

### 启动流程

![启动流程](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-225912.png)

 启动流程回答步骤：

- Zygote是怎么启动的？
- SystemServer是怎么启动的？
- 系统服务是怎么启动的？

#### 系统进程

系统启动过程中，先创建第一个用户空间进程 `init` 进程，在初始化的过程中，`init` 进程会读取启动配置文件 `init.rc `，解析出来对应 `Server` 服务进行启动。

```text
service zygote /system/bin/app_process ...
service servicemanager /system/bin/servicemanager ...
service surfaceflinger /system/bin/surfaceflinger ...
service media /system/bin/mediaserver ...
等等...
```

#### Zygote 是怎么启动的？

`Zygote` 进程就是基于 `init` 进程 `fork` 出来，在 `Zygote` 进程初始化过程中，启动了 `Java` 虚拟机，注册 `JNI` 函数，并且预加载了大量系统资源，启动 `SystemServer` 服务，最后进入 `Socket Loop` 中，等待不断地接收消息与处理消息。

- `init` 进程 `fork` 出 `zygote` 进程
- 启动虚拟机，注册 `JNI` 函数
- 预加载系统资源
- 启动 `SystemServer`
- 进入 `Socket Loop`，不断地接收消息与处理消息

#### Zygote 工作流程

当 `Socket Loop` 有消息的时候，就会执行 `runOnce` 函数。

`readArgumentList` 函数读取发过来的参数列表，通过 `forkAndSpecialize` 创建子进程， 该方法会返回两次，一个是从子进程返回 `pid == 0`，一个是从父进程返回该子进程的 `pid`

```java
boolean runOnce() {
    String[] args = readArgumentList();
    
    int pid = Zygote.forkAndSpecialize(...);
    
    if(pid == 0) {
        handleChildProc(parsedArgs, ...);
        
        // should never get here, the child is expected to either
        // throw ZygoteInit.MethodAndArgsCaller or exec().
        return true;
    } else {
        return handleParentProc(pid, ...);    
    }
}
```

#### SystemServer 是怎么启动的？

`SystemServer` 通过 Zygote 进程启动后，会执行一系列的初始化和启动操作，包括启动核心系统服务、创建应用程序线程池、注册 Binder 服务等。`SystemServer` 进程完成初始化和启动后，系统的核心功能就准备就绪，其他应用程序可以开始启动了。例如：桌面。

```java
// Zygote 进程中
public static void main(String[] argv) {
        ...
        // 启动 SystemServer
        startSystemServer();
        ...
}

private static boolean startSystemServer(...) {
    String args[] = {
        ...
        "com.android.server.SystemServer",
    };
    
    int pid = Zygote.forkSystemServer(...);
    
    if(pid == 0) {
        handleSystemServerProcess(parsedArgs);    
    }
    
    return true;
}
```

```java
void handleSystemServerProcess(Arguments parsedArgs) {
    RuntimeInit.zygoteInit(parsedArgs.targetSdkVersion,
        parsedArgs.remainingArgs, ...);
}


void zygoteInit(String[] argv, ...) {
    commonInit(); // 常规的一些初始化
    nativeZygoteInit(); // 主要调用了 onZygoteInit 方法，启动了Binder机制，并且启动了一个Binder线程，主要为了与其他进程进行通讯
    applicationInit(argv, ...); // 主要是调用Java类的入口函数。
}

virtual void onZygoteInit() {
    sp<ProcessState> proc = ProcessState::self();
    proc->startThreadPool();
}
        
void applicationInit() {
    invokeStaticMain(args, ...);
}
// SystemServer main 函数，Java类的执行入口
public static void main(String[] args) {
    new SystemServer().run();
}

// SystemServer run 函数
private void run() {
    Looper.prepareMainLooper();
    
    System.loadLibrary("android_servers");
    createSystemContext();
    
    // 分批启动
    startBootstrapServices();
    startCoreServices();
    startOtherServices();
    
    Looper.loop();
}

```

#### 怎么解析系统服务启动的互相依赖

- 分批启动
  - AMS
  - PMS
  - PKMS
  - ...
- 分阶段启动
  - 阶段一
  - 阶段二
  - ...

#### 桌面如何启动

在 AMS 服务就绪的时候，会触发 `systemReady` 函数

桌面其实也是一个系统级别的应用。  
```java
public void systemReady(final Runnable goingCallback) {
        ...
        startHomeActivityLocked(mCurrentUserId, "systemReady")
        ...
}
// startHomeActivityLocked 会启动一个 LoaderTask
mLoaderTask = new LoaderTask(mApp.getContext(), loadFlags);

// mLoaderTask 会通过 pm 查询所有已安装的应用
mPm.queryIntentActivitiesAsUser

```

### app启动交互逻辑

![Launcher启动App流程](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-225809.png)

讲完app启动逻辑以后，可以再介绍一下冷启动、温启动、热启动的内容。

>冷启动是从启动进程开始。
>温启动是进程起来了，要启动组建。
>热启动是进程和组建都启动了，只需要从后台拉到前台。

[冷启动、温启动、热启动](https://github.com/zhouhuandev/Knowledge#%E5%86%B7%E5%90%AF%E5%8A%A8%E6%B8%A9%E5%90%AF%E5%8A%A8%E7%83%AD%E5%90%AF%E5%8A%A8)

### Android系统中启动的第一个进程是哪个？

(一般不会问这么深，可以作为知识扩展了解)

这个问题涉及到内核层的启动情况了。

在Kernel层，Android系统会启动linux内核。

我们知道Android的核心系统服务都是基于Linux内核的，但是这个Linux内核到底该怎么理解呢？

Linux内核并不指的是Linux操作系统，内核只包括最基本的内存模型，进程调度，权限安全等等。操作系统值得是一个更广的概念，不光有内核，还有自己的设备驱动，应用程序框架以及一些应用程序软件等等。所以Android、Ubuntu等都是基于Linux内核的不同的操作系统。

所以启动了linux内核，就是启动了内核中内存模型，进程调度，安全机制，加载驱动等等，而linux内核中的功能都需要上册的虚拟机进行调用执行。

内核中就启动了系统中的第一个进程：

- swapper进程(pid=0)，该进程又称为idle进程, 系统初始化过程Kernel由无到有开创的第一个进程, 用于初始化进程管理、内存管理。并且会加载屏幕硬件，相机硬件等，这一步就会涉及到待会说到的HAL层了。

### 第一个用户级进程是哪个？

init进程是Android系统中用户空间的第一个进程，是所有用户进程的鼻祖。

启动入口在system/core/init/init.cpp文件中，init进程中主要做了这些事：

- 孵化出用户守护进程。守护进程就是运行在后台的特殊进程，它不存在控制终端，会周期性处理一些任务。比如logd进程，就是用来进行日志的读写操作。
- 启动了一些重要服务。比如开机动画。
- 孵化了Zygote进程。Zygote进程大家都或多或少了解一些了，我们所有的应用程序都是由它孵化出来的。
- 孵化了Media Server进程，用来启动和管理整个C++ framework，比如相机服务（camera Service）。

### Zygote进程做了些什么工作？

- 创建服务端Socket，为后续创建进程通信做准备。

- 加载虚拟机。没错，在Zygote进程中，会去加载下层的虚拟机。

- fork了System Server进程。SystemServer进程大家应该都熟悉了吧，是Zygote fork的第一个进程，负责启动和管理Java Framework层，包括ActivityManagerService，PackageManagerService，WindowManagerService、binder线程池等等。这就涉及到APP的启动流程了，后续几篇会细说下。

- fork了第一个应用进程——Launcher，以及后续的一些系统应用进程，这就到了最上面一层——应用层了。

### Activity启动流程中，大部分都是用Binder通讯，为啥跟Zygote通信的时候要用socket呢

- ServiceManager （初始化binder线程池的地方）不能保证在zygote起来的时候已经初始化好，所以无法使用Binder。
- Binder工作依赖于多线程，但是fork的时候是不允许存在多线程的，多线程情况下进程fork容易造成死锁，所以就不用Binder了。

## Activity 生命周期和启动模式

![20191226185212765.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2021/images/20191226185212765.png)

### 生命周期七种方法

- onCreate()方法：Activity首次创建时被调用。用于设置Activity的布局文件，绑定按钮监听器等一些普通静态操作。
- onStart()方法：在Activity即将可见时调用。
- onResume()方法：在Activity获取焦点开始与用户交互时调用。
- onPause()方法：在当前Activity被其他Activity覆盖或锁屏时调用。一般用于保存当前Activity中的数据。
- onStop()方法：在Activity对用户不可见时调用。
- onDestroy()方法：调用Activity的finish()方法或Android系统资源不足时被调用。
- onRestart()方法：在Activity从停止状态再次启动时调用

### 生命周期五种状态

- 启动状态：Activity的启动状态很短暂，当Activity启动后便会进入运行状态。
- 运行状态：Activity在此状态时处于屏幕最前端，它是可见、有焦点的，可以与用户进行交互。如单击、长按等事件。即使出现内存不足的情况，Android也会先销毁栈底的Activity，来确保当前的Activity正常运行。
- 暂停状态：在某些情况下，Activity对用户来说仍然可见，但它无法获取焦点，用户对它操作没有没有响应，此时它处于暂停状态。
- 停止状态：当Activity完全不可见时，它处于停止状态，但仍然保留着当前的状态和成员信息。如系统内存不足，那么这种状态下的Activity很容易被销毁。
- 销毁状态：当Activity处于销毁状态时，将被清理出内存。

### 生命周期三大循环

1.Activity的entire lifetime（全部的生命期）发生在调用onCreate()和调用onDestroy()之间。

在onCreate()方法中执行全局状态的建立(例如定义布局)，在onDestroy()方法中释放所有保存的资源。

2.Activity的visible lifetime(可见的生命期)发生在调用onStart()和onStop()之间。

在这个期间，用户能在屏幕上看见Activity，和它进行交互。系统在Activity的完整寿命中可能多次调用onStart()和onStop(),正如Activity交替地对用户可见或隐藏。

3.Activity的foreground lifetime (前台的生命期)发生在调用onResume()和onPause()之间。

在这期间，Activity在屏幕上所有其他Activity的前面，有用户输入焦点。

### Activity 的四大启动模式（LaunchMode）

#### standard – 默认模式

standard：标准模式也是系统的默认模式。每次启动一个 Activity 都会重新创建一个新的实例，不管这个实例是否已经存在。这种模式下，谁启动了这个 Activity，那么这个 Activity 就运行在启动它的那个 Activity 所在的栈中。如 Activity A 启动了 Activity B（B 是标准模式），那么 B 就会进入到 A 所在的栈中。

当我们用 ApplicationContext 去启动 standard 模式的 Activity 时会报错，错误如下：

```log
android.util.AndroidRuntimeException: Calling startActivity() from outsideof an Activity  context requires the FLAG_ACTIVITY_NEW_TASK flag. Is this really what you want?
```

这是因为 standard 模式的 Activity 默认会进入启动它的 Activity 所属的任务栈中，由于非 Activity 类型的 Context （如 ApplicationContext）并没有所属的任务栈，所以就出问题了。解决方法是为待启动 Activity 指定 FLAG_ACTIVITY_NEW_TASK 标记位，这样启动的时候会创建一个新的任务栈，这时待启动的 Activity 是以 singleTask 模式启动的

#### singleTop – 栈顶复用模式

singleTop 栈顶复用模式。如果新 Activity 已经位于任务栈的栈顶，那么此 Activity 不会被重新创建，同时它的 onNewIntent 方法会被调用，通过此方法的参数可以取出当前请求的信息。需要注意的是，这个 Activity 的 onCreate、onStart 不会被系统重新调用，因为它并没有发生改变。如果新 Activity 的实例已经存在但不是位于栈顶，那么新 Activity 仍然会重建。

适合接收通知启动的内容显示页面，当收到多条新闻推送时，用于展示新闻的 Activity 设置成此模式，根据传来的 Intent 数据显示不同的新闻信息，不会启动多个 Activity。

#### singleTask – 栈内复用模式

singleTask：栈内复用模式。这是一种单实例模式，在这种模式下，只要 Activity 在一个栈中存在，那么多次启动此 Activity 都不会重新创建实例，复用时会将它上面的 Activity 全部出栈，同时它的 onNewIntent 方法会被调用。这个过程存在一个任务栈匹配，因为这个模式启动时会在自己需要的任务栈中寻找实例，这个任务栈通过 taskAffinity 属性指定，如果这个任务栈不存在，则会创建这个任务栈。

taskAffinity 标识了一个 Activity 所需的任务栈的名字，默认情况下，所有 Activity 所需的任务栈的名字为应用的包名。我们可以为每个 Activity 都单独指定 TaskAffinity 属性，这个属性必须不能和包名相同，否则就相当于没有指定。TaskAffinity 属性主要和 singleTask 启动模式或者 allowTaskReparenting 属性配对使用。另外，任务栈分为前台任务栈和后台任务栈，后台任务栈中的 Activity 处于暂停状态，用户可以通过切换将后台任务栈再次调到前台。

适合作为程序入口点，例如浏览器的主界面，不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走 onNewIntent，并且会清空主界面上的其它页面

#### singleInstance – 单实例模式

singleInstance：单实例模式。该模式除了具备 singleTask 模式的所有特性外，该模式的 Activity 只能单独的位于一个任务栈中，具有全局唯一性，即整个系统中只有这一个实例，由于栈内复用的特性，后续的请求均不会创建新的Activity实例，除非这个特殊的任务栈被销毁了。以singleInstance模式启动的Activity在整个系统中是单例的，如果在启动这样的Activiyt时，已经存在了一个实例，那么会把它所在的任务调度到前台，重用这个实例

#### 如何指定 Activity 的启动模式？

1. 通过 AndroidMenifest 为 Activity 指定启动模式，如下所示：

```xml
<activity     
          android:name=".activity.protocol.ProtocolActivity"     
          android:launchMode="singleTask"/>
```

2. 通过 Intent 中设置标志位来为 Activity 指定启动模式，如下所示：

```java
    Intent intent = new Intent();
    intent.setClass(MainActivity.this, SecondActivity.class);
    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    startActivity(intent);
```

优先级上第二种优先级高于第一种，当两种同时存在时，以第二种方式为准；这两种方式的限定方式不同，第一种无法直接为 Activity 设定 FLAG_ACTIVITY_CLEAR_TOP 标识，第二种无法为 Activity 指定 singleInstance 模式。

#### Activity 常用 Flags

##### FLAG_ACTIVITY_NEW_TASK

为 Activity 指定 singleTask 启动模式，效果和在 XML 中指定该模式相同

FLAG_ACTIVITY_SINGLE_TOP
为 Activity 指定 singleTop 启动模式，效果和在 XML 中指定该模式相同

FLAG_ACTIVITY_CLEAR_TOP
具有此标记的 Activity，当它启动时，在同一个任务栈中所有位于它上面的 Activity 都要出栈

FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS
具有这个标记的 Activity 不会出现在历史 Activity 的列表中，在某些情况下我们不希望用户通过历史列表回到我们的 Activity 的时候这个标记比较有用。它等同于在 XML 中指定 Activity 的属性 android:excludeFromRecents="true"。

### Fragment生命周期，当hide，show，replace时候生命周期变化

1）生命周期：

- onAttach()：Fragment和Activity相关联时调用。可以通过该方法获取Activity引用，还可以通过getArguments()获取参数。
- onCreate()：Fragment被创建时调用。
- onCreateView()：创建Fragment的布局。
- onActivityCreated()：当Activity完成onCreate()时调用。
- onStart()：当Fragment可见时调用。
- onResume()：当Fragment可见且可交互时调用。
- onPause()：当Fragment不可交互但可见时调用。
- onStop()：当Fragment不可见时调用。
- onDestroyView()：当Fragment的UI从视图结构中移除时调用。
- onDestroy()：销毁Fragment时调用。
- onDetach()：当Fragment和Activity解除关联时调用。

每个调用方法对应的生命周期变化：

- add(): onAttach()->…->onResume()。
- remove(): onPause()->…->onDetach()。
- replace(): 相当于旧Fragment调用remove()，新Fragment调用add()。remove()+add()的生命周期加起来
- show(): 不调用任何生命周期方法，调用该方法的前提是要显示的 Fragment已经被添加到容器，只是纯粹把Fragment UI的setVisibility为true。
- hide(): 不调用任何生命周期方法，调用该方法的前提是要显示的Fragment已经被添加到容器，只是纯粹把Fragment UI的setVisibility为false。

### Activity 与 Fragment，Fragment 与 Fragment之间怎么交互通信

#### Activity 与 Fragment通信

Activity有Fragment的实例，所以可以执行Fragment的方法，或者传入一个接口。同样，Fragment可以通过getActivity()获取Activity的实例，也是可以执行方法。

#### Fragment 与 Fragment之间通信

1）直接获取另一个Fragmetn的实例

```
getActivity().getSupportFragmentManager().findFragmentByTag("mainFragment");
```

2）接口回调 一个Fragment里面去实现接口，另一个Fragment把接口实例传进去。

3）Eventbus等框架。

### Fragment遇到viewpager遇到过什么问题吗

- 滑动的时候，调用setCurrentItem方法，要注意第二个参数smoothScroll。传false，就是直接跳到fragment，传true，就是平滑过去。一般主页切换页面都是用false。
- 禁止预加载的话，调用setOffscreenPageLimit(0)是无效的，因为方法里面会判断是否小于1。需要重写setUserVisibleHint方法，判断fragment是否可见。
- 不要使用getActivity()获取activity实例，容易造成空指针，因为如果fragment已经onDetach()了，那么就会报空指针。所以要在onAttach方法里面，就去获取activity的上下文。
- FragmentStatePagerAdapter对limit外的Fragment销毁，生命周期为onPause->onStop->onDestoryView->onDestory->onDetach, onAttach->onCreate->onCreateView->onStart->onResume。也就是说切换fragment的时候有可能会多次onCreateView，所以需要注意处理数据。
- 由于可能多次onCreateView，所以我们可以把view保存起来，如果为空再去初始化数据。见代码：

```java
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {  
  if (null == mFragmentView) {  
    mFragmentView = inflater.inflate(getContentViewLayoutID(), null);  
    ButterKnife.bind(this, mFragmentView);           
    isDestory = false;        
    initViewsAndEvents();      
  } 
  return mFragmentView;
}
```

## 进程间通讯

### Binder机制和AIDL

#### 为什么要使用 Binder 机制

Android 会为每个进程分配独立的虚拟内存空间，每个进程的虚拟地址空间是互相隔离的，如果进程间要进行互相通信，就要使用 Android 提供的 Binder 机制来进行通信。

#### 进程间通信的方式

1. 使用 Intent 来进行进程间通信，比如在调用百度地图的时候，使用 Intent ，用startActivity 来启动百度地图的指定页面
2. 使用共享文件的方式，比如 A 、 B 进程要进行通信，那么A 进程可以把要通信的内容通过序列化的方式写入到本地文件，然后 B 进程再通过读取本地文件的方式获取A 要传递的内容
3. 使用 Messager 或者AIDL（Android Interface Define Language，事实上 Messager 也是通过 AIDL 实现的，只是 Android 为我们进行了进行了简单的封装），AIDL 的底层也是通过 Binder 来完成进程间通信的。
4. 使用 ContentProvider 进行进程间通信，作为四大组件，底层使用的也是 Binder 来完成进程间通信的
5. 使用 Socket，Socket 可以实现计算机网络中的两个进程间通信，当然也可以用于进程间通信服务端监听指定的端口，客户端链接指定的端口，成功建立链接以后，拿到 socket 对象客户端就可以向服务端发送消息，或者接受服务端传来的消息。

![IPC方式的优缺点和适用场景](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-235056.png)

### 什么是 Binder

> Binder 是 Android 为我们提供的 IPC（Inner Process Commnuication进程间通信）的一种方式，Android 中的 Activity 、Service、Broadcast、ContentProvider 都是运行在不同的进程中，Binder 是他们之间进行通信的桥梁。

### Binder通信过程和原理

![Binder通信过程和原理](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-235222.png)

首先要定义要传输的对象，并进行序列化，反序列化，然后在相同包名下定义 AIDL 接口、要传输的对象，然后定义好了以后，再 makeProject ，生成对应的类，然后再创建一个 Service 写服务端的代码，然后再客户端使用 bindService 方法进行连接，再 ServiceConnection 的回调方法中拿到服务端的接口对象，之后就可以往服务端发送消息。

1. AIDL 接口，是一个 interface，
2. Sub 类，是 Binder 的实现类，服务端通过 Sub 类来提供服务
3. Proxy 类，服务端的本地代理，客户端通过这个类调用服务端提供的方法。只有处于不同进程间通信的时候才会调用到，一个进程内不会使用这个类，直接使用 Sub 类就可以完成通信
4. asInterface() 客户端调用，用来将 IBinder 对象转换为客户端需要的服务端的 AIDL 接口类型的对象。如果客户端和服务端处于同一个进程则返回 Sub ，不同进程返回 Sub.Proxy对象
5. asBinder()根据当前调用情况返回代理Proxy的Binder对象
6. onTransact() 运行在服务端的 Binder 线程池中，当客户端发起跨进程请求以后，请求会通过这个方法来处理
7. transact() 运行在客户端，当客户端发起远程请求以后将当前线程挂起，之后调用服务端的 onTransact知 直到请求返回，当前线程才继续执行。

> 当多个模块需要 AID 来进行 IPC 的时候，此时需要创建多个 AIDL 文件，那么相应的 Service 就会有很多，必然会出现系统资源耗费严重，解决办法是创建 Binder 连接池，即将每个业务模块的 Binder 请求统一转发到一个远程 Service 中去执行，从而避免重复建立 Service。原理大概是：
> 每个业务模块创建自己的AIDL接口并实现此接口，然后向服务端提供自己的唯一标识和其对应的Binder对象。服务端只需要一个Service并提供一个queryBinder接口，它会根据业务模块的特征来返回相应的Binder对象，不同的业务模块拿到所需的Binder对象后就可以进行远程方法的调用了。

### 为什么使用 Binder

> - 性能高，效率高：传统的IPC（套接字、管道、消息队列）需要拷贝两次内存、Binder只需要拷贝一次内存、共享内存不需要拷贝内存。
> - 安全性好：接收方可以从数据包中获取发送方的进程Id和用户Id，方便验证发送方的身份，其他IPC想要实验只能够主动存入，但是这有可能在发送的过程中被修改。

#### 效率高

传输效率主要影响因素是内存拷贝的次数，拷贝次数越少，传输速率越高，传统的 IPC 使用消息队列、Socket和管道，数据先从发送方的用户空间缓存区拷贝到内核空间开辟的缓存区，然后再从内核缓存区拷贝到接收方的用户空间缓存区，一共拷贝两次。

共享内存的方式不用拷贝，效率很高，但是实现的复杂度很高 。

而 Binder 只用拷贝一次 ，使用Binder 的话，进程 A、B通讯，只用拷贝数据到内核缓存区，内核缓存区和B 的缓存区是映射到同一块物理地址的，节省了一次拷贝的过程。

另外，引用 Brian Swetland 大神的回答：

>Q: Why does [Binder] need to be done in the kernel? Couldn't any of the current Linux IPC mechanisms be re-used to accomplish this?
>
>A: Brian Swetland answers here:
>
>I believe the two notable properties of the binder that are not present in existing IPC mechanisms in the kernel (that I'm aware of) are:
>
>avoiding copies by having the kernel copy from the writer into a ring buffer in the reader's address space directly (allocating space if necessary)
>managing the lifespan of proxied remoted userspace objects that can be shared and passed between processes (upon which the userspace binder library builds its remote reference counting model)

大意就是：

1. 避免内核空间到数据接受端的直接的数据拷贝；数据接受端接收数据的时候，由于数据大小不确定，要么分配一个很大的空间装数据，要么动态扩容；两种方式都有问题；Binder使用mmap直接把接收端的内存映射到内存空间，避免了数据的直接拷贝；另外通过data_buffer等方式让数据仅包含定长的消息头，解决了接收端内存分配的问题。

2. 需要管理跨进程传递的代理对象的生命周期；这一点其他机制无法完成；Binder驱动通过引用计数技术解决这个问题。

##### 下面描述 Binder 传输过程（A进程向B进程传递数据）

1. 首先 Binder 驱动在内核空间开辟一块 **数据接收内存缓存区**
2. 接着在内核空间开辟一块 **内核缓存区**，建立**内核缓存区**和**数据接收缓存区**的映射关系，以及内核中的**数据接收缓存区**和**B用户空间 缓存区**的映射关系。
3. **发送方进程 A 使用 copyfromuser（），将数据拷贝到内核缓存区，由于内核缓存区和数据接收缓存区有映射关系，数据接收缓存区和B进程的用户空间缓存区有映射关系，因此也就相当于把数据从 A 进程传递到了 B 进程。**

![A进程向B进程传递数据](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-235354.png)

#### 稳定性好

上面说到共享内存的性能优于Binder，那为什么不采用共享内存呢，因为共享内存需要处理并发同步问题，容易出现死锁和资源竞争，稳定性较差。Socket虽然是基于C/S架构的，但是它主要是用于网络间的通信且传输效率较低。Binder基于C/S架构 ，Server端与Client端相对独立，稳定性较好。

#### 安全性

传统的 Linux IPC 的接收方无法获得对方进程可靠的 UID/PID，从而无法鉴别对方的身份，而Binder 机制为每个进程分配了 UID/PID，并且在 Binder 通信过程中会根据 UID/PID进行有效的身份验证。

### IPC 进程通讯方式

- Intent 、Bundle ：要求传递数据能被序列化，实现 Parcelable、Serializable ，适用于四大组件通信。
- 文件共享 ：适用于交换简单的数据实时性不高的场景。
- AIDL：AIDL 接口实质上是系统提供给我们可以方便实现 BInder 的工具。
- Messenger：基于 AIDL 实现，服务端串行处理，主要用于传递消息，适用于低并发一对多通信。
- ContentProvider：基于 Binder 实现，适用于一对多进程间数据共享。（通讯录 短信 等）
- Socket：TCP、UDP，适用于网络数据交换

### 进程保活

#### 进程被杀原因

- 切到后台内存不足时被杀；
- 切到后台厂商省电机制杀死；
- 用户主动清理。

#### 保活方式

- Activity 提权：挂一个 1像素 Activity 将进程优先级提高到前台进程。
- Service 提权：启动一个前台服务。（API>18会有正在运行通知栏）
- 广播拉活 。（监听 开机 等系统广播）
- Service 拉活。
- JobScheduler 定时任务拉活 。（android 高版本不行）
- 双进程拉活。
- 监听其他大厂 广播。（tx baidu 全家桶互相拉）

### Hook

Hook android 使用
<https://www.jianshu.com/p/4f6d20076922>

Hook 的选择点：静态变量和单例，因为一旦创建对象，它们不容易变化，非常容易定位。

Hook 过程：

寻找 Hook 点，原则是静态变量或者单例对象，尽量 Hook public 的对象和方法。

选择合适的代理方式，如果是接口可以用动态代理。

偷梁换柱——用代理对象替换原始对象。

多数插件化 也使用的 Hook技术

### 讲一讲进程和线程？

- 进程是对运行时程序的封装，是系统进行资源分配和调度的基本单元，而线程是进程的子任务，是CPU分配和调度的基本单元。
- 一个进程可以有多个线程，但是一个线程只能属于一个进程。
- 进程的创建需要系统分配内存和CPU，文件句柄等资源，销毁时也要进行相应的回收，所以进程的管理开销很大；但是线程的管理开销则很小。
- 进程之间不会互相影响；而一个线程崩溃会导致进程崩溃，从而影响同个进程里面的其他线程。

联系：进程与线程之间的关系：线程是存在进程的内部，一个进程中可以有多个线程，一个线程只能存在一个进程。

### 进程之间如何通讯?

- 管道通讯

  - 无名管道（pipe）：管道是一种半双工的通信方式，数据只能单向流动，而且只能在（父子进程或者兄弟进程之间）使用。进程的亲缘关系通常是指父子进程关系。
  - 高级管道（popen）：将另一个程序当做一个新的进程在当前程序进程中启动，则它算是当前程序的子进程，这种方式我们称为高级管道方式。
  - 有名管道（named pipe）：有名管道也是半双工通信方式，但是它允许无亲缘关系进程间的通信。
- 消息队列通信
  - 消息队列（message queue）：消息队列是存放在内核中的消息链表，每个消息队列由消息队列标识符标识。消息队列克服了信号传递信息少，管道只能承载无格式字节流以及缓冲区大小受限等缺点。
- 信号量通信
  - 信号量（semophore）：信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。
- 信号
  - 信号（sinal）：信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生。
- 共享内存通信
  - 共享内容（shared memory）：共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但是多个进程都可以访问。共享内存是最快的IPC方式，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号量，配合使用，来实现进程间的同步和通信。
- 套接字通信
  - 套接字（Socket）：套接字也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同机器间的进程通信。

## View

### Window WindowManager WMS

- Window ：抽象类 不是实际存在的，而是以 View 的形式存在，通过 PhoneWindow 实现。（PhoneWindow = DecorView = Title + ContentView）
- WindowManager：外界访问 Window 的入口 管理Window 中的View , 内部通过 Binder 与 WMS IPC 进程交互。
- WMS：管理窗口 Surface 的布局和次序，作为系统级服务单独运行在一个进程。
- SurfaceFlinger：将 WMS 维护的窗口按一定次序混合后显示到屏幕上。

### View 工作流程

通过 SetContentView()，调用 到PhoneWindow ，后实例DecorView ，通过 LoadXmlResourceParser() 进行IO操作 解析xml文件 通过反射 创建出View，并将View绘制在 DecorView上，这里的绘制则交给了ViewRootImpl 来完成，通过performTraversals() 触发绘制流程，performMeasure 方法获取View的尺寸，performLayout 方法获取View的位置 ，然后通过 performDraw 方法遍历View 进行绘制。

### `onCreate()`、`onResume()` 中可以获取View的宽高吗？怎么做？ `View.post{}` 为什么可以获取？

View的宽高是在onLayout阶段才能最终确定的，而在Activity#onCreate中并不能保证View已经执行到了onLayout方法，也就是说Activity的声明周期与View的绘制流程并不是一一绑定。所以onCreate() 和 onResume() 中获取不到View的宽高值。以Handler为基础，`View.post()` 将传入任务的执行时机调整到 View 绘制完成之后。

原因：View 的测绘绘制流程就是从 ViewRootImpl#performTraversals() 开始的，而这个方法的调用是在 onResume() 方法之后，所以在 onCreate() 和 onResume() 方法中拿不到 View 的测量值。

### View的 `getWidth()` 和 `getMeasuredWidth()` 有什么区别吗？

View的高宽是由View本身和Parent容器共同决定的。

`getMeasuredWidth()` 和 `getWidth()` 分别对应于视图绘制的measure和layout阶段。

- `getMeasuredWidth()` 获取的是View原始的大小，也就是这个View在XML文件中配置或者是代码中设置的大小。
- `getWidth()` 获取的是这个View最终显示的大小，这个大小有可能等于原始的大小，也有可能不相等。比如说，在父布局的onLayout()方法或者该View的onDraw()方法里调用measure(0, 0)，二者的结果可能会不同（measure中的参数可以自己定义）。

#### getWidth()

```java
    /**
     * Return the width of the your view.
     * @return The width of your view, in pixels.
     */
    @ViewDebug.ExportedProperty(category = "layout")
    public final int getWidth() {
        return mRight - mLeft;
    }
```

从源码上看，`getWidth()` 是根据 `mRight` 和 `mLeft` 之间的差值计算出来的，需要在布局之后才能确定它们的坐标，也就是说布局后在 `onLayout()` 方法里才能调用 `getWidth()` 来获取。因此，`getWidth()` 获取的宽度是在View设定好布局后整个View的宽度。

#### getMeasuredWidth()

```java
    /**
     * Like {@link #getMeasuredWidthAndState()}, but only returns the
     * raw width component (that is the result is masked by
     * {@link #MEASURED_SIZE_MASK}).
     *
     * @return The raw measured width of this view.
     */
    public final int getMeasuredWidth() {
        return mMeasuredWidth & MEASURED_SIZE_MASK;
    }
```

从源码上看，`getMeasuredWidth()` 获取的是 `mMeasuredWidth` 的这个值。这个值是一个8位的十六进制的数字，高两位表示的是这个measure阶段的Mode的值，具体可以查看MeasureSpec的原理。这里 `mMeasuredWidth & MEASURED_SIZE_MASK` 表示的是测量阶段结束之后，View真实的值。而且这个值会在调用了 `setMeasuredDimensionRaw()` 函数之后会被设置。所以 `getMeasuredWidth()` 的值是measure阶段结束之后得到的View的原始的值。

```java
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }

    protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {
        boolean optical = isLayoutModeOptical(this);
        if (optical != isLayoutModeOptical(mParent)) {
            Insets insets = getOpticalInsets();
            int opticalWidth  = insets.left + insets.right;
            int opticalHeight = insets.top  + insets.bottom;

            measuredWidth  += optical ? opticalWidth  : -opticalWidth;
            measuredHeight += optical ? opticalHeight : -opticalHeight;
        }
        setMeasuredDimensionRaw(measuredWidth, measuredHeight);
    }

    private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
        mMeasuredWidth = measuredWidth;
        mMeasuredHeight = measuredHeight;

        mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;
    }
```

总结一下，`getMeasuredWidth` 是measure阶段获得的View的原始宽度，`getWidth` 是layout阶段完成后，其在父容器中所占的最终宽度

### View.post{} 为什么可以获取控件尺寸？

#### 为什么可以在View.post()中获取控件尺寸呢？

android 运行是消息驱动，通过源码 可以看到 ViewRootImpl 中 是先将 TraversalRunnable添加到 Handler 中运行的 之后 才是 View.post()。

```java
ViewRootImpl.class

final class TraversalRunnable implements Runnable {
    @Override
    public void run() {
        doTraversal();
    }
}

void doTraversal() {
    if (mTraversalScheduled) {
        mTraversalScheduled = false;
        mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);
        // 该方法之后才有 view.post（） 
        performTraversals();
        ...
    }
}
```

因此，这个时候Handler正在执行着TraversalRunnable这个Runnable，而我们post的Runnable要等待TraversalRunnable执行完才会去执行，而TraversalRunnable这里面又会进行measure,layout和draw流程，所以等到执行我们的Runnable时，此时的View就已经被measure过了，所以获取到的宽高就是measure过后的宽高。

参考博客1
<https://blog.csdn.net/scnuxisan225/article/details/49815269>

参考博客2
<https://www.cnblogs.com/dasusu/p/8047172.html>

想要在 onCreate 中获取到View宽高的方法有：

#### View.post(runnable)

```java
view.post(new Runnable() {
            @Override
            public void run() {
                int width = view.getWidth();
                int measuredWidth = view.getMeasuredWidth();
                Log.i(TAG, "width: " + width);
                Log.i(TAG, "measuredWidth: " + measuredWidth);
            }
        });
```

利用Handler通信机制，发送一个Runnable到MessageQueue中，当View布局处理完成时，自动发送消息，通知UI进程。借此机制，巧妙获取View的高宽属性，代码简洁，相比ViewTreeObserver监听处理，还不需要手动移除观察者监听事件。

##### 源码分析 View.post(runnable)

```java
public boolean post(Runnable action) {

    final AttachInfo attachInfo = mAttachInfo;
    if (attachInfo != null) {
        return attachInfo.mHandler.post(action);
    }
    getRunQueue().post(action);
    return true;
}
```

View.post(runable) 通过将runable 封装为HandlerAction对象，如果attachInfo为null 则将Runnable事件 添加到等待数组中， attachInfo初始化是在 dispatchAttachedToWindow 方法，置空则是在detachedFromWindow方法中，所以在这两个方法生命周期中，调用View.post方法都是直接让 mAttachInfo.handler 执行。

```java
ViewRootImpl.class

mAttachInfo = new View.AttachInfo(mWindowSession, mWindow, display, this, mHandler, this, context);

final ViewRootHandler mHandler = new ViewRootHandler();
```

通过查找 mAttachInfo.handler 是在主线程中声明的，没有传参则 Looper 为主线程Looper，所以在View.post中可以更新UI。

#### ViewTreeObserver 监听界面绘制事件

监听View的onLayout()绘制过程，一旦layout触发变化，立即回调onLayoutChange方法。
注意，使用完也要主要调用removeOnGlobalListener()方法移除监听事件。避免后续每一次发生全局View变化均触发该事件，影响性能。

```java
ViewTreeObserver vto = view.getViewTreeObserver();
        vto.addOnGlobalLayoutListener(new OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                view.getViewTreeObserver().removeGlobalOnLayoutListener(this);
                Log.i(TAG, "width: " + view.getWidth());
                Log.i(TAG, "height: " + view.getHeight());
            }
        });
```

#### View.measure(int widthMeasureSpec, int heightMeasureSpec)

除了在 `onCreate()` 中获得View的高宽，还可以在Activity的 `onWindowFocusChanged()` 方法中获得高宽。

### 说说View/ViewGroup的绘制流程

View的绘制流程是从ViewRoot的performTraversals开始的，它经过measure，layout，draw三个过程最终将View绘制出来。performTraversals会依次调用performMeasure，performLayout，performDraw三个方法，他们会依次调用measure，layout，draw方法，然后又调用了onMeasure，onLayout，dispatchDraw。

**measure** ：对于自定义的单一view的测量，只需要根据父 view 传递的MeasureSpec进行计算大小。
对于ViewGroup的测量，一般要重写onMeasure方法，在onMeasure方法中，父容器会对所有的子View进行Measure，子元素又会作为父容器，重复对它自己的子元素进行Measure，这样Measure过程就从DecorView一级一级传递下去了，也就是要遍历所有子View的的尺寸，最终得出出总的viewGroup的尺寸。Layout和Draw方法也是如此。

**layout** ：根据 measure 子 View 所得到的布局大小和布局参数，将子View放在合适的位置上。
对于自定义的单一view，计算本身的位置即可。

对于ViewGroup来说，需要重写onlayout方法。除了计算自己View的位置，还需要确定每一个子View在父容器的位置以及子view的宽高（getMeasuredWidth和getMeasuredHeight），最后调用所有子view的layout方法来设定子view的位置。

**draw** ：把 View 对象绘制到屏幕上。

draw（）会依次调用四个方法：

- 1）drawBackground()，根据在 layout 过程中获取的 View 的位置参数，来设置背景的边界。
- 2）onDraw()，绘制View本身的内容，一般自定义单一view会重写这个方法，实现一些绘制逻辑。
- 3） dispatchDraw()，绘制子View
- 4）onDrawScrollBars(canvas)，绘制装饰，如 滚动指示器、滚动条、和前景.

### 说说你理解的MeasureSpec

MeasureSpec是由父View的MeasureSpec和子View的LayoutParams通过简单的计算得出一个针对子View的测量要求，这个测量要求就是MeasureSpec。

首先，MeasureSpec是一个大小跟模式的组合值,MeasureSpec中的值是一个整型（32位）将size和mode打包成一个Int型，其中高两位是mode，后面30位存的是size

```java
// 获取测量模式  
int specMode = MeasureSpec.getMode(measureSpec)  
// 获取测量大小 
int specSize = MeasureSpec.getSize(measureSpec) 
// 通过Mode 和 Size 生成新的SpecMode  
int measureSpec=MeasureSpec.makeMeasureSpec(size, mode);
```

 其次，每个子View的MeasureSpec值根据子View的布局参数和父容器的MeasureSpec值计算得来的，所以就有一个父布局测量模式，子视图布局参数，以及子view本身的MeasureSpec关系图：

![MeasureSpec关系图](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-235611.png)

其实也就是getChildMeasureSpec方法的源码逻辑，会根据子View的布局参数和父容器的MeasureSpec计算出来单个子view的MeasureSpec。

最后是实际应用时：

对于自定义的单一view，一般可以不处理onMeasure方法，如果要对宽高进行自定义，就重写onMeasure方法，并将算好的宽高通过setMeasuredDimension方法传进去。对于自定义的ViewGroup，一般需要重写onMeasure方法，并且调用measureChildren方法遍历所有子View并进行测量（measureChild方法是测量具体某一个view的宽高），然后可以通过getMeasuredWidth/getMeasuredHeight获取宽高，最后通过setMeasuredDimension方法存储本身的总宽高。

### requestLayout和invalidate

requestLayout方法是用来触发绘制流程，他会会一层层调用 parent 的requestLayout，一直到最上层也就是ViewRootImpl的requestLayout，这里也就是判断线程的地方了，最后会执行到performMeasure -> performLayout -> performDraw 三个绘制流程，也就是测量——布局——绘制。

```java
@Override
public void requestLayout() {  
  if (!mHandlingLayoutInLayoutRequest) { 
    checkThread();     
    mLayoutRequested = true;   
    scheduleTraversals();
    //执行绘制流程   
  }
}
```

其中performMeasure方法会执行到View的measure方法，用来测量大小。performLayout方法会执行到view的layout方法，用来计算位置。performDraw方法需要注意下，他会执行到view的draw方法，但是并不一定会进行绘制，只有只有 flag 被设置为 PFLAG_DIRTY_OPAQUE 才会进行绘制。

invalidate方法也是用来触发绘制流程，主要表现就是会调用draw()方法。虽然他也会走到scheduleTraversals方法，也就是会走到三大流程，但是View会通过mPrivateFlags来判断是否进行onMeasure和onLayout操作。而在用invalidate方法时，更新了mPrivateFlags，所以不会进行measure和layout。同时他也会设置Flag为PFLAG_DIRTY_OPAQUE，所以肯定会执行onDraw方法。

```java
private void invalidateRectOnScreen(Rect dirty) {    
  final Rect localDirty = mDirty;   
  //...    
  if (!mWillDrawSoon && (intersected || mIsAnimating)) {      
    scheduleTraversals();
    //执行绘制流程   
  }
}
```

最后看一下scheduleTraversals方法中三大绘制流程逻辑，是不是我们之前说的那样，FORCE_LAYOUT标志才会onMeasure和onLayout，PFLAG_DIRTY_OPAQUE标志才会onDraw：

```java
 
    public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        final boolean forceLayout = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT;
        // 只有mPrivateFlags为PFLAG_FORCE_LAYOUT的时候才会进行onMeasure方法  
        if (forceLayout || needsLayout) {
            onMeasure(widthMeasureSpec, heightMeasureSpec);
        }
        // 设置 LAYOUT_REQUIRED flag   
        mPrivateFlags |= PFLAG_LAYOUT_REQUIRED;
    }

    public void layout(int l, int t, int r, int b) {   
        ...
        //判断标记位为PFLAG_LAYOUT_REQUIRED的时候才进行onLayout方法   
        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);
        }
    }

    public void draw(Canvas canvas) {
        final int privateFlags = mPrivateFlags;
        // flag 是 PFLAG_DIRTY_OPAQUE 则需要绘制   
        final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE && (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
        mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;
        if (!dirtyOpaque) {
            drawBackground(canvas);
        }
        if (!dirtyOpaque)
            onDraw(canvas);
        // 绘制 Child  
        dispatchDraw(canvas);
        // foreground 不管 dirtyOpaque 标志，每次都会绘制  
        onDrawForeground(canvas);
    }

```

参考文章中有一段总结挺好的：

> 虽然两者都是用来触发绘制流程，但是在measure和layout过程中，只会对 flag 设置为 FORCE_LAYOUT 的情况进行重新测量和布局，而draw方法中只会重绘flag为 dirty 的区域。requestLayout 是用来设置FORCE_LAYOUT标志，invalidate 用来设置 dirty 标志。所以 requestLayout 只会触发 measure 和 layout，invalidate 只会触发 draw。

### 动画(4种类型)

View 动画的作用对象是View，它支持4中动画效果，分别是平移动画(TranslateAnimation<translate>)、缩放动画(ScaleAnimation<scale>)、旋转动画(RotateAnimation<rotate>)、透明动画(AlphaAnimation<alpha>)。除了这四种典型的变换效果外，帧动画也属于View动画，但是帧动画的表现形式和上面的四种变换效果不大一样。

#### 帧动画

**帧动画** ：AnimationDrawable 实现，在资源文件中存放多张图片，占用内存多，容易OOM。

#### 补间动画

**补间动画** ：作用对象只限于 View 视觉改变，并没有改变View 的 xy 坐标，支持 平移、缩放、旋转、透明度，但是移动后，响应时间的位置还在 原处，补间动画在执行的时候，直接导致了 View 执行 onDraw() 方法。补间动画的核心本质就是在一定的持续时间内，不断改变 Matrix 变换，并且不断刷新的过程。

#### 属性动画

**属性动画** ：ObjectAnimator、ValuetAnimator、AnimatorSet 可以是任何View，动画选择也比较多，其中包含 差速器，可以控制动画速度，节奏。类型估值器 可以根据当前属性改变的百分比计算改变后的属性值 。因为ViewGroup 在 getTransformedMotionEvent方法中通过子 View 的 hasIdentityMatrix() 来判断子 View 是否经过位移之类的属性动画。调用子 View 的 getInverseMatrix() 做「反平移」操作，然后判断处理后的触摸点是否在子 View 的边界范围内。

提升动画 可以打开 硬件加速，使GPU 承担一部分CPU的工作。

### Window中的token是什么，有什么用？

token？又是个啥呢？刚才window操作过程中也没出现啊。

token其实大家应该工作中会发现一点踪迹，比如application的上下文去创建dialog的时候，就会报错：

```log
unable to add window --token null
```

所以这个token跟window操作是有关系的，翻到刚才的addview方法中，还有个细节我们没说到，就是adjustLayoutParamsForSubWindow方法。

```java
    //Window.java
    void adjustLayoutParamsForSubWindow(WindowManager.LayoutParams wp) {
        if (wp.type >= WindowManager.LayoutParams.FIRST_SUB_WINDOW && wp.type <= WindowManager.LayoutParams.LAST_SUB_WINDOW) {
            //子Window       
            if (wp.token == null) {
                View decor = peekDecorView();
                if (decor != null) {
                    wp.token = decor.getWindowToken();
                }
            }
        } else if (wp.type >= WindowManager.LayoutParams.FIRST_SYSTEM_WINDOW && wp.type <= WindowManager.LayoutParams.LAST_SYSTEM_WINDOW) {
            //系统Window   
        } else {
            //应用Window    
            if (wp.token == null) {
                wp.token = mContainer == null ? mAppToken : mContainer.mAppToken;
            }
        }
    }
```

上述代码分别代表了三个Window的类型：

- 子Window。需要从decorview中拿到token。
- 系统Window。不需要token。
- 应用Window。直接拿mAppToken，mAppToken是在setWindowManager方法中传进来的，也就是新建Window的时候就带进来了token。

然后在WMS中的addWindow方法会验证这个token，下次说到WMS的时候再看看。

所以这个token就是用来验证是否能够添加Window，可以理解为权限验证，其实也就是为了防止开发者乱用context创建window。

拥有token的context（比如Activity）就可以操作Window。没有token的上下文（比如Application）就不允许直接添加Window到屏幕（除了系统Window）。

### Application中可以直接弹出Dialog吗？

这个问题其实跟上述问题相关：

- 如果直接使用Application的上下文是不能创建Window的，而Dialog的Window等级属于子Window，必须依附与其他的父Window，所以必须传入Activity这种有window的上下文。
- 那有没有其他办法可以在Application中弹出dialog呢？有，改成系统级Window：

//检查权限

```kotlin
if (!Settings.canDrawOverlays(this)) {   
  val intent = Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION) 
  intent.data = Uri.parse("package:$packageName") 
  startActivityForResult(intent, 0)
}
dialog.window.setType(WindowManager.LayoutParams.TYPE_SYSTEM_DIALOG)

<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
```

- 另外还有一种办法，在Application类中，可以通过registerActivityLifecycleCallbacks监听Activity生命周期，不过这种办法也是传入了Activity的context，只不过在Application类中完成这个工作。

### Activity从创建到我们看到界面，发生了哪些事

首先是通过setContentView加载布局，这其中创建了一个DecorView，然后根据然后根据activity设置的主题（theme）或者特征（Feature）加载不同的根布局文件，最后再通过inflate方法加载layoutResID资源文件，其实就是解析了xml文件，根据节点生成了View对象。流程图：

![Activity创建流程图](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-235737.png)

其次就是进行view绘制到界面上，这个过程发生在handleResumeActivity方法中，也就是触发onResume的方法。在这里会创建一个ViewRootImpl对象，作为DecorView的parent然后对DecorView进行测量布局和绘制三大流程。流程图：

![流程图](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231015-235858.png)

### Activity、View、Window 之间的关系

每个 Activity 包含了一个 Window对象，这个对象是由 PhoneWindow做的实现。而 PhoneWindow 将 DecorView作为了一个应用窗口的根 View，这个 DecorView 又把屏幕划分为了两个区域：一个是 TitleView，一个是ContentView，而我们平时在 Xml 文件中写的布局正好是展示在 ContentView 中的。

### Activity、PhoneWindow、DecorView、ViewRootImpl 的关系？

- PhoneWindow是Window 的唯一子类，每个Activity都会创建一个PhoneWindow对象，你可以理解它为一个窗口，但不是真正的可视窗口，而是一个管理类，是Activity和整个View系统交互的接口，是Activity和View交互系统的中间层。
- DecorView是PhoneWindow的一个内部类，是整个View层级的最顶层，一般包括标题栏和内容栏两部分，会根据不同的主题特性调整不同的布局。它是在setContentView方法中被创建，具体点来说是在PhoneWindow的installDecor方法中被创建。
- ViewRootImpl是DecorView的parent，用来控制View的各种事件，在handleResumeActivity方法中被创建。

### 说说Android的事件分发机制完整流程，也就是从点击屏幕开始，事件会怎么传递

我觉得事件分发机制流程可以分为三部分，分别是从外传里，从里传外，消费之后。

1）首先，从最外面一层传到最里面一层：

如果当前是viewgroup层级，就会判断 onInterceptTouchEvent是否为true，如果为true，则代表事件要消费在这一层级，不再往下传递。接着便执行当前 viewgroup 的onTouchEvent方法。如果onInterceptTouchEvent为false，则代表事件继续传递到下一层级的 dispatchTouchEvent方法，接着一样的代码逻辑，一直到最里面一层的view。

伪代码解释：

```java
public boolean dispatchTouchEvent(MotionEvent event) { 
  boolean isConsume = false;   
  if (isViewGroup) {     
    if (onInterceptTouchEvent(event)) {  
      isConsume = onTouchEvent(event);      
    } else {         
      isConsume = child.dispatchTouchEvent(event); 
    }   
  } else {  
    //isView  
    isConsume = onTouchEvent(event);   
  }  
  return isConsume;
}
```

2）到最里层的view之后，view本身还是可以选择消费或者传到外面。

到最里面一层就会直接执行onTouchEvent方法，这时候，view有没有权利拒绝消费事件呢？按道理view作为最底层的，应该是没有发言权才对。但是呢，秉着公平公正原则，view也是可以拒绝的，可以在onTouchEvent方法返回false，表示他不想消费这个事件。那么它的父容器的onTouchEvent又会被调用，如果父容器的onTouchEvent又返回false，则又交给上一级。一直到最上层，也就是Activity的onTouchEvent被调用。

伪代码解释：

```java
public void handleTouchEvent(MotionEvent event) {   
  if (!onTouchEvent(event)) {  
    getParent.onTouchEvent(event); 
  }
}
```

3）消费之后

当某一层viewGroup的onInterceptTouchEvent为true，则代表当前层级要消费事件。如果它的onTouchListener被设置了的话，则onTouch会被调用，如果onTouch的返回值返回true，则onTouchEvent不会被调用。如果返回false或者没有设置onTouchListener，则会继续调用onTouchEvent。而onClick方法则是设置了onClickListener则会被正常调用。

伪代码解释：

```java
public void consumeEvent(MotionEvent event) { 
  if (setOnTouchListener) {     
    int tag = onTouch();    
    if (!tag) {       
      onTouchEvent(event);  
    }   
  } else { 
    onTouchEvent(event); 
  }   
  if (setOnClickListener) { 
    onClick();  
  }
}
```

#### 解决滑动冲突的办法

解决滑动冲突的根本就是要在适当的位置进行拦截，那么就有两种解决办法：

- 外部拦截：从父view端处理，根据情况决定事件是否分发到子view
- 内部拦截：从子view端处理，根据情况决定是否阻止父view进行拦截，其中的关键就是requestDisallowInterceptTouchEvent方法。

1）外部拦截法，其实就是在onInterceptTouchEvnet方法里面进行判断，是否拦截，见代码：

```java
//外部拦截法：父view.java  
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {   
  boolean intercepted = false;    
  //父view拦截条件   
  boolean parentCanIntercept; 
  switch (ev.getActionMasked()) { 
    case MotionEvent.ACTION_DOWN:  
      intercepted = false;       
      break;      
    case MotionEvent.ACTION_MOVE: 
      if (parentCanIntercept) {      
        intercepted = true;          
      } else {              
        intercepted = false;  
      }          
      break; 
    case MotionEvent.ACTION_UP:   
      intercepted = false;       
      break; 
  }   
  return intercepted;
}
```

还是比较简单的，直接判断拦截条件，然后返回true就代表拦截，false就不拦截，传到子view。注意的是ACTION_DOWN状态不要拦截，如果拦截，那么后续事件就直接交给父view处理了，也就没有拦截不拦截的问题了。

2）内部拦截法，就是通过requestDisallowInterceptTouchEvent方法让父view不要拦截。

```java
//父view.java   
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {   
  if (ev.getActionMasked() == MotionEvent.ACTION_DOWN) {       
    return false;   
  } else { 
    return true; 
  }
}
//子view.java
@Override
public boolean dispatchTouchEvent(MotionEvent event) {  
  //父view拦截条件   
  boolean parentCanIntercept;   
  switch (event.getActionMasked()) {   
    case MotionEvent.ACTION_DOWN:      
      getParent().requestDisallowInterceptTouchEvent(true);   
      break;      
    case MotionEvent.ACTION_MOVE:  
      if (parentCanIntercept) {          
        getParent().requestDisallowInterceptTouchEvent(false);      
      }           
      break;  
    case MotionEvent.ACTION_UP:  
      break;   
  }   
  return super.dispatchTouchEvent(event);
}
```

requestDisallowInterceptTouchEvent(true)的意思是阻止父view拦截事件，也就是传入true之后，父view就不会再调用onInterceptTouchEvent。反之，传入false就代表父view可以拦截，也就是会走到父view的onInterceptTouchEvent方法。所以需要父view拦截的时候，就传入flase，需要父view不拦截的时候就传入true。

### 关于事件分发，事件到底是先到DecorView还是先到Window的？

经过上述一系列问题，是不是对Window印象又深了点呢？最后再看一个问题，这个是wanandroid论坛上看到的(<https://wanandroid.com/wenda/show/12119>)，

这里的window可以理解为PhoneWindow，其实这道题就是问事件分发在Activity、DecorView、PhoneWindow中的顺序。

当屏幕被触摸，首先会通过硬件产生触摸事件传入内核，然后走到FrameWork层（具体流程感兴趣的可以看看参考链接），最后经过一系列事件处理到达ViewRootImpl的processPointerEvent方法，接下来就是我们要分析的内容了：

```java
//ViewRootImpl.java 
private int processPointerEvent(QueuedInputEvent q) {   
  final MotionEvent event = (MotionEvent)q.mEvent;       
  ...        
    //mView分发Touch事件，mView就是DecorView          
    boolean handled = mView.dispatchPointerEvent(event);      
  ...      
}

//DecorView.java   
public final boolean dispatchPointerEvent(MotionEvent event) {   
  if (event.isTouchEvent()) {        
    //分发Touch事件        
    return dispatchTouchEvent(event);  
  } else {        
    return dispatchGenericMotionEvent(event);  
  }  
}   

@Override   
public boolean dispatchTouchEvent(MotionEvent ev) {  
  //cb其实就是对应的Activity  
  final Window.Callback cb = mWindow.getCallback();   
  return cb != null && !mWindow.isDestroyed() && mFeatureId < 0 ? cb.dispatchTouchEvent(ev) : super.dispatchTouchEvent(ev);  
}

//Activity.java   
public boolean dispatchTouchEvent(MotionEvent ev) {   
  if (ev.getAction() == MotionEvent.ACTION_DOWN) {     
    onUserInteraction();      
  }      
  if (getWindow().superDispatchTouchEvent(ev)) {  
    return true;   
  }      
  return onTouchEvent(ev); 
}

//PhoneWindow.java  
@Override   
public boolean superDispatchTouchEvent(MotionEvent event) {   
  return mDecor.superDispatchTouchEvent(event);  
}

//DecorView.java  
public boolean superDispatchTouchEvent(MotionEvent event) {     
  return super.dispatchTouchEvent(event);  
}    
```

事件的分发流程就比较清楚了：

`ViewRootImpl`——>`DecorView`——>`Activity`——>`PhoneWindow`——>`DecorView`——>`ViewGroup`

（这其中就用到了getCallback参数，也就是之前addView中传入的callback，也就是Activity本身）

但是这个流程确实有些奇怪，为什么绕来绕去的呢，光DecorView就走了两遍。

参考链接中的说法我还是比较认同的，主要原因就是解耦。

- ViewRootImpl并不知道有Activity这种东西存在，它只是持有了DecorView。所以先传给了DecorView，而DecorView知道有AC，所以传给了AC。
- Activity也不知道有DecorView，它只是持有PhoneWindow，所以这么一段调用链就形成了。

### Scroller是怎么实现View的弹性滑动？

- 在MotionEvent.ACTION_UP事件触发时调用startScroll()方法，该方法并没有进行实际的滑动操作，而是记录滑动相关量（滑动距离、滑动时间）
- 接着调用invalidate/postInvalidate()方法，请求View重绘，导致View.draw方法被执行
- 当View重绘后会在draw方法中调用computeScroll方法，而computeScroll又会去向Scroller获取当前的scrollX和scrollY；然后通过scrollTo方法实现滑动；接着又调用postInvalidate方法来进行第二次重绘，和之前流程一样，如此反复导致View不断进行小幅度的滑动，而多次的小幅度滑动就组成了弹性滑动，直到整个滑动过成结束。

```java
mScroller = new Scroller(context);
@Override
public boolean onTouchEvent(MotionEvent event) {  
  switch (event.getAction()) {      
    case MotionEvent.ACTION_UP:      
      // 滚动开始时X的坐标,滚动开始时Y的坐标,横向滚动的距离,纵向滚动的距离  
      mScroller.startScroll(getScrollX(), 0, dx, 0);     
      invalidate();            
      break;    
  }   
  return super.onTouchEvent(event);
}

@Override
public void computeScroll() {   
  // 重写computeScroll()方法，并在其内部完成平滑滚动的逻辑   
  if (mScroller.computeScrollOffset()) {    
    scrollTo(mScroller.getCurrX(), mScroller.getCurrY()); 
    invalidate();  
  }
}
```

## RecyclerView

### RecyclerView预取机制与缓存机制

讲一下RecyclerView的缓存机制,滑动10个，再滑回去，会有几个执行onBindView。缓存的是什么？cachedView会执行onBindView吗?

这两个问题都是关于缓存的，我就一起说了。

#### 1）首先说下RecycleView的缓存结构

Recycleview有四级缓存，分别是mAttachedScrap(屏幕内)，mCacheViews(屏幕外)，mViewCacheExtension(自定义缓存)，mRecyclerPool(缓存池)

- mAttachedScrap(屏幕内)，用于屏幕内itemview快速重用，不需要重新createView和bindView
- mCacheViews(屏幕外)，保存最近移出屏幕的ViewHolder，包含数据和position信息，复用时必须是相同位置的ViewHolder才能复用，应用场景在那些需要来回滑动的列表中，当往回滑动时，能直接复用ViewHolder数据，不需要重新bindView。
- mViewCacheExtension(自定义缓存)，不直接使用，需要用户自定义实现，默认不实现。
- mRecyclerPool(缓存池)，当cacheView满了后或者adapter被更换，将cacheView中移出的ViewHolder放到Pool中，放之前会把ViewHolder数据清除掉，所以复用时需要重新bindView。

#### 四级缓存按照顺序需要依次读取。所以完整缓存流程是

##### 保存缓存流程

- 插入或是删除itemView时，先把屏幕内的ViewHolder保存至AttachedScrap中
- 滑动屏幕的时候，先消失的itemview会保存到CacheView，CacheView大小默认是2，超过数量的话按照先入先出原则，移出头部的itemview保存到RecyclerPool缓存池（如果有自定义缓存就会保存到自定义缓存里），RecyclerPool缓存池会按照itemview的itemtype进行保存，每个itemType缓存个数为5个，超过就会被回收。

##### 获取缓存流程

- AttachedScrap中获取，通过pos匹配holder——>获取失败，从CacheView中获取，也是通过pos获取holder缓存 ——>获取失败，从自定义缓存中获取缓存——>获取失败，从mRecyclerPool中获取 ——>获取失败，重新创建viewholder——createViewHolder并bindview。

#### 3）了解了缓存结构和缓存流程，我们再来看看具体的问题 滑动10个，再滑回去，会有几个执行onBindView？

由之前的缓存结构可知，需要重新执行onBindView的只有一种缓存区，就是缓存池mRecyclerPool。

所以我们假设从加载RecyclView开始盘的话（页面假设可以容纳7条数据）：

- 首先，7条数据会依次调用onCreateViewHolder和onBindViewHolder。
- 往下滑一条（position=7），那么会把position=0的数据放到mCacheViews中。此时mCacheViews缓存区数量为1，mRecyclerPool数量为0。然后新出现的position=7的数据通过postion在mCacheViews中找不到对应的ViewHolder，通过itemtype也在mRecyclerPool中找不到对应的数据，所以会调用onCreateViewHolder和onBindViewHolder方法。
- 再往下滑一条数据（position=8），如上。
- 再往下滑一条数据（position=9），position=2的数据会放到mCacheViews中，但是由于mCacheViews缓存区默认容量为2，所以position=0的数据会被清空数据然后放到mRecyclerPool缓存池中。而新出现的position=9数据由于在mRecyclerPool中还是找不到相应type的ViewHolder，所以还是会走onCreateViewHolder和onBindViewHolder方法。所以此时mCacheViews缓存区数量为2，mRecyclerPool数量为1。
- 再往下滑一条数据（position=10），这时候由于可以在mRecyclerPool中找到相同viewtype的ViewHolder了。所以就直接复用了，并调用onBindViewHolder方法绑定数据。
- 后面依次类推，刚消失的两条数据会被放到mCacheViews中，再出现的时候是不会调用onBindViewHolder方法，而复用的第三条数据是从mRecyclerPool中取得，就会调用onBindViewHolder方法了。

#### 4）所以这个问题就得出结论了（假设mCacheViews容量为默认值2）

- 如果一开始滑动的是新数据，那么滑动10个，就会走10个bindview方法。然后滑回去，会走10-2个bindview方法。一共18次调用。
- 如果一开始滑动的是老数据，那么滑动10-2个，就会走8个bindview方法。然后滑回去，会走10-2个bindview方法。一共16次调用。

但是但是，实际情况又有点不一样。因为Recycleview在v25版本引入了一个新的机制，预取机制。

预取机制，就是在滑动过程中，会把将要展示的一个元素提前缓存到mCachedViews中，所以滑动10个元素的时候，第11个元素也会被创建，也就多走了一次bindview方法。但是滑回去的时候不影响，因为就算提前取了一个缓存数据，只是把bindview方法提前了，并不影响总的绑定item数量。

所以滑动的是新数据的情况下就会多一次调用bindview方法。

#### 5）总结，问题怎么答呢？

- 四级缓存和流程说一下。
- 滑动10个，再滑回去，bindview可以是18次调用，可以是16次调用。
- 缓存的其实就是缓存item的view，在Recycleview中就是viewholder。
- cachedView就是mCacheViews缓存区中的view，是不需要重新绑定数据的。

### 如何实现RecyclerView的局部更新，用过payload吗,notifyItemChange方法中的参数？

- `notifyDataSetChanged()`，刷新全部可见的item。
- `notifyItemChanged(int)`，刷新指定item。
- `notifyItemRangeChanged(int,int)`，从指定位置开始刷新指定个item。
- `notifyItemInserted(int)`、`notifyItemMoved(int)`、`notifyItemRemoved(int)`。插入、移动一个并自动刷新。
- `notifyItemChanged(int, Object)`，局部刷新。

可以看到，关于view的局部刷新就是 `notifyItemChanged(int, Object)` 方法，下面具体说说：

notifyItemChange有两个构造方法：

- `notifyItemChanged(int position, @Nullable Object payload)`
- `notifyItemChanged(int position)`

其中 `payload` 参数可以认为是你要刷新的一个标识，比如我有时候只想刷新 `itemView` 中的 `textview`,有时候只想刷新 `imageview` ？又或者我只想某一个view的文字颜色进行高亮设置？那么我就可以通过 `payload` 参数来标示这个特殊的需求了。

具体怎么做呢？比如我调用了 `notifyItemChanged（14,"changeColor"）`,那么在 `onBindViewHolder` 回调方法中做下判断即可：

```java
@Override
public void onBindViewHolder(ViewHolderholder, int position, List<Object> payloads) {
  if (payloads.isEmpty()) {  
    // payloads为空，说明是更新整个ViewHolder    
    onBindViewHolder(holder, position);  
  } else {      
    // payloads不为空，这只更新需要更新的View即可。   
    String payload = payloads.get(0).toString();      
    if ("changeColor".equals(payload)) {       
      holder.textView.setTextColor("");  
    } 
  }
}
```

### RecyclerView嵌套RecyclerView滑动冲突，NestScrollView嵌套RecyclerView

1）RecyclerView嵌套RecyclerView的情况下，如果两者都要上下滑动，那么就会引起滑动冲突。默认情况下外层的RecycleView可滑，内层不可滑。

之前说过解决滑动冲突的办法有两种：内部拦截法和外部拦截法。这里我提供一种内部拦截法，还有一些其他的办法大家可以自己思考下。

```kotlin
holder.recyclerView.setOnTouchListener { v, event ->
    when(event.action){
        //当按下操作的时候，就通知父view不要拦截，拿起操作就设置可以拦截，正常走父view的滑动。
        MotionEvent.ACTION_DOWN,MotionEvent.ACTION_MOVE -> v.parent.requestDisallowInterceptTouchEvent(true)
        MotionEvent.ACTION_UP -> v.parent.requestDisallowInterceptTouchEvent(false)
    }
    false
}
```

2）关于ScrclerView的滑动冲突还是同样的解决办法，就是进行事件拦截。还有一个办法就是用Nestedscrollview代替ScrollView，Nestedscrollview是官方为了解决滑动冲突问题而设计的新的View。它的定义就是支持嵌套滑动的ScrollView。

所以直接替换成Nestedscrollview就能保证两者都能正常滑动了。但是要注意设置RecyclerView.setNestedScrollingEnabled(false)这个方法，用来取消RecyclerView本身的滑动效果。

这是因为RecyclerView默认是setNestedScrollingEnabled(true)，这个方法的含义是支持嵌套滚动的。也就是说当它嵌套在NestedScrollView中时,默认会随着NestedScrollView滚动而滚动,放弃了自己的滚动。所以给我们的感觉就是滞留、卡顿。所以我们将它设置为false就解决了卡顿问题，让他正常的滑动，不受外部影响。

## LeakCanary 原理

参考博客
<https://www.cnblogs.com/jymblog/p/11656221.html>

通过 registerActivityLifecycleCallbacks 监听Activity或者Fragment 销毁时候的生命周期。（如果不想那个对象被监控则通过 AndroidExcludedRefs 枚举，避免被检测）

```java
  public void watch(Object watchedReference, String referenceName) {
        if (this == DISABLED) {
            return;
        }
        checkNotNull(watchedReference, "watchedReference");
        checkNotNull(referenceName, "referenceName");
        final long watchStartNanoTime = System.nanoTime();
        String key = UUID.randomUUID().toString();
        retainedKeys.add(key);
        final KeyedWeakReference reference = new KeyedWeakReference(watchedReference, key, referenceName, queue);
        ensureGoneAsync(watchStartNanoTime, reference);
    }
```

然后通过弱引用和引用队列监控对象是否被回收。（弱引用和引用队列ReferenceQueue联合使用时，如果弱引用持有的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。即 KeyedWeakReference持有的Activity对象如果被垃圾回收，该对象就会加入到引用队列queue）

```java
void waitForIdle(final Retryable retryable, final int failedAttempts) {  
  // This needs to be called from the main thread.  
  Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {  
    @Override public boolean queueIdle() {    
      postToBackgroundWithDelay(retryable, failedAttempts);    
      return false;    
    }
  });
}
```

IdleHandler，就是当主线程空闲的时候，如果设置了这个东西，就会执行它的queueIdle()方法，所以这个方法就是在onDestory以后，一旦主线程空闲了，就会执行一个延时五秒的子线程任务，任务：检测到未被回收则主动 gc ，然后继续监控，如果还是没有回收掉，就证明是内存泄漏了。通过抓取 dump文件，在使用 第三方 HAHA 库 分析文件，获取到到达泄露点最近的线路，通过 启动另一个进程的 DisplayLeakService 发送通知 进行消息的展示。

## OkHttp

### 同步请求流程

通过OkHttpClient new生成call实例 Realcall。

Dispatcher.executed() 中 通过添加realcall到runningSyncCalls队列中。

通过 getResponseWithInterceptorChain() 对request层层拦截，生成Response。

通过Dispatcher.finished()，把call实例从队列中移除，返回最终的response。

### 异步请求流程

生成一个AsyncCall(responseCallback)实例(实现了Runnable)。

AsyncCall通过调用Dispatcher.enqueue()，并判断maxRequests （最大请求数）maxRequestsPerHost(最大host请求数)是否满足条件，如果满足就把AsyncCall添加到runningAsyncCalls中，并放入线程池中执行；如果条件不满足，就添加到等待就绪的异步队列，当那些满足的条件的执行时 ，在Dispatcher.finifshed(this)中的promoteCalls();方法中 对等待就绪的异步队列进行遍历，生成对应的AsyncCall实例，并添加到runningAsyncCalls中，最后放入到线程池中执行，一直到所有请求都结束。

### 责任链

源码跟进 execute() 进入到 getResponseWithInterceptorChain() 方法。

```java
    Response getResponseWithInterceptorChain() throws IOException {
        //责任链 模式   
        List<Interceptor> interceptors = new ArrayList<>();
        interceptors.addAll(client.interceptors());
        interceptors.add(retryAndFollowUpInterceptor);
        interceptors.add(new BridgeInterceptor(client.cookieJar()));
        interceptors.add(new CacheInterceptor(client.internalCache()));
        interceptors.add(new ConnectInterceptor(client));
        if (!forWebSocket) {
            interceptors.addAll(client.networkInterceptors());
        }
        interceptors.add(new CallServerInterceptor(forWebSocket));
        Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0, originalRequest, this, eventListener, client.connectTimeoutMillis(), client.readTimeoutMillis(), client.writeTimeoutMillis());
        return chain.proceed(originalRequest);
    }
```

chain.proceed() 方法核心代码。每个拦截器 intercept()方法中的chain，都在上一个 chain实例的 chain.proceed()中被初始化，并传递了拦截器List与 index，调用interceptor.intercept(next)，直接最后一个 chain实例执行即停止。

```java
//递归循环下一个 拦截器   
RealInterceptorChain next = new RealInterceptorChain(interceptors, streamAllocation, httpCodec, connection, index + 1, request, call, eventListener, connectTimeout, readTimeout, writeTimeout);
Interceptor interceptor = interceptors.get(index);
Response response = interceptor.intercept(next);
```

```java
@Override 
public Response intercept(Chain chain) throws IOException {  
  Request request = chain.request();   
  RealInterceptorChain realChain = (RealInterceptorChain) chain;   
  Call call = realChain.call();   
  EventListener eventListener = realChain.eventListener();  
  StreamAllocation streamAllocation = new StreamAllocation(client.connectionPool(),   
                                                           createAddress(request.url()), call,
                                                           eventListener, callStackTrace); 
  while (true) {       
    ...       
      // 循环中 再次调用了 chain 对象中的 proceed 方法，达到递归循环。 
      response = realChain.proceed(request, streamAllocation, null, null); 
    releaseConnection = false;     
    ...    
  }
}
```

### OkHttp 流程

- 采用责任链方式的拦截器,实现分成处理网络请求,可更好的扩展自定义拦截器。（采用GZIP压缩，支持http缓存）
- 采用线程池(thread pool)和连接池(Socket pool)解决多并发问题,同时连接池支持多路复用。(http2才支持,可以让一个Socket同时发送多个网络请求,内部自动维持顺序.相比http只能一个一个发送,更能减少创建开销))
- 底层采用socket和服务器进行连接.采用okio实现高效的io流读写。

### OKHttp有哪些拦截器，分别起什么作用

OKHTTP的拦截器是把所有的拦截器放到一个list里，然后每次依次执行拦截器，并且在每个拦截器分成三部分：

- 预处理拦截器内容
- 通过proceed方法把请求交给下一个拦截器
- 下一个拦截器处理完成并返回，后续处理工作。

这样依次下去就形成了一个链式调用，看看源码，具体有哪些拦截器：

```java
Response getResponseWithInterceptorChain() throws IOException {    
  // Build a full stack of interceptors.    
  List<Interceptor> interceptors = new ArrayList<>(); 
  interceptors.addAll(client.interceptors());   
  interceptors.add(retryAndFollowUpInterceptor); 
  interceptors.add(new BridgeInterceptor(client.cookieJar()));  
  interceptors.add(new CacheInterceptor(client.internalCache())); 
  interceptors.add(new ConnectInterceptor(client));   
  if (!forWebSocket) {    
    interceptors.addAll(client.networkInterceptors()); 
  }  
  interceptors.add(new CallServerInterceptor(forWebSocket));  
  Interceptor.Chain chain = new RealInterceptorChain( 
    interceptors, null, null, null, 0, originalRequest);  
  return chain.proceed(originalRequest);
}
```

根据源码可知，一共七个拦截器：

- addInterceptor(Interceptor)，这是由开发者设置的，会按照开发者的要求，在所有的拦截器处理之前进行最早的拦截处理，比如一些公共参数，Header都可以在这里添加。
- RetryAndFollowUpInterceptor，这里会对连接做一些初始化工作，以及请求失败的充实工作，重定向的后续请求工作。跟他的名字一样，就是做重试工作还有一些连接跟踪工作。
- BridgeInterceptor，这里会为用户构建一个能够进行网络访问的请求，同时后续工作将网络请求回来的响应Response转化为用户可用的Response，比如添加文件类型，content-length计算添加，gzip解包。
- CacheInterceptor，这里主要是处理cache相关处理，会根据OkHttpClient对象的配置以及缓存策略对请求值进行缓存，而且如果本地有了可⽤的Cache，就可以在没有网络交互的情况下就返回缓存结果。
- ConnectInterceptor，这里主要就是负责建立连接了，会建立TCP连接或者TLS连接，以及负责编码解码的HttpCodec
- networkInterceptors，这里也是开发者自己设置的，所以本质上和第一个拦截器差不多，但是由于位置不同，所以用处也不同。这个位置添加的拦截器可以看到请求和响应的数据了，所以可以做一些网络调试。
- CallServerInterceptor，这里就是进行网络数据的请求和响应了，也就是实际的网络I/O操作，通过socket读写数据。

### OkHttp怎么实现连接池

#### 为什么需要连接池？

频繁的进行建立Sokcet连接和断开Socket是非常消耗网络资源和浪费时间的，所以HTTP中的keepalive连接对于降低延迟和提升速度有非常重要的作用。

keepalive机制是什么呢？也就是可以在一次TCP连接中可以持续发送多份数据而不会断开连接。所以连接的多次使用，也就是复用就变得格外重要了，而复用连接就需要对连接进行管理，于是就有了连接池的概念。

OkHttp中使用ConectionPool实现连接池，默认支持5个并发KeepAlive，默认链路生命为5分钟。

#### 怎么实现的？

- 1）首先，ConectionPool中维护了一个双端队列Deque，也就是两端都可以进出的队列，用来存储连接。

- 2）然后在ConnectInterceptor，也就是负责建立连接的拦截器中，首先会找可用连接，也就是从连接池中去获取连接，具体的就是会调用到ConectionPool的get方法。

```java
RealConnection get(Address address, StreamAllocation streamAllocation, Route route) {  
  assert (Thread.holdsLock(this));   
  for (RealConnection connection : connections) {   
    if (connection.isEligible(address, route)) {  
      streamAllocation.acquire(connection, true);    
      return connection;     
    }   
  }  
  return null;
}
```

也就是遍历了双端队列，如果连接有效，就会调用acquire方法计数并返回这个连接。

- 3）如果没找到可用连接，就会创建新连接，并会把这个建立的连接加入到双端队列中，同时开始运行线程池中的线程，其实就是调用了ConectionPool的put方法。

```java
public final class ConnectionPool {
    void put(RealConnection connection) {
        if (!cleanupRunning) {
            //没有连接的时候调用          
            cleanupRunning = true;
            executor.execute(cleanupRunnable);
        }
        connections.add(connection);
    }
}
```

- 4）其实这个线程池中只有一个线程，是用来清理连接的，也就是上述的cleanupRunnable

```java
private final Runnable cleanupRunnable = new Runnable(){
    @Override 
    public void run(){
        while(true) {
        //执行清理，并返回下次需要清理的时间。 
        long waitNanos=cleanup(System.nanoTime());
       if(waitNanos==-1)
         return;
       if(waitNanos>0){
          long waitMillis = waitNanos/1000000L;
          waitNanos -= (waitMillis*1000000L);
          synchronized (ConnectionPool.this){        
            //在timeout时间内释放锁           
            try {                      
              ConnectionPool.this.wait(waitMillis, (int) waitNanos); 
            } catch (InterruptedException ignored) {  
            }                
          }      
        }    
    } 
};
```

这个runnable会不停的调用cleanup方法清理线程池，并返回下一次清理的时间间隔，然后进入wait等待。

#### 怎么清理的呢？看看源码

```java
long cleanup(long now) {    
  synchronized (this) {   
    //遍历连接      
    for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {  
      RealConnection connection = i.next();     
      //检查连接是否是空闲状态，      
      //不是，则inUseConnectionCount + 1   
      //是 ，则idleConnectionCount + 1      
      if (pruneAndGetAllocationCount(connection, now) > 0) {   
        inUseConnectionCount++;       
        continue;     
      }     
      idleConnectionCount++;  
      // If the connection is ready to be evicted, we're done.       
      long idleDurationNs = now - connection.idleAtNanos; 
      if (idleDurationNs > longestIdleDurationNs) {      
        longestIdleDurationNs = idleDurationNs;       
        longestIdleConnection = connection;    
      }   
    }  
    //如果超过keepAliveDurationNs或maxIdleConnections，  
    //从双端队列connections中移除   
    if (longestIdleDurationNs >= this.keepAliveDurationNs     
        || idleConnectionCount > this.maxIdleConnections) {   
      connections.remove(longestIdleConnection);
    } else if (idleConnectionCount > 0) {   
      //如果空闲连接次数>0,返回将要到期的时间   
      // A connection will be ready to evict soon.   
      return keepAliveDurationNs - longestIdleDurationNs;  
    } else if (inUseConnectionCount > 0) {    
      // 连接依然在使用中，返回保持连接的周期5分钟  
      return keepAliveDurationNs;    
    } else {        
      // No connections, idle or in use. 
      cleanupRunning = false;     
      return -1;    
    } 
  }   
  closeQuietly(longestIdleConnection.socket());  
  // Cleanup again immediately.  
  return 0;  
}
```

也就是当如果空闲连接maxIdleConnections超过5个或者keepalive时间大于5分钟，则将该连接清理掉。

- 5）这里有个问题，怎样属于空闲连接？

其实就是有关刚才说到的一个方法acquire计数方法：

```java
public void acquire(RealConnection connection, boolean reportedAcquired) { 
  assert (Thread.holdsLock(connectionPool));  
  if (this.connection != null) 
    throw new IllegalStateException();  
  this.connection = connection;  
  this.reportedAcquired = reportedAcquired;   
  connection.allocations.add(new StreamAllocationReference(this, callStackTrace));
}
```

在RealConnection中，有一个StreamAllocation虚引用列表allocations。每创建一个连接，就会把连接对应的StreamAllocationReference添加进该列表中，如果连接关闭以后就将该对象移除。

- 6）连接池的工作就这么多，并不负责，主要就是管理双端队列Deque<RealConnection>，可以用的连接就直接用，然后定期清理连接，同时通过对StreamAllocation的引用计数实现自动回收。

### OkHttp里面用到了什么设计模式

#### 责任链模式

这个不要太明显，可以说是okhttp的精髓所在了，主要体现就是拦截器的使用，具体代码可以看看上述的拦截器介绍。

#### 建造者模式

在Okhttp中，建造者模式也是用的挺多的，主要用处是将对象的创建与表示相分离，用Builder组装各项配置。比如Request：

```java
public class Request {  
  public static class Builder { 
    @Nullable
    HttpUrl url; 
    String method;  
    Headers.Builder headers; 
    @Nullable 
    RequestBody body; 
    public Request build() {     
      return new Request(this);  
    }  
  }
}
```

#### 工厂模式

工厂模式和建造者模式类似，区别就在于工厂模式侧重点在于对象的生成过程，而建造者模式主要是侧重对象的各个参数配置。例子有CacheInterceptor拦截器中又个CacheStrategy对象：

```java

    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();

    public Factory(long nowMillis, Request request, Response cacheResponse) {
        this.nowMillis = nowMillis;
        this.request = request;
        this.cacheResponse = cacheResponse;
        if (cacheResponse != null) {
            this.sentRequestMillis = cacheResponse.sentRequestAtMillis();
            this.receivedResponseMillis = cacheResponse.receivedResponseAtMillis();
            Headers headers = cacheResponse.headers();
            for (int i = 0, size = headers.size(); i < size; i++) {
                String fieldName = headers.name(i);
                String value = headers.value(i);
                if ("Date".equalsIgnoreCase(fieldName)) {
                    servedDate = HttpDate.parse(value);
                    servedDateString = value;
                } else if ("Expires".equalsIgnoreCase(fieldName)) {
                    expires = HttpDate.parse(value);
                } else if ("Last-Modified".equalsIgnoreCase(fieldName)) {
                    lastModified = HttpDate.parse(value);
                    lastModifiedString = value;
                } else if ("ETag".equalsIgnoreCase(fieldName)) {
                    etag = value;
                } else if ("Age".equalsIgnoreCase(fieldName)) {
                    ageSeconds = HttpHeaders.parseSeconds(value, -1);
                }
            }
        }
    }
```

#### 观察者模式

之前我写过一篇文章，是关于Okhttp中websocket的使用，由于webSocket属于长连接，所以需要进行监听，这里是用到了观察者模式：

```java
final WebSocketListener listener;
@Override 
public void onReadMessage(String text) throws IOException {  
  listener.onMessage(this, text);
}
```

注：另外有的博客还说到了策略模式，门面模式等，这些大家可以网上搜搜，毕竟每个人的想法看法都会不同，细心找找可能就会发现。

## Glide

### Glide的缓存机制

Glide的缓存机制，主要分为2种缓存，一种是内存缓存，一种是磁盘缓存。

之所以使用内存缓存的原因是：防止应用重复将图片读入到内存，造成内存资源浪费。

之所以使用磁盘缓存的原因是：防止应用重复的从网络或者其他地方下载和读取数据。

正式因为有着这两种缓存的结合，才构成了Glide极佳的缓存效果。

> （先告诉人家有哪几种缓存，主要是为了什么目的才用的缓存，然后可以看着面试官，要么等着他继续问，如果他不问，等着你，这个时候你就可以继续的往细节处介绍）

### 说一说Glide的三级缓存原理

记得，如果需要具体谈原理时，要先宏观，后细节

读取一张图片的时候，获取顺序：
Lru算法缓存->弱引用缓存->磁盘缓存（如果设置了的话）

> 当我们的APP中想要加载某张图片时，先去LruCache中寻找图片，如果LruCache中有，则直接取出来使用，并将该图片放入WeakReference中，如果LruCache中没有，则去WeakReference中寻找，如果WeakReference中有，则从WeakReference中取出图片使用，如果WeakReference中也没有图片，则从磁盘缓存/网络中加载图片。

注：图片正在使用时存在于 activeResources 弱引用map中

![Glide的三级缓存原理](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231016-000037.png)

将图片缓存的时候，写入顺序：
弱引用缓存->Lru算法缓存->磁盘缓存中

> 当图片不存在的时候，先从网络下载图片，然后将图片存入弱引用中，glide会采用一个acquired（int）变量用来记录图片被引用的次数，
> 当acquired变量大于0的时候，说明图片正在使用中，也就是将图片放到弱引用缓存当中；
> 如果acquired变量等于0了，说明图片已经不再被使用了，那么此时会调用方法来释放资源，首先会将缓存图片从弱引用中移除，然后再将它put到LruResourceCache当中。
> 这样也就实现了正在使用中的图片使用弱引用来进行缓存，不在使用中的图片使用LruCache来进行缓存的功能。

引深：
关于LruCache

> 最近最少使用算法，设定一个缓存大小，当缓存达到这个大小之后，会将最老的数据移除，避免图片占用内存过大导致OOM。
> LruCache 内部用LinkHashMap存取数据，在双向链表保证数据新旧顺序的前提下，设置一个最大内存，往里面put数据的时候，当数据达到最大内存的时候，将最老的数据移除掉，保证内存不超过设定的最大值。

关于LinkedHashMap

> LinkHashMap 继承HashMap，在 HashMap的基础上，新增了双向链表结构，每次访问数据的时候，会更新被访问的数据的链表指针，具体就是先在链表中删除该节点，然后添加到链表头header之前，这样就保证了链表头header节点之前的数据都是最近访问的（从链表中删除并不是真的删除数据，只是移动链表指针，数据本身在map中的位置是不变的）

### Glide加载一个一兆的图片（100 *100），是否会压缩后再加载，放到一个300* 300的view上会怎样，800*800呢，图片会很模糊，怎么处理？

*++因为你缓存机制无论是看博客还是看一些面试宝典，如果只是考原理或者定义，光把上面的文字背诵下来就可以了，但是背诵和真正的理解是两回事，自己没有形成感悟，不理解这个框架，只是一味的迎合面试，这个问题就可以卡住你，另外千万别和面试官嘚瑟，果然，这个面试的哥们，这块就卡住了，支支吾吾的半天没答上来，果然是只看了博客，没真正的阅读过源码++*

当我们调整imageview的大小时，Picasso会不管imageview大小是什么，总是直接缓存整张图片，而Glide就不一样了，它会为每个不同尺寸的Imageview缓存一张图片，也就是说不管你的这张图片有没有加载过，只要imageview的尺寸不一样，那么Glide就会重新加载一次，这时候，它会在加载的imageview之前从网络上重新下载，然后再缓存。

举个例子，如果一个页面的imageview是300 *300像素，而另一个页面中的imageview是100* 100像素，这时候想要让两个imageview像是同一张图片，那么Glide需要下载两次图片，并且缓存两张图片。

```java
public <R> LoadStatus load() {   
  // 根据请求参数得到缓存的键    
  EngineKey key = keyFactory.buildKey(model, signature, width, height, transformations, 
                                      resourceClass, transcodeClass, options);
}
```

看到了吧，缓存Key的生成条件之一就是控件的长宽。

### 简单说一下内存泄漏的场景，如果在一个页面中使用Glide加载了一张图片，图片正在获取中，如果突然关闭页面，这个页面会造成内存泄漏吗？

++*注意一定要审题，因为之前问了这个小伙，内存泄漏的原因，无非是长生命周期引用了短生命周期的对象等等，然后突然画风一变，直接问了Glide加载图片会不会引起图片泄漏，这个小伙想也没想，直接回答道会引起内存泄漏，可以用LeakCanary检测，巴拉巴拉。。。*++

因为Glide 在加载资源的时候，如果是在 Activity、Fragment 这一类有生命周期的组件上进行的话，会创建一个透明的 RequestManagerFragment 加入到FragmentManager 之中，感知生命周期，当 Activity、Fragment 等组件进入不可见，或者已经销毁的时候，Glide 会停止加载资源。

但是如果，是在非生命周期的组件上进行时，会采用Application 的生命周期贯穿整个应用，所以 applicationManager 只有在应用程序关闭的时候终止加载。

### 如何设计一个大图加载框架

概括来说，图片加载包含封装，解析，下载，解码，变换，缓存，显示等操作。

![2019112206351331.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2021/images/2019112206351331.png)

- 封装参数：从指定来源，到输出结果，中间可能经历很多流程，所以第一件事就是封装参数，这些参数会贯穿整个过程；
- 解析路径：图片的来源有多种，格式也不尽相同，需要规范化；
- 读取缓存：为了减少计算，通常都会做缓存；同样的请求，从缓存中取图片（Bitmap）即可；
- 查找文件/下载文件：如果是本地的文件，直接解码即可；如果是网络图片，需要先下载；
- 解码：这一步是整个过程中最复杂的步骤之一，有不少细节，下个博客会说；
- 变换：解码出Bitmap之后，可能还需要做一些变换处理（圆角，滤镜等）；
- 缓存：得到最终bitmap之后，可以缓存起来，以便下次请求时直接取结果；
- 显示：显示结果，可能需要做些动画（淡入动画，crossFade等）。

## ARouter的原理

首先，我们了解下ARouter是干嘛的？ARouter是阿里巴巴研发的一个用于解决组件间，模块间界面跳转问题的框架。所以简单的说，就是用来跳转界面的，不同于平时用到的显式或隐式跳转，只需要在对应的界面上添加注解，就可以实现跳转，看个案例：

```java
@Route(path = "/test/activity")
public class YourActivity extend Activity {  
  ...
}
//跳转
ARouter.getInstance().build("/test/activity").navigation();
```

使用很方便，通过一个path就可以进行跳转了，那么原理是什么呢？

其实仔细思考下，就可以联想到，既然关键跳转过程是通过path跳转到具体的activity，那么原理无非就是把path和Activity一一对应起来就行了。没错，其实就是通过注释，通过apt技术，也就是注解处理工具，把path和activity关联起来了。主要有以下几个步骤：

- 代码里加入的@Route注解，会在编译时期通过apt生成一些存储path和activity.class映射关系的类文件
- app进程启动的时候会加载这些类文件，把保存这些映射关系的数据读到内存里(保存在map里)
- 进行路由跳转的时候，通过build()方法传入要到达页面的路由地址，ARouter会通过它自己存储的路由表找到路由地址对应的Activity.class
- 然后new Intent方法，如果有调用ARouter的withString()方法，就会调用intent.putExtra(String name, String value)方法添加参数
- 最后调用navigation()方法，它的内部会调用startActivity(intent)进行跳转

### ARouter怎么实现页面拦截

先说一个拦截器的案例，用作页面跳转时候检验是否登录，然后判断跳转到登录页面还是目标页面：

```java
@Interceptor(name = "login", priority = 6)
public class LoginInterceptorImpl implements IInterceptor { 
  @Override  
  public void process(Postcard postcard, InterceptorCallback callback) {  
    String path = postcard.getPath();      
    boolean isLogin = SPUtils.getInstance().getBoolean(ConfigConstants.SP_IS_LOGIN, false);
    if (isLogin) {    
      // 如果已经登录不拦截      
      callback.onContinue(postcard);  
    } else {             
      // 如果没有登录，进行拦截   
      callback.onInterrupt(postcard); 
    }  
  }
  @Override 
  public void init(Context context) {      
    LogUtils.v("初始化成功");  
  }
}
//使用
ARouter.getInstance().build(ConfigConstants.SECOND_PATH)                          .withString("msg", "123")                    
.navigation(this,new LoginNavigationCallbackImpl());    
// 第二个参数是路由跳转的回调
// 拦截的回调
public class LoginNavigationCallbackImpl  implements NavigationCallback{ 
  @Override  
  public void onFound(Postcard postcard) {   }   
  
  @Override   
  public void onLost(Postcard postcard) {    }  
  
  @Override       
  public void onArrival(Postcard postcard) {    }   
  @Override    public void onInterrupt(Postcard postcard) {   
    //拦截并跳转到登录页       
    String path = postcard.getPath();   
    Bundle bundle = postcard.getExtras();    
    ARouter.getInstance().build(ConfigConstants.LOGIN_PATH)  
      .with(bundle)              
      .withString(ConfigConstants.PATH, path)    
      .navigation();  
  }
}
```

拦截器实现IInterceptor接口，使用注解@Interceptor，这个拦截器就会自动被注册了，同样是使用APT技术自动生成映射关系类。这里还有一个优先级参数priority，数值越小，就会越先执行。

## ButterKnife

参考文章
<https://blog.csdn.net/xu_1215/article/details/80761862>

butterKnife 使用的是 APT 技术 也就是编译时注解，不同于运行时注解（在运行过程中通过反射动态地获取相关类，方法，参数等信息，效率低耗时等缺点），编译时注解 则是在代码编译过程中对注解进行处理（annotationProcessor技术），通过注解获取相关类，方法，参数等信息，然后在项目中生成代码，运行时调用，其实和直接手写代码一样，没有性能问题，只有编辑时效率问题。

ButterKnife在Bind方法中 获取到DecorView，然后通过Activity和DecorView对象获取xx_ViewBinding类的构造对象，然后通过构造方法反射实例化了这个类 Constructor。

在编写完demo之后，需要先build一下项目，之后可以在build/generated/source/apt/debug/包名/下面找到 对应的xx_ViewBinding类，查看bk 帮我们做的事情。

```java
# xx_ViewBinding.java
@UiThread
public ViewActivity_ViewBinding(ViewActivity target, View source) { 
  this.target = target; 
  target.view = Utils.findRequiredView(source, R.id.view, "field 'view'");
}

# Utils.java
public static View findRequiredView(View source, @IdRes int id, String who) { 
  View view = source.findViewById(id);  
  if (view != null) {    
    return view;  
  }  
  String name = getResourceEntryName(source, id);  
  throw new IllegalStateException("Required view ....")
}
```

通过上述上述代码 可以看到 注解也是帮我们完成了 findviewbyid 的工作。

### butterknife 实现流程

- 扫描Java代码中所有的ButterKnife注解。
- 发现注解， ButterKnifeProcessor会帮你生成一个Java类，名字<类名>$$ViewBinding.java，这个新生成的类实现了Unbinder接口，类中的各个view 声明和添加事件都添加到Map中，遍历每个注解对应通过JavaPoet生成的代码。

### butterknife未来

Gradle插件升级到5.0版本之后ButterKnife将无法再被使用，R文件中的 id将添加final标识符，虽然 jake大神通过生成R2文件的方式，尝试避开版本升级带来的影响。但是随着官方ViewBinding等技术的出现。

### 为什么final修饰的基础类型和String类型不能被反射

我们这里说下 被final修饰的基础类型和String类型为什么不能被反射？

答：由于JVM 内联优化的机制，编译器将指定的函数体插入并取代每一处调用该函数的地方（就是在方法编译前已经进行了赋值），从而节省了每次调用函数带来的额外时间开支。

## 介绍一下你们之前做的项目的架构

这个问题大家就真实回答就好，重点是要说完后提出对自己项目架构的认同或不认同的观点，也就是要有自己的思考和想法。

### MVP,MVVM,MVC 区别

#### MVC

##### 架构介绍

Model：数据模型，比如我们从数据库或者网络获取数据View：视图，也就是我们的xml布局文件Controller：控制器，也就是我们的Activity

##### 模型联系

View --> Controller，也就是反应View的一些用户事件（点击触摸事件）到Activity上。Controller --> Model, 也就是Activity去读写一些我们需要的数据。Controller --> View, 也就是Activity在获取数据之后，将更新内容反映到View上。

这样一个完整的项目架构就出来了，也是我们早期进行开发比较常用的项目架构。

##### 优缺点

这种缺点还是比较明显的，主要表现就是我们的Activity太重了，经常一写就是几百上千行了。造成这种问题的原因就是Controller层和View层的关系太过紧密，也就是Activity中有太多操作View的代码了。

但是！但是！其实Android这种并称不上传统的MVC结构，因为Activity又可以叫View层又可以叫Controller层，所以我觉得这种Android默认的开发结构，其实称不上什么MVC项目架构，因为他本身就是Android一开始默认的开发形式，所有东西都往Activity中丢，然后能封装的封装一下，根本分不出来这些层级。当然这是我个人看法，可以都来讨论下。

#### MVP

##### 架构介绍

之前不就是因为Activity中有操作view，又做Controller工作吗。所以其实MVP架构就是从原来的Activity层把view和Controller区分开，单独抽出来一层Presenter作为原来Controller的职位。然后最后演化成，将View层写成接口的形式，然后Activity去实现View接口，最后在Presenter类中去实现方法。

Model：数据模型，比如我们从数据库或者网络获取数据。View：视图，也就是我们的xml布局文件和Activity。Presenter：主持人，单独的类，只做调度工作。

##### 模型联系

View --> Presenter，反应View的一些用户事件到Presenter上。Presenter --> Model, Presenter去读写操作一些我们需要的数据。Controller --> View, Presenter在获取数据之后，将更新内容反馈给Activity，进行view更新。

##### 优缺点

这种的优点就是确实大大减少了Activity的负担，让Activity主要承担一个更新View的工作，然后把跟Model交互的工作转移给了Presenter，从而由Presenter方来控制和交互Model方以及View方。所以让项目更加明确简单，顺序性思维开发。

缺点也很明显：首先就是代码量大大增加了，每个页面或者说功能点，都要专门写一个Presenter类，并且由于是面向接口编程，需要增加大量接口，会有大量繁琐的回调。其次，由于Presenter里持有了Activity对象，所以可能会导致内存泄漏或者view空指针，这也是需要注意的地方。

#### MVVM

##### 架构介绍

MVVM的特点就是双向绑定，并且有Google官方加持，更新了Jetpack中很多架构组件，比如ViewModel，Livedata，DataBinding等等，所以这个是现在的主流框架和官方推崇的框架。

Model：数据模型，比如我们从数据库或者网络获取数据。View：视图，也就是我们的xml布局文件和Activity。ViewModel：关联层，将Model和View绑定，使他们之间可以相互绑定实时更新

##### 模型联系

View --> ViewModel -->View，双向绑定，数据改动可以反映到界面，界面的修改可以反映到数据。ViewModel --> Model, 操作一些我们需要的数据。

##### 优缺点

优点就是官方大力支持，所以也更新了很多相关库，让MVVM架构更强更好用，而且双向绑定的特点可以让我们省去很多View和Model的交互。也基本解决了上面两个架构的问题。

### 具体说说你理解的MVVM

#### 1）先说说MVVM是怎么解决了其他两个架构所在的缺陷和问题

- 解决了各个层级之间耦合度太高的问题，也就是更好的完成了解耦。MVP层中，Presenter还是会持有View的引用，但是在MVVM中，View和Model进行双向绑定，从而使viewModel基本只需要处理业务逻辑，无需关系界面相关的元素了。
- 解决了代码量太多，或者模式化代码太多的问题。由于双向绑定，所以UI相关的代码就少了很多，这也是代码量少的关键。而这其中起到比较关键的组件就是DataBinding，使所有的UI变动都交给了被观察的数据模型。
- 解决了可能会有的内存泄漏问题。MVVM架构组件中有一个组件是LiveData，它具有生命周期感知能力，可以感知到Activity等的生命周期，所以就可以在其关联的生命周期遭到销毁后自行清理，就大大减少了内存泄漏问题。
- 解决了因为Activity停止而导致的View空指针问题。在MVVM中使用了LiveData，那么在需要更新View的时候，如果观察者的生命周期处于非活跃状态（如返回栈中的 Activity），则它不会接收任何 LiveData 事件。也就是他会保证在界面可见的时候才会进行响应，这样就解决了空指针问题。
- 解决了生命周期管理问题。这主要得益于Lifecycle组件，它使得一些控件可以对生命周期进行观察，就能随时随地进行生命周期事件。

#### 2）再说说响应式编程

响应式编程，说白了就是我先构建好事物之间的关系，然后就可以不用管了。他们之间会因为这层关系而互相驱动。其实也就是我们常说的观察者模式，或者说订阅发布模式。

为什么说这个呢，因为MVVM的本质思想就是类似这种。不管是双向绑定，还是生命周期感知，其实都是一种观察者模式，使所有事物变得可观察，那么我们只需要把这种观察关系给稳定住，那么项目也就稳健了。

#### 3）最后再说说MVVM为什么这么强大？

我个人觉得，MVVM强大不是因为这个架构本身，而是因为这种响应式编程的优势比较大，再加上Google官方的大力支持，出了这么多支持的组件，来维系MVVM架构，其实也是官方想进行项目架构的统一。

优秀的架构思想+官方支持=强大

## Jetpack Architecture(架构组件)

[带你领略Android Jetpack组件的魅力](https://juejin.cn/post/6844903768614518798)

- Data Binding(数据绑定)： 数据绑定库是一种支持库，借助该库，可以使用声明式将布局中的界面组件绑定到应用中的数据源。
- Lifecycles： 方便管理 Activity 和 Fragment 生命周期，帮助开发者书写更轻量、易于维护的代码。
- LiveData：是一个可观察的数据持有者类。与常规observable不同，LiveData是有生命周期感知的。
- Navigation：处理应用内导航所需的一切。
- Paging：帮助开发者一次加载和显示小块数据。按需加载部分数据可减少网络带宽和系统资源的使用。
- Room：Room持久性库在SQLite上提供了一个抽象层，帮助开发者更友好、流畅的访问SQLite数据库。
- ViewModel：以生命周期感知的方式存储和管理与UI相关的数据。
- WorkManager：即使应用程序退出或设备重新启动，也可以轻松地调度预期将要运行的可延迟异步任务。

## ViewModel

### ViewModel是什么，说说你所理解的ViewModel？

ViewModel是MVVM架构的一个层级，用来联系View和model之间的关系。

> ViewModel 类旨在以注重生命周期的方式存储和管理界面相关的数据

官方是这么介绍的，这里面有两个信息：

#### 注重生命周期的方式

由于ViewModel的生命周期是作用于整个Activity的，所以就节省了一些关于状态维护的工作，最明显的就是对于屏幕旋转这种情况，以前对数据进行保存读取，而ViewModel则不需要，他可以自动保留数据。

其次，由于ViewModel在生命周期内会保持局部单例，所以可以更方便Activity的多个Fragment之间通信，因为他们能获取到同一个ViewModel实例，也就是数据状态可以共享了。

#### 存储和管理界面相关的数据

ViewModel层的根本职责，就是负责维护界面上UI的状态，其实就是维护对应的数据，因为数据会最终体现到UI界面上。所以ViewModel层其实就是对界面相关的数据进行管理，存储等操作。

#### ViewModel 为什么被设计出来，解决了什么问题？

在ViewModel组件被设计出来之前，MVVM又是怎么实现ViewModel这一层级的呢？

其实就是自己编写类，然后通过接口，内部依赖实现View和数据的双向绑定。所以Google出这个ViewModel组件，无非就是为了规范MVVM架构的实现，并尽量让ViewModel这一层级只触及到业务代码，不去关心VIew层级的引用等。然后配合其他的组件，包括livedata，databindingrang等让MVVM架构更加完善，规范，健硕。

#### 解决了什么问题呢？

其实上面已经说过一些了，比如：

- 1）不会因为屏幕旋转而销毁，减少了维护状态的工作
- 2）由于在作用域内单一实例的特性，使得多个fragment之间可以方便通信，并且维护同一个数据状态。
- 3）完善了MVVM架构，使得解耦更加纯粹。

### 说说ViewModel原理

#### 首先说说是怎么保存生命周期

ViewModel2.0之前呢，其实原理是在Activity上add一个HolderFragment，然后设置setRetainInstance(true)方法就能让这个Fragment在Activity重建时存活下来，也就保证了ViewModel的状态不会随Activity的状态所改变。

2.0之后，其实是用到了Activity的onRetainNonConfigurationInstance()和getLastNonConfigurationInstance()这两个方法，相当于在横竖屏切的时候会保存ViewModel的实例，然后恢复，所以也就保证了ViewModel的数据。

#### 再说说怎么保证作用域内唯一实例

首先，ViewModel的实例是通过反射获取的，反射的时候带上application的上下文，这样就保证了不会持有Activity或者Fragment等View的引用。然后实例创建出来会保存到一个ViewModelStore容器里面，其实也就是一个集合类，这个ViewModelStore 类其实就是保存在界面上的那个实例，而我们的ViewModel就是里面的一个集合类的子元素。

所以我们每次获取的时候，首先看看这个集合里面有无我们的ViewModel，如果没有就去实例化，如果有就直接拿到实例使用，这样就保证了唯一实例。最后在界面销毁的时候，会去执行ViewModelStore的clear方法，去清除集合里面的ViewModel数据。一小段代码说明下：

```java
public <T extends ViewModel> T get(Class<T> modelClass) { 
  // 先从ViewModelStore容器中去找是否存在ViewModel的实例    
  ViewModel viewModel = mViewModelStore.get(key);   
  // 若ViewModel已经存在，就直接返回   
  if (modelClass.isInstance(viewModel)) {  
    return (T) viewModel;     
  }     
  // 若不存在，再通过反射的方式实例化ViewModel，并存储进ViewModelStore    
  viewModel = modelClass.getConstructor(Application.class).newInstance(mApplication); 
  mViewModelStore.put(key, viewModel);    
  return (T) viewModel; 
}

public class ViewModelStore { 
  private final HashMap<String, ViewModel> mMap = new HashMap<>(); 
  
  public final void clear() {     
    for (ViewModel vm : mMap.values()) {       
      vm.onCleared();      
    }
    mMap.clear();  
  }
}

@Override
protected void onDestroy() {    
  super.onDestroy();  
  if (mViewModelStore != null && !isChangingConfigurations()) {     
    mViewModelStore.clear();  
  }
}
```

### ViewModel怎么实现自动处理生命周期？

为什么在旋转屏幕后不会丢失状态？为什么ViewModel可以跟随Activity/Fragment的生命周期而又不会造成内存泄漏呢？

这三个问题很类似，都是关于生命周期的问题，其实也就是问为什么ViewModel能管理生命周期，并且不会因为重建等情况造成影响。

#### ViewModel2.0之前

利用一个无view 的HolderFragment来维持它的生命周期，我们知道ViewModel实例是存储到一个ViewModelStore容器里的，那么这个空的fragment就可以用来管理这个容器，只要Activity处于活动状态，HolderFragment也就不会被销毁，就保证了ViewModel的生命周期。

而且设置setRetainInstance(true)方法可以保证configchange时的生命周期不被改变，让这个Fragment在Activity重建时存活下来。

总结来说就是用一个空的fragment来管理维护ViewModelStore，然后对应的activity销毁的时候就去把viewmodel的映射删除。就让ViewModel的生命周期保持和Activity一样了。这也是很多三方库用到的巧妙方法，比如Glide，也是建立空的Fragment来管理。

#### 2.0之后，有了androidx支持

其实是用到了Activity的一个子类ComponentActivity，然后重写了onRetainNonConfigurationInstance()方法保存ViewModelStore，并在需要的时候，也就是重建的Activity中去通过getLastNonConfigurationInstance()方法获取到ViewModelStore实例。这样也就保证了ViewModelStore中的ViewModel不会随Activity的重建而改变。

同时由于实现了LifecycleOwner接口，所以能利用Lifecycles组件组件感知每个页面的生命周期，就可以通过它来订阅当Activity销毁时，且不是因为配置导致的destory情况下，去清除ViewModel，也就是调用ViewModelStore的clear方法。

```java
getLifecycle().addObserver(new LifecycleEventObserver() {   
  @Override       
  public void onStateChanged(@NonNull LifecycleOwner source,            
                             @NonNull Lifecycle.Event event) {   
    if (event == Lifecycle.Event.ON_DESTROY) { 
      // 判断是否因为配置更改导致的destroy         
      if (!isChangingConfigurations()) {    
        getViewModelStore().clear();         
      }
    }
  }
});
```

这里的onRetainNonConfigurationInstance方法再说下，是会在Activity因为配置改变而被销毁时被调用，跟onSaveInstanceState方法调用时机比较相像，不同的是onSaveInstanceState保存的是Bundle，Bundle是有类型限制和大小限制的，而且需要在主线程进行序列号。而onRetainNonConfigurationInstance方法都没有限制，所以更倾向于用它。

所以，到这里，第三个问题应该也可以回答了，2.0之前呢，都是通过他们创建了一个空的fragment，然后跟随这个fragment的生命周期。

2.0之后呢，是因为不管是Activity或者Fragment，都实现了LifecycleOwner接口，所以ViewModel是可以通过Lifecycles感知到他们的生命周期，从而进行实例管理的。

## LiveData

> LiveData 是一种可观察的数据存储器类。与常规的可观察类不同，LiveData 具有生命周期感知能力，意指它遵循其他应用组件（如 Activity、Fragment 或 Service）的生命周期。这种感知能力可确保 LiveData 仅更新处于活跃生命周期状态的应用组件观察者。

### LiveData 是什么？

官方介绍如下，其实说的比较清楚了，主要作用在两点：

- 数据存储器类。也就是一个用来存储数据的类。
- 可观察。这个数据存储类是可以观察的，也就是比一般的数据存储类多了这么一个功能，对于数据的变动能进行响应。

主要思想就是用到了观察者模式思想，让观察者和被观察者解耦，同时还能感知到数据的变化，所以一般被用到ViewModel中，ViewModel负责触发数据的更新，更新会通知到LiveData，然后LiveData再通知活跃状态的观察者。

```kotlin
var liveData = MutableLiveData<String>()
liveData.observe(this, object : Observer<String> {   
    override fun onChanged(t: String?) {    }
})
liveData.setVaile("xixi")
//子线程调用liveData.postValue("test")
```

### LiveData 为什么被设计出来，解决了什么问题？

LiveData作为一种观察者模式设计思想，常常被和Rxjava一起比较，观察者模式的最大好处就是事件发射的上游 和 接收事件的下游 互不干涉，大幅降低了互相持有的依赖关系所带来的强耦合性。

其次，LiveData还能无缝衔接到MVVM架构中，主要体现在其可以感知到Activity等生命周期，这样就带来了很多好处：

- 不会发生内存泄漏 观察者会绑定到 Lifecycle对象，并在其关联的生命周期遭到销毁后进行自我清理。
- 不会因 Activity 停止而导致崩溃 如果观察者的生命周期处于非活跃状态（如返回栈中的 Activity），则它不会接收任何 LiveData 事件。
- 自动判断生命周期并回调方法 如果观察者的生命周期处于 STARTED 或 RESUMED状态，则 LiveData 会认为该观察者处于活跃状态，就会调用onActive方法，否则，如果 LiveData 对象没有任何活跃观察者时，会调用 onInactive()方法。

### 说说LiveData原理

说到原理，其实就是两个方法：

#### 订阅方法

也就是observe方法。通过该方法把订阅者和被观察者关联起来，形成观察者模式。

简单看看源码：

```java
    @MainThread
    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
        assertMainThread("observe");
        //...
        LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && !existing.isAttachedTo(owner)) {
            throw new IllegalArgumentException("Cannot add the same observer" + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        owner.getLifecycle().addObserver(wrapper);
    }

    public V putIfAbsent(@NonNull K key, @NonNull V v) {
        Entry<K, V> entry = get(key);
        if (entry != null) {
            return entry.mValue;
        }
        put(key, v);
        return null;
    }
```

这里putIfAbsent方法是讲生命周期相关的wrapper和观察者observer作为key和value存到了mObservers中。

#### 回调方法

也就是onChanged方法。通过改变存储值，来通知到观察者也就是调用onChanged方法。从改变存储值方法setValue看起：

```java
    @MainThread
    protected void setValue(T value) {
        assertMainThread("setValue");
        mVersion++;
        mData = value;
        dispatchingValue(null);
    }

    private void dispatchingValue(@Nullable ObserverWrapper initiator) {
        //...
        do {
            mDispatchInvalidated = false;
            if (initiator != null) {
                considerNotify(initiator);
                initiator = null;
            } else {
                for (Iterator<Map.Entry<Observer<T>, ObserverWrapper>> iterator = mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                    considerNotify(iterator.next().getValue());
                    if (mDispatchInvalidated) {
                        break;
                    }
                }
            }
        } while (mDispatchInvalidated);
        mDispatchingValue = false;
    }

    private void considerNotify(ObserverWrapper observer) {
        if (!observer.mActive) {
            return;
        }
        // Check latest state b4 dispatch. Maybe it changed state but we didn't get the event yet.    
        //    
        // we still first check observer.active to keep it as the entrance for events. So even if    
        // the observer moved to an active state, if we've not received that event, we better not    
        // notify for a more predictable notification order.    
        if (!observer.shouldBeActive()) {
            observer.activeStateChanged(false);
            return;
        }
        if (observer.mLastVersion >= mVersion) {
            return;
        }
        observer.mLastVersion = mVersion;
        //noinspection unchecked    
        observer.mObserver.onChanged((T) mData);
    }
```

这一套下来逻辑还是比较简单的，遍历刚才的map——mObservers，然后找到观察者observer，如果观察者不在活跃状态（活跃状态，也就是可见状态，处于 STARTED 或 RESUMED状态），则直接返回，不去通知。否则正常通知到观察者的onChanged方法。

当然，如果想任何时候都能监听到，都能获取回调，调用observeForever方法即可。

## MMKV
[一篇文章搞定《MMKV原理解析》](https://blog.csdn.net/weixin_45112340/article/details/131798459)

### 什么是持久化存储呢
简单来说，持久化存储就是将数据永久保存在磁盘或其他非易失性存储介质上，以便在程序重新启动或设备重启后可以重新加载和使用。

在移动应用开发中，持久化存储非常重要，因为移动设备的特点是资源有限，且具有临时性。通过持久化存储，应用可以将数据持久保存，即使应用关闭或设备重启，数据也不会丢失。常见的使用场景包括存储用户配置信息、用户登录信息、应用状态、用户生成的内容等。

### 常见持久化存储
- **文件存储**：使用IO传输的方式将数据写入文件中。可以借助FileInputStream和FileOutputStream类来进行读写操作。这种方式适用于存储较小的数据量，例如配置文件和日志等。
- **Shared Preferences**：使用键值对的方式将数据存储到SharedPreferences文件中。SharedPreferences文件是一个XML文件，可以在应用程序间共享。这种方式通常用于存储应用程序的配置参数和用户偏好设置。
- **SQLite数据库**：使用SQLite数据库引擎来存储和管理结构化数据。SQLite是一个嵌入式关系型数据库，可以提供高效的数据存储和查询功能。开发者可以借助Android提供的SQLiteOpenHelper类来创建和管理数据库。
- **DataStore**：DataStore是Android Jetpack组件库中的一种持久化数据存储解决方案，从Android Jetpack 1.0.0-alpha06版本开始引入。它提供了一种类型安全、支持协程的方式来存储和读取数据，并且可以与LiveData和Flow配合使用。DataStore支持两种存储格式：Proto DataStore和Preferences DataStore。
- **MMKV**：MMKV是基于底层的mmap文件映射技术实现的，具有快速的读写速度和较低的内存占用。MMKV适用于在Android应用中存储较大量的键值对数据。

### MMKV优点

- **高性能**：MMKV使用了一些技术手段，如mmap文件映射和跨进程通信的共享内存，以实现更高效的数据存取操作。MMKV的性能比SharedPreferences快数十倍，尤其在读写大量数据时效果更加明显。
- **小存储体积**：这是因为MMKV使用了一种更高效的序列化算法，并且将数据存储在二进制文件中，避免了XML解析和序列化的开销。相同数据量情况下，MMKV的存储体积可以减少50%以上。
- **跨进程共享**：MMKV支持多进程间的数据共享，这对于需要在多个进程之间传递数据的应用程序非常有用。MMKV通过共享内存和文件锁定机制来确保跨进程读写数据的一致性和安全性。
- **API简单易用**：MMKV提供了简洁、易用的API，使数据存取变得更加方便。您可以使用各种数据类型作为键值，而无需进行烦琐的类型转换。同时，MMKV还提供了诸如数据压缩和加密等额外功能，方便开发者进行更多的数据处理。

### MMKV劣势
凡事不是绝对好的！！

1. 他是同步去存储的，写入大的字符串不如SP、Datastore。 为什么说不如呢？
他的存储速度确实快，但是大家注意了，他是同步啊！！！ 我们关注APP的卡顿时间指的是什么呢？
是主线程的卡顿时间，所以就是他时间再短，也是阻塞了主线程的时间的。但是SP和Datastore是子线程写入的。
别被他的时间骗了哦！！！

2. 还有一个缺点：他是磁盘写入啊兄弟们。 有没有知道会出现什么问题的？
哎呦喂，答对了。断电啊、意外关机啊。那么没写完数据怎么办啊？
能咋办，丢失了呗。没办法。（可以在上层做备份缓存）
但是SP和Datastore是有备份的。

### SharedPreferences解析（为什么不用SP）

![20231011-205518.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231011-205518.png)

#### SharedPreferences的过程
##### 启动App初始化：利用IO读取XML文件（SP数据存放在XML中）
xml：（存储文件通常位于/data/data/包名/shared_prefs/目录下。）

```xml
<xml version='1.0' encoding='utf-8' standalone='yes'>
<map>
    <string name="name">user_name</string>
    <int name="age" value="28"/>
    <boolean name="xiaomeng" value="false"/> 
<map>
```

##### 通过反序列化全量的添加到内存map中

核心代码：SharedPreferencesImpl.java

```java
Map map = null;
StructStat stat = null;
try {
    stat = Os.stat(mFile.getPath());
    if (mFile.canRead()) {
        BufferedInputStream str = null;
        try {
            str = new BufferedInputStream(
                    new FileInputStream(mFile), 16*1024);
            //将xml文件转成map
            map = XmlUtils.readMapXml(str);
        } catch (Exception e) {
            Log.w(TAG, "Cannot read " + mFile.getAbsolutePath(), e);
        } finally {
            IoUtils.closeQuietly(str);
        }
    }
} catch (ErrnoException e) {
    /* ignore */
}
```

##### 上面全量加入到map中后，通过Map.get(key)0获取Value

源码如下：SharedPreferencesImpl.java

```java
public String getString(String key, @Nullable String defValue) {
    synchronized (mLock) {
        //阻塞等待sp将xml读取到内存后再get
        awaitLoadedLocked();
        String v = (String)mMap.get(key);
        //如果value为空返回默认值
        return v != null ? v : defValue;
    }
}

```

#### SharedPreferences存在的问题

##### sp.get的阻塞问题（会出现ANR）

这是个什么问题呢？我们通过上面的过程知道，sp是通过IO获取文件数据是一个耗时的操作，所以需要在子线程操作。
那么当我们数据量变大时，还没有完成XML解析全量加入Map时。因为子线程初始化，这时候我们去获取，自然是获取不到的，拿SP是怎么防止你拿不到呢？
源码如下：

```java
@Nullable
public String getString(String key, @Nullable String defValue) {
    synchronized (mLock) {
        //阻塞等待sp将xml读取到内存后再get
        awaitLoadedLocked();
        String v = (String)mMap.get(key);
        //如果value为空返回默认值
        return v != null ? v : defValue;
    }
}

private void awaitLoadedLocked() {
    ...
    // sp读取完成后会把mLoaded设置为true
    while (!mLoaded) {
        try {
            mLock.wait();
        } catch (InterruptedException unused) {
        }
    }
}

```
awaitLoadedLocked()这个操作会阻塞，当xml文件读取完成后才会释放锁mLock.notifyAll();
这时候阻塞着，如果我们在主线程中调用了呢？ 岂不是就一起在等待，也就是阻塞了。那不就ANR了吗。

##### 全量的更新问题

这个是个什么问题呢？可以看到每次更新时，会把map中的数据，从内存中全量的更新到文件中。
兄弟们全量更新属于什么啊？那不是相当于每次都重新保存吗。
还是个IO文件的操作，当文件越来越大，全量保存的代价也就越来越大了。

##### commit和apply提交内容都会ANR

- commit()方法，会进行同步写，一定存在耗时，不能直接在主线程调用。

他会一样把writeToFile任务加入主线程队列中，如果太大，导致全量的更新过慢就会ANR

源码如下：

```java
public boolean commit() {
            // 开始排队写
            SharedPreferencesImpl.this.enqueueDiskWrite(
                mcr, null /* sync write on this thread okay */);
            try {
                // 等待同步写的结果
                mcr.writtenToDiskLatch.await();
            } catch (InterruptedException e) {
                return false;
            } finally {
            }
            notifyListeners(mcr);
            return mcr.writeToDiskResult;
        }

```

- 大家都知道apply方法是异步写，但是也可能造成ANR的问题。下面我们来看apply方法的源码。

```java
public void apply() {
            // 先将更新写入内存缓存
            final MemoryCommitResult mcr = commitToMemory();
            // 创建一个awaitCommit的runnable，加入到QueuedWork中
            final Runnable awaitCommit = new Runnable() {
                    @Override
                    public void run() {
                        try {
                            // 等待写入完成
                            mcr.writtenToDiskLatch.await();
                        } catch (InterruptedException ignored) {
                        }
                    }
                };
            // 将awaitCommit加入到QueuedWork中
            QueuedWork.addFinisher(awaitCommit);
            Runnable postWriteRunnable = new Runnable() {
                    @Override
                    public void run() {
                        awaitCommit.run();
                        QueuedWork.removeFinisher(awaitCommit);
                    }
                };
            // 真正执行sp持久化操作，异步执行
            SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);
            // 虽然还没写入文件，但是内存缓存已经更新了，而listener通常都持有相同的sharedPreference对象，所以可以使用内存缓存中的数据
            notifyListeners(mcr);
        }

```
可以看到这里确实是在子线程进行的写入操作，但是为什么说apply也会引起ANR呢？
因为在Activity和Service的一些生命周期方法里，都会调用QueuedWork.waitToFinish()方法，这个方法会等待所有子线程写入完成，才会继续进行。主线程等子线程，很容易产生ANR问题。

```java
public static void waitToFinish() {
       Runnable toFinish;
       //等待所有的任务执行完成
       while ((toFinish = sPendingWorkFinishers.poll()) != null) {
           toFinish.run();
       }
   }

```
所以可以看到因为apply引起的ANR日志中会有：ActivityThread的信息出现。

```log
at java.util.concurrent.CountDownLatch.await(CountDownLatch.java:202)
at android.app.SharedPreferencesImpl$EditorImpl$1.run(SharedPreferencesImpl.java:364)
at android.app.QueuedWork.waitToFinish(QueuedWork.java:88)
at android.app.ActivityThread.handleStopActivity(ActivityThread.java:3246)
at android.app.ActivityThread.access$1100(ActivityThread.java:141)
at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1239)

```

总结：apply方法虽然是在异步线程写入，但是由于Activity和Service的生命周期会等待所有SharedPreference的写入完成，所以可能引起卡顿和ANR问题。

但是为什么会去等待呢？

当然要等待了：举个例子

如果在Activity的onPause方法中调用apply方法保存数据，在异步线程中的写入操作还没有完成时，Activity被销毁了，那么这部分数据就会丢失。
为了避免这种情况，Android系统在Activity和Service的生命周期方法中会等待所有子线程写入操作完成，然后再继续执行下面的代码。这样可以确保数据的一致性和完整性，避免数据丢失或不一致的问题。

##### 不支持多进程

### MMKV原理解析

首先MMKV是继承SharedPreferences上层也是同样的在map中进行操作的。
那MMKV是怎么解决了上面SharedPreferences的问题的呢？
- 读写方式：I/O
- 数据格式：xml
- 写入方式：全量更新

那就围绕着SharedPreferences这三个问题来了解MMKV的原理吧。

#### mmap

MMKV的核心基于 `mmap`，之所以他比sp要快很多，也是`mmap`的特性使然

##### mmap基础概念

> `mmap`是一种内存映射文件的方法，即将一个文件或者其它对象映射到进程的地址空间，实现文件磁盘地址和进程虚拟地址空间中一段虚拟地址的一一对映关系。实现这样的映射关系后，进程就可以采用指针的方式读写操作这一段内存，而系统会自动回写脏页面到对应的文件磁盘上，即完成了对文件的操作而不必再调用`read`，`write`等系统调用函数。相反，内核空间对这段区域的修改也直接反映用户空间，从而可以实现不同进程间的文件共享。

![20231011-211616.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231011-211616.png)

##### mmap零拷贝

MMKV的核心在于mmap，所以他的优点就是借用了mmap的优点。（FileChannel是典型的利用零拷贝）

首先了解一些基础知识：

我们经常说的内存是什么？

在工作中我们口中的内存都是虚拟内存：虚拟内存又分为两块，用户空间和内核空间。

- 用户空间是用户程序代码运行的地方（我们APP运行的内存）
- 内核空间是内核代码运行的地方，由所有进程共享、进程间有相互隔离的一个共享空间。

1. 首先看一下传统的IO是怎么操作内存的呢？

用户空间->内核空间（CPU copy）->虚拟内存（DMA copy：负责将数据于内核传输的）->物理内存（内存映射，虚拟内存和物理内存进行映射）

![20231011-213020.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231011-213020.png)

2. 那mmap是怎么操作内存的呢？

- 首先零拷贝只是没有CPU拷贝的、DMA copy还是有的。
- 用户空间（直接映射到）->虚拟内存（DMA copy）->物理内存

![20231011-213232.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231011-213232.png)

因为没有CPU的拷贝，所以效率是要提升很多的。

- 实现这样的映射关系后，进程就可以采用指针的方式读写操作这一段内存，而系统会自动回写脏页面到对应的文件磁盘上，即完成了对文件的操作而不必再调用read，write等系统调用函数。
- 相当于操作内存就等于操作文件。
- 相反，内核空间对这段区域的修改也直接反映用户空间，从而可以实现不同进程间的文件共享。

3. 具体是怎样映射过去的呢？先看看map的函数吧，太深了就不说了（源码在C层）

```c++
void *mmap(void *start, size_t length, int prot, int flags, int fd, off_t offset);
```

- start：映射区的开始地址。设置null即可。
- length：映射区的长度。传入文件对齐后的大小m_size。
- prot：期望的内存保护标志，不能与文件的打开模式冲突。设置可读可写。
- flags：指定映射对象的类型，映射选项和映射页是否可以共享。设置MAP_SHARED表示可进程共享，MMKV之所以可以实现跨进程使用，这里是关键。
- fd：有效的文件描述词。用上面所打开的m_fd。
- off_toffset：被映射对象内容的起点。从头开始，比较好理解

##### MMKV数据存储.defalut文件

上面也讲到了SharedPreferences是以xml文件存放的。 而MMKV是以.defalut保存到指定的目录中的。

下面看一下.defalut中什么样子的：

.defalut（这是一个二进制文件，每个字节代表一个8位的二进制数，可以表示256个不同的值（从0到255））
我们用16进制来打开：

![20231011-215029.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231011-215029.png)

这。。。。。这是什么呢？
我带大家来解读一下：先看下面的图对照着图来说。

![](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231011-215053.png)

首先0E（十六进制）：总长度为14（十进制）

其次07（十六进制）：Key长度是7（十进制）

往后数7个：61 62 63 64 65 66 67(十六进制) ：Key为abcdefg

其次01（十六进制）：Value的长度是1（十六进制）

往后数1个：01：Value为1

那么就是<abcdefg，1>

以此类推第二个键值对为<x，1>

**问题一：那么长度超过255也就是超过一个字节了呢？**

答：他是一个变长编码来存储的，可以变长到1-5个字节。这个也叫protocol buffers数据存储格式（也是一种序列化与反序列化的数据格式和Json和XML一样，但是更小）

**问题二：这么不易读，为什么还要用呢？**

第一点肯定是因为占用空间小，数据更紧凑，都是有效数据。
第二点是他可以增量更新，下面看看他的增量更新

##### 增量更新

增量更新贼简单。

直接增量的写进去就行。为什么呢？

举个例子：

我们要更改那个<x，1>：01，01，78，01为<x，2>：01，01，78，02

我们只需要在后面直接加上就可以。

![20231011-215647.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231011-215647.png)

那为什么直接加上就可进行修改呢？

大家想想我们读取出来数据是加入到哪里的？

是HashMap啊大哥们，HashMap有什么特性啊。重复的Key就覆盖了啊。那不久相对于更新了吗！！！！

不得不说作者真聪明。

那么有同学问了

**问题一：那文件不断追加，文件过大了怎么办啊？**

嘿嘿：MMKV除了增量更新还有全量更新呢。

他是利用全量更新来解决这个问题的。看看怎么解决的

1. 太多重复的Key导致
- 去重：利用Map去重后进行全量的更新进去
2. 确实需要保存更多的数据
- 扩容：先扩容->再全量的加入新扩容的内容中

##### MMKV跨进程

MMKV是一种跨进程键值存储库，其原理是利用Shared Memory映射来实现跨进程数据共享。

下面按照步骤详细说明MMKV的原理：
1. 创建共享内存映射：在进程A中，调用MMKV的初始化方法时，会创建一个共享内存映射区域，该区域会被映射到进程A的地址空间中。
2. 将数据写入共享内存：在进程A中，当调用MMKV的put方法时，需要将要存储的数据序列化，并写入到共享内存中。MMKV使用B+Tree数据结构来组织数据，因为B+Tree适合高效地进行数据的查询和修改。
3. 通知其他进程有数据更新：在进程A中，当数据写入共享内存后，会发送一个通知给其他进程，告知它们有新数据更新。进程间通信的方式可以是共享内存区域上的一个信号量或者一个读写锁。
4. 共享内存读取数据：当进程B收到进程A发出的数据更新通知后，会通过共享内存映射将该共享内存区域映射到进程B的地址空间中。
5. 从共享内存中读取数据：在进程B中，可以直接从共享内存中读取数据。MMKV通过B+Tree的索引结构，可以快速地定位数据，并进行反序列化操作，将数据转换为可用的格式。
6. 更新数据时的锁机制：在多进程环境下，多个进程可能会同时尝试修改同一个MMKV实例中的数据。为了保证数据一致性，MMKV使用了一种基于CAS（Compare and Swap）机制的自旋锁来实现数据的并发访问控制。这样可以确保每次修改数据时，只有一个进程可以成功地将数据写入共享内存。

## 注解是什么？有哪些元注解

注解，在我看来它是一种信息描述，不影响代码执行，但是可以用来配置一些代码或者功能。

常见的注解比如@Override,代表重写方法，看看它是怎么生成的：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

可以看到Override被@interface所修饰，代表注解，同时上方还有两个注解@Target和@Retention，这种修饰注解的注解叫做元注解，很好理解吧，就是最基本的注解呗。java中一共有四个元注解：

- @Target：表示注解对象的作用范围。
- @Retention：表示注解保留的生命周期
- @Inherited：表示注解类型能被类自动继承。
- @Documented：表示含有该注解类型的元素(带有注释的)会通过javadoc或类似工具进行文档化。

### 具体说下这几个元注解都是怎么用的

#### @Target

target，表示注解对象的作用范围，比如Override注解所标示的就是ElementType.METHOD，即所作用的范围是方法范围，也就是只能在方法头上加这个注解。另外还有以下几个修饰范围参数：

- TYPE：类、接口、枚举、注解类型。
- FIELD：类成员（构造方法、方法、成员变量）。
- METHOD：方法。
- PARAMETER：参数。
- CONSTRUCTOR：构造器。
- LOCAL_VARIABLE：局部变量。
- ANNOTATION_TYPE：注解。
- PACKAGE：包声明。
- TYPE_PARAMETER：类型参数。
- TYPE_USE：类型使用声明。

比如ANNOTATION_TYPE就是表示该注解的作用范围就是注解，哈哈，有点绕吧，看看Target注解的代码：

```java
@Documented          
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {    
/**     
 * @return an array of the kinds of elements an annotation type     
 * can be applied to     
 */    
 ElementType[] value();
}
```

带了一个ElementType类型的参数，也就是上面说到的作用范围参数，另外还被Target注解修饰了，传的参数就是ANNOTATION_TYPE，也就是我注解我自己，我设置我自己的作用范围是注解。大家自己绕一下。

#### @Retention

表示注解保留的生命周期，或者说表示该注解所保留的时长，主要有以下几个可选参数：

- SOURCE：仅存在Java源文件，经过编译器后便丢弃相应的注解。适用于一些检查性的操作，比如@Override。
- CLASS：编译class文件时生效，存在Java源文件，以及经编译器后生成的Class字节码文件，但在运行时VM不再保留注释。这个也是默认的参数。适用于在编译时进行一些预处理操作，比如ButterKnife的@BindView，可以在编译时生成一些辅助的代码或者完成一些功能。
- RUNTIME：存在源文件、编译生成的Class字节码文件，以及保留在运行时VM中，可通过反射性地读取注解。适用于一些需要运行时动态获取注解信息，类似反射获取注解等。

#### @Inherited

表示注解类型能被类自动继承。这里需要注意两点：

- 类。也就是说只有在类集成关系中，子类才会集成父类使用的注解中被@Inherited所修饰的那个注解。其他的接口集成关系，类实现接口关系中，都不会存在自动继承注解。
- 自动继承。也就是说如果父类有@Inherited所修饰的那个注解，那么子类不需要去写这个注解，就会自动有了这个注解。

还是看个例子：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Inherited
public @interface MyInheritedAnnotation { 
  //注解1，有Inherited注解修饰
}
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAnnotation { 
  //注解2，没有Inherited注解修饰
}
@MyInheritedAnnotation
@MyAnnotation
public class BaseClass { 
  //父类，有以上两个注解
}
public class ExtendClass extends BaseClass  { 
  //子类会继承父类的MyInheritedAnnotation注解， //而不会继承MyAnnotation注解
}
```

#### @Documented

表示拥有该注解的元素可通过javadoc此类的工具进行文档化，也就是说生成JavaAPI文档的时候会被写进文档中。

### 注解可以用来做什么

主要有以下几个用处：

- 降低项目的耦合度。
- 自动完成一些规律性的代码。
- 自动生成java代码，减轻开发者的工作量。

## 线程与协程

可借鉴 郭霖发表的 —— [一看就会！协程原来是这样啊~](https://mp.weixin.qq.com/s/nXfweTaOCpm6Bj34rW-wLA)

### 线程

- 线程是操作系统级别的概念
- 我们开发者通过编程语言(Thread.java)创建的线程，本质还是操作系统内核线程的映射
- JVM 中的线程与内核线程的存在映射关系，有“一对一”，“一对多”，“M对N”。JVM在不同操作系统中的具体实现会有差别，“一对一”是主流
  一般情况下，我们说的线程，都是内核线程，线程之间的切换，调度，都由操作系统负责
- 线程也会消耗操作系统资源，但比进程轻量得多
- 线程，是抢占式的，它们之间能共享内存资源，进程不行
- 线程共享资源导致了多线程同步问题
- 有的编程语言会自己实现一套线程库，从而能在一个内核线程中实现多线程效果，早期 JVM 的“绿色线程” 就是这么做的，这种线程被称为“用户线程”

有人会将线程比喻成：轻量级的进程

#### 线程池的运行原理

[解决Java线程池 面试所有问题](https://blog.csdn.net/qq_27828675/article/details/115357347)

- **核心线程（corePool）**：线程池最终执行任务的角色肯定还是线程，同时我们也会限制线程的数量，所以我们可以这样理解核心线程，**有新任务提交时，首先检查核心线程数，如果核心线程都在工作，而且数量也已经达到最大核心线程数，则不会继续新建核心线程，而会将任务放入等待队列。**

- **等待队列 (workQueue)**：等待队列用于存储**当核心线程都在忙时，继续新增的任务，核心线程在执行完当前任务后，也会去等待队列拉取任务继续执行**，这个队列一般是一个线程安全的阻塞队列，它的容量也可以由开发者根据业务来定制。

- **非核心线程**：**当等待队列满了，如果当前线程数没有超过最大线程数，则会新建线程执行任务**

- **线程活动保持时间 (keepAliveTime)**：线程空闲下来之后，保持存活的持续时间，超过这个时间还没有任务执行，该工作线程结束。

- **饱和策略 (RejectedExecutionHandler)**：当等待队列已满，线程数也达到最大线程数时，线程池会根据饱和策略来执行后续操作，默认的策略是抛弃要加入的任务。

##### 核心线程和非核心线程到底有什么区别呢？

说出来你可能不信，本质上它们没有什么区别，创建出来的线程也根本没有标识去区分它们是核心还是非核心的，线程池只会去判断已有的线程数（包括核心和非核心）去跟核心线程数和最大线程数比较，来决定下一步的策略。

##### Java线程池中，当核心线程数都在执行了且任务队列已满，线程数也达到了最大线程数时，继续提交任务会发生什么问题？

如果继续往线程池中提交任务，线程池的行为取决于线程池的饱和策略（RejectedExecutionHandler）。饱和策略定义了当无法将任务提交到线程池时采取的行动。

- **AbortPolicy（默认策略）**：如果线程池队列满了且线程池中的线程数达到最大线程数，新提交的任务将被拒绝，并抛出RejectedExecutionException异常。
```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    corePoolSize,
    maximumPoolSize,
    keepAliveTime,
    unit,
    workQueue,
    Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.AbortPolicy()
);
```

- **CallerRunsPolicy**：如果线程池队列满了且线程池中的线程数达到最大线程数，新提交的任务会被当前线程执行，即由提交任务的线程执行该任务。

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    corePoolSize,
    maximumPoolSize,
    keepAliveTime,
    unit,
    workQueue,
    Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.CallerRunsPolicy()
);
```

- **DiscardPolicy**：如果线程池队列满了且线程池中的线程数达到最大线程数，新提交的任务将被丢弃，不会有任何提示或错误。

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    corePoolSize,
    maximumPoolSize,
    keepAliveTime,
    unit,
    workQueue,
    Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.DiscardPolicy()
);
```

- **DiscardOldestPolicy**：如果线程池队列满了且线程池中的线程数达到最大线程数，会丢弃队列中最旧的未处理任务，然后尝试重新提交新任务。

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    corePoolSize,
    maximumPoolSize,
    keepAliveTime,
    unit,
    workQueue,
    Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.DiscardOldestPolicy()
);
```

##### 基础运行流程图

![20231012-175249.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2023/images/20231012-175249.png)

#### 线程池和CPU核心数的关系

[线程池的核心线程数 设置大小与cpu 的关系](https://blog.csdn.net/weixin_43975771/article/details/113099180)

一般说来，大家认为线程池的大小经验值应该这样设置：（其中N为CPU processors的个数）
1. 如果是CPU密集型应用，则线程池大小设置为N+1(或者是N)，线程的应用场景：主要是复杂算法
2. 如果是IO密集型应用，则线程池大小设置为2N+1(或者是2N)，线程的应用场景：主要是：数据库数据的交互，文件上传下载，网络数据传输等等

+1的原因是：即使当计算密集型的线程偶尔由于缺失故障或者其他原因而暂停时，这个额外的线程也能确保CPU的时钟周期不会被浪费。

##### CPU的核心数
CPU的核心数是指物理上，也就是硬件上存在着几个核心。

比如，双核就是包括2个相对独立的CPU核心单元组，四核就包含4个相对独立的CPU核心单元组

##### CPU的线程数
对于一个CPU，线程数总是大于或等于核心数的。
1. 一个核心最少对应一个线程，但通过超线程技术，一个核心可以对应两个线程，也就是说它可以同时运行两个线程。
2. CPU之所以要增加线程数，是源于多任务处理的需要。线程数越多，越有利于同时运行多个程序，因为线程数等同于在某个瞬间CPU能同时并行处理的任务数。

##### Java线程池最大数，线程数太大会有什么影响？

[java线程池最大线程数_Java项目中，线程池中线程数量太大会有什么影响？](https://blog.csdn.net/weixin_31486289/article/details/114040155)

- 线程栈需要分配内存空间，所有有数量上限
- CPU切换线程涉及到上下文恢复，比较耗时，如果线程非常多且切换频繁（处理IO密集型），这个时间损耗是非常可观的。

对于CPU密集型任务，因为线程中基本不会有阻塞导致让出CPU，只有在时间片用完以后，才可以让出CPU，这种情况发生线程切换的次数要少很多。因此不建议设置太大，netty的建议是设置为2倍的CPU核心数。

对于IO密集型任务，仅仅是指你的IO很可能是未就绪需要阻塞等待的任务。如果你的IO事件已经就绪，随时可读可写，任何时候都不会对线程产生任何阻塞，那实际就是CPU密集型任务了。

###### 为何不设置核心线程数呢？

因为系统上不仅仅只跑Java程序，还有别的进程也会来抢占CPU资源，因此**需要在更多的争抢机会和更少的上下文切换之间取得平衡**。

如果你要处理的是会对线程产生阻塞的IO密集型任务，那要提高吞吐量的话，一般有两种办法

一种是基于IO多路复用+NIO/AIO，这种办法实际是想办法去掉不必要的阻塞，尽量把阻塞型的IO密集任务，转成CPU密集任务，这样你只需要少量线程也可以获得很高的吞吐量。这就是为何select、poll 、epoll、nginx可以用很少的线程可以获得极大吞吐量的原因。

另一种办法就是很简单的扩大线程数了，理论上来说，**只要你的线程数，不会导致明显的上下文切换损耗，而且不会造成内存溢出，线程池就可以设置的足够大**。某大厂的rpc框架设置的默认最大线程数就超过了500，核心线程数也大大超过了CPU核心数。

###### 最大线程数可以远远大于CPU核心线程数吗？

**可以，只要你的线程数不会明显的导致上下文切换损耗，而且不会造成内存溢出，线程池就可以设置的足够大。**

但是最大线程数远远大于CPU核心线程数可能会导致以下问题：

- **资源浪费**：多余的线程可能会占用大量内存和其他系统资源。每个线程都有一些固定的开销，包括内存、线程栈等，过多的线程会导致资源浪费。

- **上下文切换成本**：线程的切换涉及到上下文切换的开销，过多的线程可能导致过多的上下文切换，降低系统性能。

- **竞争和锁定问题**：线程之间的竞争可能会增加，导致锁定和同步问题，进而影响性能。

- **系统过载**：如果创建的线程过多，可能会导致系统过载，因为系统无法有效地管理和调度大量线程。

为了避免这些问题，通常建议根据系统的实际需求和硬件配置来合理设置线程池的参数，包括核心线程数、最大线程数和队列大小。在合理范围内进行调整可以平衡系统的性能和资源利用。

### 协程

Kotlin 协程的核心竞争力在于：它能简化异步并发任务

- Kotlin 协程，不是操作系统级别的概念，无需操作系统支持
- Kotlin 协程，有点像上面提到的“绿色线程”，一个线程上可以运行成千上万个协程
- Kotlin 协程，是用户态的(userlevel)，内核对协程「无感知」
- Kotlin 协程，是协作式的，由开发者管理，不需要操作系统进行调度和切换，也没有抢占式的消耗，因此它更加「高效」
- Kotlin 协程，它底层基于状态机实现，多协程之间共用一个实例，资源开销极小，因此它更加「轻量」
- Kotlin 协程，本质还是运行于线程之上，它通过协程调度器，可以运行到不同的线程上

挂起函数是 Kotlin 协程最关键的内容

#### 挂起函数

挂起函数(Suspending Function)，从字面上理解，就是可以被挂起的函数。suspend 有：挂起，暂停的意思。在这个语境下，也有点暂停的意思。暂停更容易被理解，但挂起更准确。

挂起函数，能被「挂起」，当然也能「恢复」，他们一般是成对出现的。

suspend 的本质，就是 CallBack

```kotlin
suspend fun getUserInfo(): String { 
  withContext(Dispatchers.IO) {  
    delay(1000L)   
  }
  return "BoyCoder"
}
```

#### 说说你对协程的理解

在我看来，协程和线程一样都是用来解决并发任务（异步任务）的方案。所以协程和线程是属于一个层级的概念，但是对于kotlin中的协程，又与广义的协程有所不同。kotlin中的协程其实是对线程的一种封装，或者说是一种线程框架，为了让异步任务更好更方便使用。

#### Java线程池与Kotlin协程线程池区别

[Kotlin协程-与线程池的对比 & 铺平回调 & 协程分发器](https://juejin.cn/post/7205116210077597733)

Java线程池：核心线程+队列+其他线程

首先使用核心线程执行任务，一旦核心线程满了，就把任务加到队列中，内部根据不同的调度实现来判断是否开启其他线程来执行队列的任务。

协程线程池：全局队列+本地队列

先尝试添加到本地队列(尾部调用机制)，再添加到全局队列，协程线程池从队列中找任务（任务偷取机制）执行，PS：内部又一系列的CUP任务与非CUP任务的转换逻辑

**Java线程池与Kotlin协程线程池的区别：**

Java线程池比较开放，可以选择系统不同的线程策略，也可以自定义线程池，不同的组合可以实现不同的效果，没有区分任务是否阻塞的属性。

协程的线程池是专供协程使用，没有那么开放，内部的任务区分是否阻塞的属性，会放到不同的队列中，CoroutineScheduler类中的两个全局队列 globalCpuQueue（存非阻塞的任务） ， globalBlockingQueue（存阻塞的任务）。感觉调度会更加的合理。

suspendCoroutine 与 Dispatchers.IO 的结论：

在协程中善用去除回调的方式，尽量把异步的逻辑同步化，不破坏协程的作用域，同时善用线程调度器，区分CUP任务与非CUP任务。最大化的优化线程池效率。

Dispatchers.IO 一般放入 globalBlockingQueue 执行阻塞型任务（不怎么占用CPU的任务，比如文件、数据库、网络等操作等），Dispatchers.Default 一般放入 globalCpuQueue 非阻塞任务（占用CPU的任务，比如人脸特征提取，图片压缩处理，视频的合成等）

#### 说下协程具体的使用

比如在一个异步任务需要回调到主线程的情况，普通线程需要通过handler切换线程然后进行UI更新等，一旦多个任务需要顺序调用，那更是很不方便，比如以下情况：

```kotlin
//客户端顺序进行三次网络异步请求，并用最终结果更新UI
thread{ 
  iotask1(parameter) { value1 ->
                      iotask1(value1) { value2 -> 
                                       iotask1(value2) { value3 -> 
                                                        runOnUiThread{   
                                                          updateUI(value3) 
                                                        }
                                                       }
                                      }
                     }
}
```

简直是魔鬼调用，如果不止3次，而是5次，6次，那还得了。。

而用协程就能很好解决这个问题：

```kotlin
//并发请求GlobalScope.launch(Dispatchers.Main) { 
//三次请求并发进行 
val value1 = async { 
  request1(parameter1) 
}
val value2 = async { 
  request2(parameter2) 
}
val value3 = async { 
  request3(parameter3)
}
//所有结果全部返回后更新UI 
updateUI(value1.await(), value2.await(), value3.await())
}
//切换到io线程
suspend fun request1(parameter : Parameter){
  withContext(Dispatcher.IO){}
}
suspend fun request2(parameter : Parameter){
  withContext(Dispatcher.IO){}
}
suspend fun request3(parameter : Parameter){
  withContext(Dispatcher.IO){}
}
```

就像是同一个线程中顺序执行的效果一样，再比如我要按顺序执行一次异步任务，然后完成后更新UI，一共三个异步任务。如果正常写应该怎么写？

```kotlin
thread{ 
  iotask1() { value1 ->  
             runOnUiThread{  
               updateUI1(value1)   
               iotask2() { value2 ->  
                          runOnUiThread{  
                            updateUI2(value2)  
                            iotask3() { value3 ->  
                                       runOnUiThread{   
                                         updateUI3(value3)   
                                       }
                                      }
                          }
                         }
             }
            }
}
```

晕了晕了，不就是一次异步任务，一次UI更新吗。怎么这么麻烦，来，用协程看看怎么写：

```kotlin
GlobalScope.launch (Dispatchers.Main) { 
  ioTask1()
  ioTask1()  
  ioTask1()  
  updateUI1()  
  updateUI2()    
  updateUI3()
}
suspend fun ioTask1(){  
  withContext(Dispatchers.IO){}
}
suspend fun ioTask2(){  
  withContext(Dispatchers.IO){}
}
suspend fun ioTask3(){  
  withContext(Dispatchers.IO){}
}
fun updateUI1(){}
fun updateUI2(){}
fun updateUI3(){}
```

#### 协程怎么取消

取消协程作用域将取消它的所有子协程。

```
// 协程作用域 scopeval job1 = scope.launch { … }val job2 = scope.launch { … }scope.cancel()
```

取消子协程

```
// 协程作用域 scopeval job1 = scope.launch { … }val job2 = scope.launch { … }job1.cancel()
```

但是调用了cancel并不代表协程内的工作会马上停止，他并不会组织代码运行。比如上述的job1，正常情况处于active状态，调用了cancel方法后，协程会变成Cancelling状态，工作完成之后会变成Cancelled 状态，所以可以通过判断协程的状态来停止工作。

Jetpack 中定义的协程作用域（viewModelScope 和 lifecycleScope）可以帮助你自动取消任务，下次再详细说明，其他情况就需要自行进行绑定和取消了。

#### Kotlin协程-CoroutineScope协程作用域

[Kotlin协程-CoroutineScope协程作用域](https://juejin.cn/post/7120023947717902373)

- GlobeScope：全局范围，不会自动结束执行。
- MainScope：主线程的作用域，全局范围
- lifecycleScope：生命周期范围，用于activity等有生命周期的组件，在DESTROYED的时候会自动结束。
- viewModelScope：viewModel范围，用于ViewModel中，在ViewModel被回收时会自动结束

## Kotlin

### Kotlin中inline、noinline和corssinline到底是什么？

[Kotlin中inline、noinline和corssinline到底是什么？](https://juejin.cn/post/7244498421284945981)

- inline: 将函数内的代码直接拷贝到调用处。减少函数调用带来的开销，提高程序的性能；消除lambda表达式带来的额外开销，避免创建额外的对象
- noinline: 强制非内联，直接调用内部的代码，而非直接拷贝内部代码到调用处。
- crossinline: 强制内联的意思，crossinline可以禁止在内联的lambda表达式中使用return操作或者结束指定的内联函数继续执行

### kotlin中的reified关键字

[kotlin中的reified关键字](https://juejin.cn/post/6844904002816049159)

使用方法

- 在泛型类型前面增加reified修饰
- 在方法前面增加inline

泛型在运行时会被类型擦除，但是在inline函数中我们可以指定类型不被擦除， 因为inline函数在编译期会将字节码copy到调用它的方法里，所以编译器会知道当前的方法中泛型对应的具体类型是什么，然后把泛型替换为具体类型，从而达到不被擦除的目的，在inline函数中我们可以**通过reified关键字来标记这个泛型在编译时替换成具体类型**

## Android 和WebView 通信

### js调用android

// myObj 为在js中使用的对象名称 JavaScriptInterfaces 是我们自定义的一个类。

```
webView.addJavascriptInterface(new JavaScriptInterfaces(), "myObj");
```

myObj.xx() //js方法

#### 通过WebView的addJavascriptInterface（）进行对象映射

```java
public class AndroidtoJs extends Object {  
  // 定义JS需要调用的方法  
  // 被JS调用的方法必须加入@JavascriptInterface注解 
  @JavascriptInterface  
  public void hello(String msg) {  
    System.out.println("JS调用了Android的hello方法");   
  }
}
mWebView.addJavascriptInterface(new AndroidtoJs(), "test");
//js中：
function callAndroid(){   
 // 由于对象映射，所以调用test对象等于调用Android映射的对象   
 test.hello("js调用了android中的hello方法");
}
```

这种方法虽然很好用，但是要注意的是4.2以后，对于被调用的函数以@JavascriptInterface进行注解，否则容易出发漏洞，因为js方可以通过反射调用一些本地命令，很危险。

#### 通过 WebViewClient 的shouldOverrideUrlLoading ()方法回调拦截 url

这种方法是通过shouldOverrideUrlLoading回调去拦截url，然后进行解析，如果是之前约定好的协议，就调用相应的方法。

```java
// 复写WebViewClient类的shouldOverrideUrlLoading方法
mWebView.setWebViewClient(new WebViewClient() {   
  @Override    
  public boolean shouldOverrideUrlLoading(WebView view, String url) {  
    Uri uri = Uri.parse(url);          
    // 如果url的协议 = 预先约定的 js 协议     
    if ( uri.getScheme().equals("js")) { 
      // 如果 authority  = 预先约定协议里的 webview，即代表都符合约定的协议    
      if (uri.getAuthority().equals("webview")) {            
      System.out.println("js调用了Android的方法");          
        // 可以在协议上带有参数并传递到Android上           
        HashMap<String, String> params = new HashMap<>();    
        Set<String> collection = uri.getQueryParameterNames();  
      }
      return true;     
    }
    return super.shouldOverrideUrlLoading(view, url);  
  }
});
```

### android 调用js

无参数：

```java
mWebView.loadUrl("javascript:wave()");
```

但是这种不常用，因为它会自动刷新页面而且没有返回值，有点影响交互。

有参数：

```java
webView.evaluateJavascript(String.format("javascript:callH5Re('测试数据')"), 
                           new ValueCallback<String>() {      
                             @Override  
                             public void onReceiveValue(String value) {  
                               Log.e("Test", "onReceiveValue: "+value ); 
                             }
                           });
```

这种就比较全面了。调用方法并且获取返回值。

## 网络通信的过程，以及中间用了什么协议

### 客户端

1. 在浏览器输入网址。
2. 浏览器解析网址，并生成http请求消息。
3. 浏览器调用系统解析器，发送消息到DNS服务器查询域名对应的ip。
4. 拿到ip后，和请求消息一起交给操作系统协议栈的TCP模块。
5. 将数据分成一个个数据包，并加上TCP报头形成TCP数据包。
6. TCP报头包括发送方端口号、接收方端口号、数据包的序号、ACK号。
7. 然后将TCP消息交给IP模块。
8. IP模块会添加IP头部和MAC头部。
9. IP头部包括IP地址，为IP模块使用，MAC头部包括MAC地址，为数据链路层使用。
10. IP模块会把整个消息包交给网络硬件，也就是数据链路层，比如以太网，WIFI等。
11. 然后网卡会将这些包转换成电信号或者在光信号，通过网线或者光纤发送出去，再由路由器等转发设备送达接收方。

### 服务器端

1. 数据包到达服务器的数据链路层，比如以太网，然后会将其转换为数据包（数字信号）交给IP模块。
2. IP模块会将MAC头部和IP头部后面的内容，也就是TCP数据包发送给TCP模块。
3. TCP模块会解析TCP头信息，然后和客户端沟通表示收到这个数据包了。
4. TCP模块在收到消息的所有数据包之后，就会封装好消息，生成相应报文发给应用层，也就是HTTP层。

### TCP连接过程，三次握手和四次挥手，为什么？

#### 连接阶段（三次握手）

- 创建套接字Socket，服务器会在启动的时候就创建好，客户端是在需要访问服务器的时候创建套接字。
- 然后发起连接操作，其实就是Socket的connect方法。
- 这时候客户端会生成一个TCP数据包。这个数据包的TCP头部有几个重要信息：SYN、ACK、Seq、Ack。

> SYN，同步序列编号，是TCP/IP建立连接时使用的握手信号，如果这个值为1就代表是连接消息。  
> ACK，确认标志，如果这个值为1就代表是确认消息。Seq，数据包序号，是发送数据的一个顺序编号。
> Ack Number，确认数字号，是接收数据的一个顺序编号。

- 所以客户端就生成了这样一个数据包，其中头部信息的控制位SYN设置为1，代表连接。SEQ设置一个随机数，代表初始序号，比如100。
- 然后服务器端收到这个消息，知道了客户端是要来连接的（SYN=1），知道了传输数据的初始序号（SEQ=100）。
- 服务器端也要生成一个数据包发送给客户端，这个数据包的TCP头部会包含：表示我也要连接你的SYN（SYN=1），我已经收到了你的上个数据包的确认号ACK=1（Ack=Seq+1=101），以及服务器端随机生成的一个序号Seq（比如Seq=200）。
- 最后客户端收到这个消息后，表示客户端到服务器的连接是无误了，然后再发送一个数据包表示也确认收到了服务器发来的数据包，这个数据包的头部就主要就是一个ACK=1（Ack=Seq+1=201）。
- 至此，连接成功，三次握手结束，后面数据就会正常传输，并且每次都要带上TCP头部中的Seq和Ack。

这里有个问题是关于为什么需要三次握手？

最主要的原因就是需要通信双方都确认自己的消息被准确传达过去了。

A发送消息给B，B回一条消息表示我收到了，这个过程就保证了A的通信能力。B发送消息给A，A回一条消息表示我收到了，这个过程就保证了B的通信能力。

也就是四条消息能保证双方的消息发送都是正常的，其中B回消息和B发消息，可以融合为一次消息，所以就有了三次握手。

![微信图片_20210316114243.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2021/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20210316114243.png)

### 数据传输阶段

数据传输阶段有个改变就是Ack确认号不再是Seq+1了，而是Seq+数据长度。例如：

- A发送给B的数据包（Seq=100，长度=1000字节）
- B回给A的数据包（Ack=100+1000=1100）

这就是一次数据传输的头部信息，Ack代表下个数据包应该从哪个字节开始所以等于上个数据包的Seq+长度，Seq就等于上个数据包的Ack。

当然，TCP通信是双向的，所以实际数据每个消息都会有Seq和Ack：

- A发送给B的数据包（Ack=200，Seq=100，长度=1000字节）
- B回给A的数据包（Ack=100+1000=1100，Seq=上一个数据包的Ack=200，长度=500字节）
- A发送给B数据包（Seq=1100，Ack=200+500=700）

![微信图片_20210316114342.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2021/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20210316114342.png)

### 断开阶段（四次挥手）

和连接阶段一样，TCP头部也有一个专门用作关闭连接的值叫做FIN。

- 客户端准备关闭连接，会发送一个TCP数据包，头部信息中包括（FIN=1代表要断开连接）。
- 服务器端收到消息，回复一个数据包给客户端，头部信息中包括Ack确认号。但是此时服务器端的正常业务可能没有完成，还要处理下数据，收个尾。
- 客户端收到消息。
- 服务器继续处理数据。
- 服务器处理数据完毕，准备关闭连接，会发送一个TCP数据包给客户端，头部信息中包括（FIN=1代表要断开连接）。
- 客户端端收到消息，回复一个数据包给服务器端，头部信息中包括Ack确认号。
- 服务器收到消息，到此服务器端完成连接关闭工作。
- 客户端经过一段时间（2MSL），自动进入关闭状态，到此客户端完成连接关闭工作。

> MSL 是 Maximum Segment Lifetime，报文最大生存时间，它是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃。

这里有个问题是关于为什么需要四次挥手？

A发送断开消息给B，B回一条消息表示我收到了，这个过程就保证了A断开成功。B发送断开消息给A，A回一条消息表示我收到了，这个过程就保证了B断开成功。

其实和连接阶段的区别就在于，这里的B的确认消息和断开消息不能融合。因为A要断开的时候，B可能还有数据要处理要发送，所以要等正常业务处理完，在发送断开消息。

![微信图片_20210316114502.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2021/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20210316114502.png)

### 常用的状态码

- 1XX - 临时消息。服务器收到请求，需要请求者继续操作。
- 2XX - 请求成功。请求成功收到，理解并处理。
- 3XX - 重定向。需要进一步的操作以完成请求。
- 4XX - 客户端错误。请求包含语法错误或无法完成请求。
- 5XX - 服务器错误。服务器在处理请求的过程中发生了错误。

常见状态码：

- 200 OK - 客户端请求成功
- 301 - 资源(网页等)被永久转移到其它URL
- 302 - 临时跳转
- 400 Bad Request - 客户端请求有语法错误，不能被服务器所理解
- 404 - 请求资源不存在，错误的URL。
- 500 - 服务器内部发生了不可预期的错误。
- 503 Server Unavailable - 服务器当前不能处理客户端的请求，一段时间后可能恢复正常。

### 讲一下TCP协议和UDP协议的区别和场景

#### 第一个场景，浏览网页。（TCP场景）

- 我们访问网页，网页肯定要把所有数据都正确的显示出来吧，如果这个过程中丢包了，那么肯定也会重新传包，不可能只显示一部分网页。（保证数据正确性）
- 同样，网页中的内容肯定也需要是顺序的。比如我一个抽奖，不可能还没抽就把奖给你了。（保证数据的顺序）
- 再来，在这个对数据要求严格的过程中，我们肯定需要两方建立起一个可靠的连接，也就是我们上述说到的要经过三次握手才开始传输数据，并且每次发数据包都需要回执。（面向连接的）
- 而这种连接中传输数据就是用的字节流，也就是有根管道，你想怎么传数据都行，想怎么接受数据也都可以，只要在这一根管道里面。

所以这种需要数据准确、顺序不能错、要求稳定可靠的场景就需要用到TCP。

#### 第二个场景，打游戏。（UDP场景）

打游戏最最重要的就是即时，不然我这个技能发出去了你那边还没被打中，这就玩不了了。

- 所以UDP是需要保证数据的即时性，而不保证每个数据包都正确接收到，即使丢包了，也不会去找丢的那个是什么包，因为要显示当前时间的当前数据包。（不保证数据正确性和数据顺序，可能会丢包）
- 同样，为了数据的即时性，UDP也就不会去建立连接了，不需要什么三次握手，每次你还要确认收没收到。管你收没收到，我只要快速把每个数据包丢给你就行了。（面向无连接的）
- 因为是无连接的，所以就不需要用到字节流，直接每次丢一个数据报给你，接收方也只能接受一个数据报（不能和其他发送方的数据报混淆）。（基于数据报的）

如果你还是有点晕，可以看看这篇文章（亚当和夏娃），很形象的比喻：
<https://www.zhihu.com/question/51388497?sort=created>

### socket和WebSocket

虽然这两个货名字类似，但其实不是一个层级的概念。

- socket，套接字。上文说过了，在TCP建立连接的过程中，是调用了Socket的相关API，建立了这个连接通道。所以它只是一个接口，一个类。

- WebSocket，是和HTTP同等级，属于应用层协议。它是为了解决长时间通信的问题，由HTML5规范引出，是一种建立在TCP协议基础上的全双工通信的协议，同样下层也需要TCP建立连接,所以也需要socket。

> 科普：WebSocket在TCP连接建立后，还要通过Http进行一次握手，也就是通过Http发送一条GET请求消息给服务器，告诉服务器我要建立WebSocket连接了，你准备好哦，具体做法就是在头部信息中添加相关参数。然后服务器响应我知道了，并且将连接协议改成WebSocket，开始建立长连接。

如果硬要说这两者有关系，那就是WebSocket协议也用到了TCP连接，而TCP连接用到了Socket的API。

### Https的连接建立过程

说完了HTTP和TCP/IP，再说说HTTPS。

上一篇文章说了HTTPS是怎么保证数据安全传输
<https://mp.weixin.qq.com/s/dbmwBVxHkvQ0fzWaSdtPYg>

其中主要就是用到了数字证书。

现在完整看看Https连接建立（也叫TLS握手流程）：

#### 1.客户端发送 Client Hello 数据包消息

这个消息内容包括一个随机数（randomC）,加密族（密钥交换算法也就是非对称加密算法、对称加密算法、哈希算法）,Session ID(用作恢复回话)。

客户端要建立通信，在TCP握手之后，会发送第一个消息，也叫Client Hello消息。这个消息主要发了以上的一些内容，其中密文族就是把客户端这边支持的一些算法发给服务器，然后服务器拿来和服务器支持的算法一比较，就能得出双方都支持的最优算法了。

#### 2.服务器回复三个数据包消息：Server Hello，Certificate，Server Hello Done

Server Hello消息内容包括一个随机数（randomS），比较后得出的加密族，Session ID(用作恢复回话)。

到现在，双方已经有两个随机数了，待会再看看这两个随机数是干嘛的。然后加密算法刚才说过了，服务器协商出了三种算法并发回给客户端。

Certificate消息就是发送数字证书了。这里就不细说了。

Server Hello Done消息就是个结束标志，表示已经把该发的消息都发给你了。

#### 3.对称密钥生成过程

1）首先，客户端会对发来的证书进行验证，比如数字签名、证书链、证书有效期、证书状态。

2）证书校验完毕后，然后客户端会用证书里的服务器公钥加密发送一个随机数 pre—master secret ，服务器收到之后用自己的私钥解密。

3）到此，客户端和服务器就都有三个随机数了：randomC、randomS、pre—master secret。

4）然后客户端和服务器端分别按照固定的算法，用三个随机数生成对称密钥。

#### 4.生成Session ID

这一步和开始两个hello消息中的Session ID对应起来了。

会生成会话的id，如果后续会话断开了，那么通过这个Session ID就可以恢复对话，不需要重新进行发送证书、生成密钥过程了。

#### 5.用对称密钥传输数据

拿到对称密钥后，双方就可以使用对称密钥加密解密数据，进行正常通信了。

![微信图片_20210316114936.png](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2021/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20210316114936.png)

扩展：为什么要使用非对称加密算法协商出对称加密这种方法？

首先，网络传输数据对传输的速度要求比较高，在保证安全的前提下，所以采用了对称加密的方法，而不用耗时较多的非对称加密算法。

其次，在确定对称加密传输数据的前提下，如果传输对称加密的密钥是个涉及到安全的问题，所以就采用了安全性更高的非对称加密算法，加上证书链机制，保证了传输对称密钥相关数据的安全性。

### 请给我讲解一下数字签名，为什么真实可靠

数字签名，也就是上文中说的电子签名，再简单回顾下：

数字签名，其实也是一种非对称加密的用法。

它的使用方法是：

A使用私钥对数据的哈希值进行加密，这个加密后的密文就叫做签名，然后将这个密文和数据本身传输给B。

B拿到后，签名用公钥解密出来，然后和传过来数据的哈希值做比较，如果一样，就说明这个签名确实是A签的，而且只有A才可以签，因为只有A有私钥。

反应实际情况就是：

服务器端将数据，也就是我们要传的数据（公钥），用另外的私钥签名数据的哈希值，然后和数据（公钥）一起传过去。然后客户端用另外的公钥对签名解密，如果解密数据和数据（公钥）的哈希值一致，就能证明来源正确，不是被伪造的。

- 来源可靠。数字签名只能拥有私钥的一方才能签名，所以它的存在就保证了这个数据的来源是正确的
- 数据可靠。hash值是固定的，如果签名解密的数据和本身的数据哈希值一致，说明数据是未被修改的。

### 证书链安全机制

> 证书颁发机构（CA, Certificate Authority）即颁发数字证书的机构。是负责发放和管理数字证书的权威机构，并作为电子商务交易中受信任的第三方，承担公钥体系中公钥的合法性检验的责任。

实际情况中，服务器会拿自己的公钥以及服务器的一些信息传给CA，然后CA会返回给服务器一个数字证书，这个证书里面包括：

- 服务器的公钥
- 签名算法
- 服务器的信息，包括主机名等。
- CA自己的私钥对这个证书的签名

然后服务器将这个证书在连接阶段传给客户端，客户端怎么验证呢？

细心的小伙伴肯定知道，每个客户端，不管是电脑、手机都有自带的系统根证书，其中就会包括服务器数字证书的签发机构。所以系统的根证书会用他们的公钥帮我们对数字证书的签名进行解密，然后和证书里面的数据哈希值进行对比，如果一样，则代表来源是正确的,数据是没有被修改的。

当然中间人也是可以通过CA申请证书的，但是证书中会有服务器的主机名，这个主机名（域名、IP）就可以验证你的来源是来源自哪个主机。

扩展一下：

其实在服务器证书和根证书中间还有一层结构：叫中级证书，我们可以任意点开一个网页，点击左上角的🔒按钮就可以看到证书详情：

![微信图片_20210316115054.jpg](https://raw.githubusercontent.com/zhouhuandev/ImageRepo/master/2021/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20210316115054.jpg)
