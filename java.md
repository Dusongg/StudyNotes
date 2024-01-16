[TOC]



# 基本概念

- 编程语言：机器语言、汇编语言、高级语言

- 数据类型：
  - 八大基本类型
    - 四个整数型—— `byte` 、`short`、``int``、`long`
    - 两个浮点型—— `float`、`double`
    - 一种字符类型 —— `char`
    - 一种布尔型 —— `boolean`

  - 引用类型——  `int[]`、  对象等


# 1 类和对象

## 1.1 概念

- 封装、继承、多态

> 封装：结合对象属性和方法，对内隐藏实现细节，对外提供接口，保证数据安全性和可维护性
>
> 继承：子类调用父类方法属性，提高代码重用，实现类的层次结构
>
> 多态：通过方法的重写，继承体系中的不同对象调用同一函数做出不同的响应

- 什么是类和对象？



- 重写和重载的区别？

> **重载（`@Overload`）**意味着使用同样的名字但是不同的签名（参数、返回值）来定义多个方法。
>
> **重写（`@Override`）**意味着在子类中提供一个对方法的新的实现

## 1.2 抽象和接口

### 1.2.1 `instanceof`、`abstract`、`interface`、`implements`的使用

> - **一个类不能同时被 `final` 和 `abstract` 修饰** 
> - **构造函数、类方法不能声明为抽象方法**
> - **如果一个类包含抽象函数，那么这个类必须是抽象类**

```java

interface letter {
    public int c = 10;    //public static final
    public String toletter();    //public abstract   ，接口的方法不能有函数体
}

abstract class A {
    public void test_for_abstract() {
        int a = 10;
    }
}

class B extends A implements letter{    
    public String toletter() {
		return new Stirng("B");
    }
    public void test_for_abstract() {
        System.out.println("haha");
    }
}

class C implements letter{     
    public String toletter() {    //实现接口的方法
        System.out.println(c);
        return new String("C");
    }
}
public class learn {
    public static void test1() {
        Object[] o = {new B(), new C()};
        for (Object oo : o) {
            System.out.println(oo instanceof C);   //false   true
        }
    }
```



### 1.2.2 抽象类和接口的区别

![image-20231227162122021](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231227162122021.png)

### 1.2.3  Comparable接口  

```java
class A implements Comparable<A> {   
    private int a;
    public A(int x) {
        a = x;
    }
    @Override   //重写 接口的抽象函数
    public int compareTo(A o) {
        if (this.a > o.a) return 1;
        else if (this.a < o.a) return -1;
        else return 0;
    }
}
public class learn {
    public static void main(String[] args) {
        A a1 = new A(1);
        A a2 = new A(2);
        if (a1.compareTo(a2) == 1) {
            System.out.println("a1 > a2");
        } else {
            System.out.println("a1 <= a2");
        }
    }
}
```

## 1.3 基本类型和引用类型的区别

基本类型变量在内存中存储的是一个基本类型值，而**引用类型变量存储的是一个引用，它指向对象在内存中的位置**



## 1.4 使用this调用构造方法

```java
class A {
    public static int x = 10;
    public int y;
    A(int i, int j) {
        this.y = i + j;
    }
     A(int i) {
        this(i, 10);
    }    
}
```



## 1.5 基本类型与包装类

```java
Integer x1 = new Integer("123");

//创建一个新对象，并将指定字符串的值初始化该对象
Integer x2 = Integer.valueOf("23");

//将一个数值字符串转换为一个基本数值类型
Integer x3 = Integer.parseInt("23", 10);   //第二个参数为进制（不写默认是10进制）

int x4 = 3 + x3;    //自动开箱

Integer x5 =  3 + new Integer("234");    //自动装箱
```

## 1.6 Object类

### 1.6.1 `toString()`

```java
class B extends Object {   //等价于class B{}
    @Override
    public String toString() {
        return "hahah";
    }
}
public class learn {
    public static void main(String[] args) {
        System.out.println(new B().toString());   //output:hahah 
    }
}
```

### 1.6.2 `equals()`

```java
class Test {
    public int x;
    Test(int x) {
        this.x = x;
    }
    @Override
    public boolean equals(Object o) {
        return this.x == ((Test)o).x;
    }
}

class learn {
    public static void main(String[] args) {
        Test t1 = new Test(1);
        Test t2 = new Test(2);
        System.out.println(t1.equals(t2));     //false
    }
}
```



## 1.7 动态绑定 （类似C++）

- 引用变量的声明类型决定了编译时匹配哪个方法

