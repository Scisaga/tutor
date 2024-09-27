## Lambda表达式
把捕获外部变量的 Lambda 表达式称为闭包
* 捕获实例或静态变量是没有限制的(可认为是通过 final 类型的局部变量 this 来引用前两者)
* 捕获的局部变量必须显式的声明为 final 或实际效果的的 final 类型

如果在匿名类或 Lambda 表达式中访问的局部变量，如果不是 final 类型的话，编译器自动加上 final 修饰符。

**因为实例变量存在堆中**，**而局部变量是在栈上分配**，Lambda 表达(匿名类) 会在另一个线程中执行。如果在线程中要直接访问一个局部变量，可能线程执行时该局部变量已经被销毁了，而 final 类型的局部变量在 Lambda 表达式(匿名类) 中其实是局部变量的一个拷贝。

### 参考
* http://www.lambdafaq.org/
* [Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
* [Java 8 Stream Tutorial](http://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/)
* [详解Java 8中Stream类型的“懒”加载](http://www.cnblogs.com/heimianshusheng/p/5674372.html)

## List<V> into Map<K, V>
```java
Map<String, Choice> result = choices.stream()
        .collect(Collectors.toMap(Choice::getName, Function.identity()));
```

## 数组求和
```java
integers.values().stream().mapToInt(i -> i.intValue()).sum();
integers.values().stream().mapToInt(Integer::intValue).sum();
```

## sorted
- **sorted()**: 元素自然排序，元素类必须实现 **Comparable** 接口.
- **sorted(Comparator<? super T> comparator)**
   -  list.stream().sorted()
   - list.stream().sorted(Comparator.reverseOrder())
   - list.stream().sorted(Comparator.comparing(Student::getAge))
   - list.stream().sorted(Comparator.comparing(Student::getAge).reversed())

### sort map by value
```java
map.entrySet().stream()
    .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
    .limit(10)
    .forEach(System.out::println); // or any other terminal method
```

### 多字段排序
```java
Comparator<Employee> byFirstName = (e1, e2) ->
    e1.getEmployeeFirstName().compareTo(e2.getEmployeeFirstName());
Comparator<Employee> byLastName = (e1, e2) ->
    e1.getEmployeeLastName().compareTo(e2.getEmployeeLastName());
employees.stream()
    .sorted(byFirstName.thenComparing(byLastName))
    .forEach(e -> System.out.println(e));

Comparator<Entry<Integer, String>> byValue = (entry1, entry2) ->
    entry1.getValue().compareTo(entry2.getValue());
Optional<Entry<Integer, String>> val = CANDY_BARS.entrySet().stream()
    .sorted(byValue.reversed())
    .findFirst();
```

## map + reduce

```java
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
double bill = costBeforeTax.stream()
        .map((cost) -> cost + .12*cost)
        .reduce((sum, cost) -> sum + cost).get();
System.out.println("Total : " + bill);
```

### 并行处理
```java
final double totalPoints = tasks
    .stream()
    .parallel()
    .map( task -> task.getPoints() ) // or map( Task::getPoints ) 
    .reduce( 0, Integer::sum );
```

## filter
### 多个Predicate
```java
Predicate<String> startsWithJ = (n) -> n.startsWith("J");
Predicate<String> fourLetterLong = (n) -> n.length() == 4;
names.stream()
    .filter(startsWithJ.and(fourLetterLong))
    .forEach((n) -> System.out.print("nName, which starts with 'J' and four letter long is : " + n));
```

### 去重
```java
List<Integer> listWithDuplicates = Lists.newArrayList(1, 1, 2, 2, 3, 3);
List<Integer> listWithoutDuplicates = listWithDuplicates.stream()
    .distinct()
    .collect(Collectors.toList());
```

## collect
在 Java 8 流中，收集是一种最终操作。它接收一个给定的流，应用所有转换（主要来自 map、filter 等），并输出一个常见的 Java 集合类型的实例：List、Set、Map 等。最常用的收集器位于 java.util.stream.Collectors 工厂类中。

### Collectors.toMap()
```java
<T,K,U> Collector<T,?,MapCollector<K,U>> toMap(
        FunctionMapCollector<? super T,? extends K> keyMapper, 
        FunctionFunctionMapCollector<? super T,? extends U> valueMapper
);

<T, K, U, M extends Map<K, U>> Collector<T, ?, M> 
    java.util.stream.Collectors.toMap(
        Function<? super T, ? extends K> keyMapper,
        Function<? super T, ? extends U> valueMapper,
        BinaryOperator<U> mergeFunction,
        Supplier<M> mapSupplier
)

Map<Long, String> map = 
    list.stream()
        .sorted(Comparator.comparing(Building::getName))
        .collect(Collectors.toMap(Building::getId,
                                  Building::getName,
                                  (v1,v2)->v1, //merger: key相同时，用前面的取代后面的
                                  LinkedHashMap::new));

LinkedHashMap<KeyType,ValueType> map =
    list.stream().collect(Collectors.toMap(
        Map.Entry::getKey, 
        Map.Entry::getValue, 
        (v1,v2)->v1, 
        LinkedHashMap::new));
```
参考
* https://stackoverflow.com/questions/51186847/convert-list-of-map-entry-to-linkedhashmap
#### merger
Collectors.toMap 方法中，实现对相同key值的val进行合并操作的逻辑
```java
Map<Long, Author> totalVisitCounts = Stream.concat(visitCounts1.entrySet().stream(), visitCounts2.entrySet().stream())
    .collect(Collectors.toMap(
        entry -> entry.getKey(), // The key
        entry -> entry.getValue(), // The value
        // The "merger"
        (visitCounts1, visitCounts2) -> visitCounts1 + visitCounts2
    )
);

Map<Long, Author> totalVisitCounts = Stream.concat(visitCounts1.entrySet().stream(), visitCounts2.entrySet().stream())
    .collect(Collectors.toMap(
        entry -> entry.getKey(), // The key
        entry -> entry.getValue(), // The value
        // The "merger" as a method reference
        Integer::sum
    )
);
```

### Collectors.toList()
```java
Stream<Integer> number = Stream.of(1, 2, 3, 4, 5);
List<Integer> result2 = number.filter(x -> x!= 3).collect(Collectors.toList());
```
### Collectors.groupingBy()
```java
final Map<Status, List<Task> > map = tasks
   .stream()
   .collect(Collectors.groupingBy(Task::getStatus));
```
### List/Set of Map.Entry to LinkedHashMap
```java
LinkedHashMap<KeyType,ValueType> map = list.stream()
        .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue, (v1,v2)->v1, LinkedHashMap::new));
Map<Integer, String> mapFromSet = set.stream()
        .collect(Collectors.toMap(Entry::getKey, Entry::getValue));
Map<Object, Object> map = ArrayUtils.toMap(entrySet.toArray());
```

## SummaryStatistics
IntStream、LongStream 和 DoubleStream 等流的类中，有个非常有用的方法叫做 summaryStatistics()
```java
List<Integer> primes = Arrays.asList(2, 3, 5, 7, 11, 13, 17, 19, 23, 29);
IntSummaryStatistics stats = primes.stream().mapToInt((x) -> x).summaryStatistics();
System.out.println("Highest prime number in List : " + stats.getMax());
System.out.println("Lowest prime number in List : " + stats.getMin());
System.out.println("Sum of all prime numbers : " + stats.getSum());
System.out.println("Average of all prime numbers : " + stats.getAverage());

DoubleSummaryStatistics stats2 = Stream.of(2.4, 4.3, 3.3, 2.5)
.collect(Collectors.summarizingDouble((Double::doubleValue)));
```