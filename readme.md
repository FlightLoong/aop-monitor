# aop-monitor
[![npm](https://img.shields.io/npm/v/aop-monitor.svg)](https://www.npmjs.com/package/aop-monitor)


### Background
埋点代码常常要侵入具体的业务逻辑，这使埋点代码变得很繁琐并且容易出错。因此，最直接的做法就是将埋点代码与业务逻辑解耦，也就是“声明式编程”，从而降低埋点的难度。

### Design ideas

```js
1. 用户的每一次操作，必然会触发相应的回调，我们可以做到监控那些回调
1. 监控对象可以是类或者是对象,如果类监控的是类，则监控其的prototype对象
2. 监控函数调用发生在业务逻辑函数调用之后
3. 监控逻辑不能修改业务逻辑的数据或者返回结果
```

### Feature
```js
1. 完全与业务解耦，业务代码里可以永远见不到埋点的逻辑了
2. 埋点数据无需手动发送，只需在最开始的时候传入一次 发送的方法，埋点的方法只需要返回数据就行了，系统会自动为你发送
3. 支持在用户某个操作之后，一次性发送多个埋点，你需要做的只是将相应的数据以数组方式返回
4. 监控对象可以是类或者是对象，换句话说，支持react和vue
5. 极其轻量：所有代码加上注释没超过100行，真正用来实现功能的代码也就50行左右
```

### Getting Started

monitor.js
```js
import initAopMonitor from "aop-monitor"

const send = params => console.log(JSON.stringify(params));

export default initAopMonitor(send)
```
ps: 推荐将该逻辑放到单独一个模块，并配置webpack alias以方便引用

### Usages

```js
import monitor from 'path/to/monitor.js'
// 埋点代码： 返回的数据最终会被之前传入的send方法发送出去，如果该业务方法内有多处埋点，支持返回数组的方式
// 数组内的数据会被遍历，逐个发送
const watchList = {
    eat () {
        console.log('after eat')
        const name = this.name
        return {
            eventType: 'eat',
            name
        }
    }
}

// 监控Person类
@monitor(watchList)
class Person {
    constructor (props) {
        this.name = props.name
        this.eat()
        this.dance()
    }
    eat () {
        console.log(`${this.name} is eating`)
    }
    dance () {
        console.log(`${this.name} is dancing`)
    }
    render () {
        return null
    }
}

new Person({name: 'huax'})

```
    ps: 如果需要监控对象，写法是 
```js

const watch = monitor(watchList) 
watch({eat () {}, dance () {}, ...etc})

````
#### Results

```js

"huax is eating"
"after eat"
"{\"eventType\":\"eat\",\"name\":\"huax\"}"
"huax is dancing"

```
