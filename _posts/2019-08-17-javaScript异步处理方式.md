---
layout: post
title:  "JavaScript异步处理方式"
date:   2019-08-17 08:53:16
author: zhangtejun
categories: JavaScript
---
##### 异步处理的方式
在JavaScript中一般有3种处理异常的方式
* 回调（callback）
* 承诺（promise）
* 可观察对象（observable）

##### 回调
回调是一个函数(callback)被作为参数传递到另一个函数(f)里，待函数(f)执行完后再执函数(callback)。
```javaScript
function f(callback){
  // doSomething
  // ...
  callback();
}
function f1(){
  // doSomething
}
// 调用
f(f1);
```
缺点：不利于代码阅读和维护，函数作为参数层层嵌套（回调地狱），代码横向拉长。

##### Promise
Promise 是一个对象，它代表了一个异步操作的最终完成或者失败。本质上，Promise 是一个被某些函数传出的对象，
我们附加回调函数（callback）使用它，而不是将回调函数传入那些函数内部。
```
// 1. 传入回调函数的方式
doSomething(function(result) {
  doSomethingElse(result, function(newResult) {
    doThirdThing(newResult, function(finalResult) {
      console.log('Got the final result: ' + finalResult);
    }, failureCallback);
  }, failureCallback);
}, failureCallback);

// 2. 最新的方式就是返回一个 promise 对象，使得你可以将你的回调函数绑定在该 Promise 上，多重异步可以形成一个 Promise 链：
doSomething().then(function(result) {
  return doSomethingElse(result);
})
.then(function(newResult) {
  return doThirdThing(newResult);
})
.then(function(finalResult) {
  console.log('Got the final result: ' + finalResult);
})
.catch(failureCallback);
// 用箭头函数来表示如下：
doSomething()
.then(result => doSomethingElse(result))
.then(newResult => doThirdThing(newResult))
.then(finalResult => {
  console.log(`Got the final result: ${finalResult}`);
})
.catch(failureCallback);

// 在 ECMAScript 2017 标准的 async/await 语法糖中，这种同步形式代码的对称性得到了极致的体现：
async function foo() {
  try {
    let result = await doSomething();
    let newResult = await doSomethingElse(result);
    let finalResult = await doThirdThing(newResult);
    console.log(`Got the final result: ${finalResult}`);
  } catch(error) {
    failureCallback(error);
  }
}
```
1. 在`回调地狱`示例中，有3次`failureCallback`的调用，而在`Promise`链中只有尾部的一次调用。
2. `Promise`解决了回调地狱的基本缺陷。

* Promise.all() 和 Promise.race() 是并行运行异步操作的两个组合式工具。

```
// 1. 发起并行操作，然后等多个操作全部结束后进行下一步操作
Promise.all([func1(), func2(), func3()])
.then(([result1, result2, result3]) => { /* use result1, result2 and result3 */ });

// 2. 实现时序组合
[func1, func2, func3].reduce((p, f) => p.then(f), Promise.resolve())
.then(result3 => { /* use result3 */ });
// 3. 通常，递归调用一个由异步函数组成的数组时相当于一个 Promise 链
Promise.resolve().then(func1).then(func2).then(func3);
// 4. 可复用的函数形式
const applyAsync = (acc,val) => acc.then(val);
const composeAsync = (...funcs) => x => funcs.reduce(applyAsync, Promise.resolve(x));
// 5. composeAsync() 函数将会接受任意数量的函数作为其参数，并返回一个新的函数，该函数接受一个通过 composition pipeline 传入的初始值。
// 任一函数可以是异步或同步的，它们能被保证按顺序执行：
const transformData = composeAsync(func1, func2, func3);
const result3 = transformData(data);
// 6. 在 ECMAScript 2017 标准中, 时序组合可以通过使用 async/await 而变得更简单：
let result;
for (const f of [func1, func2, func3]) {
  result = await f(result);
}
```

`async function` 用来定义一个返回 AsyncFunction 对象的异步函数。异步函数是指通过事件循环异步执行的函数，它会通过一个隐式的 Promise 返回其结果。

