### Vue  define property

	vue 定义对象 会递归给其属性附加setter getter, 并在Obj每层定义一个 ob。 但是如果你先对data中的x赋值,这样他就已经递归生成了一个 Obj的属性监听tree。
	
	主要方法如下：
		组件初始化时调用 observe(value, vm) value为data中的值,vm为VueComponent本身。为了避免每次调用都走递归绑定所以这里放了一层逻辑 if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) 这就导致了递归绑定过的必然有以Observer 为实例的'__ob__' 对象，所以不会再递归去绑定。
		否则会调用new Observer(value)方法，定义'__ob__' 对象，数组调用observeArray，然后和非数组一样调用walk(value)，在walk方法中keys后循环调用Observer.convert -> defineReactive(this.value, key, val) 这里this.value是指data中的值本身。在defineReactive(obj, key, val)中会递归调用  var childOb = observe(val);
		
	解决方案：
		所以这里不能像angular一样 对vm随时随地去增加修改一个值，这样也是vue就这块性能优于其他的保证,所以只能先对所有数据进行处理然后一次性赋值给data的x
		
	PS:更上层的追溯 vue route 根据vue route你设置的路径对象一层层初始化components，调用vue的_resolveComponent并拿到VueComponent。回到vue route中cb调用初始化runQueue 中的step会被循环调用
		// 2. Validation phase
      transition.runQueue(deactivateQueue, canDeactivate, function () {
        transition.runQueue(activateQueue, canActivate, function () {
          transition.runQueue(deactivateQueue, deactivate, function () {
            // 3. Activation phase
  
            // Update router current route
            transition.router._onTransitionValidated(transition);
  
            // trigger reuse for all reused views
            reuseQueue && reuseQueue.forEach(function (view) {
              return reuse(view, transition);
            });
  
            // the root of the chain that needs to be replaced
            // is the top-most non-reusable view.
            if (deactivateQueue.length) {
              var _view = deactivateQueue[deactivateQueue.length - 1];
              var depth = reuseQueue ? reuseQueue.length : 0;
              activate(_view, transition, depth, cb);
            } else {
              cb();
            }
          });
        });
      });
          
          当到 deactivateQueue 回调的时候 activate(_view, transition, depth, cb) 被调用
          _view对象中的vm就是VueComponent 然后_view对象中的build方法被调用，VueComponent方法的_init方法被调用。如果这里你使用vue validator的话，你会发现他这里劫持了_init并override了一下
    function Override (Vue) {
    	// override _init
	    var init = Vue.prototype._init;
    	Vue.prototype._init = function (options) {
	      if (!this._validatorMaps) {
        	this._validatorMaps = Object.create(null);
      	}
     	 init.call(this, options);
	    };
  
    	// override _destroy
	    var destroy = Vue.prototype._destroy;
    	Vue.prototype._destroy = function () {
      	destroy.apply(this, arguments);
      	this._validatorMaps = null;
    	};
    }
    Vue._init -> Vue._initState -> Vue._initData 调用 observe。 如此形成了初始化