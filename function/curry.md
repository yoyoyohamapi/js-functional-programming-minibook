# 柯里化

在上一节中我们提到，有些时候，对函数的传参我们希望不是一次性完成的，这有利于我们获得新的函数。例如，我们创建了一个能够获得对象某个属性的函数：

```js
const prop = (key, obj) => obj[key];

const person = {
  name: '张三',
  age: 18
};

prop('age', obj); // => 18
```

如果我们能够对 `prop(key, obj)` 方法进行逐个传参，那就可以更多的函数了：

```js
const getName = prop('name');
const getAge = prop('age');

const persons = [{
  name: '张三',
  age: 18
}, {
  name: '李四',
  age: 24
}, {
  name: '王五',
  age: 33
}];

const names = persons.map(getName); // => ['张三', '李四', '王五']
const ages = persons.map(getAge); // => [18, 24, 33]
```

要使得函数能够逐个接收参数，就要对函数进行 curry 化。curry 是一个函数包装器，其魔力不亚于同样叫做 curry 的三分神射。

<div style="text-align:center">
  <img src="./curry_sc30.jpg" width="350"></img>
</div>

## curry 的实现

curry 是一个函数包装器，经过 curry 包装的函数，将具备逐个接收参数的能力：

```js
const prop = curry((key, obj) => obj[key]);
```

据此，我们应当知道，实现一个 curry 函数的要点：

- 知道被 curry 化函数的参数个数（函数元数）。
- 每接收一个参数，将返回一个新的函数，当达到参数个数时，再进行函数调用。

```js
/**
 * curry 化
 * @param {Function} func
 * @param {Array} args
 * @return {Function}
 */
const curry = function (func, args) {
  // 确定 func 的参数个数，
  const len = func.length;
  args = args || [];
  return function curried() {
    // 合并当前参数
    const combined = args.concat(Array.prototype.slice.call(arguments));
    return combined.length < len ? curry(curried, combined) : fun.call(this, combined);
  }
}
```

测试一下：

```js
const prop = curry((key, obj) => obj[key]);

const getName = prop('name');
const getAge = prop('age');

const persons = [{
  name: '张三',
  age: 18
}, {
  name: '李四',
  age: 24
}, {
  name: '王五',
  age: 33
}];

const names = persons.map(getName); // => ['张三', '李四', '王五']
const ages = persons.map(getAge); // => [18, 24, 33]
```

## ES6 curry

借助于 ES6 的 [rest 参数]() 我们能更简短的实现 curry 化方法：

```js
// `arr` 保存了制定的参数，`args` 描述了当前步骤传入的参数
const curry = (f, arr = []) =>
  (...args) => (combined => combined.length >= f.length ? f(...combined) : curry(f, combined))([...arr, ...args])
```

## 占位符

一般来说，我们进行 curry 化的顺序是按照参数声明顺序从左到右的，但是，有时候我们我们也想先制定后面的参数，这个需求可以通过占位符完成：

```js
const add = curry((x, y) => x + y);
add3 = add(_, 3);
```

为此，需要修改我们的 curry 化方法：

```js
const _ = '@@placeholder';

const curry = (f, arr = []) => {
  return (...args) => {
    let j = 0;
    // 跳过占位符
    for (let i = 0; i < arr.length; i++) {
       if (arr[i] === _) {
           arr[i] = args[j++];
       }
    }
    const combined = j < args.length ? [...arr, ...args.slice(j)] : [...arr];
    const validArgs = combined.filter(arg => arg !== _);
    return validArgs.length >= f.length ? f(...combined) : curry(f, combined);
  };
};
```

测试一下：

```js
const add3Numbers = curry((x, y, z) => {
  const sum = x + y + z;
  console.log(`${x} + ${y} + ${z} = ${sum}`);
  return sum;
});

add3Numbers(1,2,3); // => "1 + 2 + 3 = 6"
add3Numbers(1)(2)(3); // => "1 + 2 + 3 = 6"
add3Numbers(_, 2)(_, 3)(1); // => "1 + 2 + 3 = 6"
add3Numbers(_, _, 3)(1, 2); // => "1 + 2 + 3 = 6"
```

## 参考资料

- [Composing Software Part 3](https://medium.com/javascript-scene/a-functional-programmers-introduction-to-javascript-composing-software-d670d14ede30)
