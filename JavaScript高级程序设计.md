# JavaScript高级程序设计

## 第1章 JavaScript简介

### 1.2 JavaScript实现

一个完整的JavaScript实现包含下面三个部分：

* 核心（ECMAScript）
* 文档对象模型（DOM）
* 浏览器对象模型（BOM）

#### 1.2.1 ECMAScript

ECMAScript和web浏览器没有依赖关系。我们常见的Web浏览器只是ECMAScript实现可能的宿主环境之一。宿主环境不仅提供基本的ECMAScript实现，同时也会提供该语言的扩展（如DOM），以便语言与环境之间对接交互。其他宿主环境包括node。

## 第3章 基本概念

### 3.4 数据类型

> ECMAScript中有6种数据类型：
>
> 	基本类型：Undefined, Nul, Boolean, Number, String
>		
> 	引用类型：Object

#### 3.4.1 typeof操作符

使用```typeof```操作符来检测给定变量的数据类型，可能的返回值为：

* "undefined"：如果这个值声明了但是未初始化
* "boolean"：如果这个值时布尔值
* "string"：如果这个值是字符串
* "number"：如果这个值是数值
* "object"：如果这个值时对象或null
* "function"：如果这个值是函数

> 函数在ECMAScript中是对象，不是一种数据类型。然而，函数也确实有一些特殊的属性，所以通过typeof操作符来区分函数和其他对象是有必要的

#### 3.4.2 Undefined类型

Undefined类型只有一个值，即特殊的**undefined**，在***使用var声明变量但未对其加以初始化时***，这个变量的值就是**undefined**。

```js
var name;
console.log(name === undefined);			// true
```

**注意**：直接使用未声明的变量会报错，但是```typeof```后跟未声明的变量也会返回"undefined"

```js
var age;
console.log(typeof age);			// "undefined"
console.log(typeof job);			// "undefined"

console.log(age);			// undefined
console.log(job);			// Uncaught ReferenceError: job is not defined

// 注意typeof返回的是字符串"undefined"，而直接打印age，显示的是undefined
```

为什么会出现这个结果？

> 因为虽然这两种变量（一种是声明了但未初始化，一种是未声明）从技术角度上有本质区别，但实际上无论对哪种变量都不可能执行真正的操作。

**显示地用```null```初始化变量是明智的选择，这样当用```typeof```操作符返回"undefined"值时，就知道被检测的变量是还没有被声明，而不是尚未初始化。**

#### 3.4.3 Null类型

Null类型也只有一个值，即特殊的```null```。```null```值表示一个空对象指针，这正是使用```typeof```检测null值时会返回"object"的原因。

> 如果变量准备在将来用于保存对象，那么最好将该变量初始化为null

undefined值是派生自null值的，所以下式总是返回true

```js
console.log(null == undefined);
```

#### 3.4.5 Number类型

ECMAScript能够表示的最小数值和最大数值分别是```Number.MIN_VALUE```和```Number.MAX_VALUE```，超过这两个数值的值会自动转换成```Infinity```和```-Infinity```。可以通过```isFinite()```来确定一个数值是不是位于最小值和最大值之间。

```js
var result = Number.MAX_VALUE + Number.MAX_VALUE;
console.log(isFinite(result));			// false
```

##### 3. NaN

非数值，**用来表示一个本来要返回数值的操作数未返回数值的情况（这样就不会抛出错误了，不影响代码继续执行）**。

* 任何涉及NaN的操作都会返回NaN
* NaN和任何值都不相等，包括NaN

可以通过```isNaN()```来判断*是否不是数值*以及*是否可以被转换成数值*

```js
console.log(isNaN(NaN));			// true
console.log(isNaN(10));				// false
console.log(isNaN('10'));			// false，可以被转换成数值10
console.log(isNaN('blue'));			// true，无法转换成数值
console.log(isNaN(true));			// false，可以被转换成数值1
```

##### 4. 数值转换

有3个函数可以把非数值转换为数值：

* ```Number()```：用于任何数据类型
* ```parseInt()```：用于字符串转数值
* ```parseFloat()```：用于字符串转数值

**```Number()```的转换规则**：

* true，false  ==> 1，0
* null  ==> 0
* undefined ==> NaN
* 字符串遵守下列规则：
  * "0123" ==> 123
  * "01.1" ==> 1.1
  * "0xf" ==> 15
  * "" ==> 0
  * 其他情况 ==> NaN
* 对象，则调用对象的```valueOf()```方法，然后依照前面的规则转换返回的值，如果转换的结果是NaN，则调用对象的```toString()```，再依照前面的规则转换返回的字符串值

**```parseInt()```的转换规则**：

```js
console.log(parseInt("1234blue"));		// 1234
console.log(parseInt(""));				// NaN
console.log(parseInt("0xB"));			// 11
console.log(parseInt("B", 16));		   	// 11
console.log(parseInt("22.5"));			// 22
console.log(parseInt("-089"));			// -89
console.log(parseInt("70", 8));			// 56(八进制的70转成十进制的56)
console.log(parseInt("70"));			// 70
```

> 为了避免错误的解析，建议转换时都明确地指定基数

**```parseFloat()```的转换规则**：

```js
console.log(parseFloat("1234blue"));			// 1234
console.log(parseFloat("0xA"));					// 0
console.log(parseFloat("22.5"));				// 22.5
console.log(parseFloat("22.34.5"));				// 22.34
console.log(parseFloat("0908.5"));				// 908.5
console.log(parseFloat("3.125e7"));				// 31250000
```

```parseFloat()```只解析十进制

#### 3.4.6 String类型

##### 2. 字符串的特点

ECMAScript中的字符串是**不可变**的。要改变某个变量保存的字符串，首先要销毁原来的字符串，然后再用新的字符串填充该变量。

##### 3. 转换为字符串

有两种方法：

* ```toString()```：数值、布尔值、对象和字符串都可以用这个方法转成字符串，但是**null值和undefined值没有这个方法**
* ```String()```：可以将任何类型的值转换成字符串
  * 如果值有```toString()```，则调用该方案并返回相应结果
  * null ==> "null"
  * undefined ==> "undefined"

#### 3.4.7 Object类型

ECMAScript中的**Object类型是所有实例的基础**（就行java中的Object一样）。

Object的每个实例都具有下列属性和方法：

