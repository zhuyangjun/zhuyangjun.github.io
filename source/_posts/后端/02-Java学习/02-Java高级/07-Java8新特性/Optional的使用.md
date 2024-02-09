## 简介

从 Java 8 引入的一个很有趣的特性是 Optional 类。Optional 类主要解决的问题是臭名昭著的空指针异常（NullPointerException） —— 每个 Java 程序员都非常了解的异常。
本质上，这是一个包含有可选值的包装类，这意味着 Optional 类既可以含有对象也可以为空。

Optional 是 Java 实现函数式编程的强劲一步，并且帮助在范式中实现。但是 Optional 的意义显然不止于此。

## 类方法

### 创建Optional 类对象的方法

| 方法                     | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| Optional.of(T t)         | 创建一个 Optional 对象，参数 t 必须非空，如果你把 null 值作为参数传递进去，of() 方法会抛出 NullPointerException |
| Optional.empty()         | 创建一个空的 Optional 实例                                   |
| Optional.ofNullable(T t) | 创建一个 Optional 对象，如果参数 t 为非空，返回 Optional 描述的指定值，否则返回空的 Optional |

### 判断Optional容器是否包含对象

| 方法                                         | 作用                                                       |
| -------------------------------------------- | ---------------------------------------------------------- |
| boolean isPresent()                          | 如果值存在则方法会返回 true，否则返回 false。              |
| void ifPresent(Consumer<? super T> consumer) | 如果**值存在**则使用该值调用 consumer , 否则不做任何事情。 |

### 获取Optional容器对象

| 方法                                                         | 作用                                                         |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| T get()                                                      | 如果在这个 Optional 中包含这个值，返回值，否则抛出异常：`NoSuchElementException` |
| T orElse(T other)                                            | 如果存在该值，返回值， 否则返回 other。                      |
| T orElseGet(Supplier<? extends T> other)                     | 如果存在该值，返回值， 否则触发 other，并返回 other 调用的结果。 |
| <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) | 如果存在该值，返回包含的值，否则抛出由 Supplier 继承的异常   |

### 过滤转换值

| 方法                                                         | 作用                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Optional<T>   filter(Predicate<? super <T> predicate)        | 如果值存在，并且这个值匹配给定的 predicate，返回一个Optional用以描述这个值，否则返回一个空的Optional |
| <U> Optional<U>   flatMap(Function<? super T,Optional<U>> mapper) | 如果值存在，返回基于Optional包含的映射方法的值，否则返回一个空的Optional |
| <U>Optional<U>    map(Function<? super T,? extends U> mapper) | 如果有值，则对其执行调用映射函数得到返回值。如果返回值不为 null，则创建包含映射返回值的Optional作为map方法返回值，否则返回空Optional。 |

### 其他方法

| 方法                       | 作用                                      |
| -------------------------- | ----------------------------------------- |
| boolean equals(Object obj) | 判断其他对象是否等于 Optional。           |
| int hashCode()             | 返回存在值的哈希码，如果值不存在 返回 0。 |
| String toString()          | 返回一个Optional的非空字符串，用来调试    |

## 未使用Optional

我们从一个简单的用例开始。在 Java 8 之前，任何访问对象方法或属性的调用都可能导致 *NullPointerException*：

```text
String isocode = user.getAddress().getCountry().getIsocode().toUpperCase();
```

在这个小示例中，如果我们需要确保不触发异常，就得在访问每一个值之前对其进行明确地检查：

```java
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        Country country = address.getCountry();
        if (country != null) {
            String isocode = country.getIsocode();
            if (isocode != null) {
                isocode = isocode.toUpperCase();
            }
        }
    }
}
```

你看到了，这很容易就变得冗长，难以维护。

为了简化这个过程，我们来看看用 *Optional* 类是怎么做的。从创建和验证实例，到使用其不同的方法，并与其它返回相同类型的方法相结合，下面是见证 *Optional* 奇迹的时刻。

## **使用Optional **

### 1、**创建一个空的 Optional**

