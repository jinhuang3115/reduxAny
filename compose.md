#compose
函数式编程中的一个方法，从右到左组合多个函数。

参数:

需要组合的多个函数

返回:

从右到左把接收到的函数合成后的最终函数。

```apple js
function compose() {
  for (var _len = arguments.length, funcs = Array(_len), _key = 0; _key < _len; _key++) {
    funcs[_key] = arguments[_key];
  }

  if (funcs.length === 0) {
    return function (arg) {
      return arg;
    };
  }

  if (funcs.length === 1) {
    return funcs[0];
  }

  return funcs.reduce(function (a, b) {
    return function () {
      return a(b.apply(undefined, arguments));
    };
  });
}
```
首先去循环传入的所有的函数参数，并把这些函数存储到数组funcs中。
如果没有任何参数传入则返回一个函数，这个函数只返回函数参数。

如果传入了一个函数参数，则直接返回当前函数。

如果传入的函数参数为多个，则对数组使用reduce方法，并传入一个函数。(reduce方法对累加器和数组中的每个元素（从左到右）应用一个函数，将其减少为单个值。)

example：
```apple js
function fn1() {
  console.log(1)
}
function fn2() {
  console.log(2)
}
function fn3() {
  console.log(3)
}
var arr = [fn1,fn2,fn3];
arr.reduce(function(a, b) {
  return function() {
    return a(b.apply(undefined, arguments));
  }
})
```

```apple js
function fn1() {
  console.log(1)
}
function fn2() {
  console.log(2)
}
function fn3() {
  console.log(3)
}
fn1(fn2(fn3()));
```