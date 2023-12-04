# 重学Webpack

## 跨域问题

### 跨域问题和解决方案

### 1.1.跨域产生的原因

跨域是指去向一个为非本origin(协议、域名、端口任意一个不同)的目标地址发送请求的过程，这样之所以会产生问题是因为浏览器的同源策略限制。看起来同源策略影响了我们开发的顺畅性.实则不然,同源策略存在的必要性之一是为了隔离攻击。

浏览器有一个重要的安全策略，称之为「同源策略」

其中，源=协议+主机+端口源=协议+主机+端口，**两个源相同，称之为同源，两个源不同，称之为跨源或跨域**

比如：

| 源 1                                                         | 源 2                                                         | 是否同源 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------- |
| [www.baidu.com](https://link.juejin.cn/?target=http%3A%2F%2Fwww.baidu.com) | [www.baidu.com/news](https://link.juejin.cn/?target=http%3A%2F%2Fwww.baidu.com%2Fnews) | ✅        |
| [www.baidu.com](https://link.juejin.cn/?target=https%3A%2F%2Fwww.baidu.com) | [www.baidu.com](https://link.juejin.cn/?target=http%3A%2F%2Fwww.baidu.com) | ❌        |
| [http://localhost:5000](https://link.juejin.cn/?target=http%3A%2F%2Flocalhost%3A5000) | [http://localhost:7000](https://link.juejin.cn/?target=http%3A%2F%2Flocalhost%3A7000) | ❌        |
| [http://localhost:5000](https://link.juejin.cn/?target=http%3A%2F%2Flocalhost%3A5000) | [http://127.0.0.1:5000](https://link.juejin.cn/?target=http%3A%2F%2F127.0.0.1%3A5000) | ❌        |
| [www.baidu.com](https://link.juejin.cn/?target=http%3A%2F%2Fwww.baidu.com) | [baidu.com](https://link.juejin.cn/?target=http%3A%2F%2Fbaidu.com) | ❌        |

放到同一个服务器是没有跨域的



### 1.2.跨域问题的解决方案

#### 1.jsonp

最早的解决方案之一就是jsonp,实现方式是通过script标签传递数据，因为script请求不会被同源策略禁止，所以通过script标签去请求跨域数据，并且在script的cb对应func中实现对数据的获取是可行的,当然这种方式需要后端进行配合，后端在前端进行对应请求的时候返回对应的jsonp格式的数据 php案例如下:

```php
<?php
header('Content-type: application/json');
//获取回调函数名
$jsoncallback = htmlspecialchars($_REQUEST ['jsoncallback']);
//json数据
$json_data = '["customername1","customername2"]';
//输出jsonp格式的数据
echo $jsoncallback . "(" . $json_data . ")";
?>

```

  客户端用法如下:

```xml
 <script type="text/javascript">
		function callbackFunction(result, methodName)
        {
            ///result 指向对应数据
        }
</script>
<script type="text/javascript" src="http://www.runoob.com/try/ajax/jsonp.php?jsoncallback=callbackFunction"></script>
```

#### 2.CORS

CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。

整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

因此，实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。

> CORS概念

支持CORS请求的浏览器一旦发现ajax请求跨域，会对请求做一些特殊处理，对于已经实现CORS接口的服务端，接受请求，并做出回应。

有一种情况比较特殊，如果我们发送的跨域请求为“非简单请求”，浏览器会在发出此请求之前首先发送一个请求类型为OPTIONS的“预检请求”，验证请求源是否为服务端允许源，这些对于开发这来说是感觉不到的，由浏览器代理。

总而言之，客户端不需要对跨域请求做任何特殊处理。

> 简单请求与非简单请求

浏览器对跨域请求区分为“简单请求”与“非简单请求”

“简单请求”满足以下特征：

```
（1) 请求方法是以下三种方法之一：
     HEAD
     GET
     POST
（2）HTTP的头信息不超出以下几种字段：
     Accept
     Accept-Language
     Content-Language
     Last-Event-ID
     Content-Type：
     application/x-www-form-urlencoded、 multipart/form-data、text/plain
```

不满足这些特征的请求称为“非简单请求”，例如：content-type=applicaiton/json , method = PUT/DELETE...

>预检请求

**预检请求**就是在跨域的时候设置了对应的需要预检的内容,结果上会在普通跨域请求前添加了个options请求，用来检查前端headers的修改是否在后端允许范围内。 触发预检请求在跨域开发中会碰到的主要情况如下

1. 首先methods设置 **PUT**、**DELETE**、**CONNECT**、**OPTIONS**、**TRACE**会导致预检请求
2. 设置了**Accept**、**Accept-Language**、**Content-Language**、**Content-Type** 之外的headers中任一的配置，比如常见的token:authorization,缓存机制cache-contorl
3. **Content-Type**设置了简单请求不允许的值，如常用的application/json

下面是一段浏览器的JavaScript脚本。

> ```javascript
> var url = 'http://api.alice.com/cors';
> var xhr = new XMLHttpRequest();
> xhr.open('PUT', url, true);
> xhr.setRequestHeader('X-Custom-Header', 'value');
> xhr.send();
> ```

上面代码中，HTTP请求的方法是`PUT`，并且发送一个自定义头信息`X-Custom-Header`。

浏览器发现，这是一个非简单请求，就自动发出一个"预检"请求，要求服务器确认可以这样请求。下面是这个"预检"请求的HTTP头信息。

> ```http
> OPTIONS /cors HTTP/1.1
> Origin: http://api.bob.com
> Access-Control-Request-Method: PUT
> Access-Control-Request-Headers: X-Custom-Header
> Host: api.alice.com
> Accept-Language: en-US
> Connection: keep-alive
> User-Agent: Mozilla/5.0...
> ```

"预检"请求用的请求方法是`OPTIONS`，表示这个请求是用来询问的。头信息里面，关键字段是`Origin`，表示请求来自哪个源。

除了`Origin`字段，"预检"请求的头信息包括两个特殊字段。

**（1）Access-Control-Request-Method**

该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是`PUT`。

**（2）Access-Control-Request-Headers**

该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段，上例是`X-Custom-Header`。



##### **预检请求的回应**

那么预检请求我们需要如何处理呢?	

预检请求就需要后端设置更多的respones headers了，常用如下：

```makefile
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
```

上面的HTTP回应中，关键的是`Access-Control-Allow-Origin`字段，表示`http://api.bob.com`可以请求数据。该字段也可以设为星号，表示同意任意跨源请求。

**（1）Access-Control-Allow-Methods**

该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。

**（2）Access-Control-Allow-Headers**

如果浏览器请求包括`Access-Control-Request-Headers`字段，则`Access-Control-Allow-Headers`字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。

**（3）Access-Control-Allow-Credentials**

该字段与简单请求时的含义相同。

**（4）Access-Control-Max-Age**

该字段可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是20天（1728000秒），即允许缓存该条回应1728000秒（即20天），在此期间，不用发出另一条预检请求。

除此之外，后端还需要设置对options请求的判断,我在node中间件中添加的判断如下:

```ini
if (req.method == 'OPTIONS') {
    res.send(200);
  } else {
    next();
  }

```

  更多详细cors内容可见:[developer.mozilla.org/en-US/docs/…](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FHTTP%2FCORS)

实现CORS的几种方式

1. 本地代理
2. nodejs中间件
3. nginx代理

#### 3.CORS与JSONP的比较

CORS与JSONP的使用目的相同，但是比JSONP更强大。

JSONP只支持`GET`请求，CORS支持所有类型的HTTP请求。JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据。





#### 1.3 CORS实现方式

##### 本地代理

  在dva中的实现方式是在.webpackrc中添加如下代码

```json
 "proxy": {
    "/api": {
      "target": "http://127.0.0.1:8988/",
      "changeOrigin": true,
      "pathRewrite": { "^/api" : "" }
    }
  }

```

  /api代表代理的路径名，target代表代理的地址，changeOrigin代表更改发出源地址为target，pathRewrite代表路径重写，别的脚手架直接加载webpack配置文件即可



##### nodejs跨域中间件

具体实现过程我是使用[express](https://link.juejin.cn/?target=http%3A%2F%2Fwww.expressjs.com.cn%2F)+http-proxy-middleware

1. 用express脚手架生成express模具

```ini
npm install express-generator -g
express --view=pug myapp
```

1. 设置一个全局路由拦截

```lua
app.all('*', function (req, res, next) {
res.header('Access-Control-Allow-Origin', '*');
if (req.method == 'OPTIONS') {
  res.send(200);
} else {
  next();
}
})
```

1. 再设置对应的代理逻辑

```javascript
var options = {
  target: 'https://xxxx.xxx.xxx/abc/req',
  changeOrigin: true,
  pathRewrite: (path,req)=>{
    return path.replace('/api','/')
  }
}
app.use('/api', proxy(options));
```

1. 进入bin/www中设置对应的端口，或者在process.env.PORT设置port启动值

```ini
var port = normalizePort(process.env.PORT || '7002');
```

1. 启动脚手架

```ini
DEBUG=myapp:* npm start
```

启动代理后就可以直接在对应的项目中请求中间件实现跨域了



##### Nginx跨域代理

  [Nginx](https://link.juejin.cn/?target=http%3A%2F%2Fnginx.org%2Fen%2Fdocs%2F)是国外大神实现用的用于反向代理的异步web服务器 他除了用于反向代理以外还可以用于负载均衡、HTTP缓存 接下来来介绍安装方式 首先安装Nginx要安装Homebrew， 然而我的mac并没有Homebrew，那么还需要先用Ruby安装一下这个包管理器

```shell
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

  查看下brew是否安装成功，成功后可以顺利的安装Nginx啦

```
brew -v
brew install nginx
nginx -v
```

  Nginx的常用命令有这些

```r
#查看版本，以及配置文件地址
nginx -V
#查看版本 
nginx -v
#指定配置文件
nginx -c filename
#帮助
nginx -h
#重新加载配置|重启|停止|退出 nginx
nginx -s reload|reopen|stop|quit
#打开 nginx
sudo nginx
#测试配置是否有语法错误
sudo nginx -t
```

  然后就要进入Nginx配置文件

```bash
sudo vim /usr/local/etc/nginx/nginx.conf

```

  ps:nginx-http常见配置项如下

```ini
http {
    #导入类型配置文件
    include       mime.types;
    #设定默认类型为二进制流
    default_type  application/octet-stream;
    #启用sendfile()函数
    sendfile        on;
    #客户端与服务器连接的超时时间为65秒，超过65秒，服务器关闭连接
    keepalive_timeout  65;
    #是否开启gzip，默认关闭
    #gzip  on;
    #一个server块
    server {
        #服务器监听的端口为80
        listen       80;
        #服务器名称为localhost，我们可以通过localhost来访问这个server块的服务
        server_name  localhost;
        #location块，它存放在server块当中，location会尝试根据用户请求中的URI来匹配上面的/uri表达式，如果可以匹配，就选择location {}块中的配置来处理用户请求。
        location / {
            #以root方式设置资源路径，它与alias的不同请见下面的 http模块中文件路径定义
            root   html;
            #默认访问的页面，从左依次找到右，直到找到这个文件，然后返回结束请求
            index  index.html index.htm;
            #设置错误页面，对应的错误码是404，错误页面是/Users/user/Sites/404.html
            error_page 404  /404.html;
        }
    }
    include servers/*;
}

```

  接下来就是重要的实现反向代理的方式了 这里介绍的主要展示方式是如何在线上设置代理，所以代理的入口是静态资源

```ini
server {
        listen       80;
	server_name  localhost;
	location / {
            root   /Users/abc/dist/;
            index  index.html index.htm;
        }

        location /api/ {
                proxy_pass  https://xxx.xxx.xxx/req/;
        }
}

```

  location中的后的内容会尝试根据用户请求中的URI来匹配上面的/uri表达式，如果可以匹配，就选择location {}块中的配置来处理用户请求

  本项目中第一个location用于指向静态资源位置 root:目录,index:入口文件,第二个location用于进行api的跨域指向

  如果你想要对不同的端口实现代理，可以设置多个server listen同一个端口，根据server_name判断请求来源，根据location 设置代理去向

## webpack配置

### Mode配置

Mode配置选项，可以告知webpack使用相应模式的内置优化:

- 默认值是production (什么都不设置的情况下) ;
- 可选值有: 'none'| 'development'| production';

这几个选项有什么样的区别呢?

![image-20230110143128473](http://img.roydust.top/img/202301101431553.png)

Mode配置代表跟多默认设置

![image-20230110143243790](http://img.roydust.top/img/202301101432861.png)



### 认识source-map

我们的代码通常运行在浏览器上时，是通过打包压缩的:

- 也就是真实跑在浏览器.上的代码，和我们编写的代码其实是有差异的;
- 比如ES6的代码可能被转换成ES5;
- 比如对应的代码行号、列号在经过编译后肯定会不一致;
- 比如代码进行丑化压缩时，会将编码名称等修改;
- 比如我们使用了TypeScript等方式编写的代码，最终转换成JavaScript;

但是，当代码报错需要调试时(debug) ，调试转换后的代码是很困难的

但是我们能保证代码不出错吗?不可能。

那么如何可以调试这种转换后不一致的代码呢?答案就是source-map

- source-map是从**已转换的代码，映射到原始的源文件;**
- 使浏览器可以**重构原始源**并**在调试器中显示重建的原始源;**



#### 如何使用source-map

如何可以使用source-map呢?两个步骤:

- 第一步:根据源文件,生成source-map文件，webpack在打包时， 可以通过配置生成source-map;

- 第二步:在转换后的代码，最后添加一一个注释，它指向sourcemap;

  ```js
  //# sourceMappingURL=bundle.js.map
  ```

浏览器会根据我们的注释，查找相应的source-map, 并且根据source-map还原我们的代码，方便进行调试。



#### 分析source-map

最初source-map生成的文件大小是原始文件的10倍，第二二版减少了约50%，第三版又减少了50%，所以目前一个133kb的文件,最终的source-map的大小大概在300kb。

目前的source-map长什么样子呢?

- **version**:当前使用的版本，也就是最新的第三版;
- **sources**:从哪些文件转换过来的source-map和打包的代码(最初始的文件) ;
- **names**:转换前的变量和属性名称(因为我目前使用的是development模式,所以不需要保留转换前的名称) ; 
- **mappings**: source-map用来和源文件映射的信息(比如位置信息等)，一串base64 VLQ (veriable-length quantity可变长度值)编码;
- **file**:打包后的文件(浏览器加载的文件) ;
- **sourceContent**:转换前的具体代码信息(和sources是对应的关系) ;
- **sourceRoot**:所有的sources相对的根目录;

```json
{
  "version": 3,
  "file": "bundle.js",
  "mappings": "wCAAA,SAASA,EAAIC,EAAMC,GACjB,OAAOD,EAAOC,CAChB,C,4BCDIC,EAA2B,CAAC,EAGhC,SAASC,EAAoBC,GAE5B,IAAIC,EAAeH,EAAyBE,GAC5C,QAAqBE,IAAjBD,EACH,OAAOA,EAAaE,QAGrB,IAAIC,EAASN,EAAyBE,GAAY,CAGjDG,QAAS,CAAC,GAOX,OAHAE,EAAoBL,GAAUI,EAAQA,EAAOD,QAASJ,GAG/CK,EAAOD,OACf,CCrBAJ,EAAoBO,EAAI,CAACH,EAASI,KACjC,IAAI,IAAIC,KAAOD,EACXR,EAAoBU,EAAEF,EAAYC,KAAST,EAAoBU,EAAEN,EAASK,IAC5EE,OAAOC,eAAeR,EAASK,EAAK,CAAEI,YAAY,EAAMC,IAAKN,EAAWC,IAE1E,ECNDT,EAAoBU,EAAI,CAACK,EAAKC,IAAUL,OAAOM,UAAUC,eAAeC,KAAKJ,EAAKC,GCClFhB,EAAoBoB,EAAKhB,IACH,oBAAXiB,QAA0BA,OAAOC,aAC1CX,OAAOC,eAAeR,EAASiB,OAAOC,YAAa,CAAEC,MAAO,WAE7DZ,OAAOC,eAAeR,EAAS,aAAc,CAAEmB,OAAO,GAAO,E,MCL9D,MAAM,IAAE3B,GAAQ,EAAQ,KAGxB4B,QAAQC,IADQ,eAGhBD,QAAQC,IAAIC,SAEZF,QAAQC,IAAI7B,EAAI,GAAI,KAGlB4B,QAAQC,IAAI,e",
  "sources": [
    "webpack://01-source-map/./src/utils/math.js",
    "webpack://01-source-map/webpack/bootstrap",
    "webpack://01-source-map/webpack/runtime/define property getters",
    "webpack://01-source-map/webpack/runtime/hasOwnProperty shorthand",
    "webpack://01-source-map/webpack/runtime/make namespace object",
    "webpack://01-source-map/./src/main.js"
  ],
  "sourcesContent": [
    "function add(num1, num2) {\r\n  return num1 + num2\r\n}\r\n\r\n\r\nexport {\r\n  add\r\n}",
    "// The module cache\nvar __webpack_module_cache__ = {};\n\n// The require function\nfunction __webpack_require__(moduleId) {\n\t// Check if module is in cache\n\tvar cachedModule = __webpack_module_cache__[moduleId];\n\tif (cachedModule !== undefined) {\n\t\treturn cachedModule.exports;\n\t}\n\t// Create a new module (and put it into the cache)\n\tvar module = __webpack_module_cache__[moduleId] = {\n\t\t// no module.id needed\n\t\t// no module.loaded needed\n\t\texports: {}\n\t};\n\n\t// Execute the module function\n\t__webpack_modules__[moduleId](module, module.exports, __webpack_require__);\n\n\t// Return the exports of the module\n\treturn module.exports;\n}\n\n",
    "// define getter functions for harmony exports\n__webpack_require__.d = (exports, definition) => {\n\tfor(var key in definition) {\n\t\tif(__webpack_require__.o(definition, key) && !__webpack_require__.o(exports, key)) {\n\t\t\tObject.defineProperty(exports, key, { enumerable: true, get: definition[key] });\n\t\t}\n\t}\n};"
  ],
  "names": [
    "add",
    "num1",
    "num2",
    "log",
    "address"
  ],
  "sourceRoot": ""
}
```

 

#### 生成source-map

如何在使用webpack打包的时候选择不同的devtool，可以生成对相应的source-map

- webpack为我们提供了非常多的选项(目前是26个)，来处理source-map; .
- https://webpackdocschina.org/configuration/devtool/
- 选择不同的值，生成的source-map会稍微有差异, 打包的过程也会有**性能的差异**，可以根据不同的情况进行选择;

下面几个值不会生成source-map

- false: 不使用source-map, 也就是没有任何和source-map相关的内容。
- none: production模式下的默认值(什么值都不写)， 不生成source-map.
- eval: development模式下的默认值， 不生成source-map的
  - 但是它会在eval执行的代码中，添加//# sourceURL=;
  - 它会被浏览器在执行时解析，并且在调试面板中生成对应的一些文件目录,方便我们调试代码;
  - 但是它生成的文件位置可能会不那么准确

如果选择devtool: "source-map"

- 生成一一个独立的source-map文件，并且在bundle文件中有一个注释,指向source-map文件;

- bundle文件中有如下的注释:

  ```js
  //# sourceMappingURL=bundle.js.map
  ```

- 开发工具会根据这个注释找到source-map文件，并且解析,位置十分准确

![image-20230110161617616](http://img.roydust.top/img/202301101616669.png)



不常见的值:

1. eval-source-map:添加到eval 两数的后面
2. inline-source-map:添加到文件的后面
3. cheap-source-map(dev环境):低开销，更加高效
4. cheap-module-source-map: 和cheap-source-map比如相似，但是对来自Loader的source-map处理的更好
5. hidden-source-map:会生成sourcemap文件，但是 不会对source map文件进行引用





## 深入浅出Babel

### 为什么需要Babel

事实上，在开发中我们很少直接去接触babel,但是babel对于前端开发来说，目前是不可缺少的-部分:

- 开发中，我们想要使用ES6+的语法，想要使用TypeScript, 开发React项目，它们都是离不开Babel的;
- 所以，学习Babel对于我们理解代码从编写到线上的转变过程至关重要;

那么，Babel到底是什么呢?

- Babel是一个工具链， 主要用于旧浏览器或者缓解中将ECMAScript 2015+代码转换为向后兼容版本的JavaScript;
- 包括:语法转换、源代码转换、Polyfill实现目 标环境缺少的功能等;



### Babel命令行使用

babel本身可以作为一个独立的工具(和postcss一样) ，不和webpack等构建工具配置来单独使用。
如果我们希望在命令行尝试使用babel,需要安装如下库:

- @babel/core: babel的核心代码，必须安装;
-  @babel/cli:可以让我们在命令行使用babel;

```
npm install @babel/cli @babel/core
```

使用babel来处理我们的源代码: .

- src:是源文件的目录;

- --out-dir:指定要输出的文件夹dist; 

  ```
   npx babel  ./src --out-dir ./dist
  ```



比如我们需要转换箭头函数，那么我们就可以使用箭头函数转换相关的插件:

```
npm install @babel/plugin-transform-arrow-functions
```

```
npx babel  ./src --out-dir ./dist --plugins=@babel/plugin-transform-arrow-functions
```

查看转换后的结果:我们会发现const并没有转成var

- 这是因为plugin-transform-arrow-functions,并没有提供这样的功能;
- 我们需要使用plugin-transform-block-scoping 来完成这样的功能;

```
npm install @babel/plugin-transform-block-scoping
```

```
 npx babel  ./src --out-dir ./dist --plugins=@babel/plugin-transform-block-scoping,@babel/plugin-transform-arrow-functions
```



### Babel的预设

但是如果要转换的内容过多，-个个设置是比较麻烦的，我们可以使用预设(preset) :

- 后面我们再具体来讲预设代表的含义;

安装@babel/preset-env预设:
```
npm install @babel/preset-env -D
```

执行如下命令:

```
npx babel  ./src --out-dir ./dist --presets=@babel/preset-env
```



### Babel的底层原理

babel是如何做到将我们的一段代码(ES6. TypeScript. React) 转成另外一段代码(ES5) 的呢?

- 从一种源代码(原生语言)转换成另一种源代码(目标语言)，这是什么的工作呢?
- 就是编译器，事实上我们可以将babel看成就是一个编译器。
- Babel编译器的作用就是将我们的源代码，转换成浏览器可以直接识别的另外一段源代码;

Babel也拥有编译器的工作流程:

- 解析阶段(Parsing)
- 转换阶段(Transformation)
- 生成阶段(Code Generation)

![image-20230110222821794](http://img.roydust.top/img/202301102228904.png)

Babel的执行阶段：

![image-20230110223405660](http://img.roydust.top/img/202301102234718.png)



### webpack和babel

- webpack是模块化内容
- babel是代码转化内容

![image-20230111143726198](http://img.roydust.top/img/202301111437305.png)



#### **在webpack中使用babel**

在实际开发中，我们通常会在构建工具中通过配置babel来对其进行使用的,比如在webpack中。

那么我们就需要去安装相关的依赖:

- 如果之前已经安装了@babel/core,那么这里不需要再次安装;

```
npm install babel-loader @babel/core
```

我们可以设置-个规则，在加载js文件时，使用我们的babel:

```js
const path = require("path")

module.exports = {
  // 设置环境模式
  mode: "development",
  devtool: false,
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "./build"),
    filename: "bundle.js",
    // 重新打包时，现将之前打包的文件夹删掉
    clean: true
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: "babel-loader",
          // 使用插件
          options: {
            // plugins: [
            //   "@babel/plugin-transform-arrow-functions",
            //   "@babel/plugin-transform-block-scoping"
            // ]

            // 使用预设
            presets: [
              "@babel/preset-env"
            ]
          }
        }
      }
    ]
  }
}
```



#### babel-preset

如果我们一个个去安装使用插件，那么需要手动来管理大量的babel插件,我们可以直接给webpack提供一个preset,webpack会根据我们的预设来加载对应的插件列表，并且将其传递给babel。

比如常见的预设有三个:

- env
- react
- TypeScript



### Babel的配置文件

像之前一样，我们可以将babel的配置信息放到-个独立的文件中，babel给我们提供了两种配置文件的编写:

- babel.config.json (或者js, .cjs, .mjs) 文件; .
- .babelrc.json (或者babelrc, js, .cjs, .mjs) 文件;

它们两个有什么区别呢?目前很多的项目都采用了多包管理的方式(babel本身、 element-plus. umi等) ;

- **.babelrc.json**: 早期使用较多的配置方式，但是对于配置Monorepos项目是比较麻烦的;
- **babel.config.json (babel7)** :可以直接作用于Monorepos项目的子包，更加推荐;

![image-20230111163355708](http://img.roydust.top/img/202301111633777.png)







## 浏览器兼容性问题

我们来思考一个问题:开发中，浏览器的兼容性问题，我们应该如何去解决和处理?

- 当然这个问题很笼统，这里我说的兼容性问题**不是指屏幕大小的变化适配**;
- 我这里指的兼容性是**针对不同的浏览器支持的特性**:比如css特性、js语法之间的兼容性;

我们知道市面上有大量的浏览器:

- 有Chrome、Safari、 IE、 Edge、 Chrome for Android、UC Browser、QQ Browser等等; .
- 它们的市场占率是多少?我们要不要兼容它们呢?

其实在很多的脚手架配置中，都能看到类似于这样的配置信息: 

```
 > 1%
 last 2 versions
 not dead
```



![image-20230111152234208](http://img.roydust.top/img/202301111522297.png)



### 认识Browserslist工具

但是有一个问题，我们如何可以在css兼容性和js兼容性下共享我们配置的兼容性条件呢?

- 就是当我们设置了一个条件: > 1%;
- 我们表达的意思是css要兼容市场占有率大于1%的浏览器，js也要兼容市场占有率大于1%的浏览器;
- 如果我们是通过工具来达到这种兼容性的，比如我们讲到的postcss-preset-env、 babel. autoprefixer等

如何可以让他们共享我们的配置呢?

- 这个问题的答案就是Browserslist;

Browserslist是什么? Browserslist是**一个在不同的前端工具之间，共享目标浏览器和Node.js版本的配置:**

- Autoprefixer
- Babel
- postcss-preset-env
- eslint-plugin-compat
- stylelint-no-unsupported-browser-features
- postcss-normalize
- obsolete-webpack-plugin



### 浏览器查询过程

我们可以编写类似于这样的配置:

```
 > 1%
 last 2 versions
 not dead
```

那么之后，这些工具会根据我们的配置来获取相关的浏览器信息，以方便决定是否需要进行兼容性的支持:

- 条件查询使用的是caniuse-lite的工具，这个工具的数据来自于caniuse的网站上;



### Browserslist编写规则

那么在开发中，我们可以编写的条件都有哪些呢? (加粗部分是最常用的)

- **defaults: Browserslist的默认浏览器(> 0.5%, last 2 versions, Firefox ESR, not dead)。**
- **5%:通过全局使用情况统计信息选择的浏览器版本。>=, <和<=工作过。**

- **dead: 24个月内没有官方支持或更新的浏览器。现在是IE 10, IE Mob 11, BlackBerry 10, BlackBerry 7，Samsung 4和OperaMobile 12.1。**
- **last 2 versions:每个浏览器的最后2个版本。**
- node 10和node 10.4:选择最新的Node.js10.x.x 或10.4.x版本。
- ios 7:直接使用iOS浏览器版本7。
- extends browserslist-config-mycompany:从browserslist-config-mycompanynpm包中查询 。
- supports es6-module:支持特定功能的浏览器。
- browserslist config:在Browserslist配置中定义的浏览器。
- since 2015或last 2 years 
- unreleased versions或unreleased Chrome versions: Alpha和Beta版本。
- not ie <= 8:排除先前查询选择的浏览器。



配置Browserslist

我们如何可以配置browserslist呢?两种方案:

- 方案一:在package.json中配置;

  ![image-20230111154823104](http://img.roydust.top/img/202301111548146.png)

- 方案二:单独的一个配置文件.browserslistrc文件;

  ![image-20230111160128282](http://img.roydust.top/img/202301111601326.png)



#### 默认配置和条件

如果没有配置，那么也会有一个默认配置:

![image-20230111160258554](http://img.roydust.top/img/202301111602590.png)

我们编写了多个条件之后，多个条件之间是什么关系呢?

![image-20230111160313250](http://img.roydust.top/img/202301111603290.png)



### 设置目标浏览器 browserslist

我们最终打包的JavaScript代码，是需要跑在目标浏览器上的，那么如何告知babel我们的目标浏览器呢?

- browserslist工具
-  target属性

之前我们已经使用了browserslist工具,我们可以对比一下不同的配置, 打包的区别:

![image-20230111161332397](http://img.roydust.top/img/202301111613458.png)

```js
            // 使用预设
            presets: [
              ["@babel/preset-env", {
                // 在开发中针对babel的浏览器兼容查询使用browserslist工具，而不是设置target
                // 因为browserslist工具，可以在多个前端工具之间进行共享浏览器兼容性（postcss/babel）
                // targets: ">5%"
              }]
            ]
```

那么,如果两个同时配置了，哪一个会生效呢?

- 配置的targets属性会覆盖browserslist;
- 但是在开发中，更推荐通过browserslist来配置，因为类似于postcss工具，也会使用browserslist, 进行统一浏览器的适配

 

### Stage-X的preset（了解）

要了解Stage-X,我们需要先了解一下TC39的组织: .

- TC39是指技术委员会(Technical Committee)第39号;
- 它是ECMA的一部分，ECMA是"ECMAScript” 规范下的JavaScript语言标准化的机构;
- ECMAScript规范定义了JavaScript如何一步一步的进化、 发展;

TC39遵循的原则是:**分阶段加入不同的语言特性，新流程涉及四个不同的Stage**

-  Stage 0: strawman (稻草人)，任何尚未提交作为正式提案的讨论、想法变更或者补充都被认为是第0阶段的"稻草人";
-  Stage 1: proposal (提议) ，提案已经被正式化，并期望解决此问题,还需要观察与其他提案的相互影响;
-  Stage2: draft (草稿)，Stage 2的提案应提供规范初稿、草稿。此时，语言的实现者开始观察runtime的具体实现是否合理;
-  Stage 3: candidate (候补)， Stage 3提案是建议的候选提案。在这个高级阶段,规范的编辑人员和评审人员必须在最终规范上签字。Stage 3的提案不会有太大的改变，在对外发布之前只是修正一些问题;

- Stage 4: finished (完成) ，进入Stage 4的提案将包含在ECMAScript的下一一个修订版中;



在babel7之前(比如babel6中) ,我们会经常看到这种设置方式:

- 它表达的含义是使用对应的babel-preset-stage-x 预设;
- 但是从babel7开始，已经不建议使用了，建议使用preset-env来设置 ;

![image-20230111162226425](http://img.roydust.top/img/202301111622469.png)



## 认识Polyfill

Polyfill是什么呢?

- 翻译: 一种用于衣物、床具等的聚酯填充材料,使这些物品更加温暖舒适;
- 理解:更像是应该填充物(垫片)，一个补丁，可以帮助我们更好的使用JavaScript;

为什么时候会用到polyfill呢?

- 比如我们使用了一些语法特性 (例如: Promise, Generator, Symbol等以及实例方法例如Array.prototype.includes等)
- 但是某些浏览器压根不认识这些特性，必然会报错;
- 我们可以使用polyfill来填充或者说打一个补丁 ,那么就会包含该特性了;



![image-20230111163830814](http://img.roydust.top/img/202301111638881.png)



### 如何使用Polyfill？

babel7.4.0之前，可以使用@babel/polyfill的包，但是该包现在已经不推荐使用了:

![image-20230111164350543](http://img.roydust.top/img/202301111643590.png)

babel7.4.0之后，可以通过单独引入core-js和regenerator-runtime来完成polyfill的使用:

```
npm install core-js regenerator-runtime --save
```



我们需要在babel.config.js文件中进行配置，给preset-env配置一些属性:

**useBuiltIns:** 设置以什么样的方式来使用polyil;

**corejs:** 设置corejs的版本，目前使用较多的是3.x的版本，比如我使用的是3.8.x的版本;

- 另外corejs可以设置是否对提议阶段的特性进行支持;
- 设置proposals属性为true即可;



### useBuiltIns属性

useBuiltIns属性有三个常见的值
第一个值: **false**

- 打包后的文件不使用polyill来进行适配;
- 并且这个时候是不需要设置corejs属性的;

第二个值: **usage** (推荐)

- 会根据源代码中出现的语言特性，自动检测所需要的polyfill;
- 这样可以确保最终包里的polyfill数量的最小化，打包的包相对会小一些;
- 可以设置corejs属性来确定使用的corejs的版本;

```js
  presets: [
		["@babel/preset-env", {
      corejs: 3,
      useBuiltIns: "usage"
    }]
   ]
```

第三个值: **entry**

- 如果我们依赖的某一个库本身使用 了某些polyfill的特性，但是因为我们使用的是usage,所以之后用户浏览器可能会报错;
- 所以，如果你担心出现这种情况，可以使用entry;
- 并且需要在入口文件中添加import 'core-js/stable'; import 'regenerator-runtime/runtime'; .
- 这样做会根据browserslist 目标导入所有的polyfill,但是对应的包也会变大;

```js
  presets: [
    ["@babel/preset-env", {
      corejs: 3,
      useBuiltIns: "entry"
    }]
  ]
```





## Webpack对jsx支持

在我们编写react代码时，react使用的语法是jsx, jsx是可以直接使用babel来转换的。

对react jsx代码进行处理需要如”下的插件:

-  @babel/plugin-syntax-jsx
-  @babel/plugin-transform-react-jsx
-  @babel/plugin-transform-react-display-name

但是开发中，我们并不需要一个个去安装这些插件,我们依然可以使用preset来配置:

```
npm install @babel/preset-react -D
```

更改一下webpack.config.js

```js
  module: {
    rules: [
      // 针对jsx?代码进行babel处理
      {
        test: /\.jsx?$/, // x?:标识0或者1
        use: {
          loader: "babel-loader"
        }
      },
      ]
  }
```



再去babel.config.js中使用就可以了

```js
  presets: [
    ["@babel/preset-env", {
      // 在开发中针对babel的浏览器兼容查询使用browserslist工具，而不是设置target
      // 因为browserslist工具，可以在多个前端工具之间进行共享浏览器兼容性（postcss/babel）
      // targets: ">0.5%"
      // corejs: 3,
      // false 不使用我们的polyfill进行填充
      // useBuiltIns: "entry"
    }],
    // 对react代码进行处理
    ["@babel/preset-react"]
    ]
```





## Webpack对TypeScript支持

在项目开发中，我们会使用TypeScript来开发,那么TypeScript代码是 需要转换成JavaScript代码。
可以通过TypeScript的compiler来转换成JavaScript:

```
npm install typescript -D
```

另外TypeScript的编译配置信息我们通常会编写一个tsconfig.json文件:

```
tsc --init
```

生成文件如下

![image-20230111231219299](http://img.roydust.top/img/202301112312363.png)

之后我们可以运行npx tsc来编译自己的ts代码



#### 使用ts-loader

如果我们希望在webpack中使用TypeScript,那么我们可以使用ts-loader来处理ts文件:

```
npm install ts-loader -D
```

配置ts-loader:

```js
      // 针对ts代码进行处理
      {
        test: /\.ts$/,
        use: "ts-loader"
      }
```

之后，我们通过npm run build打包即可。



##### 使用babel-loader

除了可以使用TypeScript Compiler来编译TypeScript之外,我们也可以使用Babel:

- Babel是有对TypeScript进行支持;
- 我们可以使用插件: @babel/tranform-typescript; .
- 但是更推荐直接使用preset: @babel/preset-typescript;

我们来安装@babel/preset-typescript: 

```
npm install @babel/ preset-typescript -D
```



```js
  presets: [
    ["@babel/preset-env", {
      // 在开发中针对babel的浏览器兼容查询使用browserslist工具，而不是设置target
      // 因为browserslist工具，可以在多个前端工具之间进行共享浏览器兼容性（postcss/babel）
      // targets: ">0.5%"
      // corejs: 3,
      // false 不使用我们的polyfill进行填充
      // useBuiltIns: "entry"
    }],
    ["@babel/preset-react"],
    // 对ts进行babel处理
    ["@babel/preset-typescript", {
      corejs: 3,
      useBuiltIns: "usage"
    }]
  ]
```



#### ts-loader和babel-loader选择

那么我们在开发中应该选择ts-loader还是babel-loader呢?

使用ts-loader (TypeScript Compiler)

- 来直接编译TypeScript,那么只能将ts转换成js;
- 如果我们还希望在这个过程中添加对应的polyfill,那么ts-loader是无能为力的; .
- 我们需要借助于babel来完成polyill的填充功能;

使用babel-loader (Babel)

- 来直接编译TypeScript,也可以将ts转换成js, 并且可以实现polyill的功能;
- 但是babel-loader在编译的过程中，不会对类型错误进行检测;



#### **最佳实践**

![image-20230112002514381](http://img.roydust.top/img/202301120025473.png)

也就是说我们**使用Babel来完成代码的转换，使用tsc来进行类型的检查**。

但是，如何可以使用tsc来进行类型的检查呢?

- 在这里，我在scripts中添加了两个脚本，用于类型检查;
- 我们执行npm run type-check可以对ts代码的类型进行检测;
- 我们执行npm run type-check-watch可以实时的检测类型错误;

![image-20230112002928452](http://img.roydust.top/img/202301120029491.png)

就可以检测到类型错误的地方

![image-20230112003010053](http://img.roydust.top/img/202301120030147.png)



## Webpack本地服务器

### 本地服务器server

#### 为什么我们需要搭建本地服务器？

目前我们开发的代码，为了运行需要有两个操作:

- 操作一: npm run build,编译相关的代码; 
- 操作二:通过live server或者直接通过浏览器，打开index.htmI代码， 查看效果;

这个过程经常操作会影响我们的开发效率，我们希望可以做到，当文件发生变化时，可以自动的完成编译和展示; .

为了完成自动编译，webpack提供了几种可选的方式:

- webpack watch mode
- webpack-dev-server (常用)
- webpack-dev-middleware



#### webpack-dev-server

**上面的方式可以监听到文件的变化，但是事实上它本身是没有自动刷新浏览器的功能的:**

- 当然，目前我们可以在VSCode中使用live-server来完成这样的功能;
- 但是，我们希望在不适用live-server的情况下，可以具备live reloading (实时重新加载)的功能;

**安装webpack-dev-server**

```
npm install webpack-dev-server -D
```

修改配置文件，启动时可以加上参数

```js
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack ",
    "serve":"webpack serve",
  },
```

**webpack-dev-server在编译之后不会写入到任何输出文件，而是将bundle文件保留在内存中:**

- 事实上webpack- dev-server使用了-个库叫memfs (memory-fs webpack自己写的)



### server的静态资源

#### devServer的static

devServer中static对于我们直接访问打包后的资源其实并没有太大的作用，它的主要作用是如果我们打包后的资源，又依赖于其他的一些静态资源，那么就需要指定从哪里来查找这个内容:

- 比如在index.html中，我们需要依赖一-个abc.js文件，这个文件我们存放在public文件中;
- 在index.htmI中，我们应该如何去引入这个文件呢?
  - 比如代码是这样的: < script src="./public/abc.js"> < /script>;
  - 但是这样打包后浏览器是无法通过相对路径去找到这个文件夹的;
  - 所以代码是这样的: < script src= "/abc.js"> < /script>;
  - 但是我们如何让它去查找到这个文件的存在呢?设置static即可;

```js
  // webpack服务器配置
  devServer: {
    // 静态文件配置 默认为public 可以自定义文件夹
    static: ['public', 'content']
  },
```

需要使用时就直接./xxx就可以用

```html
 <link rel="icon" href="./icon.png">
```



### server的其他配置

#### host配置

host设置主机地址:

- 默认值是localhost;
- 如果希望其他地方也可以访问，可以设置为0.0.0.0;

localhost和0.0.0.0的区别: 

- localhost:本质上是一个域名，通常情况下会被解析成127.0.0.1;
- 127.0.0.1:回环地址(Loop Back Address),表达的意思其实是我们主机自己发出去的包，直接被自己接收;
  - 正常的数据库包经常应用层-传输层-网络层-数据链路层-物理层;
  - 而回环地址，是在网络层直接就被获取到了，是不会经常数据链路层和物理层的;
  - 比如我们监听127.0.0.1时，在同一个网段下的主机中，通过ip地址是不能访问的;
- 0.0.0.0:监听IPV4. 上所有的地址,再根据端口找到不同的应用程序;
  - 比如我们监听0.0.0.0时,在同一一个网段下的主机中，通过ip地址是可以访问的;

#### port

port设置监听的端口，默认情况下是8080



#### open

open是否打开浏览器: 

- 默认值是false,设置为true会打开浏览器;
- 也可以设置为类似于Google Chrome等值;



#### compress

compress是否为静态文件开启gzip compression:

- 默认值是false,可以设置为true;

![image-20230112011413281](http://img.roydust.top/img/202301120114360.png)

```js
  // webpack服务器配置
  devServer: {
    // 静态文件配置 默认为public
    static: ['public', 'content'],
    // 网关地址
    host: '0.0.0.0',
    // 端口号
    port: 8080,
    // 是否自动打开
    open: true,
    // 是否开启本地服务器代码压缩
    compress: true
  },
```





### server的proxy代理

proxy是我们开发中非常常用的一个配置选项，它的目的设置代理来解决跨域访问的问题:

- 比如我们的一-个api请求是http://ocalhost:8888,但是本地启动服务器的域名是http://ocalhost:8080,这个时候发送网络请求就会出现跨域的问题;

- 那么我们可以将请求先发送到-个代理服务器，代理服务器和API服务器没有跨域的问题，就可以解决我们的跨域问题了;

我们可以进行如下的设置:

- **target**:表示的是代理到的目标地址,比如/apihy/moment会被代理到htp://localhost:8888/api-hy/moment;
- **pathRewrite**:默认情况下，我们的/api-hy也会被写入到URL中，如果希望删除，可以使用pathRewrite; 
- **changeOrigin**:它表示是否更新代理后请求的headers中host地址;

```js
  // webpack服务器配置
  devServer: {
    // 本地服务代理
    proxy: {
      // 将目标前缀为/api 代理成需要的地址
      '/api': {
        target: "http://localhost:9000",
        // 重写请求地址
        pathRewrite: {
          "^/api": ""
        },
        changeOrigin: true
      }
    }
  },
```

发送网络请求

```js
// 8.发送网络请求获取数据
import axios from "axios";

axios.get('/api/users/list').then(res => {
  console.log(res.data);
})
```

![image-20230112013137501](http://img.roydust.top/img/202301120131595.png)



### changeOrigin作用

- 如果后端没有做校验，无论什么地址都可以拿到数据
- 如果做好了校验，那么可能只能是同地址才可以拿到数据
- 所以changeOrigin作用就是**修改我们去哪数据时的请求地址**，让服务器认为是同地址发送的请求

![image-20230112013853479](http://img.roydust.top/img/202301120138629.png)

**服务器接收请求图**

没有开启changeOrigin

![image-20230112014217756](http://img.roydust.top/img/202301120142820.png)

开启changeOrigin

![image-20230112014342736](http://img.roydust.top/img/202301120143799.png)



#### changeOrigin的解析

这个changeOrigin官方说的非常模糊，通过查看源码我发现其实是要修改代理请求中的headers中的host属性:

- 因为我们真实的请求，其实是需要通过http://localhost:8888来请求的; 
- 但是因为使用了代码，默认情况下它的值时http://localhost:8000;
- 如果我们需要修改,那么可以将changeOrigin设置为true即可;

![image-20230112014603192](http://img.roydust.top/img/202301120146265.png)



### historyApiFallback

historyApiFallback是开发中一个非常常见的属性，它主要的作用是**解决SPA页面在路由跳转之后，进行页面刷新时，返回404的错误。**

**boolean值:默认是false**

- 如果设置为true,那么在刷新时，返回404错误时，会自动返回index.html 的内容;

**object类型的值，可以配置rewrites属性:**

- 可以配置from来匹配路径,决定要跳转到哪一个页面;

事实上devServer中实现historyApiFallback功能是通过connect-history-api-fallback库的:

```js
  // webpack服务器配置
  devServer: {
		// 刷新路由转跳处理
    historyApiFallback: true
  },
```



## Webpack的性能优化

**webpack作为前端目前使用最广泛的打包工具，在面试中也是经常会被问到的。**

比较常见的面试题包括:

- 可以配置哪些属性来进行**webpack性能优化?**
- **前端有哪些常见的性能优化?** (问到前端性能优化时， 除了其他常见的,也完全可以从webpack来回答)

webpack的性能优化较多，我们可以对其进行分类:

- 优化一: **打包后的结果**，上线时的性能优化。(比如分包处理、 减小包体积、CDN服务器等)
- 优化二: **优化打包速度**,开发或者构建时优化打包速度。(比如exclude、 cache-loader等)

大多数情况下，我们会更加侧重于优化一, 这对于**线上的产品影响更大**。

**在大多数情况下webpack都帮我们做好了该有的性能优化:**

- 比如配置mode为production或者development时，默认webpack的配置信息;
- 但是我们也可以针对性的进行自己的项目优化;

![image-20230112015900374](http://img.roydust.top/img/202301120159518.png)



### 代码分离

代码分离(Code Splitting)是webpack一个非常重要的特性:

- 它主要的目的是**将代码分离到不同的bundle中**，之后我们可以按需加载，或者并行加载这些文件;
- 比如默认情况下，**所有的JavaScript代码(业务代码、第三方依赖、暂时没有用到的模块)在首页全部都加载**，就会影响首页的加载速度;
- 代码分离可以分出更小的bundle,以及**控制资源加载优先级,提供代码的加载性能;**

Webpack中常用的代码分离有三种:

- **入口起点:** 使用entry配置手动分离代码;
- **防止重复:** 使用Entry Dependencies或者SplitChunksPlugin去重和分离代码;
- **动态导入:** 通过模块的内联函数调用来分离代码;



**分包处理的优势和必要性**

![image-20230112142234672](http://img.roydust.top/img/202301121422842.png)



### webpack多入口依赖

入口起点的含义非常简单，就是配置多入口:

- 比如配置-个index.js和main.js的入口;
- 他们分别有自己的代码逻辑;

```js
  entry: {
    index: "./src/index.js",
    main: './src/main.js'
    },
    // 可以进行共享的
    shared: ["axios"]
 		},
    output: {
    path: path.resolve(__dirname, "./build"),
    // [name]为占位符
    filename: "[name]-bundle.js",
    clean: true
  },
```



#### 项目依赖

假如我们的index.js和main.js都依赖两个库: lodash、 dayjs

- 如果我们单纯的进行入口分离,那么打包后的两个bunlde都有会有一-份lodash和dayjs; .
- 事实上我们可以对他们进行共享;

```js
  entry: {
    index: {
      import: "./src/index.js",
      dependOn: "shared"
    },
    main: {
      import: './src/main.js',
      dependOn: "shared"
    },
    // 可以进行共享的
    shared: ["axios"]
  },
```



### webpack的动态导入(dynamic import)

另外一个代码拆分的方式是动态导入时，webpack提供了两种实现动态导入的方式:

- 第一-种，使用ECMAScript中的import()语法来完成,也是目前推荐的方式;
- 第二种，使用webpack遗留的require.ensure,目前已经不推荐使用;

比如我们有一个模块bar.js: 

- 该模块我们希望在代码运行过程中来加载它(比如判断一-个条件成立时加载) ;
- 因为我们并不确定这个模块中的代码一定会用到，所以最好拆分成一个独立的js文件;
- 这样可以保证不用到该内容时，浏览器不需要加载和处理该文件的js代码;
- 这个时候我们就可以使用动态导入;
- 这里实现了一个懒加载

![image-20230112171817595](http://img.roydust.top/img/202301121718670.png)

注意:使用动态导入barjs: .

- 在webpack中，通过动态导入获取到一个对象;
- 真正导出的内容，在该对象的default属性中，所以我们需要做一个简单的解构;

 ```js
 function about() {
   console.log("about function exec~");
 }
 const name = "默认导出值"
 
 export {
   about
 }
 
 export default name
 ```

```JS
btn1.onclick = function () {
  import("./router/about").then(res => {
    res.about()    
    res.default()
  })
} 
```



#### 分包重命名

动态导入的文件命名:

- 因为动态导入通常是一定会打包成独立的文件的，所以并不会在cacheGroups中进行配置;
- 那么它的命名我们通常会在output中,通过chunkFilename属性来命名;

```js
  output: {
    clean: true,
    path: path.resolve(__dirname, "./build"),
    // [name]为占位符
    filename: "[name]-bundle.js",
    // 单独针对分包的文件进行命名
    chunkFilename: "[name]_chunk.js"
  },
```

但是，你会发现默认情况下我们获取到的[name]是和id的名称保持-致的

- 如果我们希望修改name的值，可以通过magic comments (魔法注释)的方式;

```js
btn1.onclick = function () {
  import(/* webpackChunkName:"about" */"./router/about").then(res => {
    res.about()
    res.default()
  })
}
btn2.onclick = function () {
  import(/* webpackChunkName:"category" */"./router/category")
}
```



### SplitChunkPlugin

另外一种分包的模式是**splitChunk**, 它底层是使用**SplitChunksPlugin**来实现的:

- 因为该**插件webpack已经默认安装和集成**，所以我们并不需要单独安装和直接使用该插件;
- 只需要**提供SplitChunksPlugin相关的配置信息**即可;

```js
  // 优化配置
  optimization: {
    splitChunks: {
      // 所有导入的第三包都会分开
      chunks: "all"
    }
  },
```

#### 自定义配置解析

Chunks:

- 默认值是async
- all表示对同步和异步代码都进行处理

minSize:

- 拆分包的大小，少为minSize;
- 如果一个包拆分出来达不到minSize,那么这个包就不会拆分; 

maxSize:

- 将大于maxSize的包，拆分为不小于minSize的包;

cacheGroups:

- 用于对拆分的包就行分组，比如一个lodash在拆分之后，并不会立即打包，而是会等到有没有其他符合规则的包一起来打包;
- test属性:匹配符合规则的包; 
- name属性:拆分包的name属性; 
- filename属性:拆分包的名称，可以自己使用placeholder属性;

```json
  // 优化配置
  optimization: {
    splitChunks: {
      chunks: "all",
      // 当一个包大于指定值时，继续进行拆包
      maxSize: 20000,
      // 将包拆分成不小于minSize的包
      minSize: 10000,

      // 自己对需要拆包的内容进行分包
      cacheGroups: {
        vendors: {
          // 在window上 /和\都有可能是斜杠 mac上只有/
          test: /[\\/]node_modules[\\/]/, //完整写法
          filename: "[id]_vendors.js"
        },
        utils: {
          test: /utils/,
          filename: "[id]_utils.js"
        }
      }
    }
  },
```



#### optimization.chunklds配置

optimization.chunklds配置用于告知webpack模块的id采用什么算法生成。
有三个比较常见的值:

- natural:按照数字的顺序使用id;
-  named: development 下的默认值，一个可读的名称的id;
-  deterministic:确定性的，在不同的编译中不变的短数字id (有利于浏览器缓存)
  - 在webpack4中是没有这个值的; 
  - 那个时候如果使用natural,那么在一些编译发生变化时， 就会有问题;

**最佳实践:**

- 开发过程中，我们推荐使用named;
- 打包过程中，我们推荐使用deterministic;

```json
// 优化配置
  optimization: {
    // 设置生成的chunkId的算法
    // development默认:named
    //  production：deterministic(确定性的数字) 
    // webpack4中使用：natural 自然数递增
    chunkIds: "deterministic",

    // 分包插件：splitChunksPlugin
    splitChunks: {
      chunks: "all",
      // 当一个包大于指定值时，继续进行拆包
      // maxSize: 20000,
      // 将包拆分成不小于minSize的包
      // minSize: 10000,
      minSize: 10,
      // 自己对需要拆包的内容进行分包
      cacheGroups: {
        vendors: {
          // 在window上 /和\都有可能是斜杠 mac上只有/
          test: /[\\/]node_modules[\\/]/, //完整写法
          filename: "[id]_vendors.js"
        },
        utils: {
          test: /utils/,
          filename: "[id]_utils.js"
        }
      }
    },
    // 代码优化：TerserPlugin => 让代码更简单  => Terser
    minimize：true,
    minimizer: [
      // JS代码简化 去掉注释
      new TerserPlugin({
        extractComments: false
      })
    ]
  },
```





### prefetch和preload

webpack v4.6.0+增加了对预获取和预加载的支持。

**在声明import时，使用下面这些内置指令，来告知浏览器:**

- **prefetch**(预获取):将来某些导航下可能需要的资源
- **preload**(预加载):当前导航下可能需要资源 

与prefetch指令相比，preload 指令有许多不同之处:

- preload chunk会在父chunk加载时，以并行方式开始加载。prefetch chunk会在父chunk加载结束后开始加载。
- preload chunk具有中等优先级，并立即下载。prefetch chunk在浏览器闲置时下载。
- preload chunk会在父chunk中立即请求，用于当下时刻。prefetch chunk会用于未来的某个时刻。



使用魔法注释 `/* webpackPrefetch:true */` 就可以实现分包预加载

```js
btn1.onclick = function () {
  import(
    /* webpackChunkName:"about" */
    /* webpackPrefetch:true */
    "./router/about").then(res => {
      res.about()
      res.default()
    })
}
btn2.onclick = function () {
  import(
    /* webpackChunkName:"category" */
    /* webpackPrefetch:true */
    "./router/category")
}
```

![image-20230113150112037](http://img.roydust.top/img/202301131501147.png)



### CDN加速服务器配置

CDN称之为内容分发网络(Content Delivery Network或Content Distribution Network,缩写: CDN)

- 它是指通过相互连接的网络系统，利用最靠近每个用户的服务器;
- 更快、更可靠地将音乐、图片、视频、应用程序及其他文件发送给用户;
- 来提供高性能、可扩展性及低成本的网络内容传递给用户;

![image-20230113150153669](http://img.roydust.top/img/202301131501747.png)

在开发中，我们使用CDN主要是两种方式:

- 方式一:打包的所有静态资源，放到CDN服务器

  用户所有资源都是通过CDN服务器加载的;

- 方式二:一些第三方资源放到CDN服务器上;

  

#### 所有的静态资源都想要放到CDN服务器上

如果所有的静态资源都想要放到CDN服务器上，我们需要购买自己的CDN服务器;

- 目前阿里、腾讯、亚马逊、Google等都可以购买CDN服务器;
- 我们可以直接修改publicPath,在打包时添加上自己的CDN地址;

```json
  output: {
    clean: true,
    path: path.resolve(__dirname, "./build"),
    // [name]为占位符
    filename: "[name]-bundle.js",
    // 单独针对分包的文件进行命名
    chunkFilename: "[name]_chunk.js",
    // CND服务器地址
    publicPath: "http://coderwhycnd.com/"
  },
```

此时Html加载的包都是从CDN处加载

```html
    <script defer src="http://coderwhycnd.com/246_vendors.js"></script>
    <script defer src="http://coderwhycnd.com/497_utils.js"></script>
    <script defer src="http://coderwhycnd.com/main-bundle.js"></script>
```



#### 第三方库的CND

通常一些比较出名的开源框架都会将打包后的源码放到一-些比较出名的、免费的CDN服务器上:

- 国际上使用比较多的是unpkg、JSDelivr、 cdnjs; .
- 国内也有一个比较好用的CDN是bootcdn; 

在项目中，我们如何去引入这些CDN呢?

- 第一, 在打包的时候我们不再需要对类似于lodash或者dayjs这些库进行打包; 
- 第二,在html模块中，我们需要自己加入对应的CDN服务器地址;

**第一步,我们可以通过webpack配置,来排除一些库的打包:** 

```js
// 排除某些包不需要进行打包
  externals: {
    react: "React",
    // key属性名：排除的框架名称 
    // value值：从CDN地址请求下来的js中提供对应的名称
    axios: "axios"
  },
```



**第二步，在html模块中，加入CDN服务器地址:**

```html
  <body>
    <div id="root"></div>
    <script src="https://cdn.bootcdn.net/ajax/libs/axios/1.2.2/axios.min.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
  </body> 
```



### 认识shimming 

shimming是一个概念，是某一类功能的统称:

- shimming翻译过来我们称之为垫片,相当于给我们的代码填充一-些垫片来处理一-些问题;
- 比如我们现在依赖一个第三方的库,这个第三方的库本身依赖lodash,但是默认没有对lodash进行导入(认为全局存在lodash),那么我们就可以通过ProvidePlugin来实现shimming的效果;

**注意: webpack并不推荐随意的使用shimming**

- Webpack背后的整个理念是使前端开发更加模块化;
- 也就是说，需要编写具有封闭性的、不存在隐含依赖(比如全局变量)的彼此隔离的模块;

假如我们的lodash、dayjs都使用了CDN进行引入,所以相当于在全局是可以使用和dayjs的

- 假如一个文件中我们使用了axios,但是没有对它进行引入，那么下面的代码是会报错的; 

```js
// import axios from "axios";

// import dayjs from "dayjs"

axios.get("http://123.207.32.32:8000/home/multidata").then(res => {
  console.log(res);
})

console.log(dayjs(new Date()).format("YYYY-MM-DD HH:mm:ss"));

/*
如果abc.js是一个第三方库，但是他没有引用axios，但是却使用了axios的方法,
此时如果本地没有引用axios这个第三方库就会报错，所以需要shimming
*/
```

**我们可以通过使用ProvidePlugin来实现shimming的效果:**

- **ProvidePlugin能够帮助我们在每个模块中，通过一个变量来获取一-个package**;
- 如果webpack看到这个模块，它将在最终的bundle中引入这个模块;
- 另外ProvidePlugin是**webpack默认的一个插件**,所以不需要专门导入;

这段代码的本质是告诉webpack:

如果你遇到了至少一处用到axios变量的模块实例，那请你将axios package引入进来,并将其提供给需要用到它的模块。

```json
const { ProvidePlugin } = require("webpack") 
	plugins: [
    new ProvidePlugin({
      axios: ["axios", "default"],
      dayjs: "dayjs"
    })
  ]
```



### CSS样式的单独提取

**MiniCssExtractPlugin可以帮助我们将css提取到一个独立的css文件中,该插件需要在webpack4+才可以使用。**

**首先，我们需要安装mini-css-extract-plugin:** 

```
npm install mini-css-extract-plugin -D
```

**配置rules和plugins:**

```json
	module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // "style-loader",  开发阶段 直接插入到html文件
          MiniCssExtractPlugin.loader,  // 生产阶段 分离单独的css文件
          'css-loader'
        ]
      }
    ]
  },  
	plugins: [
    // 完成css的提取
    new MiniCssExtractPlugin({
      // 直接导入重命名
      filename: 'css/[name].css',
      // 动态导入重命名
      chunkFilename: 'css/[name]_chunk.css'

    })
  ]
```





### 打包重命名的Hash生成

**在我们给打包的文件进行命名的时候，会使用placeholder, placeholder中有几个属性比较相似:**

- hash、 chunkhash、 contenthash
-  hash本身是通过MD4的散列函数处理后，生成一个128位的hash值(32个十六进制) ; 

**hash值的生成和整个项目有关系:**

- 比如我们现在有两个入口index.js和mainjs;
- 它们分别会输出到不同的bundle文件中，粗在文件名称中我们有使用hash;
- 这个时候，如果修改了index.js文件中的内容，那么hash会发生变化;
- 那就意味着两个文件的名称都会发生变化; 

**chunkhash可以有效的解决上面的问题，它会根据不同的入口进行解析来生成hash值:**

- 比如我们修改了index:js,那么main.js的chunkhash是不会发生改变的;

**contenthash表示生成的文件hash名称，只和内容有关系:**

- 比如我们的index.js,引入了一个style.css, style.css有 被抽取到一个独立的css文件中; .
- 这个css文件在命名时，如果我们使用的是chunkhash;
- 那么当index.js文件的内容发生变化时, css文件的命名也会发生变化;
- 这个时候我们可以使用**contenthash**; 

```json
const path = require("path")
const MiniCssExtractPlugin = require("mini-css-extract-plugin")

module.exports = {
  mode: "development",
  entry: {
    main: "./src/main.js",
    index: "./src/index.js"
  },
  output: {
    clean: true,
    path: path.resolve(__dirname, './build'),
    filename: "[name]_[contenthash]_bundle.js",
    chunkFilename: "[contenthash]_chunk.js"
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: "[contenthash]_[name].css"
    })
  ]
}
```



### TerserPlugin代码压缩

什么是Terser呢?

- Terser是一个JavaScript的解释(Parser) 、Mangler (绞肉机) /Compressor (压缩机)的工具集;
- 早期我们会使用uglify-js来压缩、丑化我们的JavaScript代码，但是目前已经不再维护，并且不支持ES6+的语法;
- Terser是从uglify-es fork过来的,并且保留它原来的大部分API以及适配uglify-es和uglify-js@3等;

**也就是说，Terser可以帮助我们压缩、丑化我们的代码，让我们的bundle变得更小。**

**因为Terser是一个独立的工具，所以它可以单独安装:**

#全局安装

```
npm install terser -g
```

#局部安装

```
npm install terser -D
```



#### 命令行使用

我们可以在命令行中使用Terser:

```
terser [input files] [options]
```

#举例说明

```
terser js/file1.js -o foo.min.js -C -m
```

我们这里来讲解几个Compress option和Mangle(乱砍) option:

- 因为他们的配置非常多，我们不可能一个个解析,更多的查看文档即可;
- https://github.com/terser/terser#compress-options
- https://github.com/terser/terser#mangle-options



#### Compress option:

- **arrows**: class或者object中的函数, 转换成箭头函数;
- **arguments**:将函数中使用arguments[index]转成对应的形参名称;
- **dead_ code**:移除不可达的代码(tree shaking) ;
-  他属性可以查看文档;

#### Mangle option

- toplevel:默认值是false,顶层作用域中的变量名称，进行丑化(转换) ;
- keep_classnames: 默认值是false,是否保持依赖的类名称; 
-  keep_fnames:默认值是false,是否保持原来的函数名称;
- 其他属性可以查看文档;

```
npx terser abx.js -o abx.min.js -c arrows=true arguments=true,dead_code=true -m toplevel=true,keep_fnames=true
```

![image-20230113230309484](http://img.roydust.top/img/202301132303676.png)



#### Terser在webpack中配置

真实开发中，我们不需要手动的通过terser来处理我们的代码，我们可以直接通过webpack来处理:

- 在webpack中有一 个minimizer属性 ,在production模式下， 默认就是使用TerserPlugin来处理我们的代码的;
- 如果我们对默认的配置不满意，也可以自己来创建TerserPlugin的实例，組覆盖相关的配置;

首先,我们需要打开minimize,让其对我们的代码进行压缩(默认production模式 下已经打开了)

其次，我们可以在minimizer创建-个TerserPlugin: 

- **extractComments**: 默认值为true, 表示会将注释抽取到一个单独的文件中; .
  - 在开发中，我们不希望保留这个注释时，可以设置为false
- **parallel**: 使多进程并发运行提高构建的速度，默认值是true
  - 并发运行的默认数量: os.cpus().length - 1;
  - 我们也可以设置自己的个数，但是使用默认值即可; 
- **terserOptions**: 设置我们的terser相关的配置
  - compress: 设置压缩相关的选项;
  -  mangle: 设置丑化相关的选项，可以直接设置为true;
  -  toplevel: 顶层变量是否进行转换;
  -  keep_ classnames: 保留类的名称;
  -  keep_ fnames:保留函数的名称;

```json
  // 优化配置
  optimization: {
		minimize: true,
    // 代码优化：TerserPlugin => 让代码更简单  => Terser
    minimizer: [
      // JS压缩插件 TerserPlugin
      new TerserPlugin({
        extractComments: false,
        terserOptions: {
          compress: {
            // 改变arguments参数名称
            arguments: true,
            // 启用 没有使用过的代码
            unused: false
          },
          // 丑化代码
          mangle: true,
          // 顶层作用域中的变量名称
          // toplevel: false
          // 保持原来的函数名称
          keep_fnames: true
        }
      })
      // CSS压缩插件 CSSMinimizerPlugin
    ]
  }, 