```
function resolveAfter2Seconds() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('resolved');
    }, 2000);
  });
}

async function asyncCall() {
  console.log('calling');
  var result = await resolveAfter2Seconds();
  console.log(result);
  // expected output: 'resolved'
}
asyncCall()
```
##### 可观察对象
使用使用 RxJS
```
// 当订阅Observable 的时候会立即(同步地)推送值1、2、3，然后1秒后会推送值4，再然后是完成流：
var observable = Rx.Observable.create(function (observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  setTimeout(() => {
    observer.next(4);
    observer.complete();
  }, 1000);
});

console.log('just before subscribe');
// 要调用 Observable 并看到这些值，我们需要订阅 Observable：
observable.subscribe({
  next: x => console.log('got value ' + x),
  error: err => console.error('something wrong occurred: ' + err),
  complete: () => console.log('done'),
});
console.log('just after subscribe');
```

##### 迭代器
javascript1.7为for/in循环增加更多通用功能，它可以遍历任何可迭代（iterable）的对象。

迭代器是一个对象，这个对象允许对它的值集合进行遍历，并保持相应的状态，以便能跟踪到当前遍历的位置。

迭代器必须包含next方法，每一次调用该方法都返回集合中下一个值。

##### 生成器
关键字yield在函数内部使用，用法和return类似，返回函数中的一个值。

任何使用关键字yield的函数都称为生成器函数（generator function）,生成器函数通过yield返回值。

* 和return区别：使用yield的函数产生一个可保持函数内部状态的值，这个值是可以恢复的。
* 和普通函数调用的区别：生成器函数调用不是执行函数体，而是返回一个生成器对象。

生成器是一个对象，用来标识生成器函数的当前执行状态，它定义了一个next()方法，可恢复生成器函数的执行，直到遇到下一条yield语句。
这时生成器函数中的yield语句的返回值就是生成器的next()方法的返回值。如果生成器函数执行return或者到达函数体末尾，那么next()将抛出一个StopIteration。

调用close()方法可以终止生成器函数的执行。

##### Generator 函数
Generator函数在语法上可以把它理解成，Generator 函数是一个状态机，封装了多个内部状态。

执行 Generator 函数会返回一个遍历器对象，即Generator 函数除了状态机，还是一个遍历器
对象生成函数。返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。

特征：
* function关键字与函数名之间有一个星号
* 函数体内部使用yield关键字
* yield表达式只能用在 Generator 函数里面
* yield表达式本身没有返回值，或者说undefined
```
// 1. 定义了一个 Generator 函数,该函数有三个状态：first，second 和 return 语句（结束执行）
function* tGenerator() {
  yield 'first';
  yield 'second';
  return 'last';
}
// 2. 调用Generator函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象（遍历器对象[Iterator Object]）
var tg = tGenerator();
// 3. 必须调用遍历器对象的next方法，使得指针移向下一个状态，done属性是一个布尔值，表示是否遍历结束。
tg.next(); // { value: 'first', done: false }
tg.next(); // { value: 'second', done: false }
tg.next(); // { value: 'last', done: true }
tg.next(); // { value: undefined, done: true }
```
每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield表达式（或return语句）为止。
换言之，Generator 函数是分段执行的，yield表达式是暂停执行的标记，而next方法可以恢复执行。

```
// yield表达式如果用在另一个表达式之中，必须放在圆括号里面
function* demo() {
  console.log('Hello' + yield); // SyntaxError
  console.log('Hello' + yield 123); // SyntaxError

  console.log('Hello' + (yield)); // OK
  console.log('Hello' + (yield 123)); // OK
}
// yield表达式用作函数参数或放在赋值表达式的右边，可以不加括号
function* demo() {
  foo(yield 'a', yield 'b'); // OK
  let input = yield; // OK
}
```

**与 Iterator 接口的关系**

任意一个对象的Symbol.iterator方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。

