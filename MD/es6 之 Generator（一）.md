# es6 之 Generator（一）
> Generator 函数是 ES6 提供的一种异步编程解决方案，语法行为与传统函数完全不同

第一次接触这个函数的时候，第一印象是，这个单词咋读...
> generator 美\[ˈdʒɛnəˌretɚ]

然后是，这个函数是干啥的，好难理解啊，```function*```是啥啊，咋还有个```*```，咋看着像指针呢...看了好多次阮一峰老师的[ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs),总算有了一些理解。

#### 1.基本使用
来看一看基本使用吧（function后的*前后可以加空格，但是推荐下面的写法）
```javascript
    function* stateMachine() {
        yield 'hello';
        yield 'world';
        return 'ending';
        yield 'haha';
    }
```
以上函数定义了一个名为**stateMachine**的Generator函数，定义了两个内部状态，先产出'hello'，再产出'world'，最后执行'ending'。(return)后的(yield)不执行

来执行以下
```javascript
    const result = stateMachine();
    result.next(); // {value: "hello", done: false}
    result.next(); // {value: "world", done: false}
    result.next(); // {value: "ending", done: true}
    result.next(); // {value: undefined, done: true}

```
Generator内部封装了多个状态。程序员可以自主的选择"产出"状态的时机，这里的"产出"也就是(yield),
但Generator不能够无序的"产出"，此时(done)的值为(false)，只能够按照设定的顺序，依次输出，也不能够进行"回溯"，当函数走到终点，
只能执行(return)，(done)属性的值也为(true)。此时再执行next()，得到的value永远就是undefined了。(done)也为(true)。(done)表示的就是程序走到了尽头。

Generator函数返回的结果是一个*迭代器对象*，他是一个指向内部状态的指针对象。这和Java中的迭代器的使用非常类似。
执行next()方法进行遍历。每次执行，指针指向下一个状态，*yield*所标注的位置就是下一个*产出的位置*，函数执行完成后，next()的返回值永远就是```{value: undefined, done: true}```
#### 2.next 方法
next(n) n会被当做上一个*yield*表达式的返回值
这句话是什么意思，乍一看确实很难理解，来看一个例子
```javascript
    function* f() {
      for(var i = 0; true; i++) {
        var reset = yield i;
        if(reset) { i = -1; }
      }
    }

    var g = f();

    g.next() // { value: 0, done: false }
    g.next() // { value: 1, done: false }
    g.next(true) // { value: 0, done: false }
```
我们来仔细分析一下next方法的执行顺序：
1. 1.先找到函数中的第一个yield关键字，next方法即刻返回 输出```{ value: ..., done: ... }```（不管左侧是啥）
2. 2.返回完成后，yield 得到一个返回值，**该值为下一个next方法的参数**，默认为undefined，假如左侧是赋值语句，如上。第一个next()方法执行值时，rest的值都为undefined，所以下方if不通过。
**程序此时执行到右方yield处停止，左方赋值尚未执行。等到下一个next方法执行后，再得到返回值，执行左方赋值语句**(划重点)。这是Generator函数的执行机制
3. 3.但假如此时next方法得到一个参数(n)，该参数将作为刚刚暂停位置的yield的返回值。如上方代码，代码按照如下顺序：
   > 第三次next执行后，rest被置为true，进入下方if语句，i被置为-1，进入第三次循环，i++，yield输出i此时为(0)

再来看一个例子来巩固Generator函数的执行机制
```javascript
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
在这里再次强调yield的返回值和本身并无关系，他只是下一条next方法的参数。
如果你能够准确说出所有答案，那么就证明你能够较好的理解Generator的运行机制啦。

#### 3.利用next方法注入参数
了解了以上知识，我们就可以利用next方法向Generator函数中注入参数，输出我们想要的结果。
但是从上面的例子看来，*第一次执行next方法的参数是无效的*，这怎么办呢，来再看一个例子：
```javascript
    function wrapper(generatorFunction) {
      return function (...args) {
        let generatorObject = generatorFunction(...args);
        generatorObject.next();
        return generatorObject;
      };
    }

    const wrapped = wrapper(function* () {
      console.log(`First input: ${yield}`);
      return 'DONE';
    });

    wrapped().next('hello!')
    // First input: hello
```
哈哈，就是利用高阶函数，偷偷的先执行一次next方法，再返回这个"用过的"的迭代器对象，好让我们认为自己是"第一次"执行next方法就得到的输出结果

### 2.利用for...of遍历
在前面我们介绍过，for...of 是一个神奇的方法，可以遍历各种数据结构（除了对象）。既然Generator函数的返回值是一个迭代器对象，那当然可以用for...of来对他进行遍历了。
```javascript
    function *foo() {
      yield 1;
      yield 2;
      yield 3;
      yield 4;
      yield 5;
      return 6;
    }

    for (let k of foo()) {
      console.log(k);
    }
    // 1, 2, 3, 4, 5 没有6
