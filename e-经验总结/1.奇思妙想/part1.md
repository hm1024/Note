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

