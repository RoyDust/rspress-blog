# react基础与源码
## HOC

- 高阶组件：HOC，在 React 中是复用组件逻辑的技巧 -＞ HOC 并不是组件 API，是基于 React的组合特性而形成的设计模式

- 组件作为參数，返回值也是组件的区数
- HOC 是纯函数，没有修改传入的组件，返回的是将组件进行包装后的新组件

### 为什么要使用HOC？

1. 抽离重复代码，进行组件复用
2. 条件渲染：进行权限控制
3. 劫持传入组件的生命周期：获取组件渲染性能，或者日志打点



### HOC 的分类

#### 1.属性代理（从外）

![image-20231031151802836](http://img.roydust.top/img/image-20231031151802836.png)

![image-20231031152228170](http://img.roydust.top/img/image-20231031152228170.png)  

![image-20231031164433859](http://img.roydust.top/img/image-20231031164433859.png)

#### 2.反向继承（从内）

```js
const HOC = (WrapperComponent) => {
  return class extends WrapperComponent {
    render() {
      return super.render();
    }
  };
};
```

1. 能够通过 this 访问到原组件上的内容；
2. super.render（）获取到传入组件的 render（），可以进行条件渲染；
3. 可以劫持生命周期

![image-20231031165233123](http://img.roydust.top/img/image-20231031165233123.png)

![image-20231031165617671](http://img.roydust.top/img/image-20231031165617671.png)

![image-20231031165646448](http://img.roydust.top/img/image-20231031165646448.png)

![image-20231031165931015](http://img.roydust.top/img/image-20231031165931015.png)



#### 属性代理和反向继承对比

1. 属性代理：并没有深入到组件内部，在外部去操作WrapperComponent，可以操作props，抽象state/event,或者条件渲染。
2. 反向继承：更多是从“继承”的角度出发，可以在内部去操作WrapperComponent，可以在内部操作props，state，event，甚至是生命周期，功能更加强大



##### 实现组件复用

常规写法

![image-20231031171144275](http://img.roydust.top/img/image-20231031171144275.png)

使用HOC进行组件复用

![image-20231031171245564](http://img.roydust.top/img/image-20231031171245564.png)

##### 获取组件渲染性能

![image-20231031171456341](http://img.roydust.top/img/image-20231031171456341.png)



## Hooks

为什么要使用Hooks？

1. 针对函数时编程，可以将组件中相关联的部分，根据业务分开
2. 开发更友好，扩展性更强
3. class更多是一种语法糖，hooks相对函数式编程来说，学习成本更低



### 官方Hooks

#### 1.useState

定义值

![image-20231031173157714](http://img.roydust.top/img/image-20231031173157714.png)

#### 2.useEffect

一般用来处理：当组件init、DOM render 、 DOM操作、数据请求

![image-20231031173721523](http://img.roydust.top/img/image-20231031173721523.png)



#### 3. useLayoutEffect

**渲染更新之前的useEffect**



useEffect和useLayoutEffect区别？

- useEffect： 组件更新挂载完成 -> DOM绘制完成 -> 执行useEffect的回调
- useLayoutEffect：组件更新挂载完成 -> 执行useLayoutEffect -> DOM绘制完成

渲染组件问题

1. useEffect： 闪动
2. useLayoutEffect：卡顿

![image-20231101015631871](http://img.roydust.top/img/image-20231101015631871.png)

#### 4.useRef

用来获取元素。缓存数据

![image-20231101020028107](http://img.roydust.top/img/image-20231101020028107.png)



#### 5.useContext

用来获取从父级组件传递过来的context

- useContext(Context)
- Context.Consumer

![image-20231101020417777](http://img.roydust.top/img/image-20231101020417777.png)



#### 6.useReducer

入参：

1. 视为reducer，包含state action，返回根据action不同的state
2. state的初始值

出参：

1. 更新后的state
2. 派发更新的dispatch，执行dispatch，就是执行了一次useState，会重新渲染render

![image-20231101021029974](http://img.roydust.top/img/image-20231101021029974.png)

#### 7.useMemo

![image-20231101021244358](http://img.roydust.top/img/image-20231101021244358.png)

#### 8.useCallback

- useCallback 是 cb 的函数
- useMemo 是 cb 的运行结果

 ![image-20231101021802582](http://img.roydust.top/img/image-20231101021802582.png) 



###  自定义Hooks

![ ](http://img.roydust.top/img/image-20231101022201222.png)

![image-20231101022218872](http://img.roydust.top/img/image-20231101022218872.png)

## 异步组件

传统模式：渲染组件-> 请求数据 -> 再渲染组件。

异步模式：请求数据-> 渲染组件。

开启Suspense模式

```js
js
复制代码function Index(){ 
    const [ userInfo , setUserInfo ] = React.useState(0) 
    React.useEffect(()=>{ 
        /* 请求数据交互 */ 
        getUserInfo().then(res=>{ 
            setUserInfo(res) 
        }) 
    },[]) 
    return 
    <div> 
        <h1>{userInfo.name}</h1>; 
    </div> 
} 
export default function Home(){ return <div> <Index /> </div> }
```

模拟一个简单的Suspense

```js
js
复制代码export class Suspense extends React.Component{ 
    state={ isRender: true } 
    componentDidCatch(e){ 
    /* 异步请求中，渲染 fallback */ 
    this.setState({ isRender:false }) 
    const { p } = e Promise.resolve(p).then(()=>{ 
        /* 数据请求后，渲染真实组件 */ 
        this.setState({ isRender:true }) }) 
    } 
    render(){ 
        const { isRender } = this.state 
        const { children , fallback } = this.props 
        return isRender ? children : fallback 
    } 
}
```

React.lazy基本使用

```js
js
复制代码const LazyComponent = React.lazy(()=>import('./text'))
js
复制代码const LazyComponent = React.lazy(() => import('./test.js')) 
export default function Index(){ 
    return <Suspense fallback={<div>loading...</div>} > <LazyComponent /> </Suspense> 
}
```

Suspense能解决什么？

- Suspense让数据获取库与 React 紧密整合。如果一个数据请求库实现了对 Suspense 的支持，那么，在 React 中使用 Suspense 将会是自然不过的事。
- Suspense能够自由的展现，请求中的加载效果。能让视图加载有更主动的控制权。
- Suspense能够让请求数据到渲染更流畅灵活，我们不用在componentDidMount请求数据，再次触发render，一切交给Suspense解决，一气呵成。
