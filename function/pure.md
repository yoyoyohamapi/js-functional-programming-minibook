# 纯函数与副作用

函数，就是接收一些输入（input），并且产生输出（output），其中，输入又被称为参数（arguments），而输出又被称为返回值（return value）。一般来说，函数大体有如下三种作用：

1. 处理映射：函数将给定的输入映射为给定的输出。
2. 执行过程：函数执行一段由若干步骤组成的过程，这种风格下的程序设计称之为**过程式程序设计**或者**过程式编程**。
3. I/O：一些函数甚至还会与系统其他组成打交道，比如屏幕，存储，系统日志，网络等等。

**纯函数（pure function）**则是函数的子集，它的最简单的定义是：

> 给定相同的输入，那么一定能得到一样的输出。

比如下面这个函数，它能获得 `x` 的两倍值，只要传入相同的 `x`， 必然能获得一致的输出：

```js
const double = x => x * 2
let y = double(2) // => 4
```

相对地，下面这个能够获得当前时间的函数 `now()` 则是不纯的，因为我们传递给其一致的输入（这里的输入为空），并不能获得一致的输出：

```js
const now = () => new Date().toLocalTimeString()
now() // => '下午3:22:08'
now() // => '下午3:24:32'
now() // => '下午3:30:10'
```

## 副作用（Side effects）

函数有副作用，指的是一个函数在自己的作用域内和某个**外部状态**交互，改变了这个外部状态。或者函数的执行流程受到外部状态的影响，继而影响到自己的输出。副作用是函数和外部世界最常见方式，比如在过程式程序设计中，每个函数的流程都充斥着副作用。

纯函数不应当和副作用进行交互，如果我们将我们的翻倍函数改成下面这样：

```js
const double2 = x => {
  y = x * 2
  return y
}
let y
double2(2)
```

`double2()` 函数引用了外部的自由变量 `y`，并且具备修改 `y` 的能力。如果我们其他的函数 `foo()`、`bar()`、`baz()` 也使用了 `y`，我们就无法预测这些函数的输出，因为我们无法知道在这三个函数执行前，`double2()` 是否修改了 `y` 的值：

```js
foo()
bar()
baz()
```

如果我们的程序中函数总是在和副作用打交道，我们就无法预测函数的输出，因此，代码的可读性会降低。一个接手这些代码的开发者，只能完整的执行每一行，才知道每一行的结果，头皮发麻。而如果这些函数远离了副作用，那么即使我们不执行它们，我们也能胸有成竹的知道他的输出应当是多少。

在常见的 JavaScript 程序中，控制台 （console），DOM Element，XMLHttpRequest 等都是外部对象，因此都是副作用。副作用并不是坏事，只是需要我们谨慎对待，在程序撰写的过程中，我们需要尽可能分离逻辑和副作用。

## 引用透明（Referential Transparency）

有时候，我们想当然的会认为某个函数是纯的，比如：

```js
const rememberNumbers = function (nums) {
	return function caller(fn){
		return fn( nums )
  }
}

const list = [1,2,3,4,5]

const simpleList = rememberNumbers( list )
```

我们可能会认为 `simpleList` 是一个纯函数，它仅只是接收一个函数 `fn` 来操纵已经被闭包保存的数值序列 `nums`。假设传入的 `fn` 是一个取序列中位数的函数：

```js
const median = function (nums) {
  return (nums[0] + nums[num.length-1]) / 2
}

simpleList(median) // => 3

list.push(6)

simpleList(median) // => 4
```

我们发现，由于闭包包住的 `nums` 是一个序列，在 JavaScript 中，实际上包住的是一个序列引用。因此，当序列 `nums` 在外部发生变化时，`simpleList` 也会受到影响。

有时候，我们也会想当然的认为某个函数式不纯的：

```js
const PI = 3.14
const perimeter = r => {
  return PI * 2 * r
}
```

由于 `PI` 是一个不可重新赋值的常量，因此，即便 `perimeter` 引用了外部对象 `PI`，它仍然是稳定的，因此可以认为 `perimeter` 是一个纯函数。

所以，通常我们需要使用**引用透明（Referential Transparency）**来判断一个函数是否纯。引用透明指的是：

> “如果一个表达式或者函数调用能够被它的输出所代替，并且不影响到程序，那么这个表达式或者这个函数调用就是引用透明的。”

