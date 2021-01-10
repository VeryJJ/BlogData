---
title: CGLIB中常用API
tags: [Java,cglib,字节码]
date: 2020-12-31 14:30:10
categories: 技术
---

## CGLIB

CGLIB，即Code Generation Library，是一个开源项目。Github地址：[**https://github.com/cglib/cglib**](https://github.com/cglib/cglib)。

CGLIB的github简介：CGLIB - 字节码生成库，是用于生成和转换Java字节码的高级API。它被AOP、测试、数据访问框架用于生成动态代理对象和拦截字段访问。

CGLIB提供两种类型的JAR包：

- cglib-nodep-x.x.x.jar：使用nodep包不需要关联ASM的jar包，jar包内部包含ASM的类库。
- cglib-x.x.x.jar：使用此jar包需要另外提供ASM的jar包，否则运行时报错，建议选用不包含ASM类库的jar包，可以方便控制ASM的。

本文中使用的CGLIB依赖为：

```javascript
<dependency>
  <groupId>cglib</groupId>
	<artifactId>cglib-nodep</artifactId>
	<version>3.2.10</version>
</dependency>

<dependency>
  <groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<version>1.16.22</version>
</dependency>
```

<!--more-->

<br>

## CGLIB基本原理

- 基本原理：动态生成一个要代理类的子类(**被代理的类作为继承的父类**)，子类重写要代理的类的所有不是final的方法。在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。它比使用Java反射的JDK动态代理要快，因为它采用了整形变量建立了方法索引。
- 底层实现：使用字节码处理框架ASM，用于转换字节码并生成新的类。不鼓励直接使用ASM，因为它要求必须对JVM内部结构包括class文件的格式和JVM指令集都很熟悉，**否则一旦出现错误将会是JVM崩溃级别的异常**。

<br>

## CGLIB的包结构

- net.sf.cglib.core：底层字节码处理类，大部分与ASM有关。
- net.sf.cglib.transform：编译期或运行期类和类文件的转换。
- net.sf.cglib.proxy：实现创建代理和方法拦截器的类。
- net.sf.cglib.reflect：反射相关工具类。
- net.sf.cglib.util：集合排序等工具类。
- net.sf.cglib.beans：JavaBean相关的工具类。

<br>

## CGLIB常用API

下面介绍一下CGLIB中常用的几个API，先建立一个模特接口类和普通模特类：

```java
public class SampleClass {

	public String sayHello(String name) {
		return String.format("%s say hello!", name);
	}
}

public interface SampleInterface {

	String sayHello(String name);
}
```

<br>

## Enhancer

Enhancer，即(字节码)增强器。它是CGLIB库中最常用的一个类，功能与JDK动态代理中引入的Proxy类差不多，但是Enhancer既能够代理普通的Java类，也能够代理接口。

Enhancer创建一个被代理对象的子类并且`拦截所有的方法调用`（包括从Object中继承的toString和hashCode方法）。

Enhancer不能够拦截final方法，例如Object.getClass()方法，这是由于final关键字的语义决定的。基于同样的道理，Enhancer也不能对fianl类进行代理操作。这也是Hibernate为什么不能持久化final关键字修饰的类的原因。

```javascript
public class EnhancerClassDemo {

	public static void main(String[] args) throws Exception {
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(SampleClass.class);
    //使用FixedValue，拦截返回值，每次返回固定值"Doge say hello!"
		enhancer.setCallback((FixedValue) () -> "Doge say hello!");
    
		SampleClass sampleClass = (SampleClass) enhancer.create();
		System.out.println(sampleClass.sayHello("throwable-10086"));
		System.out.println(sampleClass.sayHello("throwable-doge"));
		System.out.println(sampleClass.toString());
		System.out.println(sampleClass.getClass());
		System.out.println(sampleClass.hashCode());
	}
}
```

输出结果：

```javascript
Doge say hello!
Doge say hello!
Doge say hello!
class club.throwable.cglib.SampleClass$$EnhancerByCGLIB$$6f6e7a68
Exception in thread "main" java.lang.ClassCastException: java.lang.String cannot be cast to java.lang.Number
......
```

上述代码中，FixedValue用来对所有拦截的方法返回相同的值，从输出我们可以看出来，Enhancer对非final方法test()、toString()、hashCode()进行了拦截，没有对getClass进行拦截。由于hashCode()方法需要返回一个Number，但是我们返回的是一个String，这解释了上面的程序中为什么会抛出异常。

<br>

`Enhancer#setSuperclass()`用来设置父类型，从`toString()`方法可以看出，使用CGLIB生成的类为被代理类的一个子类，类简写名称为`SampleClass$$EnhancerByCGLIB$$e3ea9b7`。

<br>

`Enhancer#create(Class[] argumentTypes, Object[] arguments)`方法是用来创建增强对象的，其提供了很多不同参数的方法用来匹配被增强类的不同构造方法。我们也可以先使用`Enhancer#createClass()`来创建字节码(.class)，然后用字节码加载完成后的类动态生成增强后的对象。
Enhancer中还有其他几个方法名为create的方法，提供不同的参数选择，具体可以自行查阅。

<br>

下面再举个例子说明一下使用Enhancer代理接口：

```javascript
public class EnhancerInterfaceDemo {

	public static void main(String[] args) throws Exception {
		Enhancer enhancer = new Enhancer();
		enhancer.setInterfaces(new Class[]{SampleInterface.class});
		enhancer.setCallback((FixedValue) () -> "Doge say hello!");
		SampleInterface sampleInterface = (SampleInterface) enhancer.create();
		System.out.println(sampleInterface.sayHello("throwable-10086"));
		System.out.println(sampleInterface.sayHello("throwable-doge"));
		System.out.println(sampleInterface.toString());
		System.out.println(sampleInterface.getClass());
		System.out.println(sampleInterface.hashCode());
	}
}
```

输出结果和上一个例子一致。

<br>

## Callback

Callback，即回调。值得注意的是，它是一个标识接口(空接口，没有任何方法)，它的回调时机是生成的代理类的方法被调用的时候。也就是说，生成的代理类的方法被调用的时候，Callback的实现逻辑就会被调用。Enhancer通过`setCallback()`和`setCallbacks()`设置`Callback`，**设置了多个Callback实例将会按照设置的顺序进行回调**。CGLIB中提供的Callback的子类有以下几种：

- NoOp
- FixedValue
- InvocationHandler
- MethodInterceptor
- Dispatcher
- LazyLoader

<br>

### NoOp

NoOp，No Operation，也就是不做任何操作。这个回调实现只是简单地把方法调用委托给了被代理类的原方法(也就是`调用原始类的原始方法`)，不做任何其它的操作，所以不能使用在接口代理。

```javascript
public class NoOpDemo {

	public static void main(String[] args) throws Exception{
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(SampleClass.class);
		enhancer.setCallback(NoOp.INSTANCE);
		SampleClass sampleClass = (SampleClass) enhancer.create();
		System.out.println(sampleClass.sayHello("throwable"));
	}
}
```

输出结果：

```javascript
throwable say hello!
```

<br>

### FixedValue

FixedValue，Fixed Value，即固定值。

它提供了一个`loadObject()`方法，不过这个方法返回的不是代理对象，而是原方法调用想要的结果。也就是说，在这个Callback里面，看不到任何原方法的信息，也就没有调用原方法的逻辑，不管原方法是什么都只会调用`loadObject()`并返回一个固定结果。

需要注意的是，如果loadObject()方法的返回值并不能转换成原方法的返回值类型，那么会抛出类型转换异常(ClassCastException)。

最前面的Enhancer两个例子就是用FixedValue做分析的，这里不再举例。

<br>

### InvocationHandler

InvocationHandler全类名为`net.sf.cglib.proxy.InvocationHandler`，它的功能和JDK动态代理中的`java.lang.reflect.InvocationHandler`类似，提供了一个`Object invoke(Object proxy, Method method, Object[] objects)`方法。

需要注意的是：所有对invoke方法的参数proxy对象的方法调用都会被委托给同一个InvocationHandler，所以可能会`导致无限循环`(因为invoke中调用的任何原代理类方法，均会重新代理到invoke方法中)。举个简单的例子：

```javascript
public class InvocationHandlerDeadLoopDemo {

	public static void main(String[] args) throws Exception{
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(SampleClass.class);
		enhancer.setCallback(new InvocationHandler() {
			@Override
			public Object invoke(Object o, Method method, Object[] objects) throws Throwable {
				return method.invoke(o, objects);
			}
		});
		SampleClass sampleClass = (SampleClass) enhancer.create();
		System.out.println(sampleClass.sayHello("throwable"));
	}
}
```

上面的main方法执行后会直接爆栈，因为`method#invoke()`方法会重新调用InvocationHandler的invoke方法，形成死循环。

正确的使用例子如下：

```javascript
public class InvocationHandlerDemo {

	public static void main(String[] args) throws Exception {
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(SampleClass.class);
		enhancer.setCallback(new InvocationHandler() {
			@Override
			public Object invoke(Object o, Method method, Object[] objects) throws Throwable {
				if (!Objects.equals(method.getDeclaringClass(), Object.class) && Objects.equals(String.class, method.getReturnType())) {
					return String.format("%s say hello!", objects);
				}
				return "No one say hello!";
			}
		});

		SampleClass sampleClass = (SampleClass) enhancer.create();
		System.out.println(sampleClass.sayHello("throwable"));
	}
}
```

输出结果：

```javascript
throwable say hello!
```

<br>

### MethodInterceptor

MethodInterceptor，即方法拦截器，这是一个功能很强大的接口，它可以实现类似于AOP编程中的环绕增强（Around Advice）。

它只有一个方法`public Object intercept(Object obj,java.lang.reflect.Method method,Object[] args,MethodProxy methodProxy) throws Throwable`。设置了MethodInterceptor后，代理类的所有方法调用都会转而执行这个接口中的intercept方法而不是原方法。如果需要在intercept方法中执行原方法可以使用参数method基于代理实例obj进行反射调用，但是使用方法代理methodProxy效率会更高（反射调用比正常的方法调用的速度慢很多）。

`MethodInterceptor的生成效率不高，它的优势在于调用效率`，它需要产生不同类型的字节码，并且需要生成一些运行时对象(InvocationHandler就不需要)。

**注意**，在使用MethodProxy调用invokeSuper方法相当于通过方法代理直接调用原类的对应方法，如果调用MethodProxy的invoke会进入死循环导致爆栈，原因跟InvocationHandler差不多。

```javascript
public class MethodInterceptorDemo {

	public static void main(String[] args) throws Exception {
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(SampleClass.class);
		enhancer.setCallback(new MethodInterceptor() {
			@Override
			public Object intercept(Object obj, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
				System.out.println("Before invoking sayHello...");
				Object result = methodProxy.invokeSuper(obj, objects);
				System.out.println("After invoking sayHello...");
				return result;
			}
		});
		SampleClass sampleClass = (SampleClass) enhancer.create();
		System.out.println(sampleClass.sayHello("throwable"));
	}
}
```

输出结果：

```javascript
Before invoking sayHello...
After invoking sayHello...
throwable say hello!
```

这个例子就是Spring的AOP中的环绕增强(Around Advice)的简化版，这里没有改变原来的方法的行为，只是在方法调用前和调用后织入额外的逻辑。

<br>

### Dispatcher

Dispatcher，即分发器，提供一个方法`Object loadObject() throws Exception;`，同样地返回一个代理对象，这个对象同样可以代理原方法的调用。Dispatcher的`loadObject()`方法在每次发生对原方法的调用时都会被调用并返回一个代理对象来调用原方法。Dispatcher可以类比为Spring中的Prototype类型。

```javascript
public class DispatcherDemo {

	private static final AtomicInteger COUNTER = new AtomicInteger(0);

	public static void main(String[] args) throws Exception {
		Enhancer enhancer = new Enhancer();
		SampleInterfaceImpl impl = new SampleInterfaceImpl();
		enhancer.setInterfaces(new Class[]{SampleInterface.class});
		enhancer.setCallback(new Dispatcher() {
			@Override
			public Object loadObject() throws Exception {
				COUNTER.incrementAndGet();
				return impl;
			}
		});
		SampleInterface sampleInterface = (SampleInterface) enhancer.create();
		System.out.println(sampleInterface.sayHello("throwable-1"));
		System.out.println(sampleInterface.sayHello("throwable-2"));
		System.out.println(COUNTER.get());
	}

	private static class SampleInterfaceImpl implements SampleInterface{

		public SampleInterfaceImpl(){
			System.out.println("SampleInterfaceImpl init...");
		}

		@Override
		public String sayHello(String name) {
			return "Hello i am SampleInterfaceImpl!";
		}
	}
}
```

输出结果：

```javascript
SampleInterfaceImpl init...
Hello i am SampleInterfaceImpl!
Hello i am SampleInterfaceImpl!
2
```

计数器输出为2，印证了每次调用方法都会回调Dispatcher中的实例进行调用。

<br>

### LazyLoader

LazyLoader，即懒加载器，它只提供了一个方法`Object loadObject() throws Exception;`，loadObject()方法会在第一次被代理类的方法调用时触发，它返回一个代理类的对象，这个对象会被存储起来然后负责所有被代理类方法的调用，就像它的名字说的那样，一种lazy加载模式。如果被代理类或者代理类的对象的创建比较麻烦，而且不确定它是否会被使用，那么可以选择使用这种lazy模式来延迟生成代理。

LazyLoader可以类比为Spring中的Lazy模式的Singleton。

```javascript
public class LazyLoaderDemo {

	private static final AtomicInteger COUNTER = new AtomicInteger(0);

	public static void main(String[] args) throws Exception {
		Enhancer enhancer = new Enhancer();
		SampleInterfaceImpl impl = new SampleInterfaceImpl();
		enhancer.setInterfaces(new Class[]{SampleInterface.class});
		enhancer.setCallback(new LazyLoader() {
			@Override
			public Object loadObject() throws Exception {
				COUNTER.incrementAndGet();
				return impl;
			}
		});
		SampleInterface sampleInterface = (SampleInterface) enhancer.create();
		System.out.println(sampleInterface.sayHello("throwable-1"));
		System.out.println(sampleInterface.sayHello("throwable-2"));
		System.out.println(COUNTER.get());
	}

	private static class SampleInterfaceImpl implements SampleInterface{

		public SampleInterfaceImpl(){
			System.out.println("SampleInterfaceImpl init...");
		}

		@Override
		public String sayHello(String name) {
			return "Hello i am SampleInterfaceImpl!";
		}
	}
}
```

输出结果：

```javascript
SampleInterfaceImpl init...
Hello i am SampleInterfaceImpl!
Hello i am SampleInterfaceImpl!
1
```

计数器输出为1，印证了LazyLoader中的实例只回调了1次，这就是懒加载。

