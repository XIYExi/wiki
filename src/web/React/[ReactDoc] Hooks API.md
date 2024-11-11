# [ReactDoc] React Hooks - API Reference

> [Hooks API Reference](https://legacy.reactjs.org/docs/hooks-reference.html) @ React Docs

## useState

> [useState](https://legacy.reactjs.org/docs/hooks-reference.html#usestate) @ React Docs - Hooks API Reference

<Alert type='info'>
特别留意：在 Function 中的 useState 和 Class 中的 setState 不同，useState的set function不会主动merge，
因此可以通过 {...preObject} 的方式复制完整的内容
</Alert>

### functional updater: 使用上一次数据

如果有需要的话，在```setCount```的函数中可以输入function， 这个function可以获取上一次count的状态：

```jsx | pure
const [foo, setFoo] = useState(initialFoo);
setFoo((prevState) => {/*do something with prevState*/})
```

使用范例：

```jsx | pure
function Counter({ initialCount }) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount((prevCount) => prevCount + 1)}>+</button>
      <button onClick={() => setCount((prevCount) => prevCount - 1)}>-</button>
    </>
  );
}
```

**在useEffect或useCallback中使用prevState避免状态依赖**

若我们在```setCount```中使用的是前一次的状态， 就可以把该状态从依赖矩阵（dependency array）中移除，
这么做的好处是页面会随时间重新渲染render，但是useEffect和里面的cleanup function并不会每次都被唤醒，如：

```jsx | pure
const Timer1 = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timerId = setInterval(() => {
      setCount(count + 1);
    }, 1000)
  }, [count])

  return(
    <div>{count}</div>
  )
}

const Timer2 = () => {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timerId = setInterval(() => {
      setCount(prevCount => prevCount + 1);
    }, 1000)
  }, [])
  
  return(
    <div>{count}</div>
  )
}
```

或参考 [如何錯誤地使用 React hooks useCallback 來保存相同的 function instance](https://as790726.medium.com/%E5%A6%82%E4%BD%95%E9%8C%AF%E8%AA%A4%E5%9C%B0%E4%BD%BF%E7%94%A8-react-hooks-usecallback-%E4%BE%86%E4%BF%9D%E5%AD%98%E7%9B%B8%E5%90%8C%E7%9A%84-function-instance-7744984bb0a6)

```jsx | pure
import React, { useState, useCallback, useRef } from 'react';
import ReactDOM from 'react-dom';
import './styles.css';

const Button = React.memo(({ handleClick }) => {
  const refCount = useRef(0);
  return <button onClick={handleClick}>{`button render count ${refCount.current++}`}</button>;
});

function App() {
  const [isOn, setIsOn] = useState(false);
  // 在 setIsOn 中帶入 prevState 可以避免把 state 或 props 放入相依陣列中
  const handleClick = useCallback(() => setIsOn((isOn) => !isOn), []);
  return (
    <div className="App">
      <h1>{isOn ? 'On' : 'Off'}</h1>
      <Button handleClick={handleClick} />
    </div>
  );
}

const rootElement = document.getElementById('root');
const root = ReactDOM.createRoot(rootElement);
root.render(<App />);
```


### Lazy initial state

> [Lazy initial state](https://legacy.reactjs.org/docs/hooks-reference.html#lazy-initial-state) @ React Docs - API - Reference
> [How to create expensive objects lazily?](https://legacy.reactjs.org/docs/hooks-faq.html#how-to-create-expensive-objects-lazily) @ Hooks FAQ

如果在```useState```内的预设值有些时候是需要经过函数处理的，则可以在```useState```的参数中带入函数：
```jsx | pure
// 沒使用 lazy initial state: someExpensiveComputation 每次 render 都會被呼叫
const [state, setState] = useState(someExpensiveComputation(props));

// 使用 lazy initial state: someExpensiveComputation 只會被呼叫一次
const [state, setState] = useState(() => someExpensiveComputation(props));
```

## useEffect vs. useLayoutEffect

简单来说```useEffect```会阻塞当前页面的转译（类似addEventListener中把```passive```设置成```false```),
需要等到useEffect中的js代码执行完成后，才会继续渲染页面；

而```useLayoutEffect```则不会阻塞页面的转译，页面会持续直接进行渲染，相当于render函数和useLayoutEffect中的代码是同步执行的。

> [useLayoutEffect](https://legacy.reactjs.org/docs/hooks-reference.html#uselayouteffect) @ React Docs


## useRef

> [useRef]() @ React Docs - Hooks API Reference

由于每次画面转译时的```state，props```都是独立的，因此若我们真的有需要保留某一次转译时的数据状态，则可以使用```useRef```。

```jsx | pure
const refContaine = useRef(<initialValue>);
```
```useRef```会回传有一个带有```.current```属性的物件，
这个```.current```是可以被改变的（mutable），同时会在该元件的完整生命周期中被保留。透过```useRef```的参数，可以把初始值带入```.current```属性中。

### 范例

```jsx | pure
// https://reactjs.org/docs/hooks-reference.html#useref
import { useRef } from 'react';

const TextInputWithFocusButton = () => {
  const inputRef = useRef < HTMLInputElement > null;

  const handleClick = () => {
    inputRef.current?.focus();
  };

  return (
    <>
      <input type="text" ref={inputRef} />
      <button type="button" onClick={handleClick}>
        Click To Focus
      </button>
    </>
  );
};

export default TextInputWithFocusButton;
```

### 保存元件中会变更的资料

> [Is there something like instance variables?](https://legacy.reactjs.org/docs/hooks-faq.html#is-there-something-like-instance-variables) @ React Docs - Hooks FAQ

在React元件中，只要state或者props变化，就会导致该元件re-render，如果希望有一些资料状态能在元件中被保存使用，
但不希望这些值改变时导致re-render，就可以使用```useRef```。

也就是说，通过```useRef```可以在元件中定义变量来使用：

- 不能直接在元件**中**直接使用```let```/```const```定义变量，因为每次元件re-render后，该变量都会被重置。
- 不能直接在元件**外**直接使用```let```/```const```定义变量，因为当很多地方使用这个元件时，他们都会参照同一个元件外的变量（[Why need useRef and not mutable variable?](https://stackoverflow.com/questions/57444154/why-need-useref-and-not-mutable-variable)）

但要注意的是，```useRef```在内部的状态改变的时候并不会发送通知，
也就是说，当```.current```属性改变时，并不会触发元件重新转译。如果你是希望
React附加（attach）或脱离（detach）某个DOM时去执行某些程序，
则应该使用[```callback ref```](https://legacy.reactjs.org/docs/hooks-faq.html#how-can-i-measure-a-dom-node)。

当我们没有使用```useRef```时，```${count}```的值会在每次该元件转译时就固定，因此会依序1，2，3，…呈现；当使用```useRef```后，可以把```count```的值存在外部一个可以该元件参照的物件，因此在最后```setTimeout```呼叫时，拿到的都会是最后一次点击counter的值，因此只会显示最后的数字：

```jsx | pure
import React, { useState, useEffect, useRef } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  // 建立一個 ref，預設值為 count，也就是 0
  const latestCount = useRef(count);

  useEffect(() => {
    // 改變 latestCount.current 的值
    latestCount.current = count;

    setTimeout(() => {
      console.log(`You clicked ${count} times(count)`);

      // 取得最新的值
      console.log(`You clicked ${latestCount.current} times(latestCount.current)`);
    }, 3000);
  });

  return (
    <div>
      <p>You click {count} times</p>
      <button onClick={() => setCount(count + 1)}>Add</button>
    </div>
  );
}

export default Counter;
```

#### 错误示范

> [Why need useRef to contain mutable variable but not define variable outside the component function?](https://stackoverflow.com/questions/57444154/why-need-useref-and-not-mutable-variable/57444430#57444430) @ StackOverflow

有些时候，可以在function component外定义一个变量（countCache），
然后在function component内去使用到这个变量（countCache），像是这样：

```jsx | pure
import React, { useState, useEffect, useRef } from 'react';

// 在 function 外定義一個變數
let countCache = 0;

function Counter() {
  const [count, setCount] = useState(0);
  countCache = 0; // 給予預設值為 0

  useEffect(() => {
    // 每次畫面轉譯完成後賦值
    countCache = count;

    setTimeout(() => {
      console.log(`You clicked ${count} times (count)`);

      // 讀取該變數值
      console.log(`You clicked ${countCache} times (countCache)`);
    }, 3000);
  });

  return (
    <div>
      <p>You click {count} times</p>
      <button onClick={() => setCount(count + 1)}>Add</button>
    </div>
  );
}

export default Counter;
```


### 搭配 TypeScript 使用

如果要保存HTMLElement的话可以这样使用：
```jsx | pure
// inputRef 的型別會是 RefObject<HTMLInputElement>，而它的 current 會是 readonly
const inputRef = useRef<HTMLInputElement>(null);
// ...
<input type="text" ref={inputRef} />;
```

但如果这个inputRef在后面才被赋值的话，其泛型需要使用```useRef<HTMLXXXElement | null>```的写法，
原因可以从React对```useRef```的定义看出端倪（[How to Fix useRef React Hook "Cannot Assign to ... Read Only Property" TypeScript Error?](https://www.designcise.com/web/tutorial/how-to-fix-useref-react-hook-cannot-assign-to-read-only-property-typescript-error)）

```tsx | pure
/**
 * An example of callback ref with TypeScript
 **/

const App = () => {
  // inputRef 的型別會是 MutableRefObject<HTMLInputElement | null>
  const inputRef = useRef<HTMLInputElement | null>(null);
  const setTextInput = useCallback((domElement: HTMLInputElement) => {
    inputRef.current = domElement;
  }, []);

  const handleClick = () => {
    inputRef.current?.focus();
  };

  return (
    <>
      <input type="text" ref={setTextInput} />
      <button type="button" onClick={handleClick}>
        Click
      </button>
    </>
  );
};
```

## useReducer

> **参考资料**
> [Extracting State Logic into a Reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer) @ React Docs Beta
> [A Complete Guide to useEffect:Decoupling Updates from Actions](https://overreacted.io/a-complete-guide-to-useeffect/#decoupling-updates-from-actions) @ Overreacted

### 基本使用

```jsx | pure
// 預先定義好的 initialState 和 reducer
const initialState = {
  bar: 'BAR',
};

function reducer(state, action) {
  const { bar } = state;
  switch (action.type) {
    case 'foo':
      return { bar: bar };
    default:
      return state;
  }
}

const App = () => {
  // 使用 useReducer，需要將定義好的 reducer 和 initialState 帶入
  const [state, dispatch] = useReducer(reducer, initialState);

  // 呼叫 reducer 的方式，dispatch(<action>)
  return <button onClick={() => dispatch('foo')}>Click Me</button>;
};
```

```useReducer```使用时机是**当你需要更新一个数据，而这个状态其实依赖另一个数据的时候**，当你写出
```setSomething(something => ...)```时，就需要考虑到可能是使用```useReducer```的时候。

通过```useReducer```，可以不再需要在```useEffect```内去读取状态，只需要在effect内去dispatch一个action
来更新state。如此，**effect不再需要更新状态，只需要说明发生了什么，更新的逻辑则都在reducer里面统一执行**.

这样，可以做到关注点分离（separate concerns）。在event handler中，通过dispatch action，只需要关注**做什么（what）**，
在reducer中则决定要**如何（how）处理**这些数据。

<Alert type='success'>
Action 的 Type 描述的是做什么（what），而不是怎么做（how），因此，有可能有两个action type 做的事情是一样的，例如，
edit_message和clear_message都可以做到把message清空，但还是应该把它们分开，以后续添加新的功能和逻辑。
</Alert>


<Alert type='warning'>
另一个十分重要的概念是，reducer function一定是要是pure的，千万不要在reducer里面执行side-effect（例如，调用API），
而是应该把要side effect放在event handler或者其他地方。React在Strict Mode中会重复执行reducer functions，以便于开发者尽早发现这些可能的错误。
</Alert>

<Alert type='info'>
React会确保 dispatch 方法在每次重新渲染后仍然是相同的，因此即使没有把dispatch放在useEffect或useCallback的dependency array中，也不会产生任何问题。
</Alert>

### 搭配# 搭配TypeScript

**只要```reducer``` function的类型有定义清楚的话，TypeScript就会自动推导```useReducer```中的类型
```tsx | pure
// 定義 reducer 中 state 的 type
type StateType = { foo: string; bar: string };
const initialState: StateType = { foo: 'foo', bar: 'bar' };

// 定義 reducer 中 action 的 type
type ActionType = {
  type: 'ACTION_A' | 'ACTION_B' | 'ACTION_C';
  payload: number; // 如果 payload 的型別都一樣
};

// reducer 的型別有定義好的話，useReducer 的地方就不用在定義
const reducer = (state: StateType, action: ActionType) => {
  /* ... */
};

const App = () => {
  const [state, dispatch] = useReducer(reducer, initialState);

  return <div>{/* ... */}</div>;
};
```

若需要把```dispatch```当作props传递给其他component时，一般来说```dispatch```的类型会是
```React.Dispatch<ReducerActionType>```（```ReducerActionType```是自己取得type名字）：

如果dispatch的action有不同类型的payload的话，可以定义多个type在搭配使用```|```，如此可以确保某些```type```
的action其```payload```一定要符合特定类型：

```tsx | pure
// reducer.ts
import { ITask } from './types';

type AddedAction = {
  type: 'added';
  id: number;
  text: string;
};

type ChangedAction = {
  type: 'changed';
  task: ITask;
};

type DeletedAction = {
  type: 'deleted';
  id: number;
};

// 如果 Action 所帶的 payload 型別不同，可以使用 union 的方式來定義 Action 的型別
type ReducerAction = AddedAction | ChangedAction | DeletedAction;

export default function tasksReducer(tasks: ITask[], action: ReducerAction) {
  switch (action.type) {
    case 'added': {
      /* ... */
    }
    case 'changed': {
      /* ... */
    }
    case 'deleted': {
      /* ... */
    }
    default: {
      // eslint-disable-next-line @typescript-eslint/restrict-template-expressions
      throw Error(`Unknown action: ${action}`);
    }
  }
}
```

如果看到错误信息 ```Argument of type 'FooBarState' is not assignable to parameter of type 'never'.```，
可以留意是不是```reducer``` function中最后没有回传state（所有case都不符合的情况下）:

```tsx | pure
const reducer = (state: StateType, action: ActionType) => {
  switch (action.type) {
    case 'ACTION_A': {
      /* ... */
    }
    case 'ACTION_B': {
      /* ... */
    }
    case 'ACTION_C': {
      /* ... */
    }
    default: {
      // 最後如何果沒回傳 default case 的話，這裡會變成 never type
      // return state;
    }
  }
};
```

### 范例：通过reducer读取并更新state

- [Timer通过useReducer获取并更新state](https://codesandbox.io/s/xzr480k0np) @ code sandbox
- [Timer通过useReducer获取props并更新state](https://codesandbox.io/s/7ypm405o8q) @ code sandbox

```tsx | pure
import React, { useEffect, useReducer } from 'react';

const initialState = {
  count: 0,
  step: 1,
};

const Timer = () => {
  const [state, dispatch] = useReducer(timerReducer, initialState);
  const { count, step } = state;

  // 使用 useReducer 可以將使用到的 state 和 props 從 useEffect 搬到 useReducer 內，
  // 在 useEffect 內只需要描述要進行的動作 dispatch(action)
  useEffect(() => {
    const timerId = setInterval(() => {
      dispatch({
        type: 'tick',
      });
    }, 1000);

    return () => {
      clearInterval(timerId);
    };
  }, [dispatch]); // dispatch 可以省略不寫

  return (
    <>
      <p>Current Count: {count}</p>
      <div>
        <input
          value={step}
          onChange={(e) => dispatch({ type: 'step', step: Number(e.target.value) })}
        />
      </div>
    </>
  );
};

function timerReducer(state, action) {
  const { count, step } = state;
  if (action.type === 'tick') {
    return {
      count: count + step,
      step,
    };
  } else if (action.type === 'step') {
    return {
      count: count,
      step: action.step,
    };
  } else {
    throw new Error();
  }
}

export default Timer;
```

## useCallback, useMemo

- [useCallback](https://legacy.reactjs.org/docs/hooks-reference.html#usecallback) @ React Docs - API Reference
- [useCallback的使用时机](https://www.developerway.com/posts/how-to-write-performant-react-code) @ Bonus: the useCallback conundrum
- [useMemo](https://legacy.reactjs.org/docs/hooks-reference.html#usememo) @ React Docs - API Reference
- [如何错误的使用 React hooks useCallback 來保存相同的 function instance](https://link.medium.com/Aed9DAtk3Z) @ medium

```jsx | pure
// useCallback(fn, deps) is equivalent to useMemo(() => fn, deps).
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);

const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```



