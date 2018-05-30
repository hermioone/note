# SpringBoot2.0新特性

## 第2章 函数式编程和Lambda表达式

* 函数式编程概念

  编程范式，熟练使用Stream流API和lambda表达式，有流相关的思想

* 为什么要使用函数式编程

  * 关注点不同

    * 命令式编程关注怎么做

    * 函数式编程关注做什么

      ```java
      public static void main(String[] args) {
          int[] nums = {2, 1, 423, 211, 435, 732, 213, 23};
      
          int min = Integer.MAX_VALUE;
          for (int i : nums) {
              if (i < min) {
                  min = i;
              }
          }
          System.out.println(min);
      
          //jdk8
          int min2 =  IntStream.of(nums).parallel().min().getAsInt();          //.parallel()并行执行
          //想让它多线程运行，传统情况下，需要写线程，需要考虑线程安全，但是在函数式编程中，只需要.parallel()说明是多线程运行，而不需要自己去实现细节。
          System.out.println(min2);
      }
      ```

### 2-6 jdk8接口新特性

* Lambda表达式

* 默认方法

  ```java
  @FunctionalInterface        //接口中只能有一个需要实现的方法，但是可以有许多默认方法
  interface Interface1 {
      int doubleNum(int i);
      
      default int add(int x, int y) {
          return x + y;
      }
  }
  ```

### 2-9 jdk8中的函数接口

```java
class MyMoney {
    private final int money;

    public MyMoney(int money) {
        this.money = money;
    }

    public void printMoney(Function<Integer, String> moneyFormat) {     //输入是Interger类型，输出是String类型
        //这里不再需要再定义一个接口也不需要知道方法名，只需要指定接口中的唯一待实现方法输入是int，输出是String就可以
        System.out.println("我的存款是： " + moneyFormat.apply(this.money));
    }
}

public class MoneyDemo {
    public static void main(String[] args) {
        MyMoney mm = new MyMoney(32312312);
        Function<Integer, String> integerStringFunction = 
            		i -> new DecimalFormat("#,###").format(i);
        //函数接口支持链式操作
        mm.printMoney(integerStringFunction.andThen(s -> "$" + s));

    }
}
```

JDK自带的函数接口

| 接口                      | 输入参数 | 返回类型 | 说明                         |
| ------------------------- | -------- | -------- | ---------------------------- |
| ```Predicate<T>```        | T        | boolean  | 断言                         |
| ```Consumer<T>```         | T        | /        | 消费一个数据                 |
| ```Function<T, R>```      | T        | R        | 输入T输出R的函数             |
| ```Supplier<T>```         | /        | T        | 提供一个数据                 |
| ```UnaryOperator<T>```    | T        | T        | 一元函数（输出输入类型相同） |
| ```MiFunction<T, U, R>``` | (T, U)   | R        | 2个输入的函数                |
| ```BinaryOperator<T>```   | (T, T)   | T        | 二元函数（输出输入类型相同） |

### 2-11 方法引用

> 当lambda表达式函数执行体中只有一个方法调用，而且函数参数和箭头左边一样的话，可以写成函数引用的方式

```java
public class MethodReferenceDemo {
    public static void main(String[] args) {
        //Consumer<String> consumer = s -> System.out.println(s);
        //与上面等价
        Consumer<String> consumer = System.out::println;	//方法引用
        consumer.accept("leihou");
    }
}
```

```java
public class MethodReferenceDemo {
    public static void main(String[] args) {

        //静态方法的引用
        Consumer<Dog> consumer1 = Dog::bark;
        Dog dog = new Dog();
        consumer1.accept(dog);

        //非静态方法，使用对象实例引用
        //UnaryOperator<Integer> function = dog::eat;
        IntUnaryOperator function = dog::eat;	//eat()方法参数中增加this，这里都没有报错
        System.out.println("还剩 " + function.applyAsInt(2) + " 斤");
        //使用类名进行方法引用
        BiFunction<Dog, Integer, Integer> eatFunction = Dog::eat;
        System.out.println("还剩 " + function.applyAsInt(2) + " 斤");

        //构造函数的方法引用（无参）
        Supplier<Dog> supplier = Dog::new;
        System.out.println("创建了新对象：" + supplier.get());
        //带参数的构造函数
        Function<String, Dog> function1 = Dog::new;
        System.out.println("创建了新对象：" + function1.apply("雷猴"));
    }
}

class Dog {

    private String name = "哮天犬";
    /**
     * 默认10斤狗粮
     */
    private int food = 10;

    public Dog() {
    }

    public Dog(String name) {
        this.name = name;
    }

    public static void bark(Dog dog) {
        System.out.println(dog + "叫了");
    }

    /**
     * 吃狗粮
     * @param this JDK默认会把当前实例传入到非静态方法，参数名为this，位置是第一个
     * @param num
     * @return
     */
    public int eat(Dog this, int num) {
        System.out.println("吃了 " + num + " 斤");
        this.food -= num;
        return this.food;
    }

    @Override
    public String toString() {
        return this.name;
    }
}
```