比如：

```js
const PI = 3.14
const perimeter = r => {
  return PI * 2 * r
}
const p = perimeter(2)

console.log('perimeter is ', p)
```

我们直接使用 `perimeter(2)` 的值 `12.56` 来代替这个语句：

```js
const PI = 3.14
const perimeter = r => {
  return PI * 2 * r
}
const p = 12.56

console.log('perimeter is ', p)
```

我们发现程序不受影响，因此 `perimeter(2)` 就是引用透明的，函数 `perimeter()` 是纯函数。

## 相对纯度

我们回顾上面提到的例子，`rememberNumbers` 是一个非纯函数，接下来我们要将它优化成一个纯函数：

```js
const rememberNumbers = function (nums) {
	return function caller(fn){
		return fn( nums )
  }
}
```

考虑到 `nums` 是一个数组，闭包只会包住数组的引用，因此我们先拷贝这个数组：

```js
const rememberNumbers = function ([..nums]) {
  return function caller(fn) {
    return fn(nums)
  }
}
```

现在，我们通过 `[...nums]` 拷贝了 `nums`，这保证了在 `rememberNumbers` 中使用的序列仅只是 `nums` 的一次拷贝，即便在外部，`nums` 发生了变化，`rememberNumbers` 使用的序列也不受影响。测试一下：

```js
const median = function (nums) {
  return (nums[0] + nums[num.length-1]) / 2
}

simpleList(median) // => 3

list.push(6)

simpleList(median) // => 3
```

但是仍然不够，我们还不能保证 `fn` 使用的 `nums` 不会发生变化：

```js
const firstValue = function (nums) {
  return nums[0]
}

const lastValue = function (nums) {
  return firstValue(nums.reverse())
}

const list = [1, 2, 3, 4, 5]
const simpleList = rememberNumbers(list)

simpleList(lastValue) // => 5
simpleList(lastValue) // => 1
```

`simpleList` 仍然是不纯的。在 JavaScript 中，`Array.prototype.reverse` 函数是不安全的，它不会返回新的数组，而是直接修改原数组。因此，我们的 `nums` 还是被修改了。我们继续修改函数 `rememberNumbers`：

```js
const rememberNumbers = function ([...nums]) {
  return function caller (fn) {
    return fn([...nums])
  }
}
```

现在，`fn` 也只是使用数组拷贝：

```js
const list = [1, 2, 3, 4, 5]
const simpleList = rememberNumbers(list)

simpleList(lastValue) // => 5
simpleList(lastValue) // => 5
```

现在 `simpleList` 就是纯函数了吗？Hold on，Hold on...：

```js
simpleList(function impureIO (nums) {
  console.log(nums.length)
})
```

如果传入 `simpleList` 的 `fn` 仍然包含了副作用，我们还是无法保证 `simpleList` 是纯的。这也反映了一点，在 JavaScript 中，往往只能追求函数**足够纯**，而无法做到**绝对纯**。


## 函数纯化

接下来还会分门别类地介绍一些能够提升函数纯度的手段。

### 包住副作用

如果一个非纯函数引用了外部的自由变量，并且这个自由变量来自于我们自己的代码，我们就能轻易地创建一个 wrapper，来限制住这个非纯函数和其引用的副作用。看到下面的一个例子，`fetchUserData` 函数引用了外部不稳定的变量 `users`，因此不是一个纯函数：

```js
const users = {}
const fetchUserData = function(userId) {
  ajax(`http://some.api/user/${userId}`, function onUserData(userData){
		users[userId] = userData
	})
}
```

由于这些代码我们可控，我们为该函数及其引用的副作用创建一个 wrapper。同样地，由于 `users` 是对象，我们需要通过 `{...users}` 使用它的拷贝：

```js
const safeFetchUserData = function (userId, {..users}) {
  fetchUserData(userId);
  return users;
}
```

### 遮盖副作用

如果不纯的函数及其引用的副作用来自于第三方库，我们就无法创建一个函数来包住这些副作用：

```js
const nums = [];
const smallCount = 0;
const largeCount = 0;

