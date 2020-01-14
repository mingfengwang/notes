## break语句解析  
有两种break  
第一种是代替原来有害的goto语句，这时的break只能存在于可中断语句(BreakableStatement)的内部，用于中断并“跳转到语句的结束位置”  
所谓“可中断语句”其实只有两种，包括全部的循环语句，以及 swtich 语句。在这两种语句内部使用的“break;”，采用的就是这种处理机制——中断当前语句，将执行逻辑交到下一语句。  
第二种是中断标签化语句  
```
break labelName;
```  
所谓标签化语句，就是在一般语句之前加上“xxx:”这样的标签，用以指示该语句。  
标签化语句理解的是“位置”，而不是语句在执行环境中的范围，所以他后面既可以跟块级作用域，也可以跟语句  
```    
aaa:{   
  ...  
}   
```  
```  
bbb: if(true){...}
```  

```
var i = 100;
function foo() { 
bbb: try { 
console.log("Hi"); 
return i++; // <-位置1：i++表达式将被执行 
} 
finally { 
break bbb; 
} 
console.log("Here"); 
return i; // <-位置2
}
```  
结果如下  
foo()  
Hi  
Here  
101  
    
位置1的return之后i++的语句被执行了，但是return没有被执行，break返回了语句的完成状态  
break 将“语句的‘代码块’”理解为位置，而不是理解为作用域 / 环境。JavaScript 设计了一种全新方法，用来清除这个跳转所带来的影响（也就是回收跳转之前的资源分配）。  
 
console.log(eval(`      
aaa: {  
1+2;   
bbb:    
 
 {   
   3+4;  
   break aaa;   
 }  
}  
`));  
返回值是7  
一个函数的调用：调用函数——执行函数体（EvaluateBody）并得到它的“完成”结果（result）。函数执行是求值，所以返回的是对该函数求值的结果（Result），该结果或是值（Value），或是结果的引用（Reference）。  
一个块语句的执行：执行块中的每行语句，得到它们的“完成”结果（result）。而语句是命令，语句执行的返回结果是该命令得以完成的状态（Completion, Completion Record Specification Type）。  
这些结果（result）包括的状态有五种，称为完成的类型：normal、break、continue、return、throw。  
#### JavaScript 是一门混合了函数式与命令式范型的语言，而这里对函数和语句的不同处理，正是两种语言范型根本上的不同抽象模型带来的差异。
当运行期出了一这个称为“中断（break）”的状态时，JavaScript 引擎需要找到这个“break”标示的目标位置（result.Target），然后与当前语句的标签（如果有的话）对比：  
如果一样，则取 break 源位置的语句执行结果为值（Value）并以正常完成状态返回；  
如果不一样，则继续返回 break 状态。
由于对“break 状态”的拦截交给语句退出（完成）之后的下一个语句，因此如果语句是嵌套的，那么其后续（也就是外层的）语句就可以得到处理这个“break 状态”的机会。  
break的类型必然是'break'，返回值必然是Empty。在顺序执行时，当语句返回 Empty 的时候，不会改写既有的其他语句的返回值。  





