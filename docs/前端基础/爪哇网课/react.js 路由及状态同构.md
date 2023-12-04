# react.js 路由及状态同构

### 手写一个简单的 react-router

1. SPA
2. History API 去修改浏览器的 url，但不会重新加载页面
   - pushState： 创建并跳转至新的 url
   - replaceState：修改当前的 url
   - back
   - forward
   - go
3. 监听前进，后退，可以通过 popstate

#### history 版本 router

```jsx
function BrowserRouter(props) {
  const RouterContext = createContext();
  const HistoryContext = createContext();

  const [path, setPath] = useState(() => {
    const { pathname } = window.location;
    return pathname || "";
  });

  useEffect(() => {
    // 监听用户点击浏览器前进，后退的事件
    window.addEventListener("popstate", handlePopState);

    return () => {
      window.removeEventListener("popstate", handlePopState);
    };
  });

  const handlePopState = (e) => {
    const { pathname } = window.location;
    setPath(pathname);
  };

  // 监听页面点击行为，包括返回上一步，转跳
  const push = (path) => {
    window.history.pushState({ path }, "", path);
  };
  const goBack = () => {
    window.history.go(-1);
  };

  return (
    <RouterContext.provider>
      <HistoryContext.provider value={{ push, goBack }}>
        {props.children}
      </HistoryContext.provider>
    </RouterContext.provider>
  );
}

const Route = (props) => {
  const { path: componentPath, Element } = props;
  return (
    <RouterContext.provider>
      {(path) => {
        return path === componentPath ? <Element /> : null;
      }}
    </RouterContext.provider>
  );
};
```

#### hash router

```jsx
function HashRouter(props) {
  const RouterContext = createContext();
  const HistoryContext = createContext();

  const [path, setPath] = useState(() => {
    const { hash } = window.location;
    if (hash) {
      return hash.slice(1);
    } else {
      return "/#/";
    }
  });

  useEffect(() => {
    // 监听用户点击浏览器前进，后退的事件
    window.addEventListener("popstate", handlePopState);

    return () => {
      window.removeEventListener("popstate", handlePopState);
    };
  });

  const handlePopState = (e) => {
    const { hash } = window.location;
    setPath(hash.slice(1));
  };

  // 监听页面点击行为，包括返回上一步，转跳
  const push = (path) => {
    // window.history.pushState({ path }, "", path);
    window.location.hash = path;
  };
  const goBack = () => {
    window.history.go(-1);
  };

  return (
    <RouterContext.provider>
      <HistoryContext.provider value={{ push, goBack }}>
        {props.children}
      </HistoryContext.provider>
    </RouterContext.provider>
  );
}

const Route = (props) => {
  const { path: componentPath, Element } = props;
  return (
    <RouterContext.provider>
      {(path) => {
        return path === componentPath ? <Element /> : null;
      }}
    </RouterContext.provider>
  );
};
```
