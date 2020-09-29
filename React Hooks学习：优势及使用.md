+++
title= "React Hooks学习：优势及使用"
date= "2019-08-15"
categories= [ "web前端" ]
tags= [ "React" ]
+++
***Hooks 是React的一次革命性升级，本文将对其优势和API进行比较全面的解析***

## 为什么要有hooks

在没有hooks之前，除了对于一些无状态组件可以使用函数来声明组件以外，大家都会使用class来声明组件。作为一个主要工作内容为Android开发的我，早已习惯万物皆class，而Android中的Activity（可以理解为每一个交互界面）就是class，所以也欣然接受，并且在ES6中class带有的constructor、super以及react的生命周期函数对使用Java的Android开发者来讲很容易理解接受。 
 
但是使用class来声明组件却存在以下问题：  

1. 状态逻辑难以复用  
假如有一段逻辑代码需要在多个组件中使用，那么在以前，可以通过以下几个方式来实现：  

- copy代码，显然不符合代码设计的原则  
- 继承，一方面，js只支持单继承（Java中可以通过接口实现对多个类的实现），想要对复用多个组件的逻辑就无能为力；另一方面，只为了复用部- 分逻辑而滥用继承，显然是违背oop原则的  
- HOC高阶组件，使用HOC复用的原理很简单，就是包裹封装，比如我们想实现对onScroll方法的复用：

```javascript
import React,{Component} from 'react'

function scrollable(Child) {
    return class ScrollWrapper extends Component {
        ref = React.createRef()
        onScroll = (...args) => {
            console.log('onscroll')
            this.ref.current.onScroll(...args)
        }
        componentDidMount() {
            document.addEventListener('scroll', this.onScroll, false);
        }

        componentWillUnmount() {
            document.removeEventListener('scroll', this.onScroll, false);
        }

        render() {
            return <Child ref={this.ref} />
        }
    }
}

class ScrollableApp extends Component {
    onScroll(){
        console.log('child onscroll')
    }
    render() { 
        return <div style={{color:'red',height:10000,width:800}}>
            
        </div>
    }
}

export default scrollable(ScrollableApp);

```
ScrollableApp通过高阶组件即函数scrollable()封装后，复用了ScrollWrapper中的onScroll()方法，并且还能再ScrollWrapper的onScroll使子组件ScrollableApp中的onScroll也能得到调用；使用渲染属性也可以实现复用：


```javascript
export class Scrollable extends Component {
    onScroll = (...args) => {
        console.log('onscroll')
        console.log(this.props.children)        
    }
    componentDidMount() {
        document.addEventListener('scroll', this.onScroll, false);
    }

    componentWillUnmount() {
        document.removeEventListener('scroll', this.onScroll, false);
    }

    render() {
        return this.props.render()
    }
}

export class ScrollableApp extends Component {
    onScroll() {
        console.log('child onscroll')
    }
    render() {
        return <div style={{ color: 'red', height: 10000, width: 800 }}>

        </div>
    }
}

```
使用时：

```javascript
function App() {
  return (
    <div>
      <Scrollable render={() => <ScrollableApp></ScrollableApp>}></Scrollable>
    </div>
  );
}

export default App;
```
这个方法和高阶组件方式差不多，不再赘述  
*使用这两者虽然能实现逻辑复用，但无疑对代码的简洁和运行性能都有不少的损耗*

- 另外，“组合优于继承”，或许可以使用策略模式，抽取出不同的类来封装这些逻辑，在使用时引入相关的类来实现，但这无疑有点过度设计，也大大增加了代码结构的复杂度  


2. 类组件复杂，难以维护，主要指生命周期函数混乱，比如上面的onScroll监听，在componentDidMount和componentWillUnmount分别要注册反注册，相关的逻辑分散在不同地方，而在componentDidMount往往还需要处理类似网络请求等各种初始化的动作，也导致不相关的逻辑混杂在一起，使得代码难以维护（这个在Android开发中其实也是再正常不过做法...）

3. this指向等问题  
上面有一段代码：

```javascript
onScroll = (...args) => {
   ...
}

```
这里使用类属性的方式定义onScroll，才能通过this.onScroll访问到该方法，而如果声明为类成员函数，则在向下一级组件传递回调函数时无法正确访问到该方法


