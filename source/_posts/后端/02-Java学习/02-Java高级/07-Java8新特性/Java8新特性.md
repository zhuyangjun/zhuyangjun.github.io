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

## 二、Stream API

#### 一、概述

Stream 是 Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。使用Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询。也可以使用 Stream API 来并行执行操作。简而言之，**Stream API 提供了一种高效且易于使用的处理数据的方式。**

特点：

        1 . 不是数据结构，不会保存数据。
    
        2. 不会修改原来的数据源，它会将操作后的数据保存到另外一个对象中。（保留意见：毕竟peek方法可以修改流中元素）
    
        3. 惰性求值，流在中间处理过程中，只是对操作进行了记录，并不会立即执行，需要等到执行终止操作的时候才会进行实际的计算。

Stream API的理解：

1. Stream关注的是对数据的运算，与CPU打交道。集合关注的是数据的存储，与内存打交道

2. java8提供了一套api,使用这套api可以对内存中的数据进行过滤、排序、映射、归约等操作。类似于sql对数据库中表的相关操作。

#### 二、分类

![image-20221116094643077](C:\Users\zhu.yangjun\AppData\Roaming\Typora\typora-user-images\image-20221116094643077.png)

无状态：指元素的处理不受之前元素的影响；

有状态：指该操作只有拿到所有元素之后才能继续下去。

非短路操作：指必须处理所有元素才能得到最终结果；

短路操作：指遇到某些符合条件的元素就可以得到最终结果，如 A || B，只要A为true，则无需判断B的结果。

#### 三、具体用法

##### 3.1、Stream的使用流程：

① Stream的实例化

② 一系列的中间操作（过滤、映射、...)

③ 终止操作

> 使用流程的注意点：
>
> 1、一个中间操作链，对数据源的数据进行处理
>
> 2、一旦执行终止操作，就执行中间操作链，并产生结果。之后，不会再被使用

##### 3.2、流的常用创建方法

**1.1、使用Collection下的 `stream() `和` parallelStream()` 方法**

 ```java
  List<String> list = new ArrayList<>();
  Stream<String> stream = list.stream();//获取一个顺序流
  Stream<String> parallelStream = list.parallelStream();//获取一个并行流
 ```

**1.2、 使用Arrays 中的` stream() `方法，将数组转成流**

   ```java
Integer[] nums = new Integer[10];
Stream<Integer> stream = Arrays.stream(nums);
   ```

1.3、使用Stream中的静态方法：`of()、iterate()、generate()`

   ```java
Stream<Integer> stream = Stream.of(1,2,3,4,5,6);
 
Stream<Integer> stream2 = Stream.iterate(0, (x) -> x + 2).limit(6);
stream2.forEach(System.out::println); // 0 2 4 6 8 10
 
Stream<Double> stream3 = Stream.generate(Math::random).limit(2);
stream3.forEach(System.out::println);
   ```

1.4、使用` BufferedReader.lines()` 方法，将每行内容转成流

```java
BufferedReader reader = new BufferedReader(new FileReader("F:\\test_stream.txt"));
Stream<String> lineStream = reader.lines();
lineStream.forEach(System.out::println);
```

1.5、使用 `Pattern.splitAsStream() `方法，将字符串分隔成流

```java
Pattern pattern = Pattern.compile(",");
Stream<String> stringStream = pattern.splitAsStream("a,b,c,d");
stringStream.forEach(System.out::println);
```

##### 3.3、流的中间操作

###### 3.3.1、筛选与切片

`filter：`过滤流中的某些元素。filter() 方法接收的是一个 Predicate（Java 8 新增的一个函数式接口，接受一个输入参数返回一个布尔值结果）类型的参数，因此，我们可以直接将一个 Lambda 表达式传递给该方法。
`limit(n)：`获取n个元素
`skip(n)：`跳过n元素，配合`limit(n)`可实现分页
`distinct：`通过流中元素的` hashCode() `和 `equals() `去除重复元素

```java
Stream<Integer> stream = Stream.of(6, 4, 6, 7, 3, 9, 8, 10, 12, 14, 14);
 
Stream<Integer> newStream = stream.filter(s -> s > 5) //6 6 7 9 8 10 12 14 14
        .distinct() //6 7 9 8 10 12 14
        .skip(2) //9 8 10 12 14
        .limit(2); //9 8
newStream.forEach(System.out::println);
//例二：
Integer[] arr = new Integer[]{1,2,3,4,2,2,3,4};
Stream<Integer> stream = Arrays.stream(arr).filter(r -> {
     if(r%2==0){
        return true;
     }
        return false;
});
```

###### 3.3.2、映射

`map`：接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
`flatMap`：接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流

```java
List<String> list = Arrays.asList("a,b,c", "1,2,3");
 
//将每个元素转成一个新的且不带逗号的元素
Stream<String> s1 = list.stream().map(s -> s.replaceAll(",", ""));
s1.forEach(System.out::println); // abc  123
 
Stream<String> s3 = list.stream().flatMap(s -> {
    //将每个元素转换成一个stream
    String[] split = s.split(",");
    Stream<String> s2 = Arrays.stream(split);
    return s2;
});
s3.forEach(System.out::println); // a b c 1 2 3
```

###### 3.3.3、排序

`sorted()`：自然排序，流中元素需实现`Comparable`接口
`sorted(Comparator com)`：定制排序，自定义`Comparator`排序器 

```java
List<String> list = Arrays.asList("aa", "ff", "dd");
//String 类自身已实现Compareable接口
list.stream().sorted().forEach(System.out::println);// aa dd ff
 
Student s1 = new Student("aa", 10);
Student s2 = new Student("bb", 20);
Student s3 = new Student("aa", 30);
Student s4 = new Student("dd", 40);
List<Student> studentList = Arrays.asList(s1, s2, s3, s4);
 
//自定义排序：先按姓名升序，姓名相同则按年龄升序
studentList.stream().sorted(
        (o1, o2) -> {
            if (o1.getName().equals(o2.getName())) {
                return o1.getAge() - o2.getAge();
            } else {
                return o1.getName().compareTo(o2.getName());
            }
        }
).forEach(System.out::println);
```

###### 3.3.4、消费

`peek：`如同于`map`，能得到流中的每一个元素。但`map`接收的是一个`Function`表达式，有返回值；而`peek`接收的是`Consumer`表达式，没有返回值。

## 三、函数式接口

## 四、Optional类

## 五、Date Time API

## 六、方法引用

## 七、默认方法



