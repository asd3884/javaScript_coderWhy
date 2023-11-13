# JavaScript高级语法

## Javascript执行原理

前端开发最主要需要掌握的三个知识点：Html、css、javaScript



html:简单易学，掌握常用的标签即可

css:属性规则较多，多做练习和项目

javaScript:上手容易，但是，精通很难，学会他需要几分钟，掌握它需要很多年

### 浏览器工作原理

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\浏览器工作原理.webp)



##### 浏览器是如何下载js文件的呢？

首先在浏览器中输入域名，然后这个域名将转化服务器的IP地址，这个IP地址将返回一个文件，就是图中的index.html文件，这个时候浏览器开始解析这个网页，解析到link标签就下载css文件，解析到script标签就下载js文件。



### 浏览器的渲染过程

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\浏览器渲染过程.webp)

浏览器将html，css，js文件下载下来后就开始解析，这个时候由浏览器内核中的HtmlParse模块将HTML标签编译成DOMTree，这个时候可以利用js进行DOM操作将HTML元素插入到DOMTree中，而浏览器则是通过浏览器引擎来解析js的，我们在下面的内容将会重点将浏览器V8引擎。

DOMTtree生成以后将会开始解析css文件，通过cssParse模块将css解析为StyleRulesTree，接下来DOMTree和StyleRulesTree将会结合形成RenderTree。

RenderTree生成以后通过Layout引擎根据浏览器大小来生成新的renderTree，然后绘制出来，浏览器开始展示。由于有些html标签定位不同，这个时候浏览器大小不同，元素位置就不同，layout引擎的作用就是重新生成一个适合当前浏览器大小的renderTree。



### V8引擎

#### JS引擎

高级的编程语言都是需要转成最终的机器指令来执行的

事实上我们编写的JavaScript无论你交给浏览器或者Node执行，最后都是需要被CPU执行的

但是CPU只认识自己的指令集，实际上是机器语言，才能被CPU所执行；

所以我们需要JavaScript引擎帮助我们将JavaScript代码翻译成CPU指令来执行



![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\js引擎.webp)



### **V8引擎的架构**

- **Parse**

  模块会将JavaScript代码转换成AST（抽象语法树），这是因为解释器并不直接认识JavaScript代码；

  - 如果函数没有被调用，那么是不会被转换成AST的；
  - Parse的V8官方文档：https://v8.dev/blog/scanner

- **Ignition**

  是一个解释器，会将AST转换成ByteCode（字节码）

  - 同时会收集TurboFan优化所需要的信息（比如函数参数的类型信息，有了类型才能进行真实的运算）；
  - 如果函数只调用一次，Ignition会执行解释执行ByteCode；
  - Ignition的V8官方文档：https://v8.dev/blog/ignition-interpreter

- **TurboFan**

  是一个编译器，可以将字节码编译为CPU可以直接执行的机器码；

  - 如果一个函数被多次调用，那么就会被标记为热点函数，那么就会经过TurboFan转换成优化的机器码，提高代码的执行性能；
  - 但是，机器码实际上也会被还原为ByteCode，这是因为如果后续执行函数的过程中，类型发生了变化（比如sum函数原来执行的是number类型，后来执行变成了string类型），之前优化的机器码并不能正确的处理运算，就会逆向的转换成字节码；
  - TurboFan的V8官方文档：https://v8.dev/blog/turbofan-j

### **V8引擎的解析图（官方）**

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\v8引擎.png)

### **V8执行的细节**

- ***\*那么我们的JavaScript源码是如何被解析（Parse过程）的呢？\****
- Blink将源码交给V8引擎，Stream获取到源码并且进行编码转换；
- Scanner会进行词法分析（lexical analysis），词法分析会将代码转换成tokens；
- 接下来tokens会被转换成AST树，经过Parser和PreParser：
  - Parser就是直接将tokens转成AST树架构；
  - PreParser称之为预解析，为什么需要预解析呢？
    - 这是因为并不是所有的JavaScript代码，在一开始时就会被执行。那么对所有的JavaScript代码进行解析，必然会影响网页的运行效率；
    - 所以V8引擎就实现了**Lazy Parsing（延迟解析）**的方案，它的作用是**将不必要的函数进行预解析**，也就是只解析暂时需要的内容，而对**函数的全量解析**是在**函数被调用时**才会进行；
    - 比如我们在一个函数outer内部定义了另外一个函数inner，那么inner函数就会进行预解析；
- 生成AST树后，会被Ignition转成字节码（bytecode），之后的过程就是代码的执行过程（后续会详细分析）。



#### 浏览器V8引擎是如何编译js的？

首先将js源代码通过parse模块编译成抽象语法树，之后通过Ignition模块将抽象语法树编译成字节码，然后V8引擎将字节码转化成汇编代码，汇编代码转化为对应平台的机器指令运行。

当一个函数多次执行时，Ignition模块将会把这个函数标记成一个hot函数，由TurboFan编译成优化的机器码，这样当下一次执行这个函数，就直接执行这串机器码。当然有一个前提，这个函数传递的参数类型必须一致。 例如：

```
function foo(n1,n2){
console.log(n1+n2)
}

foo(10,20)//此函数转化为机器码
foo(50,30)//直接执行机器码
foo("张三",20)//传递的参数类型不一样，则不会直接执行机器码，而是将机器码转化为字节码再执行

```

由于js是一门动态语言，它本身不会做类型检测，所以需要开发者本身尽量传递类型一致的参数，这样有利于性能的提升，因此Ts对性能提升也有帮助。



## **初始化全局对象**

- js引擎会在执行代码之前，会在堆内存中创建一个全局对象：Global Object（GO）
  - 该对象 所有的作用域（scope）都可以访问；
  - 里面会包含Date、Array、String、Number、setTimeout、setInterval等等；
  - 其中还有一个window属性指向自己；

### ***\*执行上下文栈（调用栈）\****

- js引擎内部有一个**执行上下文栈（Execution Context Stack，简称ECS）**，它是用于执行**代码的调用栈**。

- 那么现在它要执行谁呢？执行的是**全局的代码块**：

  - 全局的代码块为了执行会构建一个 **Global Execution Context（GEC）**；
  - GEC会 被放入到ECS中 执行；

