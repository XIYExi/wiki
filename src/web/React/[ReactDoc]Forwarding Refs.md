# [ReactDoc] Forwarding Refs

## 目的

有些时候父层组件希望获取到子层组件的DOM元素（例如，```button```或```input```),
以便能够在父层控制子层DOM元素的focus、selection或animation效果。这时就可以使用**Ref forwarding**
来让父层获取子层DOM元素，以便控制和操作它。

<Alert type='info'>
通常需要被forwardRef包裹的子层元件会是封装好的元件（例如：第三方UI组件），使用者无法直接修改内部元素，
因此才需要通过forwardRef把控制权交给父层元件，让开发者可以进行直接控制。
</Alert>

## forwardRef 基本使用

举例来说，我们建立一个```AwesomeInput```元件：
```jsx | pure
const AwesomeInput = (props) => <input type='text' />;
```

接着我们在父层```<App />```元件中使用```<AwesomeInput />```元件：
```jsx | pure 
const App = () => {
  return <AwesomeInput />
}
```
这时如果想要在App元件中得到AwesomeInput中的```<input/>```的value或者是对它进行focus的动作，就需要
通过forwardRef把这个```<input/>```传递到父层进行使用。

如要让```<App/>```可以操作```<AwesomeInput />```中的```<input />```元素，需要：

### STEP 1：在父层元件建立ref

- 先在父层元件通过```useRef```或者```createRef```建立一个ref，这里取名为```awesomeInputRef```
- 把建立好的```awesomeInputRef```通过```ref```属性传递到```<AwesomeInput />```元件中

```jsx | pure
import React from 'react';

const App = () => {
  const awesomeInputRef = React.useRef(null);
  
  return <AwesomeInput ref={awesomeInputRef} />
}
```

### STEP 2：在AwesomeInput中使用forwardRef

接着在```<AwesomeInput />```元件中，可以使用```React.forwardRef()```来把```<AwesomeInput />```内的
```<input />```传递出去：

- 一般的React Component是不会获取```ref```属性的，需要被```React.forwardRef()```包裹起来才会有```ref```属性：

```jsx | pure
import React from 'react';

const AwesomeInput = React.forwardRef((props, ref) =>
  <input type='text' ref={ref}/>
)
```

### STEP 3：在App中可以使用AwesomeInput中的input DOM元素

这样就可以直接在App中操作AwesomeInput中的```<input/>```元素，举例来说，我们希望做到autoFocus的效果，
就可以在```<App/>```元件中，通过```awesomeInputRef```取得里面的```<input/>```元素：

```jsx | pure
const App = () => {
  const awesomeInputRef = React.useRef(null);

  // App mounted 的時候讓 AwesomeInput 中的 input 元素 focus
  React.useEffect(() => {
    console.log(awesomeInputRef.current); // <input type="text">...</input>
    awesomeInputRef.current.focus(); // 對 AwesomeInput 中的 <input /> 進行操作
  }, []);

  return <AwesomeInput ref={awesomeInputRef} />;
}
```

## 在HOC中使用forwardRef

现在假设建立一个名为```logPropsHOC```的Higher Order Component（HOC），单纯是把元件的props有改变时的console打印出来：

```jsx | pure
import React from 'react';

const logPropsHOC = (WrappedComponent) => {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps, prevState, snapshot) {
      console.log('old props: ', prevProps);
      console.log('new props: ', this.props);
    }

    render() {
      return (
        <WrappedComponent {...this.props} />
      )
    }
  }
  
  return LogProps;
}
```

这个时候，如果我们把上面写过的AwesomeInput元件用此HOC包裹起来再使用：
```jsx | pure
const AwesomeInput = React.forwardRef((props, ref) => {
  return <input ref={ref} type='text' />
})

const AwesomeInputWithLogProps = logPropsHOC(AwesomeInput);
```

