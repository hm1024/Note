# 奇思妙想part1

1.接口是不是Object的子类?

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

2.静态方法和私有方法不能通过继承或其他方式重写其他版本

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

  

3.静态类

.静态类：我们对嵌套类使用static关键字。static不能用于最外层的类。静态的嵌套类和其它外层的类别无二致，嵌套只是为了方便打包。