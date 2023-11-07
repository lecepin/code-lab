- 项目仓库： https://github.com/alibaba/hooks

它是由阿里开源的 react hooks 库，前身是 umi-hooks。

目前 github star 12.7k，react hooks 库排名也在前列。

#### React Hooks

- React Hooks 是 react v16.8 的一个新特性，与 function components 关联使用。
- React class component 现在用的人应该很少了，react 官方也不推荐使用了。

hooks 的方式带来的明显的优势之一就是逻辑复用，它可以极大的提高开发效率。以前的 Class Component 中以 render Props 和 HOC 的方式进行复用，很繁琐，不灵活。

现在有了 hook 模式，就可以将需要复用的逻辑与 React 的几个基本 hooks api 结合起来，创建 useXXX ，这样 use 开头的自定义 hook。

小例子：

```jsx
import React, { useState } from "react";

const myPromiseTest = () =>
  new Promise((res, rej) => {
    setTimeout(() => {
      res("lalala");
    }, 1000);
  });

export default function App() {
  // const [loading, setLoading] = useState(false);
  // const [data, setData] = useState('');
  const { loading, data, run } = usePromise(myPromiseTest);

  return (
    <div>
      {data}
      {loading && <div>loading...</div>}
      <button
        onClick={() => {
          run();
          // setLoading(true);
          // myPromiseTest()
          //   .then((res) => {
          //     setData(res);
          //   })
          //   .finally(() => {
          //     setLoading(false);
          //   });
        }}
      >
        获取数据
      </button>
    </div>
  );
}

const usePromise = (promiseFn) => {
  const [loading, setLoading] = useState(false);
  const [data, setData] = useState("");
  const run = () => {
    setLoading(true);
    promiseFn()
      .then((res) => {
        setData(res);
      })
      .finally(() => {
        setLoading(false);
      });
  };

  return { loading, data, run };
};
```

#### ahooks

ahooks 就是将我们日常开发中常用的一些功能进行了封装，可以简单类比为 lodash 的 hooks 化。

当然 ahooks 提供的场景很多。目前有：

- useRequest
- Scene
- LifeCycle
- ​State
- ​Effect
- ​Dom
- ​Advanced
- ​Dev
