---
title: JVM性能调优学习-3-异常.md
date: 2019-12-02
tags: [java,JVM,调优]
---



# JVM如何处理异常





## 异常基本概念

Throwable 有两大直接子类。第一个是 Error，涵盖程序不应捕获的异常。当程序触发 Error 时，它的执行状态已经无法恢复，需要中止线程甚至是中止虚拟机。第二子类则是 Exception，涵盖程序可能需要捕获并且处理的异常。

Exception 有一个特殊的子类 RuntimeException，用来表示“程序虽然无法继续执行，但是还能抢救一下”的情况。

RuntimeException 和 Error 属于 Java 里的非检查异常（unchecked exception）。其他异常则属于检查异常（checked exception）。在 Java 语法中，所有的检查异常都需要程序显式地捕获，或者在方法声明中用 throws 关键字标注。通常情况下，程序中自定义的异常应为检查异常，以便最大化利用 Java 编译器的编译时检查。

异常实例的构造十分昂贵。这是由于在构造异常实例时，Java 虚拟机便需要生成该异常的栈轨迹（stack trace）。该操作会逐一访问当前线程的 Java 栈帧，并且记录下各种调试信息，包括栈帧所指向方法的名字，方法所在的类名、文件名，以及在代码中的第几行触发该异常。

## 异常的捕获

在编译生成的字节码中，每个方法都附带一个异常表。**异常表**中的每一个条目代表一**个异常处理器**，并且由 **from 指针、to 指针、target 指针以及所捕获的异常类型**构成。这些指针的值是字节码索引（bytecode index，bci），用以定位字节码。

当程序触发异常时，Java 虚拟机会从上至下遍历异常表中的所有条目。当触发异常的字节码的索引值在某个异常表条目的监控范围内，Java 虚拟机会判断所抛出的异常和该条目想要捕获的异常是否匹配。如果匹配，Java 虚拟机会将控制流转移至该条目 target 指针指向的字节码。

如果遍历完所有异常表条目，Java 虚拟机仍未匹配到异常处理器，那么它会弹出当前方法对应的 Java 栈帧，并且在调用者（caller）中重复上述操作。在最坏情况下，Java 虚拟机需要遍历当前线程 Java 栈上所有方法的异常表。（弹出当前方法，在调用方中进行寻找）

finally 代码块的编译比较复杂。当前版本 Java 编译器的做法，是复制 finally 代码块的内容，分别放在 try-catch 代码块所有**正常执行路径**以及**异常执行路径**的出口中。

针对异常执行路径，Java 编译器会生成一个或多个异常表条目，监控整个 try-catch 代码块，并且捕获所有种类的异常（在 javap 中以 any 指代）。这些异常表条目的 target 指针将指向另一份复制的 finally 代码块。并且，在这个 finally 代码块的最后，Java 编译器会重新抛出所捕获的异常。

```
public class Foo {
	private int tryBlock;
	private int catchBlock;
	private int finallyBlock;
	private int methodExit;
	public void test() {
		try {
			tryBlock = 0;
		} catch (Exception e) {
			catchBlock = 1;
		} finally {
        	finallyBlock = 2;
        }
    	methodExit = 3;
    }
}
$ javap -c Foo
...
public void test();
Code:
0: aload_0
1: iconst_0
2: putfield #20 // Field tryBlock:I
5: goto 30
8: astore_1
9: aload_0
10: iconst_1
11: putfield #22 // Field catchBlock:I
14: aload_0
15: iconst_2
16: putfield #24 // Field finallyBlock:I
19: goto 35
22: astore_2
23: aload_0
24: iconst_2
25: putfield #24 // Field finallyBlock:I
28: aload_2
29: athrow
30: aload_0
31: iconst_2
32: putfield #24 // Field finallyBlock:I
35: aload_0
36: iconst_3
37: putfield #26 // Field methodExit:I
40: return
Exception table: 
from to target type
0 5 8 Class java/lang/Exception
0 14 22 any	
```

编译结果包含三份 finally 代码块。

前两份分别位于 try 代码块和 catch 代码块的正常执行路径出口。最后一份则作为异常处理器，监控 try 代码块以及 catch 代码块。它将捕获 try 代码块触发的、未被 catch 代码块捕获的异常，以及 catch 代码块触发的异常。



>   1:使用异常捕获的代码为什么比较耗费性能？
> 因为构造异常的实例比较耗性能。这从代码层面很难理解，不过站在JVM的角度来看就简单了，因为JVM在构造异常实例时需要生成该异常的栈轨迹。这个操作会逐一访问当前线程的栈帧，并且记录下各种调试信息，包括栈帧所指向方法的名字，方法所在的类名、文件名，以及在代码中的第几行触发该异常等信息。
> 虽然具体不清楚JVM的实现细节，但是看描述这件事情也是比较费时费力的。
>
> 2:finally是怎么实现无论异常与否都能被执行的？
> 这个事情是由编译器来实现的，现在的做法是这样的，编译器在编译Java代码时，会复制finally代码块的内容，然后分别放在try-catch代码块所有的正常执行路径及异常执行路径的出口中。  



















