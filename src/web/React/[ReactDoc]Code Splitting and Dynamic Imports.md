# [ReactDoc] Code Splitting and Dynamic Imports

> [code-splitting](https://legacy.reactjs.org/docs/code-splitting.html) @ React Docs


## 概念

code splitting 的意思是将一个程序切分成许多不同的块（chunks），例如，把使用者不一定会用到的元件切成一个档案，使用者使用改元件的时候才去下载和使用。

虽然通过 code splitting 无法减少App中整体代码的体积大小，但是可以避免使用者花费过多的流量和时间
去下载那些用不到的的内容，并且可以有效改善使用者第一次载入App所需要的时间。


## 使用

### 使用 Dynamic Import 來进行 code splitting

只需要使用dynamic import的方式，bundle工具就会自动进行code splitting工作。dynamic import只需要把import变成Promise即可：
```js
//reactjs.org/docs/code-splitting.html#import
https: import('./math').then((math) => math.add(16, 26));
```

### 使用lazy动态载入react组件

在React中，要使用dynamic import来载入react元件的话，可以搭配react提供的```lazy```和```Suspense```:
- ```lazy``` 用来帮助react component实现dynamic import
- ```Suspense```和```fallback```则会在dynamic import时显示预设的画面（如加载动画等）


```jsx | pure
// STEP1: 载入lazy和Suspsense
import {lazy, Suspense} from 'react';
import Homepage from 'views/Homepage';

// STEP2: 使用lazy进行dynamic import
const DemoContext = lazy(() => import('views/DemoContext'));

const App = () => {
  return(
    <div className='App'>
      {/*STEP3: 加上Suspense*/}
      <Suspense fallback={<h2>Dynamic Loading...</h2>}>
        <Switch>
          <Route path='/use-context'>
            <DemoContext/>
          </Route>
          
          <Route path='/'>
            <Homepage />
          </Route>
        </Switch>
      </Suspense>
    </div>
  )
}
```

使用dynamic imports之后，只有当使用者切换到该页面时，才会去载入改元件对应的档案(```*.chunk.js```），同样的方式也可以用来载入和UI有关的组件（如Modal）

<Alert type="warning">
需要注意的是，并不是所有的元件或页面都应该使用dynamic import来进行code splitting。因为使用code splitting，也会需要额外的glue code来让不同位置的代码可以顺利拼接在一起，
这些代码可能会增加档案体积，因此对于原本代码规模就不大的元件来说，这么做可能会弄巧成拙。
</Alert>

### Named Exports

```React.lazy``` 目前只支持default exports，如果你的元件是使用named exports，
则需要多一个中介把他装成default exports，这样才能保证tree shaking正常运行：
```jsx | pure
// https://reactjs.org/docs/code-splitting.html#named-exports

// intermediate module to transform named exports to default exports
export { MyComponent as default } from './ManyComponents.js';
```

### Avoiding fallbacks
有些时候，我们希望沿用已有的UI，并且希望在切换元件的时候不要再次执行fallback，这时候可以使用```startTransition```API，如：

```jsx | pure
// https://reactjs.org/docs/code-splitting.html#avoiding-fallbacks

import { startTransition, useState } from 'react';

function MyComponent() {
  const [tab, setTab] = useState('photos');

  function handleTabSelect(tab) {
    startTransition(() => {
      setTab(tab);
    });
  }
  
  return(
    <div>
      <Tabs onTabSelect={handleTabSelect} />
      <Suspense fallback={<Glimmer />}>
        {tab === 'photos' && <Photos /> || <Comments />}
      </Suspense>
    </div>
  )
}
```
这样则会告诉React，这并不是一个迫切的更新，但使用者希望切换Tabs的时候，React不会再次渲染fallback，这是直接显示原本的UI，
因此只有当```<Comments/>```准备完成后才会进行切换。

### Error boundaries
通过建立Error boundaries可以处理 当网络异常导致无法载入元件或要呈现的画面 的异常：
```jsx | pure
import React, { Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

const MyComponent = () => (
  <div>
    <ErrorBoundary FallbackComponent={Error}>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </ErrorBoundary>
  </div>
);

function Error({ error }) {
  return (
    <div>
      <h1>Application Error</h1>
      <pre style={{ whiteSpace: 'pre-wrap' }}>{error.stack}</pre>
    </div>
  );
}
```