* ```constructor```：保存着用于创建当前对象的函数

  ```js
  var person = {};
  var age = 16;
  console.log(person.constructor);		// f Object() { [native code] }
  console.log(age.constructor);			// f Number() { [native code] }
  ```

* ```hasOwnProperty(propertyName)```：用于检查给定的属性在当前对象实例中（而不是在实例的原型中）是否存在

* ```isPrototypeOf(object)```：用于检查传入的对象是否是当前对象的原型

* ```toLocaleString()```：返回对象的字符串表示，该字符串与执行环境的地址对应

* ```toString()```：返回对象的字符串表示

* ```valueOf()```：返回对象的字符串、数值或布尔值表示，通常与```toString()```的返回值相同

### 3.5 操作符

#### 3.5.1 一元操作符

++、--、 +、-

这些一元操作符也可以用来操作非Number类型，此时对值先执行```Number()```再执行这些一元操作符

```js
var s2 = "1.1";	
var b = +s2;			// 1,1
```

#### 3.5.3 布尔操作符

##### 2. 逻辑与

逻辑与操作可以应用与任何类型的操作数：

* {} && xx：返回xx
* true && {}：返回{}
* false && {}：返回false
* {} && {}：返回第2个{}
* null && xx：返回null
* NaN && xx：返回NaN
* undefined && xx：返回undefined

##### 3. 逻辑或

* {} || xx：返回{}
* false || xx：xx
* true || xx：true
* {} || {}：返回第1个{}
* null || null：返回null
* NaN || NaN：返回NaN
* undefined || undefined：返回undefined

```js
var obj = preferredObject || defaultObject;
```

可以采用上例的方式赋值，如果```preferredObject```存在则优先用```preferredObject```的值来赋值，否则采用默认值```defaultObject```来赋值

### 3.7 函数

return语句可以不带有任何返回值，这种情况下，函数会返回undefined值

#### 3.7.1 理解参数

ECMAScript函数不介意传递进来多少个参数，也不在乎传进来的参数是什么数据类型（函数定义时只接收两个参数，但在调用函数时传递几个参数，甚至不传参数都可以）。因为**参数在内部是用一个数组来表示的**。函数接收到的始终是这个数组，而不关心数组中包含哪些参数。在函数体内可以通过```arguments```对象来访问这个参数数组。

**```arguments```对象只是与数组类似，但并不是Array的实例，```arguments.length```可以获知有多少个参数传递给函数**

```js
function doAdd() {
    if(arguments.length === 1) {
        console.log(arguments[0] + 10);
    }else if(arguments.length === 2) {
        console.log(arguments[0] + arguments[1]);
    }
}

doAdd(10);
doAdd(30, 20);
```

#### 3.7.2 没有重载

ECMAScript的函数不能像传统意义上那样实现重载，但是如上例中通过检查传入函数中参数的类型和数量并作出不同的反应，可以模仿方法的重载

## 第4章 变量、作用域和内存问题

### 4.1 基本类型和引用类型的值

ECMAScript变量包含两种不同数据类型的值：**基本类型值**（Undefined、Null、Boolean、Number、String）和**引用类型值**

#### 4.1.1 动态地属性

对于引用类型的值，我们可以为其添加属性和方法，也可以改变和删除其属性和方法

```js
var person = new Object();
person.name = 'John';
console.log(person.name);			// John
```

**不能给基本来写的值添加属性**，尽管这样不会出错

```js
var name = 'Mike';
name.age = 16;
console.log(name.age);				// undefined
```

> 只能给引用类型值动态地添加属性

#### 4.1.2 复制变量值

基本类型值的复制过程

