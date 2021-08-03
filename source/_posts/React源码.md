# React源码

## Reconciler:
每当更新发生时， __Reconciler__ 会做如下工作：
* 调用函数组件、或class组件的render方法，将返回的JSX转为虚拟DOM。
* 将本次虚拟DOM跟上次的对比，根据Diff算法找出变化的部分。
* 通知 __Renderer__ 将变化的虚拟DOM渲染到页面上。

## Renderer:
负责真正渲染组件到宿主环境，
React支持跨平台，不同的平台有不同的 __Renderer__，以下：
* ReactDOM
* ReactNative
* ReactTest
* ReactArt
每次更新发生时，Renderer接到Reconciler通知，将变化的组件渲染在当前宿主环境。

## React15架构-->React16架构
1. 在React15中：
* 每个组件更新都会通过Reconciler和Renderer,他们是交替工作的。
* 使用递归式更新，而整个过程都是同步的。

缺点在于一旦更新开始，中途就无法中断。当递归层级过深，时间超过了16ms,用户交互就会卡顿。

2. 在React16中：
* 新增了调度器Scheduler：
  1. 当浏览器有剩余时间的时候通知我们。
  2. 还提供了多种调度优先级的任务设置。
* Reconciler的改变：
  从递归处理虚拟Dom变成了可以中断的循环过程。只有所有组件都完成Reconciler的工作，才会统一交给Renderer，而不是交替工作。由于Reconciler的所有工作都在内存中进行，所以即使反复中断，用户也不会看见更新不完全的DOM。
* React Element数据结构的改变:
  曾经用于 __递归的无法中断的更新__ 的 __虚拟DOM__ 数据结构已经无法胜任 __异步的可中断更新__ 的需求。于是，叫做 __Fiber节点__ 的全新数据结构诞生。

## Fiber节点的结构
```
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // 作为静态数据结构的属性
  this.tag = tag;// Fiber对应组件的类型 Function/Class/Host...
  this.key = key;// key属性
  // 大部分情况同type，某些情况不同，比如FunctionComponent使用React.memo包裹
  this.elementType = null;
  // 对于 FunctionComponent，指函数本身，对于ClassComponent，指class，对于HostComponent，指DOM节点tagName
  this.type = null;
  this.stateNode = null;// Fiber对应的真实DOM节点

  // 用于连接其他Fiber节点形成Fiber树
  this.return = null;// 指向父级Fiber节点
  this.child = null;// 指向子Fiber节点
  this.sibling = null;// 指向右边第一个兄弟Fiber节点
  this.index = 0;

  this.ref = null;

  // 作为动态的工作单元的属性
  // 保存本次更新造成的状态改变相关信息
  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;

// 保存本次更新会造成的DOM操作
  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;

  // 调度优先级相关
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  // 指向该fiber在另一次更新时对应的fiber
  this.alternate = null;
}
```
