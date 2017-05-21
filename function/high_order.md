# 高阶函数

## 单薄的函数

在面向对象编程（object oriented programming）中，函数往往是类的某个成员方法：

```javascript
class Caculator {
  constructor() {}

  add(x, y) {
    return x+y;
  }
}

new Caculator().add(1, 2); // 3
```

在 **指令式** 编程中（imperative programming），函数更简单了：

```javascript
function add(a, b) {
  return a + b;
}

add(1, 2); // 3
```

无论是面向对象编程还是指令式编程，概括下来，函数只完成了那么一件事：

$$

f(x) = y

$$

其中，$$x$$ 和 $$y$$ 都仅仅是值。

上文中，我们定义了一个加法函数：

$$

add(x, y) = x + y

$$

假如我们开发的一个理财软件，某天需要给所用用户的账户都派发一个 2 元的红包，我们就需要一个新的加法函数，其中 $$x$$ 被固定为了 2：

$$

add2(y) = 2 + y

$$

在命令式编程中，我们只能这样创建这个新函数：

```javascript
function add2(y) {
  return 2 + y;
}
```

显然，不熟练的开发者直接就复制粘贴 `add(x,y)` 函数，再手动改改，就有了 `add2(y)` 函数了，这不符合 **DRY（Don't Repeat Yourself）** 原则。

# 由函数创建函数

在函数式编程中，我们可以通过 **高阶函数** 来创建新函数。简言之，高阶函数是一个能返回函数的函数。现在，我们先重新定义我们的 `add` 方法，让它能 **分步骤** 接收参数：

```javascript
function add(x) {
  return function (y) {
    return x + y;
  }
}
```

那么，我们的 `add2(y)` 方法就可以这样产生：

```javascript
const add2 = add(2);
add(2)(1); // => 3
add2(1); // => 3
add2(10); // => 12
```

你也看到了，高阶函数的实现仰仗于 **闭包**，闭包控制住了上一阶函数的参数。这里不再赘述什么是闭包，它是 JavaScript 基础中的基础。

现在还不是兴奋的时候，倘若我们的加法函数是一个三元函数：

$$

add(x, y, z) = x + y + z

$$

我们就需要创建一个更高阶的函数：

```javascript
function add(x) {
  return function(y) {
    return function(z) {
      return x + y + z;
    }
  }
}
```

这是痛苦的嵌套，就像 JavaScript 中的回调地狱一样。我们非常渴望能有那么一个机制，它能让一个函数像个投币机一样，当参数个数（函数元数）达到个数时，才开始工作。下一节中，我们会介绍，这个机制叫做 Curry -- 柯里化。