```
for...of 会在每个yield处进行返回。其实他是找到所有(done)为(false)的结果，并把(value)进行输出
来感受一下用Generator实现的斐波那契数列
```javascript
    function* fibonacci() {
      let [prev, curr] = [0, 1];
      for (;;) {
        [prev, curr] = [curr, prev + curr];
        yield curr;
      }
    }

    for (let n of fibonacci()) {
      if (n > 1000) break;
      console.log(n);
    }
```
### 3.Generator.prototype.throw()
Generator 函数返回的遍历器对象，都有一个throw方法，可以在函数体外抛出错误，然后在 Generator 函数体内捕获。
```javascript
    var g = function* () {
      try {
        yield;
      } catch (e) {
        console.log('内部捕获', e);
      }
    };

    var i = g();
    i.next();

    try {
      i.throw('a');
      i.throw('b');
    } catch (e) {
      console.log('外部捕获', e);
    }
    // 内部捕获 a
    // 外部捕获 b
```
throw方法的参数直接传到Generator内部的catch中的e处

...在此先占一个空
### 4.Generator.prototype.return()
Generator 函数返回的遍历器对象，还有一个return方法，可以返回给定的值，并且终结遍历 Generator 函数。
```javascript
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
### 5.yield*
如果在 Generator 函数内部，调用另一个 Generator 函数，默认情况下是没有效果的。
```javascript
    function* foo() {
      yield 'a';
      yield 'b';
    }

    function* bar() {
      yield 'x';
      foo();
      yield 'y';
    }

    for (let v of bar()){
      console.log(v);
    }
    // "x"
    // "y"
```
上面代码中，foo和bar都是 Generator 函数，在bar里面调用foo，是不会有效果的。

这个就需要用到```yield*```表达式，用来在一个 Generator 函数里面执行另一个 Generator 函数。
需要写成
```javascript
   function* bar() {
     yield 'x';
     yield* foo();
     yield 'y';
   }
   for (let v of bar()){
     console.log(v);
   }
   // 'x'
   // 'a'
   // 'b'
   // 'y'
```

yield*的本质其实就是对一个迭代器对象进行for...of遍历
```javascript
    function* concat(iter1, iter2) {
      yield* iter1;
      yield* iter2;
    }

    // 等同于

    function* concat(iter1, iter2) {
      for (var value of iter1) {
        yield value;
      }
      for (var value of iter2) {
        yield value;
      }
    }
```
当目标函数中含有return时，yield*的返回值就为return的返回值。
### 6.Generator函数的this
Generator 函数总是返回一个遍历器，ES6 规定这个遍历器是 Generator 函数的实例，也继承了 Generator 函数的prototype对象上的方法。
```javascript
    function* g() {}

    g.prototype.hello = function () {
      return 'hi!';
    };

    let obj = g();

    obj instanceof g // true
    obj.hello() // 'hi!'
```
上面代码表明，Generator 函数g返回的遍历器obj，是g的实例，而且继承了g.prototype。但是，如果把g当作普通的构造函数，并不会生效，因为g返回的总是遍历器对象，而不是this对象。
而且Generator函数也不能当做构造函数，也就是不能和(new)中一起使用
```javascript
    function* g() {
      this.a = 11;
    }

    let obj = g();
    obj.a // undefined
```
*Generator函数中的this指向window*

...占空
#### 7.Generator是一个状态机
Generator 是实现状态机的最佳结构。比如，下面的clock函数就是一个状态机。
```javascript
    var ticking = true;
    var clock = function() {
      if (ticking)
        console.log('Tick!');
      else
        console.log('Tock!');
      ticking = !ticking;
    }
```
上面代码的clock函数一共有两种状态（Tick和Tock），每运行一次，就改变一次状态。这个函数如果用 Generator 实现，就是下面这样。
```javascript
    var clock = function* () {
      while (true) {
        console.log('Tick!');
        yield;
        console.log('Tock!');
        yield;
      }
    };
```
上面的 Generator 实现与 ES5 实现对比，可以看到少了用来保存状态的外部变量ticking，这样就更简洁，更安全（状态不会被非法篡改）、更符合函数式编程的思想，在写法上也更优雅。Generator 之所以可以不用外部变量保存状态，是因为它本身就包含了一个状态信息，即目前是否处于暂停态。
#### 8.异步操作中的应用
Generator的一个重要的实际意义就是处理异步操作，改写回调函数
```javascript
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
```
第一次执行loader.next()，显示loading遮罩，并加载数据，第二次执行loader.next()，隐藏loading遮罩
如上代码，将所有loading页面需要的逻辑都封装在一个函数中，省去了回环曲折的回调函数，逻辑非常清楚。
