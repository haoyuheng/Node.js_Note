Node.js 回调地狱 解决方法：Eventproxy Vs Promise
====
异步编程给我们带来了非常多的好处，尤其是在程序性能方面，但同时也带来了一些编程上的困难：回调地狱（callback hell）。

(1)回调地狱
--
为了解决这个阻塞问题，JavaScript严重依赖于回调，这是在长时间运行的进程（IO，定时器等）完成后运行的函数，因此允许代码执行经过长时间运行的任务。
```js
asyncProcess(args, function(){
  ...
}
```
但是，问题来了，回调地狱虽然回调的概念在理论上是巨大的，但它可能导致一些真正令人困惑和难以阅读的代码。 想象一下，如果你需要在回调后进行回调：
```js
asyncProcess1(args1, function(){
  asyncProcess2(args2, function(){
    asyncProcess3(args3, function(){
      ...
    })
  })
})
```
你可以看到，这真的是一发不可收拾。 抛出一些if语句，for循环，函数调用或注释，你会有一些非常难读的代码。 初学者特别是这个的受害者，不理解如何避免这个“金字塔的厄运”。

这种层层嵌套的代码给开发带来了很多问题，主要体现在：

1.代码可能性变差<br>
2.调试困难<br>
3.出现异常后难以排查

(2)EventProxy
--
关于EventProxy详解见[《EventProxy详细讲解》](https://github.com/JacksonTian/eventproxy),
回到上文中的回调地狱的例子，使用EventProxy解决为:
```
asyncProcess1(args1, function(){ep.emit('get1')})
ep.on('get1', function(){
  asyncProcess2(args2, function(){ep.emit('get2')})
})
ep.on('get2', function(){
  asyncProcess3(args3, function(){ep.emit('get3')})
})
ep.on('get3', function(){
  ...
})
```
看上去优雅了很多吧。上面的代码给我们一些启示，我们可以把io操作和io操作结束后的后续动作分开来，通过事件绑定的方法来处理异步io，当io操作返回的同时，发出事件并执行回调。

(3)Promise
--
