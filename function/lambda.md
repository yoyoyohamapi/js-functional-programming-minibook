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
自由变量指未被绑定在任何一个λ上的变量，如公式：
```
λx.x + y
```
中，y没有与λ绑定，所以y就是一个自由变量。
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
其一般形式为：
```
((λV.E) E') == E [V:=E'] if free(E') subset free(E[V:E'])
```
注意后面的归约条件，它表示我们必须在保证自由变量不会与绑定变量起冲突的前提下才能进行归约操作，即保证 E' 表达式中的自由变量在归约后不会变为 E[V:E'] 表达式的绑定变量。
我们举个例子来形象的说明这一点，假设存在表达式：
```
λz.λx.z + x
```
现在我们将应用它：
```
(λz.λx.z + x) (x + 2)
```
若直接按β-归约化简则为：
```
(λx.x + 2 + x)
```
于是原本(x + 2)中的自由变量x变成了绑定变量，如果进一步应用函数：
```
(λx.x + 2 + x) 3
```
则其最后变为：
```
3 + 2 + 3
```
而如果遵守条件在归约前先对该表达式作α-变换则有：
```
(λz.λx.z + x) (x + 2) 3
=> (λz.λy.z + y) (x + 2) 3
=> (λy.x + 2 + y) 3
=> x + 2 + 3
```
显然，`3 + 2 + 3` 与 `x + 2 + 3`是完全不同的结果。
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
**自增** 运算 `n = n + 1` 相当与增加一层嵌套：
```js
const succ = n => f => x => f(n(f)(x));
```
**加法** 运算 `m + n` 实际上就是 `m` 和 `n` 对应函数的幂次相加，亦即嵌套叠加：

```js
const plus = m => n => f => x => m(f)(n(f)(x));
```
**乘法** 运算 `m * n` 实际上就是 `m` 和 `n`  对应函数的幂次相乘，可以看作求函数的 `n` 次幂的 `m` 次幂：
```js
const mutiple = m => n => f => m(n(f));
```
**自减** 运算 `n = n - 1`，其实现比较复杂，为了方便理解，我们先定义一个转移表达式：
```js
const trans = p => makePair(p(second))(succ(p(second)));
```
其传入一个pair函数，返回一个新的pair函数，其第一个参数为原来pair函数的第二个参数，第二个参数为原来pair函数的第二个参数自增后的值。
于是，自减运算可以看作是由初始状态(zero)(zero)转移n次后所返回的pair函数的第一个参数值：
```js
const pred = n => (n(trans)(makePair(zero)(zero)))(first);
```
**减法** 运算，有了**自减** 运算后，`m - n` 可以看作将m自减n次：
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

## 参考资料

- [Lambda Calculus in JavaScript — Part 1](https://medium.com/functional-javascript/lambda-calculus-in-javascript-part-1-28ff63824d4d)
- [JAVASCRIPT AND THE LAMBDA CALCULUS, PT. 1](http://fgshprd.github.io/2014/04/12/Javascript_and_the_Lambda_Calculus_pt1.html)
- [JAVASCRIPT AND THE LAMBDA CALCULUS, PT. 2](http://fgshprd.github.io/2014/05/03/Javascript_and_the_Lambda_Calculus_pt2.html)
- [Wiki--λ演算](https://zh.wikipedia.org/wiki/%CE%9B%E6%BC%94%E7%AE%97)
- [Lambda Calculus: Subtraction is hard](http://gettingsharper.de/2012/08/30/lambda-calculus-subtraction-is-hard)