```



#### CSS压缩

另一个代码的压缩是CSS:

- CSS压缩通常是去除无用的空格等，因为很难去修改选择器、属性的名称、值等;
- CSS的压缩我们可以使用另外一个插件: css-minimizer-webpack plugin;

- css-minimizer-webpack- plugin是使用cssnano工具来优化、压缩CSS (也可以单独使用) ;

第一步,安装css-minimizer-webpack-plugin:

```
npm install css-minimizer-webpack-plugin 
```

第二步,在optimization.minimizer中配置

```json
minimizer: [
      // JS压缩插件 TerserPlugin
      new TerserPlugin({
        extractComments: false,
        terserOptions: {
          compress: {
            // 改变arguments参数名称
            arguments: true,
            // 启用 没有使用过的代码
            unused: false
          },
          // 丑化代码
          mangle: true,
          // 顶层作用域中的变量名称
          // toplevel: false
          // 保持原来的函数名称
          keep_fnames: true
        }
      }),
      // CSS压缩插件 CSSMinimizerPlugin
      new CssMinimizerPlugin({
        // parallel: true,  
      })
    ]
```







### Tree Shaking的实现

**将mode设置为development模式:**

- 为了可以看到usedExports带来的效果，我们需要设置为development模式
- 因为在production模式下，webpack默认的一些优化会带来很大的影响。

```json
  optimization: {
    // 导入模块时，分析模块中那些函数有被使用，那些函数没有被使用
    usedExports: true,
  }
