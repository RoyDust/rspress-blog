	#    高阶JavaScript笔记

## 一、浏览器基础

### 1、浏览器的工作原理

![image-20220215222644207](https://s2.loli.net/2022/02/15/3ibVuTvMR67e5Ep.png)

### 2.浏览器的内核

不同的浏览器有不同的内核组成

- Gecko：早期被Netscape和Mozilla Firefox浏览器浏览器使用；
- Trident：微软开发，被IE4~IE11浏览器使用，但是Edge浏览器已经转向Blink；
- Webkit：苹果基于KHTML开发、开源的，用于Safari，Google Chrome之前也在使用；
- Blink：是Webkit的一个分支，Google开发，目前应用于Google Chrome、Edge、Opera等；

### 3.浏览器渲染过程

![image-20220217152033306](https://s2.loli.net/2022/02/17/ob4iHnTAShPKdjO.png)

### 4.认识JavaScript引擎

为什么需要JavaScript引擎呢？

- 我们前面说过，高级的编程语言都是需要转成最终的机器指令来执行的；
- 事实上我们编写的JavaScript无论你交给浏览器或者Node执行，最后都是需要被CPU执行的；
- 但是CPU只认识自己的指令集，实际上是机器语言，才能被CPU所执行；
- 所以我们需要JavaScript引擎帮助我们将JavaScript代码翻译成CPU指令来执行；

### 5.浏览器内核和JS引擎的关系

这里我们先以WebKit为例，WebKit事实上由两部分组成的：

- WebCore：负责HTML解析、布局、渲染等等相关的工作；
- JavaScriptCore：解析、执行JavaScript代码；

![image-20220217152250749](https://s2.loli.net/2022/02/17/w2hO9Su6pyP74Fk.png)

### 6.V8引擎的原理

- V8是用C ++编写的Google开源高性能JavaScript和WebAssembly引擎，它用于Chrome和Node.js等。
- 它实现ECMAScript和WebAssembly，并在Windows 7或更高版本，macOS 10.12+和使用x64，IA-32，ARM或MIPS处理器的Linux系统上运行。
- V8可以独立运行，也可以嵌入到任何C ++应用程序中

![image-20220217152626046](https://s2.loli.net/2022/02/17/UbkXFhvsTqDVlBJ.png)

- Parse模块会将JavaScript代码转换成AST（抽象语法树），这是因为解释器并不直接认识JavaScript代码；
  - 如果函数没有被调用，那么是不会被转换成AST的；
-  Ignition是一个解释器，会将AST转换成ByteCode（字节码）
  - 同时会收集TurboFan优化所需要的信息（比如函数参数的类型信息，有了类型才能进行真实的运算）；
  -  如果函数只调用一次，Ignition会执行解释执行ByteCode；
-  TurboFan是一个编译器，可以将字节码编译为CPU可以直接执行的机器码；
  - 如果一个函数被多次调用，那么就会被标记为热点函数，那么就会经过TurboFan转换成优化的机器码，提高代码的执行性能；
  -  但是，机器码实际上也会被还原为ByteCode，这是因为如果后续执行函数的过程中，类型发生了变化（比如sum函数原来执行的是number类型，后来执行变成了string类型），之前优化的机器码并不能正确的处理运算，就会逆向的转换成字节码；

#### V8执行的细节

- 我们的JavaScript源码是如何被解析（Parse过程）的呢？
-  Blink将源码交给V8引擎，Stream获取到源码并且进行编码转换；
-  Scanner会进行词法分析（lexical analysis），词法分析会将代码转换成tokens；
-  接下来tokens会被转换成AST树，经过Parser和PreParser：
  - Parser就是直接将tokens转成AST树架构；
  -  PreParser称之为预解析，为什么需要预解析呢？
    - 这是因为并不是所有的JavaScript代码，在一开始时就会被执行。那么对所有的JavaScript代码进行解析，必然会影响网页的运行效率；
    -  所以V8引擎就实现了Lazy Parsing（延迟解析）的方案，它的作用是将不必要的函数进行预解析，也就是只解析暂时需要的内容，而对函数的全量解析是在函数被调用时才会进行；
    -  比如我们在一个函数outer内部定义了另外一个函数inner，那么inner函数就会进行预解析；
-  生成AST树后，会被Ignition转成字节码（bytecode），之后的过程就是代码的执行过程（后续会详细分析）。

### 7.JavaScript的执行过程

#### 初始化全局对象

js引擎会在执行代码之前，会在堆内存中创建一个全局对象：**Global Object（GO）**

- 该对象 所有的作用域（scope）都可以访问；
- 里面会包含Date、Array、String、Number、setTimeout、setInterval等等；
- 其中还有一个window属性指向自己；

![image-20220217153337244](https://s2.loli.net/2022/02/17/E9ixycNbYQpt3Kw.png)

#### 执行上下文栈（调用栈）

- js引擎内部有一个执行上下文栈（Execution Context Stack，简称ECS），它是用于执行代码的调用栈。
- 那么现在它要执行谁呢？执行的是全局的代码块：
  - 全局的代码块为了执行会构建一个 Global Execution Context（GEC）；
  - **GEC会 被放入到ECS中 执行**；
-  GEC被放入到ECS中里面包含两部分内容：
  - 第一部分：在代码执行前，在parser转成AST的过程中，**会将全局定义的变量、函数等加入到GlobalObject中**，**但是并不会赋值；**
    - 这个过程也称之为**变量的作用域提升**（hoisting） 
  - 第二部分：在代码执行中，**对变量赋值，或者执行其他的函数；**

#### GEC被放入到ECS中

![image-20220217153817642](https://s2.loli.net/2022/02/17/wXik1gIlKpsW8RL.png)

#### GEC开始执行代码

![image-20220217153926742](https://s2.loli.net/2022/02/17/4FBybWCGKoAZhlQ.png)

#### 遇到函数如何执行？

- 在执行的过程中执行到一个函数时，就会根据函数体创建一个**函数执行上下文（Functional Execution Context，简称FEC），并且压入到ECS Stack中。**
- FEC中包含三部分内容：
  - 第一部分：在解析函数成为AST树结构时，会创建一个Activation Object（AO）：
    - AO中包含形参、arguments、函数定义和指向函数对象、定义的变量；
  - 第二部分：作用域链：由VO（在函数中就是AO对象）和父级VO组成，查找时会一层层查找；
  - 第三部分：this绑定的值：这个我们后续会详细解析；

![image-20220217154444223](https://s2.loli.net/2022/02/17/xOahfswUozVFEW4.png)

再FEC被放入到ECS中

![image-20220217154530974](https://s2.loli.net/2022/02/17/MHt1nDozpeICFZP.png)

![image-20220217165340560](https://s2.loli.net/2022/02/17/dGXPVSMx4IEjOrU.png)

### 8、内存管理

- 不管什么样的编程语言，在代码的执行过程中都是需要给它分配内存的，不同的是某些编程语言需要我们自己手动的管理内存，某些编程语言会可以自动帮助我们管理内存：
- 不管以什么样的方式来管理内存，内存的管理都会有如下的生命周期
  - 第一步：分配申请你需要的内存（申请）；
  - 第二步：使用分配的内存（存放一些东西，比如对象等）；
  - 第三步：不需要使用时，对其进行释放；

#### JS中的内存调用

- JavaScript会在定义变量时为我们分配内存。
- 但是内存分配方式是一样的吗？
  - JS对于基本数据类型内存的分配会在执行时，直接在栈空间进行分配；
  - JS对于复杂数据类型内存的分配会在堆内存中开辟一块空间，并且将这块空间的指针返回值变量引用；

![image-20220219224851774](https://s2.loli.net/2022/02/19/jIDpX7CvzhZoQWy.png)

#### JS的垃圾回收

- 因为内存的大小是有限的，所以当内存不再需要的时候，我们需要对其进行释放，以便腾出更多的内存空间。
- 在手动管理内存的语言中，我们需要通过一些方式自己来释放不再需要的内存，比如free函数：
  - 但是这种管理的方式其实非常的低效，影响我们编写逻辑的代码的效率；
  - 并且这种方式对开发者的要求也很高，并且一不小心就会产生内存泄露；
- 所以大部分现代的编程语言都是有自己的垃圾回收机制：
  - 垃圾回收的英文是Garbage Collection，简称GC；
  - 对于那些不再使用的对象，我们都称之为是垃圾，它需要被回收，以释放更多的内存空间；
  - 而我们的语言运行环境，比如Java的运行环境JVM，JavaScript的运行环境js引擎都会内存 垃圾回收器；
  - 垃圾回收器我们也会简称为GC，所以在很多地方你看到GC其实指的是垃圾回收器；

##### 常见的GC算法 – 引用计数

引用计数：

- 当一个对象有一个引用指向它时，那么这个对象的引用就+1，当一个对象的引用为0时，这个对象就可以被销毁掉；
- 这个算法有一个很大的弊端就是会产生循环引用；

![image-20220219225104388](https://s2.loli.net/2022/02/19/Zwirlso1BzdFAEO.png)

##### 常见的GC算法 – 标记清除

标记清除：

- 这个算法是设置一个根对象（root object），垃圾回收器会定期从这个根开始，找所有从根开始有引用到的对象，对于哪些没有引用到的对象，就认为是不可用的对象；

- 这个算法可以很好的解决循环引用的问题；

- JS引擎比较广泛的采用的就是标记清除算法，当然类似于V8引擎为了进行更好的优化，它在算法的实现细节上也会结合一些其他的算法

  ![image-20220219225308651](https://s2.loli.net/2022/02/19/rZQCkbHtB4xLEyI.png)

## 二、闭包

**JS中函数是一等公民**

### 定义

- 一个普通的函数function，如果它可以访问外层作用于的自由变量，那么这个函数就是一个闭包；
-  从广义的角度来说：**JavaScript中的函数都是闭包；**
-  从狭义的角度来说：**JavaScript中一个函数，如果访问了外层作用域的变量，那么它是一个闭包；**

### 闭包的访问过程

![image-20220224001448395](https://s2.loli.net/2022/02/24/wfS6RGEbHzYArPd.png)

### 闭包的执行过程

- 这个时候makeAdder函数执行完毕，正常情况下我们的AO对象会被释放；
- 但是因为在0xb00的函数中有作用域引用指向了这个AO对象，所以它不会被释放掉

![image-20220224001521304](https://s2.loli.net/2022/02/24/qzOGAu29iyB5lRV.png)

### 闭包的内存泄露

- 那么我们为什么经常会说闭包是有内存泄露的呢？

  - 在上面的案例中，如果后续我们不再使用add10函数了，那么该函数对象应该要被销毁掉，并且其引用着的父作用域AO也应该被销毁掉；
  - 但是目前因为在全局作用域下add10变量对0xb00的函数对象有引用，而0xb00的作用域中AO（0x200）有引用，所以最终会造成这些内存都是无法被释放的；
  - 所以我们经常说的闭包会造成内存泄露，其实就是刚才的引用链中的所有对象都是无法释放的；

-  那么，怎么解决这个问题呢？

  - 因为当将add10设置为null时，就不再对函数对象0xb00有引用，那么对应的AO对象0x200也就不可达了；

  - 在GC的下一次检测中，它们就会被销毁掉；

    ```js
    add10 = null
    ```

### **AO不使用的属性**

- 我们来研究一个问题：AO对象不会被销毁时，是否里面的所有属性都不会被释放？

  - 下面这段代码中name属于闭包的父作用域里面的变量；

  - 我们知道形成闭包之后count一定不会被销毁掉，那么name是否会被销毁掉呢？ 

  - 这里我打上了断点，我们可以在浏览器上看看结果；

```js
function foo() {
  var name = "why"
  var age = 18

  function bar() {
    debugger
    console.log(age)
  }

  return bar
}

var fn = foo()
fn()
```

![image-20220504220428040](https://s2.loli.net/2022/05/04/ZBS4MCIo6UsKEjP.png)

##  三、JS函数的this指向

### this绑定规则

- 绑定一：默认绑定；

  独立的函数调用我们可以理解成函数没有被绑定到某个对象上进行调用

  ```js
  function foo() {
    console.log(this)
  }
  var obj = {
    name: "why",
    foo: foo
  }
  
  var bar = obj.foo
  bar() // window
  ```

  

- 绑定二：隐式绑定；

  它的调用位置中，是通过某个对象发起的函数调用

  ```js
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
  
  obj2.bar()	// { name: 'obj2', bar: [Function: foo] }
  ```

  

- 绑定三：显示绑定；

  前提：

  - 必须在调用的对象内部有一个对函数的引用（比如一个属性）；

  - 如果没有这样的引用，在进行调用时，会报找不到该函数的错误；

  - 正是通过这个引用，间接的将this绑定到了这个对象上；

  强制调用（显示绑定）：JavaScript所有的函数都可以使用call和apply方法（这个和Prototype有关）

  > call和apply的区别：第一个参数是相同的，后面的参数，apply为数组，call为参数列表；

  这两个函数的第一个参数都要求是一个对象，这个对象的作用是什么呢？就是给this准备的。

  在调用这个函数时，会将this绑定到这个传入的对象上

- 绑定四：new绑定；

  - JavaScript中的函数可以当做一个类的构造函数来使用，也就是使用new关键字。

  - 使用new关键字来调用函数是，会执行如下的操作：

  1. 创建一个全新的对象；
  2. 这个新对象会被执行prototype连接；
  3. 这个新对象会绑定到函数调用的this上（this的绑定在这个步骤完成）；
  4. 如果函数没有返回其他对象，表达式会返回这个新对象；

```js
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


var obj = {
  foo: function() {
    console.log(this)
  }
}
```

### **规则优先级**

1. **默认规则的优先级最低**
2. **显示绑定优先级高于隐式绑定**
3. **.new绑定优先级高于隐式绑定**
4. **new绑定优先级高于bind**

new绑定 > 显示绑定(apply/call/bind) > 隐式绑定(obj.foo()) > 默认绑定(独立函数调用)

### **this规则之外**

#### **忽略显示绑定**

如果在显示绑定中，我们传入一个null或者undefined，那么这个显示绑定会被忽略，使用默认规则：

```js
function foo() {
  console.log(this)
}

foo.apply("abc")
foo.apply({})

// apply/call/bind: 当传入null/undefined时, 自动将this绑定成全局对象
foo.apply(null)
foo.apply(undefined)

var bar = foo.bind(null)
bar()
```

#### **间接函数引用**

创建一个函数的 间接引用，这种情况使用默认绑定规则

```js
// 争论: 代码规范 ;

var obj1 = {
  name: "obj1",
  foo: function () {
    console.log(this)
  }
}

var obj2 = {
  name: "obj2"
}; 

// obj2.bar = obj1.foo
// obj2.bar()

(obj2.bar = obj1.foo)()
```

#### **ES6箭头函数**

箭头函数不使用this的四种标准规则（也就是不绑定this），而是根据外层作用域来决定this

我们来看一个模拟网络请求的案例：

- 这里我使用setTimeout来模拟网络请求，请求到数据后如何可以存放到data中呢？

- 我们需要拿到obj对象，设置data； 

- 但是直接拿到的this是window，我们需要在外层定义：var _this = this

- 在setTimeout的回调函数中使用_this就代表了obj对象

```js
var obj = {
  data: [],
  getData: function() {
    // 发送网络请求, 将结果放到上面data属性中
    // 在箭头函数之前的解决方案
    // var _this = this
    // setTimeout(function() {
    //   var result = ["abc", "cba", "nba"]
    //   _this.data = result
    // }, 2000);
    // 箭头函数之后
    setTimeout(() => {
      var result = ["abc", "cba", "nba"]
      this.data = result
    }, 2000);
  }
}
```

### bind，apply，call的区别

- apply接受两个参数，第一个参数是this的指向，第二个参数是函数接受的参数，以数组的形式传入，且当第一个参数为null、undefined的时候，默认指向window(在浏览器中)，使用apply方法改变this指向后原函数会立即执行，且此方法只是临时改变this指向一次。

- call方法的第一个参数也是this的指向，后面传入的是一个参数列表（注意和apply传参的区别）。当一个参数为null或undefined的时候，表示指向window（在浏览器中），和apply一样，call也只是临时改变一次this指向，并立即执行。
- bind方法和call很相似，第一参数也是this的指向，后面传入的也是一个参数列表(但是这个参数列表可以分多次传入，call则必须一次性传入所有参数)，但是它改变this指向后不会立即执行，而是返回一个永久改变this指向的函数。

用JS代码模拟bind，apply，call的源码实现：

> call的实现

```js
// 给所有的函数添加一个hycall的方法
Function.prototype.hycall = function(thisArg, ...args) {
  // 在这里可以去执行调用的那个函数(foo)
  // 问题: 得可以获取到是哪一个函数执行了hycall
  // 1.获取需要被执行的函数
  var fn = this

  // 2.对thisArg转成对象类型(防止它传入的是非对象类型)
  thisArg = (thisArg !== null && thisArg !== undefined) ? Object(thisArg): window

  // 3.调用需要被执行的函数 相当于thisArg的隐式绑定了fn(原来要执行的函数foo())
  thisArg.fn = fn
  var result = thisArg.fn(...args)	//将args展开执行fn()
  delete thisArg.fn

  // 4.将最终的结果返回出去
  return result
}


function foo() {
  console.log("foo函数被执行", this)
}

function sum(num1, num2) {
  console.log("sum函数被执行", this, num1, num2)
  return num1 + num2
}


// 系统的函数的call方法
foo.call(undefined)
var result = sum.call({}, 20, 30)
console.log("系统调用的结果:", result)


// 自己实现的函数的hycall方法
// 默认进行隐式绑定
// foo.hycall({name: "why"})
foo.hycall(undefined)
var result = sum.hycall("abc", 20, 30)
console.log("hycall的调用:", result)

```

> apply的实现

```js
// 自己实现hyapply
Function.prototype.hyapply = function(thisArg, argArray) {
  // 1.获取到要执行的函数
  var fn = this

  // 2.处理绑定的thisArg 当thisArg=0时，都不会判断为false
  thisArg = (thisArg !== null && thisArg !== undefined) ? Object(thisArg): window

  // 3.执行函数
  thisArg.fn = fn
  var result
  // if (!argArray) { // argArray是没有值(没有传参数)
  //   result = thisArg.fn()
  // } else { // 有传参数
  //   result = thisArg.fn(...argArray)
  // }

  // argArray = argArray ? argArray: []
  argArray = argArray || []			//防止argArray没有参数为undefined
  result = thisArg.fn(...argArray)

  delete thisArg.fn

  // 4.返回结果
  return result
}

function sum(num1, num2) {
  console.log("sum被调用", this, num1, num2)
  return num1 + num2
}

function foo(num) {
  return num
}

function bar() {
  console.log("bar函数被执行", this)
}

// 系统调用
// var result = sum.apply("abc", 20)
// console.log(result)

// 自己实现的调用
// var result = sum.hyapply("abc", [20, 30])
// console.log(result)

// var result2 = foo.hyapply("abc", [20])
// console.log(result2)
```

> bind的实现

```js
Function.prototype.hybind = function(thisArg, ...argArray) {
  // 1.获取到真实需要调用的函数
  var fn = this

  // 2.绑定this
  thisArg = (thisArg !== null && thisArg !== undefined) ? Object(thisArg): window

  function proxyFn(...args) {
    // 3.将函数放到thisArg中进行调用
    thisArg.fn = fn
    // 特殊: 对两个传入的参数进行合并  展开拼接在一起
    var finalArgs = [...argArray, ...args]
    var result = thisArg.fn(...finalArgs)
    delete thisArg.fn

    // 4.返回结果
    return result
  }

  return proxyFn
}

function foo() {
  console.log("foo被执行", this)
  return 20
}

function sum(num1, num2, num3, num4) {
  console.log(num1, num2, num3, num4)
}

// 系统的bind使用
var bar = foo.bind("abc")
bar()

// var newSum = sum.bind("aaa", 10, 20, 30, 40)
// newSum()

// var newSum = sum.bind("aaa")
// newSum(10, 20, 30, 40)

// var newSum = sum.bind("aaa", 10)
// newSum(20, 30, 40)


// 使用自己定义的bind
// var bar = foo.hybind("abc")
// var result = bar()
// console.log(result)

var newSum = sum.hybind("abc", 10, 20)
var result = newSum(30, 40)
```



### 展开运算符 ... spread

```js
function sum(...nums) {
  console.log(nums)
}

sum(10)
sum(10, 20)
sum(10, 20, 30)
sum(10, 20, 30, 40, 50)

// 展开运算符 spread
var names = ["abc", "cba", "nba"]
// var newNames = [...names]
function foo(name1, name2, name3) {}
foo(...names)
```

### Arguments对象

#### arguments的使用

虽然arguments对象并不是一个数组（类数组），但是访问单个参数的方式与访问数组元素的方式相同

```js
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
  // arguments.callee() 无限死循环
}

foo(10, 20, 30, 40, 50)
```

#### arguments转Array

```JS
function foo(num1, num2) {
  // 1.自己遍历
  var newArr = []
  for (var i = 0; i < arguments.length; i++) {
    newArr.push(arguments[i] * 10)
  }
  console.log(newArr)

  // 2.arguments转成array类型
  // 2.1.自己遍历arguments中所有的元素
  // 2.2.Array.prototype.slice将arguments转成array
  var newArr2 = Array.prototype.slice.call(arguments)
  console.log(newArr2)

  // 这种方式也可以
  var newArr3 = [].slice.call(arguments)
  console.log(newArr3)

  // 2.3.ES6的语法
  var newArr4 = Array.from(arguments)
  console.log(newArr4)
  var newArr5 = [...arguments]
  console.log(newArr5)
}

foo(10, 20, 30, 40, 50)
```

#### 箭头函数没有arguments

```js
function foo() {
  var bar = () => {
    console.log(arguments)
  }
  return bar
}

var fn = foo(123)
fn()  //[Arguments] { '0': 123 }


var foo = (num1, num2, ...args) => {
  console.log(args)
}

foo(10, 20, 30, 40, 50) //[ 30, 40, 50 ]
```



## 四、纯函数与柯里化

### **理解JavaScript纯函数**

- 确定的输入，一定会产生确定的输出； 

- 函数在执行过程中，不能产生**副作用**；

  **那么这里又有一个概念，叫做副作用**，什么又是**副作用**呢？

  **副作用（side effect）**其实本身是医学的一个概念，比如我们经常说吃什么药本来是为了治病，可能会产生一

  些其他的副作用，在计算机科学中，也引用了副作用的概念，表示在执行一个函数时，除了返回函数值之外，还对调用函数产生了附加的影响，比如修改了全局变量，修改参数或者改变外部的存储； 

#### **案例**

**我们来看一个对数组操作的两个函数：**

- slice：slice截取数组时不会对原数组进行任何操作,而是生成一个新的数组；

  slice就是一个纯函数，不会修改传入的参数；

- splice：splice截取数组, 会返回一个新的数组, 也会对原数组进行修改；

  splice修改掉调用的数组对象本身,

```js
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

#### 纯函数在函数式编程中的重要性

- 因为你可以安心的编写和安心的使用； 

- 你在**写的时候**保证了函数的纯度，只是单纯实现自己的业务逻辑即可，不需要关心传入的内容是如何获得的或者依赖其他的外部变量是否已经发生了修改；
- 你在**用的时候**，你确定你的输入内容不会被任意篡改，并且自己确定的输入，一定会有确定的输出； 

![image-20220711231319106](https://s2.loli.net/2022/07/11/pes8Z3TnzDLujrI.png)

### **JavaScript柯里化**

> **柯里化**也是属于**函数式编程**里面一个非常重要的概念

- 只传递给函数一部分参数来调用它，让它返回一个函数去处理剩余的参数； 

- 这个过程就称之为柯里化；

```js
function add(x, y, z) {
  return x + y + z
}

var result = add(10, 20, 30)
console.log(result)

//柯里化代码
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

#### **柯里化让函数的职责单一**

**那么为什么需要有柯里化呢？**

- 在函数式编程中，我们其实往往希望一个函数处理的问题尽可能的单一，而不是将一大堆的处理过程交给一个函数来处理； 
- 那么我们是否就可以将每次传入的参数在单一的函数中进行处理，处理完后在下一个函数中再使用处理后的结 果

```js
function add(x, y, z) {
  x = x + 2
  y = y * 2
  z = z * z
  return x + y + z
}

console.log(add(10, 20, 30))

//职责单一
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

#### **柯里化帮助我们参数逻辑复用**

```js
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

**打印日志案例**

```js
function log(date, type, message) {
  console.log(`[${date.getHours()}:${date.getMinutes()}][${type}]: [${message}]`)
}

// log(new Date(), "DEBUG", "查找到轮播图的bug")
// log(new Date(), "DEBUG", "查询菜单的bug")
// log(new Date(), "DEBUG", "查询数据的bug")

// 柯里化的优化 每次函数逻辑可以拆分
var log = date => type => message => {
  console.log(`[${date.getHours()}:${date.getMinutes()}][${type}]: [${message}]`)
}

// 如果我现在打印的都是当前时间
var nowLog = log(new Date())
nowLog("DEBUG")("查找到轮播图的bug")
nowLog("FETURE")("新增了添加用户的功能")

var nowAndDebugLog = log(new Date())("DEBUG")
nowAndDebugLog("查找到轮播图的bug")
nowAndDebugLog("查找到轮播图的bug")
nowAndDebugLog("查找到轮播图的bug")
nowAndDebugLog("查找到轮播图的bug")

var nowAndFetureLog = log(new Date())("FETURE")
nowAndFetureLog("添加新功能~")
```

**自动柯里化函数的实现**

```js
function add1(x, y, z) {
  return x + y + z
}

// 柯里化函数的实现hyCurrying
function hyCurrying(fn) {
  function curried(...args) {
    // 判断当前已经接收的参数的个数, 可以参数本身需要接受的参数是否已经一致了
    // 1.当已经传入的参数 大于等于 需要的参数时, 就执行函数
    if (args.length >= fn.length) {
      // fn(...args)
      // fn.call(this, ...args)
      return fn.apply(this, args)
    } else {
      // 没有达到个数时, 需要返回一个新的函数, 继续来接收的参数 递归
      function curried2(...args2) {
        // 接收到参数后, 需要递归调用curried来检查函数的个数是否达到
        return curried.apply(this, args.concat(args2))
      }
      return curried2
    }
  }
  return curried
}

var curryAdd = hyCurrying(add1)


console.log(curryAdd(10, 20, 30))
console.log(curryAdd(10, 20)(30))
console.log(curryAdd(10)(20)(30))
```

### **组合函数**

**组合（Compose）函数**是在JavaScript开发过程中一种对**函数的使用技巧、模式**： 

- 比如我们现在需要对某一个数据进行函数的调用，执行两个函数fn1和fn2，这两个函数是依次执行的；

- 那么如果**每次我们都需要进行两个函数的调用**，操作上就会显得重复； 

- 那么是否可以**将这两个函数组合起来，自动依次调用**呢？

- **这个过程就是对函数的组合，我们称之为 组合函数**（Compose Function）；

```js
function double(num) {
  return num * 2
}

function square(num) {
  return num ** 2
}

var count = 10
var result = square(double(count))
console.log(result)

// 实现最简单的组合函数
function composeFn(m, n) {
  return function(count) {
    return n(m(count))
  }
}

var newFn = composeFn(double, square)
console.log(newFn(10))
```

**实现组合函数**

```js
//实现组合函数
function hyCompose(...fns) {
  var length = fns.length
  //判断是否为函数
  for (var i = 0; i < length; i++) {
    if (typeof fns[i] !== 'function') {
      throw new TypeError("Expected arguments are functions")
    }
  }

  function compose(...args) {
    var index = 0
    //apply和call是为了保持this指向不变
    var result = length ? fns[index].apply(this, args): args
    //不断调用函数直到完成
    while(++index < length) {
      result = fns[index].call(this, result)
    }
    return result
  }
  return compose
}

function double(m) {
  return m * 2
}

function square(n) {
  return n ** 2
}

var newFn = hyCompose(double, square)
console.log(newFn(10))
```

## 五、**JS额外知识补充**

### **with语句**

- **with语句** 扩展一个语句的作用域链。

- 不建议使用with语句，因为它可能是混淆错误和兼容性问题的根源。

```js
"use strict";

var message = "Hello World"
// console.log(message)

// with语句: 可以形成自己的作用域
var obj = {name: "why", age: 18, message: "obj message"}

function foo() {
  function bar() {
    with(obj) {
      console.log(message)
      console.log("------")
    }
  }
  bar()
}

foo()

var info = {name: "kobe"}
with(info) {
  console.log(name)
}
```

### **eval函数**

- eval是一个特殊的函数，它可以将传入的字符串当做JavaScript代码来运行。

- 不建议在开发中使用eval： 
  - eval代码的可读性非常的差（代码的可读性是高质量代码的重要原则）；
  - eval是一个字符串，那么有可能在执行的过程中被刻意篡改，那么可能会造成被攻击的风险；
  - eval的执行必须经过JS解释器，不能被JS引擎优化；



```js
var jsString = 'var message = "Hello World"; console.log(message);'

var message = "Hello World"
console.log(message)

eval(jsString)

```

### **认识严格模式**

- 在ECMAScript5标准中，JavaScript提出了**严格模式的概念（Strict Mode）**： 
  - 严格模式很好理解，是一种具有限制性的JavaScript模式，从而使代码隐式的脱离了 ”懒散（sloppy）模式“； 
  - 支持严格模式的浏览器在检测到代码中有严格模式时，会以更加严格的方式对代码进行检测和执行； 

- 严格模式对正常的JavaScript语义进行了一些限制：
  - 严格模式通过 抛出错误 来消除一些原有的 静默（silent）错误；
  - 严格模式让JS引擎在执行代码时可以进行更多的优化（不需要对一些特殊的语法进行处理）；\
  - 严格模式禁用了在ECMAScript未来版本中可能会定义的一些语法；

**那么如何开启严格模式呢？严格模式支持粒度话的迁移：**

- 可以支持在js文件中开启严格模式；

  ```js
  "use strict"
  
  var message = "Hello World"
  console.log(message)
  
  // 静默错误
  true.foo = "abc"
  ```

- 也支持对某一个函数开启严格模式；

  ```js
  function foo() {
    "use strict";
  
    true.foo = "abc"
  }
  
  foo()
  ```

#### **严格模式限制**

这里我们来说几个严格模式下的严格语法限制：

- JavaScript被设计为新手开发者更容易上手，所以有时候本来错误语法，被认为也是可以正常被解析的；
- 但是这种方式可能给带来留下来安全隐患；

- 在严格模式下，这种失误就会被当做错误，以便可以快速的发现和修正；

**1. 无法意外的创建全局变量**

**2. 严格模式会使引起静默失败(silently fail,注:不报错也没有任何效果)的赋值操作抛出异常**

**3. 严格模式下试图删除不可删除的属性**

**4.严格模式不允许函数参数有相同的名称**

**5. 不允许0的八进制语法**

**6. 在严格模式下，不允许使用with**

**7. 在严格模式下，eval不再为上层引用变量**

**8. 严格模式下，this绑定不会默认转成对象**

```js
"use strict"

// 1. 禁止意外创建全局变量
message = "Hello World"
console.log(message)

function foo() {
  age = 20
}

foo()
console.log(age)

// 2.不允许函数有相同的参数名称
function foo(x, y, x) {
  console.log(x, y, x)
}

foo(10, 20, 30)


// 3.静默错误
true.name = "abc"
NaN = 123
var obj = {}
Object.defineProperty(obj, "name", {
  configurable: false,
  writable: false,
  value: "why"
})
console.log(obj.name)
obj.name = "kobe"

delete obj.name

// 4.不允许使用原先的八进制格式 0123
var num = 0o123 // 八进制
var num2 = 0x123 // 十六进制
var num3 = 0b100 // 二进制
console.log(num, num2, num3)

// 5.with语句不允许使用

// 6.eval函数不会向上引用变量了
var jsString = '"use strict"; var message = "Hello World"; console.log(message);'
eval(jsString)

console.log(message)
```

#### **严格模式下的this**

在严格模式下, 自执行函数(默认绑定)会指向undefined

```js
"use strict"

// 在严格模式下, 自执行函数(默认绑定)会指向undefined
// 之前编写的代码中, 自执行函数我们是没有使用过this直接去引用window
function foo() {
  console.log(this)
}

var obj = {
  name: "why",
  foo: foo
}

foo()

obj.foo()
var bar = obj.foo
bar()


// setTimeout的this
// fn.apply(this = window)
setTimeout(function() {
  console.log(this)
}, 1000);
```



## **六、面向对象**

对象是JavaScript中一个非常重要的概念，这是因为对象可以**将多个相关联的数据封装**到一起，更好的**描述一个事物**

- 用对象来描述事物，更有利于我们将现实的事物，抽离成代码中某个数据结构： 
  - 所以有一些编程语言就是纯面向对象的编程语言，比Java； 
  - 你在实现任何现实抽象时都需要先创建一个类，根据类再去创建对象；

JavaScript其实支持多种编程范式的，包括**函数式编程和面向对象编程**： 

- JavaScript中的对象被设计成一组属性的无序集合，像是一个哈希表，有key和value组成；

- key是一个标识符名称，value可以是任意类型，也可以是其他对象或者函数类型； 

- 如果值是一个函数，那么我们可以称之为是对象的方法； 

 **如何创建一个对象呢？**

-  早期使用创建对象的方式最多的是使用Object类，并且使用new关键字来创建一个对象：
  - 这是因为早期很多JavaScript开发者是从Java过来的，它们也更习惯于Java中通过new的方式创建一个对象；

- 后来很多开发者为了方便起见，都是直接通过字面量的形式来创建对象： 
  - 这种形式看起来更加的简洁，并且对象和属性之间的内聚性也更强，所以这种方式后来就流行了起来；

```js
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



### **对属性操作的控制**

对属性基本的操作

```js
var obj = {
  name: "why",
  age: 18,
};

// 获取属性
console.log(obj.name);

// 给属性赋值
obj.name = "kobe";
console.log(obj.name);

// 删除属性
delete obj.name;
console.log(obj);

// 需求: 对属性进行操作时, 进行一些限制
// 限制: 不允许某一个属性被赋值/不允许某个属性被删除/不允许某些属性在遍历时被遍历出来

// 遍历属性
for (var key in obj) {
  console.log(key);
}
```



#### **Object.defineProperty**

如果我们想要对一个属性进行比较精准的操作控制，那么我们就可以使用属性描述符。 

- 通过属性描述符可以精准的添加或修改对象的属性； 

- 属性描述符需要使用 Object.defineProperty 来对属性进行添加或者修改；

> **Object.defineProperty()** 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此
>
> 对象

可接收三个参数：

- obj要定义属性的对象；

- prop要定义或修改的属性的名称或 Symbol； 

- descriptor要定义或修改的属性描述符；

返回值：

- 被传递给函数的对象。



#### **属性描述符分类**

属性描述符的类型有两种：

- 数据属性（Data Properties）描述符（Descriptor）；

- 存取属性（Accessor访问器 Properties）描述符（Descriptor）；

![image-20220822135251265](https://s2.loli.net/2022/08/22/78v1Trz9MpStoFg.png)



##### **数据属性描述符**

**数据数据描述符有如下四个特性：**

- [[Configurable]]：表示属性是否可以通过delete删除属性，是否可以修改它的特性，或者是否可以将它修改为存取属性描述符；
  - 当我们直接在一个对象上定义某个属性时，这个属性的[[Configurable]]为true； 
  -  当我们通过属性描述符定义一个属性时，这个属性的[[Configurable]]默认为false； 

- [[Enumerable]]：表示属性是否可以通过for-in或者Object.keys()返回该属性；
  - 当我们直接在一个对象上定义某个属性时，这个属性的[[Enumerable]]为true； 
  -  当我们通过属性描述符定义一个属性时，这个属性的[[Enumerable]]默认为false； 

- [[Writable]]：表示是否可以修改属性的值；
  - 当我们直接在一个对象上定义某个属性时，这个属性的[[Writable]]为true； 
  -  当我们通过属性描述符定义一个属性时，这个属性的[[Writable]]默认为false； 
-  [[value]]：属性的value值，读取属性时会返回该值，修改属性时，会对其进行修改；
  - 默认情况下这个值是undefined；

```js
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
  // 该特殊不可删除/也不可以重新定义属性描述符
  configurable: false, // 默认值false
  // 该特殊是配置对应的属性(address)是否是可以枚举
  enumerable: true, // 默认值false
  // 该特性是属性是否是可以赋值(写入值) 
  writable: false // 默认值false
})

// 测试configurable的作用
delete obj.name
console.log(obj.name)
delete obj.address
console.log(obj.address)

Object.defineProperty(obj, "address", {
  value: "广州市",
  configurable: true
})

// 测试enumerable的作用
console.log(obj)
for (var key in obj) {
  console.log(key)
}
console.log(Object.keys(obj))

// 测试Writable的作用
obj.address = "上海市"
console.log(obj.address)
```



##### **存取属性描述符**

**数据数据描述符有如下四个特性：**

- [[Configurable]]：表示属性是否可以通过delete删除属性，是否可以修改它的特性，或者是否可以将它修改为存取属性描述符；
  - 和数据属性描述符是一致的；
  -  当我们直接在一个对象上定义某个属性时，这个属性的[[Configurable]]为true； 
  -  当我们通过属性描述符定义一个属性时，这个属性的[[Configurable]]默认为false； 

- [[Enumerable]]：表示属性是否可以通过for-in或者Object.keys()返回该属性；
  - 和数据属性描述符是一致的；
  -  当我们直接在一个对象上定义某个属性时，这个属性的[[Enumerable]]为true； 
  -  当我们通过属性描述符定义一个属性时，这个属性的[[Enumerable]]默认为false； 

- [[get]]：获取属性时会执行的函数。默认为undefined

- [[set]]：设置属性时会执行的函数。默认为undefined

```js
var obj = {
  name: "why",
  age: 18,
  _address: "北京市"
}

// 存取属性描述符
// 1.隐藏某一个私有属性被希望直接被外界使用和赋值
// 2.如果我们希望截获某一个属性它访问和设置值的过程时, 也会使用存储属性描述符
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

// 获取了一次address的值
// 北京市
// 设置了addres的值
// 获取了一次address的值
// 上海市
```



#### **同时定义多个属性**

**Object.defineProperties()** 方法直接在一个对象上定义 **多个** 新的属性或修改现有属性，并且返回该对象

```js
var obj = {
  // 私有属性(js里面是没有严格意义的私有属性)
  _age: 18,
  _eating: function() {},
  set age(value) {
    this._age = value
  },
  get age() {
    return this._age
  }
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



#### **对象方法补充**

- 获取对象的属性描述符：
  - getOwnPropertyDescriptor
  - getOwnPropertyDescriptors

- 禁止对象扩展新属性：*Object.preventExtensions*
  - 给一个对象添加新的属性会失败（在严格模式下会报错）；

- 密封对象，不允许配置和删除属性：*Object.seal*
  - 实际是调用*preventExtensions*
  - 并且将现有属性的*configurable:false*

- 冻结对象，不允许修改现有属性： *Object.freeze* 
  - 实际上是调用*seal*
  - 并且将现有属性的*writable: false*

```js
var obj = {
  // 私有属性(js里面是没有严格意义的私有属性)
  _age: 18,
  _eating: function() {}
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

// 获取某一个特性属性的属性描述符
console.log(Object.getOwnPropertyDescriptor(obj, "name"))
console.log(Object.getOwnPropertyDescriptor(obj, "age"))

// 获取对象的所有属性描述符
console.log(Object.getOwnPropertyDescriptors(obj))
```



### **创建多个对象的方案**

如果我们现在希望创建一系列的对象：比如Person对象

- 包括张三、李四、王五、李雷等等，他们的信息各不相同； 

- 那么采用什么方式来创建比较好呢？

#### 方案一：字面量创建的方式

> 缺点：重复且麻烦

```js
var p1 = {
  name: "张三",
  age: 18,
  height: 1.88,
  address: "广州市",
  eating: function() {
    console.log(this.name + "在吃东西~")
  },
  running: function() {
    console.log(this.name + "在跑步~")
  }
}

var p2 = {
  name: "李四",
  age: 20,
  height: 1.98,
  address: "北京市",
  eating: function() {
    console.log(this.name + "在吃东西~")
  },
  running: function() {
    console.log(this.name + "在跑步~")
  }
}
```

#### 方案二：工厂模式方式

> 缺点：我们在打印对象时，对象的类型都是**Object类型**

```js
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

{
  name: '张三',
  age: 18,
  height: 1.88,
  address: '广州市',
  eating: [Function (anonymous)],
  running: [Function (anonymous)]
} {
  name: '李四',
  age: 20,
  height: 1.98,
  address: '上海市',
  eating: [Function (anonymous)],
  running: [Function (anonymous)]
} {
  name: '王五',
  age: 30,
  height: 1.78,
  address: '北京市',
  eating: [Function (anonymous)],
  running: [Function (anonymous)]
}
```



#### 方案三：构造函数方式

- 我们先理解什么是构造函数？
  - 构造函数也称之为**构造器（constructor）**，通常是我们在创建对象时会调用的函数；
  - 在其他面向的编程语言里面，构造函数是存在于类中的一个方法，称之为**构造方法**；
  - 但是JavaScript中的构造函数有点不太一样；

- JavaScript中的构造函数是怎么样的？
  - 构造函数也**是一个普通的函数**，从表现形式来说，和千千万万个普通的函数没有任何区别； 
  - 那么如果这**么一个普通的函数被使用new操作符来调用了，那么这个函数就称之为是一个构造函数；**

```js
function foo() {
  console.log("foo~, 函数体代码")
}

// foo就是一个普通的函数
// foo()

// 换一种方式来调用foo函数: 通过new关键字去调用一个函数, 那么这个函数就是一个构造函数了
var f1 = new foo
console.log(f1)


// 当我们通过new去调用一个函数时, 和通过的调用到底有什么区别?
```

##### **new操作符调用的作用**

如果一个函数被使用new操作符调用了，那么它会执行如下操作：

1. 在内存中创建一个新的对象（空对象）；

2. 这个对象内部的[[prototype]]属性会被赋值为该构造函数的prototype属性；（后面详细讲）；

3. 构造函数内部的this，会指向创建出来的新对象；
4. 执行函数的内部代码（函数体代码）；

5. 如果构造函数没有返回非空对象，则返回创建出来的新对象；

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b594f7c2518847a1a04df3f3bf01c93f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)



##### 构造函数实现创建多个变量

> 这个构造函数可以确保我们的对象是有**Person的类型的**（实际是constructor的属性)
>
> 缺点：我们需要为**每个对象的函数去创建一个函数对象实例**     如每个对象p都有一个一样的eating()

```js
// 规范: 构造函数的首字母一般是大写
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
console.log(p1.__proto__ === p2.__proto__); //true
p1.eating()
p2.eating()
console.log(p1.eating() === p2.eating()); //true
```

```js
// 当创建一个新的foo对象时，会产生一个新的bar函数 十分浪费内存
function foo() {
  function bar() {

  }
  return bar
}

var fn1 = foo()
var fn2 = foo()

console.log(fn1 === fn2)
```



#### **方案四：构造函数和原型组合**

> 将重复的函数放到Person.prototype的对象上
>
> TP:先看对象原型再来看这个方案

```js
function Person(name, age, height, address) {
  this.name = name;
  this.age = age;
  this.height = height;
  this.address = address;
}

Person.prototype.eating = function () {
  console.log(this.name + "在吃东西~");
};

Person.prototype.running = function () {
  console.log(this.name + "在跑步~");
};

var p1 = new Person("why", 18, 1.88, "北京市");
var p2 = new Person("kobe", 20, 1.98, "洛杉矶市");

p1.eating();
p2.eating();

console.log(p1.eating() === p2.eating()); // true
```



### **对象的原型**

JavaScript当中每个对象都有一个特殊的内置属性 [[prototype]]，这个特殊的对象可以指向另外一个对象

- 当我们通过引用对象的属性key来获取一个value时，它会触发 [[Get]]的操作；
- 这个操作会首先检查该属性是否有对应的属性，如果有的话就使用它；
- 如果对象中没有改属性，那么会访问对象[[prototype]]内置属性指向的对象上的属性；

那么如果通过字面量直接创建一个对象，这个对象也会有这样的属性吗？如果有，应该如何获取这个属性呢？

- 答案是有的，只要是对象都会有这样的一个内置属性；

获取的方式有两种：

- 方式一：通过对象的 __ proto __ 属性可以获取到（但是这个是早期浏览器自己添加的，存在一定的兼容性问题）；

- 方式二：通过 Object.getPrototypeOf() 方法可以获取到；

```js
// 我们每个对象中都有一个 [[prototype]], 这个属性可以称之为对象的原型(隐式原型)

var obj = { name: "why" }; // [[prototype]]
var info = {}; // [[prototype]]

// 1.解释原型的概念和看一下原型
// 早期的ECMA是没有规范如何去查看 [[prototype]]

// 给对象中提供了一个属性, 可以让我们查看一下这个原型对象(浏览器提供)
// __proto__
console.log(obj.__proto__); // {}
console.log(info.__proto__); // {}
var obj = { name: "why", __proto__: {} };

// ES5之后提供的Object.getPrototypeOf
console.log(Object.getPrototypeOf(obj));

// 2.原型有什么用呢?
// 当我们从一个对象中获取某一个属性时, 它会触发 [[get]] 操作
// 1. 在当前对象中去查找对应的属性, 如果找到就直接使用
// 2. 如果没有找到, 那么会沿着它的原型去查找 [[prototype]]
// obj.age = 18
obj.__proto__.age = 18;
console.log(obj);
console.log(obj.age);
```



#### **函数的原型 prototype**

所有的函数都有一个prototype的属性!

问题：是不是因为函数是一个对象，所以它有prototype的属性呢？

- 不是的，因为它是一个函数，才有了这个特殊的属性；

- 而不是它是一个对象，所以有这个特殊的属性；

```js
var obj = {}  
console.log(obj.prototype); //undefined  obj没有prototype这个属性
```

```js
function foo() {}

// 函数也是一个对象
// console.log(foo.__proto__) // 函数作为对象来说, 它也是有[[prototype]] 隐式原型

// 函数它因为是一个函数, 所以它还会多出来一个显示原型属性: prototype
console.log(foo.prototype);

// 在new的时候函数的隐式原型会指向他的显示原型 this.__proto__ = foo.prototype
var f1 = new foo();
var f2 = new foo();

console.log(f1.__proto__ === foo.prototype);
console.log(f2.__proto__ === foo.prototype);
```



#### **new操作符**

- 在内存中创建**一个新的对象**（空对象）；

- **这个对象内部的[[prototype]]属性会被赋值为该构造函数的prototype属性**；

那么也就意味着我们通过Person构造函数创建出来的所有对象的[[prototype]]属性都指向Person.prototype

```js
function foo() {}

// 函数也是一个对象
// console.log(foo.__proto__) // 函数作为对象来说, 它也是有[[prototype]] 隐式原型

// 函数它因为是一个函数, 所以它还会多出来一个显示原型属性: prototype
console.log(foo.prototype);

// 在new的时候函数的隐式原型会指向他的显示原型 this.__proto__ = foo.prototype
var f1 = new foo();
var f2 = new foo();

console.log(f1.__proto__ === foo.prototype);
console.log(f2.__proto__ === foo.prototype);
console.log(f1.__proto__ === f2.__proto__);
```

#### **创建对象的内存表现**

![image-20220822144046982](https://s2.loli.net/2022/08/22/TgqXBU3afQe85ku.png)



**当在p1的原型对象上增加值时**

![image-20220822144320126](https://s2.loli.net/2022/08/22/DByfTN7q8IdZLGx.png)

```js
function Person() {}

var p1 = new Person();
var p2 = new Person();

// 都是为true
console.log(p1.__proto__ === Person.prototype);
console.log(p2.__proto__ === Person.prototype);

// p1.name = "why"
// p1.__proto__.name = "kobe"
// Person.prototype.name = "james"
p2.__proto__.name = "curry";

console.log(p1.name);  // curry
```



#### **constructor属性**

事实上原型对象上面是有一个属性的：constructor

- 默认情况下原型上都会添加一个属性叫做constructor，这个constructor指向当前的函数对象；

```js
function foo() {}

// 1.constructor属性
// foo.prototype这个对象原型 中有一个constructor属性
console.log(foo.prototype);
console.log(Object.getOwnPropertyDescriptors(foo.prototype));

Object.defineProperty(foo.prototype, "constructor", {
  enumerable: true,
  configurable: true,
  writable: true,
//  value: "哈哈哈哈",
});

console.log(foo.prototype);

// prototype.constructor = 构造函数本身
console.log(foo.prototype.constructor); // [Function: foo]
console.log(foo.prototype.constructor.name);

// console.log(foo.prototype.constructor.prototype.constructor.prototype.constructor)

// 2.我们也可以添加自己的属性
foo.prototype.name = "why";
foo.prototype.age = 18;
foo.prototype.height = 18;
foo.prototype.eating = function () {};

var f1 = new foo();
console.log(f1.name, f1.age);
```



#### **重写原型对象**

如果我们需要在原型上添加过多的属性，通常我们会重新整个原型对象：

```js
// 3.直接修改整个prototype对象
foo.prototype = {
  // constructor: foo,
  name: "why",
  age: 18,
  height: 1.88,
};

var f1 = new foo();

console.log(f1.name, f1.age, f1.height);

// 真实开发中我们可以通过Object.defineProperty方式添加constructor
Object.defineProperty(foo.prototype, "constructor", {
  enumerable: false,
  configurable: true,
  writable: true,
  value: foo,
});
```

前面我们说过, 每创建一个函数, 就会同时创建它的prototype对象, 这个对象也会自动获取constructor属性；

- 而我们这里相当于给prototype重新赋值了一个对象, 那么这个新对象的constructor属性, 会指向Object构造函 数, 而不是Person构造函数了
- 如果希望constructor指向Person，那么可以手动添加： 
- 默认情况下, 原生的constructor属性是不可枚举的. 
- 如果希望解决这个问题, 就可以使用我们前面介绍的Object.defineProperty()函数了

![image-20220822144853578](https://s2.loli.net/2022/08/22/AjYOSuithf5dxrR.png)



### **面向对象的特性 – 继承**

- 面向对象有三大特性：封装、继承、多态 
  - 封装：我们前面将属性和方法封装到一个类中，可以称之为封装的过程； 
  - 继承：继承是面向对象中非常重要的，不仅仅可以减少重复代码的数量，也是多态前提（纯面向对象中）；
  - 多态：不同的对象在执行时表现出不同的形态； 

- 那么继承是做什么呢？ 
  - 继承可以帮助我们将重复的代码和逻辑抽取到父类中，子类只需要直接继承过来使用即可。



#### **JavaScript原型链**

- 在真正实现继承之前，我们先来理解一个非常重要的概念：原型链。 
  - 我们知道，从一个对象上获取属性，如果在当前对象中没有获取到就会去它的原型上面获取：

```js
var obj = {
  name: "why",
  age: 18,
};

// [[get]]操作
// 1.在当前的对象中查找属性
// 2.如果没有找到, 这个时候会去原型链(__proto__)对象上查找

obj.__proto__ = {};

// 原型链
obj.__proto__.__proto__ = {};

obj.__proto__.__proto__.__proto__ = {
  address: "上海市",
};

console.log(obj.address);
```

![image-20220825113915325](https://s2.loli.net/2022/08/25/iXMt1lBaEW5dkhb.png)

#### **Object的原型**

- Object的原型的就是原型链的尽头也就是最顶层的原型

- 从Object直接创建出来的对象的原型都是 [Object: null prototype] {}
- [Object: null prototype] {} 原型有什么特殊吗？
  - 特殊一：该对象有原型属性，但是它的原型属性已经指向的是null，也就是已经是顶层原型了；
  - 特殊二：该对象上有很多默认的属性和方法；

```js
var obj = { name: "why" };

// console.log(obj.address)

// 到底是找到哪一层对象之后停止继续查找了呢?
// 字面对象obj的原型是 [Object: null prototype] {}
// [Object: null prototype] {} 就是顶层的原型
console.log(obj.__proto__); // [Object: null prototype] {}

// obj.__proto__ => [Object: null prototype] {}
console.log(obj.__proto__.__proto__); // null


```

**创建Object对象的内存图**

![image-20220825115522529](https://s2.loli.net/2022/08/25/j84N39a7uAq2HcG.png)

**原型链关系的内存图**

![image-20220825115751436](https://s2.loli.net/2022/08/25/Qri1dW6CGKEOhM9.png)

**Object是所有类的父类**

```js
function Person() {

}

// console.log(Person.prototype)
// console.log(Object.getOwnPropertyDescriptors(Person.prototype))

console.log(Person.prototype.__proto__)	// [Object: null prototype] {}
console.log(Person.prototype.__proto__.__proto__) // null
console.log(p1.__proto__.__proto__.constructor);	// [Function: Object]
```

从我们上面的Object原型我们可以得出一个结论：

- 在原型链中，原型链顶端一定是`Object.prototype`，`Function.prototype`继承它而产生。
- **一切对象**继承自`Object.prototype`，**一切函数对象**都继承自`Function.prototype`(且Function.prototype最终继承自Object.prototype)

![image-20220825114638823](https://s2.loli.net/2022/08/25/RpGMtO1Ig9qPTyK.png)

#### **为什么要有继承**

当两个类很多方法都一样时，重复写太冗余了，定义一个父类，将公共的方法都放在父类中，子类存放自己独有的方法。

```js
// Student
function Student(name, age, sno) {
  this.name = name
  this.age = age
  this.sno = sno
}

Student.prototype.running = function() {
  console.log(this.name + " running~")
}

Student.prototype.eating = function() {
  console.log(this.name + " eating~")
}

Student.prototype.studying = function() {
  console.log(this.name + " studying")
}

// Teacher
function Teacher(name, age, title) {
  this.name = name
  this.age = age
  this.title = title
}

Teacher.prototype.running = function() {
  console.log(this.name + " running~")
}

Teacher.prototype.eating = function() {
  console.log(this.name + " eating~")
}

Teacher.prototype.teaching = function() {
  console.log(this.name + " teaching")
}
```



#### **JavaScript中的继承方案**

##### 方案一：**原型链继承**

将子类的原型直接指向父类new出来的p对象，而因为p对象是父类new出来的，所以p对象的原型指向父类的原型（由上文的new操作符那一章可知）

![image-20220825121021957](https://s2.loli.net/2022/08/25/7GAyrHPKDhoJYbF.png)

**继承创建对象的内存图**

![image-20220825121156067](https://s2.loli.net/2022/08/25/TJxvZPSIcV6gFEw.png)

**弊端**

> 因为直接将子类的原型指向父类new出来的对象，所以目前有一个很大的弊端：某些属性其实是保存在p对象上的

- 第一个弊端: 打印stu对象, 继承的属性是看不到的
  - 因为打印stu对象，只会打印stu的内容，不会打印stu原型p对象的内容
- 第二个弊端: 创建出来两个stu的对象会相互影响
  - 因为这个属性会被多个对象共享，如果这个对象是一个引用类型，那么就会造成问题
- 第三个弊端: 在前面实现类的过程中都没有传递参数
  - 因为这个对象是一次性创建的（没办法定制化）；

```js
// 父类: 公共属性和方法
function Person() {
  this.name = "why"
  this.friends = []
}

Person.prototype.eating = function() {
  console.log(this.name + " eating~")
}

// 子类: 特有属性和方法
function Student() {
  this.sno = 111
}

var p = new Person()
Student.prototype = p

Student.prototype.studying = function() {
  console.log(this.name + " studying~")
}


// name/sno
var stu = new Student()

// console.log(stu.name)
// stu.eating()

// stu.studying()


// 原型链实现继承的弊端:

// 1.第一个弊端: 打印stu对象, 继承的属性是看不到的
// console.log(stu.name)

// 2.第二个弊端: 创建出来两个stu的对象
var stu1 = new Student()
var stu2 = new Student()

// 直接修改对象上的属性, 是给本对象添加了一个新属性
stu1.name = "kobe"
console.log(stu2.name)

// 获取引用, 修改引用中的值, 会相互影响
stu1.friends.push("kobe")

console.log(stu1.friends)
console.log(stu2.friends)

// 3.第三个弊端: 在前面实现类的过程中都没有传递参数
var stu3 = new Student("lilei", 112)
```

##### **方案二：借用构造函数继承**

- 为了解决原型链继承中存在的问题，开发人员提供了一种新的技术: **constructor stealing(有很多名称: 借用构造函数或者称之为经典继承或者称之为伪造对象)**

- 借用继承的做法非常简单：**在子类型构造函数的内部调用父类型构造函数**

  - 因为函数可以在任意的时刻被调用；

  - 因此通过apply()和call()方法也可以在新创建的对象上执行构造函数；

![image-20220825130302114](https://s2.loli.net/2022/08/25/domxBL9XFMH1awP.png)

**弊端：**

1. 第一个弊端: 组合继承最大的问题就是无论在什么情况下，都会调用两次父类构造函数
2. 第二个弊端: 所有的子类实例事实上会拥有两份父类的属性：一份在当前的实例自己里面(也就是person本身的)，另一份在子类对应的原型对象中(也就是person.__ proto __里面)

```js
// 父类: 公共属性和方法
function Person(name, age, friends) {
  // this = stu
  this.name = name
  this.age = age
  this.friends = friends
}

Person.prototype.eating = function() {
  console.log(this.name + " eating~")
}

// 子类: 特有属性和方法
function Student(name, age, friends, sno) {
  // 直接将this（此时this是stu对象）和对应参数传给父类，由父类进行赋值操作，但是创建的新值都是在stu对象上的
  Person.call(this, name, age, friends)
  // 相当于下面一样
  // this.name = name
  // this.age = age
  // this.friends = friends
  this.sno = 111
}

var p = new Person()
Student.prototype = p

Student.prototype.studying = function() {
  console.log(this.name + " studying~")
}


// name/sno
var stu = new Student("why", 18, ["kobe"], 111)

// 强调: 借用构造函数也是有弊端:
// 1.第一个弊端: Person函数至少被调用了两次
// 2.第二个弊端: stu的原型对象上会多出一些属性, 但是这些属性是没有存在的必要
```

##### **方案三：原型式继承**

- 原型式继承的渊源
  - 这种模式要从道格拉斯·克罗克福德（Douglas Crockford，著名的前端大师，JSON的创立者）在2006年写的 一篇文章说起: Prototypal Inheritance in JavaScript(**在JS中使用原型式继承**) 
  - 在这篇文章中，它介绍了一种继承方法，而且这种继承方法不是通过构造函数来实现的. 
  - 为了理解这种方式，我们先再次回顾一下JavaScript想实现继承的目的：**重复利用另外一个对象的属性和方法.** 
- 最终的目的：**student对象的原型指向了person对象；**

```js
// 下面3种都是原型式继承函数
function createObject2(o) {
  function Fn() {}
  Fn.prototype = o;
  var newObj = new Fn();
  return newObj;
}

function createObject1(o) {
  var newObj = {};
  Object.setPrototypeOf(newObj, o);
  return newObj;
}

// var info = createObject2(obj)
var info = Object.create(obj);
```

优点：

1. 父类方法可复用

缺点：

1. 父类的引用会被所有子类所共享
2. 子类实例不能向父类传参



**方案四：寄生式继承函数**

- 寄生式(Parasitic)继承是与原型式继承紧密相关的一种思想, 并且同样由道格拉斯·克罗克福德(Douglas Crockford)提出和推广的；

- **寄生式继承的思路是结合原型类继承和工厂模式的一种方式**；

- 即**创建一个封装继承过程的函数, 该函数在内部以某种方式来增强对象，最后再将这个对象返回**；

```js
var personObj = {
  running: function () {
    console.log("running");
  },
};

// 工厂模式 + 原型式继承
function createStudent(name) {
  var stu = Object.create(personObj);
  stu.name = name;
  stu.studying = function () {
    console.log("studying~");
  };
  return stu;
}

var stuObj = createStudent("why");
```

使用原型式继承对一个目标对象进行浅复制，增强这个浅复制的能力



##### **最终方案：寄生组合式继承**

`寄生组合式继承，实际是通过借用构造函数来继承属性，通过原型链形式来继承方法。` 实质：不必为了指定 子类的原型而调用超类构造函数 ，我们只需超类的原型副本即可——使用寄生式继承 来继承 超类原型 ，然后将结果（实例）指定给子类原型 。

![image-20220824230709474](https://s2.loli.net/2022/08/24/wgsWANkprI1DyUh.png)

```js
// 原型式继承函数
function createObject(o) {
  function Fn() {}
  Fn.prototype = o;
  return new Fn();
}

// SubType子类 SuperType父类
function inheritPrototype(SubType, SuperType) {
  // 创建一个对象，子类的原型指向这个对象并且他的原型指向父类的原型
  SubType.prototype = Object.create(SuperType.prototype);
  // 创建新的对象里面没有constructor，所以要自己创建一个
  Object.defineProperty(SubType.prototype, "constructor", {
    enumerable: false,
    configurable: true,
    writable: true,
    value: SubType,
  });
}

// 父类
function Person(name, age, friends) {
  this.name = name;
  this.age = age;
  this.friends = friends;
}

Person.prototype.running = function () {
  console.log("running~");
};

Person.prototype.eating = function () {
  console.log("eating~");
};

// 子类
function Student(name, age, friends, sno, score) {
  Person.call(this, name, age, friends);
  this.sno = sno;
  this.score = score;
}

// 调用函数
inheritPrototype(Student, Person);

// 测试
Student.prototype.studying = function () {
  console.log("studying~");
};

var stu = new Student("why", 18, ["kobe"], 111, 100);
console.log(stu);
stu.studying();
stu.running();
stu.eating();

console.log(stu.constructor.name);	// Student
console.log(stu.__proto__); // Person { studying: [Function (anonymous)] }
console.log(stu.__proto__.__proto__);	// { running: [Function (anonymous)], eating: [Function (anonymous)] }
```



### 	JS原型补充知识点

#### **对象的判断方法补充**

- **hasOwnProperty**
  - 对象是否有某一个属于自己的属性（不是在原型上的属性）

- **in/for in 操作符**
  -  判断某个属性是否在某个对象或者对象的原型上

- **instanceof**
  -  用于检测构造函数的pototype，是否出现在某个实例对象的原型链上 

- **isPrototypeOf**
  - 用于检测某个对象，是否出现在某个实例对象的原型链上

```js
var obj = {
  name: "why",
  age: 18,
};

var info = Object.create(obj, {
  address: {
    value: "北京市",
    enumerable: true,
  },
});
console.log(info.address);
console.log(info.__proto__);

// hasOwnProperty方法判断
console.log(info.hasOwnProperty("address"));
console.log(info.hasOwnProperty("name"));

// in 操作符: 不管在当前对象还是原型中返回的都是true
console.log("address" in info);
console.log("name" in info);

// for in 不管在当前对象还是原型中返回
for (var key in info) {
  console.log(key);
}
```

```js
function createObject(o) {
  function Fn() {}
  Fn.prototype = o;
  return new Fn();
}

function inheritPrototype(SubType, SuperType) {
  SubType.prototype = createObject(SuperType.prototype);
  Object.defineProperty(SubType.prototype, "constructor", {
    enumerable: false,
    configurable: true,
    writable: true,
    value: SubType,
  });
}

function Person() {}

function Student() {}
      
inheritPrototype(Student, Person);

console.log(Person.prototype.__proto__);

var stu = new Student();

// instanceof 判断对象是否出现在某个原型链中
console.log(stu instanceof Student); // true
console.log(stu instanceof Person); // true
console.log(stu instanceof Object); // true
```

```js
function Person() {}

var p = new Person();

console.log(p instanceof Person);	// true
console.log(Person.prototype.isPrototypeOf(p));	// true

// 创建一个obj对象
var obj = {
  name: "why",
  age: 18,
};

var info = Object.create(obj);

// console.log(info instanceof obj) // 报错 因为obj不是构造函数
console.log(obj.isPrototypeOf(info));	// true
```



#### **对象-函数-原型之间的关系总结**

##### 对象的原型（隐式原型）

1. 定义

JavaScript中每个对象都有一个特殊的内置原型属性 [[prototype]]，指向另外一个对象。但是通过Object.getPropertyDescriptors(obj)也无法打印出来原型属性，所以叫做*隐式原型*，并不是因为该原型不可枚举，而是隐藏起来，要专门方法来调用
![在这里插入图片描述](https://img-blog.csdnimg.cn/0f28497a75dd4c2e98b7ca87e783a2e8.png)

2. 获取

- `obj.__proto__`属性，这个是早期浏览器自己添加的，存在一定的兼容性问题
- `Object.getPrototypeOf(obj)`，通用

3. 作用（沿着原型查找属性）

当我们通过引用对象的一个key来获取value时，实际上会触发[[get]]操作，
1、检查对象上是否有该属性，有的话就使用它
2、如果没有就去对象的原型对象上找，也没有的话去原型对象的原型对象上找，直到顶层原型

```js
const obj = {name: 'xs'};
obj.__proto__.age = 18;

console.log(obj.age) //18 
console.log(obj) //{name: 'xs'} 

/*注意：
1 引用age属性，触发[[get]]操作，会先去obj的属性里找，没找到；之后去obj原型属性对应的对象上去找，找到了，返回改值
2 age不是obj的属性，是它原型对象上的属性
*/
```



##### 函数的原型（显式原型）

1. 定义

函数也是一个对象，所以**函数内部也有隐式原型_ _ proto _ _**
此外函数还特有一个显式原型属性`prototype`，指向另外一个对象
注意显式和隐式原型所指的对象是不同的。对于函数而言，我们着重讨论它的显式原型。

```javascript
function foo() {}
```

- 查看foo函数属性, 可以打印出显式原型属性prototype

```javascript
console.log(Object.getOwnPropertyDescriprors(foo))
// 结果如下
{
  length: 。。。,  // 参数的个数
  name: 。。。,  // 函数名字
  arguments: 。。。, // 函数参数
  caller: 。。。,
  prototype: { value: {}, writable: true, enumerable: false, configurable: false } 
}
```

- 函数原型属性指向一个对象，对象内部有一个constructor属性，又指向该函数
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/47bff71faf244e6680787a955fb633ea.png)

```javascript
console.log(Object.getOwnPropertyDescriprors(foo.prototype))
// 结果如下
{
  constructor: {
    value: [Function: foo], // 指回函数对象
    writable: true,        
    enumerable: false,
    configurable: true
  }
}
```

2. 获取

显式原型可以直接获取

```javascript
foo.prototype
```

3. 作用（为实例绑定共用函数）

❎注意：**在显式原型对象上添加参数，并不会被get操作得到，get依然沿着foo的隐式原型找**

```javascript
function foo(){}

foo.prototype.eat = function(){
	console.log('eat')
}

// 调用该函数
foo.eat() // 报错，找不到函数
foo.prototype.eat() // 'eat' 可以找到
```

那显式原型有什么作用呢？
✅ 当函数作为[构造函数](https://so.csdn.net/so/search?q=构造函数&spm=1001.2101.3001.7020)时，创建的实例对象的隐式原型指向该函数的显式原型

```javascript
function foo(){}
const f1 = new foo;//如果不传递参数，也可以不写小括号
const f2 = new foo;

console.log(f1.__proto__ ==== foo.prototype) // true
console.log(f1.__proto__ ==== f2.__proto__ ) // true
123456
```

因此可以在函数原型对象上绑定函数，那么构造的对象都可以共用该函数了，节省内存

```javascript
foo.prototype.eat = function() {
	console.log('eat');
}

f1.eat() // eat
f2.eat() // eat
```



##### 函数原型和对象原型的关系

```js
function foo() {
}
const f = new foo();

console.log(f.__proto__ === foo.prototype);  //ture
```

对象f是构造函数foo创建的实例对象，上一节讲到，new操作符会进行5步操作，第二步时，会将函数foo.prototype属性赋值给对象的原型属性。所以可以得出结论

> 实例对象的原型 = 构造函数的原型



##### **原型继承关系**

> 对象里面是有一个__ proto __对象: 隐式原型对象
>
> Foo是一个函数, 那么它会有一个显示原型对象: Foo.prototype
>
> Foo.prototype来自哪里?
>
> 答案: 创建了一个函数, Foo.prototype = { constructor: Foo }
>
> 
>
> Foo是一个对象, 那么它会有一个隐式原型对象: Foo.__ proto __
>
> Foo.__ proto __来自哪里?
>
> 答案: new *Function*()  Foo.__ proto __ = *Function*.prototype
>
> *Function*.prototype = { constructor: *Function* }

- 每个function函数都有它的prototype原型对象
- 刚开始函数的显示原型prototype指向一个 { constructor: Foo },当new操作符时 Foo.__ proto __ = *Function*.prototype

![image-20220825160051760](https://s2.loli.net/2022/08/25/aYpKlRnQ3uxyg2B.png)

**原型继承关系**

![image-20220824232955325](https://s2.loli.net/2022/08/24/2Uu9jKRJScpNM5P.png)



```js
var obj = {
  name: "why",
};

console.log(obj.__proto__);

// 对象里面是有一个__proto__对象: 隐式原型对象

// Foo是一个函数, 那么它会有一个显示原型对象: Foo.prototype
// Foo.prototype来自哪里?
// 答案: 创建了一个函数, Foo.prototype = { constructor: Foo }

// Foo是一个对象, 那么它会有一个隐式原型对象: Foo.__proto__
// Foo.__proto__来自哪里?
// 答案: new Function()  Foo.__proto__ = Function.prototype
// Function.prototype = { constructor: Function }

// var Foo = new Function()
// function Foo() {}

function Foo() {}

console.log(Foo.prototype === Foo.__proto__);
console.log(Foo.prototype.constructor);
console.log(Foo.__proto__.constructor);

var foo1 = new Foo();
var obj1 = new Object();

console.log(Object.getOwnPropertyDescriptors(Function.__proto__));
```



## **七、ES6**

### **认识class定义类**

- 我们会发现，按照前面的构造函数形式创建**类**，不仅仅和编写普通的函数过于相似，而且代码并不容易理解。
  - 在ES6（ECMAScript2015）新的标准中使用了class关键字来直接定义类；
  - 但是类本质上依然是前面所讲的构造函数、原型链的语法糖而已；

- 所以学好了前面的构造函数、原型链更有利于我们理解类的概念和继承关系；

- 那么，如何使用class来定义一个类呢？
  - 可以使用两种方式来声明类：类声明和类表达式；

```js
// 类的声明
class Person {}

// babel

// 类的表达式
var Animal = class {
}
```

#### **类和构造函数的异同**

我们来研究一下类的一些特性：

- 你会发现它和我们的构造函数的特性其实是一致的；

```js
// 研究一下类的特性
console.log(Person.prototype);  // {}
console.log(Person.prototype.__proto__); // [Object: null prototype] {}
console.log(Person.prototype.constructor);  // [class Person]
console.log(typeof Person); // function

var p = new Person();
console.log(p.__proto__ === Person.prototype); // true
```

#### **类的构造函数**

如果我们希望在创建对象的时候给类传递一些参数，这个时候应该如何做呢？

- **每个类都可以有一个自己的构造函数**（方法），这个方法的名称是固定的constructor； 

- 当我们通过new操作符，操作一个类的时候会调用这个类的构造函数constructor； 

- **每个类只能有一个构造函数**，如果包含多个构造函数，那么会抛出异常；

当我们通过new关键字操作类的时候，会调用这个constructor函数，并且执行如下操作：

1. 在内存中创建一个新的对象（空对象）；

2. 这个对象内部的[[prototype]]属性会被赋值为该类的prototype属性；

3. 构造函数内部的this，会指向创建出来的新对象；

4. 执行构造函数的内部代码（函数体代码）；

5. 如果构造函数没有返回非空对象，则返回创建出来的新对象；

```js
class Person {
  // 类的构造方法
  // 注意: 一个类只能有一个构造函数
  // 1.在内存中创建一个对象 moni = {}
  // 2.将类的原型prototype赋值给创建出来的对象 moni.__proto__ = Person.prototype
  // 3.将对象赋值给函数的this: new绑定 this = moni
  // 4.执行函数体中的代码
  // 5.自动返回创建出来的对象
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

var p1 = new Person("why", 18);
var p2 = new Person("kobe", 30);
console.log(p1, p2);
```

#### **类的实例方法**

在上面我们定义的属性都是直接放到了this上，也就意味着它是放到了创建出来的新对象中：

- 在前面我们说过对于实例的方法，我们是希望放到原型上的，这样可以被多个实例来共享；

- 这个时候我们可以直接在类中定义；

```js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
    this._address = "广州市";
  }

  // 普通的实例方法
  // 创建出来的对象进行访问
  // var p = new Person()
  // p.eating()
  eating() {
    console.log(this.name + " eating~");
  }

  running() {
    console.log(this.name + " running~");
  }
}  
```



#### **类的访问器方法**

我们之前讲对象的属性描述符时有讲过对象可以添加setter和getter函数的，那么类也是可以的：

```js
class Person {
// 类的访问器方法
  get address() {
    console.log("拦截访问操作");
    return this._address;
  }

  set address(newAddress) {
    console.log("拦截设置操作");
    this._address = newAddress;
  }
}
```

#### **类的静态方法**

静态方法通常用于定义直接使用类来执行的方法，不需要有类的实例，使用static关键字来定义：

```js
class Person {
// 类的静态方法(类方法)
  // Person.createPerson()
  static randomPerson() {
    var nameIndex = Math.floor(Math.random() * names.length);
    var name = names[nameIndex];
    var age = Math.floor(Math.random() * 100);
    return new Person(name, age);
  }
}
```



### **ES6类的继承 - extends**

在ES6中新增了使用extends关键字，可以方便的帮助我们实现继承：

```js
class Person {
}
class Student extends Person {
}
```

#### **super关键字**

我们会发现在上面的代码中我使用了一个super关键字，这个super关键字有不同的使用方式：

- 注意：在子（派生）类的构造函数中使用this或者返回默认对象之前，必须先通过super调用父类的构造函数！ 

- super的使用位置有三个：子类的构造函数、实例方法、静态方法；

```js
// 父类
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  running() {
    console.log(this.name + " running~");
  }

  eating() {
    console.log(this.name + " eating~");
  }

  personMethod() {
    console.log("处理逻辑1");
    console.log("处理逻辑2");
    console.log("处理逻辑3");
  }

  static staticMethod() {
    console.log("PersonStaticMethod");
  }
}

// Student称之为子类(派生类)
class Student extends Person {
  // JS引擎在解析子类的时候就有要求, 如果我们有实现继承
  // 那么子类的构造方法中, 在使用this之前
  constructor(name, age, sno) {
    super(name, age);
    this.sno = sno;
  }

  studying() {
    console.log(this.name + " studying~");
  }

  // 类对父类的方法的重写
  running() {
    console.log("student " + this.name + " running");
  }

  // 重写personMethod方法
  personMethod() {
    // 复用父类中的处理逻辑
    super.personMethod();

    console.log("处理逻辑4");
    console.log("处理逻辑5");
    console.log("处理逻辑6");
  }

  // 重写静态方法
  static staticMethod() {
    super.staticMethod();
    console.log("StudentStaticMethod");
  }
}

var stu = new Student("why", 18, 111);
console.log(stu);
```

### **继承内置类**

我们也可以让我们的类继承自内置类，比如Array：

```js
class HYArray extends Array {
  firstItem() {
    return this[0]
  }

  lastItem() {
    return this[this.length-1]
  }
}

var arr = new HYArray(1, 2, 3)
console.log(arr.firstItem())
console.log(arr.lastItem())
```

### **类的混入mixin**

JavaScript的类只支持单继承：也就是只能有一个父类

- 那么在开发中我们我们需要在一个类中添加更多相似的功能时，应该如何来做呢？

- 这个时候我们可以使用混入（mixin）；

```js
class Person {}

function mixinRunner(BaseClass) {
  class NewClass extends BaseClass {
    running() {
      console.log("running~");
    }
  }
  return NewClass;
}

function mixinEater(BaseClass) {
  return class extends BaseClass {
    eating() {
      console.log("eating~");
    }
  };
}

// 在JS中类只能有一个父类: 单继承
class Student extends Person {}

var NewStudent = mixinEater(mixinRunner(Student));

var ns = new NewStudent();
ns.running();
ns.eating();
```



###  **JavaScript中的多态**

- 面向对象的三大特性：封装、继承、多态。
  - 前面两个我们都已经详细解析过了，接下来我们讨论一下JavaScript的多态。

- JavaScript有多态吗？
  - 维基百科对多态的定义：**多态**（英语：polymorphism）指为不同数据类型的实体提供统一的接口，或使用一个单一的符号来表示多个不同的类型。
  - 非常的抽象，个人的总结：不同的数据类型进行同一个操作，表现出不同的行为，就是多态的体现。

- 那么从上面的定义来看，JavaScript是一定存在多态的。

#### 传统面向对象的多态

> TS代码

```tsx
// 传统的面向对象多态是有三个前提:
// 1> 必须有继承(是多态的前提)
// 2> 必须有重写(子类重写父类的方法)
// 3> 必须有父类引用指向子类对象

// Shape形状
class Shape {
  getArea() {}
}

class Rectangle extends Shape {
  getArea() {
    return 100;
  }
}

class Circle extends Shape {
  getArea() {
    return 200;
  }
}

var r = new Rectangle();
var c = new Circle();

// 多态: 当对不同的数据类型执行同一个操作时, 如果表现出来的行为(形态)不一样, 那么就是多态的体现.
function calcArea(shape: Shape) {
  console.log(shape.getArea());
}

calcArea(r);
calcArea(c);

export {};
```

#### JS中多态的体现

```js
// 多态: 当对不同的数据类型执行同一个操作时, 如果表现出来的行为(形态)不一样, 那么就是多态的体现.
function calcArea(foo) {
  console.log(foo.getArea());
}

var obj1 = {
  name: "why",
  getArea: function () {
    return 1000;
  },
};

class Person {
  getArea() {
    return 100;
  }
}

var p = new Person();

calcArea(obj1);
calcArea(p);

// 也是多态的体现
function sum(m, n) {
  return m + n;
}

console.log(sum(20, 30));
console.log(sum("abc", "cba"));
```



### **字面量的增强**

- ES6中对 **对象字面量** 进行了增强，称之为 Enhanced object literals（增强对象字面量）。

- 字面量的增强主要包括下面几部分：
  - 属性的简写：**Property Shorthand**
  - 方法的简写：**Method Shorthand**
  - 计算属性名：**Computed Property Names**

```js
var name = "why";
var age = 18;

var obj = {
  // 1.property shorthand(属性的简写)
  name,
  age,

  // 2.method shorthand(方法的简写)
  foo: function () {
    console.log(this);
  },
  bar() {
    console.log(this);
  },
  baz: () => {
    console.log(this);
  },

  // 3.computed property name(计算属性名)
  [name + 123]: "hehehehe",
};

obj.baz();
obj.bar();
obj.foo();

// obj[name + 123] = "hahaha"
console.log(obj);
```



### **解构Destructuring**

- ES6中新增了一个从数组或对象中方便获取数据的方法，称之为解构Destructuring。 

- 我们可以划分为：数组的解构和对象的解构。

- 数组的解构： 基本解构过程
  - 顺序解构
  - 解构出数组
  - 默认值

- 对象的解构：
  - 基本解构过程
  - 任意顺序
  - 重命名
  - 默认值

```js
var names = ["abc", "cba", "nba"];
// var item1 = names[0]
// var item2 = names[1]
// var item3 = names[2]

// 对数组的解构: []
var [item1, item2, item3] = names;
console.log(item1, item2, item3);

// 解构后面的元素
var [, , itemz] = names;
console.log(itemz);

// 解构出一个元素,后面的元素放到一个新数组中
var [itemx, ...newNames] = names;
console.log(itemx, newNames);

// 解构的默认值
var [itema, itemb, itemc, itemd = "aaa"] = names;
console.log(itemd);

var obj = {
  name: "why",
  age: 18,
  height: 1.88,
};

// 对象的解构: {}
var { name, age, height } = obj;
console.log(name, age, height);

var { age } = obj;
console.log(age);

var { name: newName } = obj;
console.log(newName);

var { address: newAddress = "广州市" } = obj;
console.log(newAddress);

function foo(info) {
  console.log(info.name, info.age);
}

foo(obj);

function bar({ name, age }) {
  console.log(name, age);
}

bar(obj);
```

#### **解构的应用场景**

- 解构目前在开发中使用是非常多的：
  - 比如在开发中拿到一个变量时，自动对其进行解构使用；
  - 比如对函数的参数进行解构

![image-20220828002543486](https://s2.loli.net/2022/08/28/3Lf8UH1AQaoR9nF.png)

![image-20220828002549635](https://s2.loli.net/2022/08/28/JtGWyBDTc74hrK9.png)



### **let/const基本使用**

- 在ES5中我们声明变量都是使用的var关键字，从ES6开始新增了两个关键字可以声明变量：let、const
  - let、const在其他编程语言中都是有的，所以也并不是新鲜的关键字；
  - 但是let、const确确实实给JavaScript带来一些不一样的东西；

- let关键字：
  - 从直观的角度来说，let和var是没有太大的区别的，都是用于声明一个变量 

- const关键字：
  - const关键字是constant的单词的缩写，表示常量、衡量的意思； 
  - 它表示保存的数据一旦被赋值，就不能被修改； 
  - 但是如果赋值的是引用类型，那么可以通过引用找到对应的对象，修改对象的内容；

- let、const不允许重复声明变量

```js
var foo = "foo"
let bar = "bar"

// const constant(常量/衡量)
const name = "abc"
name = "cba"

// 注意事项一: const本质上是传递的值不可以修改
// 但是如果传递的是一个引用类型(内存地址), 可以通过引用找到对应的对象, 去修改对象内部的属性, 这个是可以的
const obj = {
  foo: "foo"
}

// obj = {}
obj.foo = "aaa"
console.log(obj.foo)

// 注意事项二: 通过let/const定义的变量名是不可以重复定义
var foo = "abc"
var foo = "cba"
// SyntaxError: Identifier 'foo' has already been declared

console.log(foo);
```

#### **let/const作用域提升**

- let、const和var的另一个重要区别是作用域提升： 
  - 我们知道var声明的变量是会进行作用域提升的；
  - 但是如果我们使用let声明的变量，在声明之前访问会报错； 

- 那么是不是意味着foo变量只有在代码执行阶段才会创建的呢？
  - 事实上并不是这样的，我们可以看一下ECMA262对let和const的描述； 
  - **这些变量会被创建在包含他们的词法环境被实例化时，但是是不可以访问它们的，直到词法绑定被求值；**

```js
// console.log(foo)
// var foo = "foo"

// Reference(引用)Error: Cannot access 'foo' before initialization(初始化)
// let/const他们是没有作用域提升
// foo被创建出来了, 但是不能被访问
// 作用域提升: 能提前被访问
console.log(foo);
let foo = "foo";
```

- 从上面我们可以看出，在执行上下文的词法环境创建出来的时候，变量事实上已经被创建了，只是这个变量是不能被访问的。
  - 那么变量已经有了，但是不能被访问，是不是一种作用域的提升呢？

- 事实上维基百科并没有对作用域提升有严格的概念解释，那么我们自己从字面量上理解；
  - **作用域提升：**在声明变量的作用域中，如果这个变量可以在声明之前被访问，那么我们可以称之为作用域提升； 
  - 在这里，它**虽然被创建出来了**，但是**不能被访问**，我认为**不能称之为作用域提升**； 

- 所以我的观点是let、const没有进行作用域提升，但是会在解析阶段被创建出来



#### **Window对象添加属性**

- 我们知道，在全局通过var来声明一个变量，事实上会在window上添加一个属性：
  - 但是let、const是不会给window上添加任何属性的。

- 那么我们可能会想这个变量是保存在哪里呢？

- 我们先回顾一下最新的ECMA标准中对执行上下文的描述

![image-20220828003406422](https://s2.loli.net/2022/08/28/mNjaF4qgI5XsB2n.png)

**变量被保存到VariableMap中**

- 也就是说我们声明的变量和环境记录是被添加到变量环境中的：

  - 但是标准有没有规定这个对象是window对象或者其他对象呢？

  - 其实并没有，那么JS引擎在解析的时候，其实会有自己的实现；

  - 比如v8中其实是通过VariableMap的一个hashmap来实现它们的存储的。

  - 那么window对象呢？而window对象是早期的GO对象，在最新的实现中其实是浏览器添加的全局对象，并且一直保持了window和var之间值的相等性；

![image-20220828003522843](https://s2.loli.net/2022/08/28/ClDUrtR19u8hzLk.png)

![image-20220827170714168](https://s2.loli.net/2022/08/27/IHvSgupqP9Ezf1n.png)

#### **var的块级作用域**

- 在我们前面的学习中，JavaScript只会形成两个作用域：全局作用域和函数作用域。 

![image-20220828003738145](https://s2.loli.net/2022/08/28/drjwJsOm5nT4fuB.png)

- ES5中放到一个代码中定义的变量，外面是可以访问的：

![image-20220828003751970](http://img.roydust.top/img/image-20220828003751970.png)

```js
// 声明对象的字面量
var obj = {
  name: "why",
};

// ES5中没有块级作用域
// 块代码(block code)
{
  // 声明一个变量
  var foo = "foo";
}

// console.log(foo);

// 在ES5中只有两个东西会形成作用域
// 1.全局作用域
// 2.函数作用域
function foo() {
  var bar = "bar";
}

console.log(bar);

function foo() {
  function demo() {}
}
```

#### **let/const的块级作用域**

- 在ES6中新增了块级作用域，并且通过let、const、function、class声明的标识符是具备块级作用域的限制的：
- 但是我们会发现函数拥有块级作用域，但是外面依然是可以访问的：
  - 这是因为引擎会对函数的声明进行特殊的处理，允许像var那样进行提升；

```js
// ES6的代码块级作用域
// 对let/const/function/class声明的类型是有效
{
  let foo = "why";
  function demo() {
    console.log("demo function");
  }
  class Person {}
}

// console.log(foo) // foo is not defined
// 不同的浏览器有不同实现的(大部分浏览器为了兼容以前的代码, 让function是没有块级作用域)
// demo()
var p = new Person(); // Person is not defined
```

**if-switch-for块级代码作用域**

```js
{
}

// if语句的代码就是块级作用域
if (true) {
  var foo = "foo";
  let bar = "bar";
}

console.log(foo);
console.log(bar);

// switch语句的代码也是块级作用域
var color = "red";

switch (color) {
  case "red":
    var foo = "foo";
    let bar = "bar";
}

console.log(foo);
console.log(bar);

// for语句的代码也是块级作用域
for (var i = 0; i < 10; i++) {
  console.log("Hello World" + i);
}

// console.log(i)

for (let i = 0; i < 10; i++) {}

console.log(i);
```



#### **块级作用域的应用**

实际的案例：获取多个按钮监听点击

```html
  <button>按钮1</button>
  <button>按钮2</button>
  <button>按钮3</button>
  <button>按钮4</button>
```



```js
const btns = document.getElementsByTagName("button");

// ES5对for循环的操作
for (var i = 0; i < btns.length; i++) {
  立即执行函数(function (n) {
    btns[i].onclick = function () {
      console.log("第" + n + "个按钮被点击");
    };
  })(i);
}

// console.log(i)

for (let i = 0; i < btns.length; i++) {
  btns[i].onclick = function () {
    console.log("第" + i + "个按钮被点击");
  };
}

// console.log(i)

```

#### **暂时性死区**

在ES6中，我们还有一个概念称之为暂时性死区：

- 它表达的意思是在一个代码中，使用let、const声明的变量，在声明之前，变量都是不可以访问的；

- 我们将这种现象称之为 temporal dead zone（暂时性死区，TDZ）；

```js
var foo = "foo";

// if (true) {
//   console.log(foo)
//   暂时性死区：在let/const声明变量前不能访问
//   let foo = "abc"
// }

function bar() {
  console.log(foo);
  let foo = "abc";
}

bar();

var name1 = "abc";
let name2 = "cba";
const name3 = "nba";

// 构建工具的基础上创建项目\开发项目 webpack/vite/rollup
// babel
// ES6 -> ES5

const info = { name: "why" };

info = { name: "kobe" };

```



#### **var、let、const的选择**

- 那么在开发中，我们到底应该选择使用哪一种方式来定义我们的变量呢？

- 对于var的使用：
  - 我们需要明白一个事实，var所表现出来的特殊性：比如作用域提升、window全局对象、没有块级作用域等都是一些历史遗留问题；
  - 其实是JavaScript在设计之初的一种语言缺陷；
  - 当然目前市场上也在利用这种缺陷出一系列的面试题，来考察大家对JavaScript语言本身以及底层的理解；
  - 但是在实际工作中，我们可以使用最新的规范来编写，也就是不再使用var来定义变量了；

- 对于let、const： 
  - 对于let和const来说，是目前开发中推荐使用的；
  - 我们会有限推荐使用const，这样可以保证数据的安全性不会被随意的篡改；
  - 只有当我们明确知道一个变量后续会需要被重新赋值时，这个时候再使用let； 
  - 这种在很多其他语言里面也都是一种约定俗成的规范，尽量我们也遵守这种规范；

```js
var foo = "foo";

// if (true) {
//   console.log(foo)

//   let foo = "abc"
// }

function bar() {
  console.log(foo);

  let foo = "abc";
}

bar();

var name1 = "abc";
let name2 = "cba";
const name3 = "nba";

// 构建工具的基础上创建项目\开发项目 webpack/vite/rollup
// babel
// ES6 -> ES5

const info = { name: "why" };

info = { name: "kobe" };
```



### **字符串模板基本使用**

- 在ES6之前，如果我们想要将字符串和一些动态的变量（标识符）拼接到一起，是非常麻烦和丑陋的（ugly）。 

- ES6允许我们使用字符串模板来嵌入JS的变量或者表达式来进行拼接： 
  - 首先，我们会使用 **``** 符号来编写字符串，称之为模板字符串； 
  - 其次，在模板字符串中，我们可以通过 **${expression}** 来嵌入动态的内容；

```js
// ES6之前拼接字符串和其他标识符
const name = "why";
const age = 18;
const height = 1.88;

// console.log("my name is " + name + ", age is " + age + ", height is " + height)

// ES6提供模板字符串 ``
const message = `my name is ${name}, age is ${age}, height is ${height}`;
console.log(message);

const info = `age double is ${age * 2}`;
console.log(info);

function doubleAge() {
  return age * 2;
}

const info2 = `double age is ${doubleAge()}`;
console.log(info2);
```

#### **标签模板字符串使用**

- 模板字符串还有另外一种用法：标签模板字符串（Tagged Template Literals）。

- 我们一起来看一个普通的JavaScript的函数：

- 如果我们使用标签模板字符串，并且在调用的时候插入其他的变量：

  - 模板字符串被拆分了；

  - 第一个元素是数组，是被模块字符串拆分的字符串组合；

  - 后面的元素是一个个模块字符串传入的内容；

```js
// 第一个参数依然是模块字符串中整个字符串, 只是被切成多块,放到了一个数组中
// 第二个参数是模块字符串中, 第一个 ${}
function foo(m, n, x) {
  console.log(m, n, x, "---------");
}

// foo("Hello", "World")

// 另外调用函数的方式: 标签模块字符串
// foo``

// foo`Hello World`
const name = "why";
const age = 18;

foo`Hello${name}Wo${age}rld`;	// 输出[ 'Hello', 'Wo', 'rld' ] why 18 ---------
```



### **函数的默认参数**

- 在ES6之前，我们编写的函数参数是没有默认值的，所以我们在编写函数时，如果有下面的需求： 

  - 传入了参数，那么使用传入的参数；

  - 没有传入参数，那么使用一个默认值

- 而在ES6中，我们**允许给函数一个默认值**：

- 另外**参数的默认值我们通常会将其放到最后**（在很多语言中，如果不放到最后其实会报错的）： 
  - 但是JavaScript允许不将其放到最后，但是意味着还是会按照顺序来匹配； 

- 另外默认值会改变函数的length的个数，默认值以及后面的参数都不计算在length之内了。

```js
// ES5以及之前给参数默认值
/**
 * 缺点:
 *  1.写起来很麻烦, 并且代码的阅读性是比较差
 *  2.这种写法是有bug
 */
// function foo(m, n) {
//   m = m || "aaa"
//   n = n || "bbb"

//   console.log(m, n)
// }

// 1.ES6可以给函数参数提供默认值
function foo(m = "aaa", n = "bbb") {
  console.log(m, n);
}

// foo()
foo(0, "");

// 2.对象参数和默认值以及解构
function printInfo({ name, age } = { name: "why", age: 18 }) {
  console.log(name, age);
}

printInfo({ name: "kobe", age: 40 });

// 另外一种写法
function printInfo1({ name = "why", age = 18 } = {}) {
  console.log(name, age);
}

printInfo1();

// 3.有默认值的形参最好放到最后
function bar(x, y, z = 30) {
  console.log(x, y, z);
}

// bar(10, 20)
bar(undefined, 10, 20);

// 4.会影响有默认值的函数的length属性
function baz(x, y, n = 30, z, m) {
  console.log(x, y, z, m, n);
}

console.log(baz.length); // 2
```



### **函数的剩余参数**

- **ES6中引用了rest parameter，可以将不定数量的参数放入到一个数组中**：
  - 如果**最后一个参数是 ... 为前缀的**，那么它会将剩余的参数放到该参数中，并且作为一个数组； 

- **那么剩余参数和arguments有什么区别呢？**
  - **剩余参数只包含那些没有对应形参的实参，而 arguments 对象包含了传给函数的所有实参；** 
  - arguments对象**不是一个真正的数组**，而rest参数是一个真正的数组，可以进行数组的所有操作；
  - **arguments是早期的ECMAScript中为了方便去获取所有的参数提供的一个数据结构，而rest参数是ES6中提供并且希望以此来替代arguments的；** 

- 剩余参数必须放到最后一个位置，否则会报错

```js
function foo(m, n, ...args) {
  console.log(m, n);
  console.log(args);

  console.log(arguments);
}

foo(20, 30, 40, 50, 60);

// rest paramaters必须放到最后
// Rest parameter must be last formal parameter
```



### **函数箭头函数的补充**

- 箭头函数是没有显式原型的，所以不能作为构造函数，使用new来创建对象；

```js
// function foo() {

// }

// console.log(foo.prototype)
// const f = new foo()
// f.__proto__ = foo.prototype

var bar = () => {
  console.log(this, arguments);
};
// 箭头函数没有显示原型
// undefined
console.log(bar.prototype);

// TypeError: bar is not a constructor
const b = new bar();
```



### **展开语法**

- **展开语法(Spread syntax)**： 
  - 可以在函数调用/数组构造时，将数组表达式或者string在语法层面展开；
  - 还可以在构造字面量对象时, 将对象表达式按key-value的方式展开；

- 展开语法的场景： 
  - 在函数调用时使用；
  - 在数组构造时使用；
  - 在构建对象字面量时，也可以使用展开运算符，这个是在ES2018（ES9）中添加的新特性；

- 注意：展开运算符其实是一种浅拷贝；

```js
const names = ["abc", "cba", "nba"];
const name = "why";
const info = { name: "why", age: 18 };

// 1.函数调用时
function foo(x, y, z) {
  console.log(x, y, z);
}

// foo.apply(null, names)
foo(...names);
foo(...name);

// 2.构造数组时
const newNames = [...names, ...name];
console.log(newNames);

// 3.构建对象字面量时ES2018(ES9)
const obj = { ...info, address: "广州市", ...names };
console.log(obj);

```



### **进制的表示**

- 在ES6中规范了二进制和八进制的写法： 

- 另外在ES2021新增特性：数字过长时，可以使用_作为连接符

```js
const num1 = 100; // 十进制

// b -> binary
const num2 = 0b100; // 二进制
// o -> octonary
const num3 = 0o100; // 八进制
// x -> hexadecimal
const num4 = 0x100; // 十六进制

console.log(num1, num2, num3, num4);

// 大的数值的连接符(ES2021 ES12)
const num = 10_000_000_000_000_000;
console.log(num);
```



### **Symbol的基本使用**

- Symbol是什么呢？Symbol是ES6中新增的一个基本数据类型，翻译为符号。

- **那么为什么需要Symbol呢？**
  - 在ES6之前，**对象的属性名都是字符串形式，那么很容易造成属性名的冲突；** 
  -  比如原来有一个对象，**我们希望在其中添加一个新的属性和值，但是我们在不确定它原来内部有什么内容的情况下，很容易造成冲突，从而覆盖掉它内部的某个属性；** 
  -  比如我们前面在讲apply、call、bind实现时，我们有给其中添加一个fn属性，那么如果它内部原来已经有了fn属性了呢？
  -  比如开发中我们使用混入，那么混入中出现了同名的属性，必然有一个会被覆盖掉；

- Symbol就是为了解决上面的问题，用来**生成一个独一无二的值**。 
  - Symbol值是通过Symbol函数来生成的，生成后可以作为属性名； 
  - 也就是在ES6中，对象的属性名可以使用字符串，也可以使用Symbol值； 

- Symbol即使多次创建值，它们也是不同的：Symbol函数执行后每次创建出来的值都是独一无二的；

- **我们也可以在创建Symbol值的时候传入一个描述description**：这个是ES2019（ES10）新增的特性；

```js
// 1.ES6之前, 对象的属性名(key)
var obj = {
  name: "why",
  friend: { name: "kobe" },
  age: 18,
};

obj["newName"] = "james";
console.log(obj);

// 2.ES6中Symbol的基本使用
const s1 = Symbol();
const s2 = Symbol();

console.log(s1 === s2);

// ES2019(ES10)中, Symbol还有一个描述(description)
const s3 = Symbol("aaa");
console.log(s3.description);

// 3.Symbol值作为key
// 3.1.在定义对象字面量时使用
const obj = {
  [s1]: "abc",
  [s2]: "cba",
};
console.log(obj);

// 3.2.新增属性
obj[s3] = "nba";

// 3.3.Object.defineProperty方式
const s4 = Symbol();
Object.defineProperty(obj, s4, {
  enumerable: true,
  configurable: true,
  writable: true,
  value: "mba",
});

console.log(obj[s1], obj[s2], obj[s3], obj[s4]);
// 注意: 不能通过语法获取
// console.log(obj.s1)

// 4.使用Symbol作为key的属性名,在遍历/Object.keys等中是获取不到这些Symbol值
// 需要Object.getOwnPropertySymbols来获取所有Symbol的key
console.log(Object.keys(obj));
console.log(Object.getOwnPropertyNames(obj));
console.log(Object.getOwnPropertySymbols(obj));
const sKeys = Object.getOwnPropertySymbols(obj);
for (const sKey of sKeys) {
  console.log(obj[sKey]);
}

// 5.相同值的Symbol Symbol.for(key)/Symbol.keyFor(symbol)
const sa = Symbol.for("aaa");
const sb = Symbol.for("aaa");
console.log(sa === sb);	// true

const key = Symbol.keyFor(sa);
console.log(key);	// abc
const sc = Symbol.for(key);
console.log(sa === sc); // true
```



### **Set和WeakSet的基本使用**

#### Set

- 在ES6之前，我们存储数据的结构主要有两种：数组、对象。
  - 在ES6中新增了另外两种数据结构：Set、Map，以及它们的另外形式WeakSet、WeakMap。 

- Set是一个新增的数据结构，可以用来保存数据，类似于数组，但是和数组的区别是**元素不能重复**。 
  - 创建Set我们需要通过Set构造函数（暂时没有字面量创建的方式）： 

- 我们可以发现Set中存放的元素是不会重复的，那么Set有一个非常常用的功能就是给数组去重。

```js
// 10, 20, 40, 333
// 1.创建Set结构
const set = new Set();
set.add(10);
set.add(20);
set.add(40);
set.add(333);

set.add(10);

// 2.添加对象时特别注意:
set.add({});
set.add({});

const obj = {};
set.add(obj);
set.add(obj);

// console.log(set)

// 3.对数组去重(去除重复的元素)
const arr = [33, 10, 26, 30, 33, 26];
// const newArr = []
// for (const item of arr) {
//   if (newArr.indexOf(item) !== -1) {
//     newArr.push(item)
//   }
// }

const arrSet = new Set(arr);
// const newArr = Array.from(arrSet)
// const newArr = [...arrSet]
// console.log(newArr)

// 4.size属性
console.log(arrSet.size);

// 5.Set的方法
// add
arrSet.add(100);
console.log(arrSet);

// delete
arrSet.delete(33);
console.log(arrSet);

// has
console.log(arrSet.has(100));

// clear
// arrSet.clear()
console.log(arrSet);

// 6.对Set进行遍历
arrSet.forEach((item) => {
  console.log(item);
});

for (const item of arrSet) {
  console.log(item);
}
```

- Set常见的属性：
  - size：返回Set中元素的个数；

- Set常用的方法：
  - add(value)：添加某个元素，返回Set对象本身； 
  - delete(value)：从set中删除和这个值相等的元素，返回boolean类型；
  - has(value)：判断set中是否存在某个元素，返回boolean类型；
  - clear()：清空set中所有的元素，没有返回值；
  - forEach(callback, [, thisArg])：通过forEach遍历set； 

- 另外Set是支持for of的遍历的。

#### **WeakSet(少用)**

**和Set有什么区别呢？**

- 区别一：WeakSet中只能存放对象类型，不能存放基本数据类型；

- 区别二：WeakSet对元素的引用是弱引用，如果没有其他引用对某个对象进行引用，那么GC可以对该对象进行回收；

WeakSet常见的方法： 

- add(value)：添加某个元素，返回WeakSet对象本身；

- delete(value)：从WeakSet中删除和这个值相等的元素，返回boolean类型；

- has(value)：判断WeakSet中是否存在某个元素，返回boolean类型；

WeakSet不能遍历

- 因为WeakSet只是对对象的弱引用，如果我们遍历获取到其中的元素，那么有可能造成对象不能正常的销毁。 

- 所以存储到WeakSet中的对象是没办法获取的；

```js
const weakSet = new WeakSet();

// 1.区别一: 只能存放对象类型
// TypeError: Invalid value used in weak set
// weakSet.add(10)

// 2.区别二: 对对象是一个弱引用
let obj = {
  name: "why",
};

// weakSet.add(obj)

const set = new Set();
// 建立的是强引用
set.add(obj);

// 建立的是弱引用
weakSet.add(obj);

// 3.WeakSet的应用场景
const personSet = new WeakSet();
class Person {
  constructor() {
    personSet.add(this);
  }

  running() {
    if (!personSet.has(this)) {
      throw new Error("不能通过非构造方法创建出来的对象调用running方法");
    }
    console.log("running~", this);
  }
}

let p = new Person();
p.running();
// p = null;

p.running.call({ name: "why" });
```



### **Map和WeakMap的基本使用**

#### Map

- Map，用于存储映射关系。 
  - 但是我们可能会想，在之前我们可以使用对象来存储映射关系，他们有什么区别呢？

- 事实上我们对象存储映射关系只能用字符串（ES6新增了Symbol）作为属性名（key）； 

- 某些情况下我们可能希望通过其他类型作为key，比如对象，这个时候会自动将对象转成字符串来作为key；

```js
// 1.JavaScript中对象中是不能使用对象来作为key的
const obj1 = { name: "why" };
const obj2 = { name: "kobe" };

// const info = {
//   [obj1]: "aaa",
//   [obj2]: "bbb"
// }

// console.log(info)

// 2.Map就是允许我们对象类型来作为key的
// 构造方法的使用
const map = new Map();
map.set(obj1, "aaa");
map.set(obj2, "bbb");
map.set(1, "ccc");
console.log(map);

const map2 = new Map([
  [obj1, "aaa"],
  [obj2, "bbb"],
  [2, "ddd"],
]);
console.log(map2);

// 3.常见的属性和方法
console.log(map2.size);

// set
map2.set("why", "eee");
console.log(map2);

// get(key)
console.log(map2.get("why"));

// has(key)
console.log(map2.has("why"));

// delete(key)
map2.delete("why");
console.log(map2);

// clear
// map2.clear()
// console.log(map2)

// 4.遍历map
map2.forEach((item, key) => {
  console.log(item, key);
});

for (const item of map2) {
  // console.log(item[0], item[1]);
  console.log(item);
}

for (const [key, value] of map2) {
  console.log(key, value);
}
```

- Map常见的属性：
  - size：返回Map中元素的个数；

- Map常见的方法：
  - set(key, value)：在Map中添加key、value，并且返回整个Map对象；
  - get(key)：根据key获取Map中的value； 
  - has(key)：判断是否包括某一个key，返回Boolean类型；
  - delete(key)：根据key删除一个键值对，返回Boolean类型；
  - clear()：清空所有的元素； 
  - forEach(callback, [, thisArg])：通过forEach遍历Map； 

- Map也可以通过for of进行遍历。

#### WeakMap

和Map有什么区别呢？

- 区别一：WeakMap的key只能使用对象，不接受其他的类型作为key； 

- 区别二：WeakMap的key对对象想的引用是弱引用，如果没有其他引用引用这个对象，那么GC可以回收该对象；

WeakMap常见的方法有四个：

- set(key, value)：在Map中添加key、value，并且返回整个Map对象；

- get(key)：根据key获取Map中的value； 

- has(key)：判断是否包括某一个key，返回Boolean类型；

- delete(key)：根据key删除一个键值对，返回Boolean类型；

注意：WeakMap也是不能遍历的 

- 因为没有forEach方法，也不支持通过for of的方式进行遍历； 

![image-20220829153539472](https://s2.loli.net/2022/08/29/nSEzcJwYx3WuLlr.png)

```js
const obj = { name: "obj1" };
// 1.WeakMap和Map的区别二:
const map = new Map();
map.set(obj, "aaa");

const weakMap = new WeakMap();
weakMap.set(obj, "aaa");

// 2.区别一: 不能使用基本数据类型
// weakMap.set(1, "ccc")

// 3.常见方法
// get方法
console.log(weakMap.get(obj));

// has方法
console.log(weakMap.has(obj));

// delete方法
console.log(weakMap.delete(obj));
// WeakMap { <items unknown> }
console.log(weakMap);
```



 **WeakMap的应用**

在Vue3响应式原理的使用

![image-20220829175409764](https://s2.loli.net/2022/08/29/wLyFaM1SpANU9iW.png)

```js
// 应用场景(vue3响应式原理)
const obj1 = {
  name: "why",
  age: 18,
};

function obj1NameFn1() {
  console.log("obj1NameFn1被执行");
}

function obj1NameFn2() {
  console.log("obj1NameFn2被执行");
}

function obj1AgeFn1() {
  console.log("obj1AgeFn1");
}

function obj1AgeFn2() {
  console.log("obj1AgeFn2");
}

const obj2 = {
  name: "kobe",
  height: 1.88,
  address: "广州市", 
};

function obj2NameFn1() {
  console.log("obj2NameFn1被执行");
}

function obj2NameFn2() {
  console.log("obj2NameFn2被执行");
}

// 1.创建WeakMap
const weakMap = new WeakMap();

// 2.收集依赖结构
// 2.1.对obj1收集的数据结构
const obj1Map = new Map();
obj1Map.set("name", [obj1NameFn1, obj1NameFn2]);
obj1Map.set("age", [obj1AgeFn1, obj1AgeFn2]);
weakMap.set(obj1, obj1Map);

// 2.2.对obj2收集的数据结构
const obj2Map = new Map();
obj2Map.set("name", [obj2NameFn1, obj2NameFn2]);
weakMap.set(obj2, obj2Map);

// 3.如果obj1.name发生了改变
// Proxy/Object.defineProperty
obj1.name = "james";
const targetMap = weakMap.get(obj1);
const fns = targetMap.get("name");
fns.forEach((item) => item());
```



## 八、ES7~ES12

### **ES7**

#### Array Includes

- 在ES7之前，如果我们想判断一个数组中是否包含某个元素，需要通过 indexOf 获取结果，并且判断是否为 -1。 
- 在ES7中，我们可以通过includes来判断一个数组中是否包含一个指定的元素，根据情况，如果包含则返回 true，否则返回false。

```js
const names = ["abc", "cba", "nba", "mba", NaN];

if (names.indexOf("cba") !== -1) {
  console.log("包含abc元素");
}

// ES7 ES2016
if (names.includes("cba", 2)) {
  console.log("包含abc元素");
}

if (names.indexOf(NaN) !== -1) {
  console.log("包含NaN");
}

if (names.includes(NaN)) {
  console.log("包含NaN___");
}
```

#### **指数(乘方) exponentiation运算符**

- 在ES7之前，计算数字的乘方需要通过 Math.pow 方法来完成。
- 在ES7中，增加了 ** 运算符，可以对数字来计算乘方。

```js
const result1 = Math.pow(3, 3);
// ES7: **
const result2 = 3 ** 3;
console.log(result1, result2);
```



### **ES8**

#### Object values

- 之前我们可以通过 Object.keys 获取一个对象所有的key，在ES8中提供了 Object.values 来获取所有的value值：

```js
const obj = {
  name: "why",
  age: 18
}

console.log(Object.keys(obj)) // [ 'name', 'age' ]
console.log(Object.values(obj)) // [ 'why', 18 ]

// 用的非常少
console.log(Object.values(["abc", "cba", "nba"]))
console.log(Object.values("abc"))
```

#### **Object entries**

- 通过Object.entries 可以获取到一个数组，数组中会存放可枚举属性的键值对数组。

```js
const obj = {
  name: "why",
  age: 18
}

console.log(Object.entries(obj))	// [ [ 'name', 'why' ], [ 'age', 18 ] ]
const objEntries = Object.entries(obj)
objEntries.forEach(item => {
  console.log(item[0], item[1])
})

console.log(Object.entries(["abc", "cba", "nba"]))
console.log(Object.entries("abc"))
```

##### **String Padding**

- 某些字符串我们需要对其进行前后的填充，来实现某种格式化效果，ES8中增加了 padStart 和 padEnd 方法，分别是对字符串的首尾进行填充的

```js
const message = "Hello World";

const newMessage = message.padStart(15, "*").padEnd(20, "-");
console.log(newMessage);

// 案例 比如需要对身份证、银行卡的前面位数进行隐藏：
const cardNumber = "321324234242342342341312";
const lastFourCard = cardNumber.slice(-4);
const finalCard = lastFourCard.padStart(cardNumber.length, "*");
console.log(finalCard);
```

#### **Trailing Commas**

在ES8中，我们允许在函数定义和调用时多加一个逗号：

```js
function foo(m, n) {}

foo(20, 30);
```

#### **Object Descriptors**

Object.create(prototype, descriptors) 以指定的**原型创建对象，并且可以（可选） 的 设置对象的属性** Object.defineProperty(object, propertyname, descriptor) 对指定的对象的一个属性设置丰富的值控制 



### **ES9**

#### **Async iterators：后续迭代器讲解**



#### **Object spread operators：**

你可以通过展开操作符(Spread operator)`...`扩展一个数组对象和字符串。展开运算符（spread）是三个点（…）,可以将可迭代对象转为用逗号分隔的参数序列。如同rest参数的逆运算。

#### **Promise finally：后续讲Promise讲解**



### **ES10**

#### **flat flatMap**

- flat() 方法会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回。

- flatMap() 方法首先使用映射函数映射每个元素，然后将结果压缩成一个新数组。

  - 注意一：flatMap是先进行map操作，再做flat的操作；

  - 注意二：flatMap中的flat相当于深度为1；

```js
// 1.flat的使用
const nums = [
  10,
  20,
  [2, 9],
  [
    [30, 40],
    [10, 45],
  ],
  78,
  [55, 88],
];
const newNums = nums.flat();
console.log(newNums); // [ 10, 20, 2, 9, [ 30, 40 ], [ 10, 45 ], 78, 55, 88 ]

const newNums2 = nums.flat(2);
console.log(newNums2);
/*
[
  10, 20,  2,  9, 30,
  40, 10, 45, 78, 55,
  88
] */

// 2.flatMap的使用
const nums2 = [10, 20, 30];
const newNums3 = nums2.flatMap((item) => {
  return item * 2;
});
const newNums4 = nums2.map((item) => {
  return item * 2;
});

console.log(newNums3); // [ 20, 40, 60 ]
console.log(newNums4);

// 3.flatMap的应用场景
const messages = ["Hello World", "hello lyh", "my name is coderwhy"];
const words = messages.flatMap((item) => {
  return item.split(" ");
});

console.log(words);

/*
[
  'Hello', 'World',
  'hello', 'lyh',
  'my',    'name',
  'is',    'coderwhy'
] */
```



#### **Object fromEntries**

- 在前面，我们可以通过 Object.entries 将一个对象转换成 entries ，那么如果我们有一个entries了，如何将其转换成对象呢？
- ES10提供了 Object.formEntries 来完成转换： 

```js
const obj = {
  name: "why",
  age: 18,
  height: 1.88,
};

const entries = Object.entries(obj);
console.log(entries);

const newObj = {};
for (const entry of entries) {
  newObj[entry[0]] = entry[1];
}

// 1.ES10中新增了Object.fromEntries方法
const newObj2 = Object.fromEntries(entries);

console.log(newObj2);

// 2.Object.fromEntries的应用场景
const queryString = "name=why&age=18&height=1.88";
const queryParams = new URLSearchParams(queryString);
for (const param of queryParams) {
  console.log(param);
}

const paramObj = Object.fromEntries(queryParams);
console.log(paramObj); // { name: 'why', age: '18', height: '1.88' }
```

#### **trimStart trimEnd**

- 去除一个字符串首尾的空格，我们可以通过trim方法，如果单独去除前面或者后面呢？

- ES10中给我们提供了trimStart和trimEnd；

```js
const message = "    Hello World    "

console.log(message.trim())
console.log(message.trimStart())
console.log(message.trimEnd())
/*
Hello World
Hello World    
    Hello World
*/
```

#### **Symbol description：已经讲过了**

symbol.description是JavaScript中的内置属性，用于返回指定符号对象的可选描述。\

```js
console.log(Symbol('desc').description);
// expected output: "desc"
```



#### **Optional catch binding：后面讲解try cach讲解**



### **ES11**

#### **BigInt**

- 在早期的JavaScript中，我们不能正确的表示过大的数字：

  大于MAX_SAFE_INTEGER的数值，表示的可能是不正确的。

- 那么ES11中，引入了新的数据类型BigInt，用于表示大的整数：

  BitInt的表示方法是在数值的后面加上n

```js
// ES11之前 max_safe_integer
const maxInt = Number.MAX_SAFE_INTEGER
console.log(maxInt) // 9007199254740991
console.log(maxInt + 1)
console.log(maxInt + 2)

// ES11之后: BigInt
const bigInt = 900719925474099100n
console.log(bigInt + 10n)

const num = 100
console.log(bigInt + BigInt(num))

const smallNum = Number(bigInt)
console.log(smallNum)
```

#### **Nullish Coalescing Operator**

- Nullish Coalescing Operator增加了空值合并操作符：

```js
// ES11: 空值合并运算 ??

const foo = undefined;
// const bar = foo || "default value"
const bar = foo ?? "defualt value";

console.log(bar); // defualt value
```

#### **Optional Chaining**

- 可选链也是ES11中新增一个特性，主要作用是让我们的代码在进行null和undefined判断时更加清晰和简洁：

```js
const info = {
  name: "why",
  // friend: {
  //   girlFriend: {
  //     name: "hmm"
  //   }
  // }
};

// console.log(info.friend.girlFriend.name);
if (info && info.friend && info.friend.girlFriend) {
  console.log(info.friend.girlFriend.name);
}

// ES11提供了可选链(Optional Chainling)
console.log(info.friend?.girlFriend?.name);

console.log("其他的代码逻辑");
```

#### **Global This**

- 在之前我们希望获取JavaScript环境的全局对象，不同的环境获取的方式是不一样的 
- 比如在浏览器中可以通过this、window来获取；
- 比如在Node中我们需要通过global来获取；
- 那么在ES11中对获取全局对象进行了统一的规范：globalThis

```js
// 获取某一个环境下的全局对象(Global Object)

// 在浏览器下
// console.log(window)
// console.log(this)

// 在node下
// console.log(global)

// ES11
console.log(globalThis);
```

#### **for..in标准化**

- 在ES11之前，虽然很多浏览器支持for...in来遍历对象类型，但是并没有被ECMA标准化。 

- 在ES11中，对其进行了标准化，for...in是用于遍历对象的key的：

```js
// for...in 标准化: ECMA
const obj = {
  name: "why",
  age: 18,
};

for (const item in obj) {
  console.log(item);
}
```

#### **Dynamic Import：**

后续ES Module模块化中讲解。

#### **Promise.allSettled：**

后续讲Promise的时候讲解。

#### **import meta：**

后续ES Module模块化中讲解



### **ES12**

#### **FinalizationRegistry**

- FinalizationRegistry 对象可以让你在对象被垃圾回收时请求一个回调。 
  - FinalizationRegistry 提供了这样的一种方法：当一个在注册表中注册的对象被回收时，请求在某个时间点上调用一个清理回调。（清理回调有时被称为 finalizer ）; 
  - 你可以通过调用register方法，注册任何你想要清理回调的对象，传入该对象和所含的值;

```js
// ES12: FinalizationRegistry类
const finalRegistry = new FinalizationRegistry((value) => {
  console.log("注册在finalRegistry的对象, 某一个被销毁", value)
})

let obj = { name: "why" }
let info = { age: 18 }

finalRegistry.register(obj, "obj")
finalRegistry.register(info, "value")

obj = null
info = null

```

#### **WeakRefs**

- 如果我们默认将一个对象赋值给另外一个引用，那么这个引用是一个强引用：
- 如果我们希望是一个弱引用的话，可以使用WeakRef；

```js
// ES12: WeakRef类
// WeakRef.prototype.deref:
// > 如果原对象没有销毁, 那么可以获取到原对象
// > 如果原对象已经销毁, 那么获取到的是undefined
const finalRegistry = new FinalizationRegistry((value) => {
  console.log("注册在finalRegistry的对象, 某一个被销毁", value);
});

let obj = { name: "why" };

// 弱引用
let info = new WeakRef(obj);

finalRegistry.register(obj, "obj");

obj = null;

setTimeout(() => {
  console.log(info.deref()?.name);
  console.log(info.deref() && info.deref().name);
}, 10000);
```

#### **logical assignment operators**

```js
// 1.||= 逻辑或赋值运算
let message = "hello world";
message = message || "default value";
message ||= "default value";
console.log(message);

// 2.&&= 逻辑与赋值运算
// &&
const obj = {
  name: "why",
  foo: function () {
    console.log("foo函数被调用");
  },
};

obj.foo && obj.foo();

// &&=
let info = {
  name: "why",
};

// 1.判断info
// 2.有值的情况下, 取出info.name
// info = info && info.name
info &&= info.name;
console.log(info);

// 3.??= 逻辑空赋值运算
let message2 = 0;
message2 ??= "default value";
console.log(message2);

```

#### **Numeric Separator：**

**讲过了；**

#### **String.replaceAll：**

**字符串替换；**



## 九、**监听对象与响应式原理**

### 监听对象方法

#### 第一种：Object.defineProperty

Object.defineProperty的存储属性描述符来对属性的操作进行监听

**但是这样做有什么缺点呢？**

- 首先，Object.defineProperty设计的初衷，不是为了去监听截止一个对象中所有的属性的。
  - 我们在定义某些属性的时候，初衷其实是定义普通的属性，但是后面我们强行将它变成了数据属性描述符。

- 其次，如果我们想监听更加丰富的操作，比如新增属性、删除属性，那么Object.defineProperty是无能为力的。

- 所以我们要知道，存储数据描述符设计的初衷并不是为了去监听一个完整的对象。

```js
// 遍历object所有的key
Object.keys(obj).forEach((key) => {
  let value = obj[key];

  Object.defineProperty(obj, key, {
    get: function () {
      console.log(`监听到obj对象的${key}属性被访问了`);
      return value;
    },
    set: function (newValue) {
      console.log(`监听到obj对象的${key}属性被设置值`);
      value = newValue;
    },
  }); 
});
```



#### **第二种：Proxy类**

这个类从名字就可以看出来，是用于帮助我们创建一个**代理**的：

- 也就是说，如果我们希望监听一个对象的相关操作，那么我们可以先创建一个代理对象（Proxy对象）； 

- 之后对该对象的所有操作，都通过代理对象来完成，代理对象可以监听我们想要对原对象进行哪些操作；

我们可以将上面的案例用Proxy来实现一次：

- 首先，我们需要new Proxy对象，并且传入需要侦听的对象以及一个处理对象，可以称之为handler； 
  - const p = new Proxy(target, handler)

- 其次，我们之后的操作都是直接对Proxy的操作，而不是原有的对象，因为我们需要在handler里面进行侦听；

```js
const objProxy = new Proxy(obj, {
  // 获取值时的捕获器
  get: function (target, key) {
    console.log(`监听到对象的${key}属性被访问了`, target);
    return target[key];
  },

  // 设置值时的捕获器
  set: function (target, key, newValue) {
    console.log(`监听到对象的${key}属性被设置值`, target);
    target[key] = newValue;
  },
});
```



#### **Proxy的set和get捕获器**

- 如果我们想要侦听某些具体的操作，那么就可以在handler中添加对应的捕捉器（Trap）：

- set和get分别对应的是函数类型；
  - set函数有四个参数：
    - target：目标对象（侦听的对象）；
    - property：将被设置的属性key； 
    - value：新属性值；
    - receiver：调用的代理对象；
  - get函数有三个参数：
    - target：目标对象（侦听的对象）；
    - property：被获取的属性key； 
    - receiver：调用的代理对象；

```js
const objProxy = new Proxy(obj, {
  // 获取值时的捕获器
  get: function (target, key) {
    console.log(`监听到对象的${key}属性被访问了`, target);
    return target[key];
  },

  // 设置值时的捕获器
  set: function (target, key, newValue) {
    console.log(`监听到对象的${key}属性被设置值`, target);
    target[key] = newValue;
  },

  // 监听in的捕获器
  has: function (target, key) {
    console.log(`监听到对象的${key}属性in操作`, target);
    return key in target;
  },

  // 监听delete的捕获器
  deleteProperty: function (target, key) {
    console.log(`监听到对象的${key}属性in操作`, target);
    delete target[key];
  },
});
```



**其他的捕获器**

![image-20220910003743283](http://lyc-markdownimg.test.upcdn.net/img/202209100213699.png)



#### **Proxy的construct和apply**

到捕捉器中还有construct和apply，它们是应用于函数对象的：

```js
function foo() {}

const fooProxy = new Proxy(foo, {
  apply: function (target, thisArg, argArray) {
    console.log("对foo函数进行了apply调用");
    return target.apply(thisArg, argArray);
  },
  construct: function (target, argArray, newTarget) {
    console.log("对foo函数进行了new调用");
    return new target(...argArray);
  },
});

fooProxy.apply({}, ["abc", "cba"]);
new fooProxy("abc", "cba");
```



### **Reflect对象**

- Reflect也是ES6新增的一个API，它是**一个对象**，字面的意思是**反射**。 

- **那么这个Reflect有什么用呢？**
  - 它主要提供了很多操作JavaScript对象的方法，有点像Object中操作对象的方法； 
  -  比如Reflect.getPrototypeOf(target)类似于 Object.getPrototypeOf()； 
  -  比如Reflect.defineProperty(target, propertyKey, attributes)类似于Object.defineProperty() ； 

- 如果我们有Object可以做这些操作，那么**为什么还需要有Reflect这样的新增对象**呢？
  - 这是因为在早期的ECMA规范中没有考虑到这种对 **对象本身** 的操作如何设计会更加规范，所以将这些API放到了Object上面； 
  -  但是Object作为一个构造函数，这些操作实际上放到它身上并不合适； 
  -  另外还包含一些类似于 in、delete操作符，让JS看起来是会有一些奇怪的；
  -  所以在ES6中新增了Reflect，让我们这些操作都集中到了Reflect对象上；

- 那么Object和Reflect对象之间的API关系，可以参考[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/Comparing_Reflect_and_Object_methods)



#### **Reflect的常见方法**

![image-20220910005035278](http://lyc-markdownimg.test.upcdn.net/img/202209100213900.png)



#### **Reflect的使用**

那么我们可以将之前Proxy案例中对原对象的操作，都修改为Reflect来操作：

```js
const obj = {
  name: "why",
  age: 18,
};

const objProxy = new Proxy(obj, {
  get: function (target, key, receiver) {
    console.log("get---------");
    return Reflect.get(target, key);
  },

  set: function (target, key, newValue, receiver) {
    console.log("set---------");
    target[key] = newValue;
    const result = Reflect.set(target, key, newValue);
    if (result) {
    } else {
    }
  },
});

objProxy.name = "kobe";
console.log(objProxy.name);
```



#### **Receiver的作用**

- 我们发现在使用getter、setter的时候有一个receiver的参数，它的作用是什么呢？
  - 如果我们的源对象（obj）有setter、getter的访问器属性，那么可以通过receiver来改变里面的this； 

- 我们来看这样的一个对象

```js
const objProxy = new Proxy(obj, {
  get: function (target, key, receiver) {
    // receiver是创建出来的代理对象 改变this
    console.log("get方法被访问--------", key, receiver);
    console.log(receiver === objProxy); // ture
    return Reflect.get(target, key, receiver);
  },
  set: function (target, key, newValue, receiver) {
    console.log("set方法被访问--------", key);
    Reflect.set(target, key, newValue, receiver);
  },
});
```



#### **Reflect的construct**



```js
function Student(name, age) {
  this.name = name;
  this.age = age;
}

function Teacher() {}

// const stu = new Student("why", 18)
// console.log(stu)
// console.log(stu.__proto__ === Student.prototype)

// 执行Student函数中的内容, 但是创建出来对象是Teacher对象
const teacher = Reflect.construct(Student, ["why", 18], Teacher);
console.log(teacher); // Teacher { name: 'why', age: 18 }
console.log(teacher.__proto__ === Teacher.prototype); // ture
    
```



### **响应式原理**

- 我们先来看一下响应式意味着什么？我们来看一段代码：
  - m有一个初始化的值，有一段代码使用了这个值；
  - 那么在m有一个新的值时，这段代码可以自动重新执行；

- 上面的这样一种可以自动响应数据变量的代码机制，我们就称之为是响应式的。 
  - 那么我们再来看一下对象的响应式：

```js
// 对象的响应式
const obj = {
  name: "why",
  age: 18
}

const newName = obj.name
console.log("你好啊, 李银河")
console.log("Hello World")
console.log(obj.name) // 100行

obj.name = "kobe"
```

![image-20220910010352190](http://lyc-markdownimg.test.upcdn.net/img/202209100213716.png)



- 首先，执行的代码中可能不止一行代码，所以我们可以将这些代码放到一个函数中： 
  - 那么我们的问题就变成了，当数据发生变化时，自动去执行某一个函数；

- 但是有一个问题：在开发中我们是有很多的函数的，我们如何区分一个函数需要响应式，还是不需要响应式呢？
  - 很明显，下面的函数中 foo 需要在obj的name发生变化时，重新执行，做出相应；
  - bar函数是一个完全独立于obj的函数，它不需要执行任何响应式的操作；

![image-20220910010506596](http://lyc-markdownimg.test.upcdn.net/img/202209100213683.png)



#### **响应式函数的实现watchFn**

```js
// 封装一个响应式的函数
let reactiveFns = [];
function watchFn(fn) {
  reactiveFns.push(fn);
}

// 对象的响应式
const obj = {
  name: "why",
  age: 18,
};

watchFn(function () {
  const newName = obj.name;
  console.log("你好啊, 李银河");
  console.log("Hello World");
  console.log(obj.name); // 100行
});

//当值改变时进行遍历
obj.name = "kobe";
reactiveFns.forEach((fn) => {
  fn();
```

- 但是我们怎么区分呢？

  - 这个时候我们封装一个新的函数watchFn； 

  - 凡是传入到watchFn的函数，就是需要响应式的；

  - 其他默认定义的函数都是不需要响应式的；



#### **响应式依赖的收集**

- 目前我们收集的依赖是放到一个数组中来保存的，但是这里会存在数据管理的问题：
  - 我们在实际开发中需要监听很多对象的响应式；
  -  这些对象需要监听的不只是一个属性，它们很多属性的变化，都会有对应的响应式函数；
  -  我们不可能在全局维护一大堆的数组来保存这些响应函数；

- 所以我们要设计一个类，这个类用于管理某一个对象的某一个属性的所有响应式函数：
  - 相当于替代了原来的简单 reactiveFns 的数组；

```js
class Depend {
  constructor() {
    this.reactiveFns = [];
  }

  addDepend(reactiveFn) {
    this.reactiveFns.push(reactiveFn);
  }

  notify() {
    this.reactiveFns.forEach((fn) => {
      fn();
    });
  }
}

// 封装一个响应式的函数
const depend = new Depend();
function watchFn(fn) {
  depend.addDepend(fn);
}

// 对象的响应式
const obj = {
  name: "why", // depend对象
  age: 18, // depend对象
};

watchFn(function () {
  const newName = obj.name;
  console.log("你好啊, 李银河");
  console.log("Hello World");
  console.log(obj.name); // 100行
});

watchFn(function () {
  console.log(obj.name, "demo function -------");
});

obj.name = "kobe";
depend.notify();
```

- 我们目前是创建了一个Depend对象，用来管理对于name变化需要监听的响应函数：
  - 但是实际开发中我们会有不同的对象，另外会有不同的属性需要管理；
  - 我们如何可以使用一种数据结构来管理不同对象的不同依赖关系呢？

- 在前面我们刚刚学习过WeakMap，并且在学习WeakMap的时候我讲到了后面通过WeakMap如何管理这种响应

式的数据依赖：

![image-20220910011205066](http://lyc-markdownimg.test.upcdn.net/img/202209100213456.png)



#### **监听对象的变化**

- 那么我们接下来就可以通过之前学习的方式来监听对象的变量：
  - 方式一：通过 Object.defineProperty的方式（vue2采用的方式）；
  - 方式二：通过new Proxy的方式（vue3采用的方式）；

 **Proxy的方式**

```js

// 监听对象的属性变量: Proxy(vue3)/Object.defineProperty(vue2)
const objProxy = new Proxy(obj, {
  get: function (target, key, receiver) {
    return Reflect.get(target, key, receiver);
  },
  set: function (target, key, newValue, receiver) {
    Reflect.set(target, key, newValue, receiver);
    depend.notify();
  },
});

```

Object.defineProperty**的方式**

```js
function reactive(obj) {
  Object.keys(obj).forEach(key => {
    let value = obj[key]
    Object.defineProperty(obj, key, {
      get: function() {
        return value
      },
      set: function(newValue) {
        value = newValue
        depend.notify()
      }
    })
  })
  return obj
}
```



#### **对象依赖管理的实现**

我们可以写一个getDepend函数专门来管理这种依赖关系：

```js

// 封装一个响应式的函数
const depend = new Depend()
function watchFn(fn) {
  depend.addDepend(fn)
}

// 封装一个获取depend函数 获取每个响应式函数的depend对象
const targetMap = new WeakMap()
function getDepend(target, key) {
  // 根据target对象获取map的过程
  let map = targetMap.get(target)
  if (!map) {
    map = new Map()
    targetMap.set(target, map)
  }
  // 根据key获取depend对象
  let depend = map.get(key)
  if (!depend) {
    depend = new Depend()
    map.set(key, depend)
  }
  return depend
}

// 对象的响应式
const obj = {
  name: "why", // depend对象
  age: 18 // depend对象
}

// 监听对象的属性变量: Proxy(vue3)/Object.defineProperty(vue2)
const objProxy = new Proxy(obj, {
  get: function(target, key, receiver) {
    return Reflect.get(target, key, receiver)
  },
  set: function(target, key, newValue, receiver) {
    Reflect.set(target, key, newValue, receiver)
    // depend.notify()
    const depend = getDepend(target, key)
    depend.notify()
  }
})
```



#### **正确的依赖收集**

- 我们之前收集依赖的地方是在 watchFn 中：
  - 但是这种收集依赖的方式我们根本不知道是哪一个key的哪一个depend需要收集依赖； 
  - 你只能针对一个单独的depend对象来添加你的依赖对象； 

- 那么正确的应该是在哪里收集呢？应该在我们调用了Proxy的get捕获器时
  - 因为如果一个函数中使用了某个对象的key，那么它应该被收集依赖；

```js
class Depend {
  constructor() {
    this.reactiveFns = []
  }

  addDepend(reactiveFn) {
    this.reactiveFns.push(reactiveFn)
  }

  notify() {
    console.log(this.reactiveFns)
    this.reactiveFns.forEach(fn => {
      fn()
    })
  }
}

// 封装一个响应式的函数和全局activeReactiveFn
let activeReactiveFn = null
function watchFn(fn) {
  activeReactiveFn = fn
  fn()
  activeReactiveFn = null
}

// 封装一个获取depend函数
const targetMap = new WeakMap()
function getDepend(target, key) {
  // 根据target对象获取map的过程
  let map = targetMap.get(target)
  if (!map) {
    map = new Map()
    targetMap.set(target, map)
  }

  // 根据key获取depend对象
  let depend = map.get(key)
  if (!depend) {
    depend = new Depend()
    map.set(key, depend)
  }
  return depend
}

// 对象的响应式
const obj = {
  name: "why", // depend对象
  age: 18 // depend对象
}

// 监听对象的属性变量: Proxy(vue3)/Object.defineProperty(vue2)
const objProxy = new Proxy(obj, {
  get: function(target, key, receiver) {
    // 根据target.key获取对应的depend
    const depend = getDepend(target, key)
    // 给depend对象中添加响应函数activeReactiveFn
    depend.addDepend(activeReactiveFn)
    return Reflect.get(target, key, receiver)
  },
  set: function(target, key, newValue, receiver) {
    Reflect.set(target, key, newValue, receiver)
    // depend.notify()
    const depend = getDepend(target, key)
    depend.notify()
  }
})
```



#### **对Depend重构**

- 但是这里有两个问题：
  - 问题一：如果函数中有用到两次key，比如name，那么这个函数会被收集两次；
  -  问题二：我们并不希望将添加reactiveFn放到get中，以为它是属于Depend的行为；

- 所以我们需要对Depend类进行重构：
  - 解决问题一的方法：不使用数组，而是使用Set； 
  -  解决问题二的方法：添加一个新的方法，用于收集依赖；

```js
class Depend {
  constructor() {
    this.reactiveFns = new Set();
  }

  // 不需要手动往depend里增加了响应式函数	直接增加activeReactiveFn
  depend() {
    if (activeReactiveFn) {
      this.reactiveFns.add(activeReactiveFn);
    }
  }

  notify() {
    this.reactiveFns.forEach((fn) => {
      fn();
    });
  }
}
```



#### **封装响应式对象**

我们目前的响应式是针对于obj一个对象的，我们可以创建出来一个函数，针对所有的对象都可以变成响应式对象：

```js
function reactive(obj) {
  return new Proxy(obj, {
    get: function (target, key, receiver) {
      // 根据target.key获取对应的depend
      const depend = getDepend(target, key);
      // 给depend对象中添加响应函数
      // depend.addDepend(activeReactiveFn)
      depend.depend();

      return Reflect.get(target, key, receiver);
    },
    set: function (target, key, newValue, receiver) {
      Reflect.set(target, key, newValue, receiver);
      // depend.notify()
      const depend = getDepend(target, key);
      depend.notify();
    },
  });
}
```



#### **响应式原理总结**

- 我们前面所实现的响应式的代码，其实就是Vue3中的响应式原理：
  - Vue3主要是通过Proxy来监听数据的变化以及收集相关的依赖的；
  - Vue2中通过我们前面学习过的Object.defineProerty的方式来实现对象属性的监听；

- 我们可以将reactive函数进行如下的重构： 
  - 在传入对象时，我们可以遍历所有的key，并且通过属性存储描述符来监听属性的获取和修改；
  - 在setter和getter方法中的逻辑和前面的Proxy是一致的；

##### Vue3响应式原理

```js
// 保存当前需要收集的响应式函数
let activeReactiveFn = null;

class Depend {
  constructor() {
    this.reactiveFns = new Set();
  }
  
  depend() {
    if (activeReactiveFn) {
      this.reactiveFns.add(activeReactiveFn);
    }
  }
  
  notify() {
    this.reactiveFns.forEach((fn) => {
      fn();
    });
  }
}

// 封装一个响应式的函数
function watchFn(fn) {
  activeReactiveFn = fn;
  fn();
  activeReactiveFn = null;
}

// 封装一个获取depend函数
const targetMap = new WeakMap();
function getDepend(target, key) {
  // 根据target对象获取map的过程
  let map = targetMap.get(target);
  if (!map) {
    map = new Map();
    targetMap.set(target, map);
  }

  // 根据key获取depend对象
  let depend = map.get(key);
  if (!depend) {
    depend = new Depend();
    map.set(key, depend);
  }
  return depend;
}

function reactive(obj) {
  return new Proxy(obj, {
    get: function (target, key, receiver) {
      // 根据target.key获取对应的depend
      const depend = getDepend(target, key);
      // 给depend对象中添加响应函数
      // depend.addDepend(activeReactiveFn)
      depend.depend();

      return Reflect.get(target, key, receiver);
    },
    set: function (target, key, newValue, receiver) {
      Reflect.set(target, key, newValue, receiver);
      // depend.notify()
      const depend = getDepend(target, key);
      depend.notify();
    },
  });
}

// 监听对象的属性变量: Proxy(vue3)/Object.defineProperty(vue2)
const objProxy = reactive({
  name: "why", // depend对象
  age: 18, // depend对象
});

const infoProxy = reactive({
  address: "广州市",
  height: 1.88,
});

watchFn(() => {
  console.log(infoProxy.address);
});

infoProxy.address = "北京市";

const foo = reactive({
  name: "foo",
});

watchFn(() => {
  console.log(foo.name);
});

foo.name = "bar";
```

##### Vue2响应式原理

```js
// 保存当前需要收集的响应式函数
let activeReactiveFn = null

/**
 * Depend优化:
 *  1> depend方法
 *  2> 使用Set来保存依赖函数, 而不是数组[]
 */

class Depend {
  constructor() {
    this.reactiveFns = new Set()
  }

  // addDepend(reactiveFn) {
  //   this.reactiveFns.add(reactiveFn)
  // }

  depend() {
    if (activeReactiveFn) {
      this.reactiveFns.add(activeReactiveFn)
    }
  }

  notify() {
    this.reactiveFns.forEach(fn => {
      fn()
    })
  }
}

// 封装一个响应式的函数
function watchFn(fn) {
  activeReactiveFn = fn
  fn()
  activeReactiveFn = null
}

// 封装一个获取depend函数
const targetMap = new WeakMap()
function getDepend(target, key) {
  // 根据target对象获取map的过程
  let map = targetMap.get(target)
  if (!map) {
    map = new Map()
    targetMap.set(target, map)
  }

  // 根据key获取depend对象
  let depend = map.get(key)
  if (!depend) {
    depend = new Depend()
    map.set(key, depend)
  }
  return depend
}

function reactive(obj) {
  // {name: "why", age: 18}
  // ES6之前, 使用Object.defineProperty   
  Object.keys(obj).forEach(key => {
    let value = obj[key]
    Object.defineProperty(obj, key, {
      get: function() {
        const depend = getDepend(obj, key)
        depend.depend()
        return value
      },
      set: function(newValue) {
        value = newValue
        const depend = getDepend(obj, key)
        depend.notify()
      }
    })
  })
  return obj
}

// 监听对象的属性变量: Proxy(vue3)/Object.defineProperty(vue2)
const objProxy = reactive({
  name: "why", // depend对象
  age: 18 // depend对象
})

const infoProxy = reactive({
  address: "广州市",
  height: 1.88
})

watchFn(() => {
  console.log(infoProxy.address)
})

infoProxy.address = "北京市"

const foo = reactive({
  name: "foo"
})

watchFn(() => {
  console.log(foo.name)
})

foo.name = "bar"
foo.name = "hhh"

```



## 十、**Promise详解**

### **异步任务的处理**

- 在ES6出来之后，有很多关于Promise的讲解、文章，也有很多经典的书籍讲解Promise
  - 虽然等你学会Promise之后，会觉得Promise不过如此，但是在初次接触的时候都会觉得这个东西不好理解； 

- 那么这里我从一个实际的例子来作为切入点：
  - 我们调用一个函数，这个函数中发送网络请求（我们可以用定时器来模拟）；
  - 如果发送网络请求成功了，那么告知调用者发送成功，并且将相关数据返回过去；
  - 如果发送网络请求失败了，那么告知调用者发送失败，并且告知错误信息；

```js
/**
 * 这种回调的方式有很多的弊端:
 *  1> 如果是我们自己封装的requestData,那么我们在封装的时候必须要自己设计好callback名称, 并且使用好
 *  2> 如果我们使用的是别人封装的requestData或者一些第三方库, 那么我们必须去看别人的源码或者文档, 才知道它这个函数需要怎么去获取到结果
 */

// request.js
function requestData(url, successCallback, failtureCallback) {
  // 模拟网络请求
  setTimeout(() => {
    // 拿到请求的结果
    // url传入的是coderwhy, 请求成功
    if (url === "coderwhy") {
      // 成功
      let names = ["abc", "cba", "nba"]
      successCallback(names)
    } else { // 否则请求失败
      // 失败
      let errMessage = "请求失败, url错误"
      failtureCallback(errMessage)
    }
  }, 3000);
}

// main.js
requestData("kobe", (res) => {
  console.log(res)
}, (err) => {
  console.log(err)
})

// 更规范/更好的方案 Promise承诺(规范好了所有的代码编写逻辑)
function requestData2() {
  return "承诺"
}

const chengnuo = requestData2()
```



### **Promise简介**

**什么是Promise呢？**

- 在上面的解决方案中，我们确确实实可以解决请求函数得到结果之后，获取到对应的回调，但是它存在两个主要的问题：
  - 第一，我们需要自己来设计回调函数、回调函数的名称、回调函数的使用等；
  - 第二，对于不同的人、不同的框架设计出来的方案是不同的，那么我们必须耐心去看别人的源码或者文档，以便可以理解它这个函数到底怎么用；

- 我们来看一下Promise的API是怎么样的：
  - Promise是一个类，可以翻译成 承诺、许诺 、期约；
  - 当我们需要给予调用者一个承诺：待会儿我会给你回调数据时，就可以创建一个Promise的对象；
  - 在通过new创建Promise对象时，我们需要传入一个回调函数，我们称之为executor
    - 这个回调函数会被立即执行，并且给传入另外两个回调函数resolve、reject； 
    -  当我们调用resolve回调函数时，会执行Promise对象的then方法传入的回调函数；
    -  当我们调用reject回调函数时，会执行Promise对象的catch方法传入的回调函数；

#### **Promise代码结构：**

```js
// 传入的这个函数, 被称之为 executor
// > resolve: 回调函数, 在成功时, 回调resolve函数
// >reject: 回调函数, 在失败时, 回调reject函数
const promise = new Promise((resolve, reject) => {
  // console.log("promise传入的函数被执行了")
  // resolve()
  reject()
})

promise.then(() => {

})

promise.catch(() => {

})
```

上面Promise使用过程，我们可以将它划分成三个状态：

- *待定（pending）*: 初始状态，既没有被兑现，也没有被拒绝；
  - 当执行executor中的代码时，处于该状态；

- *已兑现（fulfilled）*: 意味着操作成功完成；
  - 执行了resolve时，处于该状态；

- *已拒绝（rejected）*: 意味着操作失败；
  - 执行了reject时，处于该状态；



**用了Promise，我们就可以将之前的代码进行重构了：**

```js
// request.js
function requestData(url) {
  // 异步请求的代码会被放入到executor中
  return new Promise((resolve, reject) => {
    // 模拟网络请求
    setTimeout(() => {
      // 拿到请求的结果
      // url传入的是coderwhy, 请求成功
      if (url === "coderwhy") {
        // 成功
        let names = ["abc", "cba", "nba"];
        resolve(names);
      } else {
        // 否则请求失败
        // 失败
        let errMessage = "请求失败, url错误";
        reject(errMessage);
      }
    }, 3000);
  });
}

// main.js
const promise = requestData("coderwhy");
promise.then(
  (res) => {
    console.log("请求成功:", res);
  },
  (err) => {
    console.log("请求失败:", err);
  }
);
```



### **Executor**

- Executor是在创建Promise时需要传入的一个回调函数，这个回调函数会被立即执行，并且传入两个参数： 

- 通常我们会在Executor中确定我们的Promise状态：
  - 通过resolve，可以兑现（fulfilled）Promise的状态，我们也可以称之为已决议（resolved）；
  - 通过reject，可以拒绝（reject）Promise的状态；

- 这里需要注意：一旦状态被确定下来，Promise的状态会被锁死，该Promise的状态是不可更改的 
  - 在我们调用resolve的时候，如果resolve传入的值本身不是一个Promise，那么会将该Promise的状态变成兑现（fulfilled）；
  - 在之后我们去调用reject时，已经不会有任何的响应了（并不是这行代码不会执行，而是无法改变Promise状态）；



### **resolve不同值的区别**

- 情况一：如果resolve传入一个普通的值或者对象，那么这个值会作为then回调的参数；

- 情况二：如果resolve中传入的是另外一个Promise，那么这个新Promise会决定原Promise的状态： 

- 情况三：如果resolve中传入的是一个对象，并且这个对象有实现then方法，那么会执行该then方法，并且根据then方法的结果来决定Promise的状态：

```js
// 完全等价于下面的代码
// 注意: Promise状态一旦确定下来, 那么就是不可更改的(锁定)
new Promise((resolve, reject) => {
  // pending状态: 待定/悬而未决的
  console.log("--------");
  reject(); // 处于rejected状态(已拒绝状态)
  resolve(); // 处于fulfilled状态(已敲定/兑现状态)
  console.log("++++++++++++");
}).then(
  (res) => {
    console.log("res:", res);
  },
  (err) => {
    console.log("err:", err);
  }
);
```



### 常见方法



#### **then方法**

- then方法是Promise对象上的一个方法：它其实是放在Promise的原型上的 Promise.prototype.then

- then方法接受两个参数：
  - fulfilled的回调函数：当状态变成fulfilled时会回调的函数；
  - reject的回调函数：当状态变成reject时会回调的函数；

```js
promise.then(res => {
  console.log("res:", res);
}, err => {
  console.log("err:", err);
})
// 等价于

promise.then(res => {
  console.log("res:", res);
}).catch(err => {
  console.log("err:", err);
})
```

##### **多次调用**

- 一个Promise的then方法是可以被多次调用的：
  - 每次调用我们都可以传入对应的fulfilled回调；
  - 当Promise的状态变成fulfilled的时候，这些回调函数都会被执行；

```js
promise.then(res => {
  console.log("res1:", res)
})

promise.then(res => {
  console.log("res2:", res)
})

promise.then(res => {
  console.log("res3:", res)
})
```

##### **返回值**

- Promise有三种状态，那么这个Promise处于什么状态呢？
  - 当then方法中的回调函数本身在执行的时候，那么它处于pending状态；
  - 当then方法中的回调函数返回一个结果时，那么它处于fulfilled状态，并且会将结果作为resolve的参数；
    - 情况一：返回一个普通的值； 
    -  情况二：返回一个Promise； 
    -  情况三：返回一个thenable值；
  - 当then方法抛出一个异常时，那么它处于reject状态；

```js
// 2.then方法传入的 "回调函数: 可以有返回值
// then方法本身也是有返回值的, 它的返回值是Promise

// 1> 如果我们返回的是一个普通值(数值/字符串/普通对象/undefined), 那么这个普通的值被作为一个新的Promise的resolve值
promise
  .then((res) => {
    return "aaaaaa";
  })
  .then((res) => {
    console.log("res:", res);
    return "bbbbbb";
  });

// 2> 如果我们返回的是一个Promise
promise.then(res => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(111111)
    }, 3000)
  })
}).then(res => {
  console.log("res:", res)
})

// 3> 如果返回的是一个对象, 并且该对象实现了thenable
promise
  .then((res) => {
    return {
      then: function (resolve, reject) {
        resolve(222222);
      },
    };
  })
  .then((res) => {
    console.log("res:", res);
  });
```



#### **catch方法**

- catch方法也是Promise对象上的一个方法：它也是放在Promise的原型上的 Promise.prototype.catch
-  一个Promise的catch方法是可以被多次调用的：
  - 每次调用我们都可以传入对应的reject回调；
  - 当Promise的状态变成reject的时候，这些回调函数都会被执行

```js
promise.catch(err => {
  console.log("err:", err);
})
```

##### **返回值**

事实上catch方法也是会返回一个Promise对象的，所以catch方法后面我们可以继续调用then方法或者catch方法：

- 但是第一个promise会返回一个**undefined**

- 如果我们希望后续继续执行catch，那么需要**抛出一个异常**：

```js
promise
  .catch(err => {
    console.log("err1:", err);
    // throw new Error("error message")
  })
  .catch(err => {
    console.log("err2:", err);
  })
  .then(res => {
    console.log("res:", res);
  })
// err1: hahaha
// res: undefined

```



#### **finally方法**

- finally是在ES9（ES2018）中新增的一个特性：表示无论Promise对象无论变成fulfilled还是reject状态，最终都会被执行的代码。 

- finally方法是不接收参数的，因为无论前面是fulfilled状态，还是reject状态，它都会执行。

```js
promise
  .then((res) => {
    console.log("res:", res);
  })
  .catch((err) => {
    console.log("err:", err);
  })
  .finally(() => {
    console.log("finally code execute");
  });
```



#### **resolve方法**

- 有时候我们已经有一个现成的内容了，希望将其转成Promise来使用，这个时候我们可以使用 Promise.resolve 方法来完成。 

- Promise.resolve的用法相当于new Promise，并且执行resolve操作：

```js
// 类方法Promise.resolve
// 1.普通的值
const promise = Promise.resolve({ name: "why" })
// 相当于
const promise2 = new Promise((resolve, reject) => {
  resolve({ name: "why" })
})

// 2.传入Promise
const promise = Promise.resolve(new Promise((resolve, reject) => {
  resolve("11111")
}))

promise.then(res => {
  console.log("res:", res)
})

// 3.传入thenable对象
const promise = Promise.resolve((res) => {
  return {
    then: function (resolve, reject) {
      resolve(222222);
    },
  };
})
```

- resolve参数的形态：
  - 情况一：参数是一个普通的值或者对象
  - 情况二：参数本身是Promise
  - 情况三：参数是一个thenable



#### **reject方法**

- reject方法类似于resolve方法，只是会将Promise对象的状态设置为reject状态。 

- Promise.reject的用法相当于new Promise，只是会调用reject： 

- Promise.reject传入的参数无论是什么形态，都会直接作为reject状态的参数传递到catch的

```js
// 注意: 无论传入什么值都是一样的
const promise = Promise.reject(new Promise(() => {}));

promise
  .then((res) => {
    console.log("res:", res);
  })
  .catch((err) => {
    console.log("err:", err);
  });

```



#### **all方法**

另外一个类方法是Promise.all： 

- 它的作用是将多个Promise包裹在一起形成一个新的Promise； 

- 新的Promise状态由包裹的所有Promise共同决定：
  - 当所有的Promise状态变成fulfilled状态时，新的Promise状态为fulfilled，并且会将所有Promise的返回值组成一个数组； 
  - 当有一个Promise状态为reject时，新的Promise状态为reject，并且会将第一个reject的返回值作为参数；

```js
// 创建多个Promise
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(11111);
  }, 1000);
});

const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(22222);
  }, 2000);
});

const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(33333);
  }, 3000);
});

// 需求: 所有的Promise都变成fulfilled时, 再拿到结果
// 意外: 在拿到所有结果之前, 有一个promise变成了rejected, 那么整个promise是rejected
Promise.all([p2, p1, p3, "aaaa"])
  .then((res) => {
    console.log(res);
  })
  .catch((err) => {
    console.log("err:", err);
  });

```



#### **allSettled方法**

- 该方法会在所有的Promise都有结果（settled），无论是fulfilled，还是reject时，才会有最终的状态；

- 并且这个Promise的结果一定是fulfilled的；

```js
// allSettled
Promise.allSettled([p1, p2, p3])
  .then((res) => {
    console.log(res);
  })
  .catch((err) => {
    console.log(err);
  });
/*
[
  { status: 'fulfilled', value: 11111 },
  { status: 'rejected', reason: 22222 },
  { status: 'fulfilled', value: 33333 }
]
*/ 
```

- 我们来看一下打印的结果：
  - allSettled的结果是一个数组，数组中存放着每一个Promise的结果，并且是对应一个对象的；
  - 这个对象中包含status状态，以及对应的value值；



#### **race方法**

- 如果有一个Promise有了结果，我们就希望决定最终新Promise的状态，那么可以使用race方法：
  - race是竞技、竞赛的意思，表示多个Promise相互竞争，谁先有结果，那么就使用谁的结果；

```js
// race: 竞技/竞赛
// 只要有一个Promise变成fulfilled状态, 那么就结束
// 意外:
Promise.race([p1, p2, p3])
  .then((res) => {
    console.log("res:", res);
  })
  .catch((err) => {
    console.log("err:", err);
  });
```



#### **any方法**

- any方法是ES12中新增的方法，和race方法是类似的：

  - any方法会等到一个fulfilled状态，才会决定新Promise的状态；

  - 如果所有的Promise都是reject的，那么也会等到所有的Promise都变成rejected状态；

```js
Promise.any([p1, p2, p3])
  .then((res) => {
    console.log("res:", res);
  })
  .catch((err) => {
    console.log("err:", err.errors);
  });
/*
[
    1111,
    22222,
    3333
]
*/
```

- 如果所有的Promise都是reject的，那么会报一个AggregateError的错误



### **手写Promise**



#### 手写基础框架

```js
// 设定promise的状态码
const PROMISE_STATUS_PENDING = "pending";
const PROMISE_STATUS_FULFILLED = "fulfilled";
const PROMISE_STATUS_REJECTED = "rejected";

class HYPromise {
  // 立即执行函数constructor初始化promise
  constructor(executor) {
    // Executor是在创建Promise时需要传入的一个回调函数，这个回调函数会被立即执行 通常我们会在Executor中确定我们的Promise状态
    this.status = PROMISE_STATUS_PENDING;
    this.value = undefined;
    this.reason = undefined;

    // resolve函数
    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 更改状态
        this.status = PROMISE_STATUS_FULFILLED;
        this.value = value;
        console.log("resolve被调用");
      }
    };

    // reject函数
    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        this.status = PROMISE_STATUS_REJECTED;
        this.reason = reason;
        console.log("reject被调用");
      }
    };

    // 返回executor
    executor(resolve, reject);
  }
}

const promise = new HYPromise((resolve, reject) => {
  console.log("状态pending");
  resolve(1111);
  reject(2222);
});
```



#### 手写then方法(初始版)

- then方法会有两个函数，一个成功回调，一个失败回调
- 在调用then方法后会延时调用成功或者失败回调(因为此时在执行constructor初始化还没有调用then方法)

```js
const PROMISE_STATUS_PENDING = "pending";
const PROMISE_STATUS_FULFILLED = "fulfilled";
const PROMISE_STATUS_REJECTED = "rejected";

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING;
    this.value = undefined;
    this.reason = undefined;
    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        this.status = PROMISE_STATUS_FULFILLED;
        // queueMicrotask 延时调用 不然会this.onFulfilled is not a function
        queueMicrotask(() => {
          this.value = value;
          this.onFulfilled(this.value);
        });
      }
    };
    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        this.status = PROMISE_STATUS_REJECTED;
        queueMicrotask(() => {
          console.log("执行onRejected");
          this.reason = reason;
          this.onRejected(this.reason);
        });
      }
    };
    executor(resolve, reject);
  }

  // then方法传入两个回调函数
  then(onFulfilled, onRejected) {
    console.log("执行then方法");
    this.onFulfilled = onFulfilled;
    this.onRejected = onRejected;
  }
}

const promise = new HYPromise((resolve, reject) => {
  console.log("状态pending");
  reject(2222)
  resolve(1111);
});

// 调用then方法
promise.then(
  (res) => {
    console.log("res1:", res);
    return 1111;
  },
  (err) => {
    console.log("err:", err);
  }
);

/*
状态pending
执行then方法
执行onRejected
err: 2222
*/
```



#### 手写then方法(优化一)

- 优化调用then方法不能多次调用

```js
// ES6 ES2015
// https://promisesaplus.com/
const PROMISE_STATUS_PENDING = 'pending'
const PROMISE_STATUS_FULFILLED = 'fulfilled'
const PROMISE_STATUS_REJECTED = 'rejected'

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING
    this.value = undefined
    this.reason = undefined
    // 新增两个Fns来存放多次调用的promise的回调函数
    this.onFulfilledFns = []
    this.onRejectedFns = []

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          // 再次检测promise状态
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_FULFILLED
          this.value = value
          // 轮流执行promise的回调
          this.onFulfilledFns.forEach(fn => {
            fn(this.value)
          })
        });
      }
    }

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return
          this.status = PROMISE_STATUS_REJECTED
          this.reason = reason
          this.onRejectedFns.forEach(fn => {
            fn(this.reason)
          })
        })
      }
    }

    executor(resolve, reject)
  }

  then(onFulfilled, onRejected) {
    // 1.如果在then调用的时候, 状态已经确定下来
    if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
      onFulfilled(this.value)
    }
    if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
      onRejected(this.reason)
    }

    // 2.将成功回调和失败的回调放到数组中
    if (this.status === PROMISE_STATUS_PENDING) {
      this.onFulfilledFns.push(onFulfilled)
      this.onRejectedFns.push(onRejected)
    }
  }
}

const promise = new HYPromise((resolve, reject) => {
  console.log("状态pending")
  resolve(1111) // resolved/fulfilled
  reject(2222)
})
// 调用then方法多次调用
promise.then(res => {
  console.log("res1:", res)
}, err => {
  console.log("err:", err)
})
promise.then(res => {
  console.log("res2:", res)
}, err => {
  console.log("err2:", err)
})
// 在确定Promise状态之后, 再次调用then
setTimeout(() => {
  promise.then(res => {
    console.log("res3:", res)
  }, err => {
    console.log("err3:", err)
  })
}, 1000)
/*
状态pending
res1: 1111
res2: 1111
res3: 1111
*/ 
```



#### 手写then方法(优化二)

- 优化then方法不能链式调用

```js
const PROMISE_STATUS_PENDING = "pending";
const PROMISE_STATUS_FULFILLED = "fulfilled";
const PROMISE_STATUS_REJECTED = "rejected";

// 工具函数 检测execFn能否正常执行，如果不行就抛出异常交给reject执行
function execFunctionWithCatchError(execFn, value, resolve, reject) {
  try {
    const result = execFn(value);
    resolve(result);
  } catch (err) {
    reject(err);
  }
}

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.onFulfilledFns = [];
    this.onRejectedFns = [];

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return;
          this.status = PROMISE_STATUS_FULFILLED;
          this.value = value;
          this.onFulfilledFns.forEach((fn) => {
            fn(this.value);
          });
        });
      }
    };

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return;
          this.status = PROMISE_STATUS_REJECTED;
          this.reason = reason;
          this.onRejectedFns.forEach((fn) => {
            fn(this.reason);
          });
        });
      }
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    // 直接将一个新new出来的promise返回出去
    return new HYPromise((resolve, reject) => {
      // 1.如果在then调用的时候, 状态已经确定下来
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        // try {
        //   const value = onFulfilled(this.value)
        //   resolve(value)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onFulfilled, this.value, resolve, reject);
      }
      if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
        // try {
        //   const reason = onRejected(this.reason)
        //   resolve(reason)
        // } catch(err) {
        //   reject(err)
        // }
        execFunctionWithCatchError(onRejected, this.reason, resolve, reject);
      }

      // 2.将成功回调和失败的回调放到数组中
      if (this.status === PROMISE_STATUS_PENDING) {
        this.onFulfilledFns.push(() => {
          // try {
          //   const value = onFulfilled(this.value)
          //   resolve(value)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onFulfilled, this.value, resolve, reject);
        });
        this.onRejectedFns.push(() => {
          // try {
          //   const reason = onRejected(this.reason)
          //   resolve(reason)
          // } catch(err) {
          //   reject(err)
          // }
          execFunctionWithCatchError(onRejected, this.reason, resolve, reject);
        });
      }
    });
  }
}

const promise = new HYPromise((resolve, reject) => {
  console.log("状态pending");
  // resolve(1111) // resolved/fulfilled
  reject(2222);
  // throw new Error("executor error message")
});

// 调用then方法多次调用
promise
  .then(
    (res) => {
      console.log("res1:", res);
      return "aaaa";
      // 当抛出异常时会交给下一个promise的reject处理
      // throw new Error("err message")
    },
    (err) => {
      console.log("err1:", err);
      return "bbbbb";
      // throw new Error("err message")
    }
  )
  .then(
    (res) => {
      console.log("res2:", res);
    },
    (err) => {
      console.log("err2:", err);
    }
  );
```



#### 手写catch方法

- catch方法直接调用this.then(undefined, onRejected);
- then方法检测一下onRejected是否为undefined，如果是undefined直接抛出异常交给下一个promise的reject处理

```js
 // ES6 ES2015
// https://promisesaplus.com/
const PROMISE_STATUS_PENDING = "pending";
const PROMISE_STATUS_FULFILLED = "fulfilled";
const PROMISE_STATUS_REJECTED = "rejected";

// 工具函数
function execFunctionWithCatchError(execFn, value, resolve, reject) {
  try {
    const result = execFn(value);
    resolve(result);
  } catch (err) {
    reject(err);
  }
}

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.onFulfilledFns = [];
    this.onRejectedFns = [];

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return;
          this.status = PROMISE_STATUS_FULFILLED;
          this.value = value;
          this.onFulfilledFns.forEach((fn) => {
            fn(this.value);
          });
        });
      }
    };

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return;
          this.status = PROMISE_STATUS_REJECTED;
          this.reason = reason;
          this.onRejectedFns.forEach((fn) => {
            fn(this.reason);
          });
        });
      }
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    // 当调用onRejected为underfined时，直接抛出异常交给下一个promise处理
    const defaultOnRejected = (err) => {
      throw err;
    };
    onRejected = onRejected || defaultOnRejected;

    return new HYPromise((resolve, reject) => {
      // 1.如果在then调用的时候, 状态已经确定下来
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        execFunctionWithCatchError(onFulfilled, this.value, resolve, reject);
      }
      if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
        execFunctionWithCatchError(onRejected, this.reason, resolve, reject);
      }

      // 2.将成功回调和失败的回调放到数组中
      if (this.status === PROMISE_STATUS_PENDING) {
        if (onFulfilled)
          this.onFulfilledFns.push(() => {
            execFunctionWithCatchError(
              onFulfilled,
              this.value,
              resolve,
              reject
            );
          });
        if (onRejected)
          this.onRejectedFns.push(() => {
            execFunctionWithCatchError(
              onRejected,
              this.reason,
              resolve,
              reject
            );
          });
      }
    });
  }

  catch(onRejected) {
    this.then(undefined, onRejected);
  }
}

const promise = new HYPromise((resolve, reject) => {
  console.log("状态pending");
  // resolve(1111) // resolved/fulfilled
  reject(2222);
});

// 调用then方法多次调用
promise
  .then((res) => {
    console.log("res:", res);
  })
  .catch((err) => {
    console.log("err:", err);
  });
```



#### 手写finally方法

- 再给onFulfilled也检测一下是否为underfined，如果为underfined则交给下一个promise的resolve处理

```js
// ES6 ES2015
// https://promisesaplus.com/
const PROMISE_STATUS_PENDING = "pending";
const PROMISE_STATUS_FULFILLED = "fulfilled";
const PROMISE_STATUS_REJECTED = "rejected";

// 工具函数
function execFunctionWithCatchError(execFn, value, resolve, reject) {
  try {
    const result = execFn(value);
    resolve(result);
  } catch (err) {
    reject(err);
  }
}

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.onFulfilledFns = [];
    this.onRejectedFns = [];

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return;
          this.status = PROMISE_STATUS_FULFILLED;
          this.value = value;
          this.onFulfilledFns.forEach((fn) => {
            fn(this.value);
          });
        });
      }
    };

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        // 添加微任务
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return;
          this.status = PROMISE_STATUS_REJECTED;
          this.reason = reason;
          this.onRejectedFns.forEach((fn) => {
            fn(this.reason);
          });
        });
      }
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    const defaultOnRejected = (err) => {
      throw err;
    };
    onRejected = onRejected || defaultOnRejected;

    // 在失败回调时将OnFulfilled交给下一个promise来处理
    const defaultOnFulfilled = (value) => {
      return value;
    };
    onFulfilled = onFulfilled || defaultOnFulfilled;

    return new HYPromise((resolve, reject) => {
      // 1.如果在then调用的时候, 状态已经确定下来 
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        execFunctionWithCatchError(onFulfilled, this.value, resolve, reject);
      }
      if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
        execFunctionWithCatchError(onRejected, this.reason, resolve, reject);
      }

      // 2.将成功回调和失败的回调放到数组中
      if (this.status === PROMISE_STATUS_PENDING) {
        if (onFulfilled)
          this.onFulfilledFns.push(() => {
            execFunctionWithCatchError(
              onFulfilled,
              this.value,
              resolve,
              reject
            );
          });
        if (onRejected)
          this.onRejectedFns.push(() => {
            execFunctionWithCatchError(
              onRejected,
              this.reason,
              resolve,
              reject
            );
          });
      }
    });
  }

  catch(onRejected) {
    return this.then(undefined, onRejected);
  }

  finally(onFinally) {
    this.then(
      () => {
        onFinally();
      },
      () => {
        onFinally();
      }
    );
  }
}

const promise = new HYPromise((resolve, reject) => {
  console.log("状态pending");
  resolve(1111); // resolved/fulfilled
  // reject(2222)
});

// 调用then方法多次调用
promise
  .then((res) => {
    console.log("res1:", res);
    return "aaaaa";
  })
  .then((res) => {
    console.log("res2:", res);
  })
  .catch((err) => {
    console.log("err:", err);
  })
  .finally(() => {
    console.log("finally");
  });
```



#### 手写resolve和reject方法

```js
  static resolve(value) {
    return new HYPromise((resolve) => resolve(value));
  }

  static reject(reason) {
    return new HYPromise((resolve, reject) => reject(reason));
  }


HYPromise.resolve("Hello World").then((res) => {
  console.log("res:", res);
});

HYPromise.reject("Error Message").catch((err) => {
  console.log("err:", err);
});

```



#### 手写all和allSettled方法

```js
static all(promises) {
    // 问题关键: 什么时候要执行resolve, 什么时候要执行reject
    return new HYPromise((resolve, reject) => {
      const values = [];
      promises.forEach((promise) => {
        promise.then(
          (res) => {
            values.push(res);
            if (values.length === promises.length) {
              resolve(values);
            }
          },
          (err) => {
            reject(err);
          }
        );
      });
    });
  }

  static allSettled(promises) {
    return new HYPromise((resolve) => {
      const results = [];
      promises.forEach((promise) => {
        promise.then(
          (res) => {
            results.push({ status: PROMISE_STATUS_FULFILLED, value: res });
            if (results.length === promises.length) {
              resolve(results);
            }
          },
          (err) => {
            results.push({ status: PROMISE_STATUS_REJECTED, value: err });
            if (results.length === promises.length) {
              resolve(results);
            }
          }
        ); 
      });
    });
  }


const p1 = new Promise((resolve) => {
  setTimeout(() => {
    resolve(1111);
  }, 1000);
});
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(2222);
  }, 2000);
});
const p3 = new Promise((resolve) => {
  setTimeout(() => {
    resolve(3333);
  }, 3000);
});
HYPromise.all([p1, p2, p3]).then(res => {
  console.log(res)
}).catch(err => {
  console.log(err)
})

HYPromise.allSettled([p1, p2, p3]).then((res) => {
  console.log(res);
});
```



#### 手写raceh和any方法

```js

  static race(promises) {
    return new HYPromise((resolve, reject) => {
      promises.forEach((promise) => {
        // promise.then(res => {
        //   resolve(res)
        // }, err => {
        //   reject(err)
        // })
        promise.then(resolve, reject);
      });
    });
  }

  static any(promises) {
    // resolve必须等到有一个成功的结果
    // reject所有的都失败才执行reject
    const reasons = [];
    return new HYPromise((resolve, reject) => {
      promises.forEach((promise) => {
        promise.then(resolve, (err) => {
          reasons.push(err);
          if (reasons.length === promises.length) {
            reject(new AggregateError(reasons));
          }
        });
      });
    });
  }


const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(1111);
  }, 3000);
});
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(2222);
  }, 2000);
});
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(3333);
  }, 3000);
});

HYPromise.race([p1, p2, p3]).then(res => {
  console.log("res:", res)
}).catch(err => {
  console.log("err:", err)
})

HYPromise.any([p1, p2, p3])
  .then((res) => {
    console.log("res:", res);
  })
  .catch((err) => {
    console.log("err:", err.errors);
  })
```





#### **总结**



**一. Promise规范**

https://promisesaplus.com/

**二. Promise类设计**

```js
class HYPromise {}
function HYPromise() {}
```

**三. 构造函数的规划**

```js
class HYPromise {
  constructor(executor) {
   	// 定义状态
    // 定义resolve、reject回调
    // resolve执行微任务队列：改变状态、获取value、then传入执行成功回调
    // reject执行微任务队列：改变状态、获取reason、then传入执行失败回调
    
    // try catch
    executor(resolve, reject)
  }
}
```

**四. then方法的实现**

```js
class HYPromise {
  then(onFulfilled, onRejected) {
    // this.onFulfilled = onFulfilled
    // this.onRejected = onRejected
    
    // 1.判断onFulfilled、onRejected，会给默认值
    
    // 2.返回Promise resolve/reject
    
    // 3.判断之前的promise状态是否确定
    // onFulfilled/onRejected直接执行（捕获异常）
    
    // 4.添加到数组中push(() => { 执行 onFulfilled/onRejected 直接执行代码})
  }
}
```

**五. catch方法**

```js
class HYPromise {
  catch(onRejected) {
    return this.then(undefined, onRejected)
  }
}
```

**六. finally**

```js
class HYPromise {
  finally(onFinally) {
    return this.then(() => {onFinally()}, () => {onFinally()})
  }
}
```

**七. resolve/reject**

**八. all/allSettled**

核心：要知道new Promise的resolve、reject在什么情况下执行

all：

- 情况一：所有的都有结果
- 情况二：有一个reject

allSettled：

- 情况：所有都有结果，并且一定执行resolve

**九.race/any**

race:

- 情况：只要有结果

any:

- 情况一：必须等到一个resolve结果
- 情况二：都没有resolve，所有的都是reject



#### 实现代码

```js
const PROMISE_STATUS_PENDING = "pending";
const PROMISE_STATUS_FULFILLED = "fulfilled";
const PROMISE_STATUS_REJECTED = "rejected";

function execFunctionWithCatchError(execFn, value, resolve, reject) {
  try {
    const result = execFn(value);
    resolve(result);
  } catch (err) {
    reject(err);
  }
}

class HYPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.onFulfilledFns = [];
    this.onRejectedFns = [];

    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return;
          this.status = PROMISE_STATUS_FULFILLED;
          this.value = value;
          this.onFulfilledFns.forEach((fn) => {
            fn(this.value);
          });
        });
      }
    };

    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return;
          this.status = PROMISE_STATUS_REJECTED;
          this.reason = reason;
          this.onRejectedFns.forEach((fn) => {
            fn(this.reason);
          });
        });
      }
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    const defaultOnRejected = (err) => {
      throw err;
    };
    onRejected = onRejected || defaultOnRejected;

    const defaultOnFulfilled = (value) => {
      return value;
    };
    onFulfilled = onFulfilled || defaultOnFulfilled;

    return new HYPromise((resolve, reject) => {
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        execFunctionWithCatchError(onFulfilled, this.value, resolve, reject);
      }
      if (this.status === PROMISE_STATUS_REJECTED && onRejected) {
        execFunctionWithCatchError(onRejected, this.reason, resolve, reject);
      }
      if (this.status === PROMISE_STATUS_PENDING) {
        if (onFulfilled)
          this.onFulfilledFns.push(() => {
            execFunctionWithCatchError(
              onFulfilled,
              this.value,
              resolve,
              reject
            );
          });
        if (onRejected)
          this.onRejectedFns.push(() => {
            execFunctionWithCatchError(
              onRejected,
              this.reason,
              resolve,
              reject
            );
          });
      }
    });
  }

  catch(onRejected) {
    return this.then(undefined, onRejected);
  }

  finally(onFinally) {
    this.then(
      () => {
        onFinally();
      },
      () => {
        onFinally();
      }
    );
  }

  static resolve(value) {
    return new HYPromise((resolve) => resolve(value));
  }

  static reject(reason) {
    return new HYPromise((resolve, reject) => reject(reason));
  }

  static all(promises) {
    return new HYPromise((resolve, reject) => {
      const values = [];
      promises.forEach((promise) => {
        promise.then(
          (res) => {
            values.push(res);
            if (values.length === promises.length) {
              resolve(values);
            }
          },
          (err) => {
            reject(err);
          }
        );
      });
    });
  }

  static allSettled(promises) {
    return new HYPromise((resolve) => {
      const results = [];
      promises.forEach((promise) => {
        promise.then(
          (res) => {
            results.push({ status: PROMISE_STATUS_FULFILLED, value: res });
            if (results.length === promises.length) {
              resolve(results);
            }
          },
          (err) => {
            results.push({ status: PROMISE_STATUS_REJECTED, value: err });
            if (results.length === promises.length) {
              resolve(results);
            }
          }
        );
      });
    });
  }

  static race(promises) {
    return new HYPromise((resolve, reject) => {
      promises.forEach((promise) => {
        // promise.then(res => {
        //   resolve(res)
        // }, err => {
        //   reject(err)
        // })
        promise.then(resolve, reject);
      });
    });
  }

  static any(promises) {
    // resolve必须等到有一个成功的结果
    // reject所有的都失败才执行reject
    const reasons = [];
    return new HYPromise((resolve, reject) => {
      promises.forEach((promise) => {
        promise.then(resolve, (err) => {
          reasons.push(err);
          if (reasons.length === promises.length) {
            reject(new AggregateError(reasons));
          }
        });
      });
    });
  }
}

```





## 十一、迭代器和生成器



### 迭代器

- **迭代器**（iterator），是确使用户可在容器对象（container，例如链表或数组）上遍访的对象，使用该接口无需关心对象的内部实现细节。
  - 其行为像数据库中的光标，迭代器最早出现在1974年设计的CLU编程语言中；
  - 在各种编程语言的实现中，迭代器的实现方式各不相同，但是基本都有迭代器，比如Java、Python等；

- 从迭代器的定义我们可以看出来，迭代器是帮助我们对某个数据结构进行遍历的对象。

- 在JavaScript中，迭代器也是一个具体的对象，这个对象需要符合迭代器协议（iterator protocol）：
  - 迭代器协议定义了产生一系列值（无论是有限还是无限个）的标准方式；
  - 那么在js中这个标准就是一个特定的next方法；

- next方法有如下的要求：
  - 一个无参数或者一个参数的函数，返回一个应当拥有以下两个属性的对象：
  - **done（boolean）** 
    - 如果迭代器可以产生序列中的下一个值，则为 false。（这等价于没有指定 done 这个属性。）
    - 如果迭代器已将序列迭代完毕，则为 true。这种情况下，value 是可选的，如果它依然存在，即为迭代结束之后的默认返回值。
  - **value**
    - 迭代器返回的任何 JavaScript 值。done 为 true 时可省略

```js
// 编写的一个迭代器
const iterator = {
  next: function() {
    return { done: true, value: 123 }
  }
}

// 数组
const names = ["abc", "cba", "nba"]

// 创建一个迭代器对象来访问数组
let index = 0

const namesIterator = {
  next: function() {
    if (index < names.length) {
      return { done: false, value: names[index++] }
    } else {
      return { done: true, value: undefined }
    }
  }
}

console.log(namesIterator.next()) // { done: false, value: "nba" }
console.log(namesIterator.next()) // { done: true, value: undefined }
```



#### **可迭代对象**

- 但是上面的代码整体来说看起来是有点奇怪的：
  - 我们获取一个数组的时候，需要自己创建一个index变量，再创建一个所谓的迭代器对象；
  - 事实上我们可以对上面的代码进行进一步的封装，让其变成一个可迭代对象；

- 什么又是可迭代对象呢？
  - 它和迭代器是不同的概念；
  - 当一个对象实现了iterable protocol协议时，它就是一个可迭代对象；
  - 这个对象的要求是必须实现 @@iterator 方法，在代码中我们使用 Symbol.iterator 访问该属性；

- 当我们要问一个问题，我们转成这样的一个东西有什么好处呢？
  - 当一个对象变成一个可迭代对象的时候，进行某些迭代操作，比如 for...of 操作时，其实就会调用它的@@iterator 方法；

![image-20220914125600655](http://lyc-markdownimg.test.upcdn.net/img/202209141256822.png)

```js
// 创建一个迭代器对象来访问数组
const iterableObj = {
  names: ["abc", "cba", "nba"],
  [Symbol.iterator]: function() {
    let index = 0
    return {
      next: () => {
        if (index < this.names.length) {
          return { done: false, value: this.names[index++] }
        } else {
          return { done: true, value: undefined }
        }
      }
    }
  }
}
```



#### **原生迭代器对象**

事实上我们平时创建的很多原生对象已经实现了可迭代协议，会生成一个迭代器对象的：

- String、Array、Map、Set、arguments对象、NodeList集合；

```js
const names = ["abc", "cba", "nba"]
console.log(names[Symbol.iterator])

const iterator1 = names[Symbol.iterator]()
console.log(iterator1.next())
console.log(iterator1.next())

for (const item of names) {
  console.log(item)
}

// Map/Set
const set = new Set()
set.add(10)
set.add(100)
set.add(1000)

console.log(set[Symbol.iterator])

for (const item of set) {
  console.log(item)
}

// 函数中arguments也是一个可迭代对象
function foo(x, y, z) {
  console.log(arguments[Symbol.iterator])
  for (const arg of arguments) {
    console.log(arg)
  }
}

foo(10, 20, 30)
```



#### **可迭代对象的应用**

那么这些东西可以被用在哪里呢？

-  JavaScript中语法：for ...of、展开语法（spread syntax）、yield*（后面讲）、解构赋值（Destructuring_assignment）；
-  创建一些对象时：new Map([Iterable])、new WeakMap([iterable])、new Set([iterable])、new WeakSet([iterable]);
-  一些方法的调用：Promise.all(iterable)、Promise.race(iterable)、Array.from(iterable);

```js
// 1.for of场景

// 2.展开语法(spread syntax)
const iterableObj = {
  names: ["abc", "cba", "nba"],
  [Symbol.iterator]: function () {
    let index = 0
    return {
      next: () => {
        if (index < this.names.length) {
          return { done: false, value: this.names[index++] }
        } else {
          return { done: true, value: undefined }
        }
      }
    }
  }
}

const names = ["abc", "cba", "nba"]
const newNames = [...names, ...iterableObj]
console.log(newNames)

const obj = { name: "why", age: 18 }
// for (const item of obj) {

// }
// ES9(ES2018)中新增的一个特性: 用的不是迭代器
const newObj = { ...obj }
console.log(newObj)


// 3.解构语法
const [name1, name2] = names
// const { name, age } = obj 不一样ES9新增的特性

// 4.创建一些其他对象时
const set1 = new Set(iterableObj)
const set2 = new Set(names)

const arr1 = Array.from(iterableObj)

// 5.Promise.all
Promise.all(iterableObj).then(res => {
  console.log(res)
})
```



#### **自定义类的迭代**

在前面我们看到Array、Set、String、Map等类创建出来的对象都是可迭代对象：

- 在面向对象开发中，我们可以通过class定义一个自己的类，这个类可以创建很多的对象：

- 如果我们也希望自己的类创建出来的对象默认是可迭代的，那么在设计类的时候我们就可以添加上@@iterator 方法；

```js
// 案例: 创建一个教室类, 创建出来的对象都是可迭代对象
class Classroom {
  constructor(address, name, students) {
    this.address = address
    this.name = name
    this.students = students
  }

  entry(newStudent) {
    this.students.push(newStudent)
  }

  [Symbol.iterator]() {
    let index = 0
    return {
      next: () => {
        if (index < this.students.length) {
          return { done: false, value: this.students[index++] }
        } else {
          return { done: true, value: undefined }
        }
      },
      return: () => {
        console.log("迭代器提前终止了~")
        return { done: true, value: undefined }
      }
    }
  }
}

const classroom = new Classroom("3幢5楼205", "计算机教室", ["james", "kobe", "curry", "why"])
classroom.entry("lilei")

for (const stu of classroom) {
  console.log(stu)
  if (stu === "why") break
}
/*
james
kobe
curry
why
迭代器提前终止了~
*/
```



#### **迭代器的中断**

- 迭代器在某些情况下会在没有完全迭代的情况下中断：
  - 比如遍历的过程中通过break、continue、return、throw中断了循环操作；
  - 比如在解构的时候，没有解构所有的值；

- 那么这个时候我们想要监听中断的话，可以添加return方法

```js
[Symbol.iterator]() {
    let index = 0
    return {
      next: () => {
        if (index < this.students.length) {
          return { done: false, value: this.students[index++] }
        } else {
          return { done: true, value: undefined }
        }
      },
      return: () => {
        console.log("迭代器提前终止了~")
        return { done: true, value: undefined }
      }
    }
}
```





### **生成器**

- 生成器是ES6中新增的一种函数控制、使用的方案，它可以让我们更加灵活的控制函数什么时候继续执行、暂停执行等。

- 平时我们会编写很多的函数，这些函数终止的条件通常是返回值或者发生了异常。

- **生成器函数也是一个函数，但是和普通的函数有一些区别：** 
  - 首先，生成器函数需要在function的后面加一个符号：* 
  - 其次，生成器函数可以通过yield关键字来控制函数的执行流程： 
  - 最后，生成器函数的返回值是一个Generator（生成器）：
    - 生成器事实上是一种特殊的迭代器；



#### **生成器函数执行**

我们发现上面的生成器函数foo的执行体压根没有执行，它只是返回了一个生成器对象。

- 那么我们如何可以让它执行函数中的东西呢？调用next即可； 

- 我们之前学习迭代器时，知道迭代器的next是会有返回值的；

- 但是我们很多时候不希望next返回的是一个undefined，这个时候我们可以通过yield来返回结果；

```js
// 当遇到yield时候值暂停函数的执行
// 当遇到return时候生成器就停止执行
function* foo() {
  console.log("函数开始执行~")

  const value1 = 100
  console.log("第一段代码:", value1)
  yield value1

  const value2 = 200
  console.log("第二段代码:", value2)
  yield value2

  const value3 = 300
  console.log("第三段代码:", value3)
  yield value3

  console.log("函数执行结束~")
  return "123"
}

// 调用生成器函数时, 会给我们返回一个生成器对象
// generator本质上是一个特殊的iterator 
const generator = foo()
// 开始执行第一段代码
console.log("返回值1:", generator.next())
console.log("返回值2:", generator.next())
console.log("返回值3:", generator.next())
console.log("返回值3:", generator.next())

/*
函数开始执行~
第一段代码: 100
返回值1: { value: 100, done: false }
第二段代码: 200
返回值2: { value: 200, done: false }
第三段代码: 300
返回值3: { value: 300, done: false }
函数执行结束~
返回值3: { value: '123', done: true }
*/
```



#### **生成器传递参数 – next函数**

函数既然可以暂停来分段执行，那么函数应该是可以传递参数的，我们是否可以给每个分段来传递参数呢？

- 答案是可以的；

- 我们在调用next函数的时候，可以给它传递参数，那么这个参数会作为上一个yield语句的返回值；

- 注意：也就是说我们是为本次的函数代码块执行提供了一个值；

```js
function* foo(num) {
  console.log("函数开始执行~")

  const value1 = 100 * num
  console.log("第一段代码:", value1)
  const n = yield value1

  const value2 = 200 * n
  console.log("第二段代码:", value2)
  const count = yield value2

  const value3 = 300 * count
  console.log("第三段代码:", value3)
  yield value3

  console.log("函数执行结束~")
  return "123"
}

// 生成器上的next方法可以传递参数
const generator = foo(5)
console.log(generator.next())
// 第二段代码, 第二次调用next的时候执行的
console.log(generator.next(10))
console.log(generator.next(25))

/*
函数开始执行~
第一段代码: 500
{ value: 500, done: false }
第二段代码: 2000
{ value: 2000, done: false }
第三段代码: 7500
{ value: 7500, done: false }
*/
```



#### **生成器提前结束 – return函数**

还有一个可以给生成器函数传递参数的方法是通过return函数：

- return传值后这个生成器函数就会结束，之后调用next不会继续生成值了；

```js
function* foo(num) {
  console.log("函数开始执行~")

  const value1 = 100 * num
  console.log("第一段代码:", value1)
  const n = yield value1

  const value2 = 200 * n
  console.log("第二段代码:", value2)
  const count = yield value2

  const value3 = 300 * count
  console.log("第三段代码:", value3)
  yield value3

  console.log("函数执行结束~")
  return "123"
}

const generator = foo(10)

console.log(generator.next())

// 第二段代码的执行, 使用了return
// 那么就意味着相当于在第一段代码的后面加上return, 就会提前终端生成器函数代码继续执行
console.log(generator.return(15))
console.log(generator.next())
console.log(generator.next())
/*
函数开始执行~
第一段代码: 1000
{ value: 1000, done: false }
{ value: 15, done: true }
{ value: undefined, done: true }
{ value: undefined, done: true }
*/
```



#### **生成器抛出异常 – throw函数**

除了给生成器函数内部传递参数之外，也可以给生成器函数内部抛出异常：

- 抛出异常后我们可以在生成器函数中捕获异常；

- 但是在catch语句中不能继续yield新的值了，但是可以在catch语句外使用yield继续中断函数的执行；

```js
function* foo() {
  console.log("代码开始执行~")
  const value1 = 100
  try {
    yield value1
  } catch (error) {
    console.log("捕获到异常情况:", error)
    yield "abc"
  }
  console.log("第二段代码继续执行")
  const value2 = 200
  yield value2
  console.log("代码执行结束~")
}

const generator = foo()

const result = generator.next()
generator.throw("error message")

/*
代码开始执行~
捕获到异常情况: error message
*/ 
```



#### **生成器替代迭代器**

- 我们发现生成器是一种特殊的迭代器，那么在某些情况下我们可以使用生成器来替代迭代器： 

- 事实上我们还可以使用yield*来生产一个可迭代对象：
  - 这个时候相当于是一种yield的语法糖，只不过会依次迭代这个可迭代对象，每次迭代其中的一个值；

```js
// 1.生成器来替代迭代器
function* createArrayIterator(arr) {

  // 3.第三种写法 yield*
  yield* arr

  // 2.第二种写法
  for (const item of arr) {
    yield item
  }
  // 1.第一种写法
  yield "abc" // { done: false, value: "abc" }
  yield "cba" // { done: false, value: "abc" }
  yield "nba" // { done: false, value: "abc" }
}
```



创建一个函数, 这个函数可以迭代一个范围内的数字：

```js
// 2.创建一个函数, 这个函数可以迭代一个范围内的数字
// 10 20
function* createRangeIterator(start, end) {
  let index = start
  while (index < end) {
    yield index++
  }
```



在之前的自定义类迭代中，我们也可以换成生成器：

```js
// 3.class案例
class Classroom {
  constructor(address, name, students) {
    this.address = address
    this.name = name
    this.students = students
  }
  entry(newStudent) {
    this.students.push(newStudent)
  }
  foo = () => {
    console.log("foo function")
  }
  // [Symbol.iterator] = function*() {
  //   yield* this.students
  // }
  *[Symbol.iterator]() {
    yield* this.students
  }
}

const classroom = new Classroom("3幢", "1102", ["abc", "cba"])
for (const item of classroom) {
  console.log(item)
}
/*
abc
cba
*/
```



### **异步处理方案**

- 学完了我们前面的Promise、生成器等，我们目前来看一下异步代码的最终处理方案。

- 需求：
  - 我们需要向服务器发送网络请求获取数据，一共需要发送三次请求；
  - 第二次的请求url依赖于第一次的结果；
  - 第三次的请求url依赖于第二次的结果；
  - 依次类推；

```js
// request.js
function requestData(url) {
  // 异步请求的代码会被放入到executor中
  return new Promise((resolve, reject) => {
    // 模拟网络请求
    setTimeout(() => {
      // 拿到请求的结果
      resolve(url)
    }, 2000);
  })
}

// 需求: 
// 1> url: why -> res: why
// 2> url: res + "aaa" -> res: whyaaa
// 3> url: res + "bbb" => res: whyaaabbb

// 1.第一种方案: 多次回调
// 回调地狱
requestData("why").then(res => {
  requestData(res + "aaa").then(res => {
    requestData(res + "bbb").then(res => {
      console.log(res)
    })
  })
})


// 2.第二种方案: Promise中then的返回值来解决
requestData("why").then(res => {
  return requestData(res + "aaa")
}).then(res => {
  return requestData(res + "bbb")
}).then(res => {
  console.log(res)
})

// 3.第三种方案: Promise + generator实现
function* getData() {
  const res1 = yield requestData("why")
  const res2 = yield requestData(res1 + "aaa")
  const res3 = yield requestData(res2 + "bbb")
  const res4 = yield requestData(res3 + "ccc")
  console.log(res4)
}
```



**自动执行generator函数**

- 目前我们的写法有两个问题：
  - 第一，我们不能确定到底需要调用几层的Promise关系； 
  - 第二，如果还有其他需要这样执行的函数，我们应该如何操作呢？

- 所以，我们可以封装一个工具函数execGenerator自动执行生成器函数：

```js
function* getData() {
  const res1 = yield requestData("why")
  const res2 = yield requestData(res1 + "aaa")
  const res3 = yield requestData(res2 + "bbb")
  const res4 = yield requestData(res3 + "ccc")
  console.log(res4)
}

// 1> 手动执行生成器函数
const generator = getData()
generator.next().value.then(res => {
  generator.next(res).value.then(res => {
    generator.next(res).value.then(res => {
      generator.next(res)
    })
  })
})

// 2> 自己封装了一个自动执行的函数
function execGenerator(genFn) {
  const generator = genFn()
  function exec(res) {
    const result = generator.next(res)
    if (result.done) {
      return result.value
    }
    result.value.then(res => {
      exec(res)
    })
  }
  exec()
}

execGenerator(getData)

// 3> 第三方包co自动执行
const co = require('co')
co(getData)


// 4.第四种方案: async/await
async function getData() {
  const res1 = await requestData("why")
  const res2 = await requestData(res1 + "aaa")
  const res3 = await requestData(res2 + "bbb")
  const res4 = await requestData(res3 + "ccc")
  console.log(res4)
}

getData()
```





#### **异步函数 async function**

- async关键字用于声明一个异步函数：
  - async是asynchronous单词的缩写，异步、非同步；
  - sync是synchronous单词的缩写，同步、同时；

- async异步函数可以有很多中写法：

```js
// await/async
async function foo1() {

}

const foo2 = async () => {

}

class Foo {
  async bar() {

  }
}
```



#### **异步函数的执行流程**

- 异步函数的内部代码执行过程和普通的函数是一致的，默认情况下也是会被同步执行。

- **异步函数有返回值时，和普通函数会有区别：**
  - 情况一：异步函数也可以有返回值，但是异步函数的返回值会被包裹到Promise.resolve中；
  - 情况二：如果我们的异步函数的返回值是Promise，Promise.resolve的状态会由Promise决定；
  - 情况三：如果我们的异步函数的返回值是一个对象并且实现了thenable，那么会由对象的then方法来决定；

```js
async function foo() {
  console.log("foo function start~")
  console.log("中间代码~")
  console.log("foo function end~")

  // 1.返回一个值
  // return aaa

  // 2.返回thenable
  return {
    then: function(resolve, reject) {
      resolve("hahahah")
    }
  }

  // 3.返回Promise
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("hehehehe")
    }, 2000)
  })
}

// 异步函数的返回值一定是一个Promise
const promise = foo()
promise.then(res => {
  console.log("promise then function exec:", res)
})

/*
foo function start~
中间代码~
foo function end~
promise then function exec: hehehehe
*/
```

- 如果我们在async中抛出了异常，那么程序它并不会像普通函数一样报错，而是会作为Promise的reject来传递；

```js
async function foo() {
  console.log("foo function start~")
  console.log("中间代码~")
  // 异步函数中的异常, 会被作为异步函数返回的Promise的reject值的
  throw new Error("error message")

  console.log("foo function end~")
}

// 异步函数的返回值一定是一个Promise
foo().catch(err => {
  console.log("coderwhy err:", err)
})

console.log("后续还有代码~~~~~")

```



#### **await关键字**

- async函数另外一个特殊之处就是可以在它内部使用await关键字，而普通函数中是不可以的。

- await关键字有什么特点呢？
  - 通常使用await是后面会跟上一个表达式，这个表达式会返回一个Promise； 
  - 那么await会等到Promise的状态变成fulfilled状态，之后继续执行异步函数； 

- 如果await后面是一个普通的值，那么会直接返回这个值； 

- 如果await后面是一个thenable的对象，那么会根据对象的then方法调用来决定后续的值； 

- 如果await后面的表达式，返回的Promise是reject的状态，那么会将这个reject结果直接作为函数的Promise的reject值；

```js
// 1.await更上表达式
function requestData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      // resolve(222)
      reject(1111)
    }, 2000);
  })
}

// async function foo() {
//   const res1 = await requestData()
//   console.log("后面的代码1", res1)
//   console.log("后面的代码2")
//   console.log("后面的代码3")

//   const res2 = await requestData()
//   console.log("res2后面的代码", res2)
// }

// 2.跟上其他的值
async function foo() {
  // const res1 = await 123
  // const res1 = await {
  //   then: function(resolve, reject) {
  //     resolve("abc")
  //   }
  // }
  const res1 = await new Promise((resolve) => {
    resolve("why")
  })
  console.log("res1:", res1)
}

// 3.reject值
async function foo() {
  const res1 = await requestData()
  console.log("res1:", res1)
}

foo().catch(err => {
  console.log("err:", err)
})
```



## 十二、事件循环

### **进程和线程**

- **线程和进程是操作系统中的两个概念：**
  - 进程（process）：计算机已经运行的程序，是操作系统管理程序的一种方式；
  - 线程（thread）：操作系统能够运行运算调度的最小单位，通常情况下它被包含在进程中；

- **听起来很抽象，这里还是给出我的解释：**
  - 进程：我们可以认为，启动一个应用程序，就会默认启动一个进程（也可能是多个进程）；
  - 线程：每一个进程中，都会启动至少一个线程用来执行程序中的代码，这个线程被称之为主线程； 
  - 所以我们也可以说进程是线程的容器；

- **再用一个形象的例子解释：**
  - 操作系统类似于一个大工厂；
  - 工厂中里有很多车间，这个车间就是进程；
  - 每个车间可能有一个以上的工人在工厂，这个工人就是线程；

![image-20220919001224723](http://lyc-markdownimg.test.upcdn.net/img/202209190012826.png)

- **操作系统是如何做到同时让多个进程（边听歌、边写代码、边查阅资料）同时工作**呢？
  - 这是因为CPU的运算速度非常快，它可以快速的在多个进程之间迅速的切换； 
  - 当我们进程中的线程获取到时间片时，就可以快速执行我们编写的代码； 
  - 对于用户来说是感受不到这种快速的切换的



### **浏览器中的JavaScript线程**

- 我们经常会说**JavaScript是单线程**的，但是**JavaScript的线程应该有自己的容器进程**：浏览器或者Node。 

- **浏览器是一个进程吗，它里面只有一个线程吗？**
  - 目前多数的浏览器其实都是多进程的，当我们打开一个tab页面时就会开启一个新的进程，这是为了防止一个页面卡死而造成所有页面无法响应，整个浏览器需要强制退出；
  - 每个进程中又有很多的线程，其中包括执行JavaScript代码的线程； 

- **JavaScript的代码执行是在一个单独的线程中执行的：**
  - 这就意味着JavaScript的代码，在同一个时刻只能做一件事； 
  - 如果这件事是非常耗时的，就意味着当前的线程就会被阻塞； 

- **所以真正耗时的操作，实际上并不是由JavaScript线程在执行的：**
  - 浏览器的每个进程是多线程的，那么其他线程可以来完成这个耗时的操作； 
  - 比如网络请求、定时器，我们只需要在特性的时候执行应该有的回调即可；



### **浏览器的事件循环**

- **如果在执行JavaScript代码的过程中，有异步操作呢？**
  - 中间我们插入了一个setTimeout的函数调用；
  - 这个函数被放到入调用栈中，执行会立即结束，并不会阻塞后续代码的执行；

![image-20220919001443262](http://lyc-markdownimg.test.upcdn.net/img/202209190014369.png)



### **宏任务和微任务**

- **但是事件循环中并非只维护着一个队列，事实上是有两个队列：**
  - 宏任务队列（macrotask queue）：ajax、setTimeout、setInterval、DOM监听、UI Rendering等 
  - 微任务队列（microtask queue）：Promise的then回调、 Mutation Observer API、queueMicrotask()等 

- **那么事件循环对于两个队列的优先级是怎么样的呢？**
  1. main script中的代码优先执行（编写的顶层script代码）；
  2. 在执行任何一个宏任务之前（不是队列，是一个宏任务），都会先查看微任务队列中是否有任务需要执行
     - 也就是宏任务执行之前，必须保证微任务队列是空的；
     - 如果不为空，那么就优先执行微任务队列中的任务（回调）；

```js
setTimeout(() => {
  console.log("setTimeout")
}, 1000)

queueMicrotask(() => {
  console.log("queueMicrotask")
})

Promise.resolve().then(() => {
  console.log("Promise then")
})

function foo() {
  console.log("foo")
}

function bar() {
  console.log("bar")
  foo()
}

bar()

console.log("其他代码")
// bar
// foo
// 其他代码
// queueMicrotask
// Promise then
// setTimeout
```



### **Node的事件循环**

-  浏览器中的EventLoop是根据HTML5定义的规范来实现的，不同的浏览器可能会有不同的实现，而Node中是由libuv实现的。 

-  这里我们来给出一个Node的架构图： 
  - 我们会发现libuv中主要维护了一个EventLoop和worker threads（线程池）；
  - EventLoop负责调用系统的一些其他操作：文件的IO、Network、child-processes等 

- libuv是一个多平台的专注于异步IO的库，它最初是为Node开发的，但是现在也被使用到Luvit、Julia、pyuv等其他地方；

![image-20220919001913199](http://lyc-markdownimg.test.upcdn.net/img/202209190019295.png)



### **Node事件循环的阶段**

- 我们最前面就强调过，**事件循环像是一个桥梁**，是连接着应用程序的JavaScript和系统调用之间的通道： 
  - 无论是我们的文件IO、数据库、网络IO、定时器、子进程，在完成对应的操作后，都会将对应的结果和回调函数放到事件循环（任务队列）中；
  - 事件循环会不断的从任务队列中取出对应的事件（回调函数）来执行；

- **但是一次完整的事件循环Tick分成很多个阶段：**
  - 定时器（Timers）**：本阶段执行已经被 setTimeout() 和 setInterval() 的调度回调函数。**
  - 待定回调（Pending Callback）**：对某些系统操作（如TCP错误类型）执行回调，比如TCP连接时接收到 ECONNREFUSED。 **
  - idle, prepare**：仅系统内部使用。**
  - 轮询（Poll）**：检索新的 I/O 事件；执行与 I/O 相关的回调；**
  - 检测（check）**：setImmediate() 回调函数在这里执行。**
  - 关闭的回调函数**：一些关闭的回调函数，如：socket.on('close', ...)。

![image-20220919002025824](http://lyc-markdownimg.test.upcdn.net/img/202209190020926.png)



### **Node的宏任务和微任务**

- 我们会发现从一次事件循环的Tick来说，Node的事件循环更复杂，它也分为微任务和宏任务：
  - 宏任务（macrotask）：setTimeout、setInterval、IO事件、setImmediate、close事件；
  - 微任务（microtask）：Promise的then回调、process.nextTick、queueMicrotask； 

- 但是，Node中的事件循环不只是 微任务队列和 宏任务队列：

  - 微任务队列：

    -  next tick queue：process.nextTick； 

    - other queue：Promise的then回调、queueMicrotask； 

  - 宏任务队列：

    -  timer queue：setTimeout、setInterval； 
    -  poll queue：IO事件
    -  check queue：setImmediate； 
    -  close queue：close事件；

- 所以，在每一次事件循环的tick中，会按照如下顺序来执行代码：
  - next tick microtask queue； 
  - other microtask queue； 
  - timer queue； 
  - poll queue； 
  - check queue； 
  - close queue；



### **Promise面试题**

#### **第一题**

```js
setTimeout(function () {
  console.log("setTimeout1");
  new Promise(function (resolve) {
    resolve();
  }).then(function () {
    new Promise(function (resolve) {
      resolve();
    }).then(function () {
      console.log("then4");
    });
    console.log("then2");
  });
});

new Promise(function (resolve) {
  console.log("promise1");
  resolve();
}).then(function () {
  console.log("then1");
});

setTimeout(function () {
  console.log("setTimeout2");
});

console.log(2);

queueMicrotask(() => {
  console.log("queueMicrotask1")
});

new Promise(function (resolve) {
  resolve();
}).then(function () {
  console.log("then3");
});

// promise1
// 2
// then1
// queueMicrotask1
// then3
// setTimeout1
// then2
// then4
// setTimeout2

```

![image-20220918181055458](http://lyc-markdownimg.test.upcdn.net/img/202209181810661.png)

#### 第二题

```js
async function async1() {
  console.log('async1 start')
  await async2();
  console.log('async1 end')
}

async function async2() {
  console.log('async2')
}

console.log('script start')

setTimeout(function () {
  console.log('setTimeout')
}, 0)

async1();

new Promise(function (resolve) {
  console.log('promise1')
  resolve();
}).then(function () {
  console.log('promise2')
})

console.log('script end')
// script start
// async1 start
// async2
// promise1
// script end
// async1 end
// promise2
// setTimeout
```

![image-20220918181922854](http://lyc-markdownimg.test.upcdn.net/img/202209181819974.png)



#### 第三题

```js
Promise.resolve().then(() => {
  console.log(0);
  // 1.直接return一个值 相当于resolve(4)
  // return 4

  // 2.return thenable的值
  return {
    then: function(resolve) {
      // 大量的计算
      resolve(4)
    }
  }

  // 3.return Promise
  // 不是普通的值, 多加一次微任务
  // Promise.resolve(4), 多加一次微任务
  // 一共多加两次微任务
  return Promise.resolve(4)
}).then((res) => {
  console.log(res)
})

Promise.resolve().then(() => {
  console.log(1);
}).then(() => {
  console.log(2);
}).then(() => {
  console.log(3);
}).then(() => {
  console.log(5);
}).then(() =>{
  console.log(6);
})

// 1.return 4
// 0
// 1
// 4
// 2
// 3
// 5
// 6

// 2.return thenable
// 0
// 1
// 2
// 4
// 3
// 5
// 6

// 3.return promise
// 0
// 1
// 2
// 3
// 4
// 5
// 6
```

![image-20220918201744568](http://lyc-markdownimg.test.upcdn.net/img/202209182017681.png)



![image-20220918204323169](http://lyc-markdownimg.test.upcdn.net/img/202209182043343.png)

#### 第四题（node环境）

```js
async function async1() {
  console.log('async1 start')
  await async2()
  console.log('async1 end')
}

async function async2() {
  console.log('async2')
}

console.log('script start')

setTimeout(function () {
  console.log('setTimeout0')
}, 0)

setTimeout(function () {
  console.log('setTimeout2')
}, 300)

setImmediate(() => console.log('setImmediate'));

process.nextTick(() => console.log('nextTick1'));

async1();

process.nextTick(() => console.log('nextTick2'));

new Promise(function (resolve) {
  console.log('promise1')
  resolve();
  console.log('promise2')
}).then(function () {
  console.log('promise3')
})

console.log('script end')

// script start
// async1 start
// async2
// promise1
// promise2
// script end
// nexttick1
// nexttick2
// async1 end
// promise3
// settimetout0
// setImmediate
// setTimeout2

```



## 其他补充知识

### **阅读源码**

阅读源码大家遇到的最大的问题：

1. 一定不要浮躁

2. 看到后面忘记前面的东西

   Bookmarks的打标签的工具：command(ctrl) + alt + k 

   读一个函数的时候忘记传进来是什么？

3. 读完一个函数还是不知道它要干嘛

4. debugger
