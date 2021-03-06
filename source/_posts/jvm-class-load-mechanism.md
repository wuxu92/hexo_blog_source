---
title: Java虚拟机类加载机制
date: 2016-03-25 20:13:43
categories:
- jvm
tags:
- Java
- jvm
---
前段时间，室友面试被问到了java的类加载机制。关于类加载机制之前看《深入理解Java虚拟机》的时候看过这一章，不过都忘得差不多了，只记得有一个复杂的流程：验证－准备－解析－初始化等等，今天复习一下顺便做个笔记。

Java代码编译为字节码（.class文件）后，需要加载到虚拟机之后才能运行和使用，还有很多运行在JVM平台的其他语言（如Grovvy, scala, kotlin等）也会被编译为符合JVM字节码规范的文件后被虚拟机读取运行。那么JVM是如何运行这些字节码文件的呢。

与C语言的在编译时进行连接工作不同，Java中， 类型的加载和连接都是在程序运行时期完成的，Java的动态扩展的语言特性就是依赖于运行期动态加载和动态连接的特点。
<!-- more -->
## 类加载的时机
从类被加载到执行结束卸载出内存，它的整个声明周期包括了：**加载**-**验证**-**准备**-**解析**-**初始化**-**使用**-**卸载** 七个阶段。其中的验证-准备-解析又统称为连接（linking）。除解析外，各个阶段并没有绝对的时间先后顺序，但是开始执行的顺序是确定的。而解析有时候可以在初始化后再开始。这些阶段通常是交叉混合式进行的。

虚拟机规范并没有规定什么时候要开始类的加载过程的第一个夹断也就是“加载”，而是取决于虚拟机的具体实现。对于初始化阶段，虚拟机规范严格规定了**有且只有**4种情况必须立即对类进行初始化。

1. 遇到 `new`, `getstatic`, `putstatic` 或者 `invokestatic`4条指令的时候，如果类没有初始化过，则需要先触发其初始化。这四个指令对应的场景就是：实例化对象，读取或者设置类的静态成员，或者调用类的静态方法。
2. 使用`Java.lang.reflect`包的方法对类进行反射调用的时候
3. 当初始化一个类的时候发现其父类还没有进行过初始化，则需要先触发父类的初始化
4. 当虚拟机启动时，用户需要指定一个要执行的**主类**，即包含了main函数的类，虚拟机会先初始化这个主类。

注意，通过子类来引用父类中的静态字段只会触发父类的初始化而不会触发子类的初始化。同时我们也应该理解了，类的初始化和类的实例化是两回事。
还要注意：**常量在编译阶段会存入调用类的常量池中**，本质上没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。


## 类加载的过程

### 加载
加载（loading）是类加载的第一个阶段，在这个阶段，虚拟机主要完成三件事：

1. 通过一个类的全限定名来获取定一次类的二进制字节流
2. 通过这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
3. 在java堆中生成一个代表这个类的 Java.lang.Class 对象，作为方法去访问这些数据的入口

类的全限定名就是类的包名加类名，一般来说包名和文件夹路径名统一，类名与文件名统一。但是获取的二进制流并没有限定为是class文件，可能是ZIP包，可能是网络流也可能是运行时计算生成的（代理）。这个阶段有很大的灵活性。开发人员可以定义自己的类加载器去控制字节流的获取方法，虽然一般只使用系统的类加载器，后面会将类加载器的工作原理。

### 验证
虽然Java的编译器在编译时会检查如数组越界，错误的类型转换等工作，但是Class文件不一定是由Java便以来的，可能是有其他语言编译的字节码，甚至可以是十六进制编辑器编辑过的文件。所以在虚拟机执行字节码之前，需要对class文件进行验证。

虽然这一工作很重要，在类加载子系统中占了很大一部分，但是虚拟机规范并没有规定验证具体要做什么工作，只是说如果验证到不符合Class文件的存储格式就抛出 `Java.lang.VerifyError` 异常或其子异常。这一工作取决于虚拟机的具体实现。一般会进行下面四个方面的验证：

1. 文件格式验证。如是否魔数（`0xCAFEBABE`）开头，主次版本号，常量池中的类型等，很多验证
2. 元数据验证，如是否有父类，是否继承了不允许被继承的类（final修饰的类，如String类），抽象类验证等
3. 字节码验证，进行数据流和控制流分析，保证类的方法运行时不会危害虚拟机安全
4. 符号引用验证，在虚拟机将符号引用转换为直接引用的时候进行验证，保证解析动作正常执行。

验证阶段是一个重要的阶段，但是这个阶段不是必要的，如果所有文件被反复使用和验证过，可以通过虚拟机的  `-Xverufy:none` 参数关闭验证，缩短虚拟机类加载的时间。

