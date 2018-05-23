# Java8 函数式编程

## 第2章 Lambda表达式

### 2.2 如何辨别Lambda表达式

编写Lambda表达式的不同形式

```java
Runnable noArguments = () -> System.out.print("hello world");

ActionListener oneArgument = event -> System.out.println("button clicked");

BinaryOperator<Long> add = (x, y) -> x + y;

BinaryOperator<Long> addExplicit = (Long x, Long y) -> { x + y; } 

Predicate<Integer> atLeast5 = (Integer x) -> x >= 5;		

/*
public interface Predicate<T> {
	boolean test(T t);
}
*/
```

Lambda表达式的类型依赖与上下文环境，是由编译器推断出来的。

Java8中匿名内部类使用外部局部变量不再需要显式地声明为```final```。Lambda表达式中也一样。

```java
String name = "sherry";
button.addActionListener(event -> System.out.println("hi " + name));
//引用既成事实上的final变量
```

但是下面的代码就会报错，无法通过编译

```java
String name = "sherry";
name = "vermouth";
button.addActionListener(event -> System.out.println("hi " + name));
```

### 2.4 函数接口

函数接口是只有一个抽象方法的接口，用作Lambda表达式的类型。

## 第3章 流

### 3.1 从外部迭代到内部迭代

传统上，要计算从伦敦来的艺术家的人数：

```java
int count = 0;
for (Artist artist: allArtists) {
    if (artist.isFrom("London")) {
        count++;
    }
}
```

从背后原理来看，for循环就是一个封装了迭代的语法糖。首先调用iterator方法，产生一个新的Iterator 对象，进而控制整个迭代过程，这就是外部迭代。

**问题：**从本质上来讲是一种串行化操作。

内部迭代：stream()方法的调用，返回的不是一个控制迭代的Iterator对象，而是返回内部迭代中的相应接口：Stream。

```java
long count = allArtists.stream()
    				  .filter(artist -> artist.isFrom("London"))
    				  .count();
```

内部迭代可以分解为两步更简单的操作：

1. 找出所有来自伦敦的艺术家；
2. 计算他们的人数。

***stream()是Collection接口的方法。filter()和count()都是Stream接口的方法***

每种操作都对应Stream 接口的一个方法。为了找出来自伦敦的艺术家，需要对Stream 对象进行过滤：filter。过滤在这里是指“只保留通过某项测试的对象”。测试由一个函数完成，根据艺术家是否来自伦敦，该函数返回true 或者false。由于Stream API 的函数式编程风格，我们并没有改变集合的内容，而是描述出Stream 里的内容。count() 方法计算给定Stream 里包含多少个对象。

### 3.2 实现机制

* 惰性求值方法：像filter 这样只描述Stream，最终不产生新集合的方法。
* 及早求值方法：像count 这样最终会从Stream 产生值的方法。

下面这段代码不会输出任何信息

```java
allArtists.stream()
		  .filter(artist -> {
		      System.out.println(artist.getName());
			  return artist.isFrom("London");
			});
```

如果将同样的输出语句加入一个拥有终止操作的流，就会输出

```java
allArtists.stream()
		  .filter(artist -> {
		      System.out.println(artist.getName());
			  return artist.isFrom("London");
		  })
    	  .count();
```

***如果方法的返回值是Stream那么就是惰性求值，如果返回值是另一个值或为空，那么就是及早求值。***

### 3.3 常用的流操作

#### 3.3.1 collect

由Stream里的值生成一个列表，是一个及早求值操作。

```java
List<String> collected = Stream.of("a", "b", "c")
							 .collect(Collectors.toList());
```

```of()```方法使用一组初始值生成新的Stream。

#### 3.3.2 map

将一个流中的值转换成别的值

```java
List<String> upperCase = Stream.of("a", "b", "hello")
                               .map(string -> string.toUpperCase())
                               .collect(Collectors.toList());
```

 #### 3.3.3 filter

```java
List<String> beginningWithNumbers = Stream.of("a", "1abc", "abc1")
                .filter(string -> string.charAt(0) >= '0' && string.charAt(0) <= '9')
                .collect(Collectors.toList());
```

核心死信是保留Stream中的一些元素，而过滤掉其他的。

#### 3.3.4 flatMap

flatMap 方法可用Stream 替换值， 然后将多个Stream 连接成一个Stream。

```java
List<Integer> together = Stream.of(Arrays.asList(1, 2), Arrays.asList(3, 4))
                               .flatMap(numbers -> numbers.stream())
                               .collect(toList());

Assert.assertEquals(asList(1, 2, 3, 4), together);
```

#### 3.3.5 max和min

```java
String ret1 = strList.stream()
                    .min(Comparator.comparing(s -> s.length()))
                    .get();

//或者
String ret2 = strList.stream()
                     .min((String s1, String s2) -> s1.length() - s2.length())
                     .get();
```

查找Stream 中的最大或最小元素，首先要考虑的是用什么作为排序的指标。

```max()和min()```返回的是Optional对象，调用```get()```方法可以取出Optional对象中的值。

#### 3.3.6 通用模式

```java
Object accumulator = initialValue;
for(Object element : collection) {
	accumulator = combine(accumulator, element);
}
```

首先赋给accumulator 一个初始值：initialValue，然后在循环体中，通过调用combine 函数，拿accumulator 和集合中的每一个元素做运算，再将运算结果赋给accumulator，最后accumulator 的值就是想要的结果。

#### 3.3.7 recude

reduce 操作可以实现从一组值中生成一个值。在上述例子中用到的count、min 和max 方法，因为常用而被纳入标准库中。事实上，这些方法都是reduce 操作。

```java
int count = Stream.of(1, 2, 3)
                  .reduce(0, (acc, element) -> acc + element);
```

传入Stream 中的当前元素和acc。将两个参数相加，acc 是累加器，保存着当前的累加结果。Lambda 表达式的返回值是最新的acc，是上一轮acc 的值和当前元素相加的结果。

## 第4章 类库

