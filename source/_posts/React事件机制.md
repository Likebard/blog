# React合成事件

## 概念介绍
React合成事件(SyntheticEvent)是React自身的一套事件系统，它是模拟原生DOM事件所有能力的一个事件对象，对所有浏览器原生事件做了兼容性包装，拥有与浏览器原生事件相同的接口。

## 详细介绍
* 它通过事件委托，在documnet上注册事件代理了组件树中的所有事件(__17版本有变动__)，并且它监听的是 __document的冒泡阶段__。
* 在React中所有事件都是合成的，不是原生事件，但可以通过e.nativeEvent属性获取原生DOM事件。
* 合成事件的目的：
  1. 进行浏览器兼容，实现更好的跨平台。React 采用的是顶层事件代理机制，能够保证冒泡一致性，可以跨浏览器执行。React 提供的合成事件用来抹平不同浏览器事件对象之间的差异，将不同平台事件模拟合成事件。
  2. 避免垃圾回收。使用原生事件会被频繁创建和回收，因此引入事件池，在事件池中获取或释放事件对象。__即 React 事件对象不会被释放掉，而是存放进一个数组中，当事件触发，就从这个数组中弹出，避免频繁地去创建和销毁(垃圾回收)__。
  3. 方便事件统一管理和事务机制。

## 原生事件回顾
1. 事件捕获
当某个元素触发某个事件（如 onclick ），顶层对象 document 就会发出一个事件流，随着 DOM 树的节点向目标元素节点流去，直到到达事件真正发生的目标元素。在这个过程中，事件相应的监听函数是不会被触发的。

2. 事件目标
当到达目标元素之后，执行目标元素该事件相应的处理函数。如果没有绑定监听函数，那就不执行。

3. 事件冒泡
从目标元素开始，往顶层元素传播。途中如果有节点绑定了相应的事件处理函数，这些函数都会被触发一次。如果想阻止事件起泡，可以使用 e.stopPropagation() 或者e.cancelBubble=true（IE）来阻止事件的冒泡传播。

4. addEventListener第三个参数的作用
  * true: 表示在事件捕获阶段触发handler
  * false(默认)： 表示在事件冒泡阶段触发handler

5. 事件委托/事件代理
简单理解就是将一个响应事件委托到另一个元素。当子节点被点击时，click 事件向上冒泡，父节点捕获到事件后，我们判断是否为所需的节点，然后进行处理。其优点在于减少内存消耗和动态绑定事件。

## 原生事件与合成事件的区别

比较角度 | 原生事件 | React 事件
------------ | ------------- | -------------
事件名称命名方式 | 名称全部小写（onclick, onblur） | 名称采用小驼峰（onClick, onBlur）
事件处理函数语法 | 字符串 | 函数
阻止默认行为方式 | 事件返回 false | 使用 e.preventDefault() 方法

## React 事件与原生事件执行顺序
参考代码：
```
class App extends React.Component<any, any> {
  parentRef: any;
  childRef: any;
  constructor(props: any) {
    super(props);
    this.parentRef = React.createRef();
    this.childRef = React.createRef();
  }
  componentDidMount() {
    console.log("React componentDidMount！");
    this.parentRef.current?.addEventListener("click", () => {
      console.log("原生事件：父元素 DOM 事件监听！");
    });
    this.childRef.current?.addEventListener("click", () => {
      console.log("原生事件：子元素 DOM 事件监听！");
    });
    document.addEventListener("click", (e) => {
      console.log("原生事件：document DOM 事件监听！");
    });
  }
  parentClickFun = () => {
    console.log("React 事件：父元素事件监听！");
  };
  childClickFun = () => {
    console.log("React 事件：子元素事件监听！");
  };
  render() {
    return (
      <div ref={this.parentRef} onClick={this.parentClickFun}>
        <div ref={this.childRef} onClick={this.childClickFun}>
          分析事件执行顺序
        </div>
      </div>
    );
  }
}
export default App;
```
可以尝试触发事件看看打印结果，根据结果我们可以知道：
1. React 所有事件都挂载在 document 对象上；
2. 当真实 DOM 元素触发事件，会冒泡到 document 对象后，再处理 React 事件；
3. 所以会先执行原生事件，然后处理 React 事件；
4. 最后真正执行 document 上挂载的原生事件。

### 原生事件和合成事件能混用吗？
由以上结果我们可以得出，原生和合成事件是不能混用的。我们设想，假如在原生事件中阻止冒泡，那么react组件上的合成事件由于统一挂在document上，所以不会再生效。

## 合成事件的事件池
合成事件对象池，是react事件系统提供的一种性能优化方式。合成事件对象在事件池统一管理，不同类型的合成事件具有不同的事件池。
* 当事件池未满时，React创建新的事件对象，派发给组件。
* 当事件池装满时，React从事件池中复用事件对象，派发给组件。
  
再看以下例子:
```
function handleChange(e) {
  console.log("原始数据：", e.target)
  setTimeout(() => {
    console.log("定时任务 e.target：", e.target); // null
    console.log("定时任务：e：", e); 
  }, 100);
}
function App() {
  return (
    <div className="App">
      <button onClick={handleChange}>测试事件池</button>
    </div>
  );
}

export default App;
```

```
function handleChange(e) {
  setData(data => ({
    ...data,
    // 这在react16以及更早期的版本中会崩溃
    text: e.target.value
  }));
}
```
在react16之前的版本，合成事件对象的事件处理函数在被调用之后，所有的属性都会被置空，所以定时器中打印的e对象的属性值全为null。如果我们想在事件函数运行完之后还获取事件对象的属性，可以使用React提供的e.persist()方法，保留所有属性,如下代码:
```
// 只修改 handleChange 方法，其他不变
function handleChange(e) {
  // 只增加 persist() 执行
  e.persist();
  
  console.log("原始数据：", e.target)
  setTimeout(() => {
    console.log("定时任务 e.target：", e.target); // null
    console.log("定时任务：e：", e); 
  }, 100);
}
```


## 具体代码分析
具体到代码，可以查看 __react-reconciler/src/ReactFiberCompleteWork.old.js__ 文件：


## 17版本的更新
1. 不再使用事件池机制，所以不存在事件处理函数调用后，event对象上属性都被置空的问题。e.persist() 在 React 事件对象中仍然可用，只是无效果罢了。

2. 事件委托的变更，不再将事件处理添加到document上，而是将事件处理渲染到React树的根DOM容器中：
    ```
    const rootNode = document.getElementById('root');
    ReactDOM.render(<App />, rootNode);
    ```
    在 React 16 及之前版本中，React 会对大多数事件进行 document.addEventListener() 操作。React v17 开始会通过调用 rootNode.addEventListener() 来代替。