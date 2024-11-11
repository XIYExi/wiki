# [ReactDoc] Portals

> 参考 [Portals](https://legacy.reactjs.org/docs/portals.html) @ React Docs


## 目的

通过Portal，可以在React root DOM node（一般是```#app```)以外的地方将React的内容render出来。
同时，也能解决z-index和stacking context导致UI被其他元素覆盖的问题。


## 范例

### HTML

虽然这里建立了两个DOM，但通过```React.createPortal```可以把```#root```的
React children在```#modal-root```中render出来；此外，虽然从HTML来看，```#root```
和```#modal-root```是sibling的关系，但通过```React.createProtal```，在```#modal-root```
内所作的操作和行为（例如，event bubbling），就会像是在```#root```里面一样。

```html
<body>
  <div id='root'></div>
  <div id='modal-root'></div>
</body>
```

### React Component

#### Modal

```ReactDOM.createPortal()```的API和React17以前的```ReactDOM.render()```很想，
第一个参数是JSX，第二个参数则是要被mounted上去的DOM元素。

```jsx | pure
const Modal = ({children}) => {
  // STEP 1：建立一个div
  const el = useRef(document.createElement('div'));
  
  useEffect(() => {
    // STEP 2：找到#modal-root
    const modalRoot = document.getElementById('modal-root');
    
    // STEP 3：把div放到#modal-root中
    modalRoot.appendChild(el.current);
    
    return () => {
      // STEP 5：将#modal-root中的div移除
      modalRoot.removeChild(el.current);
    }
  }, [])
  
  // STEP 4：将children的内容放到div中
  return ReactDOM.createPortal(children, el.current);
}
```

#### App

- 在```<Modal />```中children的内容，则会出现在```#modal-root```中
```jsx | pure
const DemoPortal = () => {
  const [showModal, setShowModal] = useState(false);
  const handleShow = () => {
    setShowModal(true);
  };
  const handleHide = () => {
    setShowModal(false);
  };

  return (
    <div>
      <button onClick={handleShow}>Show Modal</button>
      {showModal && (
        <Modal>
          <div className="modal">
            <div>
              With a portal, we can render content into a different part of the DOM, as if it were
              any other React child.
            </div>
            This is being rendered inside the #modal-container div.
            <button onClick={handleHide}>Hide modal</button>
          </div>
        </Modal>
      )}
    </div>
  );
};
```

## Material UI Modal的实例

Material UI Modal的实例可以参考这几个档案：

- [Modal](https://github.com/mui/material-ui/tree/master/packages/mui-base/src/Modal)
- [Portal](https://github.com/mui/material-ui/tree/master/packages/mui-base/src/Portal)

Material UI Modal 预设会使用```React.createPortal()```并把它mount在```document.body```上