### 2-13 类型推断

```java
@FunctionalInterface
interface IMath {
    int add(int x, int y);
}

@FunctionalInterface
interface IMath2 {
    int multiply(int x, int y);
}

public class TypeDemo {
    
    public static void main(String[] args) {
        //当有方法重载时，在前面加上强转就可以
        test((IMath)(x, y) -> x + y);
    }
    
    public static void test(IMath iMath) {
        iMath.add(1, 2);
    }

    public static void test(IMath2 iMath2) {
        iMath2.multiply(1, 2);
    }
}
```

### 2-15 级联表达式和柯里化

```java
/**
 * 2-15：级联表达式和柯里化
 * 柯里化：把多个参数的函数转换为只有一个参数的函数
 * 柯里化的目的：函数标准化
 * 高阶函数：返回函数的函数
 */
public class CurryDemo {
    public static void main(String[] args) {
        //实现了x + y的级联表达式
        Function<Integer, Function<Integer, Integer>> fun = x -> y -> x + y;
        System.out.println(fun.apply(2).apply(3));
        
        Function<Integer, Function<Integer, Function<Integer, Integer>>> fun2 = x -> y -> z -> x + y + z;
        System.out.println(fun2.apply(2).apply(2).apply(2));
    }
}
```

## 第3章 Stream流编程

### 3-1 Stream流编程-概念

* 概念：是一个高级的迭代器，不是一个数据结构。把数据在流水线中处理

* 外部迭代和内部迭代

  ```java
  public class StreamDemo1 {
      public static void main(String[] args) {
          int[] nums = {1, 2, 3, 4, 5};
  
          //外部迭代
          int sum = 0;
          for (int i : nums) {
              sum += i;
          }
          System.out.println("和为：" + sum);
  
          //使用Stream的内部迭代
          int sum2 = IntStream.of(nums).map(i -> i * 2).sum();
          System.out.println("和为：" + sum2);
      }
  }
  ```

* 中间操作/终止操作和惰性求值

  ```java
  public class StreamDemo1 {
      public static void main(String[] args) {
          int[] nums = {1, 2, 3, 4, 5};
  
          //map就是中间操作（返回Stream的操作）
          //sum就是终止操作
          int sum3 = IntStream.of(nums).map(StreamDemo1::doubleNum).sum();
          System.out.println("-----------------------------------");
          IntStream.of(nums).map(StreamDemo1::doubleNum);
          System.out.println("惰性求值就是终止操作没有调用的情况下，中间操作不会执行");
      }
  
      public static int doubleNum(int i) {
          System.out.println("执行了*2");
          return i * 2;
      }
  }
  ```

### 3-2 流的创建

|            | 相关方法                                                     |
| ---------- | ------------------------------------------------------------ |
| 集合       | Collection.stream/.parallelStream                            |
| 数组       | Arrays.stream                                                |
| 数字Stream | IntStream/LongStream.range/.rangeClosed     Random.ints/.long/.doubles |
| 自己创建   | Stream.generate/.iterate                                     |

### 3-4 流的终止操作

* 非短路操作
  * ```forEach```，```forEachOrdered```
  * ```collect```，```toArray```
  * ```reduce```
  * ```min```，```max```，```count```
* 短路操作
  * ```findFirst```，```findAny```
  * ```allMatch```，```anyMatch```，```noneMatch```

