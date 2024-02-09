### 一、数组转List的四种方式总结

#### 1、Arrays.asList()

即 Java.util 提供的 Arrays 中的 asList 方法，可以直接将一般的字符串数组以及包装类后的结果做直接转换。

需要注意的是 ：通过这种方式得到的List 不能执行增删操作，否则会抛出java.lang.UnsupportedOperationException 异常，即不支持操作的异常，一般的获取get, size等可以照常使用

原因 : Arrays.asList(str)返回值是java.util.Arrays类中一个私有静态内部类 java.utiil.Arrays.Arraylist,不是我们平时用的java.util.ArrayList()

适用于 只转换后作为读取的目标，不能用于增删操作

示例：

```java
// 引用类型
        Integer[] cardNumberArray = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
        String[] strings = {"dd", "aa", "545", "ee"};
        // 基本类型
        int[] cardNumberArrayInt = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};

        // 元素转换成功
        List<Integer> integers = Arrays.asList(cardNumberArray);
        List<String> list = Arrays.asList(strings);
        // 出现了基本类型
        List<int[]> ints = Arrays.asList(cardNumberArrayInt);

        // 执行了添加操作 结果出错
        integers.add(15);
```

同时可以看的出 上面如果直接使用了 int[] 这种基本类型， 转换时并不是我们预期的 int 而且把 int[] 认为了是一个泛型中的基本类型，泛型中不支持基本数据类型，所以如果非要使用尽量转为包装类，除非只做基本的运算。

asList 返回的是一个视图，简单来说只能看和读取，不能执行其他操作

总的来说这种方式泛用性不是很强

#### 2、ArrayList 构造方法

将第一种 **Arrays.asList** 的返回值作为 arrayList 够构造方法的入参，即可构造出可以进行增删操的正常 list

示例：

```java
// 引用类型
        Integer[] cardNumberArray = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
        String[] strings = {"dd", "aa", "545", "ee"};

        List<Integer> arrayList1 = new ArrayList<>(Arrays.asList(cardNumberArray));
        List<String> arrayList2 = new ArrayList<>(Arrays.asList(strings));
```

**适用于** 数组转为需要执行增删等正常操作的 list,同时数据量不是太大

#### 3、Collections.addAll

通过 Collections.addAll(要赋值的list, 原有list)；

同样是 JDK 自带的 Collections, 先创建一个对应的 list 并赋予原有数组长度，然后给与赋值Collections.addAll() 会将数组中的元素转为二进制，然后添加到List中。这种方法 **效率相对较高**

示例：

```java
		// 引用类型
        Integer[] cardNumberArray = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
      
        List arrayList = new ArrayList<>(cardNumberArray.length);
        Collections.addAll(arrayList, cardNumberArray);
```

#### 4、 JDK8 特性 Stream

JDK 1.8 支持的 基本类型转换 int[],long[],double[]

不支持的 short[ ],byte[ ],char[]

示例：

```java
// 基本类型
        int[] cardNumberArrayInt = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
        List<Integer> collect1 = Arrays.stream(cardNumberArrayInt).boxed().collect(Collectors.toList());
```

同样的引用类型也可以使用这种方式：

```java
Integer[] cardNumberArray = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
List<Integer> collect = Arrays.stream(cardNumberArray).collect(Collectors.toList());
```

引用类型与基础类型相比 少了 boxed 这一过程，是一个转包装类的过程。

### 二、List的remove()方法陷阱以及性能优化

Java List在进行remove（）方法时通常容易踩坑，主要有以下几点

> 循环时：问题在于，删除某个元素后，因为删除元素后，后面的元素都往前移动了一位，而你的索引+1，所以实际访问的元素相对于删除的元素中间间隔了一位。

#### 几种常见方法

##### 1、错误的处理方式

1.使用for循环不进行额外处理时
```java
//错误的方法
for(int i=0;i<list.size();i++) {
 	if(list.get(i)%2==0) {
  		list.remove(i);
 	}
}
```
2.使用foreach循环
```java
for(Integer i:list) {
    if(i%2==0) {
      list.remove(i);
    }
} 
```
会报错、抛出异常：java.util.ConcurrentModificationException；
foreach的本质是使用迭代器实现，每次进入for (Integer i:list) 时，会调用ListItr.next()方法；
继而调用checkForComodification()方法， checkForComodification()方法对操作集合的次数进行了判断，如果当前对集合的操作次数与生成迭代器时不同，抛出异常

```java
public E next() {
 checkForComodification();
 if (!hasNext()) {
   throw new NoSuchElementException();
 }
  lastReturned = next;
 next = next.next;
 nextIndex++;
 return lastReturned.item;
 }
 // checkForComodification()方法对集合遍历前被修改的次数与现在被修改的次数做出对比
final void checkForComodification() {
   if (modCount != expectedModCount) {
      throw new ConcurrentModificationException();
   }         
}
```

##### 2、正确的处理方式

1.使用for循环，并且同时改变索引；

```java
for(int i=0;i<list.size();i++) {
 	if(list.get(i)%2==0) {
  		list.remove(i);
  		i--;//在元素被移除掉后，进行索引后移
 	}
}
//使用for循环，倒序进行:
for(int i=list.size()-1;i>=0;i--) {
 if(list.get(i)%2==0) {
  list.remove(i);
 }
}
```

2.使用迭代器方法（正确,推荐）

```java
//正确，并且推荐的方法
Iterator<Integer> itr = list.iterator();
while(itr.hasNext()) {
 if(itr.next()%2 ==0)
  itr.remove();
}
```

