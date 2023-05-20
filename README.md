## React只做两件事
**渲染UI，响应事件**

不要被它的内部数据绑定迷惑了，早年的`React`只做这两件事

真正的双向绑定框架只有`angular`

`angular`的理念就是双向绑定，它不处理ui的事情——即 数据发生变化，改变UI; UI发生变化，改变数据

所以真正的`MVVM`就是`angular`，angular提出的这个理念，React根本没有什么MVVM，MVC不是React干的事，React就是React，它不是一个MVC框架，它就干两件事 **渲染UI，响应事件**


## React虚拟dom的理解
它其实就是一个`json描述数据与结构`，可以说它是真实dom的`数据schema`，或者说是`数据模型`


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9e282e9276e49a0965a191c9333d83d~tplv-k3u1fbpfcp-watermark.image?)

上述代码转换成虚拟dom，如下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f1e3eb9047c4bb794bf9cc24069dce4~tplv-k3u1fbpfcp-watermark.image?)

在React版本16.0.0之前，没有引入fiber的时候，就是这样循环递归遍历一颗虚拟Dom树，树的循环递归是不可打断的，所以当组件多时，会一直的循环循环，并且循环一颗树的时间复杂度为O(n^3)，而循环一个链表只有O(n)，所以从循环来看，fiber就比虚拟Dom树快，并且fiber还有一个更快的点，那就是它可以被打断，它是先执行完结果，然后组装的

## React虚拟dom有什么弊端没有
react虚拟Dom就是一颗json树，那就注定了树循环，它的时间`复杂度为O(n^3)`，这种是数据结构决定的，所以它复杂，所以后来react在16.0版本之后引入了fiber，把虚拟Dom树变成了一个`链表式的fiber对象`，所以从数据结构上，`链表`的`循环时间复杂度就是O(n)`，从这一点来看就比虚拟Dom快了，所以虚拟Dom在渲染上有天然的时间损耗

##  虚拟Dom的更新机制

### React 16.0 之前
虚拟dom的更新采用的是循环和递归

#### 弊端：
 - 任务一旦开始，无法结束，直到任务结束，主线程一直被占用
 - 导致大量组件实例存在时，执行效率变低
 - 用户交互的动画界面，出现页面卡顿
   - 特别是节点移动的时候，比如拖动排序时，只有这种业务会不停的createElement，moveElement和deleteElement，所以运行完一段时间之后，必卡。这是react在16.0版本之前它虚拟Dom的结构设置，改变不了。
 
 
## react如何提升性能
 - react虚拟Dom是一个json描述数据与结构，它的循环时间复杂度是O(n^3)，所以说，它在不改变版本的情况下，就是继续沿用虚拟Dom的情况下，尽量减少跨层级的移动节点，删除节点，那可能场景限制我了，必须用这种业务呢？我们可以用第三方库，向拖动排序这种，一般不会涉及到改变Dom内容，更多的只改变它的位置，我们可以在conponentDidMount完成之后再去进行拖动排序类的业务
 
## fiber
将虚拟dom对象构建成fiber对象，根据fiber对象渲染成真实dom
### fiber的特性
 - 利用浏览器空闲时间执行，不会长时间占用主线程
 
 - 将对比更新dom的操作碎片化 —— 即它把dom的更新创建工作放在了diff一起执行，就是等它diff完了，新的dom已经生成了，然后再一次性的把变化的Dom收集起来，通过fiber的`commit`阶段统一一次性处理它变成真实dom。也就是说，在我diff完成之后，浏览器内存中的div已经是我改过之后的了，所以说它会快
   - 就是变成链表之后，不需要一起进行diff循环，我只需要根据我的数据关系，链表它是有指向的，有指针的对吧，有它的previous，next）
 - 碎片化的任务，可以根据需要被暂停——通过 `shouldComponentUpdate` 生命周期可以主动告诉react当前组件是否需要被打断

#### fiber是如何利用浏览器空闲时间执行的呢
`requestIdleCallback`是浏览器提供的API，其利用浏览器空闲时间执行任务，当前任务可被终止，优先执行更高级别的任务
```js
requestIdleCallback(function(deadline){
    deadline.timeRemaining() // 获取浏览器空闲时间
    deadline.didTimeout() // 超时也必须执行
})
```
**有了fiber，虚拟dom依然存在，只是不参与diff了，只循环dom树，不需要diff，diff就很麻烦，循环一棵树，只是把它取出来映射成fiber对象就很简单**

