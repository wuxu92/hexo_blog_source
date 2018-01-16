---
title: 'C#-System.TypeInitializationException'
date: 2016-03-08 12:31:29
categories:
- programming
tags:
- C#
- programming
- exception
---

今天给科委的项目加上自动上传的功能，把之前设计的自动功能彻底修改了。由于这个项目已经好几个月没有动了，把原来的功能重构之后，在调试模式下运行程序会报异常，`System.TypeInitializationException`。这个异常在运行时才会报错，编译生成项目是成功的。

以前开发时也遇到过这个问题，一直很奇怪，因为抛出异常的地方在整个项目的入口：

```
static class Program
{
	static void Main()
    {
    	Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        Application.Run(new MainForm()); // 在这里抛出异常
    }
...
}
```

从VS的直接报错里面看不出什么有用的东西，查不到到底是哪里出的问题。可以知道的是，是构造 MainForm 类的实例的时候出了问题。
<!-- more -->
我们知道，类实例化的流程是先要处理静态域（仅第一次构造该类的对象时），再调用构造函数返回对象的实例。然后我发现 MainForm 类中有不少的静态域成员，那么很有可能是初始化这些静态成员时出了问题，奇怪的是为什么VS不能给出异常发生的真实地点。。。。

我们知道可以在类中调用其它类的静态成员（public的），这时就有可能出现问题，可能在该类中调用另外的类的静态成员时，要调用的静态成员还没有实例化。这就是导致我的 `TypeInitializationException`的原因。

比如MainForm中有下面两个静态域：

```
public class MainForm {
	static Config conf;
	static Logger logger;

	// constructor
	public MainForm() {
		...
	}
}
```
而我想在conf中调用MainForm中定义的logger（这是一个不好的实现，实际的日记记录类应该有Logger类的工厂方法提供），于是在conf中：

```
public class Config {
	...
	public Config() {
		...
		MainForm.logger.info(...);
	}
	...
}
```
这样在实例化MainForm的静态成员conf时会调用一个MainForm中还没有实例化的静态变量，从而导致错误。

这是我目前所理解的，也有可能不对，但是修改这个问题后，就可以编译通过了，姑且认为是这样的吧。
