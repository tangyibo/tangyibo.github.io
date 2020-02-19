---
layout: post
title: Janino框架初识与使用教程|Java编程 
category: Git
tag: [JAVA]
---

这篇文章主要介绍Janino框架初识与使用教程。
本文分为以下几个部分：
1. Janino简介
2. Janino的应用
3. Janino的使用教程

# Janino框架初识与使用教程
 
## 一、 Janino简介
 
Janino 是一个极小、极快的 开源Java 编译器（Janino is a super-small, super-fast Java™ compiler.）。Janino 不仅可以像 JAVAC 一样将 Java 源码文件编译为字节码文件，还可以编译内存中的 Java 表达式、块、类和源码文件，加载字节码并在 JVM 中直接执行。Janino 同样可以用于静态代码分析和代码操作。

项目地址：https://github.com/janino-compiler/janino

官网地址：http://janino-compiler.github.io/janino/

## 二、Janino的应用

### 1、Kettle使用Janino框架来实现自定义"Java代码组件"步骤功能；

详细介绍地址：http://www.uml.org.cn/sjjmck/201910101.asp?artid=22506

### 2、在日志框架里当引用 slf4j + log4j/logback 的时候，常常会顺带引用 Janino 来提高日志输出的性能

## 三、使用教程

在使用前需要在pom.xml中引入如下依赖：
```
<dependency>  
    <groupId>org.codehaus.janino</groupId>
    <artifactId>janino</artifactId>
    <version>3.0.11</version>
</dependency>  
```
注：请查看官网最新版本发布情况；

### 1、打印最简单的Hello World!
```
import org.codehaus.commons.compiler.IScriptEvaluator;
import org.codehaus.janino.ScriptEvaluator;

public class JaninoTester01 {

	public static void main(String[] args) {
		try {
			String content="System.out.println(\"Hello world\");";
			IScriptEvaluator evaluator = new ScriptEvaluator();
			evaluator.cook(content);
			evaluator.evaluate(null);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}
```
### 2、执行最简单的算数表达式计算
```
import org.codehaus.janino.ExpressionEvaluator;
import org.codehaus.janino.ScriptEvaluator;

public class JaninoTester02 {

	public static void main(String[] args) {
		try {
			String express = "(1+2)*3";
			ScriptEvaluator evaluator = new ExpressionEvaluator();
			evaluator.cook(express);
			Object res = evaluator.evaluate(null);
			System.out.println(express + "=" + res);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}
```

### 3、执行带参数的表达式计算
```
import java.lang.reflect.InvocationTargetException;
import org.codehaus.commons.compiler.CompileException;
import org.codehaus.janino.ExpressionEvaluator;

public class JaninoTester03 {

	public static void main(String[] args) throws CompileException, InvocationTargetException {
		// 首先定义一个表达式模拟器ExpressionEvaluator对象
		ExpressionEvaluator ee = new ExpressionEvaluator();

		// 定义一个算术表达式，表达式中需要有2个int类型的参数a和b
		String expression = "2 * (a + b)";
		ee.setParameters(new String[] { "a", "b" }, new Class[] { int.class, int.class });

		// 设置表达式的返回结果也为int类型
		ee.setExpressionType(int.class);

		// 这里处理（扫描，解析，编译和加载）上面定义的算数表达式.
		ee.cook(expression);

		// 根据输入的a和b参数执行实际的表达式计算过程
		int result = (Integer) ee.evaluate(new Object[] { 19, 23 });
		System.out.println(expression + " = " + result);
	}

}
```
### 4、执行java脚本中的函数
```
import java.lang.reflect.InvocationTargetException;
import org.codehaus.commons.compiler.CompileException;
import org.codehaus.janino.ScriptEvaluator;

public class JaninoTester04 {

	public static void main(String[] args) throws CompileException, InvocationTargetException {
		ScriptEvaluator se = new ScriptEvaluator();
        se.cook(
            ""
            + "static void method1() {\n"
            + "    System.out.println(\"run in method1()\");\n"
            + "}\n"
            + "\n"
            + "static void method2() {\n"
            + "    System.out.println(\"run in method2()\");\n"
            + "}\n"
            + "\n"
            + "method1();\n"
            + "method2();\n"
            + "\n"

        );
 
        se.evaluate(null);
	}

}
```
### 5、向java脚本中传递参数
```
import java.lang.reflect.InvocationTargetException;
import org.codehaus.commons.compiler.CompileException;
import org.codehaus.janino.ScriptEvaluator;

public class JaninoTester05 {

	public static void main(String[] args) throws CompileException, InvocationTargetException {
		ScriptEvaluator se = new ScriptEvaluator();
		se.setParameters(new String[] { "arg1", "arg2" }, new Class[] { String.class, int.class });
        se.cook(
            ""
            + "System.out.println(arg1);\n"
            + "System.out.println(arg2);\n"
            + "\n"
            + "static void method1() {\n"
            + "    System.out.println(\"run in method1()\");\n"
            + "}\n"
            + "\n"
            + "public static void method2() {\n"
            + "    System.out.println(\"run in method2()\");\n"
            + "}\n"
            + "\n"
            + "method1();\n"
            + "method2();\n"
            + "\n"

        );
        se.evaluate(new Object[]{"aaa",22});
	}

}
```
### 6、在Java脚本中实现一个接口以供直接调用
```
import java.io.StringReader;
import org.codehaus.janino.ClassBodyEvaluator;
import org.codehaus.janino.Scanner;

public class JaninoTester06 {
	
	public interface Foo {
	    int bar(int a, int b);
	}
	
	public static void main(String[] args) throws Exception {
		Foo f = (Foo) ClassBodyEvaluator.createFastClassBodyEvaluator(
		    new Scanner(null, new StringReader("public int bar(int a, int b) { return a + b; }")),
		    Foo.class,                  // 实现的父类或接口
		    (ClassLoader) null          // 这里设置为null表示使用当前线程的class loader
		);
		System.out.println("1 + 2 = " + f.bar(1, 2));

	}

}
```
### 7、在Java脚本中自定义类与调用

- 步骤（1）：定义一个基类BaseClass
```
package com.tang.janino.obj;

public class BaseClass {
	
	private String baseId;

	public BaseClass(String baseId) {
		super();
		this.baseId = baseId;
	}

	@Override
	public String toString() {
		return "BaseClass [baseId=" + baseId + "]";
	}

}
```
- 步骤（2）：定义一个子类DerivedClass
```
package com.tang.janino.obj;

public class DerivedClass extends BaseClass {

	private String name;

	public DerivedClass(String baseId, String name) {
		super(baseId);
		this.name = name;
	}

	@Override
	public String toString() {
		return super.toString() + "DerivedClass [name=" + name + "]";
	}

}
```
- 步骤（3）：构造java脚本，调用上述类
```
import org.codehaus.commons.compiler.IScriptEvaluator;
import org.codehaus.janino.ScriptEvaluator;

public class JaninoTester07 {

	public static void main(String[] args) {
		try {
			IScriptEvaluator se = new ScriptEvaluator();
			se.setReturnType(String.class);
			se.cook("import com.tang.janino.obj.BaseClass;\n"
					+ "import com.tang.janino.obj.DerivedClass;\n"
					+ "BaseClass o=new DerivedClass(\"1\",\"join\");\n" 
					+ "return o.toString();\n");
			Object res = se.evaluate(new Object[0]);
			System.out.println(res);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}
```
## 四、其他用法

1、https://www.programcreek.com/java-api-examples/index.php?api=org.codehaus.commons.compiler.IClassBodyEvaluator

2、http://janino.unkrig.de/samples/ScriptDemo.java