接着在App中使用```AwesomeInputWithLogProps```:
```jsx | pure
const App = () => {
  const awesomeInputRef = React.useRef(null);
  
  React.useEffect(() => {
    console.log(awesomeInputRef.current); // <input type="text">...</input>
    // awesomeInputRef.current.focus(); // 對 AwesomeInput 中的 <input /> 進行操作
  }, [])
  
  return <AwesomeInputWithLogProps ref={awesomeInputRef} />;
}
```

这个时候会发现```awesomeInputRef.current```的对象变成该HOC元件，也就是```LogProps```，
而不像原本一样能够传递到AwesomeInput元件中的```<input />```元素。

为什么会这样呢？这是因为，虽然我们有在```LogProps```中这样写：

```js | pure
return <WrappedComponent {...this.props} />;
```

把所有的props都带回到原本的元件中，但**因为```ref```并不是props，所以再props中并没有ref的内容，
进而不会传递到AwesomeInput元素中**。

即使再AwesomeInput元件包裹了HOC后，为了要让App元件还是可以获取到AwesomeInput元件中的```<input />```,
我们需要再HOC中一样先通过```React.forwardRef```取出```ref```后，再把它传递到被HOC包裹起来的
WrappedComponent中，具体来说可以这样做：

### STEP 1：将HOC回传的元件也用React.forwardRef

首先将HOC回传的元件也用```React.forwardRef```取出```ref```,
接着再把这个ref传递到```LogProps```元件内：

```jsx | pure
const logPropsHOC = (WrappedComponent) => {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props: ', prevProps);
      console.log('new props: ', this.props);
    }

    render() {
      return <WrappedComponent {...this.props} />;
    }
  }
  
  return React.forwardRef((props, ref) => <LogProps {...props} forwardedRef={ref} />);
}
```

<Alert type='info'>
需要留意的是 ref 是 React 元件的关键字，这里如果单纯只是要把 ref 当成 props 往下传的话，不能用 ref 当名称，
因为这里使用 forwardedRef 作为 props 的名称
</Alert>

### STEP 2：从props中取出forwardRef并带到下层的ref

接着就可以从props中取出```forwardRef```后，通过```ref```把他带进去：

```jsx | pure
const logPropsHOC = (WrappedComponent) => {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props: ', prevProps);
      console.log('new props: ', this.props);
    }

    render() {
      const { forwardedRef, ...rest } = this.props;
      return <WrappedComponent ref={forwardedRef} {...rest} />;
    }
  }

  return React.forwardRef((props, ref) => <LogProps {...props} forwardedRef={ref} />);
}
```

<Alert info='success'>
留意Component使用ref时，是要使用ref的功能，还是只是要把父层的ref当层props往下传递。
如果是要把ref当成props往下传递，就不能使用ref当作属性名称，而要换名字，例如forwardedRef={ref}。
</Alert>

## 针对 Class Component 使用 forwardRef
针对 Class Component 使用forwardRef的方式和Function Component非常相似。首先将原本的AwesomeInput元件改成class component：

```jsx | pure
class AwesomeInput extends React.Component {
  render(){
    return <input type='text'/>
  }
}
```

### STEP 1：通过props接受父层传递过来的ref

由于要把ref当成props传递时，props的名称不能直接沿用```ref```，因此要改名为```forwardRef```，并从props中取出：

```jsx | pure
class AwesomeInput extends React.Component {
  render() {
    const {forwardedRef} = props;
    return <input type='text' ref={forwardedRef} />
  }
}
```

<Alert typ='info'>
Class Component虽然能直接使用在元素的使用上使用ref，但它会指称到的是该class的instance而非 input标签
</Alert>

### STEP 2：通过props接受父层传递过来的ref

接着要使用```React.forwardRef```先取出父层传入的```ref```，并改名为```forwardRef```后以props传入```AwesomeInput```元件中：

```jsx | pure
const AwesomeInputWithForwardRef = React.forwardRef((props, ref) => {
  // 把父層的 ref 透過 props 往下傳
  return <AwesomeInput forwardedRef={ref} {...props} />;
});
```

