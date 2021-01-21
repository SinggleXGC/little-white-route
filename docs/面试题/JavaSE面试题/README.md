# JavaSE面试题

## 自增

### Q1

请问下面代码执行之后结果是什么？

```java
public static void main(String[] args) {
    int i = 1;
    i = i++;
    int j = i++;
    int k = i + ++i * i++;
    System.out.println("i=" + i);
    System.out.println("j=" + j);
    System.out.println("k=" + k);
}
```

答案：

```
i=4
j=1
k=11
```

分析步骤如下：

1. `int i = 1;`此时局部变量 i = 1

2. `i = i++;`此时先执行i++，i为1压入操作栈中，再自增，此时局部变量为2。然后执行赋值操作，将操作栈中的i=1赋值给局部变量i=2，此时i=1。

3. 同理，`int j = i++;`得到 j = 1。此时局部变量i=2。

4. `int k = i+ ++i * i++;`。

   先遇到i，将i=2入栈。

   `++i`，此时局部变量i为3，入栈。

   `i++`，此时i=3,入栈。然后i再自增，此时局部变量i=4。

   `++i * i++`，将栈上两个数弹出相乘，3*3=9，9入栈。

   此时栈中有两个操作数：2 和 9。k = 2 + 9 = 11。

5. 此时得出`i=4, j=1, k=11`。



## 类初始化和实例初始化

### Q1

下面代码的输出结果：

```java
public class Father{
	private int i = test();
	private static int j = method();
	
	static{
		System.out.print("(1)");
	}
	Father(){
		System.out.print("(2)");
	}
	{
		System.out.print("(3)");
	}
	
	
	public int test(){
		System.out.print("(4)");
		return 1;
	}
	public static int method(){
		System.out.print("(5)");
		return 1;
	}
}
```

```java
public class Son extends Father{
	private int i = test();
	private static int j = method();
	static{
		System.out.print("(6)");
	}
	Son(){
		System.out.print("(7)");
	}
	{
		System.out.print("(8)");
	}
	public int test(){
		System.out.print("(9)");
		return 1;
	}
	public static int method(){
		System.out.print("(10)");
		return 1;
	}
	public static void main(String[] args) {
		Son s1 = new Son();
		System.out.println();
		Son s2 = new Son();
	}
}
```

答案：

```
(5)(1)(10)(6)(9)(3)(2)(9)(8)(7)
(9)(3)(2)(9)(8)(7)
```

分析结果：

1. 首先，main方法所在的类会初始化。子类的初始化导致父类的初始化。所以父类、子类初始化输出结果为

   ```
   (5)(1)(10)(6)
   ```

   这个结果即便我们不写下述代码也会出现(因为main方法所在的类会初始化)

   ```java
   Son s1 = new Son();
   System.out.println();
   Son s2 = new Son();
   ```

2. 接着执行`Son s1 = new Son();`

   开始实例化，输出结果(因为父类、子类静态初始化已经完成，所以不再需要初始化)

   注：父类的test方法被重载了，会调用的是子类的方法。

   ```
   (9)(3)(2)(9)(8)(7)
   ```

3. 然后`Son s2 = new Son();`又实例化

   ```
   (9)(3)(2)(9)(8)(7)
   ```



## 方法的参数传递机制

### Q1

程序输出结果是什么？

```java
public class Exam4 {
	public static void main(String[] args) {
		int i = 1;
		String str = "hello";
		Integer num = 200;
		int[] arr = {1,2,3,4,5};
		MyData my = new MyData();
		
		change(i,str,num,arr,my);
		
		System.out.println("i = " + i);
		System.out.println("str = " + str);
		System.out.println("num = " + num);
		System.out.println("arr = " + Arrays.toString(arr));
		System.out.println("my.a = " + my.a);
	}
	public static void change(int j, String s, Integer n, int[] a,MyData m){
		j += 1;
		s += "world";
		n += 1;
		a[0] += 1;
		m.a += 1;
	}
}
class MyData{
	int a = 10;
}
```

答案：

```
i = 1
str = hello
num = 200
arr = [2, 2, 3, 4, 5]
my.a = 11
```

解析：

1. i是基本数据类型，传递的是数据值。值不会被改变为1
2. str是String类型，修改后会重新在常量池生成helloworld常量，并将s指向helloword的地址。但是str变量指向的地址还是hello
3. num是包装类，同String类型
4. arr和my变量都是引用类型，会被改变



## 递归与迭代

### Q1

编程题：有n步阶梯，一次只能上1步或2步，共有几种走法？

递归

```java
public class Step {

  public int countStepMethod(int n) throws Exception {
    if (n<1) throw new Exception("n cannot lower than 1");
    if (n==1 || n==2) return n;

    return countStepMethod(n-1) + countStepMethod(n-2);

  }

}
```



迭代

```java
public int countStepMethod(int n) throws Exception {
    if (n<1) throw new Exception("n cannot lower than 1");
    if (n==1 || n==2) return n;

    int one = 1;
    int two = 2;
    int count = -1;
    for (int i=3; i<=n; i++) {
        count = one + two;
        one = two;
        two = count;
    }
    return count;
}
```