而hooks则很好的解决了以上的问题  


## 使用hooks

### useState

1. 使用  

在没有react hooks之前，组件可以分为有状态组件和无状态组件，如：

```javascript
class Counter extends Component {
    state = {
        count: 0
    }
    render() {
        return (
            <div>
                {this.state.count}
            </div>
        )
    }
}
``` 

```javascript
function Counter(props) {
    return (
        <div>
            {props.count}
        </div>
    )
}
```
第一种写法Counter中存储了状态state，而函数组件写法中只能通过props来获取状态

使用useState：

```javascript
import React, { useState } from 'react'

export default function Counter(props) {
    const [count, setCount] = useState(0)
    return (
        <div>
            <button
                onClick={() => {
                    setCount(count=>count + 1)
                }}>
            </button>
            {count}
        </div>
    )
}
```
这里`const [count, setCount] = useState(0)`中，相当于定义了一个count变量作为该组件state的一个属性，而useState中传入的值为count的初始默认值（也可以不传入，则为undefined），setCount为改变count值的方法；以上代码等价于：

```javascript
export class Counter extends React.Component {
    state = {
        count: 0
    }
    render() {
        return (
            <div>
                <button
                    onClick={() => {
                        this.setState(
                            {
                                count: this.state.count + 1
                            }
                        )
                    }}>
                </button>
                {this.state.count}
            </div>
        )
    }
}
```
*可见，使用useState使得代码大大简化，可以不使用class声明组件，也不用担心this指向的问题；并且从此我们不用再以有无状态来区分组件了，因为函数组件也可以拥有状态*



2.原理  
这里有几个值得探讨的问题：  

- useState()如何确定应该返回的是哪一个component的state  
这个很简单，因为js运行在单线程环境中，所以在运行到某一个useState函数时，可以获取到对应的运行上下文处在哪一个component中  

- 如何确定useState对应于哪一个返回值  
思考以下的伪代码：

```javascript
function Counter(props) {
  if (someCondition) {
    useState();
  }
  useState();
}
```
在实际执行中会报错，而且如果eslint配置了react-hooks/rules-of-hooks，会直接编译报错  
实际上为了代码尽可能简洁，useState是通过记录第一次运行时的顺序来确定之后的每次运行分别返回对应哪个state的，所以***Hooks函数必须始终以相同的次序和数量被调用***

- setState相同值的时候会否重新渲染  
改写之前的代码，setCount时每次都为0，发现并不会执行render函数

```
function Counter(props) {
    const [count, setCount] = useState(0)
    console.log('render')
    return (
        <div>
            <button
                onClick={() => {
                    setCount(0)
                }}>
            </button>
            {count}
        </div>
    )
}
```

假如我们的state中存储的是对象呢？

```javascript
function Counter(props) {
    const [countObj, setCountObj] = useState({ count: 0 })
    console.log('render')
    return (
        <div>
            <button
                onClick={() => {
                    countObj.count = countObj.count + 1
                    setCountObj(countObj)
                }}>
            </button>
            {countObj.count}
        </div>
    )
}
```
点击button，发现也未重新渲染。因此在setState时，如果为对象，对比的地址值未改变，并不会重新render，这和PureComponent类似


### useEffect

effec被翻译过来为副作用，但是这个确很容易产生语义误解；实际上副作用实际上指的是视图组件与视图组件之外系统进行交互的行为u，比如与DOM交互，网络请求，数据持久化操作等  
假如我们需要在componentDidMount之后设置onScroll监听，使用class的写法为：

```javascript
class Scrollable extends React.Component {
    onScroll = () => {
        console.log('onscroll')
    }
    componentDidMount() {
        document.addEventListener('scroll', this.onScroll, false);
    }
}
```

通过useEffect改写则为：

```javascript
function Scrollable(props) {
    useEffect(() => {
        document.addEventListener('scroll', this.onScroll, false);
    })
    return <div></div>
}

```
useEffect()中的函数会在每次componentDidMount、componentDidUpdate的时候执行，如果要在componentWillUnmount中取消监听也很简单，只需在useEffect()传入的函数中return相关处理函数即可：

