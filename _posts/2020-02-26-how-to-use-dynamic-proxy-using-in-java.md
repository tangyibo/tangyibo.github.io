---
layout: post
title: Java的动态代理及其实现原理|Java编程 
category: JAVA
tag: [JAVA]
---

这篇文章主要介绍Java的动态代理及其实现原理。
本文分为以下几个部分：
1. JDK的动态代理
2. CGlib的动态代理
3. JAVAssist的动态代理
4. Java动态代理实现的原理

# Java的动态代理及其实现原理

## 一、Java的动态代理

### 1、JDK的动态代理

JDK提供的动态代理功能只能针对(一个或多个)接口提供代理。使用步骤如下:

- (1) 新建一个接口

```
public interface ISubject {
	public void print();
}
```

- (2) 为接口创建一个实现类

```
public class SubjectImpl implements ISubject {

	@Override
	public void print() {
		System.out.println("hello world!");
	}

}
```

- (3) 创建拦截类，实现java.lang.reflect.InvocationHandler接口

```
public class MyInvokeHandler implements InvocationHandler {
	
	private Object instance;
	
	public MyInvokeHandler(Object o) {
		this.instance=o;
	}
	
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		this.before(method);
		Object result=method.invoke(this.instance, args);
		this.after(method);
		return result;
	}

	private void before(Method method) {
		System.out.println("run before method:"+method.getName());
	}
	
	private void after(Method method) {
		System.out.println("run after method:"+method.getName());
	}
}
```

- (4) 使用Proxy.newProxyInstance()函数生成代理对象

```
public class JdkProxyTester {

	public static void main(String[] args) {
		ISubject subject=new SubjectImpl();
		InvocationHandler handler=new MyInvokeHandler(subject);
		ClassLoader classLoader = ISubject.class.getClassLoader();
		Class<?>[] interfaces = { ISubject.class };
		ISubject proxy = (ISubject) Proxy.newProxyInstance(classLoader,interfaces,handler);
		proxy.print();
	}

}
```

### 2、CGlib的动态代理

CGlib也可以实现动态代理，但与JDK的不同之处是: JDK只能为接口的实现类提供代理，而CGlib可以为类的实现类提供代理，原理上使用字节码处理框架ASM来操作字节码，不能对 final类进行继承。

- (1) 添加pom依赖

```
<!-- https://mvnrepository.com/artifact/cglib/cglib -->
<dependency>
	<groupId>cglib</groupId>
	<artifactId>cglib</artifactId>
	<version>3.2.5</version>
</dependency>
```

- (2) 新建一个需要被代理的类

```
public class MySubject {
	
	private String name;

	public MySubject(String name) {
		this.name=name;
	}

	public void print() {
		System.out.println("hello world!,name=" + this.name);
	}

}
```

- (3) 创建拦截类，实现net.sf.cglib.proxy.MethodInterceptor接口

```
public class MyInvokeHandler implements MethodInterceptor {

	@Override
	public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
		this.before(method);
		Object result = proxy.invokeSuper(obj, args);
		this.after(method);
		return result;
	}

	private void before(Method method) {
		System.out.println("run before method:" + method.getName());
	}

	private void after(Method method) {
		System.out.println("run after method:" + method.getName());
	}
}
```

- (4) 使用net.sf.cglib.proxy.Enhancer的create()方法生成代理对象

```
public class CglibTester {

	public static void main(String[] args) {
		MethodInterceptor handler = new MyInvokeHandler();
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(MySubject.class);
		enhancer.setCallback(handler);
		Class<?>[] argsType = new Class<?>[] { String.class };
		Object[] argsValue = new Object[] { "aa" };
		MySubject proxy= (MySubject)enhancer.create(argsType, argsValue);
		proxy.print();
	}
}
```

### 3、JAVAssist的动态代理

- (1) 添加pom依赖

```
<!-- https://mvnrepository.com/artifact/javassist/javassist -->
<dependency>
    <groupId>javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>3.12.1.GA</version>
</dependency>
```

- (2) 新建一个需要被代理的类

```
public class MySubject {
	
	private String name;

	public MySubject(String name) {
		this.name=name;
	}

	public void print() {
		System.out.println("hello world!,name=" + this.name);
	}

}
```

- (3) 创建拦截类，实现javassist.util.proxy.MethodHandler接口

```
public class MyMethodHandler implements MethodHandler {

	@Override
	public Object invoke(Object self, Method thisMethod, Method proceed, Object[] args) throws Throwable {
		this.before(thisMethod);
		// thisMethod为被代理方法 ,proceed为代理方法, self为代理实例 ,args为方法参数
		Object result = proceed.invoke(self, args);
		this.after(thisMethod);
		return result;
	}

	private void before(Method method) {
		System.out.println("run before method:" + method.getName());
	}

	private void after(Method method) {
		System.out.println("run after method:" + method.getName());
	}
}
```

- (4) 使用javassist.util.proxy.ProxyFactory的createClass()方法生成代理对象

```
public class JavassistTester {

	public static void main(String[] args) throws Exception {
		ProxyFactory factory = new ProxyFactory();
		factory.setSuperclass(MySubject.class);
		factory.setFilter(new MethodFilter() {

			@Override
			public boolean isHandled(Method m) {
				return m.getName().equals("print");
			}
		});
		Class<?> clazz = factory.createClass();
		Class<?>[] argsType = new Class<?>[] { String.class };
		Object[] argsValue = new Object[] { "aa" };
		Constructor<?> constructor=clazz.getConstructor(argsType);
		MySubject proxy = (MySubject) constructor.newInstance(argsValue);
		proxy.print();
	}

}
```

## 二、Java动态代理实现的原理

Java动态代理实现的原理:在编译期或运行期间操作修改java的字节码。

操作java字节码的工具有两个比较流行，一个是ASM，一个是Javassit 。

- ASM

 直接操作字节码指令，执行效率高，要是使用者掌握Java类字节码文件格式及指令，对使用者的要求比较高。
 
官网地址：https://asm.ow2.io/index.html

- Javassit 

提供了更高级的API，执行效率相对较差，但无需掌握字节码指令的知识，对使用者要求较低。

官网地址：http://www.javassist.org/

应用层面来讲一般使用建议优先选择Javassit，如果后续发现Javassit 成为了整个应用的效率瓶颈的话可以再考虑ASM。
