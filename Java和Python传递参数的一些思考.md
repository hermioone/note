# Java和Python传递参数的一些思考

Java和Python传递参数的方式类似，既不是值传递，也不是引用传递，而是传递***对象引用***。这种方式相当于传值和传引用的综合。

## python
在python中：

* 如果函数收到的是一个可变对象（比如列表，字典等）的引用，就可以修改对象的原始值。

  ```python
  a = [1, 2, 3, 4]
  
  def fun(a1):
      a1.append(5)
  
  fun(a)          
  print(a)		# [1, 2, 3, 4, 5]
  # 因为append是修改自身
  ```

* 如果函数收到的是一个不可边对象（比如数字，字符，元组等）的引用，就不能直接修改原始对象

  ```python
  a = (1, 2, 3, 4)
  
  def fun2(a2):
      a2 = a2 + (6)
  
  fun2(a)         
  print(a)		# (1, 2, 3, 4)
  # 因为元组不可变，所以元组没有append()方法，+ 是创建一个新的元组，相当于把形参a2指向一个新的地址
  ```

## java

java的情况和python类似：

在java中，所有的基础类型及其对应的包装类的变量都可以看作是python中的不可变对象，所以如果函数收到的是一个基础类型的变量，那么无法直接修改原始值。

```java
public class ParamTest {

    public static void test1(Integer a) {
        a = 254356465;
    }

    public static void main(String[] args) {
        Integer a = 132313213;
        test1(a);
        System.out.println(a);
    }
}
```

如果函数收到的是除了基础类型（及其包装类）之外的参数，可以修改原始的参数值。