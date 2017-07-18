compose 函数组合
===========

Eric Elliot 大神认为，软件的编写之道就在于函数组合，只需要一些基础函数，就能组合出各种复杂的业务逻辑。这里所说的函数组合不是 mixin，而是类似管道（pipeline）或者流水线的函数连接，每个函数就类似流水线上的一个部件，数据由流水线的一端流入，经各个函数操作后，新的数据从流水线另外一端流出：

<div style="text-align:center">
  <img src="./pipeline.png" style="width: 500px"></img>
</div>

假定我们有如下的一份学生成绩数据，上图描述了一个统计优秀学生，并且排序的业务：

```js
const students = [{
  name: '张三',
  score: 73,
  sex: 'male',
}, {
  name: '王五',
  score: 100,
  sex: 'male'
}, {
  name: '刘丽',
  score: 62,
  sex: 'female'
}, {
  name: '李四',
  score: 93,
  sex: 'male'
}];
```

## 由内而外的组合

最早，我们可能是通过函数嵌套的方式来实现函数的组合：

```js
const prop = curry((name, obj) => obj[name]);
const isExcellent = curry((threshold, student) => student.score > threshold);

const filterExcellents = students => students.filter(isExcellent(70));
const sortScores = students => students.sort((a, b) => a.score - b.score > 0);
const pickName = students => students.map(prop('name'));
const getExcellents = students =>
  pickName(sortScores(filterExcellents(students)));
```

测试：

```js
getExcellents(students); // => ['张三', '李四', '王五']
```

如果业务流程更加复杂，那么这种嵌套是十分痛苦的，处理括号都成问题：

```js
// a --> b --> c --> d
d(c(b(a)));
```

## 顺序组合

### compose

在数学上，函数的组合 `g(f(x))` 可以表述为 `g ∘ f`，读作 `g after f`，亦即，函数的组合方向是**从右向左**的，我们称这样的组合为 **compose**。借助于 `Array.prototype.reduceRight` 方法，我们很容易实现 compose：

```js
const compose = (...funcs) => x => funcs.reduceRight((d, f) => f(d), x);
```

通过 `compose` 改造我们上面的业务：

```js
const getExcellents = compose(
  pickName,
  sortScores,
  filterExcellents
);

getExcellents(students); // => ['张三', '李四', '王五']
```

现在，我们的函数组合在书写上就是线性的了，便于对流程的阅读和理解。

### pipe

部分开发者可能对于**从右向左**的函数组合顺序有点接受不能，因此，最好再提供一个**从左向右**的组合方式，这种组合方向就类似于左进右出的管道，我们将之命名为 **pipe**

```js
const pipe = (...funcs) => x => funcs.reduce((d, f) => f(d), x);
```

```js
const getExcellents = pipe(
  filterExcellents,
  sortScores,
  pickName
);

getExcellents(students); // => ['张三', '李四', '王五']
```

## 参考资料
- [The Rise and Fall and Rise of Functional Programming (Composing Software)](https://medium.com/javascript-scene/the-rise-and-fall-and-rise-of-functional-programming-composable-software-c2d91b424c8c)
- [Why is `compose` right to left?](https://www.coreycleary.me/why-is-compose-right-to-left/)
