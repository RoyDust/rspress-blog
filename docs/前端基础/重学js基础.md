# 重学JS基础

## JavaScript的DOM元素

### DOM元素之间的关系

- DOM相当于是JavaScript和HTML、CSS之间的桥梁
  - 通过浏览器提供给我们的DOM API,我们可以对元素以及其中的内容做任何事情;
- 类型之间有如下的继承关系:

![image-20230103144317515](http://img.roydust.top/img/202301031443612.png)



### 获取DOM元素



#### 节点之间的导航

如果我们获取到**一个节点(Node)**后，可以根据这个节点去获取其他的节点，我们称之为节点之间的导航。

**节点之间存在如下的关系:**

- 父节点: parentNode
- 前兄弟节点: previousSibling
- 后兄弟节点: nextSibling
- 子节点: childNodes
- 第一个子节点: firstChild
- 第二个子节点: lastChild

```html
  <body>
    我是文本
    <!-- 我是注释 -->
    <div class="box">哈哈哈哈哈</div>
    <ul>
      <li>1</li>
      <li>2</li>
      <li>3</li>
    </ul>

    <script>
      // var htmlEL = document.documentElement;
      // var bodyEl = document.body;

      // console.log(htmlEL, bodyEl);

      // 1.获取节点的导航
      var bodyEl = document.body;
      // 1.1.获取body所有的子节点
      console.log(bodyEl.childNodes);
      // 1.2.获取body第一个子节点
      var bodyElFirstChild = bodyEl.firstChild;
      // 1.3.获取body的注释
      var bodyElCommentChild = bodyElFirstChild.nextSibling;
      console.log(bodyElFirstChild, bodyElCommentChild);
    </script>
  </body>
```



#### 元素之间的导航

如果我们获取到一个**元素(Element)** 后，可以根据这个元素去获取其他的元素，我们称之为元素之间的导航。

**节点之间存在如下的关系:**

- 父元素: parentElement
- 前兄弟节点: previousElementSibling
- 后兄弟节点: nextElementSibling
- 子节点: children
- 第一个子节点: firstElementChild
- 第二个子节点: lastElementChild

```html
 <body>
    我是文本
    <!-- 我是注释 -->
    <div class="box">哈哈哈哈哈</div>
    <ul>
      <li>1</li>
      <li>2</li>
      <li>3</li>
    </ul> 		

		<script>
      var bodyEl = document.body;
      // 根据body元素去获取子元素（element）
      var childElement = bodyEl.children;
      console.log(childElement);

      // 获取box元素
      var boxEl1 = bodyEl.firstElementChild;
      var boxEl2 = bodyEl.children[0];
      console.log(bodyEl, boxEl2, boxEl1 === boxEl2);

      // 获取ul元素
      var ulEl = boxEl1.nextElementSibling;
      console.log(ulEl);

      // 获取li元素
      var ilEls = ulEl.children;
      console.log(ilEls);
    </script>
</body>
```



#### table元素之间的导航

< table > 元素支持(除了上面给出的,之外)以下这些属性:

- table.rows一 < tr > 元素的集合;
- table.caption/tHead/tFoot 一引用元素< caption>， < thead>， < tfoot> ;
- table.tBodies 一< tbody> 元素的集合; 

< thead>, < tfoot>, < tbody> 元素提供了rows属性:

- tbody.rows 一表格内部< tr> 元素的集合; 

< tr>:

- tr.cells一 在给定< tr> 中的< td>和< th>单元格的集合;
- tr.sectionRowlndex一给定的< tr> 在封闭的< thead>/ < tbody>/< tfoot>中的位置(索引) ;
- tr.rowlndex一在整个表格中< tr> 的编号(包括表格的所有行) ;

< td>和< th>:

- td.clllndex一在封闭的< tr>中单元格的编号

```html
  <body>
    <!-- 高级元素 -->
    <table>
      <thead>
        <tr>
          <th>项目</th>
          <th>年龄</th>
          <th>身高</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>why</th>
          <th>18</th>
          <th>1.88</th>
        </tr>
        <tr>
          <th>wh2</th>
          <th>w8</th>
          <th>1.98</th>
        </tr>
        <tr>
          <th>wha</th>
          <th>15</th>
          <th>1.58</th>
        </tr>
      </tbody>
    </table>

    <script>
      var tableEl = document.body.firstElementChild;

      // 通过table元素获取内部的后代元素
      console.log(tableEl.tHead, tableEl.tBodies, tableEl.tFoot);
      console.log(tableEl.rows);

      // 拿到一行元素
      var rowEl = tableEl.rows[2];
      console.log(rowEl.cells[1]);
      console.log(rowEl.sectionRowIndex);
      console.log(rowEl.rowIndex);
    </script>
  </body>
```



#### form元素之间的导航

```html
    <form action="">
      <input name="account" type="text" />
      <input name="password" type="text" />
      <input name="hobbies" type="checkbox" checked />
      <select name="fruits" id="">
        <option value="apple">苹果</option>
        <option value="orange">橘子</option>
      </select>
    </form>
```

- form元素可以通过document来获取：document.forms、

  ```js
        // 1.获取form
  
        var formEl = document.forms[0];
  ```

  

- form元素可以通过elements来获取：document.elements

  ```js
        var formEl = document.body.firstElementChild;
  ```

  

- 我们也可以设置表单名来获取他们

  ```js
        // 2.获取form元素中的子元素
        // var inputEl = formEl.elements[0];
        var inputEl = formEl.elements.password;
        console.log(inputEl);
  ```



#### 获取任意元素的方法

- 当元素彼此靠近或者相邻时，DOM 导航属性(navigation property)非常有用。
  - 但是，在实际开发中，我们希望可以任意的获取到某一个元素应该如何操作呢?
- DOM为我们提供了获取元素的方法:

![image-20230104153017258](http://img.roydust.top/img/202301041530338.png)

```html
    <div class="box">
      <h2>我是标题</h2>
      <div class="container">
        <p>我是段落，<span class="keyword">lyc</span> 哈哈哈哈</p>
        <p>我是段落，<span class="keyword">ddd</span> 哈哈哈哈</p>
      </div>

      <div class="article">
        <h3 id="title">我是小标题</h3>
        <p>我是文章的内容，嘿嘿嘿嘿嘿</p>
      </div>
    </div>

    <script>
      // 一、通过导航获取
      // // 1.拿到body
      // var bodyEl = document.body;

      // // 2.拿到box
      // var boxEl = bodyEl.firstElementChild;

      // // 3.拿到container
      // var container = boxEl.children[1];

      // // 4.拿到p
      // var pEl = container.children[0];

      // // 5.拿到keyword
      // var spanEl = pEl.children[0];
      // console.log(spanEl);
      // spanEl.style.color = "red";

      // 二、直接获取keyword - getElementsBy
      // 1.通过className获取元素
      var keywords = document.getElementsByClassName("keyword");
      console.log(keywords[0]);
      keywords[0].style.color = "red";

      // 2.通过id获取元素
      var titleEl = document.getElementById("title");
      console.log(titleEl);
      titleEl.style.color = "blue";

      // 三、直接获取keyword - querySelector 选择器查询
      var keywords2 = document.querySelector(".keyword"); // 找第一个
      var keywords3 = document.querySelectorAll(".keyword"); // 找全部的
      var titleEl2 = document.querySelector("#title");

      for (const item of keywords3) {
        item.style.color = "orange";
      }
      titleEl2.style.color = "skyblue";
    </script>
```

- 开发中如何选择呢?
  - 前最常用的是guerySelector和guerySelectAll;
  - getElementByld偶尔也会使或者在适配一些低版本浏览器时;



### DOM节点的属性

#### nodeType 节点类型

- 目前，我们已经可以获取到节点了，接下来我们来看一- 下节点中有哪些常见的属性:
  - 当然，不同的节点类型有可能有不同的属性; 
  - 这里我们主要讨论节点共有的属性;
- nodeType属性:
  - nodeType属性提供了一种获取节点类型的方法;
  - 它有一-个数值型值(numeric value) ;
- 常见的节点类型有如下:

![image-20230104155048323](http://img.roydust.top/img/202301041550383.png)



#### nodeName和nodeTag

 nodeName:获取node节点的名字

 tagName:获取元素的标签名词

tagName和nodeName之间有什么不同呢?

- tagName属性仅适用于Element节点; 
- nodeName是为任意Node定义的:
  - 对于元素，它的意义与tagName相同，所以使用哪一个都是可以的;
  - 对于其他节点类型(text， comment 等)，它拥有一个对应节点类型的字符串;

```html
 <!-- 我是注释 -->
    我是文本
    <div class="box">我是box</div>

    <script>
      // 1.获取3个节点
      var bodyChildNodes = document.body.childNodes;
      var commentNode = bodyChildNodes[1];
      var textNode = bodyChildNodes[2];
      var divNode = bodyChildNodes[3];
      console.log(commentNode, textNode, divNode);

      // 2.节点属性
      // 2.1.节点类型
      console.log(commentNode.nodeType, textNode.nodeType, divNode.nodeType);

      // 2.2.nodeName 节点名称
      console.log(commentNode.nodeName, textNode.nodeName, divNode.nodeName);
      // tagName  针对元素的 
      console.log(commentNode.tagName, divNode.tagName);
    </script>
```



#### innerHTML和textContent

- innerHTML属性
  - 将元素中的HTML获取为字符串形式;
  - 设置元素中的内容;
- outerHTML属性
  - 包含了元素的完整HTML
  - innerHTML加上元素本身-样;
- textContent属性
  - 仅仅获取元素中的文本内容;
- innerHTML和textContent的区别:
  - 使用innerHTML,我们将其“作为HTML"插入，带有所有HTML标签。
  - 使用textContent,我们将其"作为文本”插入，所有符号(symbol) 均按字面意义处理。

```js
      // 2.3.data(nodeValue)/innerHTML/textContent
      // data：针对非元素节点获取数据的
      // innerHTML：对应的html也会获取
      // textContent：只会获取文本内容
      console.log(commentNode.data, textNode.data, divNode.data);
      console.log(divNode.innerHTML);
      console.log(divNode.textContent);

      // 设置文本，作用是一样的
      // 设置文本中包含元素内容，那么innerHTML浏览器会解析，textContent会当成文本的一部分
      divNode.innerHTML = "<h2>呵呵呵呵呵</h2>";
      divNode.textContent = "<h2>呵呵呵呵呵</h2>";

      console.log(divNode.outerHTML);
```

![image-20230104161343592](http://img.roydust.top/img/202301041613639.png)



#### hidden属性

hidden属性:也是一个全局属性，可以用于设置元素隐藏。

```html
    <button class="btn">切换</button>
    <div class="aaa" id="box" title="aaa">哈哈哈哈哈</div>

    <script>
      // 1.获取元素
      var boxEl = document.querySelector("#box");
      var btnEl = document.querySelector(".btn");

      // 2.监听btn的点击
      btnEl.onclick = function () {
        boxEl.hidden = !boxEl.hidden;
      };
    </script>
```



#### DOM元素还有其他属性:

- value
  < input>, < select> 和< textarea> (HTMLInputElement, HTML SelectElem......的value。
-  href
  <a href=.."> (HTMLAnchorElement) 的href。
-  id
  所有元素(HTMLElement) 的"id” 特性(attribute) 的值。



### 元素Element常见的属性

#### 元素的特征attributes

- 前面我们已经学习了如何获取节点，以及节点通常所包含的属性，接下来我们来仔细研究元素Element.
- 我们知道，一个元素除了有开始标签、结束标签、内容之外，还有很多的属性(attribute)

![image-20230104164335127](http://img.roydust.top/img/202301041643171.png)

- 浏览器在解析HTML元素时，会将对应的attribute也创建出来放到对应的元素对象上。
  - 比如id、class就是全局的attribute,会有对应的id、class属性;
  - 比如href属性是针对a元素的，type、 value属性 是针对input元素的;

- 属性attribute的分类:
  - 标准的attribute:某些attribute属性 是标准的，比如id、class、 href、 type、 value等;
  - 非标准的attribute:某些attribute属性是自定义的，比如abc、age、 height等;



##### attributes操作

对于所有的attribute访问都支持如下的方法:
-  elem.hasAttribute(name)一检查特性是否存在。
-  elem.getAttribute(name)一获取这个特性值。
-  elem.setAttribute(name, value)一设置这个特性值。
-  elem.removeAttribute(name)一移除这个特性。
-  attributes: attr对象的集合，具有name、value属性;

```html
    <!-- 属性：attribute(特性) -->
    <!-- 
      attribute的分类：
      1.如果是HTML标准指定的Attribute，称之为标准Attribute
      2.而自定义的Attribute，被称之为非标准Attribute
    -->
    <div class="box" id="box" title="box" age="18" height="1.88">我是box</div>

    <a href="https://www.baidu.com">百度一下</a>

    <script>
      var boxEl = document.querySelector(".box");
      // 1.所有的attribute都支持的操作
      console.log(
        boxEl.hasAttribute("age"),
        boxEl.hasAttribute("abc"),
        boxEl.hasAttribute("id")
      );
      console.log(
        boxEl.getAttribute("age"),
        boxEl.getAttribute("abc"),
        boxEl.getAttribute("id")
      );
      boxEl.setAttribute("id", "cba");
      boxEl.removeAttribute("age ");

      var boxAttributes = boxEl.attributes;
      for (const attr of boxAttributes) {
        console.log(attr.name, attr.value);
      }
    </script>
```

![image-20230104165418085](http://img.roydust.top/img/202301041654115.png)





- attribute具备以下特征:
  - 它们的名字是大小写不敏感的(id 与ID相同) 
  - 它们的值总是字符串类型的





#### 元素的属性properies

对于标准的attribute,会在DOM对象上创建与其对应的property属性:

![image-20230104170450093](http://img.roydust.top/img/202301041704136.png)

在大多数情况下，它们是相互作用的

- 改变property,通过attribute获取的值，会随着改变;
- 通过attribute操作修改，property的值会随着改变; 
  - 但是input的value修改只能通过attribute的方法; 
- 除非特别情况，大多数情况下，设置、获取attribute, 推荐使用property的方式:
  - 这是因为它默认情况下是有类型的;

```html
    <!-- 元素中的属性称之为attribute -->
    <!-- 标准attribute在对应的对象模型中都有对应的property -->
    <div class="box" id="abc" title="标题" age="18" height="1.88">我是box</div>
    <input type="checkbox" checked />
    账号：<input type="text" class="account" />

    <script>
      // 对象中的属性称之为property
      var obj = {
        // property
        name: "why",
      };

      // 获取box元素
      var boxEl = document.querySelector(".box");
      // 在这里的boxEl.id是一个property
      console.log(boxEl.id, boxEl.title, boxEl.age, boxEl.height);

      // input 元素
      var inputEl = document.querySelector("input");
      // 如果选择用attribute则获取不到checked属性
      // if (inputEl.getAttribute("checked")) {
      //   console.log("checkbox处于选中状态");
      // }

      if (inputEl.checked) {
        console.log("checkbox处于选中状态");
      }

      // 2.attribute和property是相互影响的
      boxEl.id = "aaa";
      console.log(boxEl.id);

      // 3.比较特殊的情况，input设置值（了解）
      var accountInputEl = document.querySelector(".account");
      // 优先级更高
      accountInputEl.value = "lyc";
      accountInputEl.setAttribute("value", "kobe");
    </script>
```



#### Dataset的使用

dataset可以获得**非标准的attribute的值**

```html
    <div id="abc" class="box" data-age="18" data-height="1.88">hhh</div>

    <script>
      var boxEl = document.querySelector(".box");
      console.log(boxEl.dataset.age);
      console.log(boxEl.dataset.height);
    </script>

		/*
		18
		1.88
		*/
```



#### 通过JavaScript动态修改样式

- 有时候我们会通过JavaScript来动态修改样式，这个时候我们有两个选择: 
  - 选择一:在CSS中编写好对应的样式，动态的添加class;
  - 选择二:动态的修改style属性;
- 开发中如何选择呢?
  - 在大多数情况下，如果可以动态修改class完成某个功能，更推荐使用动态class;
  - 如果对于某些情况，无法通过动态修改class (比如精准修改某个css属性的值) , 那么就可以修改style属性;



##### 元素中的className和classList

- 元素的class attrilbute,对应的property并非叫class,而是className:
  - 这是因为JavaScript早期是不允许使用class这种关键字来作为对象的属性，所以DOM规范使用了className;
  - 虽然现在JavaScript已经没有这样的限制，但是并不推荐，并且依然在使用className这个名称;
- 我们可以对className进行赋值，它会替换整个类中的字符串。

```js
      var boxEl = document.querySelector(".box");

      // 1.方法一： className
      boxEl.className = "abc";
```

- 如果我们需要添加或者移除单个的class,那么可以使用classList属性。
- elem.classList是一个特殊的对象:
  - elem.classList.add (class) :添加一个类
  - elem.classList.remove(class):添加/移除类。
  - elem.classList.toggle(class) 如果类不存在就添加类, 存在就移除它。
  - elem.classList.contains(class):检查给定类，返回true/false。

```js
			// 2.方法二：classList操作class
      boxEl.classList.add("abc");

      // 需求：box在active之间切换
      var btnEl = document.querySelector(".btn");
      btnEl.onclick = function () {
        // if (boxEl.classList.contains("active")) {
        //   boxEl.classList.remove("active");
        // } else {
        //   boxEl.classList.add("active");
        // }
        boxEl.classList.toggle("active");
      };
```



##### 元素中的style属性

- 如果需要单独修改某一个CSS属性， 那么可以通过style来操作:

  - 对于多词(multi-word) 属性，使用驼峰式 camelCase

  ![image-20230104211806509](http://img.roydust.top/img/202301042118550.png)

- 如果我们将值设置为空字符串，那么会使用CSS的默认样式:

  ![image-20230104211756013](http://img.roydust.top/img/202301042117059.png)

- 多个样式的写法，我们需要使用cssText属性:

  - 不推荐这种用法，因为它会替换整个字符串;

- 如果我们需要读取样式:

  - 对于内联样式，是可以通过style.*的方式读取到的;
  - 对于style、Css文件中的样式，是读取不到的;

- 这个时候，我们可以通过getComputedStyle的全局函数来实现:

  ![image-20230104211932511](http://img.roydust.top/img/202301042119568.png)







### DOM节点的创建、插入、克隆、删除

#### DOM节点的创建

- 前面我们使用过document.write方法写入一个元素:
  - 这种方式写起来非常便捷，但是对于复杂的内容、元素关系拼接并不方便;
  - 它是在早期没有DOM的时候使用的方案，目前依然被保留了下来;
- 那么目前我们想要插入一个元素，通常会按照如下步骤:
  - 步骤一: 创建一个元素;
  - 步骤二:插入元素到DOM的某一个位置;

- 创建元素: document.createElement(tag)

```js
      var boxEl = document.querySelector(".box");

      // 1.通过innerHTML（不推荐）
       boxEl.innerHTML = `
      	<h2>我是标题</h2>
       `;

      // 2.真实创建一个DOM对象
      var h2El = document.createElement("h2");
      h2El.className = "title";
      h2El.classList.add("active");
      h2El.textContent = "我是标题"; 

      boxEl.append(h2El);
```

#### DOM节点插入元素方法

![image-20230105224408075](http://img.roydust.top/img/202301052244211.png)

#### DOM节点移除元素方法

移除元素我们可以调用元素本身的remove方法:

```js
      // 2.监听removeBtn的点击
      removeBtnEl.onclick = function () {
        boxEl.remove();
      };
```

#### DOM节点复制元素方法

如果我们想要复制一个现有的元素，可以通过cloneNode方法:

- 可以传入一个Boolean类型的值，来决定是否是深度克隆;
- 深度克隆会克隆对应元素的子元素，否则不会;

```js
      // 1.获取元素
      var boxEl = document.querySelector(".box");
      var removeBtnEl = document.querySelector(".remove-btn");
      var cloneBtnEl = document.querySelector(".clone-btn");

      // 3.复制box
      var counter = 0;
      cloneBtnEl.onclick = function () {
        var newNode = boxEl.cloneNode(true);
        newNode.childNodes[0].textContent = "我也是标题" + counter++;
        // boxEl.after(newNode);
        document.body.append(newNode);
      };
```



### DOM节点的样式、类

#### DOM元素大小和滚动

- clientWidth: contentWith+ padding (不包含滚动条)
- clientHeight: contentHeight+ padding
- clientTop: border-top的宽度
- clientLeft: border-left的宽度
- offsetWidth:元素完整的宽度
- offsetHeight:元素完整的高度
- offsetLeft:距离父元素的x
- offsetHeight:距离父元素的y
- scrollHeight:整个可滚动的区域高度
- scrollTop:滚动部分的高度

![image-20230105233702045](http://img.roydust.top/img/202301052337129.png)

#### window的大小和滚动

- innerWidth、innerHeight: 获取window窗口的宽度和高度(包含滚动条)
- outerWidth、outerHeight: 获取window窗口的整个宽度和高度(包括调试工具、工具栏)
- documentElement.clientHeight、documentElement.clientWidth: 获取html的宽度和高度(不包含滚动条)

window的滚动位置:

- scrollX: X轴滚动的位置(别名pageXOffset)
- scrollY: Y轴滚动的位置(别名pageYOffset)

也有提供对应的滚动方法:

- 方法scrollBy(x,y) :将页面滚动至相对于当前位置的(X, Y)位置;
- 方法scrollTo(pageX,pageY)将页面滚动至绝对坐标;

```html
    <style>
      .box {
        width: 1000px;
        height: 1500px;
      }
      .btn {
        position: fixed;
        right: 20px;
        bottom: 20px;
      }
    </style>
  </head>
  <body>
    <button class="btn">滚动跳转</button>
    <div class="box"></div>

    <script>
      // window大小
      console.log(window.innerWidth);
      console.log(window.innerHeight);

      console.log(window.scrollX);
      console.log(window.scrollY);

      var btnEl = document.querySelector(".btn");
      btnEl.hidden = true;
      window.onscroll = function () {
        console.log(window.scrollY);
        var scrollY = window.scrollY; 
        if (scrollY > 200) {
          // btnEl.style.display = "";
          btnEl.hidden = false;
        } else {
          // btnEl.style.display = "none";
          btnEl.hidden = true;
        }
      };

      // 点击按钮后滚动到某个位置
      btnEl.onclick = function () {
        // window.scrollBy(0, 100);
        window.scrollTo(0, 0);
      };
    </script>
```



## JavaScript的事件处理



### 认识事件Event

**Web页面经常需要和用户之间进行交互，而交互的过程中我们可能想要捕获这个交互的过程**

- 比如用户点击某个按钮、用户在输入框里面输入了某个文字，用户鼠标经过了某个位置
- 浏览器需要搭建一条JavaScript代码和事件的桥梁
- 当事件发生时，让JavaScript可以响应（执行某个函数），所以我们需要针对事件编写处理程序

**如何进行事件监听**

- 在script直接监听
- DOM属性，通过元素的on来监听事件
- 通过EventTarget中的addEventListener来监听

```html
    <button onclick="console.log('btn1点击')">按钮1</button>
    <button class="btn2">按钮2</button>
    <button class="btn3">按钮3</button>

    <script>
      var btn2El = document.querySelector(".btn2");
      var btn3El = document.querySelector(".btn3");

      // 2.onclick属性
      btn2El.onclick = function () {
        console.log("btn2点击");
      };

      // 3.addEventListener(推荐)

      btn3El.addEventListener("click", function () {
        console.log("第一个btn3的时间监听");
      });
      btn3El.addEventListener("click", function () {
        console.log("第二个btn3的时间监听");
      });
      btn3El.addEventListener("click", function () {
        console.log("第三个btn3的时间监听");
      });
    </script>
```



### 事件流

事实.上对于事件有一个概念叫做事件流，为什么会产生事件流呢?

- 我们可以想到一个问题:当我们在浏览器上对着一个元素点击时，你点击的不仅仅是这个元素本身;
- 这是因为我们的HTML元素是存在父子元素叠加层级的;
- 比如一-个span元素是放在div元素上的，div元素是放在body元素 上的，body元素是放在html元素上的;

```html
    <div class="box">
      <span></span>
    </div>

    <script>
      var spanEl = document.querySelector("span");
      var divEl = document.querySelector("div");
      var body = document.body;

      spanEl.onclick = function () {
        console.log("span元素发生了点击");
      };
      divEl.onclick = function () {
        console.log("divEl元素发生了点击");
      };
      body.onclick = function () {
        console.log("body元素发生了点击");
      };
    </script>
```

![image-20230106145721753](http://img.roydust.top/img/202301061457789.png)

### 事件冒泡和事件捕获

- 默认情况下事件是从最内层的span向外以次传递的顺序，这个顺序我们称之为事件冒泡
- 还有一种监听事件流的方式就是从外层到内层，这种称之为事件捕获

- 为什么会产生两种不同的处理流呢？
  - 这是因为早期浏览器开发时，不管是IE还是Netscape公司都发现了这个问题;
  - 但是他们采用了完全相反的事件流来对事件进行了传递; .
  - IE采用了事件冒泡的方式，Netscape采用了事件捕获的方式;

- 那我们如何去监听事件捕获的过程呢？

```html
    <div class="box">
      <span></span>
    </div>

    <script>
      var spanEl = document.querySelector("span");
      var divEl = document.querySelector("div");
      var body = document.body;

      // 默认情况下是事件冒泡
      // spanEl.onclick = function () {
      //   console.log("span元素发生了点击");
      // };
      // divEl.onclick = function () {
      //   console.log("divEl元素发生了点击");
      // };
      // body.onclick = function () {
      //   console.log("body元素发生了点击");
      // };

      // 设置希望监听事件捕获的过程
      spanEl.addEventListener(
        "click",
        function () {
          console.log("span元素发生了点击");
        },
        true	//传递第三个参数为true 即是事件捕获
      );
      divEl.addEventListener(
        "click",
        function () {
          console.log("divEl元素发生了点击");
        },
        true
      );
      body.addEventListener(
        "click",
        function () {
          console.log("body元素发生了点击");
        },
        true
      );
    </script>
```

- 如果我们都监听，那么会按照如下顺序来执行:
  - 捕获阶段(Capturing phase) :
    - 事件(从Window)向下走近元素。
  - 目标阶段(Target phase) :
    - 事件到达目标元素。
  - 冒泡阶段(Bubbling phase) :
    - 事件从元素上开始冒泡。
- 事实上，我们可以通过event对象来获取当前的阶段:
  - eventPhase 
- 开发中通常会使用事件冒泡，所以事件捕获了解即可。

![](http://img.roydust.top/img/202301061527645.png)



### 事件处理

- 当一个事件发生时，就会有和这个事件相关的很多信息:
  - 比如事件的类型是什么，你点击的是哪一-个元素，点击的位置是哪里等等相关的信息;
  - 那么这些信息会被封装到一-个Event对象中，这个对象由浏览器创建，称之为event对象;
  - 该对象给我们提供了想要的一-些属性，以及可以通过该对象进行某些操作;
- 如何获取这个event对象呢?
  - event对象会在传入的事件处理(event handler)函数回调时，被系统传入;
  - 我们可以在回调函数中拿到这个event对象;

```js
      divEl.onclick = function (event) {
        console.log("div发生了点击", event);
      }
```

![image-20230106155746711](http://img.roydust.top/img/202301061557763.png)



#### 常见的属性和方法（重点）

常见的属性:

-  type:事件的类型;
-  target:当前事件发生的元素;
-  currentTarget:当前处理事件的元素; 
-  eventPhase:事件所处的阶段;
-  offsetX、offsetY: 事件发生在元素内的位置;
-  clientX、clientY: 事件发生在客户端内的位置;
-  pageX、pageY: 事件发生在客户端相对于document的位置;
-  screenX、screenY: 事件发生相对于屏幕的位置; 

````html
    <div class="box">
      <span class="btn">按钮</span>
    </div>

    <script>
      var divEl = document.querySelector("div");
      var btnEl = document.querySelector(".btn");

      divEl.onclick = function (event) {
        // 1.偶尔会使用
        console.log("div发生了点击", event);
        console.log("事件类型：", event.type);
        // 2.比较少使用
        console.log("事件阶段：", event.eventPhase);
        console.log("事件元素中的位置：", event.offsetX, event.offsetY);
        console.log("事件客服端中的位置：", event.clientX, event.clientY);

        // 3.target/currentTarget
        console.log(event.target);
        console.log(event.currentTarget);
        console.log(event.target === event.currentTarget);
      };
    </script>
````



常见的方法:

- **preventDefault:取消事件的默认行为;**
- **stopPropagation:阻止事件的进一步传递( 冒泡或者捕获都可以阻止) ;** 

```js
      // 1.阻止默认行为
      var aEl = document.querySelector("a");
      aEl.onclick = function (event) {
        event.preventDefault();
        alert("a元素发生了点击");
      };

      // 2.阻止事件进一步传递
      var btnEl = document.querySelector("button");
      var spanEl = document.querySelector("span");
      var divEl = document.querySelector("div");

      divEl.addEventListener(
        "click",
        function (event) {
          console.log("div事件捕获");
        },
        true
      );
      spanEl.addEventListener(
        "click",
        function () {
          console.log("spanEl事件捕获");
        },
        true
      );
      btnEl.addEventListener(
        "click",
        function (event) {
          console.log("btnEl事件捕获");
          event.stopPropagation();
        },
        true
      );
      divEl.addEventListener("click", function () {
        console.log("div事件冒泡");
      });
      spanEl.addEventListener("click", function () {
        console.log("spanEl事件冒泡");
      });
      btnEl.addEventListener("click", function () {
        console.log("btnEl事件冒泡");
      });
```

#### 事件处理中的this

- 在函数中，我们也可以通过this来获取当前元素：
  ```js
        var btnEl = document.querySelector("button");
        var divEl = document.querySelector("div");
  
        divEl.onclick = function (event) {
          console.log(this);
          console.log(event.currentTarget);
          console.log(divEl);
          console.log(this === divEl);
        };
  ```

- 这是因为在浏览器内部，调用event handler是绑定到当前的currentTarget上的



### EventTarget类

我们会发现，所有的节点、元素都继承自EventTarget

- 事实上window也继承于EventTarget

那么这个EventTarget是什么呢?

- EventTarget是一个DOM接口，主要用于添加、删除、派发Event事件; 

EventTarget常见的方法:

- addEventListener:注册某个事件类型以及事件处理函数;
- removeEventListener:移除某个事件类型以及事件处理函数; .
- dispatchEvent:派发某个事件类型到EventTarget.上;

```js
      // eventTarget就可以实现类似于事件总线一样的效果
      window.addEventListener("lyc", function () {
        console.log("监听到lyc事件");
      });

      setTimeout(() => {
        window.dispatchEvent(new Event("lyc"));
      }, 2000); 
```



### 事件委托

- 事件冒泡在某种情况下可以帮助我们实现强大的事件处理模式-事件委托模式(也是一种设计模式)
- 那么这个模式是怎么样的呢?
  - 因为当子元素被点击时，父元素可以通过冒泡可以监听到子元素的点击;
  - 并且可以通过event.target获取到当前监听的元素;



案例一：点击li变红，其他的li变黑

```html
    <style>
      .active {
        color: red;
      }
    </style>
  </head>
  <body>
    <ul>
      <li>1</li>
      ...
      <li>10</li>
    </ul>
    <script>
      // 1.每个li都有自己的监听，并且有自己的处理函数（自己的函数）
      // var ilEls = document.querySelectorAll("li");
      // for (var ilEl of ilEls) {
      //   ilEl.onclick = function () {
      //     this.classList.add("active");
      //   };
      // }

      // 2.统一在ul中监听
      var ulEl = document.querySelector("ul");
      ulEl.onclick = function (event) {
        console.log(event.target);
        event.target.classList.add("active");
      };

      // 3.点击li变成active，其他的取消active
      var ulEl = document.querySelector("ul");
      var activeEl = null;
      ulEl.onclick = function (event) {
        console.log(event.target);
        // 1.将之前的active移除
        // for (let i = 0; i < ulEl.children.length; i++) {
        //   var liEl = ulEl.children[i];
        //   if (liEl.classList.contains("active")) {
        //     liEl.classList.remove("active");
        //   }
        // }

        // 1.找到active的i，移除掉active
        // var activeEl = ulEl.querySelector(".active");
        // activeEl && activeEl.classList.remove("active");

        // 1.变量记录方式
        if (activeEl && event.target.tagName === "LI") {
          activeEl.classList.remove("active");
        }

        // 2.新增新的active
        if (event.target !== ulEl) {
          event.target.classList.add("active");
        }

        // 3.记录最新的active的li
        activeEl = event.target;
      };
    </script>
```



案例二：利用事件委托区分多个按钮的

```html
    <div class="box">
      <button data-action="remove">移除</button
      ><button data-action="new">新建</button
      ><button data-action="search">搜索</button>
      <button>111</button>
    </div>

    <script>
      var boxEl = document.querySelector(".box");
      boxEl.onclick = function (event) {
        var btnEl = event.target;
        var action = btnEl.dataset.action;

        switch (action) {
          case "remove":
            console.log("点击了移除按钮");
            break;
          case "new":
            console.log("点击了新建按钮");
            break;
          case "search":
            console.log("点击了搜索按钮");
            break;
          default:
            console.log("点击了其他");
        }
      };
    </script>
```



### 常见的事件列表

![image-20230106144838735](http://img.roydust.top/img/202301061448852.png)



#### 常见鼠标事件

![image-20230108144453685](http://img.roydust.top/img/202301081444820.png)

##### mouseover和mouseenter的区别

mouseenter和mouseleave

- 不支持冒泡
- 进入子元素依然属于在该元素内，没有任何反应

![image-20230108145316002](http://img.roydust.top/img/202301081453057.png)

![image-20230108161637272](http://img.roydust.top/img/202301081616321.png)

mouseover和mouseout

- 支持冒泡
- 进入元素的子元素时
  - 先调用父元素的mouseout
  - 再调用子元素的mouseover
  - 因为支持冒泡，所以会将mouseover传递到父元素中;

 ![image-20230108145458850](http://img.roydust.top/img/202301081454901.png)

![image-20230108161646925](http://img.roydust.top/img/202301081616968.png)

#### 常见键盘事件

![image-20230108161843034](http://img.roydust.top/img/202301081618086.png)

事件的执行顺序是onkeydown、 onkeypress、 onkeyup

- down事件先发生（按下）
- press发生在文本被输入
- up发生在文本输入完成（抬起）

我们可以通过key和code来区分按下的键:

- code:“按键代码” ("KeyA", "ArrowLeft" 等)，特定于键盘上按键的物理位置。
- key:字符("A", "a"等)，对于非字符(non-character) 的按键,通常具有与code相同的值。)



#### 常见表单事件

![image-20230108163614059](http://img.roydust.top/img/202301081636128.png)



### 文档加载事件

- DOMContentLoaded:浏览器已完全加载HTML,并构建了DOM树，但像< img >和样式表之类的外部资源可尚未加载完成。
- load:浏览器不仅加载完成了HTML,还加载完成了所有外部资源:图片,样式等。

```js
      // 注册事件监听
      window.addEventListener("DOMContentLoaded", function () {
        console.log("HTML内容加载完毕");

        // 获取img对应的图片高度和宽度
        var imgEl = document.querySelector("img");
        console.log("图片宽度和高度：", imgEl.offsetWidth, imgEl.offsetHeight); // 0 0
      });
      // 注册事件监听
      window.onload = function () {
        console.log("文档中所有资源都加载完毕");
        var boxEl = document.querySelector(".box");
        boxEl.style.backgroundColor = "orange";

        var imgEl = document.querySelector("img");
        console.log("图片宽度和高度：", imgEl.offsetWidth, imgEl.offsetHeight); // 400 200
      };
      // 当浏览器大小发生改变时就会触发事件
      window.onreset = function () {
        console.log("创建大小发生改变时");
      };
```

## JavaScript的BOM操作

BOM:浏览器对象模型(Browser Object Model)

- 简称BOM,由浏览器提供的用于处理文档(document) 之外的所有内容的其他对象;
- 比如navigator、location、 history等对象;

JavaScript有一个非常重要的运行环境就是浏览器

- 而且浏览器本身又作为一一个应用程序需要对其本身进行操作;
- 所以通常浏览器会有对应的对象模型(BOM, Browser Object Model) ;
- 我们可以将BOM看成是连接JavaScript脚本与浏览器窗口的桥梁;

BOM主要包括以下的对象模型:

- window: 包括全局属性、方法，控制浏览器窗口相关的属性、方法;
- location: 浏览器连接到的对象的位置(URL) ;
- history: 操作浏览器的历史;
- navigator:用户代理(浏览器)的状态和标识(很少用到) ;
- screen: 屏幕窗口信息(很少用到) ;

![image-20230109004447739](http://img.roydust.top/img/202301090044842.png)



### Window对象

window对象在浏览器中可以从两个视角来看待:

- 视角一:**全局对象**。
  - 我们知道ECMAScript其实是有一个全局对象的，这个全局对象在Node中是**global**;
  - 在浏览器中就是**window对象**;
- 视角二:**浏览器窗口对象**。
  - 作为浏览器窗口时，提供了对浏览器操作的相关的API;

当然，这两个视角存在大量重叠的地方，所以不需要刻意去区分它们:

- 事实上对于**浏览器和Node中全局对象名称不一样的情况**，目前已经指定了对应的标准，称之为**globalThis**,并且大多数现代浏览器都支持它;
- 放在**window对象**上的所有属性都可以被访问;
- 使用**var定义的变量会被添加到window对象**中;
- window默认给我们提供了**全局的函数和类**: setTimeout、 Math、 Date、 Object等 ;



#### **Window的作用**

事实上window对象.上肩负的重担是非常大的:

- 第一:包含大量的属性，localStorage、 console、 location、 history、 screenX、 scrollX等等(大概60+个属性) ;
- 第二:包含大量的方法，alert、 close、 scrollTo、 open等等(大概40+个方法) ;
- 第三:包含大量的事件，focus、 blur、 load、 hashchange等等 (大概30+个事件) ;
- 第四:包含从EventTarget继承过来的方法，addEventListener、 removeEventListener. dispatchEvent方法;

更多具体方法可以查看MDN：

- MDN文档地址：https://developer.mozilla.org/zh-CN/docs/Web



#### **window常见的属性**

![image-20230109010836687](http://img.roydust.top/img/202301090108778.png)



#### **window常见的方法**

![image-20230109010915297](http://img.roydust.top/img/202301090109364.png)



#### **window常见的事件**

![image-20230109010937399](http://img.roydust.top/img/202301090109459.png)

```js
      // 1.window的查看角度
      // ECMAScript规范：全局对象 -> globalThis
      // 对于浏览器 -> window
      // 对于node —> global
      console.log(window);
      console.log(globalThis);
      // 浏览器窗口对象
      console.log(window.outerHeight);

      // 2.补充方法
      // 打开一个新窗口
      // window.open("www.baidu.com", "_self");
      // 关闭有open打开的窗口
      // window.close();

      // 3.常见事件
      window.onfocus = function () {
        console.log("窗口获取到焦点");
      };
      window.onblur = function () {
        console.log("窗口失去焦点");
      };
      window.HashChangeEvent = function () {
        console.log("hash值发生改变");
      };
```



### Location对象

**location对象用于表示window.上当前链接到的URL信息。**

常见的属性有哪些呢?

-  href:当前window对应的超链接URL,整个URL;
-  protocol:当前的协议;
-  host:主机地址; 
-  hostname:主机地址(不带端口);
-  port:端口;
-  pathname:路径;
-  search:查询字符串;
-  hash:哈希值; 

我们会发现location其实URL的一个抽象实现：
![image-20230109125645842](http://img.roydust.top/img/202301091256945.png)

location有如下常用的方法:

- assign:赋值一-个新的URL,并且跳转到该URL中;
- replace:打开一个新的URL,并且跳转到该URL中(不同的是不会在浏览记录中留下之前的记录) ;
- reload: 重新加载页面，可以传入- -个Boolean类型;

```js
      // 1.完整的URL
      console.log(location.href);

      // 2.获取URL信息
      console.log(location.hostname);
      console.log(location.host);
      console.log(location.protocol);
      console.log(location.port);
      console.log(location.pathname);
      console.log(location.search);
      console.log(location.hash);

      // 3.location方法
      location.assign("#");
      location.replace("#");
      location.reload(false)
```



#### URLSearchParams

URLSearchParams定义了一些实用的方法来处理URL的查询字符串。

- 可以将一个字符串转化成URLSearchParams类型;
- 也可以将一个URLSearchParams类型转成字符串;

URLSearchParams常见的方法有如下:

-  get:获取搜索参数的值; 
-  set:设置一个搜索参数和值;
-  append:追加一个搜索参数和值;
-  has:判断是否有某个搜索参数;

中文会使用encodeURIComponent和decodeURIComponent进行编码和解码

```js
      // 1.完整的URL
      console.log(location.href);

      // 2.获取URL信息
      console.log(location.hostname);
      console.log(location.host);
      console.log(location.protocol);
      console.log(location.port);
      console.log(location.pathname);
      console.log(location.search);
      console.log(location.hash);

      // 3.location方法
      location.assign("#");
      location.replace("#");
      // location.reload(false)

      // 4.URLSearchParams
      var urlSearchString = location.search;
      var searchParams = new URLSearchParams(urlSearchString);
      console.log(searchParams.get("name"));
      console.log(searchParams.get("age"));
      console.log(searchParams.has("height"));
```



### History对象

history对象允许我们访问浏览器曾经的会话历史记录。
有两个属性:

- length:会话中的记录条数;
- state:当前保留的状态值; 

有五个方法:

-  back():返回上一页，等价于history.go(-1);
-  forward():前进下一页, 等价于history.go(1);
-  go():加载历史中的某一页;
-  pushState():打开一个指定的地址;
-  replaceState():打开一个新的地址，并且使用replace;

history和hash目前是vue、react等框架实现路由的底层原理

```js
// 前端路由核心：修改了URL，但是页面不刷新
      // 1> 修改hash值
      // 2> 修改history

      // 1.history对应的属性
      console.log(history.length);
      console.log(history.state);

      // 2.修改history
      var bntEl = document.querySelector("button");
      bntEl.onclick = function () {
        // history.pushState({ name: "why", age: 18 }, "", "/lyc");
        history.replaceState({ name: "why", age: 18 }, "", "/lyc");
      };
      var backBtnEl = document.querySelector(".back");
      backBtnEl.onclick = function () {
        history.back();
      };
```



**前端路由原理**

改变URL而不转跳,监听URL变化更新组件

![image-20230109132038385](http://img.roydust.top/img/202301091320459.png)



### navigator对象(很少使用)

navigator对象表示用户代理的状态和标识等信息。

![image-20230109134919449](http://img.roydust.top/img/202301091349519.png)



### screen对象(很少使用)

screen主要记录的是浏览器窗口外面的客户端显示器的信息:

- 比如屏幕的逻辑像素screen.width、screen.height;

![image-20230109135313505](http://img.roydust.top/img/202301091353574.png)





### 认识JSON

在目前的开发中，JSON是一种非常重要的数据格式，它并不是编程语言,而是一种可以在服务器和客户端之间传输的数据格式。

JSON的全称是JavaScript Object Notation (JavaScript对象符号) :

- JSON是由Douglas Crockford构想和设计的**一种轻量级资料交换格式**，算是JavaScript的一个子集;
- 但是虽然JSON被提出来的时候是主要应用JavaScript中，但是目前已经**独立于编程语言，可以在各个编程语言中使用**;
- 很多编程语言都实现了将**JSON转成对应模型**的方式;

其他的传输格式:

- XML:在早期的网络传输中主要是使用XML来进行数据交换的,但是这种格式在解析、传输等各方面都弱于JSON,所以目前已经很少在被使用了;
- Protobuf:另外一个在网络传输中目前已经越来越多使用的传输格式是protobuf,但是直到2021年的..x版本才支持JavaScript,所以目前在前端使用的较少;

目前JSON被使用的场景也越来越多:

- **网络数据的传输JSON数据;**
- **项目的某些配置文件;**
- **非关系型数据库(NoSQL) 将json作为存储格式;**



#### JSON基本语法

JSON的顶层支持三种类型的值:

- **简单值:** 数字(Number) 、字符串(String, 不支持单引号)、布尔类型(Boolean) 、 null类型;
- **对象值:** 由key. value组成，key是字符串类型，并且必须添加双引号,值可以是简单值、对象值、数组值;
- **数组值:** 数组的值可以是简单值、对象值、数组值;

```json
{
  "name": "lyc",
  "age": 18,
  "friend": {
    "name": "kel",
    "age": null
  }
}
```



#### JSON方法

某些情况下我们希望将JavaScript中的复杂类型转化成JSON格式的字符串,这样方便对其进行处理:

- 比如我们希望将一个对象保存到localStorage中;
- 但是如果我们直接存放一个对象，这个对象会被转化成[object Object] 格式的字符串，并不是我们想要的结果;

在ES5中引用了JSON全局对象，该对象有两个常用的方法:

- **stringify方法**: 将JavaScript类型转成对应的JSON字符串;
- **parse方法:** 解析JSON字符串，转回对应的JavaScript类型;

```js
      var obj = {
        name: "lyc",
        friend: {
          name: "kei",
        },
      };

      // 1.将obj对象进行序列化
      var objJSONString = JSON.stringify(obj);
      console.log(objJSONString);

      // 2.将对象存储到localStorage
      localStorage.setItem("info", objJSONString);
      console.log(localStorage.getItem("info"));

      // 3.将字符转回到对象(反序列化)
      var newObj = JSON.parse(localStorage.getItem("info"));
      console.log(newObj);
```



JSON.stringify()方法将一个JavaScript 对象或值转换为JSON字符串:

- 如果指定了一-个replacer函数，则可以选择性地替换值;
- 如果指定的replacer是数组，则可选择性地仅包含数组指定的属性;

```js
      var objJSONString = JSON.stringify(obj, function (key, value) {
        if (key === "name") {
          return "coder";
        }
        return value;
      });
      console.log(objJSONString);
```



如果对象本身包含toJSON方法，那么会直接使用toJSON方法的结果:

```js
      var obj = {
        name: "lyc",
        friend: {
          name: "kei",
        },
        toJSON: function () {
          return "lyc coder"; //lyc coder
        },
      };
```



JSON.parse()方法用来解析JSON字符串,构造由字符串描述的JavaScript值或对象。

- 提供可选的reviver函数用以在返回之前对所得到的对象执行变换(操作)。

```js
      var newObj = JSON.parse(
        localStorage.getItem("info"),
        function (key, value) {
          if (key === "age") {
            return value + 2;
          }
          return value;
        }
      );
      console.log(newObj);
```

