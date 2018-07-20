# JavaScript深入浅出

## 第1章 数据类型

### 1-2 JavaScript六种数据类型

* 原始类型
  1. number
  2. string
  3. boolean
  4. null
  5. undefined
* 对象
  6. Object
     * Function
     * Array
     * Date
     * ...

### 1-3 JavaScript隐式转换

```js
var x = 'The answere iis ' + 42;
'37' - 7  //30
'37' + 7  //377
```

```js
"1.2.3" == 1.23
0 == false
null == undefined
```

严格等于：```===```

* 类型不同，返回false
* 类型相同：
  * null === null
  * undefined === undefined
  * NaN !== NaN
  * new Object() != new Object()

不严格等于：```==```

* 类型相同，同```===```
* 类型不同，尝试类型转换和比较：
  * null == undefined
  * number == string：string转number
  * boolean == ?：布尔值转数字，true == 1, false == 0
  * object == number | string：尝试对象转为基本类型

### 1-4 包装对象

```js
var str = "leihou"
// str是基本类型（string）

var strObj = new String("leihou")
// strObj是对象类型， String{}

console.log(str.length)			// 6
str.t = 3
console.log(str.t)				// undefined
```

当把基础类型以对象的方法使用（比如```str.length```，```str.t```）时，JavaScript会把基本类型转化成对应的包装类型，当完成这样的访问以后（比如```str.t```设置以后），这个临时对象会被销毁掉。所以给str的t属性设置值后，这个临时的包装类型就会被销毁，当要取str的t属性时，又创建了一个新的包装对象，所以```str.t == undefined```，

基本类型和包装对象的对应关系：

* string：String
* number：Number
* boolean：Boolean

**String，Number和Boolean都是Function**

### 1-5 类型检测

类型检测的方法：

* ```typeof```：

  ```js
  typeof 100 === "number"
  typeof(true) === "boolean"
  typeof String === "function"
  typeof undefined === "undefined"
  typeof(new Object()) === "object"
  typeof [1, 2] === "object"
  typeof NaN === "number"
  typeof null === "object"
  ```

  typeof没法判断对象具体是什么类型，因为只能返回"object"

* ```instanceof```

  基于原型链去判断的一个操作符

  instanceof的左操作数是一个对象，右操作数是一个函数构造器。判断左边这个对象的原型链上是否有右边这个构造函数的prototype属性

  ```js
  [1, 2] instanceof Array === true
  new Object() instanceof Array === false
  ```

* ```Object.prototype.toString```

  ```js
  Object.prototype.toString.apply([]) === "[object Array]"
  Object.prototype.toString.apply(function(){}) === "[object Function]"
  Object.prototype.toString.apply(null) === "[object Null]"
  Object.prototype.toString.apply(undefined) === "[object Undefined]"
  
  // IE6,7,8中Object.prototype.toString.apply(null) === "[object Object]
  ```

* ```constructor```

* ```duck type```

```typeof```适合基本类型及function检测，遇到null失效

```Object.prototype.toString()```适合内置对象和基于类型。

```instanceof```适合自定义对象，也可以用来检测原生对象