```



**设置usedExports为true和false对比打包后的代码:**

- 在usedExports设置为true时，会有一段注释: unused harmony export mul;
- 这段注释的意义是什么呢?告知Terser在优化时，可以删除掉这段代码;

```json
/* harmony export */ __webpack_require__.d(__webpack_exports__, {
/* harmony export */   "sum": function() { return /* binding */ sum; }
/* harmony export */ });
/* unused harmony export mul */
function sum(num1, num2) {
  return num1 + num2;
}
function mul(num1, num2) {
  return num1 + num2;
}
```



**这个时候，我们讲minimize设置true:**

- usedExports设置为false时，mul函数没有被移除掉; .
-  usedExports设置为true时，mul函数有被移除掉;



#### Tree Shaking要注意的sideEffects(副作用)

sideEffects用于告知webpack compiler哪些模块时有副作用的:

- 副作用的意思是这里面的代码有执行一些特殊的任务 ，不能仅仅通过export来判断这段代码的意义;
- 副作用的问题，在讲React的纯函数时是有讲过的;

```js
// 副作用代码
// 推荐：在平时写模块时，尽量编写纯模块
window.lyric = "哈哈哈哈哈哈哈"
```



在package.json中设置sideEffects的值:

- 如果我们将sideEffects设置为false,就是告知webpack可以安全的删除未用到的exports;
- 如果有一些我们希望保留，可以设置为数组;

```js
  "sideEffects": [
    "./src/demo/parse-lyric.js"
  ],