```java
class Base {
    public void func() {
        System.out.println("Base--func()");
    }
}
class Derive1 extends Base {
    @Override
    public void func() {
        System.out.println("Derive1--func()");
    }
}
class Derive2 extends Base {
    @Override
    public void func() {
        System.out.println("Derive2--func()");
    }
}
class learn {
    public static void main(String[] args) {
        Base obj1 = new Derive1();   //Base动态绑定到Derive1
        Base obj2 = new Derive2();   //Base动态绑定到Derive2
    
        obj1.func();
        obj2.func();
    }
}
```



## 1.8 访问权限——public、protected、default、private

![qq_pic_merged_1703823047134](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/qq_pic_merged_1703823047134.jpg)

# 2 `String`

## 2.1 比较方法

![qq_pic_merged_1703809754191](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/qq_pic_merged_1703809754191.jpg)

```java
 //比较两个字符串
String s1 = new String("aaa");
String s2 = new String("aab");
System.out.println(s1.equals(s2));    //false
```

## 2.2 一些简单的方法

- `length`()
- `concat`() : 链接字符串

- `charAt`()

```java
System.out.println(s1.charAt(0));  //a
```

## 2.3 字符串转义

```java
//转义 : 反斜杠
String s = "This is a \"String\" class";
```

## 2.4 构建字符串 —— `StringBuilder`

- `append()` 、 `tostring()`....

```java
//java字符串是不可变的  -> 用StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 5; i++) {
    sb.append(i);
}
String ss = sb.toString();
System.out.println(ss);     //01234
```

## 2.5 其他类型转字符串（格式化字符串）

```java
//其他类型转字符串  （int, float, double）
int i = 10;
String s1 = String.format("%d", i);

String s2 = "" + i;
```

## 2.6 数字隐式转字符串

```java
System.out.println(1 + "wolcome " + 1 + 1);    //1wolcome 11
System.out.println(1 + "wolcome " + (1 + 1));	 //1wolcome 2
System.out.println(1 + "wolcome " + ('\u0001' + 1));	 //1wolcome 2
System.out.println(1 + "wolcome " + 'a' + 1);	 //1wolcome a1
```



## 2.7 字符串转整数

