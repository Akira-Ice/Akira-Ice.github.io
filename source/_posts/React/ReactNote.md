---
title: React Note
date: 2023/7/20
updated: 2023/7/25
categories: 
  - React
tags: 
---

# ReactNote

## handleEvent

副作用函数

**stopEvent**

react 更贴近原生 JS。

阻止事件传播：

1. 阻止事件冒泡：`e.stopPropagation()`

2. 阻止原生事件：`e.preventDefault()`

## state

**特性**：异步更新、合并（**批处理**）、顶层调用。

**异步更新**：无论是否调用更新函数，当前作用域拿到的 state 值永远是当前的快照值。

**React 会使 state 的值始终”固定“在一次渲染的各个事件处理函数内部。**

### useState

`useState<T>(initState: T): [state: T, setState: (state: T) => void]`

**setState(callback) 更新函数**：当传递的是函数时，React 会将此函数加入队列。

**setState(n+1) 值覆盖**：拿到 n 的值始终是当前组件的快照值。

> 注意：状态值在调用更新函数时始终变化，而当前组件快照值始终不变。

### update queue

**更新队列：** 状态值、更新函数，最终更新结果从 useState 中返回出去。

```js
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  for (let update of queue) {
    if (typeof update === 'function') {
      // 调用更新函数
      finalState = update(finalState);
    } else {
      // 替换下一个 state
      finalState = update;
    }
  }

  return finalState;
}
```

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
      let k=0;
        setNumber(number + 5);
        setNumber(n => {alert(k++);return n+1});// 这里的alert会执行两次
      }}>增加数字</button>
    </>
  )
}
```

alert 执行两次原因：**浏览器严格模式导致的**。（只会在DEV环境下）

![](https://s2.loli.net/2023/07/25/TfFjse3wzObItoh.png)

**React 会等到事件处理函数中的** 所有 **代码都运行完毕再处理你的 state 更新。**

内部会有一个数组存储组件的所有 state 数据 以及 setState 更新函数。

### useState simulation

**`useState `模拟**：

```js
let componentHooks = [];
let currentHookIndex = 0;

// useState 在 React 中是如何工作的（简化版）
function useState(initialState) {
  let pair = componentHooks[currentHookIndex];
  if (pair) {
    // 这不是第一次渲染
    // 所以 state pair 已经存在
    // 将其返回并为下一次 hook 的调用做准备
    currentHookIndex++;
    return pair;
  }

  // 这是我们第一次进行渲染
  // 所以新建一个 state pair 然后存储它
  pair = [initialState, setState];

  function setState(nextState) {
    // 当用户发起 state 的变更，
    // 把新的值放入 pair 中
    pair[0] = nextState;
    updateDOM();
  }

  // 存储这个 pair 用于将来的渲染
  // 并且为下一次 hook 的调用做准备
  componentHooks[currentHookIndex] = pair;
  currentHookIndex++;
  return pair;
}

function Gallery() {
  // 每次调用 useState() 都会得到新的 pair
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  // 这个例子没有使用 React，所以
  // 返回一个对象而不是 JSX
  return {
    onNextClick: handleNextClick,
    onMoreClick: handleMoreClick,
    header: `${sculpture.name} by ${sculpture.artist}`,
    counter: `${index + 1} of ${sculptureList.length}`,
    more: `${showMore ? 'Hide' : 'Show'} details`,
    description: showMore ? sculpture.description : null,
    imageSrc: sculpture.url,
    imageAlt: sculpture.alt
  };
}

function updateDOM() {
  // 在渲染组件之前
  // 重置当前 Hook 的下标
  currentHookIndex = 0;
  let output = Gallery();

  // 更新 DOM 以匹配输出结果
  // 这部分工作由 React 为你完成
  nextButton.onclick = output.onNextClick;
  header.textContent = output.header;
  moreButton.onclick = output.onMoreClick;
  moreButton.textContent = output.more;
  image.src = output.imageSrc;
  image.alt = output.imageAlt;
  if (output.description !== null) {
    description.textContent = output.description;
    description.style.display = '';
  } else {
    description.style.display = 'none';
  }
}