```



比如我们有一个format.js、 style.css文件:

- 该文件在导入时没有使用任何的变量来接受;
- 那么打包后的文件,不会保留format.js、 style.css相关的任何代码;

```json
  "sideEffects": [
    "*.css"
  ],
```



**所以，如何在项目中对JavaScript的代码进行TreeShaking呢(生成环境) ?**

- 在optimization中配置usedExports为true,来帮助Terser进行优化;
- 在package.json中配置sideEffects,直接对模块进行优化;



#### CSS的TreeShaking

**上面我们学习的都是关于JavaScript的Tree Shaking,那么CSS是否也可以进行Tree Shaking操作呢?**

- CSS的Tree Shaking需要借助于一些其他的插件;
- 在早期的时候，我们会使用PurifyCss插件来完成CSS的tree shaking,但是目前该库已经不再维护了(最新更新也是在4年前了) ;
- 目前我们可以使用另外-一个库来完成CSS的Tree Shaking: PurgeCSS, 也是一个帮助我们删除未使用的CSS的工具;



**配置这个插件(生成环境) :**

- paths: 表示要检测哪些目录下的内容需要被分析，这里我们可以使用glob;
- 默认情况下，Purgecss会将我们的html标签的样式移除掉,如果我们希望保留，可以添加一个safelist的属性;

![image-20230114204218918](http://img.roydust.top/img/202301142042017.png)

**purgecss也可以对less文件进行处理(所以它是对打包后的css进行tree shaking操作) ;**





### Scope Hoisting作用

什么是Scope Hoisting呢?

- Scope Hoisting从webpack3开始增加的一个新功能;
- 功能是对作用域进行提升,并且让webpack打包后的代码更小、运行更快;

默认情况下webpack打包会有很多的函数作用域，包括一些(比如最外层的) IIFE:

- 无论是从最开始的代码运行，还是加载一个模块， 都需要执行一系列的函数;
- Scope Hoisting可以将函数合并到一一个模块中来运行;

使用Scope Hoisting非常的简单, webpack已经内置了对应的模块:

- 在production模式下，默认这个模块就会启用;
- 在development模式下，我们需要自己来打开该模块;

```json
  // webpack插件
  plugins: [
    // 作用域提升
    new webpack.optimize.ModuleConcatenationPlugin()
  ]