- **GEC被放入到ECS中里面包含两部分内容：**

  - **第一部分：**

    在代码执行前，在parser转成AST的过程中，会将全局定义的变量、函数等加入到GlobalObject中，

    - 但是并不会赋值；
    - 这个过程也称之为变量的作用域提升（hoisting）

  - **第二部分：**在代码执行中，对变量赋值，或者执行其他的函数；

- 示例：

```
var name = "why"
 
console.log(num1)
 
var num1 = 20
var num2 = 30
var result = num1 + num2
 
console.log(result)
 
/**
 * 1.代码被解析, v8引擎内部会帮助我们创建一个对象(GlobalObject -> go)
 * 2.运行代码
 *    2.1. v8为了执行代码, v8引擎内部会有一个执行上下文栈(Execution Context Stack, ECStack)(函数调用栈)
 *    2.2. 因为我们执行的是全局代码, 为了全局代码能够正常的执行, 需要创建 全局执行上下文(Global Execution Context)(全局代码需要被执行时才会创建)
 */
var globalObject = {
  String: "类",
  Date: "类",
  setTimeount: "函数",
  window: globalObject,
  name: undefined,
  num1: undefined,
  num2: undefined,
  result: undefined
}
 

```

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\初始化全局对象.png)



### **遇到函数如何执行？**

- 在执行的过程中**执行到一个函数时**，就会根据**函数体**创建一个**函数执行上下文（Functional Execution Context，**
- **简称FEC）**，并且压入到**EC Stack**中。
- FEC中包含三部分内容：
  - 第一部分：在解析函数成为AST树结构时，会创建一个Activation Object（AO）：
    - AO中包含形参、arguments、函数定义和指向函数对象、定义的变量；
  - 第二部分：作用域链：由VO（在函数中就是AO对象）和父级VO组成，查找时会一层层查找；
  - 第三部分：this绑定的值

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\函数执行上下文.png)



![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\初始化执行过程.png)



```
var message = "Hello Global"
 
function foo() {
  console.log(message)
}
 
function bar() {
  var message = "Hello Bar"
  foo()
}
 
bar()    //输出 Hello Global
```

- 注：**函数父级作用域跟定义位置有关系，与调用位置无关。父级作用域编译时就确定了。**



![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\函数调用栈.png)



- 练习题：
- 不管是 GO 还是 AO，先扫描作用域的代码，变量设置为undefined，函数设置一个指针。、
- 对于函数，先扫描一下代码块，变量设置为undefined，没有定义的变量再去沿作用域链向上找。

```
var n = 100
 
function foo() {
  n = 200
}
 
foo()
 
console.log(n)  //输出 200
```



```
function foo() {
  console.log(n)   //undefined
  var n = 200
  console.log(n)   //200
}
 
var n = 100
foo()
```



```
var n = 100
 
function foo1() {
  console.log(n)   //2. 100
}

function foo2() {
  var n = 200
  console.log(n) //1. 200
  foo1()
}


 
foo2()
console.log(n)  //输出 3. 100
```



## 内存管理和闭包

### **JS的内存管理**

- JavaScript会在**定义变量时**为我们分配内存。
- 但是内存分配方式是一样的吗？
  - JS对于基本数据类型内存的分配会在执行时， 直接在栈空间进行分配；
  - JS对于复杂数据类型内存的分配会在堆内存 中开辟一块空间，并且将这块空间的指针返 回值变量引用；

### **JS的垃圾回收**

- 因为内存的大小是有限的，所以当内存不再需要的时候，我们需要对其进行释放，以便腾出更多的内存空间。
- 在手动管理内存的语言中，我们需要通过一些方式自己来释放不再需要的内存，比如free函数：
  - 但是这种管理的方式其实非常的低效，影响我们编写逻辑的代码的效率；
  - 并且这种方式对开发者的要求也很高，并且一不小心就会产生内存泄露；
- 所以大部分现代的编程语言都是有自己的垃圾回收机制：
  - 垃圾回收的英文是Garbage Collection，简称GC；
  - 对于那些不再使用的对象，我们都称之为是垃圾，它需要被回收，以释放更多的内存空间；
  - 而我们的语言运行环境，比如Java的运行环境JVM，JavaScript的运行环境js引擎都会内存 垃圾回收器；
  - 垃圾回收器我们也会简称为GC，所以在很多地方你看到GC其实指的是垃圾回收器；

- 但是这里又出现了另外一个很关键的问题：GC怎么知道哪些对象是不再使用的呢？
  - 这里就要用到GC的算法了

#### **常见的GC算法 – 引用计数**

- **引用计数：**
  - 当一个对象有一个引用指向它时，那么这个对象的引用就+1，当一个对象的引用为0时，这个对象就可以被销 毁掉；
  - 这个算法有一个很大的弊端就是会产生循环引用；

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\GC-引用计数.png)



#### **常见的GC算法 – 标记清除**

- **标记清除：**
  - 这个算法是设置一个根对象（root object），垃圾回收器会定期从这个根开始，找所有从根开始有引用到的对象，对 于哪些没有引用到的对象，就认为是不可用的对象；
  - 这个算法可以很好的解决循环引用的问题；

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\GC-标记清除.png)



- JS引擎比较广泛的采用的就是标记清除算法，当然类似于V8引擎为了进行更好的优化，它在算法的实现细节上也会结合 一些其他的算法。

### 闭包

闭包不是一个具体的技术，而是一种现象，是指在定义函数时，周围环境中的信息可以在函数中使用。换句话说，执行函数时，只要在函数中使用了外部的数据，就创建了闭包，而作用域链，正是实现闭包的手段。

```
 function eat(){
            var food = "食物";
            console.log(food);
        }
        eat(); // 食物
        console.log(food); // 报错
```

在上面的例子中，声明了一个名为 eat 的函数，并对它进行调用。JavaScript 引擎会创建一个 eat 函数的执行上下文，其中声明 food 变量并赋值。当该方法执行完后，上下文被销毁，food 变量也会跟着消失。这是因为 food 变量属于 eat 函数的局部变量，它作用于 eat 函数中，会随着 eat 的执行上下文创建而创建，销毁而销毁。所以当我们再次打印 food 变量时，就会报错，告诉我们该变量不存在