```java
@Test(expected = NoSuchElementException.class)
public void whenCreateEmptyOptional_thenNull() {
    Optional<User> emptyOpt = Optional.empty();
    emptyOpt.get();
}
//抛异常：NoSuchElementException
```

### 2、**使用 of()和 ofNullable() 方法创建包含值的 Optional**

两个方法的不同之处在于：

（1）、如果`of()` 方法的参数值为`null`，会抛出 `NullPointerException`，并没有完全摆脱 `NullPointerException`。因此，你应该**明确对象不为 `null`的时候使用 `of()`**

```java
@Test(expected = NullPointerException.class)
public void whenCreateOfEmptyOptional_thenNullPointerException() {
    User user = new User();
    //User user = null; //会报空指针异常
    Optional<User> opt = Optional.of(user);
}
```

（2）、如果对象即可能是 `null` 也可能是非 `null`，你就应该使用 `ofNullable()`方法

```java
Optional<User> opt = Optional.ofNullable(user);
```

## 访问 Optional 对象的值

从 `Optional` 实例中取回实际值对象的方法之一是使用 `get()` 方法：

```java
@Test
public void whenCreateOfNullableOptional_thenOk() {
    String name = "John";
    Optional<String> opt = Optional.ofNullable(name);

    assertEquals("John", opt.get());
}
```

不过，这个方法会在值为`null` 的时候抛出异常。要避免异常，你可以选择首先验证是否有值。

有以下两个方法验证是否有值：

（1）使用 `isPresent()` 方法检查是否有值

```java
@Test
public void whenCheckIfPresent_thenOk() {
    User user = new User("john@gmail.com", "1234");
    Optional<User> opt = Optional.ofNullable(user);
    assertTrue(opt.isPresent());

    assertEquals(user.getEmail(), opt.get().getEmail());
}
```

（2）使用 `ifPresent()` 方法检查是否有值，该方法除了执行检查，还接受一个*Consumer(消费者*) 参数，如果对象不是空的，就对执行传入的 Lambda 表达式：

```java
opt.ifPresent( u -> assertEquals(user.getEmail(), u.getEmail()));
```

这个例子中，只有 user 用户不为 null 的时候才会执行断言。

接下来，我们来看看提供空值的方法。

## 返回默认值

### `orElse()`

*Optional* 类提供了 API 用以返回对象值，或者在对象为空的时候返回默认值。

这里你可以使用的第一个方法是 `orElse()`，它的工作方式非常直接，如果有值则返回该值，否则返回传递给它的参数值：

```text
@Test
public void whenEmptyValue_thenReturnDefault() {
    User user = null;
    User user2 = new User("anna@gmail.com", "1234");
    User result = Optional.ofNullable(user).orElse(user2);

    assertEquals(user2.getEmail(), result.getEmail());
}
```

这里 *user* 对象是空的，所以返回了作为默认值的 *user2*。

如果对象的初始值不是 null，那么默认值会被忽略：

```text
@Test
public void whenValueNotNull_thenIgnoreDefault() {
    User user = new User("john@gmail.com","1234");
    User user2 = new User("anna@gmail.com", "1234");
    User result = Optional.ofNullable(user).orElse(user2);

    assertEquals("john@gmail.com", result.getEmail());
}
```

### `orElseGet()`

第二个同类型的 API 是 `orElseGet()` —— 其行为略有不同。这个方法会在有值的时候返回值，如果没有值，它会执行作为参数传入的 `Supplier(供应者)` 函数式接口，并将返回其执行结果：

```java
OrderEntity orderEntity = new OrderEntity();
orderEntity.setOrderSn("1");
OrderEntity orderEntity2 = new OrderEntity();
orderEntity2.setOrderSn("2");
OrderEntity orderEntity3 = Optional.ofNullable(orderEntity).orElseGet(() -> {
            return orderEntity2;
});
```

### `orElse()` 和`orElseGet()` 的不同之处

乍一看，这两种方法似乎起着同样的作用。然而事实并非如此。我们创建一些示例来突出二者行为上的异同。

我们先来看看对象为空时他们的行为：