### fiber对象的属性

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20940e7cfb2d4f279277842a9842d9e8~tplv-k3u1fbpfcp-watermark.image?)

- stateNode: 节点Dom对象或者组件实例——就是把当前最新的state实例化存在stateNode下面，所以它直接把这个stateNode组装一下就行了，因为他已经是dom属性，dom对象了

- effects：diff的核心

- effectTag：diff的核心


fiber的目的就是我先循环一遍链表，先不管旧的，然后产生一个stateNode，就是新的，和你当前数据匹配的node，已经产生了，旧的node还在，没有销毁，数据存在虚拟dom上面，当组件初始化的时候，就把那时候的虚拟dom映射成了fiber链表，然后这个时候fiber循环一下新的链和旧的链发生对比，它`循环不改变任何东西`，但是`它打标`，就是`effectTag`，你是 修改，删除，新增还是啥的，但是我不做，然后呢，根据所打的标，执行相应的策略，比如这个标是delete，就是直接将原来的那个fiber对象干掉就完了，因为我新的是delete，意思就是干掉旧的，干掉之后，还没有渲染；还有要修改的呢，就是将所有改的方法存到`effects`当中去，然后fiber在调度的时候有一个`commit`方法，就是把这些effects一起`commit`执行掉，只执行有变化的，没有变化的不在effects和effectTag上面的，就我判断一下有没有打标，没有打标的组件就是没有动，不变；所以说，它快

## 生命周期
### 16.0版本引入fiber之前

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e5b69f74ccb44ef9e2059818f02e786~tplv-k3u1fbpfcp-watermark.image?)

#### 初始化阶段：initalization
在constructor和setup里面设置props和state

#### 挂载阶段：Mounting
组件初始化props和state之后，会执行`conponentWillMount -> render -> componentDidMount`, 到此，组件初始化完成

#### 更新阶段：Updation
当我们组件初始化完成之后，我们的数据有变化

组件触发更新有两种情况：`组件⾃⼰setState触发更新，⽗组件触发传给⼦组件的props值 变化，引发⼦组件更新`

 - 比如父组件传递过来的props变了，我们会走`conponentWillReceiveProps`；然后到下面的`shouldComponentUpdate`——就是告诉react当前组件是否需要更新，如果为retuen true，则继续走`componentWillUpdate->render->componentDidUpdate`；return false，则不更新
 - 或者是组件自身的state发生了变化，那么就走`shouldComponentUpdate`告诉它我要不要变，然后下面再走`componentWillUpdate->render->componentDidUpdate`
 
 ##### shouldComponentUpdate: 16.8之前提升react性能的唯一手段
 通过这个生命周期函数我们可以主动告诉react，当前组件的数据哪些需要diff，哪些不需要
 
#### 卸载阶段：Unmounting
调用componentWillUnmount生命周期函数

### 16.0版本之后
移除了以下生命周期函数，因为fiber的diff算法实在render里面执行的，所以说render之前的这些方法改变了state，就会影响整个链表的渲染效果。
 - componentWillUpdate
 - conponentWillReceiveProps
 - componentWillUpdate
 
 并且加入了新的生命周期钩子函数
 - getDerivedStateFromProps
 - getSnapshotBeforeUpdate
 
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8456287e120c40f8a9e21891cf30a28d~tplv-k3u1fbpfcp-watermark.image?)

## diff
### 旧diff算法，没引入fiber之前

 - 针对树结构（tree diff）：对UI层的Dom节点跨层级的操作进行忽略。（数量少）
 
 - 针对组件结构（component diff）：拥有相同类的两个组件生成相似的树形结构，拥有不同类的两个组件会生成不同的属性结构
 
 - 针对元素结构（element diff）：对于同一层级的一组节点，使用具有唯一性的id区分（key属性）

#### key的作用
react中的key是用在`diff算法`中的，用来`区别`这个`节点`是否发生了`变化`，它是通过`tag`或`key`任何一个变化，都认为这个节点是新建，或者是删除，而不认为它是旧节点的复用

### fiber后新diff的过程
 - 通过state计算出新的fiber节点
 - 对比tag和key确定节点的操作（修改，删除，新增，移动）
 - effectTag 标记发生变化fiber对象
 - 收集所以标记的fiber对象，形成effectList
 - commit阶段一次性处理所有变化的节点
 
