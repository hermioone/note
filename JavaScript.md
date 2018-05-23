# JavaScript

## 第1章 课程简介

### 1-3 

* JS中使用typeof能得到的哪些类型

  => JS变量类型

* 何时使用 === 何时使用 ==

  => 强制类型转换

* ```window.onload```和```DOMContentLoaded```的区别

  => 浏览器渲染过程

* 用JS创建10个```<a>```标签，点击的时候弹出来对应的序号

  => 作用域

* 简述如何实现一个模块加载器，实现类似```require.js```的基本功能

  => JS模块化

* 实现数组的随机排序

  => JS继承算法

## 第2章 JS基础知识(上)

### 2-1 变量类型和计算

* 题目

  * JS中使用typeof能得到哪些类型

  * 何时使用 === 何时使用 ==

    * ==中有类型转换，===中没有类型转换
    * 用==的情况：```obj.a == null```，这里相当于```obj.a === null || obj.a === undefined```的简写形式
    * 除了上面的那种情况外，其他全都用===

  * JS中有哪些内置函数

    * ```Object()```、```Array()```、```Boolean()```、```Number()```、```String()```、```Function()```、```Date()```、```RegExp()```、```Error()```

  * JS变量按照存储方式区分为哪些类型，并描述其特点

    * 值类型和引用类型

  * 如何理解JSON

    ```js
    // JSON 只不过是一个JS对象而已
    
    JSON.stringify({a:"10", b:20})
    JSON.parse('{"a":10, "b":20}')
    ```

* 知识点

  * 变量类型

    * 值类型 vs 引用类型

      ```js
      var a = 100;
      var b = a;			// 这里a和b里都存储的是100
      a = 200;
      console.log(b);		// 100
      //值类型：把值分块存在内存中
      
      var a = {age:20};
      var b = a;			// 这里a和b都是指针，指向另一个内存(真实对象)
      b.age = 21;
      console.log(a.age);	// 21
      
      //引用类型：对象、数组、函数
      //引用类型：好几个变量共用一个内存块，节省内存空间
      //引用类型特点：无限扩展属性
      ```

    * ```typeof```运算符详解

      ```javascript
      typeof undefined			//undefined
      typeof 'abc';				//string
      typeof 123;					//number
      typeof true;				//boolean
      typeof {};					//object
      typeof [];					//object
      typeof null;				//object
      typeof console.log;			 //function
      
      //typeof只能区分值类型的详细类型（前四个），但是针对引用类型就无能为力（除了可以区分出函数）
      ```

  * 变量计算

    * 强制类型转换

      * 字符串拼接

        ```js
        var a = 100 + 10;		//110
        var b = 100 + '10';		//'10010'
        ```

      * == 运算符

        ```js
        100 == '100';		//true
        0 == '';			//true
        null == undefined;	 //true
        ```

      * if语句

        ```js
        var a = true;
        if (a) {}
        var b = 100
        if (b) {}			// 100 -> true
        var c = ''
        if (c) {}			// '' -> false
        ```

      * 逻辑运算

        ```js
        console.log(10 && 0);		// 0: true && 0
        console.log('' || 'abe');	// 'abc': false || 'abc'
        console.log(!window.abc);	// true
        
        var a = 100;
        console.log(!!a);			//true
        //0, '', null, undefined放在if中都会被转成false
        ```

### 2-5 原型和原型链-构造函数

* 题目

  * 如何准确判断一个变量是数组类型
    * ```arr instanceof Array```
  * 写一个原型链继承的例子
  * 描述new一个对象的过程
  * zepto（或其他框架）源码中如何使用原型链

* 知识点

  * 构造函数

    ```js
    function Foo(name, age) {
        this.name = name;
        this.age = age;
        this.class = 'class-1';
    	// return this;		//默认有这一行，解开注释也无所谓
    }
    
    var f = new Foo('zhangsan', 20);
    // var f1 = new Foo('lisi', 22);		//创建多个对象 
    
    //大小字母开头的函数基本上就是构造函数，自己写代码的时候如果函数是构造函数，那也要大写字母开头
    
    //执行流畅：
    	//1.this先变成空对象
    	//2.给this赋各种属性
    	//3.把this返回
    ```

  * 构造函数 - 扩展

    * ```var a = {}```其实是```var a = new Object()```的语法糖
    * ```var a = []```其实是```var a = new Array()```的语法糖
    * ```function Foo(){...}其实```是```var Foo = new Function(...)```的语法糖
    * 上面三个例子都推荐前面的写法
    * ***使用instanceof判断一个函数是否是一个变量的构造函数***

  * 原型规则和示例

    共有5条原型规则，原型规则是学习原型链的基础

    1. 所有的引用类型（对象，数组，函数），都具有对象特性，即可自由扩展属性（除了null）

       ```js
       var obj = {}; obj.a = 100;//对象
       var arr = []; arr.a = 100;//数组
       function fn() {}//函数		
       fn.a = 100;
       ```

    2. 所有的引用类型（对象，数组，函数），都有一个```__proto__```属性，属性值是一个普通的对象

       ```__proto__```：隐式原型

    3. 所有的函数，都有一个```prototype```属性，属性值也是一个普通的对象

       ```prototype```：显示原型

       ```js
       console.log(obj.__proto__);
       console.log(arr.__proto__);
       console.log(fn.__proto__);
       
       console.log(fn.prototype);
       ```

    4. 所有的引用类型，```__proto__```属性值指向它的构造函数的```prototype```属性值

       ```js
       console.log(obj.__proto__ === Object.prototype);		//true
       ```

    5. 当试图得到一个引用类型的某个属性时，如果这个引用类型本身没有这个属性，那么会去它的```__proto__```（即它的构造函数的```prototype```）中寻找。

       ```js
       //构造函数
       function Foo(name, age) {
           this.name = name;
       }
       Foo.prototype.alertName = function () {
           alert(this.name);			//this指向对象自身（f）
       }
       
       //创建引用类型
       var f = new Foo('zhangsan');
       f.printName = function () {
           console.log(this.name);			//zhangsan
       }
       
       //测试
       f.printName();
       f.alertName();
       
       var item;
       for (item in f) {		//加上原型中的，f共有三个属性
           //高级浏览器已经在 for in中屏蔽了来自原型的属性
           //但是这里建议还是加上这个判断，保证程序的健壮性
           if(f.hasOwnProperty(item)) {
               console.log(item);		//name，printName
           }
       }
       ```

  * 原型链

    ```js
    //测试
    f.printName();
    f.alertName();
    f.toString();		// 要去 f.__proto__.__proto__中查找（Object中有toString属性)
    // Object.prototype === null
    ```

  * instanceof

    用来判断引用类型属于哪个构造函数的方法

    * ```f instanceof Foo```的判断逻辑是：

      f 的```__proto__```一层一层往上，能否对应到```Foo.prototype```

  







