# Pointfree

## Ponitful

我们从后端拉取了一份包含了 2016 学年学生期末考的成绩的数据：

```javascript
const info = {
  year: '2016',
  students: [{
      name: '张三',
      id: 1,
      math: 90,
      english: 67
  }, {
      name: '李四',
      id: 2,
      math: 52,
      english: 80
  }, {
      name: '王五',
      id: 3,
      math: 47,
      english: 91
  }, {
      name: '赵六',
      id: 4,
      math: 88,
      english: 87
  }]
};
```

我们需要获得数学不及格的学生学号（id），如果利用 underscore 我们会这样做：

```javascript
const _ = require('underscore');

const mathNotPass = info => {
  const notPassStus = _.filter(_.property('students')(info), student => student.math >= 60);
  return _.map(notPassStus, _.property('id'));
}

mathNotPass(info); // => [1, 4]
```

如果知道 underscore 提供的链式调用的话，能再优雅一些：

```javascript
const _ = require('underscore');

const mathNotPass = info =>
  _.chain(info)
  .result('students')
  .filter(student => student.math >= 60)
  .map(_.property('id'))
  .value();

mathNotPass(info); // => [1, 4]
```

如果我们想要通过函数组合的方式：

```javascript
const _ = require('underscore');

const mathNotPass = _.compose(
  students => _.map(students, _.property('id')),
  students => _.filter(students, student => student.math >= 60),
  _.property('students')
);

mathNotPass(info); // => [1, 4]
```

underscore 使用的函数是一种 **Pointful** 风格的函数，这种风格首先关注于 **数据**，意味着函数 **先确定数据，再确定对数据的处理方式**。因此，我们构造函数组合链时，不得不创建新的函数来让 `map`、`filter` 等 API 最后再接收数据：

```javascript
students => _.filter(students, student => student.math >= 60)
```

## Pointfree

Pointfree 则是 Pointful 风格的对立。在 [Mostly Adequate Guide](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch5.html#pointfree) 中，对 Pointfree 描述非常贴切：

> Pointfree style means never having to say your data

亦即，Pointfree 风格下的函数，永远只关注处理数据的方式。Pointfree 更贴近限时，我们把数据的处理方式看做流水线上的设备，他们是很少变动的，而流水线上的数据才是易变的。

ramda 库即采用了 Pointfree 风格。利用 ramda，我们统计不及格学生的学号：

```js
const R = require('ramda');

const mathNotPass = R.compose(
  R.map(R.prop('id')),
  R.filter(student => student.math >= 60),
  R.prop('students')
);

mathNotPass(info); // => [1, 4]
```

相比较于 underscore 的 Pointful 风格，ramda 的 Pointfree 不再需要改造 `map`、`filter` 等 API。往后我们在设计函数时，尽量使用 Pointfree 的风格来应对数据的变动。

## 参考资料

- [Hey Underscore, You're Doing It Wrong!](https://www.youtube.com/watch?v=m3svKOdZijA)
- [HaskellWiki - Pointfree](https://wiki.haskell.org/Pointfree)
- [Mostly Adequate Guide](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch5.html#pointfree)