### 单节点diff

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05cce908ba1d4979b89b512ea3418fc1~tplv-k3u1fbpfcp-watermark.image?)

### 多节点diff

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c028f582c5194f859c404124ae95db91~tplv-k3u1fbpfcp-watermark.image?)

- 第一种： 四个节点都没变，说明是内容变了，tag和key没有动，属于内容的更新

- 第二种： 更新兼新增，新的fiber链表多出了一个E，说明E是新增的

- 第三种：旧链表有的而新链表没有，说明是删除

- 第四种：移动
   - 当`新链遍历到D`时，此时`旧链`中的lastIndex是B的位置，`索引是1`, `旧链中D`的索引位置是`3`，`新链中D`的位置是`2`，这个时候，`react认为`新旧链中`D的位置`都在lastIndex的`右边`，所以D不动。然后更新lastIndex，而`D在旧链中`的索引位置是`3`，`lastIndex永远指向旧链中的位置`。然后进行下一轮循环，此时`D为固定节点`，没有动过，持续新链中`循环到C`，新链中C的位置在D的右边，比D大，但是在旧链中C的位置在D的左边，说明C动了

- 第五种：移动+删除

## react状态管理

### flux
UI产⽣动作消息（点一个按钮，或是请求的数据更新），将动作传递给分发器（dipatcher），然后以广播的形式告诉所有的store，然后只有订阅的store做出反应，传递新的state的改变UI

 - UI产⽣动作消息（点一个按钮，或是请求的数据更新），将动作传递给分发器（dipatcher）
 
 - 分发器⼴播给所有store
 
 - 订阅的store做出反应，传递新的state给UI
 
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a6e2adabcb4463bbaf17d730cf633fb~tplv-k3u1fbpfcp-watermark.image?)

### redux
- 单⼀数据源，整个应⽤state存储在⼀个单⼀store树中

- State状态为只读，不应该直接修改state，⽽是通过action触发 state修改

- 使⽤纯函数进⾏状态修改，需要开发者书写reducers纯函数进⾏处 理，reducer通过当前状态树和action进⾏计算，返回⼀个新的 state


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7cf2bbd0ec6a4a7b8e0c5a0204fc3cd6~tplv-k3u1fbpfcp-watermark.image?)


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2d38cc1004b4b38bca76bd9cbdaff3a~tplv-k3u1fbpfcp-watermark.image?)

### mobx
- 定义状态并使其可观察

- 创建视图以响应状态变化

- 更改状态（⾃动响应UI变化）


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/375685c61f254d168ec9bc5ebbc76b76~tplv-k3u1fbpfcp-watermark.image?)

## Hook
### 为什么用 Hooks
- 组件之间复⽤状态逻辑很难
    
    
    React 没有提供将可复用性行为“附加”到组件的途径（例如，把组件连接到 store）。如果你使用过 React 一段时间，你也许会熟悉一些解决此类问题的方案，比如 [render props](https://react.docschina.org/docs/render-props.html) 和 [高阶组件](https://react.docschina.org/docs/higher-order-components.html)。但是这类方案需要重新组织你的组件结构，这可能会很麻烦，使你的代码难以理解。如果你在 React DevTools 中观察过 React 应用，你会发现由 providers，consumers，高阶组件，render props 等其他抽象层组成的组件会形成“嵌套地狱”。尽管我们可以[在 DevTools 过滤掉它们](https://github.com/facebook/react-devtools/pull/503)，但这说明了一个更深层次的问题：React 需要为共享状态逻辑提供更好的原生途径。

- 复杂组件变的难以理解

  例如，组件常常在 `componentDidMount` 和 `componentDidUpdate` 中获取数据。但是，同一个 `componentDidMount` 中可能也包含很多其它的逻辑，如设置事件监听，而之后需在 `componentWillUnmount` 中清除。这样一个方法中了包含了很多其他的逻辑，就很容易出现bug，导致逻辑不一致
  
- 难以理解的class——作为clss来讲呢他很难理解class的继承关系和实例化这么一个过程

  除了代码复用和代码管理会遇到困难外，我们还发现 class 是学习 React 的一大屏障。你必须去理解 JavaScript 中 `this` 的工作方式，这与其他语言存在巨大差异。还不能忘记绑定事件处理器。大家可以很好地理解 props，state 和自顶向下的数据流，但对 class 却一筹莫展
  
## 重要的Hook钩子函数

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a4720a78aaa41d8b6432f771543f70a~tplv-k3u1fbpfcp-watermark.image?)