```
function eat(){
            var food = '食物';
            return function(){
                console.log(food);
            }
        }
        var look = eat();
        look(); // 食物

```

在这个例子中，eat 函数返回一个函数，并在这个内部函数中访问 food 这个局部变量。调用 eat 函数并将结果赋给 look 变量，这个 look 指向了 eat 函数中的内部函数，然后调用它，最终输出 food 的值。

为什么能访问到 food，是因为垃圾回收器只会回收没有被引用到的变量，只要该变量还被引用着的，垃圾回收器就不会回收此变量。在上面的示例中，照理说 eat 调用完毕 food 就应该被销毁掉，但是我们向外部返回了 eat 内部的匿名函数，而这个匿名函数有引用了 food，所以垃圾回收器是不会对其进行回收的，这也是为什么在外面调用这个匿名函数时，仍然能够打印出 food 变量的值。

至此，闭包的一个优点或者特点也就体现出来了，那就是：

通过闭包可以让外部环境访问到函数内部的局部变量。
通过闭包可以让局部变量持续保存下来，不随着它的上下文环境一起销毁。

#### 闭包经典使用场景：

(1) 函数作为返回值（最常用）

```
timeClouse5(){
  function fn(){
	var name="hello";
	return function(){
	  return name;
	}
  }
	var fnc = fn();
	console.log(fnc())     //hello
},

```

fn()的返回值是一个匿名函数，这个函数在fn()作用域内部，，所以它可以获取fn()作用域下变量name的值，将这个值作为返回值赋值给全局作用域下的变量fnc,实现了在全局变量下获取局部变量中变量的值

(2) IIFE（自执行函数）

```
var n = '张三';
	(function p(){
	   console.log(n)
	})()
	/* 输出
	*   张三
	/ 
```

同样也是产生了闭包p()，存在 window下的引用 n。



(3) 定时器与闭包

写一个循环，让它按顺序打印出每次循环i的值

```
timeClouse1(){
        for(var i = 0; i < 5; i++){                             
            setTimeout(function(){
              console.log(i)
            }, 1000) 
        }       
      },

```

分析：按照预期它应该输出0 1 2 3 4，而结果它输出了5，这是为什么呢？由于js是单线程的，在执行for循环的时候定时器setTimeout被安排到任务队列中排队等待执行，而在等待过程中for循环就已经在执行，等到setTimeout可以执行的时候 ，for循环已经结束，所以打印出来五个5，那么我们为了实现预期效果应该怎么改这段代码呢？

**方案一：利用ES6中的let，将var改成let，也可以实现预期效果**

```
timeClouse1(){
        for(let i = 0; i < 5; i++){                             
            setTimeout(function(){
              console.log(i)
            }, 1000) 
        }       
      },

```

**方案二： 引入闭包来保存变量i，将setTimeout放入到立即执行函数中**

```
// 采用自执行函数解决
timeClouse2(){
  for(var i = 0; i < 5; i++){
    (function(j){                  //存在自执行函数
       setTimeout(function(){
         console.log(j)
       }, 1000) 
     })(i)
   }       
 },

```

### 函数的this指向

- this在全局作用于下指向什么？ window
- 但是，开发中很少直接在全局作用于下去使用this，通常都是在函数中使用。
  - 所有的函数在被调用时，都会创建一个执行上下文：
  - 这个上下文中记录着函数的调用栈、AO对象等；
  - this也是其中的一条记录；

```
// this指向什么, 跟函数所处的位置是没有关系的
// 跟函数被调用的方式是有关系
 
function foo() {
  console.log(this)
}
 
// 1.直接调用这个函数
foo()
 
// 2.创建一个对象, 对象中的函数指向foo
var obj = {
  name: 'why',
  foo: foo
}
 
obj.foo()
 
// 3.apply调用
foo.apply("abc")

```

- 我们先来看一个让人困惑的问题：
  - 定义一个函数，我们采用三种不同的方式对它进行调用，它产生了三种不同的结果
- 这个的案例可以给我们什么样的启示呢？
  - 1.函数在调用时，JavaScript会默认给this绑定一个值；
  - 2.this的绑定和定义的位置（编写的位置）没有关系；
  - 3.this的绑定和调用方式以及调用的位置有关系；
  - 4.this是在运行时被绑定的；



- 那么this到底是怎么样的绑定规则呢？ 

  - 绑定一：默认绑定；

  - 绑定二：隐式绑定；

  - 绑定三：显示绑定；

  - 绑定四：new绑定

    

#### this指向-默认绑定

- 什么情况下使用默认绑定呢？独立函数调用。

  - **独立的函数调用我们可以理解成函数没有被绑定到某个对象上进行调用**；

- 我们通过几个案例来看一下，常见的默认绑定，下面的this全是window

  

  ```
  // 默认绑定: 独立函数调用
  // 1.案例一:
  function foo() {
    console.log(this)
  }
   
  foo()
   
  // 2.案例二:
  function foo1() {
    console.log(this)
  }
   
  function foo2() {
    console.log(this)
    foo1()
  }
   
  function foo3() {
    console.log(this)
    foo2()
  }
   
  foo3()
   
   
  // 3.案例三:
  var obj = {
    name: "why",
    foo: function() {
      console.log(this)
    }
  }
   
  var bar = obj.foo
  bar() // window
   
   
  // 4.案例四:
  function foo() {
    console.log(this)
  }
  var obj = {
    name: "why",
    foo: foo
  }
   
  var bar = obj.foo
  bar() // window
   
  // 5.案例五:
  function foo() {
    function bar() {
      console.log(this)
    }
    return bar
  }
  ```




#### this指向-隐式绑定

- 另外一种比较常见的调用方式是通过**某个对象**进行调用的：
  - 也就是**它的调用位置中，是通过某个对象发起的函数调用**。
- 我们通过几个案例来看一下，常见的默认绑定
  - 隐式绑定: object.fn()
  - **object对象会被js引擎绑定到fn函数的中this里面**