### STEP 3：在父层中可直接获取到AwesomeInput元件中的input元素

App元素并不需要做什么更改，即可获取到```<AwesomeInput />```元件中的```<input />```元素：

```jsx | pure
const App = () => {
  const awesomeInputRef = React.useRef(null);

  React.useEffect(() => {
    console.log(awesomeInputRef.current); // <input type="text">...</input>
    awesomeInputRef.current.focus(); // 對 AwesomeInput 中的 <input /> 進行操作
  }, []);

  return <AwesomeInputWithForwardRef ref={awesomeInputRef} />;
};
```

## 几种不同的变化型

若有需要在父层获得子层的ref，React在官网分拣中的 [Exposing DOM Refs to Parent Components](https://legacy.reactjs.org/docs/refs-and-the-dom.html#exposing-dom-refs-to-parent-components)
中建议使用```React.forwardRef```的做法，但若你使用的是React 16.2以前的版本，或需要更弹性的用法，则
直接把ref当成props传递到子层也是可行的，但props的名称不可以沿用```ref```，需要另外取名。

### Function Component：使用React.forwardRef

- 直接在JSX的Function Component中使用```ref```

```jsx | pure
const AwesomeInput = React.forwardRef((props, ref) => {
  return <input type="text" ref={ref} />;
});

const App = () => {
  const awesomeInputRef = React.useRef(null);

  React.useEffect(() => {
    console.log(awesomeInputRef.current); // <input type="text">...</input>
    awesomeInputRef.current.focus(); // 對 AwesomeInput 中的 <input /> 進行操作
  }, []);

  // 直接在 JSX 的 Function Component 中使用 ref
  return <AwesomeInput ref={awesomeInputRef} />;
};
```


### Function Component：直接把ref当成props往下传

- 在JSX的Function Component中，不用```ref```当属性名称，换成其他名称后以props的方式往下传（例如：```forwardedRef```)

```jsx | pure
const AwesomeInput = (props) => {
  // 取用的是 forwardedRef
  return <input type="text" ref={props.forwardedRef} />;
};

const App = () => {
  const awesomeInputRef = React.useRef(null);

  React.useEffect(() => {
    console.log(awesomeInputRef.current); // <input type="text">...</input>
    awesomeInputRef.current.focus(); // 對 AwesomeInput 中的 <input /> 進行操作
  }, []);

  // 將 ref 當成 props（不使用 ref 作為屬性名稱）往下傳
  return <AwesomeInput forwardedRef={awesomeInputRef} />;
};
```


### Class Component：直接使用ref往下传

#### 错误写法

- 虽然class component可以直接接收```ref```做为参数，但它会得到的是该Component的instance而不是Component内的DOM元素

```jsx | pure
// 错误写法：ref 会是 AwesomeInput 元件，而不是內部的 input 元素
class AwesomeInput extends React.Component {
  render() {
    const { ref } = this.props;
    return <input type="text" ref={ref} />;
  }
}

const App = () => {
  const awesomeInputRef = React.useRef(null);

  React.useEffect(() => {
    // 这里得到的会是 `<AwesomeInput />`
    console.log(awesomeInputRef.current); // <AwesomeInput />
  }, []);

  // 直接通过 ref 往下传递
  return <AwesomeInput ref={awesomeInputRef} />;
};
```

#### 使用forwardRef

参考针对 [针对 Class Component 使用 forwardRef] 的说明。


### 可行方法

```jsx | pure
// 在 AwesomeInput 建立另一個 ref 以指稱到 input 元素
class AwesomeInput extends React.Component {
  innerRef = React.createRef();

  render() {
    return <input type="text" ref={this.innerRef} />;
  }
}

const App = () => {
  const awesomeInputRef = React.useRef(null);

  React.useEffect(() => {
    // 先指稱到 AwesomeInput 元件，再進去裡面找到 input 元素
    console.log(awesomeInputRef.current.innerRef.current);
  }, []);

  return <AwesomeInput ref={awesomeInputRef} />;
};
```