```





### HTTP文件压缩传输

HTTP压缩是一种内置在服务器和客户端之间的，以改进传输速度和带宽利用率的方式;

HTTP压缩的流程什么呢?

- 第一步: HTTP数据在服务器发送前就已经被压缩了; (可以在webpack中完成)

- 第二步:兼容的浏览器在向服务器发送请求时，会告知服务器自己支持哪些压缩格式; 

  ![image-20230114210421623](http://img.roydust.top/img/202301142104670.png)

- 第三步:服务器在浏览器支持的压缩格式下，直接返回对应的压缩后的文件，并且在响应头中告知浏览器; 

  ![image-20230114210440932](http://img.roydust.top/img/202301142104977.png)

目前的压缩格式非常的多:

- compress - UNIX的"compress" 程序的方法(历史性原因，不推荐大多数应用使用，应该使用gzip或deflate) ;
-  deflate - 基于deflate算法(定义于RFC 1951)的压缩，使用zlib数据格式封装;
-  gzip - GNU zip格式(定义于RFC 1952)，是目前使用比较广泛的压缩算法;
-  br - 一种新的开源压缩算法，专为HTTP内容的编码而设计;



#### webpack对文件的压缩

webpack中相当于是实现了HTTP压缩的第一步操作， 我们可以使用CompressionPlugin.

- 第一步,安装CompressionPlugin:

  ```
  npm install compression-webpack-plugin -D
  ```

- 第二步，使用CompressionPlugin即可:

![image-20230114211424049](http://img.roydust.top/img/202301142114110.png)

```json
  plugins: [
    // HTTP压缩
    new CompressionPlugin({
      test: /\.(css|js)/,
      minRatio: 0.7,
      algorithm: "gzip"
    })
  ]
```





### HTML文件的压缩

我们之前使用了HtmlWebpackPlugin插件来生成HTML的模板，事实上它还有一些其他的配置:

**inject:设置打包的资源插入的位置**

- true、false 、body、head

cache:设置为true,只有当文件改变时，才会生成新的文件(默认值也是true)

minify: 默认会使用一个插件html-minifier-terser

```json
plugins: [
      new HtmlWebpackPlugin({
        template: "./index.html",
        // 缓存，如果有更新才生成新html
        cache: true,
        minify: isProduction ? {
          // 移除注释
          removeComments: true,
          // 移除属性
          removeEmptyAttributes: true,
          // 移除默认属性
          removeRedundantAttributes: true,
          // 折叠空白字符
          collapseWhitespace: true,
          // 压缩内联css
          minifyCSS: true,
          // 压缩JavaScript
          minifyJS: {
            mangle: {
              toplevel: true
            }
          }
        } : false
      })
    ]
```





### webpack打包分析

#### 对打包时间分析

如果我们希望看到每一个loader. 每一个Plugin消耗的打包时间， 可以借助于一个插件: speed-measure-webpack-plugin

第一步,安装speed-measure-webpack plugin插件

```
npm install speed-measure-webpack-plugin -D
```

第二步,使用speed-measure-webpack-plugin插件

- 创建插件导出的对象SpeedMeasurePlugin;
- 使用smp.wrap包裹我们导出的webpack配置; 

```json
	const smp = new SpeedMeasurePlugin()


  const finalConfig = merge(mergeConfig, getCommonConfig(isProduction))
  return smp.wrap(finalConfig)
```



### 对打包文件分析



#### 方案一：生成一个stats.json的文件

```json
    "build": "webpack --config ./config/comm.config.js --env production --profile --json=stats.json",
```

通过执行npm run build:status可以获取到一 个stats.json的文件:

- 这个文件我们自己分析不容易看到其中的信息;

  ![image-20230114230903945](http://img.roydust.top/img/202301142309094.png)

- 可以放到https://webpack.github.io/analyse/进行分析

  ![image-20230114231135143](http://img.roydust.top/img/202301142311288.png)





#### 方案二:使用webpack bundle-analyzer工具

- 另一个非常直观查看包大小的工具是webpack- bundle-analyzer

第一步：安装

```
pnpm add webpack-bundle-analyzer -D
```

第二步：我们可以在webpack配置中使用该插件:

```json
  plugins: [
    // 对打包结果分析
    new BundleAnalyzerPlugin()
  ]
