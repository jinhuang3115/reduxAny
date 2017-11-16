# bindActionCreators
惟一使用 bindActionCreators 的场景是当你需要把 action creator 往下传到一个组件上，却不想让这个组件觉察到 Redux 的存在，而且不希望把 Redux store 或 dispatch 传给它。

参数:
1. actionCreators(一个 action creator，或者键值是 action creators 的对象)
2. dispatch

返回:

一个与原对象类似的对象，只不过这个对象中的的每个函数值都可以直接 dispatch action。如果传入的是一个函数，返回的也是一个函数。

```apple js
function bindActionCreators(actionCreators, dispatch) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch);
  }

  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error('bindActionCreators expected an object or a function, instead received ' + (actionCreators === null ? 'null' : typeof actionCreators) + '. ' + 'Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?');
  }

  var keys = Object.keys(actionCreators);
  var boundActionCreators = {};
  for (var i = 0; i < keys.length; i++) {
    var key = keys[i];
    var actionCreator = actionCreators[key];
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch);
    }
  }
  return boundActionCreators;
}
```

首先回去判断如果actionCreators是函数，就执行bindActionCreator并把actionCreators和dispatch传入进去。
```apple js
function bindActionCreator(actionCreator, dispatch) {
  return function () {
    return dispatch(actionCreator.apply(undefined, arguments));
  };
}
```
bindActionCreator会返回一个函数，这个函数会去直接dispatch action。

如果不是函数又不是对象也不是null，那么就会报出错误。

如果actionCreators是一个对象，会去执行如下操作:
1. 定义一个keys来存储actionCreators中的所有键。
2. 定义一个空对象boundActionCreators。
3. 循环keys进行如下操作:
    1. 定义key来存储当前的键
    2. 定义actionCreator取出当前的actionCreators
    3. 如果actionCreator是函数就把bindActionCreator返回的结果存储到boundActionCreators中以当前key为键

    
最后返回boundActionCreators。

实际上就是把 action creators 转成拥有同名 keys 的对象，但使用 dispatch 把每个 action creator 包围起来，这样可以直接调用它们。