### 准备
准备阶段正式为类变量分配内存（分配在方法区）并设置类变量初始值（数据类型的零值）。在这里进行内存分配的仅包括类变量（被static修饰的变量），而不包括实例变量，实例变量将在对象实例化时随着对象一起分配在java堆中。

对于常量属性会在准备阶段就设置值，而其他属性则只会设置零值。

### 解析
解析是虚拟机将常量池内的符号引用替换为直接引用的过程。解析动作主要针对类或接口、字段、类方法、接口方法四类符号引用进行。

关于解析阶段不详细介绍，比较复杂

### 初始化
类初始化是类加载过程的最后一步，到初始化阶段真正开始执行类中定义的Java程序代码（前面的自定义类加载器也算）。在初始化阶段根据程序员通过程序制定的计划初始化类变量。可以理解为初始化阶段执行了一个类构造器 `<clinit>()` 方法的过程。

`<clinit>()`方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块中语句合并产生的。静态语句块只能访问定义在静态语句块之前的变量，在前面的静态语句块中可以赋值但是不能访问。

如果类没有静态语句块也没有对变来那个的赋值，编译器就不会生成 `<clinit>()`方法。虚拟机会保证一个类的 clinit 方法在多线程环境下被正确地加锁和同步。只有一个线程去执行类的 `<clinit>()`方法。

## 类加载器
Java的类加载器最早是为 Applet 需求开发出来的，现在 Applet已经很少用了，类加载器却在 类层次划分、OSGi、热部署、代码加密等领域大放异彩。

类加载器虽然只用于实现类的加载动作，但在java程序中起的作用远不限于加载阶段。任何一个类需要类本身和加载这个类的类加载**二者**才能确立其在虚拟机中的唯一性。即使两者使用同一个Class文件，只要类加载器不同，那两个类就不想等，也就是不同类加载器加载的同一个class文件的实例使用Equals、instanceof等方法的结果是不会相等的。

### 双亲委派模型
Java虚拟机角度来看，存在两种不同的类加载器，一种是启动类加载器（Bootstrap ClassLoader）,这个类加载器是虚拟机的一部分，另一种就是其他类加载器，独立于虚拟机，并且全部继承自抽象类`Java.lang.ClassLoader`。

从开发者角度看，一般会使用下面三种类加载器：

1. 启动类加载器，同上，负责加载 JAVA_HOME/lib 目录中，或是 -Xbootclasspath 指定路径下的类。该加载器无法被java程序直接引用
2. 扩展类加载器，有`sun.misc.Launcher$ExtClassLoader`实现，加载 JAVA_HOME/lib/ext 目录中的类库。开发者可以直接使用该类
3. 应用程序类加载器，这个类有 `sun.misc.Launcher$AppClassLoader `实现，一般称为系统类加载器，是默认的用户类加载器，通过 ClassLoader.getSystemClassLoader()方法返回。

应用程序由这三种类加载器配合进行加载，还可以自定义类加载器，只要继承 ClassLoader 接口，实现其 `LoadClass()` 方法即可。

在JDK 1.2中，引入了类加载器的**双亲委派模型**，双亲委派模型要求除了顶层的启动类加载器外，其余的类加载应该有自己的父类加载器。加载器之间的父子关系通常使用组合而不是继承来实现。

![](/images/lang/Java-classloader.png)

其工作工程是，如果一个类加载器收到类加载请求，它首先不会自己尝试去加载这个类，而是把请求委派为父类加载器去完成，只有当父类加载器返回自己无法完成请求时，子类加载器才会自己加载。

双亲委派模型使得Java类随着它的类加载器一起具备了带有优先级的层次关系。这样用户定义的类不会随意覆盖Java库中定义的类。双亲委派模型对于Java程序的稳定运行很重要。其实现其实很简单，只是先检查是否已经加载过，没有则调用父加载器的loadClass方法，如果父类加载器失败了调用自己的findClass()方法进行加载。

### 破坏双亲委派模型
双亲委派模型并不是一个强制的编程约束，而是java设计者推荐的实现方法，实际上自己定义的类加载可以不实现该模型，这样就破坏了双亲委托模型。

实际上很多的Java组件都通过破坏双亲委派模型实现自己的功能，Java设计团队通过引入一个 **线程上下文类加载器**实现，这个类加载器可以通过 `Java.lang.Thread` 类的 `setContectClassLoader()` 方法进行设置，如果创建线程时都没有设置过，那么这类加载器默认就是应用程序类加载器。

JNDI服务，JDBC， JCE，JBI等（SPI：service provider interface）都使用线程上下文类加载器去加载所需要的代码，也就是父类加载器请求子类加载器去完成类加载动作。

像代码热替换，模块热部署等都是通过破坏双亲委派模型实现的，OSGi是现在业界事实上的java模块化标准，在后续的Java版本中会优化其模块化功能。关于OSGi的内容，我也没用过，就不记了。


参考：

- 深入理解Java虚拟机-JVM高级特性与最佳实践
