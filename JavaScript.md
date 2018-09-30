# JavaScript知识点总结

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

## 第2章 JS基础知识（上）

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

## 第3章 JS基础知识（中）

### 3-1 作用域和闭包-执行上下文

* 题目

  * 说一下对变量提升的理解

    * 变量定义
    * 函数声明（注意和函数表达式的区别）

  * 说明this几种不同的使用场景

  * 创建10个```<a>```标签，点击的时候弹出来对应的序号

    ```js
    //这是一个错误的写法
    var i, a;
    for (i = 0; i < 10; i++) {
        a = document.createElement('a');
        a.innerHTML = i + '<br/>';
        a.addEventListener('click', function (e) {
            e.preventDefault();
            alert(i);		//当执行onClick()方法时，i是自由变量，去父作用域去找，此时i === 10
        })
        document.body.appendChild(a);
    }
    
    //正确写法是：
    var i;
    for (i = 0; i < 10; i++) {
        (function (i) {
            var a = document.createElement('a');
            a.innerHTML = i + '<br/>';
            a.addEventListener('click', function (e) {
                e.preventDefault();
                alert(i);		
            })
            document.body.appendChild(a);
        })(i);   	//相当于每次循环都生成一个函数
    }
    ```

  * 如何理解作用域

  * 实际开发中闭包的应用

    * 封装变量

    * 收敛权限

      ```js
      function isFirstLoad() {
          var _list = [];
          return function (id) {
              if(_list.indexOf(id) >= 0) {
                  return false;
              }else {
                  _list.push(id);
                  return true;
              }
          }
      }
      
      //这样可以防止_list被人修改，isFirstLoad函数外面根本不可能修改掉 _list 的值
      var firstLoad = isFirstLoad();
      firstLoad(10);		//true
      firstLoad(10);		//false
      firstLoad(20);		//true
      ```

