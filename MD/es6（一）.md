# es6（一）
> ECMAScript 6.0（以下简称 ES6）是 JavaScript 语言的下一代标准，已经在2015年6月正式发布了。它的目标，是使得 JavaScript 语言可以用来编写复杂的大型应用程序，成为企业级开发语言。

* es6在2017年可谓是大火，被越来越多的人频频提及，在React、ReactNative开发中广泛使用。javascript在es6的推出后，是越来越健全，真不能认为它是一个"玩具语言"啦。haha
较为深入的学习es6是非常有必要的，我总结了一些es6相关的语法特性及较为详细的使用说明，其中还有一些学习心得，希望可以给大家带来帮助。


### 什么是ES6 ？
    es6的全名是ECMAscript 2015

### es6有哪些新特性?
### 一、let
#### 1. 没有变量提升
```javascript
    console.log(a);
    let a = 4;
    // a is not defined
```
#### 2. 不能重复申明
```javascript
    let a = 4;
    let a = 5;
    console.log(a);
    // Identifier 'a' has already been declared
```
#### 3. 临时失效区（暂时性死区）
```javascript
    var a = 5;
    function fn(){
        console.log(a);
        let a = 4;
    }
    fn();
    // a is not defined
```
#### 4. 具有块级作用域（由花括号包裹的区域）
>   同样具有块级作用域的还有const。

来看一个例子
```javascript
    var a = 5;
    function foo(){
        console.log(a); // undefined
        if(false){
            var a = 6;
        }
    }
```

这显然不是我们想要的结果，因为
>   js在es6之前没有块级作用域

```javascript
    for(let i = 0; i < 3; i++){
        console.log(i);
    }
    console.log(i);
    // 0 1 2 i is not defined
```
```javascript
    {
        let a = 10;
    }
    console.log(a);
    // a is not defined
```
老生常谈的闭包问题 实质上就是为了解决没有块级作用域带来的问题
> ## 闭包
    * 外部函数（作用域）中有内部函数（作用域）
    * 内部函数调用了外部函数的局部变量
    * 外部函数执行完后，因为内部函数还在使用该局部变量，所以该局部变量不被释放，保证内部函数正常使用
    * 闭包函数（立即执行函数）为
        (function(){
        //  ...函数体
        })();

```javascript
    var aLi = document.getElementsByTagName('li');// len = n
    for(var i = 0; i < aLi.length; i++){
        aLi[i].onclick = function(){
            console.log(i);
        }
    }
    // n......n
```
```javascript
    var aLi = document.getElementsByTagName('li');// len = n
    for(var i = 0; i < aLi.length; i++){
        (function(i){
            aLi[i].onclick = function(){
                console.log(i);
            }
        })(i)
    }
    // 0、1、2....n
```
- for循环实质可以理解为 由n个"{}"构成的，在每个花括号中，let的作用域都是独立的，所以可以拿到每个i值。
- 但对于var来说实质是一个作用域，所以无法保留每个i的值。
### 二、const
在es6之前，如果我们想定义常量，通常由程序员自己**约定**
```javascript
    var NUM = 100;
```
将所有字母都大写的变量**约定**为常量，但本质还是变量
#### 1.const定义只是地址不能修改
```javascript
    const NUM = 100;
    NUM = 200;
    // Assignment to constant variable.
```
由const定义的常量，无法修改其**地址值**。  
在这里为啥要强调是**地址值**  
因为 以下代码在**非严格模式下**是可以通过的
```javascript
    const OBJ = {a: 100};
    OBJ.a = 200;
    console.log(OBJ.A);
    // 200
```
这时候会发现这很坑啊！！！  
怎么办呢，如果我们想要定义一个常量对象呢
```javascript
    const OBJ = {a: 100};
    Object.freeze(OBJ);
    OBJ.a = 200;
    console.log(OBJ.a);
    // 100
```
这样就可以了..
#### 2.没有变量提升
#### 3.块级作用域
#### *let 和 const 声明的变量不挂在window上*
### 三、class 类
在es6之前，我们要定义一个类，只能通过function来模拟
```javaScript
    function Person(){
        this.name = 'xxx';
    }
    Person.prototype.say = function(){

    }
    var person = new Person();
```
> 问：将属性放在原型中会怎么样？
    如 ```Person.prototype.name = 'xxx';```  
  答：对于普通数据类型，没有问题，但对于引用类型，对一个对象的修改，会影响所有继承该类的对象

