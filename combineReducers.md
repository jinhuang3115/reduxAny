# combineReducers
combineReducers 把一个由多个不同 reducer 函数作为 value 的 object，合并成一个最终的 reducer 函数。合并后的 reducer 可以调用各个子 reducer，并把它们的结果合并成一个 state 对象。state 对象的结构由传入的多个 reducer 的 key 决定。

参数:

reducers(一个对象，属性为不同的reducer)

返回:

返回一个包含了所有reducers的reducer函数，并且创建一个与reducers 对象结构相同的 state 对象。

```apple js
function combineReducers(reducers) {
  var reducerKeys = Object.keys(reducers);
  var finalReducers = {};
  for (var i = 0; i < reducerKeys.length; i++) {
    var key = reducerKeys[i];

    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning('No reducer provided for key "' + key + '"');
      }
    }

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key];
    }
  }
}
```

首先会通过Oject.keys方法把传入的reducers对象的键都取出来放入reducerkeys里，并且创建一个空对象finalReducers。之后会循环把reducers里的属性方法复制到finalReducers中。

```apple js
function combineReducers(reducers) {
  ...
  ...
  ...
  var finalReducerKeys = Object.keys(finalReducers);

  var unexpectedKeyCache = void 0;
  if (process.env.NODE_ENV !== 'production') {
    unexpectedKeyCache = {};
  }

  var shapeAssertionError = void 0;
  try {
    assertReducerShape(finalReducers);
  } catch (e) {
    shapeAssertionError = e;
  }
}
```
之后会定义一个finalReducerKeys来存储finalReducers中的键。在之后会调用一个叫assertReducerShape的函数，并把reducers传进去。

```apple js
function assertReducerShape(reducers) {
  Object.keys(reducers).forEach(function (key) {
    var reducer = reducers[key];
    var initialState = reducer(undefined, { type: ActionTypes.INIT });

    if (typeof initialState === 'undefined') {
      throw new Error('Reducer "' + key + '" returned undefined during initialization. ' + 'If the state passed to the reducer is undefined, you must ' + 'explicitly return the initial state. The initial state may ' + 'not be undefined. If you don\'t want to set a value for this reducer, ' + 'you can use null instead of undefined.');
    }

    var type = '@@redux/PROBE_UNKNOWN_ACTION_' + Math.random().toString(36).substring(7).split('').join('.');
    if (typeof reducer(undefined, { type: type }) === 'undefined') {
      throw new Error('Reducer "' + key + '" returned undefined when probed with a random type. ' + ('Don\'t try to handle ' + ActionTypes.INIT + ' or other actions in "redux/*" ') + 'namespace. They are considered private. Instead, you must return the ' + 'current state for any unknown actions, unless it is undefined, ' + 'in which case you must return the initial state, regardless of the ' + 'action type. The initial state may not be undefined, but can be null.');
    }
  });
}
```
assertReducerShape函数是用来判断传入的reducers中的reducer是否都是合法的reducer，如果有不符合标准的就会报出相应的错误。

最后来看combineReducers的返回值
```apple js
function combination() {
    var state = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : {};
    var action = arguments[1];

    if (shapeAssertionError) {
      throw shapeAssertionError;
    }

    if (process.env.NODE_ENV !== 'production') {
      var warningMessage = getUnexpectedStateShapeWarningMessage(state, finalReducers, action, unexpectedKeyCache);
      if (warningMessage) {
        warning(warningMessage);
      }
    }

    var hasChanged = false;
    var nextState = {};
    for (var _i = 0; _i < finalReducerKeys.length; _i++) {
      var _key = finalReducerKeys[_i];
      var reducer = finalReducers[_key];
      var previousStateForKey = state[_key];
      var nextStateForKey = reducer(previousStateForKey, action);
      if (typeof nextStateForKey === 'undefined') {
        var errorMessage = getUndefinedStateErrorMessage(_key, action);
        throw new Error(errorMessage);
      }
      nextState[_key] = nextStateForKey;
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey;
    }
    return hasChanged ? nextState : state;
}
```
返回的是一个叫combination的函数，这个函数是一个标准的reducer函数。

combination函数的第二个参数是一个state，先定义一个变量存储起来。
定义一个变量nextState存储一个空对象，作为新的reducers的容器。
之后回去对传入的reducers进行循环。在循环中会进行如下操作：
1. 定义一个_key来存储当前key。
2. 定义一个reducer存储当前reducer。
3. 定义一个previousStateForKey存储state。
4. 定义一个nextStateForKey存储初始化的state。
5. 如果初始化的state为undefined，就报出错误。
6. 把当前的state nextStateForKey存储到nextState中以_key为键。
7. 用hasChanged来标记state是否改变。

最后如果有改变就返回新的state否则返回原始的state。

combineReducers把多个reducer合并成一个reducers并生成一个state。

