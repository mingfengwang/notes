##es6 note

#### let & const

	增加块作用域 {}
	let 和 const 都不再提升作用域且同名变量不可在同块作用域内多次声明，同块作用域若有变量以let const声明则全局变量不再生效，若在声明之前引用会报错
	let 主要是变量 而 const是常量，故const声明后必须立即初始化并不可修改值，如果是引用类型则是不可修改引用地址，如果需要完全冻结对象 Object.freeze({})
	跨模块常量则可以使用模块化的 import
	
#### 解构赋值

	解构赋值右侧不可是不具备Iterator接口，换言之必须可遍历
	function*返回的是迭代器对象，方法体不立即执行 调用迭代器的next()方法后返回yield值(按顺序，如果yield* 指向了一个新的function*，则先插入新的function*的yield返回值)
	如果一个数组成员不严格等于undefined，默认值是不会生效的。
	字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类数组的对象。
	
	function move({x = 0, y = 0} = {}) {
  		return [x, y];
	}
	不等价于
	function move({x, y} = { x: 0, y: 0 }) {
  		return [x, y];
	}