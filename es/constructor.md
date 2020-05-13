构造器  
new运算会向对象内部写“原型”这个属性（称为"[[Prototype]]"内部槽）
``` x = new Car()``` 等价于 ``` x.[[Prototype]] === Car.prototype```  
instanceof运算会再次取“x.[[Prototype][[Prototype]]”这个内部原型，也就是顺着原型链向上查找  
因为 ```x.[[Prototype]] === Car.prototype```  且 ``` Car.prototype = new Device()```  所以 ``` x.[[Prototype][[Prototype]] === Device.prototype ```

es6之后有了 class，类和函数才有了明显区别  
类智能用new来创建，而不能直接用()来调用  
对象的方法与函数也有了区分，方法不能用new来运算  

在 ECMAScript 6 之后，函数可以简单地分为三个大类：  
类：只可以做 new 运算；  
方法：只可以做调用“( )”运算；  
一般函数：（除部分函数有特殊限制外，）同时可以做 new 和调用运算。 
 
其中，典型的“方法”在内部声明时，有三个主要特征：  
具有一个名为“主对象[[HomeObject]]”的内部槽；  
没有名为“构造器[[Construct]]”的内部槽；  
没有名为“prototype”的属性。  

JavaScript 所谓的函数，其实是“一个有[[Call]]内部槽的对象”。而Function()作为 JavaScript 原生的函数构造器，它能够在创建的对象（例如this）中添加这个内部槽，而当使用继承逻辑时，用户代码（例如MyFunction()）就只是创建了一个普通的对象，因为用户代码没有能力操作 JavaScript 引擎层面才支持的那些“内部槽”  
ES6之后通过class创建的要求所有子类的构造过程都不得创建这个this实例，并主动的把这个创建的权力“交还”给父类、乃至祖先类。  

new 运算结果，如果构造器无返回值则把已经创建的this对象返回，或者直接拿构造器return的值。对于那些派生的子类（即声明中使用了extends子句的类），ECMAScript 要求严格遵循“不得在构造器中返回非对象值（以及 null 值）”的设计约定，并在这种情况下直接抛出异常。