![](http://petfhoepw.bkt.clouddn.com/markdown_2018-09-29_09-56-01.png)

引用类型值的复制过程

![](http://petfhoepw.bkt.clouddn.com/markdown_2018-09-29_09-56-11.png)

#### 4.1.3 传递参数

ECMAScript中所有函数的参数都是**按值传递**的，和变量的复制过程一样。但是访问变量有按值和按引用两种方式。

#### 4.1.4 检测类型

##### 检测基本数据类型

可以使用```typeof```，```typeof```可以确定一个变量是字符串、数值、布尔值还是undefined

##### 检测引用类型

因为检测引用类型时，```typeof```只能区分出来object和function，不能检测出来这个值是什么类型的对象。为此，我们可以使用```instanceof```操作符

> ```instanceof```操作符是根据原型链来判断变量是否是给定引用类型的实例。

* 所有引用类型都是Object的实例，因此在检测一个引用类型值和Object构造函数时，始终返回true
* 当然如果使用```instanceof```操作符检测基本类型时，始终返回false

### 4.2 执行环境及作用域

>  执行环境：定义了变量或函数有权访问的其他数据，决定了它们各自的行为。

每个执行环境都有一个与之关联的***变量对象***，环境中定义的所有变量和函数都保存在这个对象中。虽然我们编写的代码无法访问这个对象，但解析器在处理数据时会在后台使用它。

全局执行环境是最外围的一个执行环境，根据ECMAScript实现所在的宿主环境不同，表示全局执行环境的对象也不一样。在web浏览器中，全局执行环境是***window***对象，因此所有全局变量和函数都是作为window对象的属性和方法创建的。某个执行环境中所有代码执行完毕后，该环境会被销毁，保存在其中的所有变量和函数定义也随之销毁（全局执行环境直到应用程序退出——例如关闭网页或浏览器——时才会销毁）。

**每个函数都有自己的执行环境**。当执行流进入一个函数时，函数的环境就会被推入一个环境栈；而在函数执行之后，栈将其环境弹出。将控制权返回给之前的执行环境。

当代码在一个环境中执行时，会创建变量对象的一个***作用域链***。作用域链前端时当前执行的代码所在环境的变量对象。活动对象在最开始时只包含一个变量，即```arguments```对象（这个对象在全局环境中不存在）。作用域链中下一个变量对象来自外部环境，这样一直延续到全局执行环境。**全局执行环境的变量对象始终都是作用域链中的最后一个对象**。标识符解析是沿着作用域链一级一级搜索标识符的过程。搜素过程始终从作用域链前端开始，然后逐级向后回溯，直至找到标识符为止（如果找不到则报错）。

```js
var color = 'blue';

function changeColor() {
    var anotherColor = 'red';
    
    function swapColors() {
        var tempColor = anotherColor;
        anotherColor = color;
        color = tempColor;
        
        // 这里可以访问color、anotherColor和tempColor
    }
    
    // 这里可以访问color和anotherColor，但不能访问tempColor
    swapColors();
}

// 这里只能访问color
changeColor();
```

![](http://petfhoepw.bkt.clouddn.com/markdown_2018-09-29_10-39-06.png)

如图所示，每个矩形表示特定的执行环境。对应```swapColors()```来说，其作用域链包含3个对象：```swapColors()```变量对象、```changeColor()```变量对象、全局变量对象。

#### 4.2.1 延长作用域链

虽然作用域链的类型只有两种——***全局和局部***，但有其他办法来延长作用域链。

当执行流进入下列任何一个语句时，作用域链会得到加长：

* ```try-catch```语句的**```catch```块**
* ```with```语句

这两个语句都会**在作用域链的前端添加一个变量对象**

* ```with```会将指定的对象添加到作用域链中
* ```catch```会创建一个新的变量对象，其中包含的是被抛出的错误对象

##### with详解

with的作用就是引用对象，并对该对象上的属性进行操作，但是有几个问题需要注意：

1. 在```with(obj)```代码块中，先查找是不是```obj```对象的属性，如果不是继续查找是不是局部变量（```代码v1```中的```console.log(obj)```的结果中的```url```）
2. 在```with(obj)```代码块中，使用```var```来声明变量（url），**该变量是声明在```buildUrl()```执行环境中的**，（也可以理解为js中没有块级作用域）。这就是```代码v1```和```代码v2```的区别
3. 在```with(obj)```代码块中，只能改变```obj```对象中已有属性的值（比如url、href、qs），但是不能给```obj```对象添加新的属性（```代码v1```中的url```代码v3```）

```js
// 代码v1

var obj = {
    url: '123',
    href: 'http://localhost:8080',
    qs: '?debug=fasle'
}

function buildUrl() {
    var qs = "?debug=true";
    
    with(obj) {
        var url = href + qs;	// 此时修改的是obj中的url
    }
    // 上面等价于
    /*
     * 	var url;
     * 	with (obj) {
     * 		url = href + qs;	
     * 	}
     */
    
    return url;
}

console.log(buildUrl());		// undefined
console.log(obj);				
/*
{ 
	url: 'http://localhost:8080?debug=false',
	href: 'http://localhost:8080',
	qs: '?debug=fasle'
}
*/
```

```js
// 代码v2

var obj = {
    url: '123',
    href: 'http://localhost:8080'
}

function buildUrl() {
    var qs = "?debug=true";
    with(obj) {
        url = href + qs;		// 修改的是obj中的url
    } 
    return url;
}

console.log(buildUrl());		// ReferenceError: url is not defined
console.log(obj);
```

```js
// 代码v2

var obj = {
    href: 'http://localhost:8080'
}

function buildUrl() {
    var qs = "?debug=true";
    with(obj) {
        url = href + qs;	// 此时url为全局变量
        // 如果用var声明url，则最后一个console.log(url)会报错
    }
    return url;
}

console.log(buildUrl());	// http://localhost:8080?debug=true
console.log(obj);			// { href: 'http://localhost:8080', qs: '?debug=true' }
console.log(url);			// http://localhost:8080?debug=true
```

##### with的缺陷

从前面的说明中可以看出with的作用就是简化代码，但是由于其将函数执行时处于作用域链最前端的对象挤下一层，这样会导致查询的性能变慢；同时使用with语句还会带来语义不明的问题，如下面代码所示：

```js
function fn(a,b){
    with(a){
        console.log(b);
    }
}
```

调用fn函数的时候，输出来的b可能是a对象里面的b属性，也可能是传入的参数b，还有可能是undefined，造成代码的难以理解；同时with语句会造成潜在的bug，如下所示：

```js
function fn(obj){
    with(obj){
        a = 1;
    }
};
var obj1 = { a:2 };
var obj2 = { b:3 };
fn(obj1);
fn(obj2);
console.log(obj1.a);//1
console.log(obj2.a);//undefined
console.log(a);//1
```

在fn函数中，由于传入的obj2没有a属性，所以会**将a属性自动升级为全局变量**（这与不加var字符串就直接声明变量的原理是一样的，即变量提升），而传入的obj1中有a属性，就会修改obj1中的a属性。

> 所以，***尽量减少with的使用***。

### 4.3 垃圾收集

JavaScript具有自动垃圾收集机制。垃圾收集器必须跟踪哪个变量有用哪个变量没用，对于不再由于的变量打上标记，以备将来回收其占用的内存。有两种标识无用变量的策略

#### 4.3.1 标记清除

当变量进入环境（例如，在函数中声明一个变量）时，就将这个变量标记为“进入环境”，当变量离开环境时，则将其标记为“离开环境”。

垃圾收集器在运行的时候会给存储在内存中的所有变量都加上标记。然后，它会去掉环境中的变量已经被环境中的变量引用的变量的标记。之后再被标记的变量将被视为准备删除的变量，最后垃圾收集器完成内存清除工作。

大部分浏览器的JavaScript实现使用的都是标记清除式的垃圾收集策略

#### 4.3.2 引用记数

跟踪记录每个值被引用的次数。引用记数的策略会有**循环引用的问题**。

#### 4.3.4 管理内存

由于分配给web浏览器的可用内存数量通常少于分配给桌面应用程序的内存数量，因为确保占用最少的内存可以让页面获得更好的性能。

优化内存占用的最佳方式就是：**为执行中的代码只保存必要的数据，一旦数据不再有用，最好通过将其值设置为null来释放其引用**。这一做法适用于大多数全局变量和全局对象的属性。

## 第5章 引用类型

### 5.2 Array类型

#### 5.2.1 检测数组

可以使用```instanceof```来检测数组

```js
if (value instanceof Array) {
    // ..
}
```

在ECMAScript5中新增了一个检测值是不是数组的方法

```js
if (Array.isArray(value)) {
    // ..
}
```

#### 5.2.2 转换方法

除了```toString()```、```toLocaleString()```、```valueOf()```外，还有一个```join()```

```js
var colors = ['red', 'green', 'blue'];
console.log(colors.join('|'));		// red|green|blue
```

#### 5.2.3 栈方法

* ```push(value)```：向数组末尾添加元素，返回修改后数组的长度
* ```pop()```：从数组末尾移除一个元素并返回移除的项

#### 5.2.4 队列方法

* ```push(value)```
* ```shift()```：移除数组中的第一个项并返回该项

##### 反向队列

* ```unshift()```：在数组最前端添加任意个项，返回修改后数组的长度
* ```pop()```

#### 5.2.5 重排序方法

* ```sort()```
* ```reverse()```

```sort()```可以接收一个比较函数作为参数

```js
var values = [5, 1, 15, 10];
values.sort();
console.log(values);			// 1, 10, 15, 5
// 因为sort()是按照字符串来比较的，

values.sort(function(value1, value2) {
    if (value1 < value2) {
        return -1;
    } else if(value1 > value2) {
        return 1;
    } else {
        return 0;
    }
});
console.log(values);			// 1, 5, 10, 15
```

#### 5.2.6 操作方法

* ```arr.concat()```：基于当前数组中所有项创建一个新数组，然后将接收到的所有参数添加到这个副本的末尾，最后返回新购进的数组。**参数中如果有数组，则把数组中的每一项都添加到结果数组末尾，其他值会被见到地添加到结果数组末尾**。

  ```js
  var colors = ['red', 'green'];
  var colors2 = colors.concat('yellow', ['black', 'brown']);
  console.log(colors);		// 不变
  console.log(colors2);		// red, green, yellow, black, brown
  ```

* ```arr.alice()```：截取数组，也是创建副本，原数组不变

  ```js
  colors.slice(start);			// [start, colors.length)
  colors.slice(start, end);		// [start, end)
  colors.slice(-start, -end);		// [colors.length - start, colors.length - end)
  ```

* ```arr.splice()```：返回一个数组，包含从原始数组中删除的项（如果没有删除则返回空数组）

  * 删除：```splice(1, 2)```，从index=1开始从数组中删除2项
  * 插入：```splice(2, 0, v1, v2, ..)```，在index=2处插入元素v1，v2等等
  * 替换：```splice(3, 1, v1, v2, ..)，在index-3处删去一个元素，再添加上多个元素

#### 5.2.7 位置方法

* ```indexOf('red', 4)```：从index=4开始查找'red'的索引，第二个参数可选
* ```lastIndexOf('red', 4)```：从index=4开始往前查，参数同上

#### 5.2.8 迭代方法

共有5个迭代方法，每个方法接收两个参数：**要在每项上运行的函数**、**（可选）运行该函数的作用域对象**。传入这些方法中的函数会接收三个参数：数组项的值、该项在数组中的位置、数组对象本身。

> 这些方法都不会修改数组中包含的值

* ```every()```：数组中的每一项运行给定函数都返回true，则返回true
* ```filter()```：过滤掉那些运行给定函数返回false的数组项，返回过滤后的数组
* ```forEach()```：对数组中每一项运行给定函数，没有返回值
* ```map()```：返回数组中每一项运行给定函数的返回值所组成的数组
* ```some()```：数组中任意一项运行给定函数返回true，则返回true

#### 5.2.9 归并方法

* ```reduce()```：从前到后迭代数组的所有项，然后构建一个最终返回的值
* ```reduceRight()```：从后到前迭代数组的所有项，然后构建一个返回的值

这两个方法接收两个参数：**要在每项上运行的函数**、**（可选）为归并基础的初始值**。第一个参数（每项上运行的函数）接收4个参数：前一个值、当前值、项的索引和数组对象。***这个函数返回的任何值都会作为第一个参数自动传给下一项***。第一次迭代发生在数组的第二项上。

### 5.4 RegExp类型

使用下面的语法可以创建一个正则表达式

```js
var expression1 = /pattern/flags;

// or

var expression2 = new RegExp(pattern, flags);
```

* pattern部分可以是任何正则表达式
* 每个正则表达式都可带有一或多个flags，用以标明正则表达式的行为
  * ```g```：表示全局模式（global），模式被**应用于所有字符串，而非在发现第一个匹配项时立即停止**
  * ```i```：不区分大小写模式
  * ```m```：多行模式，在到达一行文本末尾时还会继续查找下一行中是否存在与模式匹配的项

ECMAScript 5种规定，使用正则表达式字面量和直接调用RegExp构造函数一样，每次都创建新的RegExp实例。

#### 5.4.1 RegExp实例属性

* global：表示是否设置了g标志
* ignoreCase：表示是否设置了i标志
* lastIndex：开始搜索下一个匹配项的字符位置，从0开始
* multiline：表示是否设置了m标志
* source：正则表达式的字符串表示，按照字面量形式而非传入构造函数中的字符串模式返回

#### 5.4.2 RegExp实例方法

* ```exec()```：接受一个参数（要应用模式的字符串），返回包含第一个匹配项信息的数组；或者没有匹配项时返回null。数组中，第一项是与整个模式匹配的字符串，其他项是与模式中的捕获组匹配的字符串。**数组的长度等于模式中捕获组的个数+1**。

  ```js
  var text = 'mom and dad and babymom';
  var pattern = /mom( and dad( and baby)?)?/gi;
  
  var matches = pattern.exec(text);
  console.log(matches[0]);		// 'mom and dad and baby'
  console.log(matches[1]);		// ' and dad and baby'
  console.log(matches[2]);		// ' and baby'
  
  matches = pattern.exec(text);
  console.log(matches[0]);		// 'mom'
  console.log(matches[1]);		// undefined
  console.log(matches[2]);		// undefined
  ```

  > 对于```exec()```方法而言，即使在模式中设置了全局标志(g)，它每次也只会**返回一个匹配项**。在不设置全局标志的情况下，在同一个字符串上多次调用```exec()```将始终返回第一个匹配项的信息。而在设置全局标志的情况下，每次调用```exec()```都会在字符串中继续查找新匹配项。

* ```test()```：接受一个字符串参数，在模式与该参数匹配的情况下返回true；否则返回false

### 5.5 Function类型

每个函数都是```Function```类型的实例，与其他引用类型一样具有属性和方法。**函数名是指向函数对象的指针**。因此函数名与包含对象指针的其他变量没有不同，即一个函数可能会有多个名字。

```js
function sum(num1, num2) {
    return num1 + num2;
}
console.log(sum(1, 2));

var anotherSum = sum;
console.log(another(1, 2));
```

#### 5.5.1 没有重载

> 因为函数名是指针，所以ECMAScript中没有函数重载的概念

#### 5.5.2 函数声明与函数表达式

解析器在向执行环境中加载数据时，对函数声明和函数表达式并非一视同仁。**解析器会先读取函数声明，并使其在执行任何代码之前可用（可以访问）；至于函数表达式，则必须等到解析器执行到它所在的代码行才会真正被解释执行**。

#### 5.5.4 函数内部属性

在函数内部，有两个特殊的**对象**：```arguments```和```this```

##### arguments

```arguments```还有一个```callee```的属性，指向拥有这个```arguments```对象的函数

```js
function factorial (num) {
    if (num <= 1) {
        return 1;
    } else {
      return num * arguments.callee(num - 1);	// num * factorial(num - 1)  
    }
}
```

##### this

```this```引用的是函数据以执行的环境对象。

##### caller

ECMAScript 5规范化了另一个**函数对象的属性**：```caller```。这个属性中保存着调用这个函数的函数的引用，**如果是在全局作用域中调用这个函数，它的值为null**。

```js
function outer () {
    inner();
}

function inner() {
    console.log(outer.caller);			// null
    console.log(arguments.callee.caller);			// f outer() { inner(); }
}

outer();	
```

#### 5.5.5 函数属性和方法

##### 属性

函数是对象，因此函数也有属性和方法，函数包含两个属性：```length```和```prototype```。```length```表示函数希望接收的命名参数的个数。

```js
function sum (num1, num2) {
	return num1 + num2;
}
console.log(sum.length);		// 2
```

##### 方法

每个函数包含两个非继承而来的方法：```apply()```和```call()```。这两个方法用途都是**在特定的作用域中调用函数**（实际上等于设置函数体内```this```对象的值）。

###### apply

```js
function sum (num1, num2) {
    return num1 + num2;
}
function applySum (num1, num2) {
    return sum.apply(this, arguments);		// 或者sum.apply(this, [num1, num2]);
}

console.log(applySum(1, 2));		// 此时applySum中的this指向window
```

* 第一个参数：运行函数的作用域
* 第二个参数：Array的实例（参数数组），或者是```arguments```对象

###### call

```call()```和```apply()```作用相同，区别仅在于接收参数的方式不同

```js
function callSum (num1, num2) {
    return sum.call(this, num1, num2);
}
```

###### call和apply的作用

> 能够扩充函数赖以运行的作用域

```js
window.color = 'red';
var o = { color: 'yellow' };

function sayColor() {
    console.log(this.color);
}

sayColor();						// red
sayColor.call(this);			// red
sayColor.call(window);			// red
sayColor.apply(o);				// yellow
```

使用```call()```（或```apply()```）来扩充作用域的最大好处，就是**对象不需要与方法有任何耦合关系**。

### 5.6 基本包装类型

有3个特殊的引用类型：Boolean、Number和String。

```js
var s1 = "some text";
var s2 = s1.substring(2);
```

字符串是基本类型值，所以从逻辑上不该有方法，而实际上，在读取模式中（在line 2中```var s2 = s1.substring(2)```）访问字符串时，后台会完成下列处理：

1. 创建String类型的一个实例
2. 在实例上调用指定的方法
3. 销毁这个实例

也就是相当于：

```js
var s1 = "some text";
var ss = new String(s1);
var s2 = ss.substring(2);
ss = null;
```

这三个步骤也适用于Boolean和Number。

使用new创建的引用类型的实例**在执行流离开当前作用域之前都一直保存在内存中**；自动创建的基本包装类的对象则**只存在于一行代码的执行瞬间，然后立即被销毁**。

> 因此我们不能在运行时为基本类型值添加属性和方法

```js
var s1 = "some text";
s1.color = "red";
console.log(s1.color);			// undefined
```

第二行创建的String对象在执行第三行代码时已经被销毁了，第三行代码又创建自己的String对象，该对象没有color属性。

> Object构造函数也会像工厂方法一样，根据传入值的类型返回相应基本包装类型的实例

```js
var obj = new Object("some text");
console.log(obj instanceof String);			// true
```

使用```new```调用基本包装类型的构造函数、与直接调用同名的转型函数是不一样的

```js
var value = "25";
var number = Number(value);
console.log(typeof number);			// "number"

var obj = new Number(value);
console.log(typeof obj);			// "object"
```

#### 5.6.1 Boolean类型

建议永远不要使用Boolean对象，因为：

```js
var falseObj = new Boolean(false);
console.log(falseObj && true);			// true
```

#### 5.6.2 Number类型

也不建议使用Number对象

#### 5.6.3 String类型

##### 1. 属性

```length```属性：字符串中的字符数量

##### 2. 字符方法

```charAt()```：以单字符字符串的形式返回给定位置的那个字符

```charCodeAt()```：得到指定位置的字符的字符编码

```js
var str = "hello world";
console.log(str.charAt(1));			// "e"
console.log(str.charCodeAt(1));		// "101"
```

也可以使用类似数组的方式来访问特定字符

```js
console.log(str[1]);				// "e"
```

##### 3. 字符串操作方法

> 以下方法都不会改变原字符串

###### concat()

将多个字符串拼接起来，返回拼接得到的新字符串

###### slice()、substring()、substr()

返回子字符串，只是参数意义不一样。

* 参数都为正值时

  ```js
  var str = "hello world";
  console.log(str.slice(3));					// "lo world"
  console.log(str.substring(3));				// "lo world"
  console.log(str.substr(3));					// "lo world"
  
  console.log(str.slice(3, 7));				// "lo w"		[3, 7)
  console.log(str.substring(3, 7));			// "lo w"		[3, 7)
  console.log(str.substr(3, 7));				// "lo worl"	[3, 3 + 7)
  ```

* 参数为负数时

  情况太多

##### 4. 字符串位置方法

```indexOf()```：从前往后搜索子字符串的位置

```lastIndexOf()```：从后往前搜索子字符串的位置

##### 5. trim()

创建字符串副本，删除前置及后缀的所有空格

##### 6. 大小写转换方法

```toLowerCase()```、```toUpperCase()```、```toLocaleLowerCase()```、```toLocaleUpperCase()```

##### 7. 字符串的模式匹配方法

###### match()

本质与调用RegExp的```exec()```方法相同。```match()```只接受一个参数，要么是正则表达式，要么是一个RegExp对象。

```js
var text = 'mom and dad and babymom';
var pattern = /mom( and dad( and baby)?)?/gi;

var matches = text.match(pattern);
```

matches和5.4.2节得到的结果一样。

###### search()

唯一参数与```match()```参数相同。返回字符串中第一个匹配项的索引；如果没找到则返回-1

###### replace()

第一个参数可以是一个RegExp对象或者一个字符串（不会被转成正则表达式）；第二个参数是一个字符串或者一个函数。如果参数一是字符串，则只会替换第一个字符串，要向替换所有子字符串，参数一应为正则表达式，而且要指定全局标志。

> 如果第二个参数是字符串，还可以使用一些特殊的字符序列，将正则表达式操作得到的值插入到结果字符串中

```js
var text = "cat, bat, sat, fat";
var result = text.replace(/(.at)/g, "word ($1)");
console.log(result);		// word (cat), word (bat), word (sat), word (fat)
```

$n：匹配第n个捕获组的字符串

###### split()

参数一是RegExp对象或者字符串，第二个参数可选，指定返回数组的大小。

### 5.7 单体内置对象

#### 5.7.2 Math对象

Math对象的```max()```和```min()```不接收数组作为参数，只能把要比较的数一个个作为参数：```Math.max(1, 2, 3, 4, 5)```。因此，如果参数是数组，可以这样调用：

```js
var values = [1, 2, 3, 4, 5];
var max = Math.max.apply(Math, values);
```

## 第6章 面向对象的程序设计

可以把ECMAScript中的对象想象成散列表。

### 6.1 理解对象

#### 6.1.1 属性类型

ECMA-262第5版在定义只有内部才用的特性时，描述了属性的各种特征（例如[[Enumerable]]）

ECMAScript中有两种属性：**数据属性**和**访问器属性**

##### 1. 数据属性

数据属性包含一个数据值的位置。在这个位置可以读取和写入值。数据属性有4个描述其行为的特性

* ```[[configurable]]```：表示能否通过```delete```删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为访问器属性
* ```[[Enumerable]]```：表示能否通过```for-in```循环返回属性
* ```[[Writable]]```：表示能否修改属性的值。
* ```[[Value]]```：包含这个属性的数据值。读取属性值的时候，从这个位置读；写入属性值的时候，把新值保存在这个位置。这个特性的默认值为```undefined```。

```js
var person = {
    name: 'Harry';
}
```

像这样直接在对象上定义的属性，它们的```[[configurable]]```、```[[Enumerable]]```、```[[Writable]]```特性都被设置为**true**，而```[[Value]]```特性被设置为指定的值（'Harry'）。

要修改属性默认的特性，必须使用```Object.defineProperty()```：

```js
var person = {};
Object.defineProperty(person, "name", {
    writable: false,
    value: 'Harry'
});

console.log(person.name);			// Harry
person.name = 'Hermione';
console.log(person.name);			// Harry

delete person.name;
console.log(person.name);			// Harry
```

**一旦把```[[configurable]]```设置为false后，就不能再修改除```[[writable]]```之外的其他特性**。

在调用```Object.defineProperty()```创建一个新的属性时，如果不指定，```[[configurable]]```、```[[Enumerable]]```、```[[Writable]]```特性的默认值都是**false**；如果调用```Object.defineProperty()```修改已定义的属性的特性，则无此限制。

##### 2. 访问器属性

访问器属性不包含数据值；它们包含一对```getter()```和```setter()```（不过这两个函数都不是必须的）。在读取访问器属性时，会调用```getter()```函数；在写入访问器属性时，会调用```setter()```并传入新值。访问器属性有4个特性：

* ```[[configurable]]```
* ```[[Enumerable]]```
* ```[[Get]]```：读取属性时调用的函数，默认值为undefined
* ```[[Set]]```：写入属性时调用的函数，默认值为undefined

**访问器属性不能直接定义，必须使用```Object.defineProperty()```来定义**：

```js
var book = {
    _year: 2004,
    edition: 1
};

Object.defineProperty(book, "year", {
    get: function () {
        return this._year;
    },
    set: function (newValue) {
        if (newValue > 2004) {
            this._year = newValue;
            this.edition += newValue - 2004;
        }
    }
});

book.year = 2005;
console.log(book.edition);		// 2
```

```_year```前面的下划线是一种常用的记号，表示只能通过对象方法访问的属性。

不一定非要同时指定```getter()```和```setter()```，只指定```getter()```意味着属性是不能写的；只指定```setter()```的函数也不能读。

#### 6.1.2 定义多个属性

利用```Object.defineProperties()```可以一次定义多个属性：

```js
var book = {};

Object.defineProperties(book, {
    _year: {
        writable: true,
        value: 2004
    },
    edition: {
        writable: true,
        value: 1
    },
    year: {
        get: function() {
            // ..
        },
        set: function(newValue) {
            // ..
        }
    }
});
```

#### 6.1.3 读取属性的特性

利用```Object.getOwnPropertyDescriptor()```可以取得给定属性的描述符：

```js
var desc = Object.getOwnPropertyDescriptor(person, 'name');
console.log(desc);			
// {value: "Harry", writable: false, enumerable: false, configurable: false}
```

该方法只能用于实例属性，如果要取得原型属性的描述符，则必须直接在原型对象上调用：

```js
Object.getOwnPropertyDescriptor(Person.prototype, 'name');
```

### 6.2 创建对象

使用Object构造函数或对象字面量创建对象有一个明显缺点：**使用同一个接口创建很多对象，会产生大量的重复代码**。可以使用工厂模式的一种变体来解决这个问题

#### 6.2.1 工厂模式

```js
function createPerson(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.sayName = function () {
        console.log(this.name);
    };
    return o;
}

var person = createPerson('Harry', 16, 'Magician');
```

工厂模式虽然解决了多个相似对象的问题，但**没有解决对象识别的问题**（即怎样知道一个对象的类型）。构造函数模式可以解决这个问题

#### 6.2.2 构造函数模式

```js
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function () {
        console.log(this.name);
    }
}

var person = new Person('Harry', 16, 'Magician');
```

> 安装惯例，构造函数始终都应该以一个大写字母开头，而非构造函数应该以一个小写字母开头。

使用```new```来创建Person的新实例时，实际经历以下4个步骤：

* 创建一个新对象
* 将构造函数的作用域赋给新对象（相当于调用```apply()```方法，把this指向这个新对象）
* 执行构造函数中的代码
* 返回新对象

person对象有一个```constructor```属性，该属性指向Person：

```js
console.log(person.constructor === Person);			// true
```

使用构造函数模式创建对象可以标识新对象的类型：

```js
console.log(person instanceof Object);				// true
console.log(person instanceof Person);				// true
```

##### 1. 将构造函数当作函数

**构造函数与其他函数的唯一区别，就在于调用它们的方式不同**。实际上，任何函数只要通过```new```操作符来调，那就可以作为构造函数；而任何函数如果不通过```new```来调，那就和普通函数不会有什么两样：

```js
// 当作构造函数调用
var person = new Person('Harry', 16, 'Magician');
person.sayName();		// Harry

// 当作普通函数调用
Person('Hermione', 16, 'Magician');
window.sayName();		// Hermione

// 在另一个对象的作用域中调用
var o = new Object();
Person.call(o, 'Weasley', 16, 'Magician' )
o.sayName();			// Weasley
```

##### 2. 构造函数的问题

构造函数方法的主要问题就是**每个方法都要在每个实例上重新创建一遍**。

```js
var harry = new Person('Harry', 16, 'Magician');
var hermione = new Person('Hermione', 16, 'Magician');
console.log(harry.sayName === hermione.sayName);			// false
```

因为有```this```对象的存在，不用在执行代码前就把函数绑定到特定对象上面。所以可以通过下面方式解决这个问题：

```js
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = sayName;
}

function sayName() {
    console.log(this.name);
}
```

但这种方式也有问题，**在全局作用域中定义的函数实际上只能被某个对象调用，如果对象需要定义很多方法，那么就要定义很多个全局函数，于是我们这个自定义的引用类型就丝毫没有封装性可言了**。可以用原型模式来解决上述问题

#### 6.2.3 原型模式

我们创建的每个函数都有一个```prototype```属性，指向一个原型对象。通过```new```来创建的这个函数的每个实例都共享原型对象所包含的属性和方法。

##### 1. 理解原型对象

只要创建一个新函数，就会根据一组特定的规则为该函数创建一个```prototype```属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个```constructor```属性，这个属性是一个是指向这个新函数的指针。

```js
Person.prototype.constructor === Person
```

创建了自定义的构造函数之后，其原型对象默认只会有```constructor```属性，至于其他方法，则都要从Object继承而来。当调用构造函数创建一个新实例后，该实例的内部将包含一个指针，指向构造函数的原型对象。ECMAScript-262第5版中管这个指针叫```[[Prototype]]```，虽然在脚本中没有标准的方式访问```[[Prototype]]```，但Firefox、Safari和Chrome在每个对象上都支持一个属性```__proto__```。

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-11_13-48-07.png)

如图可知，**Person的每个实例person1和person2与构造函数没有直接的关系**。

可以通过```isPrototypeOf()```来判断是否是原型：

```js
console.log(Person.prototype.isPrototypeOf(person1));		// true
```

还可以通过```Object.getPrototypeOf()```来得到一个对象的原型

```js
console.log(Object.getPrototypeOf(person1) === Person.prototype);		// true
```

**当读取某个对象的某个属性时，首先会在该对象中搜索，如果找到了给定名字的属性，则返回该属性的值；如果没有找到，则继续搜索指针所指向的原型对象**。

不能通过对象实例重写原型中的值，在实例中添加了一个属性（与实例原型中的一个属性同名），会在实例中创建该属性，并屏蔽原型中的这个属性

```js
function Person() {}
Person.prototype.name = 'Harry';

var person1 = new Person();
var person2 = new Person();
person1.name = 'Hermione';

console.log(person1.name);			// Hermione
console.log(person2.name);			// Harry
```

使用```hasOwnProperty()```可以检测一个属性是存在于实例中，还是存在于原型中

```js
console.log(person1.hasOwnProperty('name'));		// true
console.log(person2.hasOwnProperty('name'));		// false
```

##### 2. 原型与in操作符

###### a. 单独使用in

在单独使用时，**只要通过对象能够访问给定属性就返回true**，无论该属性在实例中还是原型中。

```js
function Person() {}
Person.prototype.name = 'Harry';
var person = new Person();
person.age = 16;

console.log('age' in person);					// true
console.log('name' in person);					// true
```

###### b. for-in

在使用for-in循环时，返回的是**所有能够通过对象访问的、可枚举的属性**，包括实例属性和原型属性。

###### c. 可枚举的实例属性

可以通过```Object.keys()```得到对象上所有可枚举的实例属性

```js
function Person() {}

Person.prototype.name = 'Harry';
Person.prototype.age = 16;
Person.prototype.sayName = function () {
    console.log(this.name);
}

console.log(Object.keys(Person.prototype));		// ["name", "age", "sayName"]

var person = new Person();
person.name = 'Hermione';
console.log(Object.keys(person));		// ["name"]
```

可以通过```Object.getOwnPropertyNames()```来得到所有实例属性，包括不可枚举的。

```js
console.log(Object.getOwnPropertyNames(Person.prototype));
// ["constructor", "name", "age", "sayName"]
```

如上例所示，```constructor```属性不可枚举

##### 3. 更简单的原型语法

```js
Person.prototype = {
    name: 'Harry'
}
```

这种方式，**原型对象的```constructor```属性不再指向Person了**。这里的语法，本质上完全重写了默认的原型对象（因为每创建一个函数，就会同时创建它的原型对象，这个原型对象有```constructor```属性），此时原型对象中就没有```constructor```属性了（此时原型对象就是个普通对象），但是在原型对象的继承链上还有```constructor```属性。所以**如果用这种简单写法，定义原型对象一定要在创建对象之前（如下所示）**

![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-11_14-50-16.png)

此时仍旧可以使用```instanceof```来判断类型

```js
var friend = new Person();
console.log(friend instanceof Person);			// true
```

##### 4. 原型的动态性

* 可以随时修改原型对象的属性，即使先创建实例后修改原型也一样

  ```js
  var friend = new Person();
  Person.prototype.sayHi = function() { console.log('Hi') };
  
  friend.sayHi();		// Hi
  ```

* 但是把原型重新赋值和创建实例的顺序就有关系

  ```js
  function Person() {}
  
  var friend = new Person();
  
  Person.prototype = {
      constructor: Person,
      name: 'Nicholas',
      age: 29,
      job: 'Software Engineer',
      sayName: function () {
          console.log(this.name);
      }
  }
  
  friend.sayName();			// error
  ```

  ![](http://sherry-pic.oss-cn-hangzhou.aliyuncs.com/markdown_2018-10-11_15-06-31.png)

  重写原型对象切断了现有原型与任何之前已经存在的对象实例之间的联系。这些对象实例引用的仍然是最初的原型。

##### 5. 原生对象的原型

**原生的引用类型也是采用原型模式创建的**。所有原生引用类型（Object、Array、String等）都在其构造函数的原型上定义了方法。可以修改原生对象的原型，但不推荐这么做。

##### 6. 原型对象的问题

* 所有的实例在默认情况下都将取得相同的属性值。

* 原型的引用类型的属性会出现问题

  ```js
  function Person() {}
  
  Person.prototype = {
      friends: ['Harry', 'Hermione', 'Weasley']
  };
  
  var person1 = new Person();
  var person2 = new Person();
  
  person1.friends.push('Voldemort');
  
  console.log(person1.firends);	// ["Harry", "Hermione", "Weasley", "Voldemort"]
  console.log(person2.firends);	// ["Harry", "Hermione", "Weasley", "Voldemort"]
  ```

#### 6.2.4 组合使用构造函数模式和原型模式

创建自定义类型的最常见方式就是**组合使用构造函数模式与原型模式**。构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。

```js
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = ['Harry', 'Hermione', 'Weasley'];
}