由于 Generator 函数就是遍历器生成函数，因此可以把 Generator 赋值给对象的Symbol.iterator属性，从而使得该对象具有 Iterator 接口。
```
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

[...myIterable] // [1, 2, 3]

// gen是一个 Generator 函数，调用它会生成一个遍历器对象g。
// 它的Symbol.iterator属性，也是一个遍历器对象生成函数，执行后返回它自己
function* gen(){
  // some code
}
var g = gen();
g[Symbol.iterator]() === g
// true
```

**next 方法的参数**
yield表达式本身没有返回值（undefined）。next方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值。
```
function* f() {
  for(var i = 0; true; i++) {
    var reset = yield i; // 变量reset的值总是undefined。
    if(reset) { i = -1; } // 当next方法带一个参数true时，变量reset就被重置为这个参数（即true）
  }
}
var g = f();
g.next() // { value: 0, done: false }
g.next() // { value: 1, done: false }
g.next(true) // { value: 0, done: false }
```
意义: Generator 函数从暂停状态到恢复运行，它的上下文状态（context）是不变的。
通过next方法的参数，就有办法在 Generator 函数开始运行之后，继续向函数体内部注入值。
即可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而达到调整函数行为的效果。
```
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}
var a = foo(5);
a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}
a.next() // Object{value:NaN, done:true}
var b = foo(5);
b.next() // { value:6, done:false }
b.next(12) // { value:8, done:false }
b.next(13) // { value:42, done:true }
```
由于next方法的参数表示上一个yield表达式的返回值，所以在第一次使用next方法时，传递参数是无效的。

next()是将yield表达式替换成一个值,next(1)方法就相当于将yield表达式替换成一个值1。如果next方法没有参数，就相当于替换成undefined。
```
const g = function* (x, y) {
  let result = yield x + y;
  return result;
};

const gen = g(1, 2);
gen.next(); // Object {value: 3, done: false}
gen.next(1); // Object {value: 1, done: true}
// 相当于将 let result = yield x + y
// 替换成 let result = 1;
```

**for...of 循环**
`for...of`循环可以自动遍历 Generator 函数运行时生成的Iterator对象，且此时不再需要调用next方法。
```
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
  return 4;
};
for (let v of myIterable){
    console.log(v)
}
// 1 2 3
// next方法的返回对象的done属性为true，for...of循环就会中止。代码的return语句返回的4，不包括在for...of循环之中。
```

Generator 函数返回的遍历器对象，还有一个return方法，可以返回给定的值，并且终结遍历 Generator 函数。
```
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}
var g = gen();
g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()        // { value: undefined, done: true }
```

**`yield* `表达式**
如果在 Generator 函数内部，调用另一个 Generator 函数。需要在前者的函数体内部，自己手动完成遍历。
```
function* foo() {
  yield 'a';
  yield 'b';
}

function* bar() {
  yield 'A';
  // 需要手动遍历 foo()
  for (let i of foo()) {
    console.log(i);
  }
  yield 'B';
}

for (let v of bar()){
  console.log(v); // A a b B
}
```
ES6 提供了yield*表达式，作为解决办法，用来在一个 Generator 函数里面执行另一个 Generator 函数。
```
function* foo() {
  yield 'a';
  yield 'b';
}
function* bar() {
  yield 'A';
  yield* foo();
  yield 'B';
}

// 等同于
function* bar() {
  yield 'A';
  yield 'a';
  yield 'b';
  yield 'B';
}

// 等同于
function* bar() {
  yield 'A';
  for (let v of foo()) {
    yield v;
  }
  yield 'B';
}

for (let v of bar()){
  console.log(v); // A a b B
}
```
##### Generator 与状态机
Generator 是实现状态机的最佳结构。例如clock函数就是一个状态机。
clock函数一共有两种状态（Tick和Tock），每运行一次，就改变一次状态。
```
var ticking = true;
var clock = function() {
  if (ticking)
    console.log('Tick!');
  else
    console.log('Tock!');
  ticking = !ticking;
}
// Generator 实现
// 与 ES5 实现对比，可以看到少了用来保存状态的外部变量ticking
var clock = function* () {
  while (true) {
    console.log('Tick!');
    yield;
    console.log('Tock!');
    yield;
  }
};
```
##### Generator 与协程