> 给你一个字符串 `date` ，按 `YYYY-MM-DD` 格式表示一个 [现行公元纪年法](https://baike.baidu.com/item/公元/17855) 日期。返回该日期是当年的第几天。
>
> **示例 1：**
>
> ```
> 输入：date = "2019-01-09"
> 输出：9
> 解释：给定日期是2019年的第九天。
> ```

```java

class Solution {
    public int dayOfYear(String date) {
        int year = Integer.parseInt(date.substring(0, 4));    //左闭右开区间
        //C++: int month = stoi(date.substr(5,2))
        int month = Integer.parseInt(date.substring(5, 7));
        int day = Integer.parseInt(date.substring(8));

        int[] amount = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
        if (year % 400 == 0 || (year % 4 == 0 && year % 100 != 0)) {
            ++amount[1];
        }

        int ans = 0;
        for (int i = 0; i < month - 1; ++i) {
            ans += amount[i];
        }
        return ans + day;
    }
}

```



# 3 数组

## 3.1 数组拷贝

- `Arrays.copyOfRange()`

```java
int[] arr = {1,2,3};
int[] arr2 = Arrays.copyOfRange(arr, 0, arr.length);
```

- `System.arraycopy()`

```java
int[] arr3 = new int[3];
System.arraycopy(arr, 0, arr3, 0, arr.length);
```

- `clone()`

```java
int[] a = {1, 2, 3}, aa = a.clone();    
aa[0] = 3;
System.out.println(Arrays.toString(a));   //1 2 3
System.out.println(Arrays.toString(aa));   //3 2 3
```



## 3.2 打印数组内容

```java
//一次性打印数组内容
System.out.println(Arrays.toString(arr));   
arr2[0] = 4;
System.out.println(Arrays.toString(arr2));	

```

## 3.3 数组大小和默认值

- **数组大小是属性，字符串长度是方法**

```java
String s = new String("asdfs");
int n = s.length();

int[] arr = new int[2];
int m = arr.length;
```

- 默认值

八大基本数据类型默认值分别为

- `byte` \ `short` \ `int` \ `long`    :`0`
- `char`: `'\u0000'`    
- `boolean`: `false`

## 3.4 Arrays类

- equal
- sort
- fill

```java
    int[] arr = {1,2,3}, arr2 = {1,2,3};
    if (Arrays.equals(arr, arr2)) {
        System.out.println("yes");   //yes
    }
    int[] arr3 = new int[3];
    Arrays.fill(arr3,2);
    System.out.println(Arrays.toString(arr3));   //222
```

## 3.5 可变长参数列表

```java
    public static double sum(double... doublearr) {
        double ret = 0.0;
        for (int i = 0; i < doublearr.length; i++) {
            ret += doublearr[i];
        }
        return ret;
    }
```

# 4 `Scanner`

```java
public class learn {
	public static void main (String[] args) {
        Scanner scaner = new Scanner(System.in);
        StringBuilder line = new StringBuilder();
        if (scaner.hasNextLine()) {    //用while会死循环
            line.append(scaner.nextLine());
        }
        System.out.printf(line.toString());
   	}
}   

```

# 5 `ArrayList`

## 5.1 增删查改

- 增：`add`
- 删：`remove`
- 查：`get`
- 改：`set`

```java
public class learn {
    public static void main(String[] args) {
        ArrayList<String> arr = new ArrayList<String>();
        for (int i = 0; i <= 5; i++) {
            String ss = String.format("%d", i);
            arr.add(String.format("%d", i));
        }
        System.out.println(arr.toString());   //012345

        System.out.println(arr.get(3));   //3

        arr.set(3, String.format("%d", 333));   //不能使用arr[3]
        System.out.println(arr.get(3));   //333

        arr.remove(3);
        System.out.println(arr.toString());   //01245
    }
}
```

## 5.2 排序

```java
Collections.sort(arr);
```

## 5.3 计算大小

```java
arr.size()
```

## 5.4 用数组初始化

```java
Integer[] arr = {1,23,5,5};
ArrayList<Integer> la = new ArrayList<>(Arrays.asList(arr));
```



# 6 异常

## 6.1 异常类型

- 免检异常：`Error`(系统错误)  、 `RuntimeException`(运行时异常)以及他们的子类
  - `Error`: 链接、虚拟机崩溃
  - `RuntimeException`：算数、引用null、超出范围、传递给方法的参数非法或不合适

```java
class learn {
    public static void main(String[] args) {
        try {
            Object o = null;
            System.out.println(o.toString());   
        }catch (RuntimeException e) {
            System.out.println(e);      //java.lang.NullPointerException
        }
    }
}
```



- 必检异常：所有其他异常

## 6.2 `throw`/`thorws`

- `throw`

```java
public void checkNumber(int num) {
  if (num < 0) {
    throw new IllegalArgumentException("Number must be positive");
  }
}
```

- `thorws`

> **throws** 关键字用于在方法声明中指定该方法可能抛出的异常。当方法内部抛出指定类型的异常时，该异常会被传递给调用该方法的代码，并在该代码中处理异常。
>
> 例如，下面的代码中，当 `readFile` 方法内部发生 `IOException` 异常时，会将该异常传递给调用该方法的代码。在调用该方法的代码中，必须捕获或声明处理 `IOException` 异常。

```java
public void readFile(String filePath) throws IOException {
  BufferedReader reader = new BufferedReader(new FileReader(filePath));
  String line = reader.readLine();
  while (line != null) {
    System.out.println(line);
    line = reader.readLine();
  }
  reader.close();
}
```

## 6.3 自定义异常

> 在 Java 中你可以自定义异常。编写自己的异常类时需要记住下面的几点。
>
> - 所有异常都必须是 `Throwable` 的子类。
> - 如果希望写一个检查性异常类，则需要继承 `Exception` 类。
>   - 如果你想写一个运行时异常类，那么需要继承 `RuntimeException` 类。

```java
class MyException extends Exception{
}
```



# 7 文件IO

## 7.1 `File`类

- File类是文件名及其目录路径的一个包装类

```java
File file = new File("./data.txt");
System.out.println(file.exists());   //true
```

## 7.2 `PrintWriter`类

- 用于创建一个文件并向文本文件写入数据

```java
public static void main(String[] args) throws Exception{
    File file = new File("./data.txt");
    System.out.println(file.exists());   //true

    try (PrintWriter writer = new PrintWriter(file)) {
        writer.print("haha");    //向data.txt文件中写入haha
    }
}
```

## 7.3 try-with-resources 处理多个资源

```java
import java.io.*;
import java.util.*;
class RunoobTest {
    public static void main(String[] args) throws IOException{
        try (Scanner scanner = new Scanner(new File("testRead.txt"));    //读
            PrintWriter writer = new PrintWriter(new File("testWrite.txt"))) {    //写
            while (scanner.hasNext()) {
                writer.print(scanner.nextLine());
            }
        }
    }
}
```