### useState
**React hook：not magic, just arrays**

#### useState的使用
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fff2fd8163cf49bf8c671fddc757de68~tplv-k3u1fbpfcp-watermark.image?)

#### useState执行过程
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8dbcb6bd45d6484eab41cb2aa17346d3~tplv-k3u1fbpfcp-watermark.image?)

创建两个空的array，state and setters，如下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87154a8defbe45ad835b9c4f16c31d7a~tplv-k3u1fbpfcp-watermark.image?)

第⼀次渲染，循环所有useState，把state和 setter⽅法分别压⼊两个数组

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/168eaae2dcd44ce6b3fde222406a66b5~tplv-k3u1fbpfcp-watermark.image?)

更新state，触发render，cursor被重置，根据 useState的声明顺序依次拿出state值，更新UI

#### 使用useState需要注意什么

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48b4d03b43b44c7ca6a137c7ca8d1306~tplv-k3u1fbpfcp-watermark.image?)

如上代码：
 - 第⼀次执⾏，state有三个元素
 - 更新state触发render
 - 此时tag=false，不会执⾏if流程，也就是没有useState
 - 最后导致setNum2不起作⽤
 
如下图：第一次执行时，正常，但是更新之后，函数会重新执行一遍，状态state会被重新码一遍，但是setUnusedNum函数没有被重新执行，因为flase没有执行到里面，所以现在的setNum的state和setters错位了，所以导致setNum2不生效

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a20fa3106f2b440ca374864d204065a0~tplv-k3u1fbpfcp-watermark.image?)

##### 结论
不要在循环、判断逻辑中使⽤ useState，最好是在函数顶部使⽤

### useEffect
副作⽤函数
- useEffect函数第⼀个参数为callback函数，就是要执⾏的函数

- 第⼆个参数为⼀个数组，数组中是要监听的变量

- []空数组之所以会起到componentDidMount的作⽤，是因为每次 更新，[]空数组没有任何监听的变量，也就是不存在变化⼀说，所以只 会被执⾏⼀次

- useEffect函数返回的第⼀个函数作为销毁state使⽤

### useLayoutEffect
useLayoutEffect主要在useEffect函数结果不满意时才被⽤到，⼀般的经验是当处理dom改变带来的副作⽤才会被⽤到，该Hook执⾏时，浏览器并未对dom进⾏渲染，较useEffect执⾏要早

#### useEffect and useLayoutEffect 的区别
- useEffect和useLayoutEffect在Mount和Update阶段执⾏⽅法 ⼀样，传参不⼀样

- useEffect异步执⾏，⽽useLayoutEffect同步执⾏

- Effect⽤HookPassive标记useEffect，⽤HookLayout标记 useLayoutEffect

### useMemo
传递⼀个函数和依赖项，当依赖项发⽣变 化，执⾏函数，该函数需要返回⼀个值

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ac576244ab44e13b505f6ef85ea69a7~tplv-k3u1fbpfcp-watermark.image?)

### useCallback
返回⼀个函数，当监听的数据发⽣变 化才会执⾏回调函数

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad5c9eaa8cb64a8b8287c047859dba54~tplv-k3u1fbpfcp-watermark.image?)

上述代码：只有当count2变化时，才会执行，如果设置为空数组，则会执行一次初始化，但是`count2`的值不会变

### useRef
```js
const refContainer = useRef(initialValue);
```

`useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（`initialValue`）。返回的 ref 对象在组件的整个生命周期内持续存在

### useReducer
- 聚合参数，以达到开发效率的提升以及代码的简洁
- useReducer在re-render时候不会改变存储位置，state作为 props传给⼦component不会产⽣diff的效率问题（useMemo优化）

#### 如何使用

比如下面这个案例：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/001570f91a3e4b6d89ff53c4e2fdb127~tplv-k3u1fbpfcp-watermark.image?)

App组件

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0062e1f92bcc44318a7b82a2c47a58ee~tplv-k3u1fbpfcp-watermark.image?)

定义的一个reducer
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa20834f79174aa7b973ae81570170c5~tplv-k3u1fbpfcp-watermark.image?)

用`useReducer`改写App组件

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40f63e2ae2834b51b7d24fd28b9dc64a~tplv-k3u1fbpfcp-watermark.image?)
