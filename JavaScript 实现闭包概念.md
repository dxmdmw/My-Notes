## JavaScript 中的闭包概念
### 函数作为返回值
高阶函数除了可以接受函数作为参数外，还可以把函数作为结果值返回。  
要实现对一个 Array 的求和，通常写法如下：
```js
function sum(arr) {
    return arr.reduce(function (x, y) {
        return x + y;
    });
}

sum([1, 2, 3, 4, 5]); // 15
```
但是，如果不需要立即求和，而是在后面的代码中，根据需要再计算怎么办？可以不返回求和的结果，而是返回求和的函数。  
```js
function lazy_sum(arr) {
    var sum = function () {
        return arr.reduce(function (x, y) {
            return x + y;
        });
    }
    return sum;
}
```
当我们调用 lazy_sum 时，返回的并不是求和结果，而是求和函数。  
```js
var f = lazy_sum([1, 2, 3, 4, 5]); // function sum()
```
调用函数 `f` 时，才真正计算求和的结果：
```js
f(); // 15
```
在这个例子中，我们在函数 `lazy_sum` 中又定义了函数 `sum` ，并且内部函数 `sum` 可以引用外部函数 `lazy_sum` 的参数和局部变量，当 `lazy_sum` 返回函数 `sum` 时，相关参数和变量都保存在返回的函数中，这种称为「闭包（Closure）」的程序结构拥有极大的威力。  
请再注意一点，当我们调用 `lazy_sum()` 时，每次调用都会返回一个新的函数，即使传入相同的参数：
```js
var f1 = lazy_sum([1, 2, 3, 4, 5]);
var f2 = lazy_sum([1, 2, 3, 4, 5]);
f1 === f2; // false
```
`f1()` 和 `f2()` 的调用结果互不影响。  

### 闭包
注意到返回的函数在其定义内部引用了局部变量 `arr`，所以，当一个函数返回了另一个函数后，其内部的局部变量还被新函数引用，所以闭包用起来简单，实现起来可不容易。  
另一个需要注意的问题是，返回的函数并没有立刻执行，而是直到调用了 `f()` 才执行。我们来看一个例子：  
```js
function count() {
    var arr = [];
    for (var i=1; i<=3; i++) {
        arr.push(function () {
            return i * i;
        });
    }
    return arr;
}

var results = count();
var f1 = results[0];
var f2 = results[1];
var f3 = results[2];
```
在上面的例子中，每次循环，都创建了一个新的函数，然后把创建的三个函数都添加到一个 `Array` 中返回了。  
你可能认为调用 `f1()`，`f2()`，`f3()` 结果应该是 `1`，`4`，`9`，但实际结果是：
```js
f1(); // 16 
f2(); // 16 
f3(); // 16 
```
全部都是 `16`！原因就在于返回的函数引用了变量 `i`，但它并非立即执行。等到 3 个函数都返回时，它们所引用的变量 `i` 已经变成了 `4`，因此最终结果为 `16`。  
返回闭包时牢记的一点就是：返回函数不要引用任何循环变量，或者后续会发生变化的变量。  
如果一定要引用循环变量怎么办？方法是再创建一个函数，用该函数的参数绑定循环变量当前的值，无论该循环变量后续如何更改，已绑定到函数参数的值不变。  
```js
function count() {
    var arr = [];
    for (var i=1; i<=3; i++) {
        arr.push((function (n) {
            return function () {
                return n * n;
            }
        })(i));
    }
    return arr;
}

var results = count();
var f1 = results[0]; 
var f2 = results[1]; 
var f3 = results[2]; 

f1(); // 1
f2(); // 4
f3(); // 9
```
注意这里用了一个「创建一个匿名函数并立即执行」的语法：
```js
(function (x) {
    return x * x;
})(3); // 9
```
当然，闭包不仅仅是可以返回一个函数以延迟执行。举例：  
在面向对象的程序设计语言里，比如 Java 和 C++，要在对象内部封装一个私有变量，可以用 `private` 修饰一个成员变量。  
在没有 `class` 机制，只有函数的语言里，借助闭包，同样可以封装一个私有变量。我们用 JavaScript 创建一个计数器：
```js
'use strict';

function create_counter() {
    var x = initial || 0;
    return {
        inc: function () {
            x += 1;
            return x;
        }
    }
}
```
它用起来像这样：
```js
var c1 = create_counter();
c1.inc(); // 1
c1.inc(); // 2
c1.inc(); // 3

var c2 = create_counter(10);
c2.inc(); // 11
c2.inc(); // 12
c2.inc(); // 13
```
在返回的对象中，实现了一个闭包，该闭包携带了局部变量 `x`，并且，从外部代码根本无法访问到变量 `x`。换句话说，闭包就是携带状态的函数，并且它的状态可以完全对外隐藏起来。  
闭包还可以把多参数的函数变成单参数的函数。例如，要计算 x<sup>y</sup> 可以用 `Math.pow(x, y)` 函数，不过考虑到经常计算 x<sup>2</sup> 或 x<sup>3</sup>，我们可以利用闭包创建新的函数 `pow2` 和 `pow3`。  
```js
'use strict';

function make_pow(n) {
    return function (x) {
        return Math.pow(x, n);
    }
}

// 创建两个新函数：
var pow2 = make_pow(2);
var pow3 = make_pow(3);

console.log(pow2(5)); // 25
console.log(pow3(7)); // 343
```

