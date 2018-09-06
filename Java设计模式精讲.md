# Java设计模式精讲

## 第2章 UML极速入门

### 2-1 本章导航

**UML特点**：

* UML是一种开放的方法
* 用于说明、可视化、构建和编写一个正在开发的面向对象的、软件密集系统的制品和开发方法
* UML展现了一系列最佳工程实践

**UML2.2分类**

* 结构式图形：强调的是系统式的建模
  * 静态图（类图、对象图、包图）
  * 实现图（组件图、部署图）
  * 剖面图
  * 复合结构图
* 行为式图形：强调系统模型中触发的事件
  * 活动图
  * 状态图
  * 用例图
* 交互式图形：属于行为式图形子集合，强调系统模型中资料流程
  * 通信图
  * 交互概述图
  * 时序图
  * 时间图

**记忆技巧**：

1. UML箭头方向：从子类指向父类

   * 定义子类时需要通过extends关键字指定父类
   * 子类一定知道父类定义，但是父类并不知道子类的定义
   * 只有知道对方信息时才能指向对方，所以 子类 ==> 父类
2. 实线：继承；虚线：实现

   * 空心三角箭头：继承或实现
   * 实线：继承，不虚，很结实
   * 虚线：代表虚，无实体
3. 实线：关联；虚线：依赖

   * 虚线：临时用一下，表示一种使用关系，一般是一个类使用另一个类作为参数使用或作为返回值
   * 实线：关系稳定，通常是一个类中有另一个类作为属性
4. 空心菱形：聚合；实心菱形：组合

   * 菱形就是一个装东西的器皿
   * 聚合：
     * 代表空器皿中可以放很多相同东西，聚在一起（箭头指向的类），比如大雁聚合在一起形成大雁群
     * 整体和局部的关系，两者有独立的生命周期，是has-a的关系
   * 组合：
     * 代表器皿里已经有实体结构的存在，比如鸟由翅膀组成
     * 整体和局部的关系，两者有相同的生命周期，是contains-a的关系
5. 常见数字表达式及含义，假设有A类和B类，数字在A类侧

   * 0..1：某一时刻，B的实例可以和0个或1个A的实例相关
   * 0..*：0或多个
   * 1..1：有且仅有一个实例
   * 1：同上
   * 1..*：B的实例可以和1个或多个A的实例相关

**UML时序图**：