* 知识点

  * 执行上下文

    ```js
    //在这个script里面，会定义一个全局的执行上下文
    //执行第一行之前，会把这个script中的所有的变量和函数的声明都拿过来：a（还没有赋值），fn（把整个函数拿过来）。
    console.log(a);		
    // undefined：因为a已经声明了，但是还没执行到，所以没有赋值，此时就用undefined占位
    var a = 100;
    
    fn('zhangsan');		
    // 'zhangsan' 20：因为执行fn()之前已经把整个函数都拿出来了，所以可以执行
    function fn(name) {
        age = 20;
        console.log(name, age);
        var age;
    }
    ```

    ```js
    //上诉代码相当于：
    var a;
    function fn(name) {
        //var this;		//自身的引用
        //var name;		//参数name值
        //var age;		/*在执行之前会相当于有这几个的声明*/
        age = 20;
        console.log(name, age);
    }
    ```

    

    * 范围执行上下文：一段```<script>```或者一个函数
    * 全局执行上下文：变量定义、函数声明
    * 函数执行上下文：变量定义、函数声明、this、arguments

    **注**：注意函数声明和函数表达式的区别

    ```js
    var fn1 = function () {}		//函数表达式
    function fn2() {}				//函数声明
    
    fn()		//不会报错		
    function fn() {}
    
    fn2()		//会报错，因为在最开始相当于有一个var fn2 = undefined
    var fn2 = function() {}
    ```

  * this

    **this要在执行时才能确认值，定义时无法确认**

    ```js
    var a = {
        name: 'A',
        fn: function () {
            console.log(this.name);
        }
    }
    a.fn();  // this === a
    a.fn.call({name: 'B'});		//this === {name: 'B'}
    var fn1 = a.fn;
    fn1();		// this === window
    ```

    * 作为构造函数执行

      ```js
      function Foo(name) {
          // this = {}
          this.name = name;
          //return this;		//等价于加上注释的这两行
      }
      var f = new Foo('zhangsan');
      ```

    * 作为对象属性执行

      ```js
      var obj = {
          name: 'A',
          printName: function () {
              console.log(this.name);
          }
      }
      obj.printName()		//此时this就是obj
      ```

    * 作为普通函数执行：window

      ```js
      function fn() { console.log(this); }
      fn();			//this === window
      ```

    * call、apply、bind：**最常用的是call**

      ```js
      function fn1(name, age) { 
      	alert(name);
          console.log(this); 
      }
      fn1.call({x: 100}, 'zhangsan', 20);			//this === {x: 100}, name = 'zhangsan'
      fn1.apply({x: 100}, ['zhangsan', 20]);		//更常用call
      
      var fn2 = function (name, age) { 
      	alert(name);
          console.log(this); 
      }.bind({y:200})				//.bind必须是函数表达式
      fn2('zhangsan', 20);		//此时 this === {y: 200}
      ```

  * 作用域

    * JS没有块级作用域

    * 只有函数和全局作用域

      ```js
      //无块级作用域
      if (true) {
          var name = 'zhangsan';
      }
      console.log(name);
      
      //函数和全局作用域
      var a = 100;
      function fn() {
          var a = 200;
          console.log('fn', a);		//fn 200
      }
      console.log('global', a);		//global 100
      fn();
      console.log('global', a);		//global 100
      ```

  * 作用域链

    作用域链：一个自由变量在**执行时**一直不断地往父级作用域中去找。

    * **函数的父级作用域是函数定义时候的父级作用域，不是执行的时候的**。函数在哪定义，那父级作用域就在哪。

      ```js
      var a = 100;
      function fn() {
          var b = 200;
          
          //当前作用域没有定义的变量，即“自由变量”
          //执行时自由变量去父级作用域中去找
          //fn的父级作用域就是全局作用域
          console.log(a);
          console.log(b);
      }
      fn();
      ```

      ```js
      var a = 100;
      function F1() {
          var b = 200;
          function F2() {		//F2的父级作用域是F1
              var c = 300;
              console.log(a);	//a 是自由变量
              console.log(b);	//b 是自由变量
              console.log(c);
          }
          F2();
      }
      F1();
      ```

  * 闭包

    闭包的使用场景：

    * 函数作为返回值

      ```js
      function F1() {
          var a = 100;		//F1作用域中的a
          
          //返回一个函数（函数作为返回值）
          return function() {
              console.log(a);			//a是个自由变量，去父级作用域中去找
          }
      }
      var f1 = F1();
      var a = 200;		//全局作用域中的a，和F1作用域中的a无关
      f1();			// 100
      ```

    * 函数作为参数传递

      ```js
      function F1() {
          var a = 100;		//F1作用域中的a
          return function() {
              console.log(a);			//a是个自由变量，去父级作用域中去找
          }
      }
      var f1 = F1();
      
      function F2(fn) {
          var a = 200;
          fn();
      }
      var a = 300;
      F2(f1);		// 100
      ```

## 第4章 JS基础知识（下）

### 4-1 异步和单线程-什么是异步

* 题目

  * 同步和异步的区别是什么？分别举一个同步和异步的例子

    同步会阻塞代码执行，而异步不会。

  * 一个关于setTimeout的笔试题

    ```js
    console.log(1);
    setTimeout(function () {
        console.log(2);
    });
    console.log(3);
    setTimeout(function () {
        console.log(4);
    }, 1000);
    console.log(5);
    //1 3 5 2 4
    ```

  * 前端使用异步的场景有哪些

* 知识点

  * 什么是异步

    ```js
    console.log(100);
    setTimeout(function () {
        console.log(200);
    }, 1000);
    console.log(300);
    ```

  * 前端使用异步的场景

    * 定时任务：setTimeout，setInterval
    * 网络请求：ajax请求，动态```<img>```加载
    * 事件绑定

  * 异步和单线程

    ```js
    console.log(100);
    setTimeout(function () {
        console.log(200);
    });
    console.log(300);
    //执行结果是100， 300， 200
    ```

    上例的执行流程：

    1. 执行第一行，打印100；
    2. 执行setTimeout后，传入setTimeout的函数会被暂存起来，不会立即执行（单线程的特点，不能同时干两件事）；
    3. 执行最后一行，打印300；
    4. 待所有程序执行完，处于空闲状态时，会立马看有没有暂存起来的要执行；
    5. 发现暂存起来的setTimeout中的函数无需等待时间，就立即执行。