```
function foo() {
  console.log(this)
}
 
// 独立函数调用
// foo()
 
// 1.案例一:
var obj = {
  name: "why",
  foo: foo
}
 
obj.foo() // obj对象
 
// 2.案例二:
var obj = {
  name: "why",
  eating: function() {
    console.log(this.name + "在吃东西")
  },
  running: function() {
    console.log(obj.name + "在跑步")
  }
}
 
obj.eating()
obj.running()
 
 
 
// 3.案例三:
var obj1 = {
  name: "obj1",
  foo: function() {
    console.log(this)
  }
}
 
var obj2 = {
  name: "obj2",
  bar: obj1.foo
}
 
obj2.bar()
```



#### this指向-显示绑定

- 隐式绑定有一个前提条件：
  - 必须在调用的对象内部有一个对函数的引用（比如一个属性）；
  - 如果没有这样的引用，在进行调用时，会报找不到该函数的错误；
  - 正是通过这个引用，间接的将this绑定到了这个对象上；
- 如果我们不希望在对象内部 包含这个函数的引用，同时又希望在这个对象上进行强制调用，该怎么做呢？
  - JavaScript所有的函数都可以使用call和apply方法（这个和Prototype有关）。
    - 它们两个的区别这里不再展开；
    - 其实非常简单，第一个参数是相同的，后面的参数，apply为数组，call为参数列表；
  - 这两个函数的第一个参数都要求是一个对象，这个对象的作用是什么呢？就是给this准备的。
  - 在调用这个函数时，会将this绑定到这个传入的对象上。
- 因为上面的过程，我们明确的绑定了this指向的对象，所以称之为 显示绑定



```
function foo() {
  console.log("函数被调用了", this)
}
//1.foo直接调用和call/apply调用的不同在于this绑定的不同
foo直接调用指向的是全局对象(window)
foo()
var obj = {
  name: "obj"
}
//call/apply是可以指定this的绑定对象
foo.call(obj)
foo.apply(obj)
foo.apply("aaaa")
```



#### this 指向-new绑定

- JavaScript中的函数可以当做一个类的构造函数来使用，也就是使用new关键字。
- 使用new关键字来调用函数是，会执行如下的操作：
  - 1.创建一个全新的对象；
  - 2.这个新对象会被执行prototype连接；
  - 3.这个新对象会绑定到函数调用的this上（this的绑定在这个步骤完成）；
  - 4.如果函数没有返回其他对象，表达式会返回这个新对象；

```
// 我们通过一个new关键字调用一个函数时(构造器), 这个时候this是在调用这个构造器时创建出来的对象
// this = 创建出来的对象
// 这个绑定过程就是new 绑定
 
function Person(name, age) {
  this.name = name
  this.age = age
}
 
var p1 = new Person("why", 18)
console.log(p1.name, p1.age)
 
var p2 = new Person("kobe", 30)
console.log(p2.name, p2.age)
```



### **规则优先级**

- **1.默认规则的优先级最低**

  - 毫无疑问，默认规则的优先级是最低的，因为存在其他规则时，就会通过其他规则的方式来绑定this

- **2.显示绑定优先级高于隐式绑定**

  ```
  var obj = {
    name: "obj",
    foo: function() {
      console.log(this)
    }
  }
   
  obj.foo()
   
  // 1.call/apply的显示绑定高于隐式绑定
  obj.foo.apply('abc')
  obj.foo.call('abc')
   
  // 2.bind的优先级高于隐式绑定
  function foo() {
    console.log(this)
  }
   
  var obj = {
    name: "obj",
    foo: foo.bind("aaa")
  }
   
  obj.foo()
  ```

- **3.new绑定优先级高于隐式绑定**

  ```
  var obj = {
    name: "obj",
    foo: function() {
      console.log(this)
    }
  }
   
  // new的优先级高于隐式绑定
  var f = new obj.foo()  // foo {}
  ```

  

- **4.new绑定优先级高于bind**

  - new绑定和call、apply是不允许同时使用的，所以不存在谁的优先级更高
  - new绑定可以和bind一起使用，new绑定优先级更高

  ```
  // new的优先级高于bind
  function foo() {
    console.log(this)
  }
   
  var bar = foo.bind("aaa")
   
  var obj = new bar()  // foo {}
  ```

  - **new绑定 > 显示绑定(apply/call/bind) > 隐式绑定(obj.foo()) > 默认绑定(独立函数调用)**

  **this规则之外 – 忽略显示绑定**

  - 如果在显示绑定中，我们传入一个null或者undefined，那么这个显示绑定会被忽略，使用默认规则：

  ```
  function foo() {
    console.log(this)
  }
  
  var obj = {
    name: "obj"
  }
  
  foo.call(obj)  //obj
  foo.call(null)  //window
  foo.call(undefined) //window
  
  var bar= foo.call(null)
  bar() //window
  ```

  

  **this规则之外 - 间接函数引用**

  - 另外一种情况，创建一个函数的 间接引用，这种情况使用默认绑定规则。
    - 赋值(obj2.foo = obj1.foo)的结果是foo函数；
    - foo函数被直接调用，那么是默认绑定；

  ```
  function foo() {
    console.log(this)
  }
  
  var obj1 = {
    name: "obj1",
    foo:foo
  }
  
  var obj2 = {
    name: "obj2"
  }
  
  obj1.foo()   //obj1
  
  (obj1.foo=obj2.foo)()  //window
  
  ```

  

### **箭头函数 arrow function**

- 箭头函数是ES6之后增加的一种编写函数的方法，并且它比函数表达式要更加简洁：
  - 箭头函数不会绑定this、arguments属性；
  - 箭头函数不能作为构造函数来使用（不能和new一起来使用，会抛出错误）；
- 箭头函数如何编写呢？
  - (): 函数的参数
  - {}: 函数的执行体

箭头函数的编写优化

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\箭头函数的编写优化.png)



