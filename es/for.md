#### 词法声明 for 语句解析
```
for (let i in x) ...;
```  
只有一个作用域并不够，es在这类for语句的时候，首先是创建一个forEnv给let i声明,然后会为循环体创建一个forEnv的子作用域loopEnv让i in x循环，这样let i就不会每次循环都重复声明,在运行的时候还会创建n个loopEnv的副本iterationEnv，他们作为loopEnv的子作用域，用来维系管理每次循环上下文中标识符。
```
for (let/const i; i < 2; i++) 
```  
如果只支持一个作用域，那么循环体就会和词法声明在一个作用域中，就可能导致词法声明覆盖的问题。其实我们并没有机会在循环体中覆盖词法声明，比如单语句的时候
```
for (let x = 102; x < 105; x++) let x = 200;  
```  
这样会报错，因为单语句没有{}，就没有iterationEnv的子作用域，所以如果支持词法声明就会导致每次循环都重复声明。
而我们如果用{}包裹代码，那么这其实是作为iterationEnv的子作用域存在的。  
forEnv > loopEnv > iterationEnv > {}用户自己创建的块级作用域.  
##### 那么这里可以类推什么是闭包？  
函数像for循环一样，会被重复进入，那么他需要有一个作用域来管理重复进入之前的标识符，也就是所谓的oldEnv，在for循环里就相当于loopEnv，而这个用来管理老标识符的函数作用域，就称之为闭包。

