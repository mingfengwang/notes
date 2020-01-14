angular 1.x promise render lint

```js
 $_postJson(`/iim/bps/process/queryBackNodes`, {
   id: MainOptions.businessNo
 }, res => {
   MainOptions.searchIng = false
   if (res.success) {
     MainData.backNodes = res.data
   } else {
     $_notify(res.errorMsg, true)
   }
 })
```

一个promise 返回后$scope的值变了但是页面并没有重新渲染

这和浏览器解析渲染有关 先理解microtask & macrotask

- **macrotasks**: `setTimeout`, `setInterval`, `setImmediate`, I/O, UI rendering

- **microtasks**: `process.nextTick`, `Promises`, `MutationObserver`     

   

**一般情况下，macrotask queues 我们会直接称为 task queues，只有 microtask queues 才会特别指明。**

那么也就是说如果我的某个 microtask 任务又推入了一个任务进入 microtasks 队列，那么在主线程完成该任务之后，仍然会继续运行 microtasks 任务直到任务队列耗尽。

而事件循环每次只会入栈一个 macrotask ，主线程执行完该任务后又会先检查 microtasks 队列并完成里面的所有任务后再执行 macrotask



如何理解angular中的这个问题

首先顺序解析代码 顺序执行 包括jsdiff 和 batch UI update($apply  ->   digest) 其中的异步放到下一个tasks里 其中的promise相关放入到microtasks中 

顺序代码执行完 开始执行microtasks中的代码这时候有2种情况 apply执行完了和apply没执行完 这就是为什么 有时候调用apply就搞定了 有时候调用了apply报错 digest还在执行中 所以建议是最好避免使用apply

可以用angular自带的 $ timeout 或者 lodash的 _.defer (本质是1毫秒的setTimeout)

angular1不是vue vue是用MutationObserver 做批量处理的(https://www.zhihu.com/question/55364497/answer/144215284) 所以他在microtask做了页面重新渲染！直接提高了效率 而angular1直接走了html standard 每个task运行后 UI才会重新渲染 这就是为什么promise之后没有重新渲染而要用其他方式把任务重新压到tasks栈里去触发渲染