```
// 1.编写箭头函数
// 1> (): 参数
// 2> =>: 箭头
// 3> {}: 函数的执行体
var foo = (num1, num2, num3) => {
  console.log(num1, num2, num3)
  var result = num1 + num2 + num3
  console.log(result)
}
 
function bar(num1, num2, num3) {
}
 
// 高阶函数在使用时, 也可以传入箭头函数
var nums = [10, 20, 45, 78]
nums.forEach((item, index, arr) => {})
 
// 箭头函数有一些常见的简写:
// 简写一: 如果参数只有一个, ()可以省略
nums.forEach(item => {
  console.log(item)
})
 
// 简写二: 如果函数执行体只有一行代码, 那么{}也可以省略
// 强调: 并且它会默认将这行代码的执行结果作为返回值
nums.forEach(item => console.log(item))
var newNums = nums.filter(item => item % 2 === 0)
console.log(newNums)
 
// filter/map/reduce
var result = nums.filter(item => item % 2 === 0)
                 .map(item => item * 100)
                 .reduce((preValue, item) => preValue + item)
console.log(result)
 
// 简写三: 如果一个箭头函数, 只有一行代码, 并且返回一个对象, 这个时候如何编写简写
// var bar = () => {
//   return { name: "why", age: 18 }
// }
 
var bar = () => ({ name: "why", age: 18 })
```

#### **this规则之外 – ES6箭头函数**

- ***\*箭头函数不使用this的四种标准规则（也就是不绑定this），而是根据外层作用域来决定this。\****

  ```
  var name = "why"
   
  var foo = () => {
    console.log(this)
  }
   
  foo()
  var obj = {foo: foo}
  obj.foo()
  foo.call("abc")
  ```



### ***arguments、纯函数、柯里化、组合函数***

### **认识arguments**

- arguments 是一个 对应于 传递给函数的参数 的 类数组(array-like)对象。

```
function foo(x, y, z) {
  console.log(arguments)
  //[Arguments]{'0':10, '1':20, '2':30}
}

foo(10, 20, 30)
```

- array-like意味着它不是一个数组类型，而是一个对象类型：
  - 但是它却拥有数组的一些特性，比如说length，比如可以通过index索引来访问；
  - 但是它却没有数组的一些方法，比如forEach、map等；

```
function foo(num1, num2, num3) {
  // 类数组对象中(长的像是一个数组, 本质上是一个对象): arguments
  // console.log(arguments)
 
  // 常见的对arguments的操作是三个
  // 1.获取参数的长度
  console.log(arguments.length)
 
  // 2.根据索引值获取某一个参数
  console.log(arguments[2])
  console.log(arguments[3])
  console.log(arguments[4])
 
  // 3.callee获取当前arguments所在的函数
  console.log(arguments.callee)
  // arguments.callee()
}
 
foo(10, 20, 30, 40, 50)
```

### **箭头函数不绑定arguments**

- 箭头函数是不绑定arguments的，所以我们在箭头函数中使用arguments会去上层作用域查找：

### **理解JavaScript纯函数**

- **函数式编程**中有一个非常重要的概念叫**纯函数**，JavaScript符合**函数式编程的范式**，所以也**有纯函数的概念**；
  - 在**react开发中纯函数是被多次提及**的；
  - 比如**react中组件就被要求像是一个纯函数**（为什么是像，因为还有class组件），**redux中有一个reducer的概念**，也 是要求必须是一个纯函数；
  - 所以**掌握纯函数对于理解很多框架的设计**是非常有帮助的；
- **纯函数的维基百科定义：**
  - 在程序设计中，若一个函数符合以下条件，那么这个函数被称为纯函数：
  - 此函数在相同的输入值时，需产生相同的输出。
  - 函数的输出和输入值以外的其他隐藏信息或状态无关，也和由I/O设备产生的外部输出无关。
  - 该函数不能有语义上可观察的函数副作用，诸如“触发事件”，使输出设备输出，或更改输出值以外物件的内容等。
- **当然上面的定义会过于的晦涩，所以我简单总结一下：**
  - ***\*确定的输入，一定会产生确定的输出\**\**；\****
  - ***\*函数在执行过程中，不能产生副作用\**\**；\****



#### **纯函数的案例**

- 我们来看一个对数组操作的两个函数：
  - slice：slice截取数组时不会对原数组进行任何操作,而是生成一个新的数组；
  - splice：splice截取数组, 会返回一个新的数组, 也会对原数组进行修改；

```
var names = ["abc", "cba", "nba", "dna"]
 
// slice只要给它传入一个start/end, 那么对于同一个数组来说, 它会给我们返回确定的值
// slice函数本身它是不会修改原来的数组
// slice -> this
// slice函数本身就是一个纯函数
var newNames1 = names.slice(0, 3)
console.log(newNames1)
console.log(names)
 
// ["abc", "cba", "nba", "dna"]
// splice在执行时, 有修改掉调用的数组对象本身, 修改的这个操作就是产生的副作用
// splice不是一个纯函数
var newNames2 = names.splice(2)
console.log(newNames2)
console.log(names)
```



```
// foo函数是否是一个纯函数?
// 1.相同的输入一定产生相同的输出
// 2.在执行的过程中不会产生任何的副作用
function foo(num1, num2) {
  return num1 * 2 + num2 * num2
}
 
// bar不是一个纯函数, 因为它修改了外界的变量
var name = "abc" 
function bar() {
  console.log("bar其他的代码执行")
  name = "cba"
}
 
bar()
 
console.log(name)
 
// baz也不是一个纯函数, 因为我们修改了传入的参数
function baz(info) {
  info.age = 100
}
 
var obj = {name: "why", age: 18}
baz(obj)
console.log(obj)
 
// test是否是一个纯函数? 是一个纯函数
function test(info) {
  return {
    ...info,
    age: 100
  }
}
 
test(obj)
test(obj)
 
 
// React的函数组件(类组件) 
function HelloWorld(props) {
  props.info = {}       //不要这样修改 props
  props.info.name = "why"
}
```



### **纯函数的优势**

- **为什么纯函数在函数式编程中非常重要呢？**
  - 因为你可以安心的编写和安心的使用；
  - 你在**写的时候**保证了函数的纯度，只是单纯实现自己的业务逻辑即可，不需要关心传入的内容是如何获得的或 者依赖其他的外部变量是否已经发生了修改；
  - 你在**用的时候**，你确定你的输入内容不会被任意篡改，并且自己确定的输入，一定会有确定的输出；
