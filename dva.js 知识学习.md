# dva.js 知识学习

## JavaScript语言

### 变量声明

#### const和let

* const 常量
* let 变量

		const DELAY = 1000;
		let count = 0;
		count = count + 1;
		
#### 模板字符串

	const user = 'world';
	console.log(`hello ${user}`);
	
	const content = `
		Hello ${user},
		Thanks for ordering the {user}
	`
	
#### 默认参数

	function logActivity(activity = 'skiing'){
		console.log(activity);
	}
	
	logActivity();
	
### 箭头函数

函数的快捷写法，不需要function关键字，省略return关键字

	[1, 2, 3].map(x => x + 1);  // [2, 3, 4]

### 模块的Import和Export

import用于引入模块, export用于导出模块

	//引入全部
	import dva from 'dva';
	
	//引入部分
	import { connect } from 'dva';
	import { Link, Route } from 'dva/router'
	
	//引入全部并作为github对象
	import * as github from './services/github';
	
	//导出默认
	export default App;
	//部分导出，需import {App} from './file' 引入
	export class App extend Component {};
	
### ES6对象和数组

#### 析构赋值

析构赋值让我们从Object或Array里取部分数据存为变量

	//对象
	const user = { name: 'guanguan', age: 2};
	const { name, age } = user;
	console.log(`${name} : ${age}`); //guanguan : 2
	
	//数组
	const arr = [1, 2];
	const [foo, bar] = arr;
	console.log(foo); // 1
	
析构传入的函数参数

	const add = (state, { payload }) => {
		return state.concat(playload);
	};

析构搭配alias

	const add = (state, { playload: todo }) => {
		return state.concat(todo);
	};
	
#### 对象字面量改进

析构的反向操作，用于重新组织一个Object

	const name = 'duoduo';
	const age = 8;
	const user = {name, age}; //{name: 'duoduo', age:8}
	
定义对象方法时，还可以省去function关键字

	app.model(
		reducers: {
			add() {} //等同于 add: function(){}
		},
		effects: {
			*addRemote() {} //等同于 addRemote: function*() {}
		},
	);
	
#### Spread Operator(展开运算符)

Sperator即3个点 ...,有几种不同的使用方式

组装数据

	const todos = ['Learn dva']
	[...todos, 'learn antd']; //['Learn dva', 'Learn antd']

也可用于获取数组的部分项

	const arr = ['a', 'b', 'c'];
	const [first, ...rest] = arr;
	rest; // ['b', 'c']
	
	// With ignore
	const [first, , ...rest] = arr;
	rest; // ['c']
	
还可收集函数参数为数组

	function directions(first, ...rest) {
		console.log(rest);
	}
	directions('a', 'b', 'c'); //['b', 'c'];
	
代替apply

	function foo(x, y, z) {}
	const args = [1,2,3];
	
	//下面两句效果相同
	foo.apply(null, args);
	foo(...args);
	
对于Object而言，用于组合新的Object。

	const foo = {
		a: 1,
		b: 2,
	};
	
	const bar = {
		b: 3,
		c: 2,
	};
	const d = 4;
	
	const ret = { ...foo, ...bar, d}; // { a:1, b:3, c:2, d:4 }
	
此外，在JSX中Spread Operator还可用于扩展props

#### Promises

Promise用于更优雅地处理异步请求

	fetch('/api/tosos')
		.then(res => res.json())
		.then(data => ({ data }))
		.catch(err => ({ err }));
		
定义Promise

	const delay = (timeout) => {
		return new Promise(resolve => {
			setTimeout(resolve, timeout);
		});
	};
	
	delay(1000).then(_ => {
		console.log('executed');
	});
	
#### Generators

生成器，通过yield关键字实现暂停功能

	app.model(
		namespace: 'todos',
		effects: {
			*addRemote({ payload: todo }, { put, call }) {
      			yield call(addTodo, todo);
      			yield put({ type: 'add', payload: todo });
    		},
		},
	);

### React Component

#### Stateless Functional Components(React 无状态组件)

简洁，无状态，不是Object，没有this作用域，是pure function

	function App(props) {
		function handleClick() {
			props.dispatch({ type: 'app/create' });
		}
		
		return <div onClick={handleClick}>${props.name}</div>
	}
	
等同于

	class App extends React.Component {
		handleClick() {
			this.props.dispatch({type: 'app/create'});
		}
		render() {
			return <div onClick={this.handleClick.bind(this)}>${this.props.name}</div>
		}
	}
	
#### JSX

##### Component嵌套
类似HTML，JSX里可以给组件添加子组件

	<App>
		<Header />
		<MainContent />
		<Footer />
	</App>
	
##### className
class是保留词，所以添加样式时，需要用className代替class

	<h1 className="fancy">Hello dva</h1>
	
##### JavaScript 表达式
JavaScript表达式需要用{}括起来，会执行并返回结果

	<h1>{ this.props.title }</h1>
	
##### Mapping Arrays to JSX
可以把数组映射为JSX元素列表

	<ul>
		{ this.props.todos.map((todo, i) => <li key={i}>{todo}</li>) }
		
##### 注释
尽量别用//做单行注释

##### Spread Attributes
用于扩充组件props

	const attrs = {
		href: 'http://example.org',
		target: '_blank',
	};
	<a {...attrs}>Hello</a>
	
