# 偏函数应用

上一节中，我们学习了 curry 化技术，curry 帮助我们逐个对函数进行参数填充。在函数式编程中，还有一个与 curry 非常类似的技术，叫做 partial application，偏函数应用。

偏函数是一个接收了 **部分** 参数的函数，如果它再接收完 **剩余** 参数，那么该函数就能被调用：

$$

\begin{align*}
& \mbox{原函数：} sourceFunction(x,y,z) \\
& \mbox{新函数：} newFunction(y, z) = factory(sourceFunction, x)
\end{align*}

$$

partial application 和 curry 的区别在于：curry 化的过程总是返回一个一元函数，直到所有参数接收完毕，这个性质对于之后提到的函数组合（compose）至关重要。而 partial application 就没有那么严苛，旨在于通过接收部分参数来获得新的函数。

## partial 实现

我们已经实现了 curry，所以实现与之类似的 partial application 也不难，仍然考虑占位符：

```js
const _ = '@@placeholder';

const partial = (...args) => {
    const func = args.shift();
    return (...a) => {
        const len = args.length;
        let j = 0;
        for (let i = 0; i < len; i++) {
            if (args[i] === _) {
                args[i] = a[j++];
            }
        }
        if (j < a.length) {
            args = [...args, ...a];
        }
        return func(...args);
    }
}
```

测试一下：

```js
const parseInt10 = partial(parseInt, _, 10);
const parseInt16 = partial(parseInt, _, 16)
const parseInt2 = partial(parseInt, _, 2);

parseInt10('10'); // => 10
parseInt16('10'); // => 16
parseInt2('10'); // => 2
```

## 参考资料

- [Curry or Partial Application?](https://medium.com/javascript-scene/curry-or-partial-application-8150044c78b8)
- [What is the difference between currying and partial application?

](http://stackoverflow.com/questions/218025/what-is-the-difference-between-currying-and-partial-application)