## JavaScript 中的 generator
我们先复习函数的概念。一个函数是一段完整的代码，调用一个函数就是传入参数，然后返回结果：
```js
function foo(x) {
    return x + x;
}

var r = foo(1); // 调用foo函数
```
函数在执行过程中，如果没有遇到 `return` 语句（函数末尾如果没有 `return`，就是隐含的 `return undefined;`），控制权无法交回被调用的代码。  
`generator` 跟函数很像，定义如下：  
```js
function* foo(x) {
    yield x + 1;
    yield x + 2;
    return x + 3;
}
```
`generator` 与函数不同的是，`generator` 由 `function*` 定义，并且除了 `return` 语句，还可以用 `yield` 返回多次。  
我们以斐波那契数列为例，它以 `0`，`1` 开头：  
```
0 1 1 2 3 5 8 13 21 34 ...
```
要编写一个产生斐波那契数列的函数，可以这么写：
```js
function fib(max) {
    var 
        t,
        a = 0,
        b = 1,
        arr = [0, 1];
    while (arr.length < max) {
        [a, b] = [b, a+b];
        arr.push(b);
    }
    return arr; 
}
```
函数只能返回一次，所以只能返回一个 `Array`。但是如果换成 generator，就可以一次返回一个数，不断返回多次。用 generator 改写如下：  
```js
function fib(max) {
    var 
        t,
        a = 0,
        b = 1,
        n = 0;
    while (n < max) {
        yield a;
        [a, b] = [b, a+b];
        n++;
    }
    return;
}
```
直接调用会返回一个 generator：
```js
fib(5); // fib {[[GeneratorStatus]]: "suspended", [[GeneratorReceiver]]: Window}
```
调用 generator 对象有两个方法，一个是不断调用 generator 对象的 `next()` 方法：
```js
var f = fib(5);
f.next(); // {value: 0, done: false}
f.next(); // {value: 1, done: false}
f.next(); // {value: 1, done: false}
f.next(); // {value: 2, done: false}
f.next(); // {value: 3, done: false}
f.next(); // {value: undefined, done: true}
```
`next()` 方法会执行 generator 的代码，然后，每次遇到 `yield x;` 就返回一个对象 `{value: x, done: true/false}`，然后“暂停”。返回的 `value` 就是 `yield` 的返回值，`done` 表示这个 generator 是否已经执行结束了。如果 `done` 为 `true`，则 `value` 就是 `return` 的返回值。  
第二种方法就是直接用 `for ... of` 循环迭代 generator 对象，这种方式不需要我们自己判断 `done`：
```js
for (var x of fib(10)) {
    console.log(x); // 依次输出 0, 1, 1, 2, 3, ...
}
```