等同于

	const attrs = {
		href: 'http://example.org',
		target: '_blank',
	};
	<a href={attrs.href} target={attrs.target}>Hello</a>
	
#### Props
用于传递数据

##### propTypes
JavaScript是弱类型语言，尽量声明propTypes对props进行校验

	function App(props) {
		return <div>{props.name}</div>;
	}
	App.propTypes = {
		name: React.PropTypes.string.isRequired,
	};
	
内置的prop type有:

* PropTypes.array
* PropTypes.bool
* PropTypes.func
* PropTypes.number
* PropTypes.object
* PropTypes.string

#### CSS Modules

##### 理解CSS Modules

button在构建之后会被重命名。因此可以用简短的描述性名字。不需要关心命名冲突问题

##### 定义全局CSS

CSS Modules默认是局部作用域的，想要声明一个全局规则，可用:global语法

比如：

	.title {
		color: red;
	}
	:global(.title) {
		color: green;
	}

然后在引用的时候：

	<App className={styles.title} /> // red
	<App className="title" /> // green
	
##### classnames Package

在一些复杂场景中，一个元素可能对应多个className，而每个className又基于一些条件来决定是否出现。这时可以使用classNames这个库

	import classnames from 'classnames';
	const App = (props)  => {
		const cls = classnames({
			btn: true,
			btnLarge: props.type === 'submit',
			btnSmall: props.type === 'edit',
		});
		return <div className={ cls } />;
	}
	
这样传入不同的type给App组件，就会返回不同的className组合

	<App type="submit" /> // btn btnLarge
	<App type="edit" /> // btn btnSmall
	
### Reducer
reducer是一个函数，接受state和action，返回老的或新的state。即: (state, action) => state

#### 增删改

	app.model({
		namespace: 'todos',
		state: [],
		reducers: {
			add(state, { payload: todo }){
				return state.concat(todo);
			}，
			remove(state, { payload: id }){
				return state.filter(todo => todo.id !== id);
			},
			update(state, {payload: updatedTodo}){
				return state.map(
					todo => {
						if (todo.id === updatedTodo.id) {
							return {...todo, ...updatedTodo};
						} else {
							return todo;
						}
					}
				);
			}
		},
	});
	
#### 嵌套数据的增删改

建议最多一层嵌套，以保持state的扁平化

		app.model({
			namespace: 'app',
			state: {
				todos: [],
				loading: false,
			},
			reducers: {
				add(state, { payload: todo }){
					const todos = state.todos.concat(todo);
					return { ...state, todos };
				},
			},
		});
		
### Effect

	app.model({
		namespace: 'todos',
		effects: {
			*addRemote({payload: todo}, {put, call}) {
				yield call(addTodo, todo);
				
			}
		},
	});
	
#### Effects

##### put
用于触发action

	yield put({ type: 'todos/add', paypoad: 'Learn Dva' });
	
##### call
用于调用异步逻辑，支持promise

	const result = yield call(fetch, '/todos');
	
##### select
用于从state里获取数据

	const todos = yield select(state => state.todos);
	
#### 错误处理

##### 全局错误处理

可以统一进行错误处理

##### 本地错误处理

如果需要对某些effects错误进行特殊处理，需要在effect内部增加try catch

	app.model({
		effects: {
			*addRemote(){
				try {
				} catch(e){
					console.log(e.message);
				}
			}
		},
	});
	
#### 异步请求

##### GET和POST

	import request from '../util/request';
	
	//GET
	request('/api/todos');
	
	//POST
	request('api/todos', {
		method: 'POST',
		body: JSON.stringify({a : 1}),
	});
	
##### 统一错误处理

	{
		status: 'error',
		message: '',
	}
	
编辑utils/request.js，加入以下中间件

	function parseErrorMessage( { data } ) {
		const {status === 'error'}{
			throw new Error(message);
		}
		return { data };
	}
	
这样这类错误就会统一走onError hook

### Subscription
订阅，用于订阅一个数据源，然后根据需要dispatch相应的action

#### 异步数据初始化

	app.model({
		subscription: {
			setup({ dispatch, history }) {
				history.listen(({pathname}) => {
					if (pathname === '/users') {
						dispatch({
							type: 'users/fetch',
						});
					}
				}
				);
			}
		}
	});
	
### Router

#### Config with JSX Element(router.js)

	<Route path="/" component={App}>
		<Route path="accounts" component={Accounts}/>
		<Route path="statements" component={Statements}/>
	</Route>
	
#### Route Components
Route Components 是指 ./src/routes/目录下的文件，他们是./src/router.js里匹配的Component

##### 通过connect绑定数据

	import {connect} from 'dva';
	function App() {}
	
	function mapStateToProps(state, ownProps){
		return {
			users: state.users,
		};
	}
	export default connect(mapStateToProps)(App);
	
然后在App里就有了dispatch和users两个属性

#### Injected Props (e.g. location)
Route Component会有额外的props用来获取路由信息

### 基于action进行页面跳转

	import { routerRedux } from 'dva/router';
	
	//Inside Effects
	yield put(routerRedux.push('/logout'));
	
	//Outside Effects
	dispatch(routerRedux.push('/logout'));
	
	//With query
	routerRedux.push({
		pathname: '/logout',
		query: {
			page: 2,
		},
	});
	
	