#### 1.语法
```javaScript
    class Person{
        constructor(name){
            this.name = name;
        }
        say() {
            console.log(this.name);
        }
    }
    var person = new Person();
    person.say();
```
#### 2.继承的实现
```javaScript
    class Man extends Person{
        constructor(name, food){
            super(name, food); // 必须放在构造器的第一行，代表调用父类的构造方法
            this.food = food;
        }
        eat(){
            console.log(this.food);
        }
    }
    var man = new Man();
    man.say();
```
#### 3.static 静态方法/变量
```javaScript
    class Person{
        constructor(){
        }
        static say(){
            console.log(123); // 可以认为这是一个静态方法
        }
        static name(){
            return 'xxx'; // 可以认为这是一个静态变量
        }
    }
    Person.say();
```
在es6中，规定Class 内部只有静态方法， 没有静态属性  
以下写法都无效
```javascript
    class Person{
        constructor(){

        }
        name: 2
        static name: 2
    }
    console.log(Person.name)
    // undefined
```

### 四、Set 集合
Set是一个不能有重复元素的集合，重复添加无效
```javascript
    var s = new Set();
    s.add(1);
    s.add(2);
    // s.delete(2)  删除
    // s.clear() 清空
    // s.size() 长度
```
#### 1.遍历Set
```javascript
    var keys = s.keys(); // 返回一个迭代器
    for(var k of keys){
        console.log(k);
    }

    var values = s.values();
    for(var v of values){
        console.log(v);
    }

    var entries = s.entries(); // 将key和value 组合成一个数组返回
    for(var e of entries){
        console.log(e);
    }
```
如果我们想在es6之前使用给数组去重
#### 2.Array数组去重问题
##### 方法一：
```javascript
    var arr = [1,2,3,4,1,1,1];
    function fn(arr){
        var map = {};
        var newArr = arr.filter(function(item, index){
            if(!map[item]){
                map[item] = true;
                return item;
            }
        });
        return newArr;
    }
    fn(arr);
```
##### 方法二：
```javascript
    var arr = [1,2,3,4,1,1,1];
    function fn(arr){
        var newArr = [];
        for(var i = 0; i < arr.length; i++){
            if(newArr.indexOf(arr[i]) === -1){
                newArr.push(arr[i]);
            }
        }
        return newArr;
    }
    fn(arr);
```
##### 利用Array.from的方法三：
```javascript
    var arr = [1,2,3,4,1,1,1];
    function fn(arr){
        var s = new Set(arr);
        return Array.from(s);
    }
    fn(arr);
```
#### 3.Array返回只出现一次的元素
```javascript
    var arr = [1,2,3,4,1,1,1];
    function fn(arr){
        var newArr = [];
        for(var i = 0; i < arr.length; i++){
            if(arr.indexOf(arr[i]) === arr.lastIndexOf(arr[i])){
                newArr.push(arr[i]);
            }
        }
        return newArr;
    }
    fn(arr);
```
### 五、Map（键值对）
```javascript
    var m = new Map();
    m.set('a',1);
    m.set('b',2);
    m.get('a'); // 1

    // m.delete('a')
```
对象的属性名 认为是字符串，但Map的键 可以是所有**数据类型**
>```forEach```可以遍历Array、Set、Map。  
>```for of```被专门用来遍历迭代器。  

### 六、arrow 箭头函数
终于来到了箭头函数的环节```()=>{}```
这是一种特别简洁、优雅的函数写法
他可以这么写
```javascript
    var fn = (item, index) => {console.log(item)}
```
也可以这么写
```javascript
    var fn = item => {console.log(item)}
```
还可以这么写
```javascript
    var fn = item => (item)
    fn(2) // 2
```
还可以更简洁
```javascript
    var fn = item => item
    fn(2) // 2
```

>```=>```后使用小括号```()``` 表示将结果作为返回值，单行结果时还可以省略
>当参数唯一时，还可以将前面的```()``` 省略

但是他失去了一些东西...
```javascript
    var fn = () => {
        console.log(arguments);
    }
    fn(2) // arguments is not defined
```
解决方法如下：
```javascript
    var fn = (...arg) => {
        console.log(arg);
    }
    fn(2) // [2]
```

箭头函数不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。

>```...```称为位扩展运算符，在下面会详细介绍
>但箭头函数最厉害的地方还不在此

#### 1.改变默认的this指向
箭头函数能够将函数外面的this指向同步到函数内部，比如说这样：
```javascript
    var aLi = document.getElementsByTagName('li');// len = n
    for(var i = 0; i < aLi.length; i++){
        aLi[i].onclick = function(){
            setTimeout(function(){
                console.log(this.innerHTML);
                // 此时无法获取，因为this指向调用他的对象，而setTimeout为window下的方法，此时this指向window
            })
        }
    }
```
将function改为箭头函数就能够正确运行
```javascript
    var obj = {
        a: 1,
        say: function(){
            console.log(this.a);
        }
    }
    obj.say() // 1
```
上面乍一看这个挺正常的结果呀... 但是如果改成箭头函数
```javascript
    var obj = {
        a: 1,
        say: () => {
            console.log(this.a);
        }
    }
    obj.say() // undefined
```
所以箭头函数虽好，也要看清场合使用啊。