```

在打包webpack的时候，这个工具是帮助我们打开- -个8888端口上的服务,我们可以直接的看到每个包的大小。

- 比如有一个包时通过一个Vue组件打包的，但是非常的大，那么我们可以考虑是否可以拆分出多个组件，并且对其进行懒加载;
- 比如一-个图片或者字体文件特别大，是否可以对其进行压缩或者其他的优化处理;

![image-20230114232404039](http://img.roydust.top/img/202301142324579.png)









## Webpack配置分离

1. 在package.json中添加运行参数

   ```json
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1",
       "build": "webpack --config ./config/comm.config.js --env production",
       "serve": "webpack serve --config ./config/comm.config.js --env development",
       "ts-check": "tsc --noEmit",
       "ts-check-watch": "tsc --noEmit --watch"
     },
   ```

   

2. 将配置文件导出的是一个函数，而不是个对象

   ```js
   const getCommonConfig = function (isProduction) {
     return {
       ...
     }
   }
     
   // webpack 允许我们导出一个函数
   module.exports = function (env) {
     const isProduction = env.production;
     let mergeConfig = isProduction ? prodConfig : devConfig
     // 将comm配置文件变成一个回调函数，通过出传入参数isProduction来判断环境，改变部分内容
     return merge(mergeConfig, getCommonConfig(isProduction))
   }
   ```

3. 从上向下查看所有的配置属性应该属于哪一 个文件

   - comm/dev/prod

   > comm(基础配置)

   ```json
   const path = require("path")
   const HtmlWebpackPlugin = require("html-webpack-plugin")
   const { ProvidePlugin } = require("webpack")
   const MiniCssExtractPlugin = require("mini-css-extract-plugin")
   const { merge } = require('webpack-merge')
   const devConfig = require("./dev.config")
   const prodConfig = require("./prod.config")
   
   const getCommonConfig = function (isProduction) {
     return {
       devtool: false,
       entry: "./src/main.js",
       output: {
         clean: true,
         path: path.resolve(__dirname, "../build"),
         // [name]为占位符
         filename: "js/[name]-bundle.js",
         // 单独针对分包的文件进行命名
         chunkFilename: "js/[name]_chunk.js",
         // CND服务器地址
         // publicPath: "http://coderwhycnd.com/"
       },
       // 排除某些包不需要进行打包 
       // externals: {
       //   react: "React",
       //   // key属性名：排除的框架名称 
       //   // value值：从CDN地址请求下来的js中提供对应的名称
       //   axios: "axios"
       // },
       resolve: {
         extensions: ['.js', '.json', '.wasm', '.jsx', '.ts']
       },
   
       module: {
         rules: [
           {
             test: /\.jsx?$/, // x?:标识0或者1
             use: {
               loader: "babel-loader"
             }
           },
           {
             test: /\.ts$/,
             use: "babel-loader"
           },
           {
             test: /\.css$/,
             use: [
               // "style-loader",  // 开发阶段 直接插入到html文件
               // MiniCssExtractPlugin.loader,  // 生产阶段 分离单独的css文件
               // 动态根据环境来更改css loader
               isProduction ? MiniCssExtractPlugin.loader : "style-loader",
               'css-loader'
             ]
           }
         ]
       },
       // webpack插件
       plugins: [
         new HtmlWebpackPlugin({
           template: "./index.html"
         }),
         new ProvidePlugin({
           axios: ["axios", "default"],
           // get: ['axios', 'get'],
           dayjs: "dayjs"
         }),
       ]
     }
   }
   
   // webpack 允许我们导出一个函数
   module.exports = function (env) {
     const isProduction = env.production;
     let mergeConfig = isProduction ? prodConfig : devConfig
     // 将comm配置文件变成一个回调函数，通过出传入参数isProduction来判断环境，改变部分内容
     return merge(mergeConfig, getCommonConfig(isProduction))
   }
   ```
   
   > dev(开发环境配置)
   
   ```json
   module.exports = {
     mode: "development",
     devServer: {
       static: ['public', 'content'],
       port: 8080,
       compress: true,
       proxy: {
         '/api': {
           target: "http://localhost:9000",
           pathRewrite: {
             "^/api": ""
           },
           changeOrigin: true
         }
       },
       historyApiFallback: true
     },
   
     // webpack插件
     plugins: [
     ]
   }
   ```
   
   > prod(生产环境配置)
   
   ```json
   const TerserPlugin = require("terser-webpack-plugin")
   const MiniCssExtractPlugin = require("mini-css-extract-plugin")
   const CssMinimizerPlugin = require("css-minimizer-webpack-plugin")
   
   module.exports = {
     mode: "production",
     // 优化配置
     optimization: {
       chunkIds: "deterministic",
   
       // 分包插件：splitChunksPlugin
       splitChunks: {
         chunks: "all",
         minSize: 10,
         // 自己对需要拆包的内容进行分包
         cacheGroups: {
           vendors: {
             test: /[\\/]node_modules[\\/]/, //完整写法
             filename: "js/[id]_vendors.js"
           },
           utils: {
             test: /utils/,
             filename: "js/[id]_utils.js"
           }
         }
       },
       minimize: true,
       // 代码优化：TerserPlugin => 让代码更简单  => Terser
       minimizer: [
         // JS压缩插件 TerserPlugin
         new TerserPlugin({
           extractComments: false,
           terserOptions: {
             compress: {
               // 改变arguments参数名称
               arguments: true,
               // 启用 没有使用过的代码
               unused: false
             },
             // 丑化代码
             mangle: true,
             // 顶层作用域中的变量名称
             // toplevel: false
             // 保持原来的函数名称
             keep_fnames: true
           }
         }),
         // CSS压缩插件 CSSMinimizerPlugin
         new CssMinimizerPlugin({
           // parallel: true,  
         })
       ]
     },
   
     // webpack插件
     plugins: [
       // 完成css的提取
       new MiniCssExtractPlugin({
         // 直接导入重命名
         filename: 'css/[name].css',
         // 动态导入重命名
         chunkFilename: 'css/[name]_chunk.css'
       })
     ]
   }
   ```
   
   

4. 针对单独的配置文件进行定义化

   - CSS加载: .使用的不同的loader可以根据isProduction动态获取
   
   ```json
   		module: {
         rules: [
           {
             test: /\.jsx?$/, // x?:标识0或者1
             use: {
               loader: "babel-loader"
             }
           },
           {
             test: /\.ts$/,
             use: "babel-loader"
           },
           {
             test: /\.css$/,
             use: [
               // "style-loader",  // 开发阶段 直接插入到html文件
               // MiniCssExtractPlugin.loader,  // 生产阶段 分离单独的css文件
               // 动态根据环境来更改css loader
               isProduction ? MiniCssExtractPlugin.loader : "style-loader",
               'css-loader'
             ]
           }
         ]
       },
   ```
   
   



## Webpack自定义Loader

### Webpack源码执行过程

![image-20230115161457472](http://img.roydust.top/img/202301151614664.png)



### 认识自定义Loader

**Loader是用于对模块的源代码进行转换(处理)，之前我们已经使用过很多Loader,比如css-loader. style-loader. babel-loader等。**

**这里我们来学习如何自定义自己的Loader:**

- Loader本质上是- -个导出为函数的JavaScript模块;
- loader runner库会调用这个函数，然后将上一-个loader产生的结果或者资源文件传入进去; 

**编写一个hy-loader01.js模块这个函数会接收三三个参数:**

- content: 资源文件的内容;
- map: sourcemap相关的数据;
- meta: 一些元数据; 

```js
module.exports = function (content, map, meta) {
  console.log(content);
  return content
} 
```



### Loader的加载顺序

**是一个类似栈一样的先入后出顺序**

![image-20230115221441772](http://img.roydust.top/img/202301152214876.png)





### 同步和异步Loader

#### 同步Loader

什么是同步的Loader呢?

- 默认创建的Loader就是同步的Loader; 
- 这个Loader必须通过return或者this.callback 来返回结果，交给下一个loader来处理;
- 通常在有错误的情况下，我们会使用this.callback;

this.callback的用法如下:

- 第一个参数必须是Error或者null;
- 第二个参数是一个string或者Buffer;

```js
module.exports = function (content) {
  // this 绑定对象
  const callback = this.callback

  // callback进行调用：
  // 参数一：错误信息
  // 参数二：传递给下一个loader的内容
  callback(null, "哈哈哈哈哈哈")
}
```



#### 异步Loader

什么是异步的Loader呢?

- 有时候我们使用Loader时会进行一些异步的操作;
- 我们希望在异步操作完成后，再返回这个loader处理的结果;
- 这个时候我们就要使用异步的Loader了;

**loader-runner已经在执行loader时给我们提供了方法，让loader变成一个异步的loader：**

```js
// 异步loader
module.exports = function (content) {
  // this 绑定对象
  // 获取到同步callback
  // const callback = this.callback

  // 获取异步callback
  const callback = this.async()

  // 进行耗时操作
  setTimeout(() => {
    console.log(content);
    callback(null, content + "aaa")
  }, 1000);
}
```



### 获取以及校验参数



在使用loader时，传入参数。

我们可以通过一个webpack官方提供的一个解析库loader-utils,安装对应的库。

- 最新的版本webpack也可以直接通过**this.getOptions()**来获取;

```js
module.exports = function (content, map, meta) {

  // 1.获取使用loader时，传递进来的参数
  // 方式一：早期时，需要单独使用loader-utils（webpack开发）的库来获取参数
  // 方式二：目前，已经可以直接通过this.getOptions()直接获取到参数

  const options = this.getOptions()
  console.log(options);

  console.log("yc-loader04" + content);
  return content
} 
```



#### 校验参数



我们可以通过一-个webpack官方提供的校验库schema-utils,安装对应的库:

```js
npm install schema-utils
```

再设置一下校验配置

```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "description": "请输入名称，并且是string类型"
    },
    "age": {
      "type": "number",
      "description": "请输入年龄，并且是number类型"
    }
  }
}

```

```js
const { validate } = require("schema-utils")
const loader04Schema = require("./schema/loader04_schema.json")

module.exports = function (content, map, meta) {

  // 1.获取使用loader时，传递进来的参数
  // 方式一：早期时，需要单独使用loader-utils（webpack开发）的库来获取参数
  // 方式二：目前，已经可以直接通过this.getOptions()直接获取到参数
  const options = this.getOptions()
  console.log(options);

  // 2.校验参数
  validate(loader04Schema, options)

  console.log("yc-loader04" + content);
  return content
} 
```





### Loader案例练习

 **ycmd-loader(自定义loader)**

解析md，并且在页面渲染

> ycmd-loader.js

```js
const { marked } = require("marked")
const hljs = require("highlight.js")


module.exports = function (content) {
  // 让marked库解析语法的时候能将代码高亮内容标识出来
  marked.setOptions({
    highlight: function (code, lang) {
      return hljs.highlight(lang, code).value
    }
  })

  // 将我们的md语法转化成html元素结构
  const htmlContent = marked(content)
  console.log(htmlContent);

  // 返回的结果必须是模块化内容
  const innerContent = "`" + htmlContent + "`";
  const moduleContent = `var code = ${innerContent}; export default code;`

  return moduleContent
}
```

> webpack.config.js

```js
const path = require('path')
const HtmlWebpackPlugin = require("html-webpack-plugin")

module.exports = {
  mode: "development",
  devtool: false,
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, './build'),
    filename: "bundle.js"
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          "./yc-loader/yc-loader01.js",
          // "./yc-loader/yc-loader02.js",
          // "./yc-loader/yc-loader03.js",

          // 给loader传递参数
          {
            loader: "./yc-loader/yc-loader04.js",
            options: {
              name: "lyc",
              age: 18
            }
          }
        ]
      },
      {
        test: /\.md$/,
        use: {
          loader: "./yc-loader/ycmd-loader.js",
        }
      },
      {
        test: /\.css$/,
        use: [
          "style-loader",
          "css-loader"
        ]
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin()
  ]
}
```

> main.js

```js
import code from "./md/learn.md"
import 'highlight.js/styles/default.css'
import "./css/code.css"

const message = "Hello World"
console.log(message);

// 1.对code进行打印
// console.log(code);

// 2.将它显示在页面中
document.body.innerHTML = code
```



## Webpack自定义Loader

### tapable的使用介绍

我们知道webpack有两个非常重要的类: Compiler和Compilation

- 他们通过注入插件的方式，来监听webpack的所有生命周期;
- 插件的注入离不开各种各样的Hook,而他们的Hook是如何得到的呢?
- 其实是创建了Tapable库中的各种Hook的实例;

所以，如果我们想要学习自定义插件，最好先了解一一个库: Tapable

- Tapable是官方编写和维护的一个库;
- Tapable是管理着需要的Hook,这些Hook可以被应用到我们的插件中;



### Tapable的分类

同步和异步的:

- 以sync开头的，是同步的Hook;
- 以async开头的，两个事件处理回调，不会等待上一次处理回调结束后再执行下一-次回调;

其他的类别

-  bail:当有返回值时，就不会执行后续的事件触发了;
-  Loop:当返回值为true,就会反复执行该事件，当返回值为undefined或者不返回内容,就退出事件;
-  Waterfall: 当返回值不为undefined时,的会将这次返回的结果作为下次事件的第一个参数;
- Parallel:并行，不会等待事件处理回调结束，才执行下一次事件处理回调;
- Series:串行，会等待上一是异步的Hook;

![image-20230116001802048](http://img.roydust.top/img/202301160018129.png)



### webpack与tapable

**webpack负责做第一和第三步，插件负责做第二步**

![image-20230116002910885](http://img.roydust.top/img/202301160029997.png)



### tapable的同步操作

- 基本使用

```js
const { SyncHook } = require("tapable")

class YCCompiler {
  constructor() {
    this.hooks = {
      // 1.创建hooks
      syncHook: new SyncHook(["name", "age"])
    }


    // 用hooks监听事件(自定义plugin)
    this.hooks.syncHook.tap("event1", (name, age) => {
      console.log("event1事件监听执行了:", name, age);
    })
    this.hooks.syncHook.tap("event2", (name, age) => {
      console.log("event2事件监听执行了:", name, age);
    })
  }
}

const compiler = new YCCompiler()
// 3.发出去事件
compiler.hooks.syncHook.call("lyc", 19)

event1事件监听执行了: lyc 19
event1事件监听执行了: lyc 19
```



-  bail:当有返回值时，就不会执行后续的事件触发了;

```js
const { SyncBailHook } = require("tapable")

class YCCompiler {
  constructor() {
    this.hooks = {
      // 1.创建hooks
      // bail的特点：如果有返回值，那么可以阻断后续事件继续执行
      SyncBailHook: new SyncBailHook(["name", "age"])
    }


    // 用hooks监听事件(自定义plugin)
    this.hooks.SyncBailHook.tap("event1", (name, age) => {
      console.log("event1事件监听执行了:", name, age);
      return 123
    })
    this.hooks.SyncBailHook.tap("event2", (name, age) => {
      console.log("event2事件监听执行了:", name, age);
    })
  }
}

const compiler = new YCCompiler()
// 3.发出去事件
setTimeout(() => {
  compiler.hooks.SyncBailHook.call("lyc", 19)
}, 1000);

event1事件监听执行了: lyc 19
```

-  Loop:当返回值为true,就会反复执行该事件，当返回值为undefined或者不返回内容,就退出事件;

```js
const { SyncLoopHook } = require("tapable")