* 其他知识点

  * 日期

    ```js
    Date.now();			//获取当前时间毫秒数，Date是构造函数
    var dt = new Date();
    dt.getTime();		//获取毫秒数
    dt.getFullYear();	//年
    dt.getMonth();		//月（0-11）
    dt.getDate();		//日（0-31）
    dt.getHours();		//小时
    dt.getMinutes();	//分钟（0-59）
    dt.getSeconds();	//秒（0-59）
    ```

  * Math

    ```js
    Math.random();		// > 0, < 1，一般是用来清除缓存
    ```

  * 数组API

    * ```forEach```：遍历所有元素

    * ```every```：判断所有元素是否都符合条件

    * ```some```：判断是否有至少一个元素符合条件

    * ```sort```：排序

    * ```map```：对元素重新组装，生成新数组

    * ```filter```：过滤符合条件的元素

      ```js
      var arr = [4, 6, 5, 2, 3];
      arr.forEach(function (item, index) {
          //遍历数组的所有元素
          console.log(index, item);
      });
      
      var result = arr.every(function (item, index) {
          //用来判断所有的数组元素，都满足一个条件
          if (item < 4) {
              return true;
          }
      });
      console.log(result);			//false
      
      result = arr.some(function (item, index) {
          //用来判断所有的数组元素，只要有一个满足一个条件即可
          if (item < 4) {
              return true;
          }
      });
      console.log(result);			//true
      
      var arr2 = arr.sort(function (a, b) {
          //从小到大排序
          return a - b;
          
          //从大到小排序
          //return b - a;
      });
      console.log(arr2); //[2, 3, 4, 5, 6]
      
      arr2 = arr.map(function (item, index) {
          //将元素重新组装，并返回
          return item + 'a';
      });
      console.log(arr2);			//["4a", "6a", "5a", "2a", "3a"]
      
      arr2 = arr.filter(function (item ,index) {
          //通过某一个条件过滤数组
          if (item >= 4) {
              return true;
          }
      });
      console.log(arr2);			//[4, 6, 5]
      ```

  * 对象API

    ```js
    var obj = {
        x: 100,
        y: 200,
        z: 300
    };
    
    var key;
    for (key in obj) {
        //注意这里的hasOwnProperty，再讲原型链时候讲过
        if (obj.hasOwnProperty(key)) {
            console.log(key, obj[key]);
        }
    }
    ```

## 第5章 JS-Web-API（上）

### 5-1 从基础知识到JSWebAPI

常说的JS（浏览器执行的JS）包含两部分：

* JS基础知识：ECMA 262标准

* JS-Web-API：W3C标准

  * DOM操作

  * BOM操作

  * 事件绑定

  * ajax请求（包括http协议）

  * 存储

    页面弹框是```window.alert(123)```，浏览器需要做：

    * 定义一个window全局变量，对象类型
    * 给它定义一个```alert```属性，属性值是一个函数

    获取元素```document.getElementById(id)```，浏览器需要：

    * 定义一个```document```全局变量，对象类型
    * 给它定义一个```getElementById```属性，属性值是一个函数

    > 但是W3C标准没有规定任何JS基础相关的东西，只管定义用于浏览器中JS操作页面的API和全局变量

### 5-2 DOM本质

* 题目

  * DOM是哪种基本的数据结构

    树

* 知识点

  * DOM本质

    html是一种特殊的xml（有一些固定的规则）。

  * DOM节点操作

    * 获取DOM节点

    * prototype（属性）

      ```js
      var pList = document.querySelectorAll('p');		//获取所有p标签
      var p = pList[0];			//p本质上是个js对象
      console.log(p.style.width);		//获取样式
      p.style.width = '100px';		//修改样式
      console.log(p.className);
      
      //style和className等都是p的prototype
      ```

    * Attribute

      ```js
      var pList = document.querySelectorAll('p');		//获取所有p标签
      var p = pList[0];			//p本质上是个js对象
      p.getAttribute('data-name');
      p.setAttribute('style', 'font-size:30px;');
      
      //这里的attribute是html文档标签的属性，不是js对象的属性
      <p data-name="" style="font-size:20px;"></p>
      ```

  * DOM结构操作

    * 新增节点

      ```js
      var p1 = document.createElement('p');
      ```

    * 获取父元素

      ```js
      var parent = div1.parentElement;
      ```

    * 获取子元素

      ```js
      var childs = div.childNodes;
      ```

    * 删除节点

### 5-8 BOM操作-知识点

