#React-Redux

###单一数据源 

	store 全局只有一个, reducers可多个，操作store的不同部分
	触发action是改变status的唯一方式,是store数据的唯一来源
		
	reducer 就是一个函数，接收旧的 state 和 action，返回新的 state。
		
```
import { combineReducers } from 'redux';

const todoApp = combineReducers({
  visibilityFilter,
  todos
});

export default todoApp;
```
注意上面的写法和下面完全等价：

```
export default function todoApp(state, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  };
}
```

发起action

```
	store.dispatch(addTodo('Learn about actions'));
```

Redux 应用中数据的生命周期遵循下面 4 个步骤：

	1. 调用 store.dispatch(action)
	2. Redux store 调用传入的 reducer 函数
		PS: 注意 reducer 是纯函数。它应该是完全可预测的：多次传入相同的输入必须产生相同的输出。它不应做有副作用的操作，如 API 调用或路由跳转。这些应该在 dispatch action 前发生。
	3. 根 reducer 应该把多个子 reducer 输出合并成一个单一的 state 树。
	4. Redux store 保存了根 reducer 返回的完整 state 树。
	
	
例如Todo这个例子 你输入了一串text之后 点击回车发生了什么呢

	1. component TodoTextInput 被 Header引用，所以自身的handleSubmit 被调用 
	2. 在该方法中this.props.onSave(text); 由于onSave: PropTypes.func.isRequired, onSave已被注册为prop并且类型为function。 并且在Header中 有如下代码 onSave={this.handleSave.bind(this)}
	3. 所以 createPrototypeProxy中的proxyMethod被调用 参数传入为handleSave
	4. 在handleSave中 this.props.addTodo(text); 同理Header被App引用且
	Header.propTypes = {
  		addTodo: PropTypes.func.isRequired
	};
	5. 区别在于这里的App 是连接到Redux的Smart Components， 而不是Dumb Components,
	function mapDispatchToProps(dispatch) {
  		return {
    		actions: bindActionCreators(TodoActions, dispatch)
  		};
	}
	export default connect(
      mapStateToProps,
      mapDispatchToProps
    )(App);
	所以bindActionCreators中的bindActionCreator被调用，
	actionCreator.apply(undefined, arguments)
	传入参数为你在input中的输入，
	action中的addTodo被调用
	6. 触发dispatch，dispatch(actionCreator.apply(undefined, arguments));入参是返回的action(这里是ADD_TODO的返回值)
	7. dispatch中currentReducer(currentState, action), currentState可理解为当前的数据合集store中的合集 currentReducer 即调用combineReducers中的combination(state, action)
	combination中的_utilsMapValues2是调用mapValues中的mapValues方法
	reducer(previousStateForKey, action)中 reducer就是我们的reducers/todos.js的function todos(state = initialState, action),然后返回新的state
	mapValues方法接收到新的state并return继续回到combination方法，得到finalState即使该返回值。
	8. 返回到dispatch方法并将isDispatching置为flase,调用所有注册的监听器,原理是subscribe(listener),现在我们通过connect方法
	export default connect(
  		mapStateToProps,
  		mapDispatchToProps
	)(App);
	connect(.js) 中Connect.prototype.handleChange被调用setState,state被更新(全数据)。中间有复杂的逻辑包括dirty check，diff之类的
	dispatch方法返回action,结束
	9. 回到handleSubmit,header组件的setState被调用
	10. 之后是一串复杂的校验，关闭逻辑(react原生),终于到了ReactCompositeComponent(.js)的_renderValidatedComponentWithoutOwnerOrContext方法,开始渲染组件
	var renderedComponent = inst.render();
	Header的render被调用(react dom)
	TodoTextInput被渲染(react dom)
	<header className="header"> 这个tag被渲染
	update header(true dom)
	update input(true dom)
	同理开始处理MainSection：
 	顺序为 先section容器 然后从下置下，从内往外，复杂组件调用其render，依然是从下置下，从内往外
 	
 	总结: 绑定事件先触发，依次调用关联到的其他组件的方法，直到到顶部组件(smart components),由于顶层组件连接到Redux, actionCreator.apply调用对应action，获得返回值后触发dispatch，然后调用reducer,生成新的state,调用所有监听器(目前主要就是connect的修改store的state),开始渲染所有组件。
 	

###总结
	
	react-redux 与 flux的理念是一致的，单一数据流，所有的改变必须通过触发action来达到，Redux 试图让 state 的变化变得可预测，将模型的更新逻辑全部集中于一个特定的层（Flux 里的 store，Redux 里的 reducers）。
	Redux 没有 dispatcher 的概念。Redux 设想你永远不会变动你的数据，如果要改变不能在reducers里，reducers必须保持纯度，上面有提到，所以应该是创建新的action和state去更新state
	redux最重要的一点是单一数据源，基于virtualDom 和 js Diff，单一数据源的巨大好处不言而喻，如时间旅行、记录/回放或热加载。因为只是state的改变，你可以将你的代码响应像电影一样一帧帧的前进后退，或者是退到之前的某一帧！
	


	

###Redux action with socket
	socket通信这里不做介绍,但是与后端通信必然为异步,所以需要middleware去处理异步的action。在实际做简易web通讯工具的时候遇到了问题
	你创建socket的时机必然不在middleware的return function中,因为这是每次action都会触发的,但是socket只创建一次,而每次socket的监听的那些后端返回聊天信息必须在return function中,这会导致这样一种情况:客户A登录,分配到1号聊天室,这时候去告诉客服,客服确认后进去1号聊天室,可是他们的对于聊天记录的socket监听函数都在return function内仍未触发,这就意味着你需要在外部触发一个action去使某些监听函数注册但又不至于影响正常聊天的功能。

我们声明2个变量,第一个是这条消息时候再需要广播,第二个是这条消息的内容

```
	let {shouldDilivery, diliveryText} = {shouldDilivery : true, diliveryText : ""};	
```
然后我们先在return function外监听聊天消息,并发起新的action

```
	socket.on('chat message', function(msg) {
		shouldDilivery = false;
		diliveryText = msg;
    	dispatch(addTodo(msg));
    });
```
最后我们在return function内控制逻辑

```
	if (shouldDilivery) {
	    	socket.emit('chat message', action.text);
	    } else {
	    	shouldDilivery = true;
	    	action.text = diliveryText;
    		return next(action);
	    }
```
将原来每次都触发聊天记录的行为改成需要广播才触发,否则只是在本地修改states
	
	