let count = 0;
class YCCompiler {
  constructor() {
    this.hooks = {
      // 1.创建hooks
      // Loop:当返回值为true,就会反复执行该事件，当返回值为undefined或者不返回内容,就退出事件;
      SyncLoopHook: new SyncLoopHook(["name", "age"])
    }


    // 用hooks监听事件(自定义plugin)
    this.hooks.SyncLoopHook.tap("event1", (name, age) => {
      if (count < 5) {
        console.log("event1事件监听执行了:", name, age);
        count++;
        return true
      }
    })
    this.hooks.SyncLoopHook.tap("event2", (name, age) => {
      console.log("event2事件监听执行了:", name, age);
    })
  }
}

const compiler = new YCCompiler()
// 3.发出去事件
setTimeout(() => {
  compiler.hooks.SyncLoopHook.call("lyc", 19)
}, 1000);

event1事件监听执行了: lyc 19
event1事件监听执行了: lyc 19
event1事件监听执行了: lyc 19
event1事件监听执行了: lyc 19
event1事件监听执行了: lyc 19
event2事件监听执行了: lyc 19
```

-  Waterfall: 当返回值不为undefined时,的会将这次返回的结果作为下次事件的第一个参数;

```js
const { SyncWaterfallHook } = require("tapable")

class YCCompiler {
  constructor() {
    this.hooks = {
      // 1.创建hooks
      SyncWaterfallHook: new SyncWaterfallHook(["name", "age"])
    }


    // 用hooks监听事件(自定义plugin)
    this.hooks.SyncWaterfallHook.tap("event1", (name, age) => {
      console.log("event1事件监听执行了:", name, age);
      return "哈哈哈哈哈"
    })
    this.hooks.SyncWaterfallHook.tap("event2", (name, age) => {
      console.log("event2事件监听执行了:", name, age);
    })
  }
}

const compiler = new YCCompiler()
// 3.发出去事件
setTimeout(() => {
  compiler.hooks.SyncWaterfallHook.call("lyc", 19)
}, 1000);

event1事件监听执行了: lyc 19
event2事件监听执行了: 哈哈哈哈哈 19
```





### tapable的异步操作

- Parallel:并行，不会等待事件处理回调结束，才执行下一次事件处理回调;

```js
const { AsyncParallelHook } = require("tapable")

class YCCompiler {
  constructor() {
    this.hooks = {
      // 1.创建hooks
      AsyncParallelHook: new AsyncParallelHook(["name", "age"])
    }

    // 用hooks监听事件(自定义plugin)
    this.hooks.AsyncParallelHook.tapAsync("event1", (name, age) => {
      setTimeout(() => {
        console.log("event1事件监听执行了:", name, age);
      }, 2000);
    })
    this.hooks.AsyncParallelHook.tapAsync("event2", (name, age) => {
      setTimeout(() => {
        console.log("event2事件监听执行了:", name, age);
      }, 2000);
    })
  }
}

const compiler = new YCCompiler()
// 3.发出去事件
setTimeout(() => {
  compiler.hooks.AsyncParallelHook.callAsync("lyc", 19)
}, 1000);

同步出现
event1事件监听执行了: lyc 19
event2事件监听执行了: lyc 19
```



- Series:串行，会等待上一是异步的Hook;

```js
const { AsyncSeriesHook } = require("tapable")

class YCCompiler {
  constructor() {
    this.hooks = {
      // 1.创建hooks
      AsyncSeriesHook: new AsyncSeriesHook(["name", "age"])
    }

    // 用hooks监听事件(自定义plugin)
    this.hooks.AsyncSeriesHook.tapAsync("event1", (name, age, callback) => {
      setTimeout(() => {
        console.log("event1事件监听执行了:", name, age);
        callback()
      }, 2000);
    })
    this.hooks.AsyncSeriesHook.tapAsync("event2", (name, age, callback) => {
      setTimeout(() => {
        console.log("event2事件监听执行了:", name, age);
        callback()
      }, 2000);
    })
  }
}

const compiler = new YCCompiler()
// 3.发出去事件
setTimeout(() => {
  compiler.hooks.AsyncSeriesHook.callAsync("lyc", 19, () => {
    console.log("所有任务执行完~");
  })
}, 1000);

依次出现
event1事件监听执行了: lyc 19
event2事件监听执行了: lyc 19
所有任务执行完~
```



### 自定义Plugin的流程

在之前的学习中，我们已经使用了非常多的Plugin:

- CleanWebpackPlugin
- HTMLWebpackPlugin
- MiniCSSExtractPlugin .
- CompressionPlugin
- 等等。。。

这些Plugin是如何被注册到webpack的生命周期中的呢?

- 第一:在webpack函数的createCompiler方法中，注册了所有的插件;
- 第二:在注册插件时，会调用插件函数或者插件对象的apply方法;
- 第三:插件方法会接收compiler对象,我们可以通过compiler对象来注册Hook的事件;
- 第四:某些插件也会传入一个compilation的对象, 我们也可以监听compilation的Hook事件;



#### 核心源码图

![image-20230116211922245](http://img.roydust.top/img/202301162119353.png)



### 自定义Plugin的练习



#### 将静态文件自动上传服务器中

![image-20230116212452604](http://img.roydust.top/img/202301162124709.png)

自定义插件的过程:

- 创建AutoUploadWebpackPlugin类;
- 编写apply方法:
  - 通过ssh连接服务器;
  - 删除服务器原来的文件夹;
  - 上传文件夹中的内容;
- 在webpack的plugins中，使用AutoUploadWebpackPlugin类;

> webpack.config.js

```jsopn
  plugins: [
    new HtmlWebpackPlugin(),
    new AutoUploadWebpackPlugin({
      host: "117.50.190.217",
      username: "root",
      password: "xxx",
      remotePath: "/root/test"
    })
  ]
```

> AutoUploadWebpackPlugin.js

```js
const { NodeSSH } = require("node-ssh")

class AutoUploadWebpackPlugin {
  constructor(options) {
    this.ssh = new NodeSSH()
    this.options = options
  }

  apply(compiler) {
    // console.log("AutoUploadWebpackPlugin被注册", compiler);
    // 完成的事情：注册hooks监听事件
    // 等到assets已经输出到output目录上时，完成自动上传的功能
    compiler.hooks.afterEmit.tapAsync("AutoPlugin", async (compilation, callback) => {
      // 1.获取输出文件夹路径(其中资源)
      const outputPath = compilation.outputOptions.path
      console.log(outputPath);

      // 2.链接远程服务器 SSH
      await this.contentServe()

      // 3.删除原有的文件夹中内容
      const remotePath = this.options.remotePath;
      this.ssh.execCommand(`rm -rf ${remotePath}/*`)

      // 4.将文件夹中资源上传到服务器中
      await this.uploadFiles(outputPath, remotePath)

      // 5.关闭ssh连接
      this.ssh.dispose()

      // 完成所有操作后调用callback()
      callback()
    })
  }

  async contentServe() {
    await this.ssh.connect({
      host: this.options.host,
      username: this.options.username,
      password: this.options.password
    })
    console.log("服务器连接成功");
  }

  async uploadFiles(localPath, remotePath) {
    await this.ssh.putDirectory(localPath, remotePath)
  }
}

module.exports = AutoUploadWebpackPlugin
module.exports.AutoUploadWebpackPlugin = AutoUploadWebpackPlugin
```



## 自动化工具 Gulp

### gulp和webpack

什么是Gulp?

- A toolkit to automate & enhance your workflow;
- 一个工具包，可以帮你自动化和增加你的工作流;

![image-20230117010503615](http://img.roydust.top/img/202301170105697.png)

#### Gulp和Webpack区别

gulp的核心理念是**task runner**

- 可以定义自己的**一系列任务**，等待任务被执行;
- 基于**文件Stream**的构建流;
- 我们可以**使用gulp的插件体系**来完成某些任务; .

webpack的核心理念是**module bundler**

- webpack是**一个模块化的打包工具**;
- 可以使用各种各样的**loader**来加载不同的模块;
- 可以使用各种 各样的插件在**webpack打包的生命周期**完成其他的任务;



**gulp相对于webpack的优缺点:**

- gulp相对于webpack思想更加的**简单、易用**中更适合编写**一些自动化的任务**
- 但是目前对于大型项目(Vue、 React、 Angular) 并不会使用gulp来构建，比如默认gulp是不支持模块化的;



### gulp的基本使用

首先，我们需要安装gulp:

```
npm install gulp
```

其次，编写gulpfile.js文件, 在其中创建一个任务:

```js
// 编写简单任务
const foo = (cb) => {
  console.log("第一个gulp任务");
  cb()
}

// 导出任务
module.exports = {
  foo
}
```

最后，执行gulp命令: 

```
npx gulp foo
```



#### 创建gulp任务



**每个gulp任务都是一个异步的JavaScript函数:**

- 此函数可以接受一个callback作为参数,调用callback函数那么任务会结束;
- 或者是一个返回stream、promise、 event emitter. child process或observable类型的函数;

任务可以是public或者private类型的;

- **公开任务(Public tasks) **从gulpfile中被导出(export) , 可以通过gulp命令直接调用;
- **私有任务(Private tasks) **被设计为在内部使用，通常作为series() 或parallel() 组合的组成部分;



**npx gulp 会执行默认任务**

```js
// 默认任务
module.exports.default = (cb) => {
  console.log("default 默认任务");
}
```



### gulp的任务组合

通常一个函数中能完成的任务是有限的(放到-一个函数中也不方便代码的维护) ,所以我们会将任务进行组合。

gulp提供了两个强大的组合方法:

- series(): 串行任务组合;
- parallel():并行任务组合;

```js
const { series, parallel } = require("gulp")

const foo1 = (cb) => {
  setTimeout(() => {
    console.log("foo1 task exec~");
    cb()
  }, 1000);
}

const foo2 = (cb) => {
  setTimeout(() => {
    console.log("foo2 task exec~");
    cb()
  }, 2000);
}

const foo3 = (cb) => {
  setTimeout(() => {
    console.log("foo3 task exec~");
    cb()
  }, 3000);
}

// 串行
const seriesFoo = series(foo1, foo2, foo3)
// 并行
const parallelFoo = parallel(foo1, foo2, foo3)

module.exports = {
  foo1, foo2, foo3, seriesFoo,parallelFoo
}
```



### gulp的文件操作

gulp暴露了src() 和dest() 方法用于处理计算机上存放的文件。

- src() 接受参数，并从文件系统中读取文件然后生成一个Node流 (Stream) ，它将所有匹配的文件**读取到内存中并通过流(Stream)进行处理;**
- 由src()产生的流(stream) 应当从任务(task函数) 中返回并发出**异步完成的信号;**
- dest() 接受一个输出目录作为参数，并且它还会产生一个Node流(stream),通过**该流将内容输出到文件中;**

```js
const { src, dest } = require("gulp")


const copyFile = (cb) => {
  // 1.读取文件  // 2.写入文件
  return src("./src/**/*.js").pipe(dest("./dist"))

}

module.exports = {
  copyFile
}
```

流(stream)所提供的主要的API是.pipe()方法，pipe方法的原理是什么呢?

- pipe方法接受一个**转换流(Transform streams)**或**可写流(Writable streams) ;** 
- 那么转换流或者可写流，拿到数据之后可以**对数据进行处理**，再次**传递给下一个转换流或者可写流;**



#### glob文件匹配规则

src()方法接受-一个glob字符串或由多个glob字符串组成的数组作为参数，用于确定哪些文件需要被操作。

- glob或glob数组必须至少匹配到一个匹配项，否则src()将报错;

glob的匹配规则如下:

- (一个星号*):在一个字符串中，匹配任意数量的字符，包括零个匹配;

  ```js
  '*.js'
  ```

- (两个星号**):在多个字符串匹配中匹配任意数量的字符串，通常用在匹配目录下的文件;

  ```js
  ' scripts/**/*.js'
  ```

- (取反!):

  - 由于glob匹配时是按照每个glob在数组中的位置依次进行匹配操作的;
  - 所以glob数组中的取反(negative) glob 必须跟在一个非取反 (non-negative) 的glob后面; 
  - 第一个glob匹配到一组匹配项，然瓜后面的取反glob删除这些匹配项中的一部分;

  ```js
  [' script/**/*.js'，' !scripts/vendor/']
  ```

  

### gulp的案例演练

**自动读取js，将ES6转化为ES5，并且压缩代码**

![image-20230117014815153](http://img.roydust.top/img/202301170148226.png)

如果在这个过程中，我们希望对文件进行某些处理，可以使用社区给我们提供的插件。

- 比如我们希望ES6转换成ES5,那么可以使用babel插件;
- 如果我们希望对代码进行压缩和丑化,那么可以使用uglify或者terser插件;

```js
const { src, dest } = require("gulp")
const babel = require("gulp-babel")
const terser = require("gulp-terser")


const jsTask = () => {
  return src("./src/**/*.js")
    .pipe(babel({ presets: ["@babel/preset-env"] }))
    .pipe(terser({ mangle: { toplevel: true } }))
    .pipe(dest("./dist"))
}

module.exports = {
  jsTask
}
```



### gulp开发和构建

接下来，我们编写一个案例，通过gulp来开启本地服务和打包:

- 打包html文件; 
  - 使用gulp-htmlmin插件;
- 打包JavaScript文件;
  - 使用gulp-babel, gulp-terser插件;
- 打包less文件;
  - 使用gulp-less插件;
-  html资源注入
  - 使用gulp-inject插件;
- 开启本地服务器
  - 使用browser-sync插件;
- 创建打包任务
- 创建开发任务

```js
const { src, dest, parallel, series, watch } = require("gulp")
const htmlmin = require("gulp-htmlmin")
const babel = require("gulp-babel")
const terser = require("gulp-terser")
const less = require("gulp-less")
const inject = require("gulp-inject")

const browserSync = require("browser-sync")

// 对html进行打包
const htmlTask = () => {
  return src("./src/**/*.html")
    .pipe(htmlmin({ collapseWhitespace: true }))
    .pipe(dest("./dist"))
}

// 对js进行打包
const jsTask = () => {
  return src("./src/**/*.js")
    .pipe(babel({ presets: ["@babel/preset-env"] }))
    .pipe(terser({ toplevel: true }))
    .pipe(dest("./dist"))
}

// 对less进行打包
const lessTask = () => {
  return src("./src/**/*.less")
    .pipe(less())
    .pipe(dest("./dist"))
}

// 在html中注入js和css
const injectTask = () => {
  return src("./dist/**/*.html")
    .pipe(inject(src(["./dist/**/*.js", "./dist/**/*.css"]), { relative: true }))
    .pipe(dest("./dist"))
}

// 开启本地服务
const bs = browserSync.create()
const serve = () => {
  watch("./src/**", buildTask)
  bs.init({
    port: 9000,
    open: true,
    files: "./dist/*",
    server: {
      baseDir: "./dist"
    }
  })
}

// 创建项目构建任务 
const buildTask = series(parallel(htmlTask, jsTask, lessTask), injectTask)
const serveTask = series(buildTask, serve)
// webpack搭建本地 webpack-dev-serve