Person.prototype = {
    constructor: Person,
    sayName: function () {
        console.log(this.name);
    }
}
```

#### 6.2.5 动态原型模式

```js
function Person(name, age, job) {
    // 属性
    this.name = name;
    this.age = age;
    this.job = job;
    
    // 方法
    if (typeof this.sayName != "function") {
        Person.prototype.sayName = function () {
            console.log(this.name);
        }
    }
}
```

### 6.3 继承

ECMAScript中只支持实现继承，不支持接口继承。

#### 6.3.2 原型链

实现原型链有一种基本模式：

```js
function Parent () {
    this.parentValue = true;
}
Parent.prototype.getParentValue = function () {
    return this.parentValue;
}

function Child () {
    this.childValue = false;
}
// 继承了Parent
Child.prototype = new Parent();
Child.prototype.getChildValue = function () {
    return this.childValue;
}

var instance = new Child();
console.log(instance.getParentValue());			// true

console.log(instance.__proto__.__proto__ == Parent.prototype);			// true
```

实现继承的本质是**重新原型对象，将子的原型对象指向父的实例**。

##### 1.别忘记默认的原型

> ***所有函数的默认原型都是Object的实例***

```js
function Course () {}
var course = new Course();

console.log(course.__proto__.__proto__ === Object.prototype);		// true
```

##### 2. 确定原型和实例的关系

###### a. instanceof

只要用这个操作符来测试实例与原型链中出现过的构造函数，结果都会返回true

```js
console.log(instance instanceof Object);		// true
console.log(instance instanceof Parent);		// true
console.log(instance instanceof Child);		// true
```

###### b. isPrototypeOf

只要是原型链中出现过的原型，都会返回true

```js
console.log(Object.prototype.isPrototypeOf(instance));		// true
console.log(Parent.prototype.isPrototypeOf(instance));		// true
console.log(Child.prototype.isPrototypeOf(instance));		// true
```

##### 3. 谨慎地定义方法

> 给原型添加方法的代码一定要放在替换原型的语句之后，如6.3.2节中的line 12-15

##### 4. 原型链的问题

###### 问题一

引用类型值的原型属性会被所有实例共享。而通过原型来实现继承时，原型实际上变成另一个类型的实例，于是父类型中的实例属性也就变成了子类型中的原型属性：

```js
function Parent() {
    this.firends = ['蓉儿', '盈盈', '赵敏'];
}
function Child() {}

