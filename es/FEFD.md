##函数声明  
```
function a() {}   
```
没有执行  
返回值为empty  
必须具名   
静态语义

##函数表达式
```
(function(){})
(function b(){})
```
有执行体  
返回值不为empty  
可以具名也可以不具名，但是都无法找到名字，因为它是闭包，无法作为export的对象   
动态语义

##函数定义
```
var/const/let a = function(){}
var/const/let a = function b (){}
```
是函数表达式的一层概念封装,在外层有名字  
它又是表达式，它的执行结果又是闭包，又是实例并且保留执行返回不为empty  
静态语义  
如果右侧为匿名，则外层名字为a，a.name因为function的名字缺省所以也为a  
如果右侧为具名，则外层名字为a，a.name为b

##export & import
export 只能导出有名字的，export会形成“名字表”，它的 
```
export default <expression> 
```
中default是特殊的名字  
在export的时候，所有的expression不执行  
import 是名字空间（名字空间是用户代码可以操作的组件，它映射自内部的模块导入名字表）  
如果不是import * as ... 该名字表只存在于词法分析过程中，无实例  
import的时候，esModule构建依赖树，并且生成名字表，  
然后执行模块最顶层代码，给名字(别名)绑定值，就出现了“变量提升”的效果  
这里的名字是迟绑定的，只是绑定的时候在执行用户代码之前  

    考虑TDZ(暂时性死区)和变量提升，有什么特点