```java
public class StreamDemo4 {
    public static void main(String[] args) {
        String str = "hello world leihou alielie ab";

        //使用并行流，输出是乱序的
        str.chars().parallel()
            .forEach(i -> System.out.print((char)i));
        System.out.println();
        //使用forEachOrdered保证顺序
        str.chars().parallel()
            .forEachOrdered(i -> System.out.print((char)i));
        System.out.println();

        List<String> list = Stream.of(str.split(" ")).collect(Collectors.toList());
        System.out.println(list);

        //使用reduce拼接字符串
        Optional<String> letters = Stream.of(str.split(" "))
            .reduce((s1, s2) -> s1 + " | " + s2);
        System.out.println(letters.orElse(""));     //如果没有的话返回空串

        //带初始值的reduce
        String reduce = Stream.of(str.split(" "))
            .reduce("", (s1, s2) -> s1 + " | " + s2);
        System.out.println(reduce);

        //计算所有单词的总长度
        int length = Stream.of(str.split(" ")).map(s -> s.length())
            .reduce(0, (s1, s2) -> s1 + s2);
        System.out.println("单词总长度为： " + length);

        Optional<String> max = Stream.of(str.split(" "))
            .max((s1, s2) -> s1.length() - s2.length());
        System.out.println("最大值为：" + max.get());

        //短路操作
        OptionalInt optionalInt = new Random().ints().findFirst();
        System.out.println(optionalInt.getAsInt());
    }
}
```

### 3-5 并行流

```java
public class StreamDemo5 {
    public static void main(String[] args) {
        //并行流使用的线程池：ForkJoinPool.commonPool
        //默认的线程数是当前机器的cpu个数

        /*//修改默认的线程数
        System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "6");*/

        /*//使用自己的线程池，不使用默认线程池，防止任务被阻塞
        ForkJoinPool pool = new ForkJoinPool(20);
        pool.submit(() -> IntStream.range(1, 100).parallel().peek(StreamDemo5::debug).count());
        pool.shutdown();*/
		
        //jdk10无效
        IntStream.range(1, 100).parallel().peek(StreamDemo5::debug).count();

        /*//现在要实现这样的效果：先并行，再串行
        //这样的话以最后一次为准，即串行的
        IntStream.range(1, 100)
                //先并行
                .parallel().peek(StreamDemo5::debug)
                //再串行
                .sequential().peek(StreamDemo5::debug).count();*/

    }

    public static void debug(int i) {
        System.out.println(Thread.currentThread().getName() + " debug: " + i);
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### 3-6 收集器

```java
public class CollectDemo {
    public static void main(String[] args) {
        List<Student> students = Arrays.asList(
                new Student("1", 1),
                new Student("2", 2),
                new Student("3", 6),
                new Student("4", 6),
                new Student("5", 5),
                new Student("6", 6),
                new Student("7", 7),
                new Student("8", 8),
                new Student("9", 9)
        );

        /*//不能这么用，这么写是把students这个List整体当作一个Stream
        Stream.of(students).map(Student::getAge)
                .collect(Collectors.toList());*/

        Stream.of(new Student("1", 1),
                new Student("2", 2)).map(Student::getAge)
                .collect(Collectors.toList());

        //能用方法表达式就不要使用lambda表达式，因为lambda表达式会生成类似lambda$0这样的函数
        Set<Integer> ages = students.stream().map(Student::getAge)
                .collect(Collectors.toCollection(TreeSet::new));
        System.out.println(ages);

        //统计汇总信息
        IntSummaryStatistics collect = students.stream().collect(Collectors.summarizingInt(Student::getAge));
        System.out.println("年龄信息汇总：" + collect);

        //分块
        Map<Boolean, List<Student>> map = students.stream().collect(Collectors.partitioningBy(s -> s.getAge() > 6));
        System.out.println(map);

        //分组
        Map<Integer, List<Student>> collect1 = students.stream().collect(Collectors.groupingBy(Student::getAge));
        System.out.println(collect1);

        //得到每个年龄学生个数的列表
        Map<Integer, Long> collect2 = students.stream()
            .collect(Collectors.groupingBy(Student::getAge, Collectors.counting()));
        System.out.println(collect2);
}

class Student {
    private String name;
    private Integer age;

