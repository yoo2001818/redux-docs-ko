# 단어장

이 문서는 Redux에서 중요한 단어들을 타입과 함께 설명해 놓은 단어장입니다. 
타입들은 [Flow notation](http://flowtype.org/docs/quick-reference.html)를 사용하여 작성되었습니다.

## State

```js
type State = any;
```

*State*(*state tree* 라고도 불림)는 포괄적인 단어이지만, Redux API에서는 Store가 관리하고 [`getState()`](api/Store.md#getState)로 반환되는 하나의 상태 값을 의미합니다. State는 Redux 앱의 전체 상태를 표현합니다. 이는 주로 깊게 가지가 뻗어있는 오브젝트입니다.

일반적으로 최상위 state는 오브젝트나 맵과 같은 key-value 콜렉션이지만, 사실 아무 타입이나 들어갈 수 있습니다. 그래도, state를 직렬화 가능하게 하는 것이 권장됩니다. JSON으로 쉽게 바꿀 수 없는 객체들은 안에 집어넣지 마세요.

## Action

```js
type Action = Object;
```

*Action*은 state를 변경하기 위한 행동을 표현하는 plain object입니다. Action은 store에 데이터를 집어넣기 위한 유일한 방법입니다. UI에서 오든, 네트워크 콜백이든, WebSocket과 같은 다른 곳에서 오든, 모든 데이터는 Action으로써 dispatch되어야 합니다.

일반적으로, Action은 현재 실행되고 있는 action의 타입을 표시하는 `type` 필드가 있어야 합니다.
타입은 상수로써 정의될 수 있고 다른 모듈이 임포트해 갈 수 있습니다. `type`에는 [Symbol](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol)보다는 문자열이 바로 직렬화 될 수 있기 때문에 권장됩니다.

`type`을 제외한, Action의 다른 부분은 개발자 맘대로 정의할 수 있습니다. 
[Flux Standard Action](https://github.com/acdlite/flux-standard-action)은 Action이 어떻게 만들어져야 하는지 권장 사항을 나타냅니다.

아래의 [비동기 action](#async-action) 도 참조하세요.

## Reducer

```js
type Reducer<S, A> = (state: S, action: A) => S;
```

*Reducer*(*reducing 함수*라고도 불림)는 컬렉션과 값을 받아서 새 컬렉션을 반환하는 함수입니다. 배열의 원소들을 하나의 값으로 줄이는데 사용됩니다.

Reducer는 Redux에서 처음 도입하는 개념이 아닙니다. 함수형 프로그래밍에서의 기본적인 컨셉입니다. 함수형 프로그래밍 언어가 아니어도 대부분의 언어는, 예를 들면 JavaScript는 reducing을 위한 빌트인 API가 존재합니다. 자바스크립트에서는 [`Array.prototype.reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)입니다.

Redux에서는, 컬렉션이 State 객체입니다. 그리고 들어가는 값은 Action입니다. Reducer는 이전 state와 action을 보고 새 state를 계산합니다. 이 Reducer는 순수한 함수(pure function)여야 합니다- 입력을 받고 완전히 같은 출력을 내보내는 함수입니다. 또 이 함수는 부작용을 일으키면 안됩니다. (역주: 입력되는 객체들을 바꾸면 안됩니다) 이는 시간 되돌리기나 실시간 리로딩(hot reloading) 같은 것을 구현할 수 있게 하는 요소입니다.

Reducer는 Redux에서 가장 중요한 컨셉입니다.

*Reducer 안에서 API를 호출하지 마세요.*

## Dispatching Function

```js
type BaseDispatch = (a: Action) => Action;
type Dispatch = (a: Action | AsyncAction) => any;
```

*dispatching function*(아니면 간단하게 *dispatch function*)은 Action이나 [비동기 action](#async-action)을 받습니다. 그 후 이 함수는 Store에 1개 이상의 Action을 dispatch하거나 하지 않을 수도 있습니다.

일반적인 Dispatching function과 Middleware가 하나도 적용되지 않은 Store가 제공하는 기본 [`dispatch`](api/Store.md#dispatch) 함수는 구분되어야 합니다.

기본 dispatch 함수는 *항상* 계산하기 위해서 Action을 Store의 Reducer를 바로 전 State와 함께 보냅니다. 이 함수가 받는 Action은 Reducer가 바로 소비할 수 있는 plain object입니다.

[Middleware](#middleware)는 이 기본 dispatch 함수를 둘러쌉니다. 이는 dispatch 함수가 [비동기 action](#async-action)도 처리할 수 있게 합니다. Middleware는 다음 Middleware로Action이나 비동기 Action이 전달되게 전에 변형하거나, 지연시키거나 무시하거나 해석할 수 있게 합니다. 자세한 사항은 아래를 참고해 주세요.

## Action 생성자

```js
type ActionCreator = (...args: any) => Action | AsyncAction;
```

*Action 생성자*는, 간단하게 말하자면 새 Action을 생성하는 함수입니다. 헷갈리지 마세요: Action은 정보를 둘러싼 객체이고, Action 생성자는 Action을 생성하는 Factory입니다.

Action 생성자를 호출하면 Action을 생성하기는 하지만 이를 dispatch하지는 않습니다. 실제로 State를 변형하기 위해서는 Store의 [`dispatch`](api/Store.md#dispatch) 함수를 호출해야 합니다. Action 생성자를 호출하고 결과값을 곧바로 dispatch하는 함수를 우리는 가끔 *bound action creator*라고 부릅니다.

만약 Action 생성자가 현재 State를 읽거나, API를 호출하거나, 부작용(예를 들면routing transition - 리다이렉트)를 발생시켜야 한다면, 함수는 Action 대신 [비동기 Action](#async-action)를 반환해야 합니다.

## 비동기 Action

```js
type AsyncAction = any;
```

An *async action* is a value that is sent to a dispatching function, but is not yet ready for consumption by the reducer. It will be transformed by [middleware](#middleware) into an action (or a series of actions) before being sent to the base [`dispatch()`](api/Store.md#dispatch) function. Async actions may have different types, depending on the middleware you use. They are often asynchronous primitives, like a Promise or a thunk, which are not passed to the reducer immediately, but trigger action dispatches once an operation has completed.

## Middleware

```js
type MiddlewareAPI = { dispatch: Dispatch, getState: () => State };
type Middleware = (api: MiddlewareAPI) => (next: Dispatch) => Dispatch;
```

A middleware is a higher-order function that composes a [dispatch function](#dispatching-function) to return a new dispatch function. It often turns [async actions](#async-action) into actions.

Middleware is composable using function composition. It is useful for logging actions, performing side effects like routing, or turning an asynchronous API call into a series of synchronous actions.

See [`applyMiddleware(...middlewares)`](./api/applyMiddleware.md) for a detailed look at middleware.

## Store

```js
type Store = {
  dispatch: Dispatch;
  getState: () => State;
  subscribe: (listener: () => void) => () => void;
  getReducer: () => Reducer;
  replaceReducer: (reducer: Reducer) => void;
};
```

A store is an object that holds the application’s state tree.  
There should only be a single store in a Redux app, as the composition happens on the reducer level.

- [`dispatch(action)`](api/Store.md#dispatch) is the base dispatch function described above.
- [`getState()`](api/Store.md#getState) returns the current state of the store.
- [`subscribe(listener)`](api/Store.md#subscribe) registers a function to be called on state changes.
- [`getReducer()`](api/Store.md#getReducer) and [`replaceReducer(nextReducer)`](api/Store.md#replaceReducer) can be used to implement hot reloading and code splitting. Most likely you won’t use them.

See the complete [store API reference](api/Store.md#dispatch) for more details.

## Store creator

```js
type StoreCreator = (reducer: Reducer, initialState: ?State) => Store;
```

A store creator is a function that creates a Redux store. Like with dispatching function, we must distinguish the base store creator, [`createStore(reducer, initialState)`](api/createStore.md) exported from the Redux package, from store creators that are returned from the store enhancers.

## Store enhancer

```js
type StoreEnhancer = (next: StoreCreator) => StoreCreator;
```

A store enhancer is a higher-order function that composes a store creator to return a new, enhanced store creator. This is similar to middleware in that it allows you to alter the store interface in a composable way.

Store enhancers are much the same concept as higher-order components in React, which are also occasionally called “component enhancers”.

Because a store is not an instance, but rather a plain-object collection of functions, copies can be easily created and modified without mutating the original store. There is an example in [`compose`](api/compose.md) documentation demonstrating that.

Most likely you’ll never write a store enhancer, but you may use the one provided by the [developer tools](https://github.com/gaearon/redux-devtools). It is what makes time travel possible without the app being aware it is happening. Amusingly, the [Redux middleware implementation](api/applyMiddleware.md) is itself a store enhancer.
