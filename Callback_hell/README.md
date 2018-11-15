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
在很多时候，eventProxy完全可以满足我们的需求，它能够很好的帮助我们解决回调地狱问题，但是在有些方面他还是不够好，比如错误处理，比如多异步同时处理等等。我们可以通过扩展我们的eventProxy库来得到更好的编程体验，但是还是不够直观。幸好，我们有promise.
下面我们以网络请求为例，来看看promise如何强大。
```
var http = require('http')
const baseUrl = "http://httpbin.org/get"
http.get(baseUrl+"?title=1", function(res){
  res.setEncoding('utf8');
  res.on('data', function(chunk){
    console.log(chunk)
  })
  res.on('error', function(e){
    console.log(e)
  })
})
```
上面是一段基本的get请求，在回调函数中，我们获得了请求的返回结果。下面我们用promise来改造它。
```
function loadData(str){
  return new Promise(function(resolve, reject){
    http.get(baseUrl+str, function(res){
      res.setEncoding('utf8');
      res.on('data', function(chunk){
        resolve(chunk)
      })
      res.on('error', function(e){
        console.log(e)
      })
    })
  })
}

loadData("?title=1").then(function(response){
  console.log(response)
}).catch(function(e){
  console.log(e)
})
```
我们创建了一个loadData函数，而这个函数返回了一个promise对象，promise对象有两个方法，分别是catch和then。then给promise对象提供resolve函数，当promise执行的结果正确返回时执行resolve;catch给promise对象提供reject函数，当promise执行结果没有正确返回时执行reject。
但是，看不出这样写有什么牛逼的地方啊。现在，假设我们要请求三次网络，并且，必须要有顺序的处理。如果用最原始的回调函数写，我们至少需要三层嵌套。但是现在，我们有了promise
```
loadData("?title=1").then(function(response){
  console.log(response)
  return loadData("?title=2")
}).then(function(response){
  console.log(response)
  return loadData("?title=3")
}).then(function(response){
  console.log(response)
}).catch(e){
  //handle error
}
```
简直太简单，太直观，太优雅了有木有，而且，错误处理被集中了起来，砍掉一堆代码有木有。

promise.all
有时候，我们同时发出几个网络请求，但是对返回顺序并没有要求，我们可以用promise的all方法，让我们写出更美的代码
```
Promise.all(
  [loadData("?title=1"),
   loadData("?title=2"),
   loadData("?title=3")
 ]).then(function(responses){
   console.log(responses[0]);
   console.log(responses[1]);
   console.log(responses[2]);
 })
```
参考
--
[【1】为什么我们需要promise](https://www.jianshu.com/p/b68029f80c83)
[【2】避免Node.js中回调地狱](https://www.cnblogs.com/greatluoluo/p/6288931.html)