Child.prototype = new Parent();

var c1 = new Child();
var c2 = new Child();

c1.firends.push('水笙');
console.log(c2.friends);		// ["蓉儿", "盈盈", "赵敏", "水笙"]
```

###### 问题二

在创建子类型的实例时，不能向父类型的构造函数中传递参数。

**实践中很少单独使用原型链**

#### 6.3.2 借用构造函数

在子类型构造函数的内部调用超类型构造函数。

```js
function Parent() {
    this.firends = ['蓉儿', '盈盈', '赵敏'];
    this.name = name;
}

function Child() {
    // 继承了Parent
    Parent.call(this, 'vermouth');
}

var c1 = new Child();
var c2 = new Child();

c1.firends.push('水笙');
console.log(c1.friends);		// ["蓉儿", "盈盈", "赵敏", "水笙"]
console.log(c2.friends);		// ["蓉儿", "盈盈", "赵敏"]
```

##### 1. 传递参数

相对于原型链，借用构造函数可以在子构造函数中向父构造函数传递参数

##### 2. 借用构造函数的问题

* 方法都在构造函数中定义，6.2.5节构造函数模式的问题也会在这里出现
* 在父类型的原型中定义的方法，对子类型也不可见，结果所有类型都只能使用构造函数模式

**实践中很少单独使用借用构造函数**

#### 6.3.3. 组合继承

将原型链和借用构造函数组合起来。**使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性实例属性的继承**。

```js
function Parent() {
    this.firends = ['蓉儿', '盈盈', '赵敏'];
    this.name = name;
}
Parent.prototype.sayName = function() {
    console.log(this.name);
}

function Child() {
    // 继承了Parent
    Parent.call(this, 'vermouth');
    this.age = age;
}

Child.prototype = new Parent();
Child.prototype.sayAge = function() {
    console.log(this.age);
}
```

#### 6.3.4 原型式继承

```js
function object(o) {
    function F() {}
    F.prototype = o;
    return new F();
}
```

从本质上讲，```object()```对传入其中的对象执行了一次浅复制。

```js
var p1 = {
    name: 'vermouth',
    friends: ['蓉儿', '盈盈', '赵敏']
};

var p2 = object(p1);
p2.name = 'sherry';
p2.friends.push('夏青青');

var p3 = object(p1);
p3.name = 'conan';
p3.friends.push('水笙');

console.log(p1.friends);		// ["蓉儿", "盈盈", "赵敏", "夏青青", "水笙"]
console.log(p1.name);			// vermouth
```

ECMAScript5种新增了```Object.create()```规范化了原型式继承

```js
var p1 = {
    name: 'vermouth',
    friends: ['蓉儿', '盈盈', '赵敏']
};

var p2 = Object.create(p1, {
    name: {
        value: "sherry"
    }
});
```

第二个参数是可选的。





















