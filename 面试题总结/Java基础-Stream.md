## Stream

#### 1. Java 8 Stream

Java 8 API添加了一个新的抽象称为流 Stream，可以让你以一种声明的方式处理数据。

Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java <font color="red">**集合**</font>运算和表达的高阶抽象。

Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。

这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。

元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

#### 2. 简单了解 Stream()

Stream（流）是一个来自数据源的元素队列并支持聚合操作

- 元素是特定类型的对象，形成一个队列。 Java中的Stream并**不会存储元素**，而是按需计算。
- **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。
- **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。

和以前的Collection操作不同， Stream操作还有两个基础的特征：

- **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
- **内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

```sql
+--------------------+       +------+   +------+   +---+   +-------+
| stream of elements +-----> |filter+-> |sorted+-> |map+-> |collect|
+--------------------+       +------+   +------+   +---+   +-------+
```

#### 3. stream() 优点：

**无存储**。stream不是一种数据结构，它只是某种数据源的一个**视图**（可以理解为copy了一个副本，对stream的操作都是在副本上进行的），数据源可以是一个数组，Java容器或I/O channel等。

**为函数式编程而生**。对stream的任何修改都不会修改背后的数据源，比如对stream执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新stream。

**惰式执行**。stream上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。

**可消费性**。stream只能被“消费”一次，一旦遍历过就会失效，就像容器的迭代器那样，想要再次遍历必须重新生成。

#### 4. 生成流

在 Java 8 中, 集合接口有两个方法来生成流：

- **stream()** − 为集合创建串行流。
- **parallelStream()** − 为集合创建并行流。

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
```

#### 5. Stream 的常用函数

- **`forEach`**

Stream 提供了新的方法 forEach 来迭代流中的每个数据。以下代码片段使用 forEach 输出了10个随机数：

```java
Random random = new Random(); 
random.ints().limit(10).forEach(System.out::println);
```

- **`filter`**

filter 方法用于通过设置的条件过滤出元素（`return true`表示不拦截，`return false`表示拦截）。以下代码片段使用 filter 方法过滤出空字符串：

```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); // 获取空字符串的数量 
long count = strings.stream()
    .filter(string -> string.isEmpty())
    .count();
```

- **`map`**

map 方法用于映射每个元素到对应的结果，以下代码片段使用 map 输出了元素对应的平方数：

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); // 获取对应的平方数 
List<Integer> squaresList = numbers.stream()
    .map(i -> i*i)
    .distinct()
    .collect(Collectors.toList());
```

- **`limit`**

limit 方法用于获取指定数量的流。 以下代码片段使用 limit 方法打印出 10 条数据：

```java
Random random = new Random(); 
random.ints().limit(10).forEach(System.out::println);
```

- **`sorted`**

sorted 方法用于对流进行排序。以下代码片段使用 sorted 方法对输出的 10 个随机数进行排序：

```java
Random random = new Random(); 
random.ints()
    .limit(10)
    .sorted((x, y) -> y - x)
    .forEach(System.out::println);
```

- **`collect`**