let nextButton = document.getElementById('nextButton');
let header = document.getElementById('header');
let moreButton = document.getElementById('moreButton');
let description = document.getElementById('description');
let image = document.getElementById('image');
let sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Although Colvin is predominantly known for abstract themes that allude to pre-Hispanic symbols, this gigantic sculpture, an homage to neurosurgery, is one of her most recognizable public art pieces.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'A bronze statue of two crossed hands delicately holding a human brain in their fingertips.'  
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: 'This enormous (75 ft. or 23m) silver flower is located in Buenos Aires. It is designed to move, closing its petals in the evening or when strong winds blow and opening them in the morning.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'A gigantic metallic flower sculpture with reflective mirror-like petals and strong stamens.'
}];

// 使 UI 匹配当前 state
updateDOM();
```

`setState -> updateDOM -> render`

`snapshop -> setState -> render -> new snapshop`

`reRender: reCall the Function -> return new JSX -> diff -> patch -> react update UI`

### object update

**在 React 中将所有的数据视为 不可变数据 或 只读 。**

尤其是**引用类型**。

**对象展开 & 覆盖**：

```js
setPerson({
  ...person, // 复制上一个 person 中的所有字段
  firstName: e.target.value // 但是覆盖 firstName 字段 
});
```

**嵌套对象**：

更新嵌套对象，需要多次展开。

```js
const nextArtwork = { ...person.artwork, city: 'New Delhi' };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);

/** or */

setPerson({
  ...person, // 复制其它字段的数据 
  artwork: { // 替换 artwork 字段 
    ...person.artwork, // 复制之前 person.artwork 中的数据
    city: 'New Delhi' // 但是将 city 的值替换为 New Delhi！
  }
});
```

### array update

|     | change arr         | return new arr  |
| --- | ------------------ | --------------- |
| add | push、unshift       | concat、[...arr] |
| del | pop、shift、splice   | filter、slice    |
| rep | splice、arr[i] = xx | map             |
| sor | reverse、sort       | clone and sort  |

同对象一样，也要看做不可变数据。

数组中的对象，实际上存储的也是对象的地址，所以直接修改也是不可以的，可以使用 map 生成新的数组进行更新。

```js
setMyList(myList.map(artwork => {
  if (artwork.id === artworkId) {
    // 创建包含变更的*新*对象
    return { ...artwork, seen: nextSeen };
  } else {
    // 没有变更
    return artwork; 
 }
}));
updateMyTodos(draft => {
  const artwork = draft.find(a => a.id === artworkId);
  artwork.seen = nextSeen;
});
```

### Immer

**Immer 简化更新**：

```js
updatePerson(draft => {
  draft.artwork.city = 'Lagos';
});
```

> 由 Immer 提供的 `draft` 是一种特殊类型的对象，被称为 [Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)，它会记录你用它所进行的操作。这就是你能够随心所欲地直接修改对象的原因所在！从原理上说，Immer 会弄清楚 `draft` 对象的哪些部分被改变了，并会依照你的修改创建出一个全新的对象。

## render

`createRoot -> render`

**React 仅在渲染之间存在差异时才会更改 DOM 节点。**

## stateManage

**核心**：`UI = fn(state)`

**避免冗余、矛盾、重复、深度嵌套的 state**。

**UI树：**

<img src="https://react.docschina.org/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_dom_tree.png&w=1920&q=75" title="" alt="Diagram with three sections arranged horizontally. In the first section, there are three rectangles stacked vertically, with labels 'Component A', 'Component B', and 'Component C'. Transitioning to the next pane is an arrow with the React logo on top labeled 'React'. The middle section contains a tree of components, with the root labeled 'A' and two children labeled 'B' and 'C'. The next section is again transitioned using an arrow with the React logo on top labeled 'React'. The third and final section is a wireframe of a browser, containing a tree of 8 nodes, which has only a subset highlighted (indicating the subtree from the middle section)." data-align="inline">

> **对 React 来说重要的是组件在 UI 树中的位置,而不是在 JSX 中的位置！**

**所以如果希望在重新渲染的时候保留 state 需要保证几次渲染的树结构中应该互相“匹配”**

基本上一些 ui 上的 bug 都是渲染出错导致，主要检查 key 值的设置。

### useReducer

纯函数

`useReducer(callback: (state: any, action: any) => void, initState: any): [state: any, dispatch: () => void];`

### useImmerReducer

```js
import { useImmerReducer } from 'use-immer';

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false,
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('未知 action：' + action.type);
    }
  }
}
```

## Context

类似 Vue 中的 provide/injext，主要用来解决 props 逐级透传问题。

react 作为单向数据流，数据来自上层，但如果**状态提升**到太高的层级就会导致“逐层透传props”的情况。

![image.png](https://s2.loli.net/2023/07/25/23Tdlw8hItBMJAV.png)

context 则是先下注入数据，不论深度，但存在**就近原则**：

![image.png](https://s2.loli.net/2023/07/25/a8Vk2WGdfAgQTem.png)

`createContext(value) -> <XxxContext.Provider> -> useContext(XxxContext)`

```js
/** createContext */
import { createContext } from 'react';
export const LevelContext = createContext(1);