module.exports = {
  buildTask, serveTask
}
```



## 库打包工具Rollup

### rollup的基本使用

我们来看一下官方对rollup的定义:

- Rollup is a module bundler for JavaScript which compiles small pieces of code into something larger and more complex,such as a library or application.
- Rollup是一个**JavaScript的模块化打包工具**， 可以帮助**我们编译小的代码到一个大的、复杂的代码**中，比如**一个库或者一个应用程序;**

**我们会发现Rollup的定义、定位和webpack非常的相似:**

- Rollup也是**一个模块化的打包工具**，但是Rollup主要是针对ES Module进行打包的;
- 另外webpack通常可以**通过各种loader处理各种各样的文件**，以及处理它们的依赖关系;
-  rollup更多时候是**专注于处理JavaScript代码**的(当然也可以处理css、font、 vue等文件) ;
- 另外rollup的配置和理念**相对于webpack来说，更加的简洁和容易理解;**
- 在早期webpack不支持tree shaking时，rollup具备 更强的优势;

**目前webpack和rollup分别应用在什么场景呢?**

- 通常在**实际项目开发过程**中，我们都会使用webpack (比如react、 angular项目都是基于webpack的) ;
- 在**对库文件进行打包**时，我们通常会使用rollup (比如vue、react、 dayjs源码本身都是基于rollup的，Vite底层使用Rollup) ;



**安装使用**

我们可以先安装rollup:

```
npm install rollup -D
```

创建main.js文件,打包到bundle.js文件中: 

```
npx rollup ./lib/index.js -f umd --name lycUtils -o dist/bundle.js
```





**rollup对不同环境的支持**

![image-20230117205515446](http://img.roydust.top/img/202301172055630.png)



#### rollup配置

我们可以将配置信息写到配置文件中rollup.config.js文件

我们可以对文件进行分别打包，打包出更多的库文件(用户可以根据不同的需求来引入) :

```js
module.exports = {
  // 入口
  input: "./lib/index.js",
  // 出口
  output: [
    {
      format: "umd",
      name: "lycUtils",
      file: "./build/bundle.umd.js"
    },
    {
      format: "cjs",
      file: "./build/bundle.cjs.js"
    },
    {
      format: "iife",
      name: "lycUtils",
      file: "./build/bundle.iife.js"
    }
  ]
}
```



### rollup的常见插件

#### 解决commonjs和第三方库问题

安装解决commonjs的库:

```
npm install @rollup/ plugin-commonjs -D
```

安装解决node_ modules的库:

```
npm install @rollup/ plugin-node-resolve -D
```

打包和排除lodash

```js
// 默认lodash没有被打包是因为它使用commonjs,rollup默认情况下只对ES module
const commonjs = require("@rollup/plugin-commonjs")
const nodeResolve = require("@rollup/plugin-node-resolve")

module.exports = {
  // 入口
  input: "./lib/index.js",
  // 出口
  output: {
    format: "umd",
    name: "lycUtils",
    file: "./build/bundle.umd.js",
    globals: {
      lodash: "_"
    }
  },
  external: ["lodash"],
  plugins: [
    commonjs(),
    nodeResolve()
  ]
}
```



#### 对ES6代码转换ES5

安装解决babel的库:

```
npm install @rollup/plugin-babel -D
```

安装解决依赖的库:

```
npm install "@babel/core -D
npm install @babel/preset-env -D
```

```js
// 默认lodash没有被打包是因为它使用commonjs,rollup默认情况下只对ES module
const commonjs = require("@rollup/plugin-commonjs")
const nodeResolve = require("@rollup/plugin-node-resolve")
const { babel } = require("@rollup/plugin-babel")

module.exports = {
  // 入口
  input: "./lib/index.js",
  // 出口
  output: {
    format: "umd",
    name: "lycUtils",
    file: "./build/bundle.umd.js",
    globals: {
      lodash: "_"
    }
  },
  external: ["lodash"],
  plugins: [
    commonjs(),
    nodeResolve(),
    babel({
      babelHelpers: "bundled",
      exclude: /node-modules/
    })
  ]
}
```

```js
module.exports = {
  presets: [["@babel/preset-env"],]
}
```



#### 代码压缩

如果我们希望对代码进行压缩，可以使用@ rollup/ plugin-terser:

```
npm install @rollup/plugin-terser -D
```

配置terser:

```js
// 默认lodash没有被打包是因为它使用commonjs,rollup默认情况下只对ES module
const commonjs = require("@rollup/plugin-commonjs")
const nodeResolve = require("@rollup/plugin-node-resolve")
const { babel } = require("@rollup/plugin-babel")
const terser = require("@rollup/plugin-terser")

module.exports = {
  // 入口
  input: "./lib/index.js",
  // 出口
  output: {
    format: "umd",
    name: "lycUtils",
    file: "./build/bundle.umd.js",
    globals: {
      lodash: "_"
    }
  },
  external: ["lodash"],
  plugins: [
    commonjs(),
    nodeResolve(),
    babel({
      babelHelpers: "bundled",
      exclude: /node-modules/
    }),
    terser()
  ]
}
```





### rollup的css打包

如果我们希望对代码进行压缩，可以使用@rollup/plugin-terser:

```
npm insta1l @rollup/plugin-terser -D
```

配置terser:

```js
const commonjs = require("@rollup/plugin-commonjs")
const nodeResolve = require("@rollup/plugin-node-resolve")
const { babel } = require("@rollup/plugin-babel")
const terser = require("@rollup/plugin-terser")
const postcss = require("rollup-plugin-postcss")

module.exports = {
  // 入口
  input: "./src/index.js",
  // 出口
  output: {
    format: "umd",
    name: "lycUtils",
    file: "./build/bundle.umd.js",
    globals: {
      lodash: "_"
    }
  },
  plugins: [
    commonjs(),
    nodeResolve(),
    babel({
      babelHelpers: "bundled",
      exclude: /node-modules/
    }),
    terser(),
    postcss()
  ]
}
```



> postcss.config.js

```js
module.exports = {
  // 添加浏览器前缀
  plugins: [require("postcss-preset-env")]
}
```





### rollup的vue打包

处理vue文件我们需要使用rollup-plugin-vue插件: .

- 但是注意:默认情况下我们安装的是vue3.x的版本，所以我这里指定了一下rollup-plugin-vue的版本;

```
pnpm add @vue/compiler-sfc rollup-plugin-vue -D
```

使用vue的插件:

```js
  plugins: [
    commonjs(),
    nodeResolve(),
    babel({
      babelHelpers: "bundled",
      exclude: /node-modules/
    }),
    terser(),
    postcss(),
    vue()
  ]
```



在我们打包vue项目后，运行会报如下的错误:

![image-20230118160038264](http://img.roydust.top/img/202301181600376.png)

这是因为在我们打包的vue代码中，用到process.env.NODE_ ENV,所以我们可以使用一个插件rollup-plugin-replace设置它对应的值:

```
npm install @ro11up/plugin-replace -D
```

配置插件信息:

```js
  plugins: [
    commonjs(),
    nodeResolve(),
    babel({
      babelHelpers: "bundled",
      exclude: /node-modules/
    }),
    terser(),
    postcss(),
    vue(),
    replace({
      "process.env.NODE_ENV": `"production"`
    })
  ]
```







### rollup本地服务器

第一步:使用rollup-plugin-serve搭建服务

```
npm install rollup-plugin-serve -D
```

第二步:当文件发生变化时，自动刷新浏览器

```js
npm install rollup-plugin-livereload -D
```

```
  plugins: [
    commonjs(),
    nodeResolve(),
    babel({
      babelHelpers: "bundled",
      exclude: /node-modules/
    }),
    terser(),
    postcss(),
    vue(),
    replace({
      "process.env.NODE_ENV": `"production"`
    }),
    serve({
      port: 8000,
      open: true,
      contentBase: "."
    }),
    livereload()
  ]
```

第三步:启动时，开启文件监听

```
npx rollup -C -W
```





### rollup环境的区分

我们可以在package.json中创建一个开发和构建的脚本:

```
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "rollup -c --environment NODE_ENV:production",
    "dev": "rollup -c --environment NODE_ENV:development -w"
  },
```

```js
// 默认lodash没有被打包是因为它使用commonjs,rollup默认情况下只对ES module
const commonjs = require("@rollup/plugin-commonjs")
const nodeResolve = require("@rollup/plugin-node-resolve")
const { babel } = require("@rollup/plugin-babel")
const terser = require("@rollup/plugin-terser")
const postcss = require("rollup-plugin-postcss")
const vue = require("rollup-plugin-vue")
const replace = require("rollup-plugin-replace")
const serve = require("rollup-plugin-serve")
const livereload = require("rollup-plugin-livereload")

// 区分环境
const isProduction = process.env.NODE_ENV === "production"
const plugins = [
  commonjs(),
  nodeResolve(),
  babel({
    babelHelpers: "bundled",
    exclude: /node-modules/
  }),
  postcss(),
  vue(),
  replace({
    "process.env.NODE_ENV": `"production"`
  }),
]

if (isProduction) {
  plugins.push(terser())
} else {
  const extraPlugins = [serve({
    port: 8000,
    open: true,
    contentBase: "."
  }),
  livereload()]
  plugins.push(...extraPlugins)
}

module.exports = {
  // 入口
  input: "./src/index.js",
  // 出口
  output: {
    format: "umd",
    name: "lycUtils",
    file: "./build/bundle.umd.js",
    globals: {
      lodash: "_"
    }
  },
  plugins: plugins
}
```



## 快速开发工具Vite

### 认识Vite核心理念

Vite是一种新型前端构建工具，能够显著提升前端开发体验。它主要由两部分组成：

- 一个开发服务器，它基于 [原生 ES 模块](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 提供了 [丰富的内建功能](https://cn.vitejs.dev/guide/features.html)，如速度快到惊人的 [模块热更新（HMR）](https://cn.vitejs.dev/guide/features.html#hot-module-replacement)。
- 一套构建指令，它使用 [Rollup](https://rollupjs.org/) 打包你的代码，并且它是预配置的，可输出用于生产环境的高度优化过的静态资源。

Vite 意在提供开箱即用的配置，同时它的 [插件 API](https://cn.vitejs.dev/guide/api-plugin.html) 和 [JavaScript API](https://cn.vitejs.dev/guide/api-javascript.html) 带来了高度的可扩展性，并有完整的类型支持。

 

### 浏览器模块化支持

原生浏览器是支持模块化的

```js
import { sum, mul } from "./utils/math.js";
import _ from "../node_modules/lodash-es/lodash.default.js"

const message = "hello world"
console.log(message);

const foo = () => {
  console.log("foo function exec~");
}
foo()

// 模块化代码使用
console.log(sum(10, 10));
console.log(mul(10, 20));

console.log(_.join(['abc', 'cba']));
```

#### 浏览器原生加载js模块缺点

但是如果我们不借助于其他工具，直接使用ES Module来开发有什么问题呢:

- 首先，我们会发现在使用loadash时，加载了上百个模块的js代码，对于浏览器发送请求是巨大的消耗;
- 其次,我们的代码中如果有TypeScript、less、 vue等代码时, 浏览器并不能直接识别;

![image-20230203004059815](http://img.roydust.top/img/202302030040900.png)



### Vite基础打包能力

事实上，vite就帮助我们解决了上面的所有问题

vite底层的服务器会将文件整合js、自动填充后缀名、ts文件进行转换，再转发给本地服务器



#### vite对TypeScript支持

vite对TypeScript是原生支持的， 它会直接使用ESBuild来完成编译:

- 只需要直接导入即可;

如果我们查看浏览器中的请求，会发现请求的依然是ts的代码:

- 这是因为vite中的服务器Connect会对我们的请求进行转发;
- 获取ts编译后的代码，给浏览器返回，浏览器可以直接进行解析;

注意:在vite2中，已经不再使用Koa了，而是使用Connect来搭建的服务器

![image-20230203004859992](http://img.roydust.top/img/202302030049089.png)



#### vite打包css

vite可以直接支持css的处理

- 直接导入css即可;

vite可以直接支持css预处理器，比如less

- 直接导入less; 
- 之后安装les编译器;

```
npm install less -D
```

vite直接支持postcss的转换:

- 只需要安装postcss,并且配置postcss.config.js的配置文件即可; 

```
pnpm add postcss postcss-preset-env -D
```

```js
module.exports = {
  plugins: [require("postcss-preset-env")]
}
```





### Vite打包Vue、React



#### vite对Vue支持

vite对vue提供第一优先级支持:

- Vue 3单文件组件支持: @vitejs/ plugin-vue
- Vue 3 JSX支持: @vitejs/plugin-vue-jsx
- Vue 2支持: underfin/vite-plugin-vue2

安装支持vue的插件:

```
npm install @vitejs/plugin-vue -D
```

在vite.config.js中配置插件:

```js
import { defineConfig } from 'vite'
import vue from "@vitejs/plugin-vue"

export default defineConfig({
  plugins: [
    vue()
  ]
})
```



#### vite对React支持

.jsx和.tsx 文件同样开箱即用，它们也是通过ESBuild来完成的编译:

- 所以我们只需要直接编写react的代码即可; 
- 注意:在index.html加载main.js时,我们需要将main.js的后缀,修改为main.jsx作为后缀名; 

```jsx
// React代码渲染
const root = ReactDom.createRoot(document.querySelector("#root"))
root.render(<ReactApp />)
```



#### Vite打包项目

我们可以直接通过vite build来完成对当前项目的打包工具:
```
npx vite build
```

![image-20230203013913373](http://img.roydust.top/img/202302030139417.png)

我们可以通过preview的方式，开启一个本地服务来预览打包后的效果:
```
npx vite preview
```





### Vue脚手架工具使用

在开发中，我们不可能所有的项目都使用vite从零去搭建，比如一个react项目、Vue项目;

- 这个时候vite还给我们提供了对应的脚手架工具;

所以Vite实际上是有两个工具的:

- vite: 相当于是一个构件工具，类似于webpack、 rollup;
- @vitejs/create-app: 类似vue-cli、 create-react-app;

如果使用脚手架工具呢?

```
npm create vite
```



### ESBuild工具的解析

ESBuild的特点:

- 超快的构建速度,并且不需要缓存;
- 支持ES6和CommonJS的模块化;
- 支持ES6的Tree Shaking;
- 支持Go、JavaScript的API; .
- 支持TypeScript、JSX等语法编译;
- 支持SourceMap; 
- 支持代码压缩;
- 支持扩展其他插件;

ESBuild的构建速度和其他构建工具速度对比: 

![image-20230203015123246](http://img.roydust.top/img/202302030151310.png)

ESBuild为什么这么快呢?

- 使用Go语言编写的，可以直接转换成机器代码，而无需经过字节码;
- ESBuild可以充分利用CPU的多内核,尽可能让它们饱和运行;
- ESBuild的所有内容都是从零开始编写的，而不是使用第三方，所以从-开始就可以考虑各种性能问题;

但是ESBuild不够成熟，对代码分割和css处理不够完善，所以Vite不使用ESBuild来进行开发环境打包，还是使用的稳定的Rollup