* 题目

  * 如何监测浏览器的类型

    ```js
    var ua = navigator.userAgent
    var isChrome = ua.indexOf('Chrome');
    
    //screen
    console.log(screen.width);
    
    //location
    console.log(location.href);
    ```

  * 拆解url的各部分

## 第6章 JS-Web-API（下）

### 6-1 事件-知识点

* 知识点

  * 通用事件绑定

  * 事件冒泡

    ```html
    <!-- 点击激活弹出激活。点击取消弹出取消 -->
    <body>
        <div id="div1">
            <p id="p1">取消</p>
            <p id="p2">激活</p>
            <p id="p3">激活</p>
            <p id="p4">激活</p>
        </div>
        <div id="div2">
            <p id="p5">取消</p>
            <p id="p6">取消</p>
        </div>
    </body>
    
    <script>
        var p1 = document.getElementById("p1");
        var body = document.body;
    
        bindEvent(p1, 'click', function (e) {
            e.stopPropatation();
            alert("激活");
        });
        bindEvent(dody, 'click', function (e) {
            alert("取消");
        });
        
        function bindEvent(elem, type, selector, fn) {
            if (fn == null) {
                fn = selector;
                selector = null;
            }
    
            elem.addEventListener(type, function (e) {
                var target;
                if (selector) {     //使用代理
                    target = e.target;
                    if (target.matches(selector)) {
                        fn.call(target, e);
                    }
                }else {         //不使用代理
                    fn(e);
                }
            })
        }
    </script>
    ```

  * 代理

    ```html
    <body>
        <div id="div1">
            <p id="p1">取消</p>
            <p id="p2">激活</p>
            <p id="p3">激活</p>
            <p id="p4">激活</p>
        </div>
        <div id="div2">
            <p id="p5">取消</p>
            <p id="p6">取消</p>
        </div>
    </body>
    
    <script>
        var p1 = document.getElementById("p1");
        var body = document.body;
    
        bindEvent(body, 'click', 'a', function (e) {
            console.log(this.innerHTML);
        });
    </script>
    ```

### 6-4 Ajax-XMLHttpRequest

* 题目：

  * 跨域的几种实现方式

* 知识点

  * XMLHttpRequest

  * 状态码说明

    ![](http://static.zybuluo.com/vermouth9/wyzylnflb9s3cge5grhdsmh9/image.png)

  * 跨域

    * 什么是跨域

      > 浏览器有同源策略，不允许ajax访问其他域接口

      跨域条件：协议、域名、端口，有一个不同就算跨域

      有三个标签允许跨域加载资源

      * ```<img src='xxx'>```
      * ```<link href='xxx'>```
      * ```<script src='xxx'></script>```

      所有跨域请求都必须经过信息提供方允许。

      如果未经允许即可获取，那是浏览器同源策略出现漏洞。

    * 解决方案：

      * JSONP

        ```html
        <script>
            window.callback = function (data) {
                console.log(data);
            }
        </script>
        <script src="http://coding.m.imooc.com/api.js"></script>
        <!-- 以上将返回callback({x:100, y:200})，也就是返回的是  
        		调用callback()的代码，传入的参数是{x:100, y: 200} 
        -->
        ```

        这个方法需要前端和后端的协商，后端需要返回JS的一个片段（调用页面中的某个函数，同时传入参数），但是需要前端页面中有这个函数。

      * 服务器端设置http header

        需要服务器端来做。

### 6-6 存储

* 题目

  * 请描述一下cookie，sessionStorage和localStorage的区别。

* 知识点

  * cookie

    本身用于客户端和服务器端通信

    但是它有本地存储的功能，于是就被“借用”

    使用```document.cookie=```来获取和设置

    * 缺点：
      * 存储量太小，只有4KB
      * 所有http请求都带着，会影响获取资源的效率
      * API简单，需要封装才能用 

  * localStorage和sessionStorage

    * HTML专门为存储而设计，最大容量5M（因为不需要在请求中带着）
    * API简单易用
    * ```localStorage.setItem(key, value);```，```localStorage.getItem(key);```

## 第7章 开发环境

### 7-3 git常用命令

* ```git add.```
* ```git checkout xxx```：还原回去
* ```git commit -m "xxx"```：把改动提交到本地
* ```git push origin master```：把改动提交到远程仓库
* ```git pull origin master```：从远程仓库更新本地代码。













