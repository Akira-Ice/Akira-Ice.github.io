---
title: ReduxNote
date: 2023/7/28 10:43:00
updated: 2023/7/31
categories:
  - React
tags:
---

Redux 是 JavaScript 应用的状态容器，提供可预测的状态管理。

# 快速入门

工具包：[配置 store](https://redux-toolkit.js.org/api/configureStore)、[创建 Reducer 并编写 immutable 更新逻辑](https://redux-toolkit.js.org/api/createreducer)、[创建整个 State 的 “slice”](https://redux-toolkit.js.org/api/createslice)

## 安装

```shell
# Toolkit - 1
yarn add @reduxjs/toolkit

# Redux core lib - 2
yarn add redux

# create React Redux app

# Redux + Plain JS template
npx create-react-app my-app --template redux

# Redux + TypeScript template
npx create-react-app my-app --template redux-typescript

```

## 基础示例

```jsx
/** redux */
import { createStore } from "redux";

/** reducer*/
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case "counter/incremented":
      return { value: state.value + 1 };
    case "counter/decremented":
      return { value: state.value - 1 };
    default:
      return state;
  }
}

/** createStore */
let store = createStore(counterReducer);

/** subscribe */
store.subscribe(() => console.log(store.getState()));

/** dispatch */
store.dispatch({ type: "counter/incremented" });
store.dispatch({ type: "counter/incremented" });
store.dispatch({ type: "counter/decremented" });

/** redux-toolkit */
/** 1. createSlice -> { reducers, actions } */
import { createSlice } from "@reduxjs/toolkit";

export const counterSlice = createSlice({
  name: "counter",
  initialState: {
    value: 0,
  },
  /** reducers */
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

/** actions */
export const { increment, decrement, incrementByAmount } = counterSlice.actions;

export const incrementAsync = (amount) => (dispatch) => {
  setTimeout(() => {
    dispatch(incrementByAmount(amount));
  }, 1000);
};

/** getter */
export const selectCount = (state) => state.counter.value;

export default counterSlice.reducer;

/** 2. createStore */
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from '../features/counter/counterSlice';

export default configureStore({
  reducer: {
    counter: counterReducer,
  },
});

/** useStore */
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
/** store & Provider */
import store from './app/store';
import { Provider } from 'react-redux';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);

/** use in component */
import React, { useState } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import {
  decrement,
  increment,
  incrementByAmount,
  incrementAsync,
  selectCount,
} from './counterSlice';

export function Counter() {
  const count = useSelector(selectCount);
  const dispatch = useDispatch();

  return (
    <div>
      <button onClick={() => { dispatch(increment())}}></button>
    </div>
  )
}
```

# 创建 Redux Store

`configureStore({ reducer: {} }) -> action`

reducer 以键值对的形式存储，各个功能模块的 reducer 以及 state。

```js
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "../features/counter/counterSlice";

export default configureStore({
  reducer: {
    counter: counterReducer,
  },
});
```

# 创建 Redux Slice

**“slice” 是应用中单个功能的 Redux reducer 逻辑和 action 的集合**, 通常一起定义在一个文件中。该名称来自于将根 Redux 状态对象拆分为多个状态 “slice”。

`toolkit -> createSlice -> { reducers, action_string, action_createFunction, action_object }`

```js
import { createSlice } from "@reduxjs/toolkit";

export const counterSlice = createSlice({
  name: "counter",
  initialState: {
    value: 0,
  },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;

export default counterSlice.reducer;
```

原则上不能在 Redux 中直接修改 state，但 toolkit 库集成了 immer.js 进而简化覆写过程。

## Thunk 异步

thunk 是一种特定类型的 Redux 函数，可以包含异步逻辑。Thunk 是使用两个函数编写的：

1. 一个内部 thunk 函数，它以 dispatch 和 getState 作为参数

2. 外部创建者函数，它创建并返回 thunk 函数

```js
export const incrementAsync = (amount) => (dispatch) => {
  setTimeout(() => {
    dispatch(incrementByAmount(amount));
  }, 1000);
};
```

## prepare 构造 action

可以使用一个正确的内容模板去构造（prepare）action 对象，例如添加文章时需要添加唯一的 id，通过 prepare 可在 slice 中进行添加，不需要外部再做处理。

```js
/** slice */
const postsSlice = createSlice({
  name: "posts",
  initialState,
  reducers: {
    postAdded: {
      reducer(state, action) {
        state.push(action.payload);
      },
      prepare(title, content) {
        return {
          payload: {
            id: nanoid(),
            title,
            content,
          },
        };
      },
    },
    // other reducers here
  },
});

/** xxComponent */
const onSavePostClicked = () => {
  if (title && content) {
    dispatch(postAdded(title, content));
    setTitle("");
    setContent("");
  }
};
```

## extraReducers

extraReducers 选项是一个接收名为 builder 的参数的函数。

builder 对象提供了一些方法，让我们可以定义额外的 case reducer，这些 reducer 将响应在 slice 之外定义的 action。

我们将使用 builder.addCase(actionCreator, reducer) 来处理异步 thunk dispatch 的每个 action。

```js
export const fetchPosts = createAsyncThunk("posts/fetchPosts", async () => {
  const response = await client.get("/fakeApi/posts");
  return response.data;
});

const postsSlice = createSlice({
  name: "posts",
  initialState,
  reducers: {
    // omit existing reducers here
  },
  extraReducers(builder) {
    builder
      .addCase(fetchPosts.pending, (state, action) => {
        state.status = "loading";
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.status = "succeeded";
        // Add any fetched posts to the array
        state.posts = state.posts.concat(action.payload);
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.status = "failed";
        state.error = action.error.message;
      });
  },
});
```

# 全局注入 stote

任何调用 useSelector 或 useDispatch 的 React 组件都可以访问 `<Provider>` 中的 store。

```js
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import store from "./app/store";
import { Provider } from "react-redux";
import * as serviceWorker from "./serviceWorker";

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);
```

# 接收 store & dispatch

接收 store 以及 dispatch 的有两种方式：Hooks API、Connect API。

> connect still works and is supported in React-Redux 8.x. However, [we recommend using the hooks API as the default](https://react-redux.js.org/api/hooks).

## Hooks API

Hooks 方式直接通过 useDispatch 和 useSelector 获取 state 以及 dispatch。

**useDispatch & useSelector :**

React 组件使用 useSelector 钩子从 store 读取数据

1. 选择器函数接收整个 state 对象，并且返回需要的部分数据

2. 每当 Redux store 更新时，选择器将重新运行，如果它们返回的数据发生更改，则组件将重新渲染

React 组件使用 useDispatch 钩子 dispatch action 来更新 store

1. createSlice 将为我们添加到 slice 的每个 reducer 函数生成 action creator 函数

2. 在组件中调用 dispatch(someActionCreator()) 来 dispatch action

3. Reducers 将运行，检查此 action 是否相关，并在适当时返回新状态

4. 表单输入值等临时数据应保留为 React 组件状态。当用户完成表单时，dispatch 一个 Redux action 来更新 store。

```jsx
import { useSelector, useDispatch } from "react-redux";

const selectCount = (state) => state.counter.value;

export default function App() {
  const counter = useSelector(selectCount);
  const dispatch = useDispatch();

  return (
    <>
      <div
        onClick={() => {
          dispatch({ type: "xxx" });
        }}
      >
        {counter}
      </div>
    </>
  );
}
```

## Connect API

通过 Connect 函数将 store 以及 dispatch 注入到组件的 props 中。

`function connect(mapStateToProps?, mapDispatchToProps?, mergeProps?, options?)`

参数：

1. mapStateToProps?: Function

2. mapDispatchToProps?: Function | Object

3. mergeProps?: Function

4. options?: Object

> 1. mapStateToProps -> 传递 state 到 props，需要返回一个对象用于合并到 props 中。
>
> 2. mapDispatchToPorps -> 传递 dispatch 到 props（通常不需要传递，redux 内部会自动回传一个 dispatch 到 props）。

```jsx
import { connect } from "react-redux";

function App(props) {
  return (
    <>
      <div
        onClick={() => {
          porps.dispatch({ type: "xxx" });
        }}
      >
        {props.state}
      </div>
    </>
  );
}

function mapStateToPorps(state) {
  return state;
}

export default connect(mapStateToPorps)(App);
```

# Middleware

Redux 有多种异步 middleware，每一种都允许你使用不同的语法编写逻辑。最常见的异步 middleware 是 redux-thunk，它可以让你编写可能直接包含异步逻辑的普通函数。

![](https://cn.redux.js.org/assets/images/ReduxAsyncDataFlowDiagram-d97ff38a0f4da0f327163170ccc13e80.gif)

Redux 使用叫做“ middleware ”这样的插件模式来开发异步逻辑

1. 官方的处理异步 middleware 叫 redux-thunk，包含在 Redux Toolkit 中

2. Thunk 函数接收 dispatch 和 getState 作为参数，并且可以在异步逻辑中使用它们

## thunk

thunk 接收 dispatch 和 getState 作为参数，可进行再次 dispatch。

```js
/** ReduxSlice */
import { createSlice } from "@reduxjs/toolkit";

export const counterSlice = createSlice({
  name: "counter",
  initialState: {
    value: 0,
  },
  reducers: {
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

export const { incrementByAmount } = counterSlice.actions;

/** thunk */
export const incrementAsync = (amount) => (dispatch) => {
  setTimeout(() => {
    dispatch(incrementByAmount(amount));
  }, 1000);
};

export default counterSlice.reducer;
```

## createAsyncThunk

`createAsyncThunk(actionString: string, callback: () => Promise)`

thunk 生成器。

```js
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";
import { client } from "../../api/client";

const initialState = {
  posts: [],
  status: "idle",
  error: null,
};

export const fetchPosts = createAsyncThunk("posts/fetchPosts", async () => {
  const response = await client.get("/fakeApi/posts");
  return response.data;
});

export const addNewPost = createAsyncThunk("posts/addNewPost", async (initialPost) => {
  const response = await client.post("/fakeApi/posts", initialPost);
  return response.data;
});

const postsSlice = createSlice({
  name: "posts",
  initialState,
  reducers: {
    // omit other reducer
  },
  extraReducers(builder) {
    builder
      .addCase(fetchPosts.pending, (state, action) => {
        state.status = "loading";
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.status = "succeeded";
        // Add any fetched posts to the array
        state.posts = state.posts.concat(action.payload);
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.status = "failed";
        state.error = action.error.message;
      })
      .addCase(addNewPost.fulfilled, (state, action) => {
        state.posts.push(action.payload);
      });
  },
});

export default postsSlice.reducer;
```

# 原则

1. 必须依赖 state 和 action 参数去计算出一个新 state

2. 必须通过拷贝旧 state 的方式去做 不可变更新 (immutable updates)

3. 不能包含任何异步逻辑或其他副作用

4. Redux Toolkit 的 createSlice API 内部使用了 Immer 库才达到表面上直接修改（"mutating"）state 也实现不可变更新（immutable updates）的效果

> Redux action 和 state 应该只能包含普通的 JS 值，如对象、数组和基本类型。不要将类实例、函数或其他不可序列化的值放入 Redux！。

# 性能优化

[性能与数据范式化](https://cn.redux.js.org/tutorials/essentials/part-6-performance-normalization)

1. 记忆化 selector

2. meno()

3. 范式化数据

## 记忆化 Selector

通过 createSelector 创建记忆化 selector。

接受多个 selector（前面称作’输入 selector‘，最后一个称作’输出 selector‘），最后一个作为输出 selector，进而创建记忆化 selector。

在使用 useSelector 时，会将所有的参数传递给所有的’输入 selector‘，最后将所有的’输入 selector‘的返回值作为参数传入’输出 selector‘，进而只对最后的参数进行依赖追踪。

```js
import { createSelector } from "@reduxjs/toolkit";

const shopItemsSelector = (state) => state.shop.items;
const taxPercentSelector = (state) => state.shop.taxPercent;

const subtotalSelector = createSelector(shopItemsSelector, (items) => items.reduce((subtotal, item) => subtotal + item.value, 0));

const taxSelector = createSelector(subtotalSelector, taxPercentSelector, (subtotal, taxPercent) => subtotal * (taxPercent / 100));

const totalSelector = createSelector(subtotalSelector, taxSelector, (subtotal, tax) => ({ total: subtotal + tax }));

const exampleState = {
  shop: {
    taxPercent: 8,
    items: [
      { name: "apple", value: 1.2 },
      { name: "orange", value: 0.95 },
    ],
  },
};

console.log(subtotalSelector(exampleState)); // 2.15
console.log(taxSelector(exampleState)); // 0.172
console.log(totalSelector(exampleState)); // { total: 2.322 }
```

## 范式化数据

“范式化数据”是指：

1. 我们 state 中的每个特定数据只有一个副本，不存在重复。

2. 已范式化的数据保存在查找表中，其中项目 ID 是键，项本身是值。

3. 也可能有一个特定项用于保存所有 ID 的数组。

```js
{
  users: {
    ids: ["user1", "user2", "user3"],
    entities: {
      "user1": {id: "user1", firstName, lastName},
      "user2": {id: "user2", firstName, lastName},
      "user3": {id: "user3", firstName, lastName},
    }
  }
}
```

Redux-Toolkit 提供 [createEntityAdapter](https://cn.redux.js.org/tutorials/essentials/part-6-performance-normalization#%E4%BD%BF%E7%94%A8-createentityadapter-%E7%AE%A1%E7%90%86%E8%8C%83%E5%BC%8F%E5%8C%96-state) 进行范式化管理 state 数据。

# Redux 内部 Store

实际逻辑：

1. store 内部有当前的 state 值和 reducer 函数

2. getState 返回当前 state 值

3. subscribe 保存一个监听回调数组并返回一个函数来移除新的回调

4. dispatch 调用 reducer，保存 state，并运行监听器

5. store 在启动时 dispatch 一个 action 来初始化 reducers 的 state

6. store API 是一个对象，里面有 {dispatch, subscribe, getState}

```js
function createStore(reducer, preloadedState) {
  let state = preloadedState;
  const listeners = [];

  function getState() {
    return state;
  }

  function subscribe(listener) {
    listeners.push(listener);
    return function unsubscribe() {
      const index = listeners.indexOf(listener);
      listeners.splice(index, 1);
    };
  }

  function dispatch(action) {
    state = reducer(state, action);
    listeners.forEach((listener) => listener());
  }

  dispatch({ type: "@@redux/INIT" });

  return { dispatch, subscribe, getState };
}
```
