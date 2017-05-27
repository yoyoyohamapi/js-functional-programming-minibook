# λ 表达式

## ES6 的箭头函数

在 ES5 时代，我们需要用 `function` 来表达一个匿名函数：

```js
[1, 2, 3, 4].map(function (element) {
    return element * 2;
});
// => [2, 4, 6, 8]
```

而 ES6 为我们提供了 λ 表达式来定义匿名函数，形式上看，它是一个箭头函数：

```js
[1, 2, 3, 4].map(element => element * 2);
// => [2, 4, 6, 8]
```

## λ 表达式组成

λ 表达式源于 [λ 演算](https://zh.wikipedia.org/wiki/%CE%9B%E6%BC%94%E7%AE%97)。在 λ 演算中，每个表达式都代表一个 **一元函数**，并且只返回一个值。假定我们有一个线性函数：

$$

f(x) = x + 2

$$

用 λ 表达式就可以定义为：

```
λx.x + 2
```

λ 表达式由下面三个部分组成：

- 变量（Variables），需要声明函数需要的变量 `x`
- 抽象（Abstraction），对应到匿名函数 `λx.x + 2`
- 应用（Application），对应到匿名函数的调用，应用过程是左结合的 `(λx.x + 2) 5`

## 自由变量

自由变量指未被绑定在任何一个 λ 上的变量，如公式 `λx.x + y` 中，`y` 没有与 `λ` 绑定，所以 `y` 就是一个自由变量。
对于自由变量我们可以通过宏进行赋值，比如：

```
define y 3
(λx.x + y) 2
=> (2 + 3)
=> 5
```

## α-变换

**α-变换** 表示被绑定变量命名是可以被替换的，比如：

```
λx.x == λy.y
```

其一般形式为：

```
λV.E == λW.E[V:=W]
```

## β-归约

**β-归约** 表示对函数应用的化简，比如：

```
(λx.x+2) 3 == (3 + 2) == 5
```

```
((λV.E) E') == E [V:=E'] if free(E') subset free(E[V:E'])
```

注意后面的归约条件，它表示我们必须在保证自由变量不会与绑定变量起冲突的前提下才能进行归约操作，即保证 `E'` 表达式中的自由变量在归约后不会变为 `E[V:E']` 表达式的绑定变量。
我们举个例子来形象的说明这一点，假设存在表达式：

```
λz.λx.z + x
```

现在我们将应用它：

```
(λz.λx.z + x) (x + 2)
```

若直接按 β-归约 化简则为：

```
(λx.x + 2 + x)
```

于是原本 `x + 2` 中的自由变量 `x` 变成了绑定变量，如果进一步应用函数：

```
(λx.x + 2 + x) 3
```

则其最后变为：

```
3 + 2 + 3
```

而如果遵守条件在归约前先对该表达式作 α-变换 则有：

```
(λz.λx.z + x) (x + 2) 3
=> (λz.λy.z + y) (x + 2) 3
=> (λy.x + 2 + y) 3
=> x + 2 + 3
```

显然，`3 + 2 + 3` 与 `x + 2 + 3`是完全不同的结果。

## η-变换
**η-变换** 表示可以将等价的两个λ表达式进行替换，等价指的是对于所有的输入两个表达式都会输出相同的结果。比如：
```
λx.f x == f
```
其一般形式为：
```
M == λx. M x if x notsubset free(M)
```
后面的变换条件表示x不是M的自由变量。

## 基本函数

### Identity function 恒等式

| λ 表达式 | ES6 定义 |
|:---------|:---------|
| `λx.x`   | `x => x` |

恒等式将返回其接收的参数，无论这个参数是什么类型：

```js
const identity = x => x;
identity(1);      // => 1
identity(y => y); // => y => y
```

### Self application function 自应用

| λ 表达式   | ES6 定义    |
|:-----------|:------------|
| `λs.(s s)` | `s => s(s)` |

将恒等式传递给自应用：

```js
const selfApply = (s => s(s));
selfApply(identity); // => x => x
```

### Function application function 函数应用

| λ 表达式      | ES6 定义         |
|:--------------|:-----------------|
| `λf.λx.(f x)` | `f => x => f(x)` |

```js
const apply = f => x => f(x);
const getName = obj => obj.name;
apply(getName)({name: '张三'}); // => '张三'
```

### Select first function 选择第一个

| λ 表达式               | ES6 定义                   |
|:-----------------------|:---------------------------|
| `λfirst.λsecond.first` | `first => second => first` |

```js
const first = first => second => first;
first(1)(2); // => 1
```

## Select second function 选择第二个

| λ 表达式                | ES6 定义                    |
|:------------------------|:----------------------------|
| `λfirst.λsecond.second` | `first => second => second` |

```js
const second = first => second => second;
second(1)(2); // => 2
```

## Make pair function 组成一对

| λ 表达式                              | ES6 定义                                   |
|:--------------------------------------|:-------------------------------------------|
| `λfirst.λsecond.λf((f first) second)` | `first => second => f => f(first)(second)` |

```js
const makePair = first => second => f => f(first)(second);
const pair = makePair(1)(2);
pair(first); // => 1
pair(second); // => 2
```

## 算术运算

λ 演算中的自然数不是 **“真正的”** 的自然数，而是用函数的幂数来代替：

```
0 = λf.λx.x
1 = λf.λx.f x
2 = λf.λx.f (f x)
3 = λf.λx.f (f (f x))
```

```js
const ZERO = f => x => x;
const ONE = f => x => f(x);
const TWO = f => x => f(f(x));
const THREE = f => x => f(f(f(x)));
```

### 自增

**自增** 运算 `n = n + 1` 相当与增加函数的幂次，亦即增加一层嵌套：

```js
const succ = n => f => x => f(n(f)(x));
```

### 加法

**加法** 运算 `m + n` 实际上就是 `m` 和 `n` 对应函数的幂次相加，亦即嵌套叠加：

```js
const plus = m => n => f => x => m(f)(n(f)(x));
```

### 乘法

**乘法** 运算 `m * n` 实际上就是 `m` 和 `n`  对应函数的幂次相乘，可以看作求函数的 `n` 次幂的 `m` 次幂：

```js
const multiple = m => n => f => m(n(f));
```
### 自减

**自减** 运算 `n = n - 1` 的实现比较复杂，为了方便理解，我们先定义一个转移表达式：
```js
const trans = p => makePair(p(second))(succ(p(second)));
```
其传入一个 `pair` 函数，返回一个新的 `pair` 函数，新函数中第一个参数值为原函数的第二个参数值，第二个参数值为原函数的第二个参数自增后的值，比如：
```js
trans(makePair(zero)(zero)) => makePair(zero)(one);
```
从上不难发现转移表达式每次执行将当前状态转移到了前驱状态然后将自增后的状态转移到当前状态，用公式可以表示为：

$$

(lastNumber, currentNumber) => (currentNumber, SuccNumber)

$$

于是可以推出由初始状态 `(zero, zero)` 转移n次后，其前驱状态为 `n-1` ，所以 **自减** 运算可以表示为：
```js
const pred = n => (n(trans)(makePair(zero)(zero)))(first);
```

### 减法

**减法** 运算，有了 **自减** 运算后，`m - n` 可以看作将 `m` 自减 `n` 次：

```js
const subt = m => n => n(pred)(m);
```

## 逻辑运算

主要的逻辑运算包含有 **AND（且）**，**OR（或）**，**NOT（非）**，逻辑运算的输出为 **TRUE（真）** 或者 **FALSE（假）**。因此，我们首先定义真值和假值，由于 `true`、`false` 是 JavaScript 中的保留字，我们将真假分别定义为 `TRUE`、`FALSE` ：

```js
const TRUE = first;
const FALSE = second;
const condition = makePair;
```

由于真假的对立性，我们将其用 first 和 second 进行对应。我们再思考，一个条件表达式可以叙述为：

```js
condition ? true : false
```

| condition | output |
|:----------|:-------|
| 1         | 1      |
| 0         | 0      |

借助于上面的定义，我们可以这样实现这则表达式：


```js
const myCondition = condition(TRUE)(FALSE);
TRUE === myCondition(TRUE);   // => true
FALSE === myCondition(FALSE); // => true
```

### NOT

NOT 表达式可以描述为：

```js
not ? false : true
```

| x | output |
|:--|:-------|
| 0 | 1      |
| 1 | 0      |

```js
const not = x => condition(FALSE)(TRUE)(x);
not(TRUE) === FALSE;   // => true
not(FALSE) === TRUE; // => true
```

当 `x` 为真值 `TRUE` 时，将进行 Select first 操作，选取对立的 `FALSE`。

### AND

AND 可以描述为:

```js
x ? y  : false
```

| x | y | output |
|:--|:--|:-------|
| 1 | 1 | 1      |
| 0 | 0 | 0      |
| 1 | 0 | 0      |
| 0 | 1 | 0      |

```js
const and = x => y => condition(y)(FALSE)(x);
and(TRUE)(TRUE) === TRUE;    // => true
and(FALSE)(FALSE) === FALSE; // => true
and(TRUE)(FALSE) === FALSE;  // => true
and(FALSE)(TRUE) === FALSE;  // => true
```

AND 运算的实现思路为，当 `x` 为 `TRUE` 时，

### OR

OR 可以描述为：

```js
x ? true : y
```

| x | y | output |
|:--|:--|:-------|
| 1 | 1 | 1      |
| 0 | 0 | 0      |
| 1 | 0 | 1      |
| 0 | 1 | 1      |

```js
const or = x => y => condition(TRUE)(y)(x);
or(TRUE)(TRUE) === TRUE;    // => true
or(FALSE)(FALSE) === FALSE; // => true
or(TRUE)(FALSE) === TRUE;   // => true
or(FALSE)(TRUE) === TRUE;   // => true
```

### NOT AND

有了几个基本逻辑运算，我们也能更多的逻辑运算，比如 NOT AND（且非）：

| x | y | output |
|:--|:--|:-------|
| 1 | 1 | 0      |
| 0 | 0 | 1      |
| 1 | 0 | 1      |
| 0 | 1 | 1      |

```js
const nand = x => y => not(and(x)(y));
nand(TRUE)(TRUE) === FALSE;    // => true
nand(FALSE)(FALSE) === TRUE; // => true
nand(TRUE)(FALSE) === TRUE;  // => true
nand(FALSE)(TRUE) === TRUE;  // => true
```

## 递归运算

递归运算在命令式语言里很容易实现，比如实现一个累乘运算 `factorial` ，其递归函数可以定义为：

```js
function factorial (n) {
  if (n == 0) {
    return 1;
  } else {
    return n * factorial(n - 1);
  }
}
```

这个函数非常简单且被我们所熟知，它实现的递归其核心在于自身调用自身，且函数名可以标识自身。
然而在λ演算中，没有命名的概念，函数都是匿名函数，无法通过函数名调用自身，所以也就无法通过上述方式进行递归运算。
为了帮助匿名函数实现递归运算，著名数学家 Haskell B. Curry 提出了 **y-combinator** 函数，由这个函数我们可以将普通函数转换为可递归的函数。
要理解该函数，我们需要一步一步的进行推导。首先，我们将引入不动点（fix-point）的概念，所谓不动点就是满足如下公式的函数：

$$
fix = f(fix)
$$

其高阶形式为：

$$
fix \space f = f(fix \space f) \space \space  for \space all \space f
$$

此时的 `fix` 函数被称为 **fixed-point-combinator**，y-combinator 就是属于 fixed-point_combinator 中的一种实现，所以它满足：

$$
Y(f) = f(Y(f)) = f(f(Y(f))) = ... = { f ^ n }(Y(f))
$$

显然，函数 `f` 在 `Y` 的帮助下不断调用自身，成为了一个递归函数。最后我们用 JavaScript 语言来描述Y函数:

```js
function Y (f) {
  return f(Y(f));
}
```

然而这段代码在实际执行中出现了一个很严重问题，这个递归会无限进行下去。这是由于  JavaScript 属于急性求值语言，凡是采取急性求值（Eager Evaluation）策略的语言，都会按上面公式所展现的那样的顺序，在调用` f `前先求取 `Y(f)` 的值，然后不断进行下去陷入无限递归。
在正规的函数式语言如 Haskell 语言中一般采取的是惰性求值（Lazy Evaluation）的策略，这两者的区别在于求值时机的不同，在惰性求值中执行 `f(Y(f))` 时，不会立即求取 `Y(f)` 的值，而是先调用 `f` ，然后在 `f` 内部使用到 `Y(f)` 的时候才计算 `Y(f)` 的值。所以不会造成死循环，但同时也不能保证 `f` 递归执行，因为其内部如果没有执行参数的语句那么 `Y(f)` 将永远不会被执行。
但通过对 `f` 进行定制，我们可以让递归在惰性求值的语言中顺利进行。针对不同的递归函数其定制方式有所区别，但核心思想就是让 `f` 成为一个高阶函数其传入一个用以递推的函数参数并返回一个满足递归要素（递推公式和终止条件）的新函数。下面我们以 `factorial` 来举例其对应的 `f` 函数为：

```js
function f (factorial) {
  return function (n) {
    if (n === 0) return 1;
    else {
      return n * factorial(n-1);
    }
  }
}
```

如果 JavaScript 为惰性求值语言，那么求解 `Y(f)(4)` 将会有如下推导过程：

```
Y(f)(4)
=> f(Y(f))(4)
=> (n => n * (Y(f)(n-1)))(4)
=> 4 * (Y(f)(3))
=> 4 * 3 * (Y(f)(2))
=> 4 * 3 * 2 * (Y(f)(1))
=> 4 * 3 * 2 * 1 * (Y(f)(0))
=> 4 * 3 * 2 * 1 * 1
=> 24
```

可以看到，`Y` 顺利使 `f` 实现了递归运算，但这是建立在惰性求值的基础之上，实际中我们知道 JavaScript 是急性求值语言， 所以我们还需要对 `Y` 进行改造。
改造的目的是让 JavaScript 不立即对 `Y(f)` 进行求值，而让 JavaScript 延迟代码执行的办法，就是在用匿名函数将要执行代码包裹起来，比如下面这段代码：

```js
function log () {
  console.log('我被执行了');
}
log(); // => 我被执行了
```

如果想不立即执行这段打印代码，我们可以这这样做：

```js
function delayLog () {
  return function () {
    console.log('我被执行了');
  }
}
var log = delayLog(); // => [Function]
log(); // => 我被执行了
```

那么如何包裹能让原函数的意图不发生改变呢，这时我们就要引入之前提过的 η-变换，对函数进行等价变换，根据 η-变换 公式我们有：

```
M == λx.M x => Y(f) == x => Y(f)(x)
```
因此我们得出 `Y` 为：
```js
function Y (f) {
  return f(x => Y(f)(x));
}
```

此时，我们要做的最后一步就是将 `Y` 匿名化。那么如何完成这样的变化呢，这需要一点点灵感，我们考虑如果能将 `Y` 以参数的形式传递进来并进行自调用，那么就能做到匿名化，但是我们的 `f` 是用来传递待递归函数，所以我们要将 `Y` 变为高阶函数，让它同时也能传递自身，最后代码就变成了：

```js
var Y = function (f) {
  return (function (y) {
    return f(x => y(y)(x));
  })(function (y) {
    return f(x => y(y)(x));
  });
}
```

ES6 的形式为：

```js
const Y = f => (y => f(x => y(y)(x)))(y => f(x => y(y)(x)));
```

急性求值的 λ 演算形式为：

```
Y = λf.(λy.f(λx.y y x))(λy.f(λx.y y x))
```

惰性求值的 λ 演算形式为：

```
Y = λf.(λy.f(y y))(λy.f(y y))
```

自此 y-combinator 的表达式就推导完毕了。
最后，我们利用 y-combinator 实现 `factorial` 递归函数的代码：

```js
const factorial = Y(r => n => n === 0 ? 1 : n * r(n - 1));
factorial(4); // => 24
```

## 参考资料

- [Wiki -- Lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus)
- [Lambda Calculus in JavaScript — Part 1](https://medium.com/functional-javascript/lambda-calculus-in-javascript-part-1-28ff63824d4d)
- [JAVASCRIPT AND THE LAMBDA CALCULUS, PT. 1](http://fgshprd.github.io/2014/04/12/Javascript_and_the_Lambda_Calculus_pt1.html)
- [JAVASCRIPT AND THE LAMBDA CALCULUS, PT. 2](http://fgshprd.github.io/2014/05/03/Javascript_and_the_Lambda_Calculus_pt2.html)
- [Wiki--λ演算](https://zh.wikipedia.org/wiki/%CE%9B%E6%BC%94%E7%AE%97)
- [Lambda Calculus: Subtraction is hard](http://gettingsharper.de/2012/08/30/lambda-calculus-subtraction-is-hard)
- [lambda calculus:Y-combinator
](http://staynoob.cn/post/2017/03/lambda-calculus-y-combinator)
- [Wiki -- Fixed-point combinator
](https://en.wikipedia.org/wiki/Fixed-point_combinator)