- React中就要求我们无论是**函数还是class声明一个组件**，这个组件都必须**像纯函数一样**，**保护它们的props不被修****改**

### **JavaScript柯里化**

- **柯里化**也是属于**函数式编程**里面一个非常重要的概念。
- **我们先来看一下维基百科的解释：**
  - 在计算机科学中，**柯里化**（英语：Currying），又译为**卡瑞化**或**加里化**；
  - 是把接收多个参数的函数，变成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参 数，而且返回结果的新函数的技术；
  - 柯里化声称 “如果你固定某些参数，你将得到接受余下参数的一个函数”；
- **维基百科的结束非常的抽象，我们这里做一个总结：**
  - 只传递给函数一部分参数来调用它，让它返回一个函数去处理剩余的参数；
  - 这个过程就称之为柯里化；

```
function add(x, y, z) {
  return x + y + z
}
 
var result = add(10, 20, 30)
console.log(result)
 
function sum1(x) {
  return function(y) {
    return function(z) {
      return x + y + z
    }
  }
}
 
var result1 = sum1(10)(20)(30)
console.log(result1)
 
// 简化柯里化的代码
var sum2 = x => y => z => {
  return x + y + z
}
 
console.log(sum2(10)(20)(30))
 
var sum3 = x => y => z => x + y + z
console.log(sum3(10)(20)(30))
```

#### **让函数的职责单一**

- **那么为什么需要有柯里化呢？**
  - 在函数式编程中，我们其实往往希望一个函数处理的问题尽可能的单一，而不是将一大堆的处理过程交给一个 函数来处理；
  - 那么我们是否就可以将每次传入的参数在单一的函数中进行处理，处理完后在下一个函数中再使用处理后的结 果

```
function add(x, y, z) {
  x = x + 2
  y = y * 2
  z = z * z
  return x + y + z
}
 
console.log(add(10, 20, 30))
 
 
function sum(x) {
  x = x + 2
 
  return function(y) {
    y = y * 2
 
    return function(z) {
      z = z * z
 
      return x + y + z
    }
  }
}
 
console.log(sum(10)(20)(30))
```

#### **柯里化的复用**

- 另外一个使用柯里化的场景是可以帮助我们可以**复用参数逻辑**：
  - makeAdder函数要求我们传入一个num（并且如果我们需要的话，可以在这里对num进行一些修改）；
  - 在之后使用返回的函数时，我们不需要再继续传入num了；

```
function sum(m, n) {
  return m + n
}
 
// 假如在程序中,我们经常需要把5和另外一个数字进行相加
console.log(sum(5, 10))
console.log(sum(5, 14))
console.log(sum(5, 1100))
console.log(sum(5, 555))
 
function makeAdder(count) {
  count = count * count
 
  return function(num) {
    return count + num
  }
}
 
// var result = makeAdder(5)(10)
// console.log(result)
var adder5 = makeAdder(5)
adder5(10)
adder5(14)
adder5(1100)
adder5(555)
```



### 原型链与继承

- JavaScript其实支持多种编程范式的，包括函数式编程和面向对象编程：
  - JavaScript中的对象被设计成一组属性的无序集合，像是一个哈希表，有key和value组成；
  - key是一个标识符名称，value可以是任意类型，也可以是其他对象或者函数类型；
  - 如果值是一个函数，那么我们可以称之为是对象的方法；

#### 创建对象的两种方式

```
// 创建一个对象, 对某一个人进行抽象(描述)
// 1.创建方式一: 通过new Object()创建
var obj = new Object()
obj.name = "why"
obj.age = 18
obj.height = 1.88
obj.running = function() {
  console.log(this.name + "在跑步~")
}
 
// 2.创建方式二: 字面量形式
var info = {
  name: "kobe",
  age: 40,
  height: 1.98,
  eating: function() {
    console.log(this.name + "在吃东西~")
  }
}
```

#### 对属性操作的控制

```
var obj = {
  name: "why",
  age: 18
}
 
// 获取属性
console.log(obj.name)
 
// 给属性赋值
obj.name = "kobe"
console.log(obj.name)
 
// 删除属性
// delete obj.name
// console.log(obj)
```

- 需求: 对属性进行操作时, 进行一些限制
- 限制: 不允许某一个属性被赋值/不允许某个属性被删除/不允许某些属性在遍历时被遍历出来

```
// 遍历属性
for (var key in obj) {
  console.log(key)
}
```

如果我们想要对一个属性进行比较精准的操作控制，那么我们就可以使用属性描述符。

- 通过属性描述符可以精准的添加或修改对象的属性；
- 属性描述符需要使用 Object.defineProperty 来对属性进行添加或者修改；

#### Object.defineProperty

- Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。

```
Object.defineProperty(obj,  prop, descriptor) 
可接收三个参数：
  obj要定义属性的对象；
  prop要定义或修改的属性的名称或 Symbol；
  descriptor要定义或修改的属性描述符；
返回值：被传递给函数的对象
```

#### **属性描述符分类**

- 属性描述符的类型有两种：
  - **数据属性**（Data Properties）描述符（Descriptor）；
  - **存取属性**（Accessor访问器 Properties）描述符（Descriptor）；

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\属性描述符.png)

#### **数据属性描述符**

- 数据数据描述符有如下四个特性：

  - **[[Configurable]]**

    ：表示属性是否可以通过delete删除属性，是否可以修改它的特性，或者是否可以将它修改为存取属性描述符；

    - 当我们直接在一个对象上定义某个属性时，这个属性的[[Configurable]]为true；
    - 当我们通过属性描述符定义一个属性时，这个属性的[[Configurable]]默认为false；

  - **[[Enumerable]]**

    ：表示属性是否可以通过for-in或者Object.keys()返回该属性；

    - 当我们直接在一个对象上定义某个属性时，这个属性的[[Enumerable]]为true；
    - 当我们通过属性描述符定义一个属性时，这个属性的[[Enumerable]]默认为false；

  - **[[Writable]]**

    ：表示是否可以修改属性的值；

    - 当我们直接在一个对象上定义某个属性时，这个属性的[[Writable]]为true；
    - 当我们通过属性描述符定义一个属性时，这个属性的[[Writable]]默认为false；

  - **[[value]]**

    ：属性的value值，读取属性时会返回该值，修改属性时，会对其进行修改；

    - 默认情况下这个值是undefined；