    public Student(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

### 3-7 Stream运行机制

```java
students.stream()
                .peek(System.out::print)
                .filter(s -> s.getAge() > 6)
                .count();
```

```java
 students.stream()
                //无状态操作
                .peek(System.out::println)
                //第二个无状态操作
                .filter(s -> {
                    System.out.println("filter: " + s);
                    return s.getAge() > 6;
                })
                //有状态操作
                .sorted((s1, s2) -> {
                    System.out.println("排序： + " + s1 + " , " + s2);
                    return s1.getAge() - s2.getAge();
                })
                //又一个无状态操作
                .peek(System.out::println)
                .collect(Collectors.toList());

/*
Student{name='1', age=1}
filter: Student{name='1', age=1}
Student{name='2', age=2}
filter: Student{name='2', age=2}
Student{name='3', age=6}
filter: Student{name='3', age=6}
Student{name='4', age=6}
filter: Student{name='4', age=6}
Student{name='5', age=5}
filter: Student{name='5', age=5}
Student{name='6', age=6}
filter: Student{name='6', age=6}
Student{name='7', age=7}
filter: Student{name='7', age=7}
Student{name='8', age=8}
filter: Student{name='8', age=8}
Student{name='9', age=9}
filter: Student{name='9', age=9}
排序： + Student{name='8', age=8} , Student{name='7', age=7}
排序： + Student{name='9', age=9} , Student{name='8', age=8}
Student{name='7', age=7}
Student{name='8', age=8}
Student{name='9', age=9}
/*
```

1. 所有操作是**链式调用**，一个元素只迭代一次

2. 每一个中间操作返回一个新的流，流里面有一个属性sourceStage，指向同一个地方，就是Head

3. sourceStage中有个nextStage属性，nextStage中又有nextStage

   sourceStage => nextStage（peek） => nextStage（filters）=> null

4. 有状态操作会把无状态操作截断，单独处理

5. **入参如果是两个参数，说明需要和别的进行比较，所以是有状态参数**

6. 并行环境下，有状态的中间操作不一定能并行操作

7. ```parallel()```和```sequetial()```也是中间操作，但是它们不创建流，它们只修改sourceStage中的parallel值（并行标志）。

## 第4章 reactive stream响应式流

### 4-2 Reactive Stream主要接口

* ```class Flow```
  * ```interface Publisher<T>```
    * ```public void subscribe(Subscriber<? super T> subscriber);```
  * ```interface Subscriber<T>```
    * ```public void onSubscribe(Subscription subscription);```
    * ```public void onNext(T item);```
    * ```public void onError(Throwable throwable);```
    * ```public void onComplete();```
  * ```interface Subscription```
    * ```public void request(long n);```
    * ```public void cancel();```
  * ```interface Processor<T,R> extends Subscriber<T>, Publisher<R>```

### 4-3 完整案例

```java
public class FlowDemo {
    public static void main(String[] args) throws InterruptedException {
        //1.定义发布者，发布的数据类型是Integer
        //直接使用jdk自带的SubmissionPublisher，它实现了Publisher接口
        SubmissionPublisher<Integer> publisher = new SubmissionPublisher<>();

        //2.定义订阅者
        Subscriber<Integer> subscriber = new Subscriber<Integer>() {

            private Subscription subscription;

            @Override
            public void onSubscribe(Subscription subscription) {
                //保存订阅关系，需要用它来给发布者响应
                this.subscription = subscription;

                //请求一个数据，这里请求一条数据，才能接收到一条数据
                this.subscription.request(1);
            }

            @Override
            public void onNext(Integer item) {
                //接受到一个数据，处理
                System.out.println("接收到数据： " + item);

                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                //处理完再请求一个数据
                this.subscription.request(1);

                //或者已经达到了目标，调用cancel告诉发布者不再接受数据了
                //this.subscription.cancel();
            }

            @Override
            public void onError(Throwable throwable) {
                //出现异常(例如处理数据的时候产生了异常)
                throwable.printStackTrace();

                this.subscription.cancel();
            }

            @Override
            public void onComplete() {
                System.out.println("处理完了");
            }
        };

        //3. 发布者和订阅者建立订阅关系
        publisher.subscribe(subscriber);

        //4. 生产数据，并发布
        //submit是个block方法，当发布者在发布消息的时候，如果订阅者的缓冲池满了(订阅者的subscription中有个Array)，submit就会被阻塞
        //缓冲池默认长度是256，
        //通过这种方式，可以调节生产者的速度
        for (int i = 0; i < 1000; i++) {
            publisher.submit(i);
            System.out.println("发布消息：" + i);
        }

        //5.结束后，关闭发布者
        publisher.close();

        Thread.currentThread().join(1000);
    }
}
```

## 第5章 Webflux服务端开发

### 5-1 初识Spring WebFlux

* 概念：非阻塞的新的web开发模式
* 和springmvc的关系
  * 阻塞和非阻塞
    * webflux是异步非阻塞的，一个线程中可以处理更多请求
    * springmvc是同步阻塞的，一个请求对应一个线程
  * 运行环境不同
    * springmvc基于Servlet API，必须运行在Servlet容器上
    * webflux基于Reactive Stream，可以运行在Servlet 3.1以后的容器或者运行在netty上
  * 数据库
    * webflux暂时不支持关系型数据库
* 优势
  * 可以支持很高的并发量

### 5-2 异步Servlet

传统servlet

```java
@WebServlet(name = "SyncServlet",
        urlPatterns = "/SyncServlet",
        loadOnStartup = 1)
public class SyncServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
        		throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
        		throws ServletException, IOException {
        long start = System.currentTimeMillis();
        //执行业务代码
        doSomeThing(request, response);

        long end = System.currentTimeMillis();
        System.out.println("sync use: " + (end - start));		//5001ms
    }

    private void doSomeThing(HttpServletRequest request, HttpServletResponse response) 
        		throws IOException {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        response.getWriter().append("done");
    }
}
```



* 为什么要使用异步Servlet
  * 同步servlet阻塞了tomcat容器的servlet线程，当请求到达tomcat容器后，tomcat会为每一个请求启动一个线程，线程里面会调用对应的servlet，当使用同步servlet时，业务代码花多少时间，这个servlet线程就要等多少时间。
* 异步Servlet怎么工作

```java
import javax.servlet.AsyncContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.concurrent.CompletableFuture;

@WebServlet(asyncSupported = true,
        name = "AsyncServlet",
        urlPatterns = "/AsyncServlet",
        loadOnStartup = 1)
public class AsyncServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        long start = System.currentTimeMillis();

        //1.开启异步
        AsyncContext asyncContext = request.startAsync();

        //2.开启一个线程处理业务代码，这里使用的是jdk8中的CompletableFuture线程池
        CompletableFuture.runAsync(
                () -> doSomeThing(asyncContext, asyncContext.getRequest(), asyncContext.getResponse())
        );


        long end = System.currentTimeMillis();
        System.out.println("sync use: " + (end - start));		//16ms
    }

    private void doSomeThing(AsyncContext asyncContext, ServletRequest request, ServletResponse response) {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        try {
            response.getWriter().append("done");
        } catch (IOException e) {
            e.printStackTrace();
        }

        //3.业务代码处理完毕，通知结束
        asyncContext.complete();
    }
}
```

异步是对于服务器来说的，但是对于浏览器，都是同步的，都需要花费5秒钟。

### 5-3 webflux开发

Flux和Mono

```java
import org.reactivestreams.Subscriber;
import org.reactivestreams.Subscription;
import reactor.core.publisher.Flux;

import java.util.concurrent.TimeUnit;

public class ReactorDemo {

    public static void main(String[] args) {
        //reactor = jdk8 stream + jdk9 reactive stream
        // Mono：0或者1个元素
        // Flux：0-N个元素
        String[] strs = {"1", "2", "3"};

        //2.定义订阅者
        //注意这里的Subscriber和第4章的Subscriber不是一个包下的
        Subscriber<Integer> subscriber = new Subscriber<Integer>() {

            private Subscription subscription;

            @Override
            public void onSubscribe(Subscription subscription) {
                // 保存订阅关系, 需要用它来给发布者响应
                this.subscription = subscription;

                // 请求一个数据
                this.subscription.request(1);
            }

            @Override
            public void onNext(Integer item) {
                // 接受到一个数据, 处理
                System.out.println("接受到数据: " + item);

                try {
                    TimeUnit.SECONDS.sleep(3);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                // 处理完调用request再请求一个数据
                this.subscription.request(1);

                // 或者 已经达到了目标, 调用cancel告诉发布者不再接受数据了
                // this.subscription.cancel();
            }

            @Override
            public void onError(Throwable throwable) {
                // 出现了异常(例如处理数据的时候产生了异常)
                throwable.printStackTrace();

                // 我们可以告诉发布者, 后面不接受数据了
                this.subscription.cancel();
            }

            @Override
            public void onComplete() {
                // 全部数据处理完了(发布者关闭了)
                System.out.println("处理完了!");
            }

        };

        // 这里就是jdk8的stream
        Flux.fromArray(strs).map(s -> Integer.parseInt(s))
                // 最终操作
                // 这里就是jdk9的reactive stream
                .subscribe(subscriber);
    }
}
```

webflux

```java
@RestController
public class TestController {

    private final static Logger LOGGER = LogManager.getLogger(TestController.class);

    @GetMapping("/1")
    private String get1() {
        LOGGER.info("get1 start");
        String result = createStr();
        LOGGER.info("get1 end");
        return result;
    }

    private String createStr() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "some string";
    }

    @GetMapping("/2")
    private Mono<String> get2() {
        LOGGER.info("get2 start");
        Mono<String> result = Mono.fromSupplier(() ->createStr());
        LOGGER.info("get2 end");
        return result;
    }
    
    /**
     * 返回0-n个元素
     * 像流一样一条一条地返回
     * @return
     */
    @GetMapping(value = "/3", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    private Flux<String> get3() {
        LOGGER.info("get3 start");
        Flux<String> result = Flux.fromStream(IntStream.range(1, 5).mapToObj(i -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "flux data--" + i;
        }));
        LOGGER.info("get3 end");
        return result;
    }
}

/*
18:38:58.533  INFO 19452 --- [ctor-http-nio-2] c.i.c.TestController                 : get2 start
18:38:58.534  INFO 19452 --- [ctor-http-nio-2] c.i.c.TestController                 : get2 end
18:39:20.506  INFO 19452 --- [ctor-http-nio-2] c.i.c.TestController                 : get1 start
18:39:25.507  INFO 19452 --- [ctor-http-nio-2] c.i.c.TestController                 : get1 end
*/
```

## 5-5 server-sent events

Flux返回多次数据是怎么实现的？Http协议是基于一问一答的形式。

**答**：用到的是H5的server-sent events

```java
@WebServlet(name = "SSE", urlPatterns = "/sse", loadOnStartup = 1)
public class SSE extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //这两个字段必须设置
        response.setContentType("text/event-stream");
        response.setCharacterEncoding("utf-8");

        for (int i = 0; i < 5; i++) {
            response.getWriter().write("data: " + i + "\n\n");      //固定格式data:xxx\n\n
            response.getWriter().flush();

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

可以用于服务器向前端推送数据

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title></title>
  </head>
  <body>


  <script type="text/javascript">
      //初始化，参数为url
      var sse = new EventSource("sse");		//和服务器端的 urlPatterns 相等
      sse.onmessage = function (evt) {
          console.log("message: ", evt.data, evt);
          if (e.data == 3) {
              sse.close();
          }
      }
  </script>
  </body>
</html>
```

前端浏览器访问这个页面的时候，会自动监听```/sse```这个url，如果有数据到达，就在控制台输出。

## 5-6 完整例子

整合mongodb

### 1. 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
</dependency>
```

### 2. 添加注解

```java
@SpringBootApplication
@EnableMongoRepositories
public class WebfluxApplication {

	public static void main(String[] args) {
		SpringApplication.run(WebfluxApplication.class, args);
	}
}
```

### 3. 定义对象

```java
@Document(collection = "user")
public class User {

    @Id
    private String id;

    private String name;

    private int age;

    public String getId() { return id; }

    public void setId(String id) { this.id = id; }

    public String getName() { return name; }

    public void setName(String name) { this.name = name; }

    public int getAge() { return age; }

    public void setAge(int age) { this.age = age; }

    @Override
    public String toString() {
        return "User{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

### 4. 定义仓库

```java
@Repository
public interface UserRepository extends ReactiveMongoRepository<User, String> {
}
```

### 5. Controller

```java
@RestController
@RequestMapping(value = "/user")
public class UserController {

    private final UserRepository repository;

    /**
     * 推荐这种写法，和spirng的耦合度更低
     * @param userRepository
     */
    public UserController(UserRepository userRepository) {
        this.repository = userRepository;
    }

    /**
     * 以数组形式一次返回数据
     * @return
     */
    @GetMapping(value = "/")
    public Flux<User> getAll() {
        return repository.findAll();
    }

    /**
     * 以SSE形式多次返回数据
     * @return
     */
    @GetMapping(value = "/stream/all", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<User> streamGetAll() {
        return repository.findAll();
    }

    @PostMapping(value = "/")
    public Mono<User> createUser(User user) {
        //spring data jpa中，新增和修改都是save，有id是修改，id为空是新增
        return this.repository.save(user);
    }

    /**
     * 根据id删除用户
     * 存在的时候返回200，不存在返回404
     * @param id
     * @return
     */
    @DeleteMapping("/{id}")
    public Mono<ResponseEntity<Void>> deleteUser(@PathVariable("id") String id) {
        //deleteById返回Mono<Void>，不能用它
        return this.repository.findById(id)
                //当要操作数据，并返回一个Mono，这个时候使用flatMap
                //如果不操作数据，只是转换数据，使用map
                .flatMap(user -> this.repository.delete(user).then(Mono.just(new ResponseEntity<Void>(HttpStatus.OK))))
                .defaultIfEmpty(new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }

    @PutMapping("/{id}")
    public Mono<ResponseEntity<User>> updateUser(@PathVariable("id") String id, User user) {
        Date start = new Date();
        LOGGER.info("start time: {}", start);
        Mono<ResponseEntity<User>> responseEntityMono = this.repository.findById(id)
                //操作数据
                .flatMap(u -> {
                    u.setAge(user.getAge());
                    u.setName(user.getName());
                    try {
                        Thread.sleep(3000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    return this.repository.save(u);
                })
                //转换数据
                .map(u -> new ResponseEntity<>(u, HttpStatus.OK))
                .defaultIfEmpty(new ResponseEntity<>(HttpStatus.NOT_FOUND));
        Date end = new Date();
        LOGGER.info("end time: {}", end);
        LOGGER.info("花费时间: {} ms", (end.getTime() - start.getTime()));  //花费时间: 10 ms
        return responseEntityMono;
    }
    
    @GetMapping("/{id}")
    public Mono<ResponseEntity<User>> findUserById(@PathVariable("id") String id) {
        return this.repository.findById(id)
                .map(user -> new ResponseEntity<>(user, HttpStatus.OK))
                .defaultIfEmpty(new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }
}
```

### 6. 配置

```properties
spring.data.mongodb.uri=mongodb://localhost:27017/webflux
```

## 5-10 RouterFunction模式

handler:

```java
package com.imooc.handler;

import com.imooc.domain.User;
import com.imooc.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.core.publisher.Mono;

@Component
public class UserHandler {

    @Autowired
    private UserRepository userRepository;

    public Mono<ServerResponse> getAllUser(ServerRequest request) {
        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON_UTF8)
                .body(this.userRepository.findAll(), User.class);
    }

    /**
     * 创建用户
     * @param request
     * @return
     */
    public Mono<ServerResponse> createUser(ServerRequest request) {
        Mono<User> user = request.bodyToMono(User.class);
        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON_UTF8)
                .body(this.userRepository.saveAll(user), User.class);
    }

    /**
     * 根据Id删除用户
     * @param request
     * @return
     */
    public Mono<ServerResponse> deleteUserById(ServerRequest request) {
        String id = request.pathVariable("id");
        return this.userRepository.findById(id)
                .flatMap(user ->
                    this.userRepository.delete(user).then(ServerResponse.ok().build()))
                .switchIfEmpty(ServerResponse.notFound().build());
    }
}
```

router

```java
package com.imooc.router;

import com.imooc.handler.UserHandler;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.server.RequestPredicates;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.RouterFunctions;
import org.springframework.web.reactive.function.server.ServerResponse;

@Configuration
public class AllRouters {

    /**
     * 把/router/user/和userHandler的getAllUser()绑定
     * @param userHandler
     * @return
     */
    @Bean
    RouterFunction<ServerResponse> userRouter(UserHandler userHandler) {
        return RouterFunctions.nest(
                //相当于类上面的@RequestMapping("/router/user")
                RequestPredicates.path("/router/user"),
                //相当于类里面的@RequestMapping
                RouterFunctions.route(RequestPredicates.GET("/"), userHandler::getAllUser));
    }
}
```

## 5-11 RouterFunction模式的参数校验

```java
public Mono<ServerResponse> createUser(ServerRequest request) {
    Mono<User> user = request.bodyToMono(User.class);
    /*return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON_UTF8)
                .body(this.userRepository.saveAll(user), User.class);*/
    return user.flatMap(u -> {
        //这里放校验代码 
        //CheckUtil.checkName(u.getName());
        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON_UTF8)
            .body(this.userRepository.save(u), User.class);
    });
}
```

