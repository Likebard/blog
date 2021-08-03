# Hooks特性探究

## 与Class组件不同的心智模型

### 使用hooks组件，props和state、还有内部的函数，它们在每一次的render中都是彼此独立的。一次render中生成的props和state也仅属于这次特定的渲染

考虑如下代码，它采用class写法:

```import React from "react";

export default class ClassDiv extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            count: 0,
        };
        this.showAlert = this.showAlert.bind(this);
    }
    showAlert() {
        setTimeout(() => {
            alert(`count is ${this.state.count}`);
        }, 3000);
    }
    render() {
        return <div>
            <div>{this.state.count}</div>
            <button onClick={() => {
                this.setState((preState) => {
                    return {
                        count: preState.count + 1
                    }
                })
            }}>点击+1</button>
            <button onClick={this.showAlert}>alert</button>
        </div>
    }
}
```

当我们触发含有弹窗的点击事件后，在这3秒内继续对state上的count累加，当弹窗出现时，显示的是count的最新值。
再看如下hook写法：

```import { useState, useEffect } from "react";

export default function HookDiv() {
    const [count, setCount] = useState(0);
    function handelAlertClick(){
        setTimeout(() => {
            alert(`count is ${count}`);
        }, 3000);
    }
    return (
      <div>
        <div>{count}</div>
        <button onClick={() => {setCount((count) => count+1)}}>加一</button>
        <button onClick={handelAlertClick}>alertCount</button>
      </div>
    );
}
```

同样是在弹窗的点击事件触发后的3秒内对count累加，3秒后显示的是点击事件触发“那一刻”count的值。这揭示了两者之间存在着某种差异，官网上提过把hooks组件的每一次render看作“一帧”，结合这个例子来看，确实弹窗哪怕被延迟弹出，显示的count依然是那一次渲染的count值，而非像class组件那样总是显示最新的。底层的具体机制还有待探究，但这种情况确实是hook机制依赖闭包的一个证明

### Ref

针对上述情况，假如我想在某次渲染中，拿到未来的props和state，实现跟class组件一样的效果怎么实现？那就要使用useRef,参考如下代码：

```import { useState, useEffect } from "react";

export default function Example() {
    const [count, setCount] = useCount(0);
    const lastCount = useRef(count);

    useEffect(() => {
        //设置要预测的值
        lastCount.current = count;
        setTimeout(() => {
            //读取要预测的值
            console.log("count is", lastCount.current);
        }, 3000);
    });

    return <div>
        <div>{count}</div>
        <button onClick={() => {　setCount(count +１)}　}>+1</button>
    </div>
}

## 避免effect多次调用
可以设置deps依赖项，相当于告诉react，我在effect函数中的逻辑只依赖于deps中的某个元素。此外其他的props,state发生变化后，不要执行effect
