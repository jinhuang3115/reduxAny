# applyMiddleware
使用包含自定义功能的 middleware 来扩展 Redux 是一种推荐的方式。Middleware 可以让你包装 store 的 dispatch 方法来达到你想要的目的。同时， middleware 还拥有“可组合”这一关键特性。多个 middleware 可以被组合到一起使用，形成 middleware 链。其中，每个 middleware 都不需要关心链中它前后的 middleware 的任何信息。

参数:

中间件或多个中间件

返回:

应用了中间件的store

```apple js
function applyMiddleware() {
  for (var _len = arguments.length, middlewares = Array(_len), _key = 0; _key < _len; _key++) {
    middlewares[_key] = arguments[_key];
  }

  return function (createStore) {
    return function (reducer, preloadedState, enhancer) {
      var store = createStore(reducer, preloadedState, enhancer);
      var _dispatch = store.dispatch;
      var chain = [];

      var middlewareAPI = {
        getState: store.getState,
        dispatch: function dispatch(action) {
          return _dispatch(action);
        }
      };
      chain = middlewares.map(function (middleware) {
        return middleware(middlewareAPI);
      });
      _dispatch = compose.apply(undefined, chain)(store.dispatch);

      return _extends({}, store, {
        dispatch: _dispatch
      });
    };
  };
}
```

首先，applyMiddleware会把传入的所有中间件放入数组middlewares中。

然后会返回一个匿名函数，这个函数接收createStore作为参数,并且这个函数还会返回一个匿名函数，这个函数接收3个参数：
1. reducer
2. preloadedState
3. enhancer
这三个参数在createStore中有介绍。[createStore](createStore.md)
在这个匿名函数中，首先会定义一些变量:
1. store: 存储createStore返回的store
2. _dispatch: 存储这个store的dispatch方法
3. chain: 一个空数组
4. middlewareApi: 一个对象。这个对象有两个属性:
    1. getState: getState(当前的state)
    2. dispatch: 函数dispatch(这个函数返回之前获取的_dispatch)）
    
之后会循环传入的中间件middlewares，并把传入middlewareAPI执行后的结果存入chain中。

_dispatch会用来存储这些中间件组合执行后的结果，并把当前store的dispatch传入进去。

最后返回一个新的组合好的store。

applyMiddleware实际上做的是让中间件可以对store和dispatch进行包装。