```javascript
function Scrollable(props) {
    useEffect(() => {
        document.addEventListener('scroll', this.onScroll, false);
        return () => {
            document.removeEventListener('scroll', this.onScroll, false);
        };
    })
    return <div></div>
}

```

除此之外，useEffect还可以传入第二个参数，该参数类型为数组；这里分为三种情况：

1.不传入数组参数：在不传入该数组的情况下（参考上面的代码），每次componentDidMount、componentDidUpdate或componentWillUnmount时都会执行对应的副作用函数

2.传入空数组：该副作用会在组件整个生命周期中只执行一次、清理一次；这很适用于对网络请求、事件监听等操作

3.传入非空数组：该副作用会在数组中的各个参数发生变化时（对象比较地址值），才会在对应的生命周期中重新执行  

通过useEffect，可以使得我们的代码更简洁，逻辑更清晰易维护，对数组参数的控制也能帮助我们更轻松的写出高性能的代码

### useContext

在没有hooks之前Context就已经存在，用以实现跨层级数据传递，一般使用Consumer和ContextType实现
  
但是Context的似乎使用得不是很多，多数还是通过redux的store存储全局数据；useContext使得Context的使用更容易，可以在函数组件中使用Context，并且不用依赖ContextType，避免了每一个组件只能对应一个ContextType的缺点，当然也不需要Consumer

使用很简单（这里只是演示，代码结构可以根据实际做优化；这里顺便列出以往使用Consumer和ContextType实现Context数据传递的写法作对比）：

```javascript
import React, { Component, createContext, useContext, useState } from 'react'
const CountContext = createContext(0)

function App() {
  const [count, setCount] = useState(0)
  return (
    <div>
      <button
        onClick={() => {
          setCount(count + 1)
        }}>
      </button>
      <CountContext.Provider value={count}>
        <CounterByConsummer></CounterByConsummer>
        <Counter></Counter>
        <CounterByContextType></CounterByContextType>
      </CountContext.Provider>
    </div>
  );
}

//useContext写法
function Counter(props) {
  const count = useContext(CountContext)
  return (
    <div>
      {count}
    </div>
  )
}
//Consummer写法
class CounterByConsummer extends Component {
  render() {
    return (
      <CountContext.Consumer>
        {count => <div>{count}</div>}
      </CountContext.Consumer>
    )
  }
}

//ContextType写法
class CounterByContextType extends Component {
  static contextType = CountContext
  render() {
    const count = this.context
    return (
      <div>
        {count}
      </div>
    )
  }
}

export default App;
```

需要注意的一点是，不要滥用context，因为会破坏组件的独立性

### useMemo&useCallback

#### 理解memo
为了提高react的运行效率，避免无用的重渲染，我们常使用继承PureComponent的方式；而在函数组件则可以使用`React.memo(Component)`来达到同样的效果；

#### 使用useMemo
React.memo()针对组件，而useMemo则是针对组件的方法，思考如下代码：

```javascript
import React, { useMemo, useState } from 'react'

function App() {
  const [name, setName] = useState('smartzheng')
  const [age, setAge] = useState(18)
  return (
    <>
      <button
        onClick={() => {
          setName(name + 'changed')
        }}>
        changeName
      </button>
      <button
        onClick={() => {
          setAge(age + 1)
        }}>
        changeAge
      </button>
      <Description age={age} name={name}></Description>
    </>
  );
}


function Description({ name, age }) {
  function getAge(age) {
    console.log('changeAge')
    return age + '岁'
  }
  const newAge = useMemo(() => getAge(age), [age])

  return <>
    <div>
      {name}
    </div>
    <div>
      {newAge}
    </div></>
}
export default App;

```
这段代码主要做的是显示两个button，一个用来改变name，一个改变age，而子组件中对name和age进行显示，并且通过getAge()在age后加上“岁”字。测试发现，点击changeName和changeAge都会导致子组件重新执行getAge()，这并不是我们想要的结果；这里就可以通过useMemo来实现只有在age发生变化时才执行getAge()，使用很简单，只需将 `const newAge = useMemo(() => getAge(age), [age])` 改为`const newAge = useMemo(() => getAge(age), [age])`即可  

