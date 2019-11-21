## react 16.x 之前生命周期
![react 16.x 之前生命周期][1]

### initialization

执行Component构造函数，接收父组件传来的props，并创建本组件的state.

```
import React, { Component } from 'react';

class ChildComp extends Component {
  constructor(props) {
    super(props);
  }
}

```

注意：<br>

- 只有继承了 Component，并书写了 constructor(props) {super(props);}, 组件才会有后续的生命周期。
- 父组件传来的 props 对子组件只读。 且props可以传递父组件的一个函数给子组件调用
- 子组件只能修改自己组件内的 state
- **[最佳实践]** 在constructor中定义state和初始化默认值。
### Mounting

注意整个过程中，只有componentDidMount中可以查找操作新的DOM，其他两个函数中DOM都是在内存中，还没最终渲染。<br>

- componentWillMount
	-  组件整个生命周期 仅且执行一次
	-  这里调用 this.setState 只会修改内存中的DOM，不会引发组件渲染。
	-  **[最佳实践]** 不要在componentWillMount中去请求ajax更新state。
	-  **[最佳实践]** 不要使用componentWillMount这个函数，原因：16.x以后不保证 componentWillMount只执行一次，而且 componentWillMount能做的都可以在constructor中做。
- render
	-	生命周期中会执行多次，当props更新，state更新都会触发 render执行
	- **[最佳实践]** 不能再里面执行 this.setState

- componentDidMount
	-	无论16.x之前还是16.x之后，这个函数只会执行一次。
	-  **[最佳实践]** 可以在这里请求ajax初始化数据
	-  **[最佳实践]** 在这里可以用函数查找，操作新的DOM

<br>

原因解释：<br>

1. 不要在componentWillMount中去请求ajax更新state ? <br>

 异步请求数据中这一次返回的是空数据（null）,因为是异步的,请求需要时间,但render不会等你慢慢请求.所以在渲染的时候没有办法等到数据到来.正确的处理方式就不要在这里请求数据,而是让组件的状态（state）在这里正确的初始化. React16之后采用了Fiber架构，只有componentDidMount声明周期函数是确定被执行一次的，ComponentWillMount的生命周期钩子都有可能执行多次
所以不加以在这些生命周期中做有副作用的操作，比如请求数据之类。

2. 不要使用componentWillMount这个函数? <br>
es6中,使用extend component的方式里的constructor函数和componentWillMount是通用的作用,所以你在构造函数里初始化了组件的状态就不必在WillMount做重复的事情了

### Updation
在props的整个过程中， 只有在componentDidUpdate函数发生以后， 父组件传过来的 props才会替换this.props。 在之前的componentWillReceiveProps，shouldComponentUpdate，componentWillUpdate，中要想操作父组件传过来的props，只能用 nextProps操作。

<br>

- props
	- componentWillReceiveProps(nextProps)
		-	**[WARN]** 父组件不能保证重传给当前组件的props是有变化的. 所以在此方法中根据nextProps和this.props来查明重传的props是否改变，以及如果改变了要执行啥，比如根据新的props调用this.setState出发当前组件的重新render

	- shouldComponentUpdate(nextProps, nextState)
		- 	**[最佳实践]** 子组件可以对比 nextProps, nextState 和当前组件中的this.props，this.state，返回true时当前组件将继续执行更新过程，返回false则当前组件更新停止，以此可用来减少组件的不必要渲染，优化组件性能。
	- componentWillUpdate(nextProps, nextState)
		- 此方法在调用render方法前执行，在这边可执行一些组件更新发生前的工作
		- **[最佳实践]** 比较少用
	- render
	- componentDidUpdate(prevProps, prevState)
		- 此方法在组件更新后被调用，可以操作组件更新的DOM
- states
	- shouldComponentUpdate(nextProps, nextState), 同上
	- componentWillUpdate(nextProps, nextState), 同上
	- render, 同上
	- componentDidUpdate(prevProps, prevState), 同上

<br>
##### 触发 update生命周期的情况
- 父组件重新render
	- 父组件调用render，无论props是否改变，都会重传props. 原因：render中可能会 <ChildComp parendData={xxx}> 
		- **[最佳实践]** 子组件可以 通过shouldComponentUpdate(nextProps, nextState)老决定是否更新子组件
	- 
-  组件本身调用 setState
	- **[WARN]** 无论state有没有变化,都会触发update流程
		- **[最佳实践]** 子组件可以 通过shouldComponentUpdate(nextProps, nextState)老决定是否更新子组件

### componentWillUnmount

- **[最佳实践]** 组件被卸载前调用，可以在这里执行一些清理工作,比如清楚组件中使用的定时器，清楚componentDidMount中手动创建的DOM元素等，以避免引起内存泄漏

## react 16.x 之后生命周期
![react 16.x 之前生命周期][2]

### 变动原因
 react v16推出的Fiber之后， 支持了async rendering，在render函数之前的所有函数，都有可能被执行多次。<br>
 原来（React v16.0前）的生命周期有哪些是在render前执行的呢？

- componentWillMount
- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate
<br>
如果开发者开了async rendering，而且又在以上这些render前执行的生命周期方法做AJAX请求的话，那AJAX将被无谓地多次调用.明显不是我们期望的结果。而且在componentWillMount里发起AJAX，不管多快得到结果也赶不上首次render，而且componentWillMount在服务器端渲染也会被调用到（当然，也许这是预期的结果），这样的IO操作放在componentDidMount里更合适。
 禁止不能用比劝导开发者不要这样用的效果更好，所以除了shouldComponentUpdate，其他在render函数之前的所有函数（componentWillMount，componentWillReceiveProps，componentWillUpdate）都被getDerivedStateFromProps替代。

也就是用一个静态函数getDerivedStateFromProps来取代被deprecate的几个生命周期函数，就是强制开发者在render之前只做无副作用的操作，而且能做的操作局限在根据props和state决定新的state

React v16.0刚推出的时候，是增加了一个componentDidCatch生命周期函数，这只是一个增量式修改，完全不影响原有生命周期函数；但是，到了React v16.3，大改动来了，引入了两个新的生命周期函数。

新引入了两个新的生命周期函数：getDerivedStateFromProps，getSnapshotBeforeUpdate

- getDerivedStateFromProps(props, state)  ： 在组件创建时和更新时的render方法之前调用，它应该返回一个对象来更新状态，或者返回null来不更新任何内容。 getDerivedStateFromProps本来（React v16.3中）是只在创建和更新（由父组件引发部分），也就是不是不由父组件引发，那么getDerivedStateFromProps也不会被调用，如自身setState引发或者forceUpdate引发。这样的话理解起来有点乱，在React v16.4中改正了这一点，让getDerivedStateFromProps无论是Mounting还是Updating，也无论是因为什么引起的Updating，全部都会被调用，具体可看React v16.4 的生命周期图。React v16.4后的getDerivedStateFromProps
- getSnapshotBeforeUpdate() 被调用于render之后，可以读取但无法使用DOM的时候。它使您的组件可以在可能更改之前从DOM捕获一些信息（例如滚动位置）。此生命周期返回的任何值都将作为参数传递给componentDidUpdate（）


[1]: ../images/react/react-before-16.x-lifecycle.png
[2]: ../images/react/react-after-16.x-lifecycle.png