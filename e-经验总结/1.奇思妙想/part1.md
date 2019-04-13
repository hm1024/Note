# 奇思妙想part1

1. [接口是不是Object的子类](#接口是不是Object的子类)
2. [静态方法和私有方法不能通过继承或其他方式重写其他版本](#静态方法和私有方法不能通过继承或其他方式重写其他版本)
3. [静态类](#静态类)
4. [java编译器和java虚拟机的关系？](#java编译器和java虚拟机的关系)
5. [什么是运行时异常和连接时异常?](#什么是运行时异常和连接时异常)


### 接口是不是Object的子类

答：接口是Object的子类

```java
interface MyInterface{
    public void show();
    @Override
    String toString();
}
public class MyDemo {
	public static void main(String[] args) {
		String c =new MyInterface() {
			@Override
			public void show() {
				System.out.println("ddddddddd");
			}
			@Override
			public String toString() {
				// TODO Auto-generated method stub
				return "MyDemo";
			}
		}.toString();
		System.out.println(c);
	}
}
```

### 静态方法和私有方法不能通过继承或其他方式重写其他版本

* 静态方法

  ```java
  class A{
  	public static void say() {
  		System.out.println("我是A");
  	}
  	public  void eat() {
  		System.out.println("我吃A蛋糕");
  	}
  }
  class B extends A{
  	public static void sya() {
  		System.out.println("我是B");
  	}
  	public  void eat() {
  		System.out.println("我吃B蛋糕");
  	}
  }
  public class TestStatic {
  	public static void main(String[] args) {
  		A a = new B();
  		a.say();
  		a.eat();
  	}
  }/*OutPut:
  我是A
  我吃B蛋糕
  */
  ```

* 私有方法

  因为私有方法只有在私有方法所属的类中才能被访问

  ```java
  class A{
  	public void publicMethod(String str) {
  		System.out.println("publicMethos"+str);
  	}
  	private void privateMethod(String str) {
  		System.out.println("publicMethos"+str);
  	}
  }
  public class MyTest {
  	public static void main(String[] args) {
  		A a = new A();
  		a.publicMethod("方法调用");
  		//在类外无法访问私有方法
  		//The method privateMethos() from the type A is not visible
  		a.privateMethos();
  	}
  }
  ```

  
### 静态类

静态类：我们对嵌套类使用static关键字。static不能用于最外层的类。静态的嵌套类和其它外层的类别无二致，嵌套只是为了方便打包。

### java编译器和java虚拟机的关系

答：  java的编译器，或者说jdk，是用来将源码编译成class字节码的，是java的开发环境；虚拟机就是装有jre的可以运行class字节码的东东，可以是手机、电脑、和其他，只要能安装上java的运行环境jre，就能在其上面运行class，这就构成了一个jvm，java虚拟机，是java的运行环境！！

另外，两者分开的，但是jdk上自带有jre，因为要开发java的话是必须有jdk和jre的；如果纯粹只要能运行java程序的话，就只要安装jre就好了！！

jdk：Java Development Kit
jre：Java Runtime Environment
jvm：Java Virtual Machine  

### 什么是运行时异常和连接时异常

* 运行时异常（RuntimeException）也称作未检测的异常（unchecked exception），通俗点来说，**运行时异常就是只要代码不运行到这一行就不会有问题**。RuntimeException是所有可以在运行时抛出的异常的父类。一个方法除要捕获异常外，如果它执行的时候可能会抛出RuntimeException的子类，那么它就不需要用throw语句来声明抛出的异常。 例如：NullPointerException，ArrayIndexOutOfBoundsException，等等 。
* 连接时异常（也称受检查异常（checked exception））**即使会导致连接时异常的代码放在 一条无法执行到的分支路径上，类加载时（Java的连接过程不在编译阶段，而在类加载阶段）也照样会抛出异常。** 通过throws语句或者try{}cathch{} 语句块来处理检测异常。编译器会分析哪些异常会在执行一个方法或者构造函数的时候抛出。  