##### Generator 与上下文
JavaScript 代码运行时，会产生一个全局的上下文环境（context，又称运行环境），包含了当前所有的变量和对象。
然后，执行函数（或块级代码）的时候，又会在当前上下文环境的上层，产生一个函数运行的上下文，变成当前（active）
的上下文，由此形成一个上下文环境的堆栈（context stack）。

这个堆栈是“后进先出”的数据结构，最后产生的上下文环境首先执行完成，退出堆栈，然后再执行完成它下层的上下文，直至所有代码执行完成，堆栈清空。

Generator 函数执行和以上方式不一样，它执行产生的上下文环境，一旦遇到yield命令，就会暂时退出堆栈，但是并不消失清空堆栈，
里面的所有变量和对象会冻结在当前状态。等到对它执行next命令时，这个上下文环境又会重新加入调用栈，冻结的变量和对象恢复执行。


#####  Generator 应用场景
特点：
* 可以暂停函数执行
* 可以返回任意表达式的值

>异步操作的同步化表达
>>Generator 函数的暂停执行的效果，意味着可以把异步操作写在yield表达式里面，等到调用next方法时再往后执行。
```
function* loadUI() {
  showLoadingScreen();
  yield loadUIDataAsynchronously();
  hideLoadingScreen();
}
var loader = loadUI();
// 加载UI
loader.next()

// 卸载UI
loader.next()

//2.
function* test() {
  var result = yield request(url);
  var resp = JSON.parse(result); //  result = response
    console.log(resp.value);
}

function request(url) {
  Ajax(url, function(response){
    it.next(response); // next方法，必须加上response参数，因为yield表达式，本身是没有值的，总是等于undefined
  });
}
var it = test();
it.next();
```
>控制流管理
>>如果有一个多步操作非常耗时,有以下绝决方案
```
// 1. 回调函数
step1(function (value1) {
  step2(value1, function(value2) {
    step3(value2, function(value3) {
      step4(value3, function(value4) {
        // do something with value4
      });
    });
  });
});
// 2. Promise
Promise.resolve(step1)
  .then(step2)
  .then(step3)
  .then(step4)
  .then(function (value4) {
    // do something with value4
  }, function (error) {
    // Handle any error from step1 through step4
  })
  .done();

// 3.1 Generator 函数
function* longRunningTask(value1) {
  try {
    var value2 = yield step1(value1);
    var value3 = yield step2(value2);
    var value4 = yield step3(value3);
    var value5 = yield step4(value4);
    // Do something with value4
  } catch (e) {
    // Handle any error from step1 through step4
  }
}
// 3.2 使用一个函数，按次序自动执行所有步骤
scheduler(longRunningTask(initialValue));
function scheduler(task) {
  // task都必须是同步的，不能有异步操作
  var taskObj = task.next(task.value);
  // 如果Generator函数未结束，就继续调用
  if (!taskObj.done) {
    task.value = taskObj.value
    scheduler(task);
  }
}
// 4. 控制流管理 for...of + yield
// 4.1 数组steps封装了一个任务的多个步骤
let steps = [step1Func, step2Func, step3Func];
function* iterateSteps(steps){
  for (var i=0; i< steps.length; i++){
    var step = steps[i];
    yield step();
  }
}
// 4.2 数组jobs封装了一个项目的多个任务
let jobs = [job1, job2, job3];
function* iterateJobs(jobs){
  for (var i=0; i< jobs.length; i++){
    var job = jobs[i];
    yield* iterateSteps(job.steps);
  }
}
// 4.3 for...of循环一次性依次执行所有任务的所有步骤
for (var step of iterateJobs(jobs)){
  console.log(step.id);
}

```
>部署 Iterator 接口
>>利用 Generator 函数，可以在任意对象上部署 Iterator 接口。
```
function* itEntries(obj) {
  let keys = Object.keys(obj);
  for (let i=0; i < keys.length; i++) {
    let key = keys[i];
    yield [key, obj[key]];
  }
}
let myObj = { foo: 3, bar: 7 };
for (let [key, value] of itEntries(myObj)) {
  console.log(key, value);
}
```
>作为数据结构
>>Generator 可以看作是数据结构,Generator 函数可以返回一系列的值，这意味着它可以对任意表达式，提供类似数组的接口。
```
function* doStuff() {
  yield fs.readFile.bind(null, 'hello.txt');
  yield fs.readFile.bind(null, 'world.txt');
  yield fs.readFile.bind(null, 'and-such.txt');
}
// 可以像处理数组一样，处理这三个返回的函数
for (task of doStuff()) {
  // task是一个函数
  // use task doSomething
}
```