```
// name和age虽然没有使用属性描述符来定义, 但是它们也是具备对应的特性的
// value: 赋值的value
// configurable: true
// enumerable: true
// writable: true
 
var obj = {
  name: "why",
  age: 18
}
 
// 数据属性描述符
// 用了属性描述符, 那么会有默认的特性
Object.defineProperty(obj, "address", {
  // 很多配置
  value: "北京市", // 默认值undefined
  // 该特性不可删除/也不可以重新定义属性描述符
  configurable: false, // 默认值false
  // 该特性是配置对应的属性(address)是否是可以枚举
  enumerable: true, // 默认值false
  // 该特性是属性是否是可以赋值(写入值) 
  writable: false // 默认值false
})
```

#### **存取属性描述符**

- Configurable Enumerable 与上面相同
- [[get]]：获取属性时会执行的函数。默认为undefined
- [[set]]：设置属性时会执行的函数。默认为undefined
   
- 存取属性描述符
  - 1.隐藏某一个私有属性被希望直接被外界使用和赋值
  - 2.如果我们希望截获某一个属性它访问和设置值的过程时, 也会使用存储属性描述符

```
var obj = {
  name: "why",
  age: 18,
  _address: "北京市"
}
 
Object.defineProperty(obj, "address", {
  enumerable: true,
  configurable: true,
  get: function() {
    foo()
    return this._address
  },
  set: function(value) {
    bar()
    this._address = value
  }
})
 
console.log(obj.address)
 
obj.address = "上海市"
console.log(obj.address)
 
function foo() {
  console.log("获取了一次address的值")
}
 
function bar() {
  console.log("设置了addres的值")
}
```

#### **同时定义多个属性**

- Object.defineProperties() 方法直接在一个对象上定义 多个 新的属性或修改现有属性，并且返回该对象。

```
var obj = {
  // 私有属性(js里面是没有严格意义的私有属性)
  _age: 18,
  _eating: function() {},
  // set age(value) {
  //   this._age = value
  // },
  // get age() {
  //   return this._age
  // }
}
 
Object.defineProperties(obj, {
  name: {
    configurable: true,
    enumerable: true,
    writable: true,
    value: "why"
  },
  age: {
    configurable: true,
    enumerable: true,
    get: function() {
      return this._age
    },
    set: function(value) {
      this._age = value
    }
  }
})
 
obj.age = 19
console.log(obj.age)
 
console.log(obj)
```

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\属性.png)



-  获取属性

```
// 获取某一个特性属性的属性描述符
console.log(Object.getOwnPropertyDescriptor(obj, "name"))
console.log(Object.getOwnPropertyDescriptor(obj, "age"))
 
// 获取对象的所有属性描述符
// console.log(Object.getOwnPropertyDescriptors(obj))
```

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\属性2.png)



#### **Object的方法对对象限制**

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\属性3.png)



```
var obj = {
  name: 'why',
  age: 18
}
 
// 1.禁止对象继续添加新的属性
Object.preventExtensions(obj)
 
obj.height = 1.88
obj.address = "广州市"
 
console.log(obj)
 
// 2.禁止对象配置/删除里面的属性
// for (var key in obj) {
//   Object.defineProperty(obj, key, {
//     configurable: false,
//     enumerable: true,
//     writable: true,
//     value: obj[key]
//   })
// }
 
Object.seal(obj)
 
delete obj.name
console.log(obj.name)
 
// 3.让属性不可以修改(writable: false)
Object.freeze(obj)
 
obj.name = "kobe"
console.log(obj.name)
```



### **创建对象**

#### **工厂模式**

```
// 工厂模式: 工厂函数
function createPerson(name, age, height, address) {
  var p = {}
  p.name = name
  p.age = age
  p.height = height;
  p.address = address
 
  p.eating = function() {
    console.log(this.name + "在吃东西~")
  }
 
  p.running = function() {
    console.log(this.name + "在跑步~")
  }
 
  return p
}
 
var p1 = createPerson("张三", 18, 1.88, "广州市")
var p2 = createPerson("李四", 20, 1.98, "上海市")
var p3 = createPerson("王五", 30, 1.78, "北京市")
 
// 工厂模式的缺点(获取不到对象最真实的类型)
console.log(p1, p2, p3)
```

#### **构造函数**

- 工厂方法创建对象有一个比较大的问题：我们在打印对象时，对象的类型都是Object类型
- JavaScript中的构造函数是怎么样的？
  - 构造函数也是一个普通的函数，从表现形式来说，和千千万万个普通的函数没有任何区别；
  - 那么如果这么一个普通的函数被使用new操作符来调用了，那么这个函数就称之为是一个构造函数；
- **new操作符调用的作用**

1. 如果一个函数被使用new操作符调用了，那么它会执行如下操作：
2. 在内存中创建一个新的对象（空对象）；
3. 这个对象内部的[[prototype]]属性会被赋值为该构造函数的prototype属性； 
4. 构造函数内部的this，会指向创建出来的新对象；
5. 执行函数的内部代码（函数体代码）；
6. 如果构造函数没有返回非空对象，则返回创建出来的新对象；

```
function foo() {
  console.log("foo~, 函数体代码")
}
 
// foo就是一个普通的函数
// foo()
 
// 换一种方式来调用foo函数: 通过new关键字去调用一个函数, 那么这个函数就是一个构造函数了
var f1 = new foo
console.log(f1)
```

#### **创建对象方案-构造函数**

```
function Person(name, age, height, address) {
  this.name = name
  this.age = age
  this.height = height
  this.address = address
 
  this.eating = function() {
    console.log(this.name + "在吃东西~")
  }
 
  this.running = function() {
    console.log(this.name + "在跑步")
  }
}
 
 
var p1 = new Person("张三", 18, 1.88, "广州市")
var p2 = new Person("李四", 20, 1.98, "北京市")
 
console.log(p1)
console.log(p2)
p1.eating()
p2.eating()
```