const generateMoreRandoms = function(count) {
	for (let i = 0; i < count; i++) {
		const num = Math.random();

		if (num >= 0.5) {
			largeCount++;
		}
		else {
			smallCount++;
		}

		nums.push(num);
	}
}
```

这个函数引用了三个外部状态 `nums`、`smallCount`、`largeCount`。我们创建一个安全版本的 `generateMoreRandoms`，需要经历如下几个步骤：

1. 传入初始化状态。
2. 缓存外部状态。
3. 使用初始状态来初始化即将成为副作用的状态。
4. 执行非纯函数。
5. 缓存执行完成后的副作用状态。
6. 使用第 2 步中的缓存来修复外部状态。
7. 返回第 5 步中缓存的副作用状态。


```js
//   1. 传入初始状态 intialState
const safeGenerateMoreRandoms = function (count, initialState) {
  // 2. 缓存外部状态
  const originalState = {
    nums,
    smallCount,
    largeCount
  }

  // 3. 初始化即将成为副作用的状态
  nums = [...initialState.nums]
  smallCount = initialState.smallCount
  largeCount = initialState.largeCount

  // 4. 运行非纯函数
  generateMoreRandoms(count)

  // 5. 缓存副作用
  const sides = {
    nums,
    smallCount,
    largeCount
  }

  // 6. 修复外部状态
  nums = originalState.nums
  smallCount = originalState.smallCount
  largeCount = originalState.largeCount

  return sides
}
```

这种通过**缓存--修复**的手段来保障外部状态安全的手段更加通用，但明显地，由于对副作用的缓存和对外部状态的修复都是同步的，因此这个方式只适用于同步代码，而不适用于非纯函数在**异步流程**中和副作用交互的场景。

### 深度拷贝

我们已经学会了在处理可变对象（数组，对象等）作为输入时，使用它们的拷贝。但要注意，有时候，这些可变对象中的元素也是可变对象，比如我们的一份用户信息列表，它是一个 array，它内部的元素又都是 object。我们就需要进行**深度拷贝**来保证函数的纯度：

```js
const handleInactiveUsers = function (userList, dateCutoff) {
  for (let i = 0; i < userList.length; i++) {
    if (userList[i].lastLogin == null) {
      // remove the user from the list
      userList.splice( i, 1 )
      i--;
    } else if (userList[i].lastLogin < dateCutoff) {
      userList[i].inactive = true
    }
  }

}
```

```js
const safeHandleInactiveUsers = function (userList, dateCutoff) {
  // deep copy
  const copiedList = userList.map(user => {...user})
  // call impure function
  handleInactivceUsers(copiedList, dateCutoff)
  return copiedList
}
```

### 注意 `this`

我们都知道，`this` 在 JavaScript 是不 “稳定” 的，实际上，我们可以认为 `this` 作为隐式参数输入到了函数中：

```js
const ids = {
	prefix: "_",
	generate() {
		return this.prefix + Math.random();
	}
}
```

更好的做法是，执行上下文我们不依赖于 `this` ，而是显式指定：

```js
const safeGenerate = function (context) {
	return ids.generate.call(context)
}

safeGenerate({prefix: '@@'})
```

你可以在 underscore 或者 loadash 这样流行的函数库里面看到，它们的很多函数都是显示指定上下文的：

```js
// underscore 1.8.3
_.some = _.any = function(obj, predicate, context) {
  predicate = cb(predicate, context);
  var keys = !isArrayLike(obj) && _.keys(obj),
      length = (keys || obj).length;
  for (var index = 0; index < length; index++) {
    var currentKey = keys ? keys[index] : index;
    if (predicate(obj[currentKey], currentKey, obj)) return true;
  }
  return false;
};
```

## 总结

纯函数的好处是显而易见的，不仅能提高业务逻辑的可读性，还方便函数的测试。而副作用则是程序 Bug 频出的地方，但是不存在没有副作用的程序。所以，在编撰 JavaScript 应用程序的时候，我们应当尽可能分离逻辑和副作用。你可以在 [redux-saga](https://github.com/redux-saga/redux-saga)，或者 [cycle.js](https://github.com/cyclejs/cyclejs) 等等流行库或者框架中，都在强调或者约束将副作用从逻辑中分离出去。

## 参考资料
- [Side effect (computer science)](https://www.wikiwand.com/en/Side_effect_(computer_science))
- [Functional-Light JavaScript -- Chapter 5: Reducing Side Effects](https://github.com/getify/Functional-Light-JS/blob/master/ch5.md)
- [Master the JavaScript Interview: What is a Pure Function?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976)
