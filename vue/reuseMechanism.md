### Vue  reuse mechanism

	vue 会尽可能的复用component已达到最小限度的rerender消耗,但是比如说我们有2个很相近的页面,
	他们路由不同来回切换的时候你会发现某个组件竟然被复用了,这是无法接受的,因为我们需要的是他初
	始化的状态,那么我们就要去了解vue的复用机制是以什么为标准的
	
	是否记得在写v-for的时候 你要求你写一个 :key 的,说是为了防止复用，那仅仅如此吗，其实不是，
	而是vue所有的组件的复用机制都是以key为判定标准的 感觉有点react 所以在这些特殊的情况下你
	需要强制定义全局唯一的key来将你需要rerender的组件脱离出vue的复用机制 但是这里不建议所有
	的组件都来定义key why？
	
	key这个属性是vue的保留属性，custom的组件是不允许定义key为prop的，这意味着这个属性vue是
	不希望开发者关心的，他是vue复用机制的内在实现凭据，所以我们应该尽量避免人为的去定义key，
	毕竟你在使用v-ifv-else的时候根本没必要关心组件是否复用了，除非碰到一些特殊情况，但是也有
	替代之法就是强制的去re-create这个组件：
	The docs have claimed that child components are properly destroyed and re-created.