#### 异步遍历器
next()方法返回的是一个 Promise 对象，这样就不行，不符合 Iterator 协议，只要代码里面包含异步操作都不行。
也就是说，Iterator 协议里面next()方法只能包含同步操作。

解决方法: 将异步操作包装成 Thunk 函数或者 Promise 对象，即next()方法返回值的value属性是一个 Thunk 函数或者 Promise 对象，
等待以后返回真正的值，而done属性则还是同步产生的。

```
function idMaker() {
  let index = 0;
  return {
    next: function() {
      return {
        value: new Promise(resolve => setTimeout(() => resolve(index++), 1000)),
        done: false
      };
    }
  };
}
const it = idMaker();
it.next().value.then(o => console.log(o)) // 1
it.next().value.then(o => console.log(o)) // 2
it.next().value.then(o => console.log(o)) // 3
```
ES2018 引入了“异步遍历器”（Async Iterator），为异步操作提供原生的遍历器接口，即value和done这两个属性都是异步产生。

特点: 是调用遍历器的next方法，返回的是一个 Promise 对象。一个对象的同步遍历器的接口，部署在Symbol.iterator属性上面。
同样地，对象的异步遍历器接口，部署在Symbol.asyncIterator属性上。
```
asyncIterator.next().then(({ value, done }) => {
    console.log(value, done)
}
```

```
// 1. 创建异步迭代器
const createAsyncIterator = items => {
    const keys = Object.keys(items)
    const len = keys.length
    let pointer = 0
    return {
        next() {
            const done = pointer >= len
            const value = !done ? items[keys[pointer++]] : undefined
            return Promise.resolve({
                value,
                done
            })
        }
    }
// 2.
const asyncIterable = createAsyncIterable(['a', 'b']);
const asyncIterator = asyncIterable[Symbol.asyncIterator]();
asyncIterator
.next()
.then(itResult1 => {
  console.log(itResult1); // { value: 'a', done: false }
  return asyncIterator.next();
})
.then(itResult2 => {
  console.log(itResult2); // { value: 'b', done: false }
  return asyncIterator.next();
})
.then(itResult3 => {
  console.log(itResult3); // { value: undefined, done: true }
});

// 由于异步遍历器的next方法，返回的是一个 Promise 对象。因此，可以把它放在await命令后面。
async function f() {
  const asyncIterable = createAsyncIterable(['a', 'b']);
  const asyncIterator = asyncIterable[Symbol.asyncIterator]();
  console.log(await asyncIterator.next());
  // { value: 'a', done: false }
  console.log(await asyncIterator.next());
  // { value: 'b', done: false }
  console.log(await asyncIterator.next());
  // { value: undefined, done: true }
}
// 异步遍历器的next方法是可以连续调用的，不必等到上一步产生的 Promise 对象resolve以后再调用
const asyncIterable = createAsyncIterable(['a', 'b']);
const asyncIterator = asyncIterable[Symbol.asyncIterator]();
const [{value: v1}, {value: v2}] = await Promise.all([
  asyncIterator.next(), asyncIterator.next()
]);

console.log(v1, v2); // a b
```
** for await...of**
for await...of循环用于遍历异步的 Iterator 接口。
```
async function f() {
  for await (const x of createAsyncIterable(['a', 'b'])) {
    console.log(x);
  }
}
```