![](D:\bilibili\myWorkSpace\githubReact\深入javaScript高级\images\属性4.png)



- 但是构造函数就没有缺点了吗？
  - 构造函数也是有缺点的，它在于我们需要**为每个对象的函数去创建一个函数对象实例**；

### **对象的原型**

















### ES6常见遍历数组函数的用法

#### **一、forEach**

[回调函数](https://so.csdn.net/so/search?q=回调函数&spm=1001.2101.3001.7020)参数，item(数组元素)、index(序列)、arr(数组本身) 循环数组，无返回值，不改变原数组
不支持return操作输出，return只用于控制循环是否跳出当前循环

```
var myArr=[{id:1,name:“sdf”},{id:2,name:“dfsdf”}, {id:3,name:“fff”}]
myArr.forEach((item,index)=>{
      console.log(item.id);}
       //结果：1,2,3
})

```

注意：[foreach](https://so.csdn.net/so/search?q=foreach&spm=1001.2101.3001.7020)适用于只是进行集合或数组遍历，for则在较复杂的循环中效率更高。

#### **二、includes**

判断数组是否包含某个元素
1、参数：第一个是查询的元素，第二个是从什么位置开始查询(可以是负数)
2、不用return，不用回调函数，返回布尔值。
3、[ES6](https://so.csdn.net/so/search?q=ES6&spm=1001.2101.3001.7020)提供了Array.includes()函数判断是否包含某一元素，除了不能定位外，解决了indexOf的上述的两个问题。它直接返回true或者false表示是否包含元素，对NaN一样能有有效。

```
const arr1 = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', NaN]
console.log('%s', arr1.includes('c'))  //true
console.log('%s', arr1.includes(NaN))  //true
console.log('%s', arr1.includes('d', 4))  //false
console.log('%s', arr1.includes('k', -1)) //false

```

注意：在ES5，Array已经提供了indexOf用来查找某个元素的位置，如果不存在就返回-1，但是这个函数在判断数组是否包含某个元素时有两个小不足，第一个是它会返回-1和元素的位置来表示是否包含，在定位方面是没问题，就是不够语义化。另一个问题是不能判断是否有NaN的元素。

```
const arr1 = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', NaN]
console.log('%s', arr1.indexOf(NaN))
//输出：-1

```

#### **三、filter**

使用return操作输出，会循环数组每一项，并在回调函数中操作
过滤筛选的作用，数组filter后，返回的结果为新的数组

```
let arr1 = [
            {name: 'zhangsan', age: 18, sex: 'male'},
            {name: 'lisi', age: 30, sex: 'male'},
            {name: 'xiaohong', age: 20, sex: 'female'}
        ];
let arr2 = arr1.filter(item => item.age > 20);
        console.log(arr2);   
       // [{name: "lisi", age: 30, sex: "male"}]
       
        console.log(arr1);   
      // [{name: 'zhangsan', age: 18, sex: 'male'},{name: 'lisi', age: 30, sex: 'male'},{name: 'xiaohong', age: 20, sex: 'female'}]

```



#### **四、map**

输出的是return什么就输出什么新数组
原数组被“映射”成对应新数组，返回新数组，不改变原数组

```
var num = [1,2,3,4];
var dataAdd = num.map(n => n+n);
var datadeep = num.map(n => n-1);
console.log(dataAdd);//[2, 4, 6, 8]
console.log(datadeep);//[0,1,2,3]

```

#### **五、find**

用来查找目标元素，找到就返回该元素，找不到返回undefined
输出的是一旦判断为true则跳出循环输出符合条件的[数组元素](https://so.csdn.net/so/search?q=数组元素&spm=1001.2101.3001.7020)

```
let arr1 = [
            {name: 'zhangsan', age: 18, sex: 'male'},
            {name: 'lisi', age: 30, sex: 'male'},
            {name: 'xiaohong', age: 20, sex: 'female'}
        ];
 let arr2 = arr1.filter(item => item.name === 'zhangsan');
        console.log(arr2); 
        //返回两条数据 [{name: "zhangsan", age: 18, sex: "male"}1: {name: "zhangsan", age: 20, sex: "female"}]

```

#### **六、some**

some 英语翻译为一些,every翻译为所有,每个，所以some方法 只要其中一个为true 就会返回true的，相反，every（）方法必须所有都返回true才会返回true，哪怕有一个false，就会返回false；every（）和 some（）目的：确定数组的所有成员是否满足指定的测试。

```
var computers = [
    {name:"Apple",ram:8},
    {name:"IBM",ram:4},
    {name:"Acer",ram:32},
];
 var result= computers.every(function(computer){
   return computer.ram > 16
})
console.log(result)//false;

var some = computers.some(function(computer){
   return computer.ram > 16
})
console.log(some)//true;

```

#### **七、reduce**

累加器，输出的是return叠加什么就输出什么
回调函数参数四个
pre: 初始值（之后为上一操作的结果）
cur: 当前元素之
index: 当前索引
arr: 数组本身
主要有以下几种用法：

*1.数组求和*

```
[1,2,3,4].reduce((pre, cur) => pre + cur)//10

```

*2.二维数组转为一维数组*

```
[[1, 2], [3, 4], [5, 6]].reduce(( acc, cur ) => [...acc, ...cur], [])//[1, 2, 3, 4, 5, 6]

```

*3.计算数组中每个元素出现的次数*

```
const arraySum = (arr, val) => arr.reduce((acc, cur) => {
return cur == val ? acc + 1 : acc + 0
}, 0);
let arr = [1, 2, 3, 4, 5];
arraySum(arr, 0) //3

```

**八、new Set()**

基本思路：
1、ES6提供了新的数据结构Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。
2、Set函数可以接受一个数组（或类似数组的对象）作为参数，用来初始化。
3、任何类似数组的对象可以用扩展运算符转换为真正的数组。

```
let arr = [1, 2, 2, 3];
let set = new Set(arr);
let newArr = Array.from(set);
console.log(newArr); // [1, 2, 3]

```

https://www.dmzshequ.com/thread-21337-1-1.html