```java
@Test
public void givenEmptyValue_whenCompare_thenOk() {
    User user = null
    logger.debug("Using orElse");
    User result = Optional.ofNullable(user).orElse(createNewUser());
    logger.debug("Using orElseGet");
    User result2 = Optional.ofNullable(user).orElseGet(() -> createNewUser());
}

private User createNewUser() {
    logger.debug("Creating New User");
    return new User("extra@gmail.com", "1234");
}
```

上面的代码中，两种方法都调用了 *createNewUser()* 方法，这个方法会记录一个消息并返回 *User* 对象。

代码输出如下：

```text
Using orElse
Creating New User
Using orElseGet
Creating New User
```

由此可见，**当对象为空而返回默认对象时，行为并无差异。**

我们接下来看一个类似的示例，但这里 *Optional* 不为空：

```java
@Test
public void givenPresentValue_whenCompare_thenOk() {
    User user = new User("john@gmail.com", "1234");
    logger.info("Using orElse");
    User result = Optional.ofNullable(user).orElse(createNewUser());
    logger.info("Using orElseGet");
    User result2 = Optional.ofNullable(user).orElseGet(() -> createNewUser());
}
```

这次的输出：

```text
Using orElse
Creating New User
Using orElseGet
```

这个示例中，两个`Optional` 对象都包含非空值，两个方法都会返回对应的非空值。不过，`orElse()` 方法仍然创建了 `User` 对象。**与之相反，`orElseGet()` 方法不创建 `User` 对象。**

在执行较密集的调用时，比如调用 Web 服务或数据查询，**这个差异会对性能产生重大影响**。

## 返回异常

除了 *orElse()* 和 *orElseGet()* 方法，Optional 还定义了 *orElseThrow()* API —— 它会在对象为空的时候抛出异常，而不是返回备选的值：

```java
@Test(expected = IllegalArgumentException.class)
public void whenThrowException_thenOk() {
     OrderEntity order = null;
     OrderEntity order1 = Optional.ofNullable(order).orElseThrow(() -> {
            throw new RRException("参数为空");
     });
}
```

这里，如果 `user`值为 null，会抛出`IllegalArgumentException`。

这个方法让我们有更丰富的语义，可以决定抛出什么样的异常，而不总是抛出`NullPointerException`。

现在我们已经很好地理解了如何使用 Optional，我们来看看其它可以对 *Optional* 值进行转换和过滤的方法。

## 转换值

有很多种方法可以转换 *Optional* 的值。我们从 `map()` 和 `flatMap()` 方法开始。

###  `map()` 

先来看一个使用 *map()* API 的例子：

```java
@Test
public void whenMap_thenOk() {
    //和stream流的map方法基本一致
	//map() 接受一个函数式接口，返回的还是Optional对象，只不过Optional的值可以转换成任意类型的【函数式接口返回啥就是啥】
    /**
    
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
    
    @FunctionalInterface
	public interface Function<T, R> {
     	 R apply(T t);
     }
    */
     Optional<String> s = Optional.ofNullable(orderEntity).map(order -> {
         return order.getOrderSn();
     });
    User user = new User("anna@gmail.com", "1234");
    String email = Optional.ofNullable(user)
      .map(u -> u.getEmail()).orElse("default@gmail.com");

    assertEquals(email, user.getEmail());
}
```

**`map()`对值应用(调用)作为参数的函数，然后将返回的值包装在 Optional 中。**这就使对返回值进行链试调用的操作成为可能 

 这里的下一环就是 *orElse()*。

###  `flatMap()` 

相比这下，`flatMap()` 也需要 函数 作为参数，并对值调用这个函数，然后直接返回结果。

下面的操作中，我们给 *User* 类添加了一个方法，用来返回 *Optional*：

```java
public class User {    
    private String position;

    public Optional<String> getPosition() {
        return Optional.ofNullable(position);
    }

    //...
}
```

既然 getter 方法返回 String 值的 *Optional*，你可以在对 *User 的 Optional* 对象调用 *flatMap()* 时，用它作为参数。其返回的值是解除包装的 String 值：