![](http://static.zybuluo.com/vermouth9/bgugtl0rzswvm4l7faj8tzo1/image.png)

### 2-2 UML类图讲解

![](http://static.zybuluo.com/vermouth9/0ixvn5akzaffnfj3movu9gov/image.png)

### 2-2 UML类图讲解-自上向下

![](http://static.zybuluo.com/vermouth9/jo66338ehggl29dbt59gxom9/image.png)

* 动物 **依赖** 氧气和水（新陈代谢()依赖氧气和水来实现功能）
* 鸟 **is a** 动物（继承）
* 每个鸟 **contains** 2个翅膀（组合）
* 大雁、鸭、企鹅 **继承* *鸟
* 企鹅类中 **有** 气候这个成员 **变量** （关联）
* 大雁群 **has a** 大雁（聚合）
* 大雁 **实现** 了飞翔接口
* 唐老鸭 **继承** 了鸭，**实现** 了讲人话接口

## 第3章 软件设计七大原则

设计原则讲究一个度，讲究平衡，不是一定要遵守。

### 3-2 开闭原则

 #### 讲解

定义：一个软件实体（类、模块、函数）应该对扩展开放，对修改关闭

用抽象构建框架，用实现扩展细节

实现开闭原则的核心思想：**面向抽象编程**

### 3-4 依赖倒置原则

#### 讲解

定义：高层模块不应该依赖底层模块，二者都应该依赖其抽象。

抽象不应该依赖细节；细节应该依赖抽象。

针对接口编程，不用针对实现编程。

优点：减少类间的耦合性，提高系统稳定性。

#### coding

```java
public class Geely {
    public void studyImoocCourse(ICourse iCourse) {
        iCourse.studyCourse();
    }
}

interface ICourse {
    void studyCourse();
}

class JavaCourse implements ICourse {
    @Override
    public void studyCourse() {
        System.out.println("Geely在学习Java课程");
    }
}

class FECourse implements ICourse {
    @Override
    public void studyCourse() {
        System.out.println("Geely在学习前端课程");
    }
}

public class Test {
    private static void test2() {
        Geely geely = new Geely();
        geely.studyImoocCourse(new JavaCourse());
        geely.studyImoocCourse(new FECourse());
    }

    public static void main(String[] args) {
		test2();
    }
}
```

### 3-5 单一职责原则

#### 讲解

定义：不要存在多于一个导致类变更的原因

一个类、接口、方法只负责一项职责

优点：降低类的复杂度，提供类的可读性，提供系统的可维护性，降低变更引起的风险。

在实际开发中，**接口和方法一定要遵守单一职责，类看情况考虑**。

### 3-7 接口隔离原则

#### 讲解

定义：用多个专门的接口，而不使用单一的总接口，客户端不应该依赖它不需要的接口。

一个类对一个类的依赖应该建立在最小的接口上。

建立单一接口，不要建立庞大臃肿的接口。

优点：符合高内聚低耦合，从而使得类具有很好的可读性、可扩展性和可维护性。

### 3-8 迪米特法则

#### 讲解

定义：一个对象应该对其他对象保持最少的了解，又叫最少知道原则。

尽量降低类与类之间的耦合。

优点：降低类之间的耦合。

### 3-9 里氏替换原则

#### 讲解

定义：如果对每个类型为T1的对象o1，都有类型为T2的对象o2，使得以T1定义的所有程序P在所有的对象o1都替换成o2时，程序P的行为没有发生变化，那么类型T2是类型T1的子类型。

定义扩展：一个软件实体如果适用一个父类的话，那一定适用于其子类，所有引用父类的地方必须能透明地使用其子类的对象，子类对象能够替换父类对象，而程序逻辑不变。

引申意义：子类可以扩展父类的功能，但不能改变父类原有的功能。

* 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。
* 子类中可以增加自己特有的方法。
* 当子类的方法重载父类的方法时，方法的前置条件（即方法的输入、入参）要比父类方法的输入参数更宽松。
* 当子类的方法实现父类的方法时，方法的后置条件（即方法的输出、返回值）要比父类更严格或相等。

优点：

1. 约束继承泛滥，开闭原则的一种体现。
2. 加强程序的健壮性，提供程序的维护性、扩展性。

### 3-11 合成复用原则

#### 讲解

定义：尽量**使用对象组合、聚合，而不是继承关系**达到软件复用的目的。

聚合has-a和组合contains-a

优点：可以使系统更加灵活，降低类与类之间的耦合度。

* 聚合：黑箱复用
* 继承：白箱复用

## 第4章 简单工厂模式

### 4-1 简单工厂讲解

定义：由一个工厂对象决定创建出哪一种产品类的实例

类型：创建型，但不属于GOF23种设计模式

#### 适用场景

1. 工厂类负责创建的对象比较少
2. 客户端只知道传入工厂类的参数，对于如何创建对象不关心

#### 优点

1. 实现了客户端和具体实现类的解耦

#### 缺点

1. 工厂类的职责权责相对过重，增加新的产品需要修改工厂类的判断逻辑，违背开闭原则。

#### 典型疑问

> 简单工厂不就是把客户端里面的```new JavaVideo()```移动到了简单工厂里面吗？

理解这个问题的重点就在于***理解简单工厂所处的位置***。

接口是用来封装隔离具体的实现的，目标是为了不让客户端知道封装体内部的具体实现（比如不让Test.java知道JavaVideo）。简单工厂的位置是位于封装体内的，也就是简单工厂是跟接口和具体的实现在一起，算是封装体内部的一个类，所以简单工厂知道具体的实现类是没有关系的。

#### 思考简单工厂

简单工厂的本质是：**选择实现**

### 4-2 简单工厂coding

```java
public class VideoFactory {
    public static Video getVideo(String type) {
        if ("java".equalsIgnoreCase(type)) {
            return new JavaVideo();
        }else if ("python".equalsIgnoreCase(type)) {
            return new PythonVideo();
        }
        return null;
    }

    /**
     * 通过反射增强简单工厂
     * @param cl Video的具体类型
     * @return 生成的Video对象
     */
    public static Video getVideo(Class cl) {
        Video video = null;
        try {
            video = (Video)cl.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return video;
    }
}
```

### 4-3 简单工厂源码解析

Calendar.java中的

```java
private static Calendar createCalendar(TimeZone zone, Locale aLocale) {
    CalendarProvider provider =
            LocaleProviderAdapter.getAdapter(CalendarProvider.class, aLocale)
                                 .getCalendarProvider();
        // ...

        Calendar cal = null;

        if (aLocale.hasExtensions()) {
            String caltype = aLocale.getUnicodeLocaleType("ca");
            if (caltype != null) {
                switch (caltype) {
                case "buddhist":
                cal = new BuddhistCalendar(zone, aLocale);
                    break;
                case "japanese":
                    cal = new JapaneseImperialCalendar(zone, aLocale);
                    break;
                case "gregory":
                    cal = new GregorianCalendar(zone, aLocale);
                    break;
                }
            }
        }
        if (cal == null) {
            if (aLocale.getLanguage() == "th" && aLocale.getCountry() == "TH") {
                cal = new BuddhistCalendar(zone, aLocale);
            } else if (aLocale.getVariant() == "JP" && aLocale.getLanguage() == "ja"
                       && aLocale.getCountry() == "JP") {
                cal = new JapaneseImperialCalendar(zone, aLocale);
            } else {
                cal = new GregorianCalendar(zone, aLocale);
            }
        }
        return cal;
}
```

## 第5章 工厂方法模式

### 5-1 工厂方法讲解

定义：定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类。工厂方法让类的实例化推迟到子类中进行。

类型：创建型

#### 使用场景

1. 创建对象需要大量重复的代码
2. 客户端不依赖与产品类实例如何被创建、实现等细节
3. 一个类通过其子类来指定创建哪个对象

#### 优点

* 用户只需要关心所需产品对应的工厂，无须关心创建细节
* 加入新产品符合开闭原则，提供可扩展性

#### 缺点

* 类的个数容易过多，增加复杂度
* 增加了系统的抽象性和理解难度

#### 产品族和产品等级

* Java的视频，Python的视频，JavaScript的视频属于同一产品等级；
* Python的视频、Python的手记属于同一产品族；
* 美的的冰箱，海尔的冰箱属于同一产品等级；
* 美的的冰箱，美的的洗衣机都是美的属于同一产品族；

> 抽象工厂解决同一产品族的问题，工厂方法解决同一产品等级的问题。

### 5-2 工厂方法coding

```java
public abstract class Video {
    public abstract void produce();
}

public class JavaVideo extends Video {
    @Override
    public void produce() {
        System.out.println("Java课程");
    }
}

public class PythonVideo extends Video {
    @Override
    public void produce() {
        System.out.println("Python视频");
    }
}

/**
 * 为什么这里使用抽象类？可能这里面有一些已知的实现
 */
public abstract class VideoFactory {
    public abstract Video getVideo();
}

public class JavaVideoFactory extends VideoFactory {
    @Override
    public Video getVideo() {
        return new JavaVideo();
    }
}

/**
 * 具体产生什么视频，是由VideoFactory的子类来决定，而不是VideoFactory决定
 */
public class PythonVideoFactory extends VideoFactory {
    @Override
    public Video getVideo() {
        return new PythonVideo();
    }
}
```

![](http://static.zybuluo.com/vermouth9/0p70vs7yajwqfx4khp0sceiu/image.png)

### 5-3 工厂方法源码解析

```java
public interface Collection<E> extends Iterable<E> {
	Iterator<E> iterator();
}

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
	public ListIterator<E> listIterator() {
        return new ListItr(0);
    }
    
    private class Itr implements Iterator<E> {
        // ...
    }
}
```

所有Collection接口的子类都实现了自己的```iterator()```方法，返回了自己定义的```Iterator```接口的子类。

```java
public interface URLStreamHandlerFactory {
    URLStreamHandler createURLStreamHandler(String protocol);
}

public class Launcher {
    private static class Factory implements URLStreamHandlerFactory {
        private static String PREFIX = "sun.net.www.protocol";

        public URLStreamHandler createURLStreamHandler(String var1) {
            String var2 = PREFIX + "." + var1 + ".Handler";
            Class var3 = Class.forName(var2);
            return (URLStreamHandler)var3.newInstance();
            }
        }
    }
}
```

这个是简单工厂和工厂方法的结合

## 第6章 抽象工厂模式

### 6-1 抽象工厂讲解

定义：提供一个创建一系列相关或相互依赖对象的接口

扩展：无须指定它们具体的类

类型：创建型

#### 适用场景

1. 客户端不依赖于产品类实例如何被创建、实现等细节
2. 强调一系列相关的产品对象（属于同一产品族）一起使用创建，创建对象需要大量重复的代码
3. 提供一个产品类的库，所有的产品以同样的接口出现，从而使客户端不依赖于具体实现

#### 优点

* 具体产品在应用层代码隔离，无须关心创建细节
* 将一个系列的产品族统一到一起创建

#### 缺点

* 规定了所有可能被创建的产品集合，产品族中扩展新的产品困难，需要修改抽象工厂的接口
* 增加了系统的抽象性和理解难度

#### 产品等级结构和产品族

#### 几种工厂模式的对比

* 简单工厂和抽象工厂
  * 简单工厂是用来选择实现的，一个简单工厂可以有多个用于选择并创建对象的方法，**多个方法创建的对象可以有关系也可以没有关系**
  * 抽象工厂是用来选择产品簇的实现的，抽象工厂里面有多个用于选择并创建对象的方法，但**这些方法创建的对象之间是有关系的**
  * 如果抽象工厂只有一个实现，不分层次，那么就相当于简单工厂了
* 简单工厂和工厂方法
  * 工厂方法的本质也是选择实现的，但是把选择具体实现的功能延迟到子类去实现
  * 如果把工厂方法中选择的实现放到父类直接实现，那就等同于简单工厂

![](http://static.zybuluo.com/vermouth9/5l7gtsl0xka91bo1dr69pn11/image.png)

### 6-2 抽象工厂coding

```java
public interface CourseFactory {
    /**
     * 视频
     */
    Video getVideo();

    /**
     * 手记
     */
    Article getArticle();
}

public class JavaCourseFactory implements CourseFactory {

    @Override
    public Video getVideo() {
        return new JavaVideo();
    }

    @Override
    public Article getArticle() {
        return new JavaArticle();
    }
}

public abstract class Article {
    public abstract void produce();
}

public abstract class Video {
    public abstract void produce();
}

public class JavaVideo extends Video {
    @Override
    public void produce() {
        System.out.println("录制java视频");
    }
}

public class JavaArticle extends Article {
    @Override
    public void produce() {
        System.out.println("编写java手记");
    }
}

public class Test {
    public static void main(String[] args) {
        CourseFactory courseFactory = new JavaCourseFactory();
        
        // Video和Article都是Java产品族的
        Video video = courseFactory.getVideo();
        Article article = courseFactory.getArticle();
        video.produce();
        article.produce();
    }
}
```

![](http://static.zybuluo.com/vermouth9/jglf9h90zvmar2ml1ncetpac/image.png)

这个UML图是**抽象工厂和工厂方法的结合**

### 6-3 抽象工厂源码解析

```java
package java.sql

public interface Connection  extends Wrapper, AutoCloseable {
	Statement createStatement() throws SQLException;
	
	PreparedStatement prepareStatement(String sql)
        throws SQLException;
}
```

如果是MySQL，那么```Statement```和```PreparedStatement```都是MySQL的；如果是Oracle，那么```Statement```和```PreparedStatement```都是Oracle。

## 第7章 建造者模式

### 7-1 建造者模式讲解

定义：讲一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示

用户只需指定需要建造的类型就可以得到它们，建造过程及细节不需要知道

类型：创建型

#### 适用场景

* 如果一个对象有非常复杂的内部结构（很多属性）
* 想把复杂对象的创建和使用分离

比如做一盘番茄炒蛋，必需的步骤有炒鸡蛋，炒西红柿，加盐，但是每个人做这些步骤的顺序可能不同。

#### 优点

* 封装性好，创建和使用分离
* 扩展性好、建造类之间独立、一定程度上解耦

#### 缺点

* 产生多余的Builder对象
* 产品内部发生变化，建造者都要修改，成本较大

#### 建造者模式和工厂模式的区别

* 建造者模式注重方法的调用顺序，工厂模式注重创建产品
* 建造者模式可以创建一些复杂的产品，由各种复杂的构件组成；而工厂模式创建出来的都是一个样子
* 工厂模式只要把对象创建出来就okay了，而建造者模式不止要创建出产品，还要知道产品是由哪些部件组成的。

> 建造者模式适用于产生差异化的产品，产品由

### 7-2 建造者coding

```java
public abstract class Builder {
    public abstract void buildCPU(String cpu);
    public abstract void buildMaindBoard(String mainBoard);
    public abstract void buildhardDist(String hardDisk);
    public abstract void buildDisplayCard(String displayCard);
    public abstract void buildPower(String power);
    public abstract void buildMemory(String memory);

    public abstract Computer createComputer();
}

public class ActualBuilder extends Builder {
    private Computer computer = new Computer();
    @Override
    public void buildCPU(String cpu) { computer.setCpu(cpu); }
    @Override
    public void buildMaindBoard(String mainBoard) { computer.setMainBoard(mainBoard); }
    @Override
    public void buildhardDist(String hardDisk) { computer.setHardDisk(hardDisk); }
    @Override
    public void buildDisplayCard(String displayCard) { computer.setDisplayCard(displayCard); }

    @Override
    public void buildPower(String power) { computer.setPower(power); }

    @Override
    public void buildMemory(String memory) { computer.setMemory(memory); }

    @Override
    public Computer createComputer() { return computer; }
}

@Data
public class Computer {
    private String cpu;
    private String mainBoard;
    private String hardDisk;
    private String displayCard;
    private String power;
    private String memory;
}

@Data
public class Boss {
    private Builder builder;

    public Computer createComputer(String cpu,
            String mainBoard,
            String hardDisk,
            String displayCard,
            String power,
            String memory) {

        this.builder.buildCPU(cpu);
        this.builder.buildDisplayCard(displayCard);
        this.builder.buildhardDist(hardDisk);
        this.builder.buildMaindBoard(mainBoard);
        this.builder.buildMemory(memory);
        this.builder.buildPower(power);

        return this.builder.createComputer();
    }
}
```

建造者模式中```Boss```这个类不是必须的，它的作用不是建造产品，而是指挥建造产品的顺序或者产品包含哪些组件。但是这种方式

```java
@Data
public class Computer {
    private String cpu;
    private String mainBoard;
    private String hardDisk;
    private String displayCard;
    private String power;
    private String memory;

    public Computer(ComputerBuilder computerBuilder) {
        this.cpu = computerBuilder.cpu;
        this.mainBoard = computerBuilder.mainBoard;
        this.hardDisk = computerBuilder.hardDisk;
        this.displayCard = computerBuilder.displayCard;
        this.power = computerBuilder.power;
        this.memory = computerBuilder.memory;
    }

    public static class ComputerBuilder {
        private String cpu;
        private String mainBoard;
        private String hardDisk;
        private String displayCard;
        private String power;
        private String memory;

        public ComputerBuilder buildCPU(String cpu) {
            this.cpu = cpu;
            return this;
        }

        public ComputerBuilder buildMaindBoard(String mainBoard) {
            this.mainBoard = mainBoard;
            return this;
        }

        public ComputerBuilder buildhardDist(String hardDisk) {
            this.hardDisk = hardDisk;
            return this;
        }

        public ComputerBuilder buildDisplayCard(String displayCard) {
            this.displayCard = displayCard;
            return this;
        }

        public ComputerBuilder buildPower(String power) {
            this.power = power;
            return this;
        }

        public ComputerBuilder buildMemory(String memory) {
            this.memory = memory;
            return this;
        }

        public Computer build() {
            return new Computer(this);
        }

    }
}

public class Boss {
    public static Computer createComputer() {
        return (new Computer.ComputerBuilder())
                .buildCPU("i9")
                .buildDisplayCard("1080Ti")
                .build();
    }
}
```

### 7-3 建造者模式源码解析

guava中的Immutable系列

```java
Set<String> set = ImmutableSet.<String>builder()
                .add("a")
                .add("b")
                .build();
```

## 第8章 单例模式

### 8-1 单例模式讲解

定义：保证一个类仅有一个实例，并提供一个全局访问点

类型：创建型

#### 适用场景

想确保任何情况下都绝对只有一个实例

#### 优点

* 在内存中只有一个实例，减少了内存开销
* 避免对资源的多重占用
* 设置全局访问点，严格控制访问

#### 缺点

* 没有接口，扩展困难

#### 单例重点

* 私有构造器
* 线程安全
* 延迟加载
* 序列化和反序列化安全
* 反射

### 8-2 单例设计模式-懒汉式及多线程Debug实战

```java
public class LazySingleton {
    private static LazySingleton lazySingleton = null;

    private LazySingleton() {}

    @UnRecommend("线程不安全")
    public static LazySingleton getInstanceUnSafe() {
        if (lazySingleton == null) {
            lazySingleton = new LazySingleton();
        }
        return lazySingleton;
    }

    // 锁了LazySingleton这个类
    @UnRecommend("比较消耗资源，性能较差")
    public synchronized static LazySingleton getInstance() {
        if (lazySingleton == null) {
            lazySingleton = new LazySingleton();
        }
        return lazySingleton;
    }
}
```

### 8-3 单例设计模式-DoubleCheck双重检查实战及原理解析

```java
package com.vermnouth.pattern.creational.singleton.lazy;

import com.vermnouth.annotations.UnRecommend;

public class LazyDoubleCheckSingleton {
    private static LazyDoubleCheckSingleton instance = null;
    private LazyDoubleCheckSingleton() {}
    public static LazyDoubleCheckSingleton getInstanceUnSafe() {
        if (instance == null) {
            synchronized (LazyDoubleCheckSingleton.class) {
                if (instance == null) {
                    instance = new LazyDoubleCheckSingleton();
                }
            }
        }
        return instance;
    }
}

```

这种方法有一个潜在的危害：**指令重排序**

```instance = new LazyDoubleCheckSingleton()```实际运行时分成3步：

1. 分配内存给这个对象
2. 初始化对象
3. 设置instance指向刚分配的内存

这里的步骤2和步骤3很可能会发生指令重排序，使得实际的执行顺序是1 => 3 => 2。

Java语言规范中规定所有线程在执行Java程序时，必须遵守 intra-thread semantics，保证重排序不会改变单线程内的程序执行结果（对于单线程来说，1->3->2的顺序不会改变结果）。

但是对多线程就会有问题

![](http://static.zybuluo.com/vermouth9/2wuyl3sy19nyf3prr12nf3dm/image.png)

在这里，线程0进行到步骤3的时候被挂起，线程1开始执行，此时因为```instance```已经指向了分配的内存空间，所有此时```instance != null```，线程1直接就把```instance```返回了，但此时```instance```指向的内存空间还没有初始化，所以就出错了。

***解决办法***：

1. 不允许2和3重排序：使用```volatile```关键字

   ```volatile```不仅可以保证内存可见性，还可以阻止JVM对相关代码进行指令重排。

   ```java
   public class LazyDoubleCheckSingleton {
       private static volatile LazyDoubleCheckSingleton instance = null;
       private LazyDoubleCheckSingleton() {}
       public static LazyDoubleCheckSingleton getInstance() {
           if (instance == null) {
               synchronized (LazyDoubleCheckSingleton.class) {
                   if (instance == null) {
                       instance = new LazyDoubleCheckSingleton();
                   }
               }
           }
           return instance;
       }
   }
   ```

2. 步骤2和3的重排序对其它线程不可见：下一节

### 8-4 单例设计模式-静态内部类-基于类初始化的延迟加载解决方案

```java
public class StaticInnerClassSingleton {
    private static class InnerClass {
        private static StaticInnerClassSingleton instance = new StaticInnerClassSingleton();
    }

    public static StaticInnerClassSingleton getInstance() {
        return InnerClass.instance;
    }
    
    private StaticInnerClassSingleton() {}
}
```

由于```instance```是静态的域，因此只会在虚拟机装载类```InnerClass```的时候初始化依次，并由虚拟机保证线程它的安全。

> 这种方式在加载StaticInnerClassSingleton类的时候并不会加载InnerClass类，也就不会创建instance实例，只有在调用InnerClass.instance时才执行了InnerClass类的初始化，创建了instance对象，这就是和饿汉式单例模式的区别

假设一个类为A，以下的这几种情况都会导致A被立刻初始化：

* 有一个A类的对象被创建
* A类中的静态方法被调用
* A类中的静态成员被赋值
* A类中的静态成员被使用，并且不是常量成员

![](http://static.zybuluo.com/vermouth9/pjt7du1h5ac09h4hruqgyi9o/image.png)

### 8-5 单例设计模式-饿汉式

```java
public class HungrySingleton {
    private final static HungrySingleton instance = new HungrySingleton();
    private HungrySingleton() {}
    public static HungrySingleton getInstance() {
        return instance;
    }
}
```

### 8-6 单例设计模式-序列化破坏单例模式原理及解决方案

```java
public class Test {
    public static void main(String[] args) throws Exception {
        test1();            // false
    }
    private static void test1() throws Exception {
        HungrySingleton instance = HungrySingleton.getInstance();

        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("singleton.txt"));
        oos.writeObject(instance);

        File file = new File("singleton.txt");
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));

        HungrySingleton newInstance = (HungrySingleton) ois.readObject();

        System.out.println(instance == newInstance);
    }
}
```

因为在执行反序列化时，ObjectInputStream中

```java
obj = desc.isInstantiable() ? desc.newInstance() : null;
```

因为实现了```Serializable```接口，因此```obj = desc.newInstance()```，调用了反射创建新的对象。

解决方法：

```java
@Recommend("可以防止序列化破坏单例模式")
public class HungrySingletonSeria implements Serializable {
    private final static HungrySingletonSeria instance = new HungrySingletonSeria();
    private HungrySingletonSeria() {}
    public static HungrySingletonSeria getInstance() {
        return instance;
    }

    private Object readResolve() {
        return instance;
    }
}
```

在反序列化的过程中，通过反射创建了新的对象后，会通过反射检查有没有```readResolve()```方法，如果有的话会调用这个方法并把返回结果返回回去，所以可以保证序列化不会破坏单例。

**但是其实在序列化的过程中还是创建了新的对象。**

### 8-7 单例设计模式-反射攻击解决方案及原理分析

对于饿汉式和静态内部类懒汉式来说，可以通过在构造器中判断instance是否为空 ，如果不为空则抛出异常来

```java
public class HungrySingletonSeriaRefl implements Serializable {
    private final static HungrySingletonSeriaRefl instance = new HungrySingletonSeriaRefl();

    private HungrySingletonSeriaRefl() {
        if (instance != null) {
            throw new RuntimeException("单例模式不允许反射调用");
        }
    }

    public static HungrySingletonSeriaRefl getInstance() {
        return instance;
    }

    private Object readResolve() {
        return instance;
    }
}
```

**对于其他的懒汉式，无法防止反射攻击。**因为不管设置再复杂的逻辑，添加什么成员变量，都可以通过反射修改。

### 8-8 单例设计模式-Enum枚举单例

枚举类单例可以不受序列化和反射攻击的影响。

```java
public enum EnumInstance {
    INSTANCE{
        @Override
        protected void printTest() {
            System.out.println("printTest");
        }
    };

    private Object data;

    public Object getData() {
        return data;
    }
    public void setData(Object data) {
        this.data = data;
    }
    public static EnumInstance getInstance() {
        return INSTANCE;
    }

    protected abstract void printTest();		// 别的类可以用printTest()
}

public class Test {
    public static void main(String[] args) {
        EnumInstance instance = EnumInstance.getInstance();
        instance.printTest();
    }
}
```

## 第9章 原型模式

### 9-1 原型模式讲解

定义：原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象

不需要知道任何创建的细节，不调用构造函数

类型：创建型

#### 适用场景

* 类初始化消耗较多资源
* new产生的一个对象需要非常繁琐的过程（数据准备、访问权限）
* 构造函数较复杂
* 循环体重生产大量的对象时

#### 优点

* 原型模式性能比直接new一个对象性能高
* 简化创建过程

#### 缺点

* 必须配备克隆方法```clone()```
* 对克隆复杂对象或对克隆出的对象进行复杂改造时，容易引入风险
* 深拷贝、浅拷贝要运用得当

### 9-2 原型模式coding

```java
@Data
public class Pig implements Cloneable {
    private String name;
    private Date birthday;

    public Pig(String name, Date birthday) {
        this.name = name;
        this.birthday = birthday;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        Pig pig = (Pig) super.clone();               // 浅克隆
        
        // 要向实现深克隆，那些引用类型的成员变量也必须克隆
        // pig.birthday = (Date) this.birthday.clone();
        return pig;
    }
}

public class Test {
    public static void main(String[] args) throws Exception {
        Date birthday = new Date(0L);
        Pig pig1 = new Pig("佩奇", birthday);
        Pig pig2 = (Pig) pig1.clone();

        System.out.println(pig1);      // Pig(name=佩奇, birthday=Thu Jan 01 08:00:00 CST 1970)
        System.out.println(pig2);      // Pig(name=佩奇, birthday=Thu Jan 01 08:00:00 CST 1970)

        pig1.getBirthday().setTime(666666L);

        System.out.println(pig1);      // Pig(name=佩奇, birthday=Thu Jan 01 08:11:06 CST 1970)
        System.out.println(pig2);      // Pig(name=佩奇, birthday=Thu Jan 01 08:11:06 CST 1970)
    }
}
```

**克隆也会破坏单例**。

解决办法：

```java
@Override
protected Object clone() throws CloneNotSupportedException {
    return getInstance();
}
```

## 第10章 外观模式

### 10-1 外观模式讲解

定义：又叫门面模式，提供了一个统一的接口，用来访问子系统中的一群接口

外观模式定义了一个高层接口，让子系统更容易使用

类型：结构型

#### 适用场景

* 子系统越来越复杂，增加外观模式提供简单调用接口
* 构建多层系统结构，利用外观对象作为每层入口，简化层间调用

#### 优点

* 简化了调用过程，无需了解深入子系统，防止带来风险
* 减少系统依赖、松散耦合
* 更好地划分访问层次
* 符合迪米特法则，即最少知道原则

#### 缺点

* 增加子系统，扩展子系统行为容易引入风险
* 不符合开闭原则

#### 和外观模式相关的设计模式

##### 外观模式和中介模式

外观模式：关注外界和子系统的交互

中介模式：子系统内部之间的交互

##### 外观模式和单例模式

通常把外观模式中的外观对象做成单例模式

##### 外观模式和抽象工厂模式

外观类可以通过抽象工厂获取子系统的实例

### 10-2 外观模式coding

计算机要想开机工作，需要CPU开机，硬盘开机。但是对于使用计算机的人来说，不需要知道CPU怎么开机，硬盘怎么开机，只需要知道计算机只有一个开机按钮，一按电脑就开机了。

```java
public class CPU {
    public void open() {
        System.out.println("open disk");
    }

    public void close() {
        System.out.println("close disk");
    }
}
public class Disk {
    public void open() {
        System.out.println("open disk");
    }

    public void close() {
        System.out.println("close disk");
    }
}

public class Computer {
    private CPU cpu;
    private Disk disk;

    public Computer() {
        this.cpu = new CPU();
        this.disk = new Disk();
    }

    public void open() {
        this.cpu.open();
        this.disk.open();
    }

    public void close() {
        this.cpu.close();
        this.disk.close();
    }
}

public class Test {
    public static void main(String[] args) {
        Computer computer = new Computer();
        computer.open();
        System.out.println("玩一会电脑");
        computer.close();
    }
}
```

#### 适配器、代理、装饰、外观之间的区别

##### 定义：

* 适配器：把一个接口转换成客户期望的类型；
* 代理：为一个对象提供一个替身或者占位符，以控制这个对象的访问；
* 装饰：动态的给一个对象附加责任或者行为
* 外观模式：提供一个统一的接口，用来访问子系统的一群接口。

##### 功能上：

* 适配器：让两个不同类型的接口合作
* 代理：控制被代理对象的访问或延迟创建消耗大的对象的创建时机
* 装饰：动态附加责任，通过委托|组合使模式，避免类中出现很多静态的不需要的代码功能
* 外观：简化接口，更方便的访问子系统

##### 实现上：

* 适配器：创建class A 并实现客户期望的接口，A中拥有被适配者的接口类型引用（Has-a 关系），把接口功能委托给被适配者；适配者和被适配者属于不同的类型；
* 代理：代理类和被代理类实现同一个接口，代理对象是被代理对象的替身，拥有被代理接口的对象引用(Has-a关系)；代理者和被代理者属于同一个类型；
* 装饰：装饰者和被装饰者继承同一个子类，被装饰者也可以装饰其他装饰者，所以装饰者内部拥有一个基类的引用（Has-a）；装饰者和被装饰者属于同一个类型；
* 外观：外观类会提供一个统一的接口，同时也会暴露子系统的接口，让那些需要使用子系统底层功能的客户端调用

## 第11章 装饰者模式

### 11-1 装饰者模式讲解

定义：在不改变原有对象的基础之上，将功能附加到功能上。

提供了比继承更有弹性的替代方案（扩展原有对象功能）

类型：结构型

#### 适用场景

* 扩展一个类的功能或给一个类添加附加职责
* 动态地给一个对象添加功能，这些功能可以再动态地撤销

#### 优点

* 继承的有力补充，比继承灵活，不改变原有对象的情况下给一个对象扩展功能
* 通过使用不同装饰类以及这些装饰类的排列组合，可以实现不同效果
* 符合开闭原则

#### 缺点

* 会出现更多的代码，更多的类，增加程序复杂性
* 动态装饰时，多层装饰时会更复杂

#### 装饰者相关设计模式

* 装饰者模式和代理模式
  * 装饰者模式
    * 关注在一个对象上动态添加方法
    * 把原始对象作为一个**参数传给装饰者的构造器**
  * 代理模式：
    * 关注控制对对象的访问，代理模式中的代理类可以对客户隐藏一个对象的具体信息
    * 在代理类中创建一个对象的实例（**成员变量**）
* 装饰者模式和适配器模式
  * 装饰者模式和适配器模式都可以叫做Wrapper
  * 装饰者和被装饰者可以**实现相同的接口**或者装饰者是被装饰者的子类
  * 适配器模式中，适配器和被适配的类**具有不同的接口**

### 11-2 装饰者模式coding 

![](http://static.zybuluo.com/vermouth9/rjx6m3lwbtiy7wpcoopel2sf/image.png)

```java
public abstract class ABattercake {
    protected abstract String getDescription();
    protected abstract int getPrice();
}

public abstract class AbstractorDecorator extends ABattercake {

    private ABattercake aBattercake;

    public AbstractorDecorator(ABattercake aBattercake) {
        this.aBattercake = aBattercake;
    }

    @Override
    protected String getDescription() {
        return aBattercake.getDescription();
    }

    @Override
    protected int getPrice() {
        return aBattercake.getPrice();
    }
    
//    protected abstract void doSomething();   根据具体业务场景决定有没有这个抽象方法
}

public class EggDecorator extends AbstractorDecorator {

    public EggDecorator(ABattercake aBattercake) {
        super(aBattercake);
    }

    @Override
    protected String getDescription() {
        return super.getDescription() + " 加一个鸡蛋";
    }

    @Override
    protected int getPrice() {
        return super.getPrice() + 1;
    }
}

public class SausageDecorator extends AbstractorDecorator {
    public SausageDecorator(ABattercake aBattercake) {
        super(aBattercake);
    }

    @Override
    protected String getDescription() {
        return super.getDescription() + " 加一根香肠";
    }

    @Override
    protected int getPrice() {
        return super.getPrice() + 2;
    }
}

public class Test {
    public static void main(String[] args) {
        ABattercake aBattercake = new Battercake();
        aBattercake = new EggDecorator(aBattercake);
        aBattercake = new EggDecorator(aBattercake);
        aBattercake = new SausageDecorator(aBattercake);
        System.out.println(aBattercake.getDescription() + ": " + aBattercake.getPrice());
    }
}
```

### 11-3 装饰者模式源码解析

Java IO中的类

```java
public class BufferedReader extends Reader {

    private Reader in;

    public BufferedReader(Reader in, int sz) {
        super(in);
        this.in = in;
    }
}
```

## 第12章 适配器模式

### 12-1 适配器模式讲解

定义：将一个类的接口转换成客户期望的另一个接口

使原本接口不兼容的类可以一起工作

类型：结构型

#### 适用场景

* 已经存在的类，它的方法和需求不匹配时（方法结果相同或相似）
* 不是软件设计阶段考虑的设计模式，是随着软件维护，由于不同产品、不同厂家造成功能类似而接口不相同情况下的解决方案

#### 优点

* 提高类的透明性和复用，现有的类复用但不需要改变
* 目标类和适配器类解耦，提高程序扩展性
* 符合开闭原则

#### 缺点

* 适配器编写过程需要全面考虑，可能会增加系统的复杂性
* 增加系统代码可读的难度

#### 扩展

* 对象适配器
* 类适配器

#### 适配器相关设计模式

* 适配器模式和外观模式
  * 外观模式
    * 定义了新的接口
    * 现有的系统中提供一个更为方便的访问入口
  * 适配器模式
    * 复用了原有的接口
    * 使原来已有的两个接口协同工作

### 12-2 适配器模式coding

#### 类适配器

![](http://static.zybuluo.com/vermouth9/sp6fyctubs7myq2ss8k7z3si/image.png)

通过**继承**获取被适配者的一些方法。

```java
public class Adaptee {
    public void adapteeRequest() {
        System.out.println("被适配者的方法");
    }
}

public interface Target {
    void request();
}

public class Adapter extends Adaptee implements Target {
    @Override
    public void request() {
        super.adapteeRequest();
    }
}

public class ConcreteTarget implements Target {
    @Override
    public void request() {
        System.out.println("concreteTarget目标方法");
    }
}
```

#### 对象适配器

![](http://static.zybuluo.com/vermouth9/jm7fxk7qy1rvon820g6kfa1j/image.png)

通过**组合**来获取被适配者的方法

```java
public class Adaptee {
    public void adapteeRequest() {
        System.out.println("被适配者的方法");
    }
}


public interface Target {
    void request();
}

public class Adapter implements Target {

    private Adaptee adaptee = new Adaptee();

    @Override
    public void request() {
        adaptee.adapteeRequest();
    }
}

public class ConcreteTarget implements Target {
    @Override
    public void request() {
        System.out.println("concreteTarget目标方法");
    }
}
```

### 12-3 适配器模式源码解析

## 第13章 享元模式

### 13-1 享元模式讲解

定义：提供了减少对象数量从而改善应用所需的对象结构的方式

运用共享技术有效地支持大量细粒度的对象

类型：结构型

#### 适用场景

* 应用与系统底层的开发，以便解决系统的性能问题（String，数据库连接池）
* 系统有大量相似对象，需要缓冲池的场景

#### 优点

* 减少对象的创建，降低内存中对象的数量
* 减少内存之外的其他资源占用

#### 缺点

* 关注内部、外部状态，关注线程安全问题
* 使系统、程序的逻辑复杂化

#### 扩展

* 内部状态：在享元对象的内部，并且不会随着环境改变而改变的共享部分
* 外部状态：随着环境改变而改变的部分

比如画圆形，此时圆形的半径就是外部状态，但是π就是内部状态。

#### 享元模式相关的设计模式

* 享元模式和单例模式
  * 容器单例就是享元模式和单例模式的一个结合。

### 13-2 享元模式coding

```java
public interface Employee {
    void report();
}

public class Manager implements Employee {
    private String department;             // 外部状态
    private String reportContent;
    private String title = "部门经理";      // 内部状态

    public Manager(String department) {
        this.department = department;
    }
    @Override
    public void report() { System.out.println(reportContent); }
    public void setReportContent(String reportContent) { this.reportContent = reportContent; }
}

public class EmployeeFactory {
    private EmployeeFactory() {}
    private static final Map<String, Employee> EMPLOYEE_MAP = new HashMap<>();

    public static Employee getManager(String department) {
        Manager manager = (Manager) EMPLOYEE_MAP.get(department);
        if (manager == null) {
            manager = new Manager(department);
            System.out.println("创建部门经理：" + department);

            manager.setReportContent(department + "部门汇报");
            System.out.println("创建报告");

            EMPLOYEE_MAP.put(department, manager);
        }
        return manager;
    }
}
```

### 13-3 享元模式源码解析

jdk中的```Integer```类

```java
public final class Integer extends Number implements Comparable<Integer> {
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
    
    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;
        }

        private IntegerCache() {}
    }
}

Integer a = Integer.valueOf(100);
Integer b = 100;
Integer c = Integer.valueOf(1000);
Integer d = 1000;
System.out.println(a == b);			// true
System.out.println(c == d);			// false
```

> IntegerCache.low == -128; IntegerCache.high == 127
>
> IntegerCache.high的值可以修改

## 第14章 组合模式

### 14-1 组合模式讲解

定义：将对象组合成树形结构以表示“部分-整体”的层次结构

组合模式使客户端对单个对象和组合对象保持一致的方式处理

类型：结构型

#### 适用场景

* 希望客户端可以忽略组合对象与单个对象的差异时
* 处理一个树形结构时

#### 优点

* 清楚地定义分层次的复杂对象，表示对象的全部或部分层次
* 让客户端忽略了层次的差异，方便对整个层次结构进行控制
* 简化客户端代码
* 符合开闭原则

#### 缺点

* 限制类型时会较为复杂
* 使设计变得更加抽象

### 14-2 组合模式coding

![](http://static.zybuluo.com/vermouth9/8mc6qzqpj3v8viq8n3nnby1d/%E6%8D%95%E8%8E%B7.PNG)

```java
public abstract class CatalogComponent {
    public void add(CatalogComponent catalogComponent) {
        throw new UnsupportedOperationException("不支持添加操作");
    }
    public void remove(CatalogComponent catalogComponent) {
        throw new UnsupportedOperationException("不支持删除操作");
    }
    public String getName(CatalogComponent catalogComponent) {
        throw new UnsupportedOperationException("不支持获取名称操作");
    }
    public double getPrice(CatalogComponent catalogComponent) {
        throw new UnsupportedOperationException("不支持获取价格操作");
    }
    public void print() {
        throw new UnsupportedOperationException("不支持打印操作");
    }
}

public class Course extends CatalogComponent {
    private String name;
    private double price;

    public Course(String name, double price) {
        this.name = name;
        this.price = price;
    }

    @Override
    public String getName(CatalogComponent catalogComponent) { return this.name; }

    @Override
    public double getPrice(CatalogComponent catalogComponent) { return price; }

    @Override
    public void print() {
        System.out.println("Course name: " + name + ", Course price: " + price);
    }
}

public class CourseCatalog extends CatalogComponent {
    private List<CatalogComponent> items = new ArrayList<>();
    private String name;
    private Integer level;

    public CourseCatalog(String name, Integer level) {
        this.name = name;
        this.level = level;
    }

    @Override
    public void add(CatalogComponent catalogComponent) { items.add(catalogComponent); }

    @Override
    public void remove(CatalogComponent catalogComponent) { items.remove(catalogComponent); }

    @Override
    public void print() {
        System.out.println(this.name + "下：");
        for (CatalogComponent catalogComponent : items) {
            if (this.level != null) {
                for (int i = 0; i < this.level; i++) {
                    System.out.print("\t");
                }
            }
            catalogComponent.print();
        }
    }

    @Override
    public String getName(CatalogComponent catalogComponent) { return this.name; }
}

public class Test {
    public static void main(String[] args) {
        CatalogComponent root = new CourseCatalog("课程目录", 1);
        CatalogComponent sbDir = new CourseCatalog("Spring Boot目录", 2);
        CatalogComponent jsDir = new CourseCatalog("JavaScript目录", 2);
        CatalogComponent sbCour1 = new Course("Spring Boot入门", 10);
        CatalogComponent sbCour2 = new Course("Spring Boot进阶", 20);
        CatalogComponent jsCour1 = new Course("JavaScript课程", 10);
        CatalogComponent vueDir = new CourseCatalog("Vue课程目录", 3);
        CatalogComponent vueCour1 = new Course("Vue入门", 10);
        CatalogComponent vueCour2 = new Course("Vue进阶", 10);
        CatalogComponent jsCour3 = new Course("NodeJS课程", 10);
        CatalogComponent aiCour = new Course("ai课程", 100);

        root.add(sbDir);
        root.add(jsDir);
        root.add(aiCour);
        sbDir.add(sbCour1);
        sbDir.add(sbCour2);
        jsDir.add(jsCour1);
        jsDir.add(vueDir);
        jsDir.add(jsCour3);

        vueDir.add(vueCour1);
        vueDir.add(vueCour2);

        root.print();
    }
}

/*
 * 课程目录下：
 * 	Spring Boot目录下：
 * 		Course name: Spring Boot入门, Course price: 10.0
 * 		Course name: Spring Boot进阶, Course price: 20.0
 * 	JavaScript目录下：
 * 		Course name: JavaScript课程, Course price: 10.0
 * 		Vue课程目录下：
 * 			Course name: Vue入门, Course price: 10.0
 * 			Course name: Vue进阶, Course price: 10.0
 * 		Course name: NodeJS课程, Course price: 10.0
 * 	Course name: ai课程, Course price: 100.0
 */
```

### 14-3 组合模式源码解析

![](http://static.zybuluo.com/vermouth9/i6mzck2rckc98f6j525ikq4v/image.png)

## 第15章 桥接模式

### 15-1 桥接模式讲解

定义：将抽象部分与具体实现部分分离，使它们都可以独立地变化

通过组合的方式建立两个类之间联系，而不是继承

类型：结构型

#### 适用场景

* 抽象和具体实现之间增加更多的灵活性
* 一个类存在两个或多个独立变化的维度，且这多个维度都需要独立进行扩展
* 不希望使用继承，或因为多层继承导致系统类的个数剧增

#### 优点

* 分离抽象部分及其具体实现部分
* 提高了系统的可扩展性
* 符合合成复用原则

#### 缺点

* 增加了系统的理解与设计难度
* 需要正确地识别出系统中两个独立变化的维度

#### 桥接模式相关设计模式

* 桥接模式和组合模式
  * 组合模式更强调部分和整体间的组合
  * 桥接模式强调平行类间的组合
* 桥接模式和适配器模式
  * 共同点：为了让两个类配合工作
  * 不同点
    * 适配器模式：改变已有类的接口
    * 桥接模式：分离抽象和具体的实现

### 15-2 桥接模式coding

这是两个独立的继承体系。

![](http://static.zybuluo.com/vermouth9/t1s9hj18kb5b4m11ybpmahnd/image.png)

```java
public interface Account {
    void openAccount();
    void showAccountType();
}

public abstract class Bank {
    protected Account account;

    public Bank(Account account) {
        this.account = account;
    }

    public abstract Account openAccount();
}

public class DepositAccount implements Account {
    @Override
    public void openAccount() {
        System.out.println("打开定期账号");
    }

    @Override
    public void showAccountType() {
        System.out.println("这是一个定期账号");
    }
}

public class SavingAccount implements Account {
    @Override
    public void openAccount() {
        System.out.println("打开活期账号");
    }

    @Override
    public void showAccountType() {
        System.out.println("这是一个活期账号");
    }
}

public class ABCBank extends Bank {
    public ABCBank(Account account) {
        super(account);
    }

    @Override
    public Account openAccount() {
        System.out.println("打开中国农业银行账号");
        account.openAccount();
        return account;
    }
}

public class ICBCBank extends Bank {
    public ICBCBank(Account account) {
        super(account);
    }

    @Override
    public Account openAccount() {
        System.out.println("打开工商银行账号");
        return account;
    }
}
```











