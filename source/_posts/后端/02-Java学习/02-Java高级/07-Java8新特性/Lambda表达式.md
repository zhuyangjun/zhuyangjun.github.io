## 一、Lambda表达式

### 1、lambda表达式的组成

形似如下：

```java
(a,b) -> Integer.compare(a,b);
左边 -> 右边
```

- `->` 被称为lambda操作符或箭头操作符

- `左边`：lambda形参列表（其实就是接口中的抽象方法的形参列表）
- `右边`：lambda体 （其实就是重写的抽象方法的方法体）

### 2、lambda表达式的使用

#### 2.1、格式一

> 无参无返回值

```java
	@Test
    public void test01(){
        Runnable runnable1 = new Runnable(){
            @Override
            public void run() {
                System.out.println("hello");
            }
        };
        runnable1.run();
    }
    //lambda形式
    @Test
    public void test02(){
        Runnable runnable1 = () -> System.out.println("hello");
        runnable1.run();
    }
```

#### 2.2、格式二

> 有一个参数但无返回值

```java
	@Test
	public void test01(){
        Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };
        consumer.accept("yzj");
    }
    //lambda形式
    @Test
    public void test02(){
        Consumer<String> consumer = (String s) -> System.out.println(s);
        consumer.accept("hhhh");
    }
```

#### 2.3、格式三

> 数据类型可以省略，由编译器去推断出，称为“类型推断”

```java
	@Test
	public void test01(){
        Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };
        consumer.accept("yzj");
    }
    //lambda形式 ;数据类型可以省略
    @Test
    public void test02(){
        Consumer<String> consumer = (s) -> System.out.println(s);
        consumer.accept("hhhh");
    }
```

#### 2.4、格式四

> lambda若只需要一个参数时，参数的小括号可以省略

```java
@Test
	public void test01(){
        Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };
        consumer.accept("yzj");
    }
    //lambda形式 ;参数的小括号可以省略
    @Test
    public void test02(){
        Consumer<String> consumer = s -> System.out.println(s);
        consumer.accept("hhhh");
    }
```

#### 2.5、格式五

> lambda需要两个或以上的参数，多条执行语句，并且可以有返回值

```java
 	@Test
    public void test01(){
        Comparator<Integer> comparator = new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                System.out.println(o1);
                System.out.println(o2);
                return Integer.compare(o1,o2);
            }
        };
        int compare = comparator.compare(23, 24);
        System.out.println(compare);
    }
    //lambda形式
    @Test
    public void test02(){
        Comparator<Integer> comparator = (o1,o2) -> {
          System.out.println(o1);
          System.out.println(o2);
          return Integer.compare(o1,o2);
        };
        int compare = comparator.compare(33, 22);
        System.out.println(compare);
    }
```

#### 2.6、格式六

> 当lambda体只有一条语句时，return与大括号若有，都可以省略

```java
@Test
    public void test01(){
        Comparator<Integer> comparator = new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return Integer.compare(o1,o2);
            }
        };
        int compare = comparator.compare(23, 24);
        System.out.println(compare);
    }
    //lambda形式	
@Test
    public void test03(){
        Comparator<Integer> comparator = (o1,o2) -> Integer.compare(o1,o2);
        int compare = comparator.compare(33, 22);
        System.out.println(compare);
    }
```

### 3、 总结

`lambda`接口的实质：作为函数式接口的实例,关键是这个匿名内部类的简化 和 省略。

## 二、方法引用

### 概念

**方法引用使用一对冒号`::`，通过方法的名字来指向一个方法。**

方法引用可以使语言的构造更紧凑简洁，减少冗余代码。

当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用！方法引用可以看做是Lambda表达式深层次的表达。换句话说，方法引用就是Lambda表达式，也就是函数式接口的一个实例，通过方法的名字来指向一个方法，可以认为是Lambda表达式的一个语法糖。

### 使用情况： 

**①、对象`::`实例方法名**

**②、类`::`静态方法名**

**③、类`::`实例方法名**

### 要求：

1、要求接口中的抽象方法的形参列表和返回值类型与方法引用的方法的形参列表和返回值类型相同！（针对于情况1和情况2）

2、当函数式接口方法的第一个参数是需要引用方法的调用者，并且第二个参数是需要引用方法的参数(或无参数)时：ClassName::methodName（针对于情况3）

### 使用建议：

如果给函数式接口提供实例，恰好满足方法引用的使用情境，大家就可以考虑使用方法引用给函数式接口提供实例。如果大家不熟悉方法引用，那么还可以使用lambda表达式。

### 使用举例：

#### **①、对象 :: 实例方法**

```java
//Consumer中的void accept(T t)
//PrintStream中的void println(T t)
@Test
public void test1() {
	Consumer<String> con1 = str -> System.out.println(str);
	con1.accept("北京");
    System.out.println("*******************");
    //等价于下面的写法
	PrintStream ps = System.out;
	Consumer<String> con2 = ps::println;
	con2.accept("beijing");
}
```


```java
//Supplier中的T get()
//Employee中的String getName()
@Test
public void test2() {
	Employee emp = new Employee(1001,"Tom",23,5600);
    Supplier<String> sup1 = () -> emp.getName();
	System.out.println(sup1.get());

	System.out.println("*******************");
	Supplier<String> sup2 = emp::getName;
	System.out.println(sup2.get());
}
```

#### **②、类 :: 静态方法**

```java
//Comparator中的int compare(T t1,T t2)
//Integer中的int compare(T t1,T t2)
@Test
public void test3() {
	Comparator<Integer> com1 = (t1,t2) -> Integer.compare(t1,t2);
	System.out.println(com1.compare(12,21));
    System.out.println("*******************");

	Comparator<Integer> com2 = Integer::compare;
	System.out.println(com2.compare(12,3));
}
```

```java
//Function中的R apply(T t)
//Math中的Long round(Double d)
@Test
public void test4() {
	Function<Double,Long> func = new Function<Double, Long>() {
		@Override
		public Long apply(Double d) {
			return Math.round(d);
		}
	};
    System.out.println("*******************");

	Function<Double,Long> func1 = d -> Math.round(d);
	System.out.println(func1.apply(12.3));

	System.out.println("*******************");

	Function<Double,Long> func2 = Math::round;
	System.out.println(func2.apply(12.6));
}
```

#### ③、类 :: 实例方法  (难度)

```java
// Comparator中的int comapre(T t1,T t2)
// String中的int t1.compareTo(t2)
@Test
public void test5() {
	Comparator<String> com1 = (s1,s2) -> s1.compareTo(s2);
	System.out.println(com1.compare("abc","abd"));

	System.out.println("*******************");
	
	Comparator<String> com2 = String :: compareTo;
	System.out.println(com2.compare("abd","abm"));

}
```

```java
//BiPredicate中的boolean test(T t1, T t2);
//String中的boolean t1.equals(t2)
@Test
public void test6() {
	BiPredicate<String,String> pre1 = (s1,s2) -> s1.equals(s2);
	System.out.println(pre1.test("abc","abc"));

	System.out.println("*******************");
	BiPredicate<String,String> pre2 = String :: equals;
	System.out.println(pre2.test("abc","abd"));

}
```

```java
// Function中的R apply(T t)
// Employee中的String getName();
@Test
public void test7() {
	Employee employee = new Employee(1001, "Jerry", 23, 6000);


	Function<Employee,String> func1 = e -> e.getName();
	System.out.println(func1.apply(employee));
	
	System.out.println("*******************");
	
	Function<Employee,String> func2 = Employee::getName;
	System.out.println(func2.apply(employee));
}
```