##### 异步 Generator 函数
异步 Generator 函数就是async函数与 Generator 函数的结合。
```
async function* gen() {
  yield 'hello';
}
const genObj = gen();
genObj.next().then(x => console.log(x));
// { value: 'hello', done: false }

// 同步 Generator 函数
function* map(iterable, func) {
  const iter = iterable[Symbol.iterator]();
  while (true) {
    const {value, done} = iter.next();
    if (done) break;
    yield func(value);
  }
}

// 异步 Generator 函数
async function* map(iterable, func) {
  const iter = iterable[Symbol.asyncIterator]();
  while (true) {
    const {value, done} = await iter.next();
    if (done) break;
    yield func(value);
  }
}
```
##### 相等运算符用于比较两个值
```
// The comparison x == y, where x and y are values, produces true or false.

ReturnIfAbrupt(x). // 如果x不是正常值（比如抛出一个错误），中断执行
ReturnIfAbrupt(y).

// 如果Type(x)与Type(y)相同，执行严格相等运算x === y
If Type(x) is the same as Type(y), then Return the result of performing Strict Equality Comparison x === y.

// 如果x是null，y是undefined，返回true
If x is null and y is undefined, return true.

// 如果x是undefined，y是null，返回true。
If x is undefined and y is null, return true.

// 如果Type(x)是数值，Type(y)是字符串，返回x == ToNumber(y)的结果
If Type(x) is Number and Type(y) is String, return the result of the comparison x == ToNumber(y).

// 如果Type(x)是字符串，Type(y)是数值，返回ToNumber(x) == y的结果。
If Type(x) is String and Type(y) is Number, return the result of the comparison ToNumber(x) == y.

// 如果Type(x)是布尔值，返回ToNumber(x) == y的结果。
If Type(x) is Boolean, return the result of the comparison ToNumber(x) == y.

// 果Type(y)是布尔值，返回x == ToNumber(y)的结果。
If Type(y) is Boolean, return the result of the comparison x == ToNumber(y).

// 如果Type(x)是字符串或数值或Symbol值，Type(y)是对象，返回x == ToPrimitive(y)的结果。
If Type(x) is either String, Number, or Symbol and Type(y) is Object, then return the result of the comparison x == ToPrimitive(y).

// 如果Type(x)是对象，Type(y)是字符串或数值或Symbol值，返回ToPrimitive(x) == y的结果
If Type(x) is Object and Type(y) is either String, Number, or Symbol, then return the result of the comparison ToPrimitive(x) == y.

//返回false
Return false.
```
#### toPrimitive(input)
```
toPrimitive(input)
1. 如果input是原始值，直接返回这个值；
2. 否则，如果input是对象，调用input.valueOf()，如果结果是原始值，返回结果；
3. 否则，调用input.toString()。如果结果是原始值，返回结果；
4. 否则，抛出错误。
```

##### async 函数
ES2017 标准引入了 async 函数，使得异步操作变得更加方便。
它其实就是 Generator 函数的语法糖。
```
const fs = require('fs');

const readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) return reject(error);
      resolve(data);
    });
  });
};
// 1.
const gen = function* () {
  const f1 = yield readFile('/etc/fstab');
  const f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
// 2. 函数gen可以写成async函数
const asyncReadFile = async function () {
  const f1 = await readFile('/etc/fstab');
  const f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```
async函数就是将 Generator 函数的星号（*）替换成async，将yield替换成await。

**多个请求并发执行**

```
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = await Promise.all(promises);
  console.log(results);
}
// 或者使用下面的写法
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = [];
  for (let promise of promises) {
    results.push(await promise);
  }
  console.log(results);
}

// async 函数的实现原理，就是将 Generator 函数和自动执行器，包装在一个函数里
async function fn(args) {
  // ...
}
// 等同于
function fn(args) {
  return spawn(function* () { // spawn函数就是自动执行器
    // ...
  });
}
function spawn(genF) {
  return new Promise(function(resolve, reject) {
    var gen = genF();
    function step(nextF) {
      try {
        var next = nextF();
      } catch(e) {
        return reject(e);
      }
      if(next.done) {
        return resolve(next.value);
      }
      Promise.resolve(next.value).then(function(v) {
        step(function() { return gen.next(v); });
      }, function(e) {
        step(function() { return gen.throw(e); });
      });
    }
    step(function() { return gen.next(undefined); });
  });
}
```
