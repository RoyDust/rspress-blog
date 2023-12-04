# 4、JS模块化详解



## 历史

### 幼年期：无模块化

1. 开始需要在页面加载不同的js了：动画库、表单库、格式化工具
2. 多种js文件被分在不同的文件中
3. 不同的文件又被同一个模板所引用

```html
<script src="jquery.js"></script>
<script src="tool.js"></script>
<script src="main.js"></script>
```



文件分离是最基础的模块化，第一步



> script标签的两个参数 - async & defer

```js
<script src="jquery.js" async></script>
```

总结：
普通 -解析到标签，立刻pending，并且下载且执行
defer -解析到标签开始异步下载，继续解析完成后开始执行
async - 解析到标签开始异步下载，下载完成之后立刻执行并且阻塞解析，执行完成后，再继续解析

#### 问题

* 污梁全局作用域  => 不利于大型项目的开发以及多人团队共建



### 成长期：模块化的雏形 - IIFE（语法侧的优化）

```js
  let count = 0;
  const increase = () => ++count;

  const rest = () => {
    count = 0;
  };
  increase();

  rest();
```

利用函数块级作用域：

```js
  (() => {
    let count = 0;
    const increase = () => ++count;

    const rest = () => {
      count = 0;
    };
    increase();

    rest();
  })();
```

定义函数 + 立即执行 => 独立的空间
初步实现了一个最最最最最最最简单的模块
尝试定义一个最简单的模块 - 模块 and 外部

```js
  const iifeModule = (() => {
    let count = 0;
    return {
      increase: () => ++count,
      rest: () => {
        count = 0;
      },
    };
  })();
```



**追问：有额外依赖的时候，如何优化处理IIFE**

> 依赖其他的IIFE

```js
  const iifeModule = ((dep1, dep2) => {
    let count = 0;
    return {
      increase: () => ++count,
      rest: () => {
        count = 0;
      },
    };
  })(dep1, dep2);
```

>IIFE + 传参调配

实际上，传统框架应用了一种revealing的写法
揭示模式

```js
  const iifeModule = ((dep1, dep2) => {
    let count = 0;
    const increase = () => ++count;
    const rest = () => {
      count = 0;
    };
    return {
      increase,
      rest,
    };
  })(dep1, dep2);
	iifeModule.increse(1)
	//返回的是能力 = 便用方传参 + 本身逻辑能力 + 依赖能里
	//$（""）.attr（'title'，'new"）；
```

 

### 成熟期

#### CJS - Common.js

 node.js指定的标准
特征：
* 通过module + export 去对外暴露接口
* 通过require去调用其他模块

模块组织方式：

```js
const doSomething = require('./doSomething.js');

const doSomething = (n) => { 
    // do something 
}

exports.doSomething =  doSomething

module.exports = {
  doSomething
}

-----------------------------------------------------
  
  const { doSomething } = require("xx.js")
  doSomething()
```

 

> 可能会被问到的问题：为什么部分开源项目要传入全局和jQuery作为参数

![image-20231109032313031](http://img.roydust.top/img/image-20231109032313031.png)

* 优点：
  CJS服务侧角度解决了依赖全局污染的问题

* 缺憾：
  针对服务端

  没有异步依赖

#### AMD

通过异步加载 + 允许定制回调函数
经典实现框架：require.js

```js
define(function(require, exports, module) { 
    var a = require('./a'); //在需要时申明
    a.doSomething(); 
    if (false) { 
        var b = require('./b'); 
        b.doSomething();
    } 
}); 
 /** sea.js **/ 
 // 定义模块 math.js 
 define(function(require, exports, module) { 
     var $ = require('jquery.js'); 
     var add = function(a,b){ return a+b; } 
     exports.add = add; 
 }); 
 // 加载模块 
 seajs.use(['math.js'], function(math){ 
 var sum = math.add(1+2); 
 });
```

** 面试题：如何对已有代码做兼容

1. 增加定义阶段
2. 返回作为回调内部的return



#### **UMD的出现**

 面试题：兼容判断AMD&CJS

```js
    (function (root, factory) {
        if (typeof define === 'function' && define.amd) {
            // AMD
            define(['jquery', 'underscore'], factory);
        } else if (typeof exports === 'object') {
            // Node, CommonJS之类的
            module.exports = factory(require('jquery'), require('underscore'));
        } else {
            // 浏览器全局变量(root 即 window)
            root.returnExports = factory(root.jQuery, root._);
        }
    }(this, function ($, _) {
        // 属性
        var PI = Math.PI;
        // 方法
        function a() { };                   // 私有方法，因为它没被返回
        function b() { return a() };        // 公共方法，因为被返回了
        function c(x, y) { return x + y };  // 公共方法，因为被返回了
        // 暴露公共方法
        return {
            ip: PI,
            b: b,
            c: c
        }
    }));
```

目标：一次性去区分CJS和AMD
1. CJS factory
2. module module.exports  
3. define



- 优点：解决了浏览器的异步动态依赖
- 缺点：会有引入成本，没有考虑按需加载



#### CMD规范

主要应用框架 sea.js

```js
// 依赖就近
define(function(require, exports, module) {
    var $ = require('jquery.js');
    var add = function(a,b){
        return a+b;
    }
    exports.add = add;
});
```

* 优点：按需加载，依赖就近
* 缺憾：1. 依赖于打包  2. 扩大了模块内的体积

区别：按需加载、依赖就近



### 新时代：ESM

新增定义：
引入关键字 - impori
导出关键字 - export

```js
 /** 定义模块 math.js **/
var basicNum = 0;
var add = function (a, b) {
    return a + b;
};
export { basicNum, add };

/** 引用模块 **/
import { basicNum, add } from './math';
function test(ele) {
    ele.textContent = add(99 + basicNum);
}
```



> 追问：ESM动态模块 考察：export promise

ES11原生解决方案

```js
import('./esModule.js').then（dynamicEsModule =>
dynamicEsModule. increase（）；
｝）
```

- 优点
  通过一种最统一的形态整合了所有js的模块化