在`useMemo(() => getAge(age), [age])`中，传入的是一个函数，可以理解为一个回调，数组`[age]`代表该回调只有在age变化时才会执行，而执行的内容为`getAge(age)`

#### 使用useCallback

useCallback和useMemo很类似，不过他返回的是缓存的函数：`const fnA = useCallback(fnB, [a])`，代表useCallback会将fnB函数返回，返回值是否改变依赖于a值是否改变。举例如下：

```javascript
import React, { useState, useCallback } from 'react';
const set = new Set();

export default function Callback() {
  const [count, setCount] = useState(0);
  const [val, setVal] = useState('');

  const callback = useCallback(() => {
    console.log(count);
  }, [count]);
  set.add(callback);
  return <div>
    <h1>{count}</h1>
    <h1>{set.size}</h1>
    <div>
      <button onClick={() => setCount(count + 1)}>changeCount</button>
      <input value={val} onChange={event => setVal(event.target.value)} />
    </div>
  </div>;
}
```

这里的callback就是对`() => {console.log(count)`的缓存，通过一个set来存放它，只有当点击changeCount的按钮是set的size才会发生变化，说明只有在count变化时，才会返回新的callback方法，这对减少重复创建相同的方法对象很有帮助


### useRef

在hooks之前，常用createRef来创建ref来获取DOM元素的引用，React Hooks中则提供了useRef

1. 使用useRef获取DOM元素的引用  

```javascript
import React, { PureComponent, useRef } from 'react';

function App(props) {
  const countRef = useRef();
  return (
    <>
      <Counter ref={countRef}></Counter>
      <button onClick={() => { console.log(countRef.current) }}></button>
    </>
  )
}

class Counter extends PureComponent {
  render() {
    return (
      <div>

      </div>
    )
  }
}
export default App
```
在上述代码中，通过useRef()创建了countRef并在Counter组件上进行赋值，点击button，每次都会正确打印出Counter组件  
值得注意的是，这里的Counter如果使用函数组件则会报错，提示`function components can not be given refs`，原因是函数组件会被React底层处理，被class wrap，所以直接给该函数组件进行ref赋值没有意义；从这里也可以发现函数组件还不能完全替代类组件

2. 使用useRef存储对象
正常情况下，如果在函数组件中声明一个变量，那么该变量会在每次渲染时重新创建，而使用useRef可以实现跨越声明周期存储数据，思考如下代码：

```javascript
import React, { useRef,useEffect,useState } from 'react'

function App(props) {
  const [count, setCount] = useState(0)
  let interval;
  useEffect(()=>{
    interval = setInterval(()=>{
      setCount(count+1)
    },1000)
  },[])
  if(count>5){
    clearInterval(interval)
  }
  return (
    <>
      <div>{count}</div>
    </>
  )
}

export default App
```
当count大于5时，清除定时器，这样写肯定是无效的，因为每次都会创建一个新的interval，clearInterval中的interval并不是最开始的interval，通过useRef改写即可实现：

```javascript
import React, { useRef,useEffect,useState } from 'react'

function App(props) {
  const [count, setCount] = useState(0)
  let interval = useRef();
  useEffect(()=>{
    interval.current = setInterval(()=>{
      setCount(count+1)
    },1000)
  },[])
  if(count>5){
    clearInterval(interval.current)
  }
  return (
    <>
      <div>{count}</div>
    </>
  )
}

export default App
```

### 自定义Hooks

前面提到类组件有三个缺点，首当其冲的是逻辑复用问题，我们可以通过自定义Hooks来解决该问题，例如我们通过自定义useCount来复用一个定时器：

```javascript
import React, { useRef, useEffect, useState } from 'react'

function App(props) {
  const [count] = useCount(0)
  return (
    <>
      <div>{count}</div>
    </>
  )
}

function useCount(defaultCount){
  const [count, setCount] = useState(0)
  let interval = useRef();
  useEffect(() => {
    interval.current = setInterval(() => {
      setCount(count => count + 1)
    }, 1000)
  }, [])
  useEffect(()=>{
    if (count >= 5) {
      clearInterval(interval.current)
    }
  })
  return [count, setCount]
}
export default App
```

***关于React Hooks的优势和常用API使用先写到这，后面的文章再对Hooks的深层实现原理和自定义Hooks学习和解析***