```java
@Test
public void whenFlatMap_thenOk() {
    User user = new User("anna@gmail.com", "1234");
    user.setPosition("Developer");
    String position = Optional.ofNullable(user)
      .flatMap(u -> u.getPosition()).orElse("default");

    assertEquals(position, user.getPosition().get());
}
```

## 过滤值

除了转换值之外，*Optional* 类也提供了按条件“过滤”值的方法。

**`filter()` 接受一个 `Predicate`参数**，返回测试结果为 true 的值。如果测试结果为 false，会返回一个空的 *Optional*。

来看一个根据基本的电子邮箱验证来决定接受或拒绝 *User(用户)* 的示例：

```java
@Test
public void whenFilter_thenOk() {
    User user = new User("anna@gmail.com", "1234");
    Optional<User> result = Optional.ofNullable(user)
      .filter(u -> u.getEmail() != null && u.getEmail().contains("@"));

    assertTrue(result.isPresent());
}
```

如果通过过滤器测试，*result* 对象会包含非空值。

## **Optional 类的链式方法**

为了更充分的使用 *Optional*，你可以链接组合其大部分方法，因为它们都返回相同类似的对象。

我们使用 *Optional* 重写最早介绍的示例。

首先，重构类，使其 getter 方法返回 *Optional* 引用：

User类：

```java
@Data
public class User {

    private String name;

    private String email;

    private Address address;
    
    public Optional<Address> getAddress() {
        return Optional.ofNullable(address);
    }
}
```

Address类

```java
@Data
public class Address {

    private Country country;

    public Optional<Country> getCountry() {
        return Optional.ofNullable(country);
    }
}
```

Country类

```java
@Data
public class Country {

    private String cName;

    private String cCode;
}
```

### 链式调用

```java
User user = new User();
Optional<User> user1 = Optional.ofNullable(user);
String cName = user1.flatMap(u -> u.getAddress())
                    .flatMap(a -> a.getCountry())
                    .map(c -> c.getCName())
                    .orElse("123");
log.info("cName={}",cName);
```

## **Optional 应该怎样用？**

在使用 *Optional* 的时候需要考虑一些事情，以决定什么时候怎样使用它。

重要的一点是 **Optional** **不是** **Serializable**。因此，它不应该用作类的字段。

如果你需要序列化的对象包含 *Optional* 值，[Jackson 库](https://link.zhihu.com/?target=https%3A//stackify.com/java-xml-jackson/)支持把 *Optional* 当作普通对象。也就是说，*Jackson* 会把空对象看作 *null*，而有值的对象则把其值看作对应域的值。这个功能在 [jackson-modules-java8](https://link.zhihu.com/?target=https%3A//github.com/FasterXML/jackson-modules-java8) 项目中。

它在另一种情况下也并不怎么有用，就是在将其类型用作方法或构建方法的参数时。这样做会让代码变得复杂，完全没有必要：

```text
User user = new User("john@gmail.com", "1234", Optional.empty());
```

使用重载方法来处理非要的参数要容易得多。

**Optional主要用作返回类型。**在获取到这个类型的实例后，如果它有值，你可以取得这个值，否则可以进行一些替代行为。

*Optional* 类有一个非常有用的用例，就是将其与流或其它返回 *Optional* 的方法结合，以**构建流畅的API**。

我们来看一个示例，使用 *Stream* 返回 *Optional* 对象的 *findFirst()* 方法：

```java
@Test
public void whenEmptyStream_thenReturnDefaultOptional() {
    List<User> users = new ArrayList<>();
    User user = users.stream().findFirst().orElse(new User("default", "1234"));

    assertEquals(user.getEmail(), "default");
}
```

## 总结

*Optional* 是 Java 语言的有益补充 —— 它旨在减少代码中的 *NullPointerExceptions*，虽然还不能完全消除这些异常。

它也是精心设计，自然融入 Java 8 函数式支持的功能。

总的来说，这个简单而强大的类有助于创建简单、可读性更强、比对应程序错误更少的程序。