/** XxxContext.Provider */
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}

/** useContext */
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    default:
      throw Error('未知的 level：' + level);
  }
}
```

应用场景：颜色主题、全局数据共享、局部数据向下透传。

Context 不局限于静态值。如果你在下一次渲染时传递不同的值，React 将会更新读取它的所有下层组件！这就是 context 经常和 state 结合使用的原因。

**技巧：**

1. 当文件中存在多个 context 时，可进行整合并声明新的组件（**注意预留 children**）。

2. 将相关逻辑迁移到一个文件当中，生产自定义 Hook：`createContext -> useContext -> useHook`

### Reduce & Context

通过 Context 将 Reduce 的 state 以及 dispatch 分发到子孙组件中。

```js
/** App.js */
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}

/** TaskContext.js */
import { createContext, useContext, useReducer } from 'react';

const TasksContext = createContext(null);
const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

// useHooks
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Visit the temple', done: false },
  { id: 2, text: 'Drink matcha', done: false }
];

```

## Ref

`useRef<T>(initState: T): { curretn: T}`

数据由 React 保存，与 state 不同的是，ref 更变不触发 render 。

| ref                                         | state                                                                 |
|:-------------------------------------------:|:---------------------------------------------------------------------:|
| `useRef<T>(initialValue: T): { current: T}` | `useState<T>(initState: T): [state: T, setState: (state: T) => void]` |
| 更改时不会触发重新渲染                                 | 更改时触发重新渲染。                                                            |
| 可变 —— 你可以在渲染过程之外修改和更新 current 的值。           | “不可变” —— 你必须使用 state 设置函数来修改 state 变量，从而排队重新渲染。                       |
| 你不应在渲染期间读取（或写入） current 值。                  | 你不应在渲染期间读取（或写入） current 值。                                            |

**使用场景：**

1. timeoutID

2. DOM

3. JSX

### get DOM

1. 字符串形式（回调形式简写，接收 DOM 赋值给 ref.current）

2. 回调函数形式

```js
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);
  const buttonRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick} ref={c => buttonRef.current = c}>
        聚焦输入框
      </button>
    </>
  );
}
```

**多 ref 管理：**

1. ref 绑到父层，通过 [querySelectorAll](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/querySelectorAll) 获取，但 DOM 结构发生变化，可能会失效。

2. 通过 ref 回调的形式存储多个 DOM 至 map 中，map 也是需要通过 ref 保存。

```js
import { useRef } from "react";

const list = [];
export default function Component() {
  const refs = useref(null);

  // Singal model
  function getRefs() {
    if (!refs.current) refs.current = new Map();
    return refs.current;
  }

  return (
    <>
      {list.map((item) => {
        <div
          ref={(node) => {
            const refs = getRefs();
            if (node) refs.set(item.id, node);
            else refs.delete(item.id);
          }}
        ></div>;
      })}
    </>
  );
}
```

### forwardRef

**仅原生 DOM，自定义组件不生效且报错**，但可以通过 `forwardRef` 将 `ref` 进行**透传**到内部 **DOM** 上，并返回给上层 ref 回调中。

```js
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}

```

### useImperativeHandle

主动返回 DOM 节点。

通过 useImperativeHandle 返回一个对象给上层 ref 回调中。

```js
import {
  forwardRef, 
  useRef, 
  useImperativeHandle
} from 'react';

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // 只暴露 focus，没有别的
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
    console.log(inputRef.current)
  }

  return (
    <>
      //<MyInput ref={c => inputRef.current = c} />
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}

```
