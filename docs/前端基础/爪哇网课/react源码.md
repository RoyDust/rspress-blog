# react 源码

## 理念

react 就是 UI = render(data) 单向数据流

特点： 使用 JS 构建快速响应的大型 web 应用

1. CPU 卡顿：大量计算性能卡顿
2. IO 卡顿：网络请求

### CPU 卡顿

- 60HZ/120HZ 1000ms/60 = 16.6ms

- JS 线程跟 GUI 线程互斥 -> JS 脚本 render、painiting 不能同时执行

  JS 执行 -> 样式布局 -> 样式绘制

### IO 卡顿

解决方法

- suspense

  在加载时显示过度代码，而在完成后显示获取完数据的代码

- useDeferredValue

  延迟渲染不紧急的部分

并不是所有加载都需要 loading

- 加载页面： loading
- 加载时间长： loading
- 加载时间短： loading（闪烁）

### 快速响应

同步的长尾更新转化可中断的异步更新

**实现时间切片**

![img](http://img.roydust.top/img/v2-35a7b9858d93c3f8645e6dc1ef68b6e8_720w.webp)

![img](http://img.roydust.top/img/v2-55bc056d360a389314b779b9f4dc24eb_720w.webp)

## React 架构对比

### React15

**reconciler**：协调器 找出要变化的组件

- this.setState
- this. forceUpdate
- ReactDOM. render

1. 视图更新时，调用 render 方法，JSX -> VDOM
2. 要更新的 VDOM 上次的 VDOM 进行对比
3. 找出变化的部分 diff

4. 通知 renderer 将要变化的部分渲染出来

**renderer**： 渲染器 将要变化的组件渲染到页面

1. React 是跨平台：ReactDOM. RN、 ReactArt SvG、 canvas 1.

- mount: reconciler mountedComponent
- update: reconciler updateComponent

递归更新子组件

React15 缺点：

- 层级很深，递归执行时间 CPU 时间过长，超过 16.6ms
- 不支持将同步任务折分成可中断的异步更新

渲染过程

![image-20231106001707311](http://img.roydust.top/img/image-20231106001707311.png)

### React 16

- scheduler：调度器 负责调度任务的优先级，高优先没的任务进人 reconciler
- reconciler：协调器 找出要变化的组件
- renderer：渲染器 将要变化的组件渲染到页面

scheduler 机制：

1. 当浏览器有空余时间时，需要告诉我们去执行任务
2. 提供多种任务调度优先级的设置 lane

requestIdleCallback

2. 浏览器兼容性不行
3. 切换 tab，不同刘览器触发事件频率不一样

![image-20231105205920073](http://img.roydust.top/img/image-20231105205920073.png)

渲染过程

![](http://img.roydust.top/img/image-20231105230754125.png)

## Fiber 架构

1. 支持不同的优先级
2. 可中断 可恢其
3. 恢冥后要能够回到之前执行的状态

#### **reconciler**

- React16 前，reconciler 是通过递归执行，数据是存储在递归调用栈 -> stack reconciler
- React16, fiber reconciler
- 静态的数据结构：每个 fiber 都用来标识对应的一个 React Element
- 动态的数据结构：effect update delete

#### Fiber Node

```js
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode
) {
  // Instance
  // 静态数据存储的属性
  // 定义光纤的类型。在reconciliation算法中使用它来确定需要完成的工作。如前所述，工作取决于React元素的类型。函数createFiberFromTypeAndProps将React元素映射到相应的光纤节点类型
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  // 定义与此光纤关联的功能或类。对于类组件，它指向构造函数，对于DOM元素，它指定HTML标记。我经常使用此字段来了解光纤节点与哪些元素相关。
  this.type = null;
  // 保存对组件，DOM节点或与光纤节点关联的其他React元素类型的类实例的引用。通常，我们可以说此属性用于保存与光纤关联的局部状态。
  this.stateNode = null;

  // Fiber
  // Fiber关系相关属性，用于生成Fiber Tree结构
  this.return = null; // 指向父级
  this.child = null; // 指向子级
  this.sibling = null; // 指向兄弟
  this.index = 0;

  this.ref = null;

  // 动态数据&状态相关属性
  // new props,新的变动带来的新的props，即nextProps
  this.pendingProps = pendingProps;
  // prev props，用于在上一次渲染期间创建输出的Fiber的props
  this.memoizedProps = null;
  // 状态更新，回调和DOM更新的队列，Fiber对应的组件，所产生的update，都会放在该队列中
  this.updateQueue = null;
  // 当前屏幕UI对应状态，上一次输入更新的Fiber state
  this.memoizedState = null;
  // 一个列表，存储该Fiber依赖的contexts，events
  this.dependencies = null;

  // conCurrentMode和strictMode
  // 共存的模式表示这个子树是否默认是 异步渲染的
  // Fiber刚被创建时，会继承父Fiber
  this.mode = mode;

  // Effects
  // 当前Fiber阶段需要进行任务，包括：占位、更新、删除等
  this.flags = NoFlags;
  this.subtreeFlags = NoFlags;
  this.deletions = null;

  // 优先级调度相关属性
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  // current tree和working in prgoress tree关联属性
  // 在FIber树更新的过程中，每个Fiber都有与其对应的Fiber
  // 我们称之为 current <==> workInProgress
  // 在渲染完成后，会指向对方
  this.alternate = null;

  // 探查器记录的相关时间
  if (enableProfilerTimer) {
    // Note: The following is done to avoid a v8 performance cliff.
    //
    // Initializing the fields below to smis and later updating them with
    // double values will cause Fibers to end up having separate shapes.
    // This behavior/bug has something to do with Object.preventExtension().
    // Fortunately this only impacts DEV builds.
    // Unfortunately it makes React unusably slow for some applications.
    // To work around this, initialize the fields below with doubles.
    //
    // Learn more about this here:
    // https://github.com/facebook/react/issues/14365
    // https://bugs.chromium.org/p/v8/issues/detail?id=8538
    this.actualDuration = Number.NaN;
    this.actualStartTime = Number.NaN;
    this.selfBaseDuration = Number.NaN;
    this.treeBaseDuration = Number.NaN;

    // It's okay to replace the initial doubles with smis after initialization.
    // This won't trigger the performance cliff mentioned above,
    // and it simplifies other profiler code (including DevTools).
    this.actualDuration = 0;
    this.actualStartTime = -1;
    this.selfBaseDuration = 0;
    this.treeBaseDuration = 0;
  }

  if (__DEV__) {
    // This isn't directly used but is handy for debugging internals:

    this._debugSource = null;
    this._debugOwner = null;
    this._debugNeedsRemount = false;
    this._debugHookTypes = null;
    if (!hasBadMapPolyfill && typeof Object.preventExtensions === "function") {
      Object.preventExtensions(this);
    }
  }
}
```

### Fiber 是如何更新 DOM？

**双缓存机构**

在内存中，绘制当前的 fiber dom，再去替换上一帧的 fiber DOM（通过 this.alternate 指针直接切换）不会出现两帧之间更新的过程，不会出现视图中从 A->B 的中间态

```js
currentFiber.alternate = workInProgressFiber；
workInProgressFiber.alternate = currentFiber；
```

**首次执行 ReactDOM.render，会创建 fberRootNode（fberRoot**

fiber 双缓存图

![img](http://img.roydust.top/img/v2-dcc6a08b49d8ae2ab47e01b600d4586d_r.jpg)

### jsx 和 fiber 是同一个东西吗？

JSX -> babel -> React.createElement

**看看 React 的 createElement 源码**

```js
export function createElement(type, config, children) {
  let propName;

  // Reserved names are extracted
  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  // 不为 undefined 和 null
  if (config != null) {
    // config 是否含有有效的 ref 属性
    if (hasValidRef(config)) {
      ref = config.ref;
    }

    // config 是否含有有效的 key 属性
    if (hasValidKey(config)) {
      key = "" + config.key;
    }

    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // 剩余的属性被添加到一个新的 props 对象中
    // RESERVED_PROPS 包含四个属性 ref、key、__self、__source
    // 这里就是拷贝除这四个属性之外的其他属性
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }

  // children 可以是多个参数，这些参数被转移到新分配的 props 对象上。
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    if (__DEV__) {
      // do somthing
    }
    props.children = childArray;
  }

  // 解析默认 props
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  if (__DEV__) {
    // do something
  }
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props
  );
}
```

```js
const element = {
  // This tag allows us to uniquely identify this as a React Element
  $$typeof: REACT_ELEMENT_TYPE,

  // Built-in properties that belong on the element
  type: type,
  key: key,
  ref: ref,
  props: props,

  // Record the component responsible for creating this element.
  _owner: owner,
};
```

JSX 跟 Fiber 的关系

1. JSX 是用来表达组件内的数据结构，不包括 scheduler、reconciler、renderer；
2. Fiber 更多是一种更新机制
   - mount：reconciler 根据 JSX 的内容生产对应的 fiber 节点
   - update：reconciler 现将更新后的 JSX 生产 fiber，然后新老 fiber 进行对比，打上 effect

## React 基本架构

### 1. render

render 架构图

![20bd3445-3f46-45ca-b3f7-e0b061ec24d7](http://img.roydust.top/img/20bd3445-3f46-45ca-b3f7-e0b061ec24d7.png)

fiber 节点创建并构造成 render 树
fiber reconciler：可中断的异步递归 DFS

**beginwork：递**
root Fiber 深度遍历优先，为遍历的所有 fiber 节点调用 beginwork；
遍历结束后，会进入 completework；

**completework： 归**
当存在兄弟 fiber fiber.sibling !== null -> 兄弟 fiber 的递
当不存在兄弟 fiber，会进入父 fiber 的归， 最后会进 rootFiber

![image-20231106152321202](http://img.roydust.top/img/image-20231106152321202.png)

执行顺序：

```js
rootFiber beginwork
beginhork AppFiber
beginhork i am Fiber
i am Fiber completelork
span Fiber beginhork
text Fiber beginhork
completelork 1 text Fiber
completellork span Fiber
i am fiber completelork
div
App
rootFiber
```

#### beginWork

```js
function beginWork(
  current: Fiber | null, //当前存在于dom树中对应的Fiber树
  workInProgress: Fiber, //正在构建的Fiber树
  renderLanes: Lanes //第12章在讲
): Fiber | null {
  // 1.update时满足条件即可复用current fiber进入bailoutOnAlreadyFinishedWork函数
  if (current !== null) {
    const oldProps = current.memoizedProps;
    const newProps = workInProgress.pendingProps;
    if (
      oldProps !== newProps ||
      hasLegacyContextChanged() ||
      (__DEV__ ? workInProgress.type !== current.type : false)
    ) {
      didReceiveUpdate = true;
    } else if (!includesSomeLane(renderLanes, updateLanes)) {
      didReceiveUpdate = false;
      switch (
        workInProgress.tag
        // ...
      ) {
      }
      return bailoutOnAlreadyFinishedWork(current, workInProgress, renderLanes);
    } else {
      didReceiveUpdate = false;
    }
  } else {
    didReceiveUpdate = false;
  }

  //2.根据tag来创建不同的fiber 最后进入reconcileChildren函数
  switch (workInProgress.tag) {
    case IndeterminateComponent:
    // ...
    case LazyComponent:
    // ...
    case FunctionComponent:
    // ...
    case ClassComponent:
    // ...
    case HostRoot:
    // ...
    case HostComponent:
    // ...
    case HostText:
    // ...
  }
}
```

##### reconcileChildren/mountChildFibers

创建子 fiber 的过程会进入 reconcileChildren，该函数的作用是为 workInProgress fiber 节点生成它的 child fiber 即 workInProgress.child。然后继续深度优先遍历它的子节点执行相同的操作。

```js
//ReactFiberBeginWork.old.js
export function reconcileChildren(
  current: Fiber | null,
  workInProgress: Fiber,
  nextChildren: any,
  renderLanes: Lanes
) {
  if (current === null) {
    //mount时
    workInProgress.child = mountChildFibers(
      workInProgress,
      null,
      nextChildren,
      renderLanes
    );
  } else {
    //update
    workInProgress.child = reconcileChildFibers(
      workInProgress,
      current.child,
      nextChildren,
      renderLanes
    );
  }
}
```

#### completeWork

针对不同 fiber 上的 tag 标签进行不同的处理

```js
//ReactFiberCompleteWork.old.js
function completeWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  const newProps = workInProgress.pendingProps;

//根据workInProgress.tag进入不同逻辑，这里我们关注HostComponent，HostComponent，其他类型之后在讲
  switch (workInProgress.tag) {
    case IndeterminateComponent:
    case LazyComponent:
    case SimpleMemoComponent:
    case HostRoot:
   	//...

    case HostComponent: {
      popHostContext(workInProgress);
      const rootContainerInstance = getRootHostContainer();
      const type = workInProgress.type;

      if (current !== null && workInProgress.stateNode != null) {
        // update时
       updateHostComponent(
          current,
          workInProgress,
          type,
          newProps,
          rootContainerInstance,
        );
      } else {
        // mount时
        const currentHostContext = getHostContext();
        // 创建fiber对应的dom节点
        const instance = createInstance(
            type,
            newProps,
            rootContainerInstance,
            currentHostContext,
            workInProgress,
          );
        // 将后代dom节点插入刚创建的dom里
        appendAllChildren(instance, workInProgress, false, false);
        // dom节点赋值给fiber.stateNode
        workInProgress.stateNode = instance;

        // 处理props和updateHostComponent类似
        if (
          finalizeInitialChildren(
            instance,
            type,
            newProps,
            rootContainerInstance,
            currentHostContext,
          )
        ) {
          markUpdate(workInProgress);
        }
     }
      return null;
    }
```

从简化版的 completeWork 中可以看到，这个函数做了一下几件事

- 根据 workInProgress.tag 进入不同函数，我们以 HostComponent 举例
- update 时（除了判断 current===null 外还需要判断 workInProgress.stateNode===null），调用 updateHostComponent 处理 props（包括 onClick、style、children ...），并将处理好的 props 赋值给 updatePayload,最后会保存在 workInProgress.updateQueue 上
- mount 时 调用 createInstance 创建 dom，将后代 dom 节点插入刚创建的 dom 中，调用 finalizeInitialChildren 处理 props（和 updateHostComponent 处理的逻辑类似）

之前我们有说到在 beginWork 的 mount 时，rootFiber 存在对应的 current，所以他会执行 mountChildFibers 打上 Placement 的 effectTag，在冒泡阶段也就是执行 completeWork 时，我们将子孙节点通过 appendAllChildren 挂载到新创建的 dom 节点上，最后就可以一次性将内存中的节点用 dom 原生方法反应到真实 dom 中。

在 beginWork 中我们知道有的节点被打上了 effectTag 的标记，有的没有，而在 commit 阶段时要遍历所有包含 effectTag 的 Fiber 来执行对应的增删改，那我们还需要从 Fiber 树中找到这些带 effectTag 的节点嘛，答案是不需要的，这里是以空间换时间，在执行 completeWork 的时候遇到了带 effectTag 的节点，会将这个节点加入一个叫 effectList 单向链表中,所以在 commit 阶段只要遍历 effectList 就可以了（rootFiber.firstEffect.nextEffect 就可以访问带 effectTag 的 Fiber 了）

### 2. commit

首先，找到 firstEffect 的 fiber，找到对应 fiber 上的 updateQueue，诉我们要变化的 props 是哪些

```js
fiber｛
effectlist
firstEffect
lastEffect
}
```

1. before mutation: DOM 执行前
2. mutation：DOM 执行中
3. layout：DOM 行后

#### before mutation

1. 调用 flushPassiveEffects 执行完所有 effect 的任务
2. 初始化相关变量
3. 赋值 firstEffect 给后面遍历 effectList 用
4. 执行 useEffect

```js
//ReactFiberWorkLoop.old.js
do {
  // 调用flushPassiveEffects执行完所有effect的任务
  flushPassiveEffects();
} while (rootWithPendingPassiveEffects !== null);

//...

// 重置变量 finishedWork指rooFiber
root.finishedWork = null;
//重置优先级
root.finishedLanes = NoLanes;

// Scheduler回调函数重置
root.callbackNode = null;
root.callbackId = NoLanes;

// 重置全局变量
if (root === workInProgressRoot) {
  workInProgressRoot = null;
  workInProgress = null;
  workInProgressRootRenderLanes = NoLanes;
} else {
}
//将effectList城值给firstEffect
//由于每个fiber的effectlist只包含他的子孙节点
//所以根节点如果有effectTeg则不会被包含进来
//所以这里将有effectTag的根节点插入到effectlist尾部
//这样才能保证有effect的fiber都在effectList中

//rootFiber可能会有新的副作用 将它也加入到effectLis
let firstEffect;
if (finishedWork.effectTag > PerformedWork) {
  if (finishedWork.lastEffect !== null) {
    finishedWork.lastEffect.nextEffect = finishedWork;
    firstEffect = finishedWork.firstEffect;
  } else {
    firstEffect = finishedWork;
  }
} else {
  firstEffect = finishedWork.firstEffect;
}
```

**Reactv16, componentwil1xxx 为什么是 UNSAFE 前缀**

1. legacy: ReactDoM. render
2. blocking： ReactDoM. createBlockingRoot 开启 concurrent mode
3. concurrent: ReactDoM.createRoot

#### mutation

`commitMutationEffects` 中会根据对 effectList 进行第二次遍历，根据 flags 的类型进行二进制与操作，然后根据结果去执行不同的操作，对真实 dom 进行修改：

- ContentReset: 如果 flags 中包含 ContentReset 类型，代表文本节点内容改变，则执行 `commitResetTextContent` 重置文本节点的内容
- Ref: 如果 flags 中包含 Ref 类型，则执行 `commitDetachRef` 更改 ref 对应的 current 的值
- Placement: 上一章 diff 中讲过 Placement 代表插入，会执行 `commitPlacement` 去插入 dom 节点
- Update: flags 包含 Update 则会执行 `commitWork` 执行更新操作
- Deletion: flags 包含 Deletion 则会执行 `commitDeletion` **执行更新操作**

```js
// packages/react-reconciler/src/ReactFiberWorkLoop.old.js

function commitMutationEffects(
  root: FiberRoot,
  renderPriorityLevel: ReactPriorityLevel
) {
  // 对 effectList 进行遍历
  while (nextEffect !== null) {
    setCurrentDebugFiberInDEV(nextEffect);

    const flags = nextEffect.flags;

    // ContentReset：重置文本节点
    if (flags & ContentReset) {
      commitResetTextContent(nextEffect);
    }

    // Ref：commitDetachRef 更新 ref 的 current 值
    if (flags & Ref) {
      const current = nextEffect.alternate;
      if (current !== null) {
        commitDetachRef(current);
      }
      if (enableScopeAPI) {
        if (nextEffect.tag === ScopeComponent) {
          commitAttachRef(nextEffect);
        }
      }
    }

    // 执行更新、插入、删除操作
    const primaryFlags = flags & (Placement | Update | Deletion | Hydrating);
    switch (primaryFlags) {
      case Placement: {
        // 插入
        commitPlacement(nextEffect);
        nextEffect.flags &= ~Placement;
        break;
      }
      case PlacementAndUpdate: {
        // 插入并更新
        // 插入
        commitPlacement(nextEffect);
        nextEffect.flags &= ~Placement;

        // 更新
        const current = nextEffect.alternate;
        commitWork(current, nextEffect);
        break;
      }
      // ...
      case Update: {
        // 更新
        const current = nextEffect.alternate;
        commitWork(current, nextEffect);
        break;
      }
      case Deletion: {
        // 删除
        commitDeletion(root, nextEffect, renderPriorityLevel);
        break;
      }
    }
    resetCurrentDebugFiberInDEV();
    nextEffect = nextEffect.nextEffect;
  }
}
```

#### layout

接下来通过 `commitLayoutEffects` 为入口函数，执行第三次遍历，这里会遍历 effectList，执行 `componentDidMount`、`componentDidUpdate` 等生命周期，另外会执行 `componentUpdateQueue` 函数去执行回调函数。

> componentwillUnmount: mutation current fiber 还是旧的，此时获取 DOM，还是更新前的
> componentDidMount componentDidupdate: layout current fiber 转为了 workInProgress fiber 此时获取 DOM，是更新后的

```js
// packages/react-reconciler/src/ReactFiberWorkLoop.old.js

function commitLayoutEffects(root: FiberRoot, committedLanes: Lanes) {
  // ...

  // 遍历 effectList
  while (nextEffect !== null) {
    setCurrentDebugFiberInDEV(nextEffect);

    const flags = nextEffect.flags;

    if (flags & (Update | Callback)) {
      const current = nextEffect.alternate;
      // 执行 componentDidMount、componentDidUpdate 以及 componentUpdateQueue
      commitLayoutEffectOnFiber(root, current, nextEffect, committedLanes);
    }

    // 更新 ref
    if (enableScopeAPI) {
      if (flags & Ref && nextEffect.tag !== ScopeComponent) {
        commitAttachRef(nextEffect);
      }
    } else {
      if (flags & Ref) {
        commitAttachRef(nextEffect);
      }
    }

    resetCurrentDebugFiberInDEV();
    nextEffect = nextEffect.nextEffect;
  }
}
```

## React diff

> diff 算法：对于 update 的组件，他会将当前组件与该组件在上次更新是对应的 Fiber 节点比较，将比较的结果生成新的 Fiber 节点。

```
! 为了防止概念混淆，强调
```

> 一个 DOM 节点，在某一时刻最多会有 4 个节点和他相关。

```javascript
- 1.current Fiber。如果该DOM节点已在页面中，current Fiber代表该DOM节点对应的Fiber节点。
- 2.workInProgress Fiber。如果该DOM节点将在本次更新中渲染到页面中，workInProgress Fiber代表该DOM节点对应的Fiber节点。
- 3.DOM节点本身。
- 4.JSX对象。即ClassComponent的render方法的返回结果，或者FunctionComponent的调用结果，JSX对象中包含描述DOM节点信息。
```

**diff 算法的本质上是对比 1 和 4，生成 2**。也就是我们更新 jsx 与原来的 current fiber 进行对比(diff)生成 workInProgress fiber 再生成真正的 DOM 节点

#### Diff 的瓶颈以及 React 如何应对

```javascript
复制代码由于diff操作本身也会带来性能损耗，React文档中提到，即使在最前沿的算法中
将前后两棵树完全比对的算法的复杂程度为 O(n 3 )，其中 n 是树中元素的数量。

如果在React中使用了该算法，那么展示1000个元素所需要执行的计算量将在十亿的量级范围。
这个开销实在是太过高昂。
```

### 所以为了降低算法复杂度，React 的 diff 会预设 3 个限制：

```css
 1.同级元素进行Diff。如果一个DOM节点在前后两次更新中跨越了层级，那么React不会尝试复用他。
 2.不同类型的元素会产生出不同的树。如果元素由div变为p，React会销毁div及其子孙节点，并新建p及其子孙节点。
 3.者可以通过 key prop来暗示哪些子元素在不同的渲染下能保持稳定。
```

### 那么我们接下来看一下 Diff 是如何实现的

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d640e75281b64f8695c89d5882ac2c50~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 我们可以从同级的节点数量将 Diff 分为两类：

```typescript
- 1.当newChild类型为object、number、string，代表同级只有一个节点
- 2.当newChild类型为Array，同级有多个节点
```

#### 单节点

```js
function reconcileSingleElement(
  returnFiber: Fiber, // 父 fiber
  currentFirstChild: Fiber | null, // 父 fiber 下第一个开始对比的旧的子  fiber
  element: ReactElement, // 当前的 ReactElement内容
  lanes: Lanes // 更新的优先级
): Fiber {
  const key = element.key;
  let child = currentFirstChild;
  // 处理旧的 fiber 由多个节点变成新的 fiber 一个节点的情况
  // 循环遍历父 fiber 下的旧的子 fiber，直至遍历完或者找到 key 和 type 都与新节点相同的情况
  while (child !== null) {
    if (child.key === key) {
      const elementType = element.type;
      if (elementType === REACT_FRAGMENT_TYPE) {
        if (child.tag === Fragment) {
          // 如果新的 ReactElement 和旧 Fiber 都是 fragment 类型且 key 相等
          // 对旧 fiber 后面的所有兄弟节点添加 Deletion 副作用标记，用于 dom 更新时删除
          deleteRemainingChildren(returnFiber, child.sibling);

          // 通过 useFiber, 基于旧的 fiber 和新的 props.children，克隆生成一个新的 fiber，新 fiber 的 index 为 0，sibling 为 null
          // 这便是所谓的 fiber 复用
          const existing = useFiber(child, element.props.children);
          existing.return = returnFiber;
          if (__DEV__) {
            existing._debugSource = element._source;
            existing._debugOwner = element._owner;
          }
          return existing;
        }
      } else {
        if (
          // 如果新的 ReactElement 和旧 Fiber 的 key 和 type 都相等
          child.elementType === elementType ||
          (__DEV__
            ? isCompatibleFamilyForHotReloading(child, element)
            : false) ||
          (enableLazyElements &&
            typeof elementType === "object" &&
            elementType !== null &&
            elementType.$$typeof === REACT_LAZY_TYPE &&
            resolveLazy(elementType) === child.type)
        ) {
          // 对旧 fiber 后面的所有兄弟节点添加 Deletion 副作用标记，用于 dom 更新时删除
          deleteRemainingChildren(returnFiber, child.sibling);
          // 通过 useFiber 复用新节点并返回
          const existing = useFiber(child, element.props);
          existing.ref = coerceRef(returnFiber, child, element);
          existing.return = returnFiber;
          if (__DEV__) {
            existing._debugSource = element._source;
            existing._debugOwner = element._owner;
          }
          return existing;
        }
      }
      // 若 key 相同但是 type 不同说明不匹配，移除旧 fiber 及其后面的兄弟 fiber
      deleteRemainingChildren(returnFiber, child);
      break;
    } else {
      // 若 key 不同，对当前的旧 fiber 添加 Deletion 副作用标记，继续对其兄弟节点遍历
      deleteChild(returnFiber, child);
    }
    child = child.sibling;
  }

  // 都遍历完之后说明没有匹配到 key 和 type 都相同的 fiber
  if (element.type === REACT_FRAGMENT_TYPE) {
    // 如果新节点是 fragment 类型，createFiberFromFragment 创建新的 fragment 类型 fiber并返回
    const created = createFiberFromFragment(
      element.props.children,
      returnFiber.mode,
      lanes,
      element.key
    );
    created.return = returnFiber;
    return created;
  } else {
    // createFiberFromElement 创建 fiber 并返回
    const created = createFiberFromElement(element, returnFiber.mode, lanes);
    created.ref = coerceRef(returnFiber, currentFirstChild, element);
    created.return = returnFiber;
    return created;
  }
}
```

1. 先判断 key 是否一样，key 一样，type 一样，意味看 DOM 可以复用
2. key 一样，type 不一样，要将该 fiber 及其兄弟 fiber 标记为删除
3. key 不一样，标记该 fiber 为待删除，进入下一个兄弟 fiber

#### 多节点 diff

同级有多个节点 diff

在实际开发中，更新的频率 gengao

```js
function reconcileChildrenArray(
  returnFiber: Fiber,
  currentFirstChild: Fiber | null,
  newChildren: Array<*>,
  lanes: Lanes
): Fiber | null {
  // 开发环境下会校验 key 是否存在且合法，否则会报 warning
  if (__DEV__) {
    let knownKeys = null;
    for (let i = 0; i < newChildren.length; i++) {
      const child = newChildren[i];
      knownKeys = warnOnInvalidKey(child, knownKeys, returnFiber);
    }
  }

  let resultingFirstChild: Fiber | null = null; // 最终要返回的第一个子 fiber
  let previousNewFiber: Fiber | null = null;

  let oldFiber = currentFirstChild;
  let lastPlacedIndex = 0;
  let newIdx = 0;
  let nextOldFiber = null;
  // 因为在实际的应用开发中，react 发现更新的情况远大于新增和删除的情况，所以这里优先处理更新
  // 根据 oldFiber 的 index 和 newChildren 的下标，找到要对比更新的 oldFiber
  for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
    if (oldFiber.index > newIdx) {
      nextOldFiber = oldFiber;
      oldFiber = null;
    } else {
      nextOldFiber = oldFiber.sibling;
    }
    // 通过 updateSlot 来 diff oldFiber 和新的 child，生成新的 Fiber
    // updateSlot 与上面两种类型的 diff 类似，如果 oldFiber 可复用，则根据 oldFiber 和 child 的 props 生成新的 fiber；否则返回 null
    const newFiber = updateSlot(
      returnFiber,
      oldFiber,
      newChildren[newIdx],
      lanes
    );
    // newFiber 为 null 说明不可复用，退出第一轮的循环
    if (newFiber === null) {
      if (oldFiber === null) {
        oldFiber = nextOldFiber;
      }
      break;
    }
    if (shouldTrackSideEffects) {
      if (oldFiber && newFiber.alternate === null) {
        deleteChild(returnFiber, oldFiber);
      }
    }
    // 记录复用的 oldFiber 的 index，同时给新 fiber 打上 Placement 副作用标签
    lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);

    if (previousNewFiber === null) {
      // 如果上一个 newFiber 为 null，说明这是第一个生成的 newFiber，设置为 resultingFirstChild
      resultingFirstChild = newFiber;
    } else {
      // 否则构建链式关系
      previousNewFiber.sibling = newFiber;
    }
    previousNewFiber = newFiber;
    oldFiber = nextOldFiber;
  }

  if (newIdx === newChildren.length) {
    // newChildren遍历完了，说明剩下的 oldFiber 都是待删除的 Fiber
    // 对剩下 oldFiber 标记 Deletion
    deleteRemainingChildren(returnFiber, oldFiber);
    return resultingFirstChild;
  }

  if (oldFiber === null) {
    // olderFiber 遍历完了
    // newChildren 剩下的节点都是需要新增的节点
    for (; newIdx < newChildren.length; newIdx++) {
      // 遍历剩下的 child，通过 createChild 创建新的 fiber
      const newFiber = createChild(returnFiber, newChildren[newIdx], lanes);
      if (newFiber === null) {
        continue;
      }
      // 处理dom移动，// 记录 index，同时给新 fiber 打上 Placement 副作用标签
      lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
      // 将新创建 fiber 加入到 fiber 链表树中
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      previousNewFiber = newFiber;
    }
    return resultingFirstChild;
  }

  // oldFiber 和 newChildren 都未遍历完
  // mapRemainingChildren 生成一个以 oldFiber 的 key 为 key， oldFiber 为 value 的 map
  const existingChildren = mapRemainingChildren(returnFiber, oldFiber);

  // 对剩下的 newChildren 进行遍历
  for (; newIdx < newChildren.length; newIdx++) {
    // 找到 mapRemainingChildren 中 key 相等的 fiber, 创建新 fiber 复用
    const newFiber = updateFromMap(
      existingChildren,
      returnFiber,
      newIdx,
      newChildren[newIdx],
      lanes
    );
    if (newFiber !== null) {
      if (shouldTrackSideEffects) {
        if (newFiber.alternate !== null) {
          // 删除当前找到的 fiber
          existingChildren.delete(
            newFiber.key === null ? newIdx : newFiber.key
          );
        }
      }
      // 处理dom移动，记录 index，同时给新 fiber 打上 Placement 副作用标签
      lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
      // 将新创建 fiber 加入到 fiber 链表树中
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      previousNewFiber = newFiber;
    }
  }

  if (shouldTrackSideEffects) {
    // 剩余的旧 fiber 的打上 Deletion 副作用标签
    existingChildren.forEach((child) => deleteChild(returnFiber, child));
  }

  return resultingFirstChild;
}
```

在实际开发过程中，更新的频率更高，所以 diff 会有限比较当前的节点是否是要更新的

### **2 轮遍历**

1. 处理更新的节点
2. 处理非更新的节点

**第一轮遍历**

1. let i =0 进行遍历，将 jsx 中 newChild 跟 old fiber 中 oldFiber 进行比较
2. 如果可以复用，i++
3. 是否可以复用
   - key 一样，type 一样，可以頁用
   - key 不一样，跳出第一轮遍历
   - key 一样，type 不一样，标记 DELETION
4. 当 jsx 或者 old fiber 其中一个遍历完，结束第一轮遍历

**第二轮遍历**

1. newChildren oldFiber 都遍历完，直接更新，不进行第二轮遍历
2. newChildren 没遍历完，oldFiber 遍历完，直接加來下的没有遍历的 nenChildren 直接添加到 workInProgress fiber 上，标记为 PLACENENT
3. oldFiber 没遍历完，newChildren 有遍历完，oldFiber 没有遍历的要标记为 DELETION
4. newChildren 跟 oldFiber 都没有遍历完

将 oldFiber 中没有用到的以 kv 存起来

```js
Const exist ingChildren - mapRemainingChildren（returnFiber， oldFiber）；
```

遍历剩余的 newChildren，根据遍历的 newChildren 中 key 跟 existingChildren 对比，找到相同的 key 的 old fiber

#### 例子

```
// 之前 abcd` `// 之后 acdb
```

> ===第一轮遍历开始===

```css
css
复制代码a（之后）vs a（之前）

key不变，可复用

此时 a 对应的oldFiber（之前的a）在之前的数组（abcd）中索引为0

所以 lastPlacedIndex = 0;
```

> 继续第一轮遍历...

```javascript
javascript
复制代码c（之后）vs b（之前）

key改变，不能复用，跳出第一轮遍历

此时 lastPlacedIndex === 0;
```

> ===第一轮遍历结束===

> ===第二轮遍历开始===

```javascript
javascript
复制代码newChildren === cdb，没用完，不需要执行删除旧节点

oldFiber === bcd，没用完，不需要执行插入新节点

将剩余oldFiber（bcd）保存为map
// 当前oldFiber：bcd
// 当前newChildren：cdb
```

> 继续遍历剩余 newChildren

```javascript
javascript
复制代码key === c 在 oldFiber中存在

const oldIndex = c（之前）.index;

此时 oldIndex === 2;  // 之前节点为 abcd，所以c.index === 2

比较 oldIndex 与 lastPlacedIndex;

如果 oldIndex >= lastPlacedIndex 代表该可复用节点不需要移动

并将 lastPlacedIndex = oldIndex;

如果 oldIndex < lastplacedIndex 该可复用节点之前插入的位置索引小于这次更新需要插入的位置索引，代表该节点需要向右移动

在例子中，oldIndex 2 > lastPlacedIndex 0，

则 lastPlacedIndex = 2;

c节点位置不变
```

> 继续遍历剩余 newChildren

```
// 当前oldFiber：bd
// 当前newChildren：db
javascript
复制代码key === d 在 oldFiber中存在

const oldIndex = d（之前）.index;

oldIndex 3 > lastPlacedIndex 2 // 之前节点为 abcd，所以d.index === 3

则 lastPlacedIndex = 3;

d节点位置不变
```

> 继续遍历剩余 newChildren

```
// 当前oldFiber：b
// 当前newChildren：b
javascript
复制代码key === b 在 oldFiber中存在

const oldIndex = b（之前）.index;

oldIndex 1 < lastPlacedIndex 3 // 之前节点为 abcd，所以b.index === 1

则 b节点需要向右移动
```

> ===第二轮遍历结束===

```
!最终acd 3个节点都没有移动，b节点被标记为移动
```

#### 例子 2

```
// 之前 abcd
// 之后 dabc
```

> ===第一轮遍历开始===

```css
css
复制代码d（之后）vs a（之前）
key改变，不能复用，跳出遍历
```

> ===第一轮遍历结束===

> ===第二轮遍历开始===

```javascript
javascript
复制代码newChildren === dabc，没用完，不需要执行删除旧节点

oldFiber === abcd，没用完，不需要执行插入新节点

将剩余oldFiber（abcd）保存为map

继续遍历剩余newChildren
// 当前oldFiber：abcd
// 当前newChildren dabc
javascript
复制代码key === d 在 oldFiber中存在

const oldIndex = d（之前）.index;

此时 oldIndex === 3; // 之前节点为 abcd，所以d.index === 3

比较 oldIndex 与 lastPlacedIndex;

oldIndex 3 > lastPlacedIndex 0

则 lastPlacedIndex = 3;

d节点位置不变
```

> 继续遍历剩余 newChildren

```
// 当前oldFiber：abc
// 当前newChildren abc
javascript
复制代码key === a 在 oldFiber中存在

const oldIndex = a（之前）.index; // 之前节点为 abcd，所以a.index === 0

此时 oldIndex === 0;

比较 oldIndex 与 lastPlacedIndex;

oldIndex 0 < lastPlacedIndex 3

则 a节点需要向右移动
```

> 继续遍历剩余 newChildren

```
// 当前oldFiber：bc
// 当前newChildren bc
javascript
复制代码key === b 在 oldFiber中存在

const oldIndex = b（之前）.index; // 之前节点为 abcd，所以b.index === 1

此时 oldIndex === 1;

比较 oldIndex 与 lastPlacedIndex;

oldIndex 1 < lastPlacedIndex 3

则 b节点需要向右移动
```

> 继续遍历剩余 newChildren

```
// 当前oldFiber：c
// 当前newChildren c
javascript
复制代码key === c 在 oldFiber中存在

const oldIndex = c（之前）.index; // 之前节点为 abcd，所以c.index === 2

此时 oldIndex === 2;

比较 oldIndex 与 lastPlacedIndex;

oldIndex 2 < lastPlacedIndex 3

则 c节点需要向右移动
```

> ===第二轮遍历结束===

```
!可以看到，我们以为从 abcd 变为 dabc，只需要将d移动到前面。` `!但实际上React保持d不变，将abc分别移动到了d的后面。
```

### lane

##### 不同的优先级机制

React 中有三套优先级机制：

1. React 事件优先级
2. Lane 优先级
3. Scheduler 优先级

```js
// lane使用31位二进制来表示优先级车道共31条, 位数越小（1的位置越靠右）表示优先级越高
export const TotalLanes = 31;

// 没有优先级
export const NoLanes: Lanes = /*                        */ 0b0000000000000000000000000000000;
export const NoLane: Lane = /*                          */ 0b0000000000000000000000000000000;

// 同步优先级，表示同步的任务一次只能执行一个，例如：用户的交互事件产生的更新任务
export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000001;

// 连续触发优先级，例如：滚动事件，拖动事件等
export const InputContinuousHydrationLane: Lane = /*    */ 0b0000000000000000000000000000010;
export const InputContinuousLane: Lanes = /*            */ 0b0000000000000000000000000000100;

// 默认优先级，例如使用setTimeout，请求数据返回等造成的更新
export const DefaultHydrationLane: Lane = /*            */ 0b0000000000000000000000000001000;
export const DefaultLane: Lanes = /*                    */ 0b0000000000000000000000000010000;

// 过度优先级，例如: Suspense、useTransition、useDeferredValue等拥有的优先级
const TransitionHydrationLane: Lane = /*                */ 0b0000000000000000000000000100000;
const TransitionLanes: Lanes = /*                       */ 0b0000000001111111111111111000000;
const TransitionLane1: Lane = /*                        */ 0b0000000000000000000000001000000;
const TransitionLane2: Lane = /*                        */ 0b0000000000000000000000010000000;
const TransitionLane3: Lane = /*                        */ 0b0000000000000000000000100000000;
const TransitionLane4: Lane = /*                        */ 0b0000000000000000000001000000000;
const TransitionLane5: Lane = /*                        */ 0b0000000000000000000010000000000;
const TransitionLane6: Lane = /*                        */ 0b0000000000000000000100000000000;
const TransitionLane7: Lane = /*                        */ 0b0000000000000000001000000000000;
const TransitionLane8: Lane = /*                        */ 0b0000000000000000010000000000000;
const TransitionLane9: Lane = /*                        */ 0b0000000000000000100000000000000;
const TransitionLane10: Lane = /*                       */ 0b0000000000000001000000000000000;
const TransitionLane11: Lane = /*                       */ 0b0000000000000010000000000000000;
const TransitionLane12: Lane = /*                       */ 0b0000000000000100000000000000000;
const TransitionLane13: Lane = /*                       */ 0b0000000000001000000000000000000;
const TransitionLane14: Lane = /*                       */ 0b0000000000010000000000000000000;
const TransitionLane15: Lane = /*                       */ 0b0000000000100000000000000000000;
const TransitionLane16: Lane = /*                       */ 0b0000000001000000000000000000000;

const RetryLanes: Lanes = /*                            */ 0b0000111110000000000000000000000;
const RetryLane1: Lane = /*                             */ 0b0000000010000000000000000000000;
const RetryLane2: Lane = /*                             */ 0b0000000100000000000000000000000;
const RetryLane3: Lane = /*                             */ 0b0000001000000000000000000000000;
const RetryLane4: Lane = /*                             */ 0b0000010000000000000000000000000;
const RetryLane5: Lane = /*                             */ 0b0000100000000000000000000000000;

export const SomeRetryLane: Lane = RetryLane1;

export const SelectiveHydrationLane: Lane = /*          */ 0b0001000000000000000000000000000;
```

可以看到 lane 使用 31 位二进制来表示优先级车道，共 31 条, 位数越小（1 的位置越靠右）表示优先级越高。

### 实例

一、类型不同，完全重新创建

```jsx
<div>
	<Cmp></Cmp>
</div>

<p>
	<Cmp></Cmp>
</p>
```

- 销設 DOM，执行 componentwil1Unmount
- 新建 DOM, componentwillMount componentDidMount

二、类型相同

```jsx
<div className="before" title="stuff" />
<div className="after" title="stuff" />
```

DOM 保留，对比更新 props

三、相同类型的组件元素

```jsx
<ul>
	<li>1</li>
 <li>2</li>
</ul>

<ul>
	<li>1</li>
 <li>2</li>
 <li>3</li>
</ul>
```

在 render 过程中，会执行 diff，当去遍历两个子元素内容时，有差异，会生成 mutation

`注意` 如果在头部插入，会销毁列表

```jsx
<ul>
	<li>1</li>
  <li>2</li>
</ul>
// 销毀子元素列表，新建新的子元素列表，有性能问题
<ul>
  <li>3</li>
	<li>1</li>
  <li>2</li>
</ul>
```

可以在渲染的时候使用`key`来解决此问题

**在 diff 中建议：**

1. diff 不会比较不同类型子树，如果内容差不多，不建议更新类型；
2. key 要求稳定，唯一

**为什么 React 里不能使用双指针 diff**

vue： array

react：effectList 单向链表

**diff 本身的瓶颈**

1. 只针对同层元素进行 diff
2. 两个不同类型的元素产生的 diff，推倒重建
3. 可以使用稳定且唯一的 key 方便 diff 比较

Diff 算法代码

```js
function reconcileChildFibers(
  returnFiber: Fiber, // 父 Fiber
  currentFirstChild: Fiber | null, // 父 fiber 下要对比的第一个子 fiber
  newChild: any, // 更新后的 React.Element 内容
  lanes: Lanes // 更新的优先级
): Fiber | null {
  // 对新创建的 ReactElement 最外层是 fragment 类型单独处理，比较其 children
  const isUnkeyedTopLevelFragment =
    typeof newChild === "object" &&
    newChild !== null &&
    newChild.type === REACT_FRAGMENT_TYPE &&
    newChild.key === null;
  if (isUnkeyedTopLevelFragment) {
    newChild = newChild.props.children;
  }

  // 对更新后的 React.Element 是单节点的处理
  if (typeof newChild === "object" && newChild !== null) {
    switch (newChild.$$typeof) {
      // 常规 react 元素
      case REACT_ELEMENT_TYPE:
        return placeSingleChild(
          reconcileSingleElement(
            returnFiber,
            currentFirstChild,
            newChild,
            lanes
          )
        );
      // react.portal 类型
      case REACT_PORTAL_TYPE:
        return placeSingleChild(
          reconcileSinglePortal(returnFiber, currentFirstChild, newChild, lanes)
        );
      // react.lazy 类型
      case REACT_LAZY_TYPE:
        if (enableLazyElements) {
          const payload = newChild._payload;
          const init = newChild._init;
          return reconcileChildFibers(
            returnFiber,
            currentFirstChild,
            init(payload),
            lanes
          );
        }
    }

    // 更新后的 React.Element 是多节点的处理
    if (isArray(newChild)) {
      return reconcileChildrenArray(
        returnFiber,
        currentFirstChild,
        newChild,
        lanes
      );
    }

    // 迭代器函数的单独处理
    if (getIteratorFn(newChild)) {
      return reconcileChildrenIterator(
        returnFiber,
        currentFirstChild,
        newChild,
        lanes
      );
    }

    throwOnInvalidObjectType(returnFiber, newChild);
  }

  // 纯文本节点的类型处理
  if (typeof newChild === "string" || typeof newChild === "number") {
    return placeSingleChild(
      reconcileSingleTextNode(
        returnFiber,
        currentFirstChild,
        "" + newChild,
        lanes
      )
    );
  }

  if (__DEV__) {
    if (typeof newChild === "function") {
      warnOnFunctionType(returnFiber);
    }
  }

  // 不符合以上情况都视为 empty，直接从父节点删除所有旧的子 Fiber
  return deleteRemainingChildren(returnFiber, currentFirstChild);
}
```

> 用张老生常谈的图

![img](http://img.roydust.top/img/aa6d65e26b2a4b24be86f06a873852f1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)
