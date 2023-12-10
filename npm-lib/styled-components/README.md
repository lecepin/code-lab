- 项目仓库： https://github.com/styled-components/styled-components

## 1. What

CSS-IN-JS 方式用于给 React 编写样式组件的库。Github CSS-IN-JS 排行版第一。

采用 [字符模板标签函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#tagged_templates) 的方式供用户使用，编写灵活、快速。

```tsx
const Button = styled.a<{ $primary?: boolean }>`
  --accent-color: white;

  background: transparent;
  border-radius: 3px;
  border: 1px solid var(--accent-color);
  color: var(--accent-color);
  display: inline-block;
  margin: 0.5rem 1rem;
  padding: 0.5rem 0;
  transition: all 200ms ease-in-out;
  width: 11rem;

  &:hover {
    filter: brightness(0.85);
  }

  &:active {
    filter: brightness(1);
  }

  ${(props) =>
    props.$primary &&
    css`
      background: var(--accent-color);
      color: black;
    `}
`;
```

入门简单， 支持 less/sass 的嵌套语法，编写方便。

## 2. Why

#### 2.1 优势

- 唯一 className，不会出现重复及类样式污染的问题。
- 样式与组件捆绑，不使用组件，不生成样式。
- 避免样式与其他组件的耦合。
- 内置自动加前缀的功能。
- 样式动态化，与 JS 关联更密切，比 less/sass 更方便。
- 无需工程改造，即可使用。
- 减少样板目录的创建，直接单文件消费。

#### 2.2 劣势

- 运行时生成，对性能有一定影响（小工程无需担心）。
- 打包 JS 文件会包含样式，偏大。
- className 动态，影响消费者覆写样式。

## 3. How

```
npm install styled-components
```

```tsx
import styled from "styled-components";

const Title = styled.h1`
  font-size: 1.5em;
  text-align: center;
  color: #bf4f74;
`;

const Wrapper = styled.section`
  padding: 4em;
  background: papayawhip;
`;

render(
  <Wrapper>
    <Title>Hello World!</Title>
  </Wrapper>
);
```

#### 3.1 API

| 名称                                                                                          | 说明                                                                                                                                             |
| --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| styled.tagName\`\` \| styled(Compoent)\`\`                                                    | 正常的 CSS 编写的方式。<br /> 属性值可以是字符串，或者透传 `props` 的函数。 <br /> `color: ${props => (props.$whiteColor ? 'white' : 'black')};` |
| styled.tagName.attr({} \| Function)\`\`                                                       | 给组件注入指定的属性，当外层透传时，会被些处覆盖                                                                                                 |
| `<Comp as="htmlTag" />`                                                                       | 在样式组件中使用 `as` 进行渲染 DOM 的元素转换                                                                                                    |
| `<Comp $prop="" />`                                                                           | 使用 `$` 对组件添加属性，可避免将此属性渲染到 DOM 中                                                                                             |
| ❌ styled.tagName.withConfig({ shouldForwardProp: (prop) => !['hidden'].includes(prop),})\`\` | 通过 `shouldForwardProp` 来控制透传属性 ，可结合上一个 `$` 前缀使用                                                                              |
| `<ThemeProvider theme={{xx:''}} />`                                                           | 主题控制，通过此标签，内部样式组件可通过 `props.theme.xx` 获取值                                                                                 |
| createGlobalStyle``                                                                           | 全局样式                                                                                                                                         |
| css``                                                                                         | CSS 片段，可用来组装                                                                                                                             |
| keyframes\`\`                                                                                 | 动画场景专用，可用来写 keyframes                                                                                                                 |
| `<StyleSheetManager enableVendorPrefixes>`                                                    | 用来控制开启关闭自动加不同浏览器厂商的前缀                                                                                                       |

#### 3.2 示例

属性动态控制：

```tsx
const Button = styled.button<{ $primary?: boolean }>`
  background: ${(props) => (props.$primary ? "#BF4F74" : "white")};
  color: ${(props) => (props.$primary ? "white" : "#BF4F74")};

  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid #bf4f74;
  border-radius: 3px;
`;

render(
  <div>
    <Button>Normal</Button>
    <Button $primary>Primary</Button>
  </div>
);
```

扩展样式组件：

```tsx
const Button = styled.button`
  color: #bf4f74;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid #bf4f74;
  border-radius: 3px;
`;

const TomatoButton = styled(Button)`
  color: tomato;
  border-color: tomato;
`;

render(
  <div>
    <Button>Normal Button</Button>
    <TomatoButton>Tomato Button</TomatoButton>
  </div>
);
```

强制指定不同 DOM 渲染：

```tsx
const Button = styled.button`
  display: inline-block;
  color: #bf4f74;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid #bf4f74;
  border-radius: 3px;
  display: block;
`;

render(
  <div>
    <Button>Normal Button</Button>
    <Button as="a" href="#">
      Link with Button styles
    </Button>
  </div>
);
```

可指定 自定义 的组件：

```tsx
const Button = styled.button`
  display: inline-block;
  color: #bf4f74;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid #bf4f74;
  border-radius: 3px;
  display: block;
`;

const ReversedButton = (props) => (
  <Button {...props} children={props.children.split("").reverse()} />
);

render(
  <div>
    <Button>Normal Button</Button>
    <Button as={ReversedButton}>Custom Button with Normal Button styles</Button>
  </div>
);
```

被包装的 自定义组件 可以通过 `props.className` 来关联：

```tsx
const Link = ({ className, children }) => (
  <a className={className}>{children}</a>
);

const StyledLink = styled(Link)`
  color: #bf4f74;
  font-weight: bold;
`;

render(
  <div>
    <Link>Unstyled, boring Link</Link>
    <br />
    <StyledLink>Styled, exciting Link</StyledLink>
  </div>
);
```

可用 `&` 来替代当前组件的 className，编译后会被替换：

```tsx
const Thing = styled.div`
  color: blue;

  &:hover {
    color: red; // <Thing> when hovered
  }

  & ~ & {
    background: tomato; // <Thing> as a sibling of <Thing>, but maybe not directly next to it
  }

  & + & {
    background: lime; // <Thing> next to <Thing>
  }

  &.something {
    background: orange; // <Thing> tagged with an additional CSS class ".something"
  }

  .something-else & {
    border: 1px solid; // <Thing> inside another element labeled ".something-else"
  }
`

render(
  <React.Fragment>
    <Thing>Hello world!</Thing>
    <Thing>How ya doing?</Thing>
    <Thing className="something">The sun is shining...</Thing>
    <div>Pretty nice day today.</div>
    <Thing>Don't you think?</Thing>
    <div className="something-else">
      <Thing>Splendid.</Thing>
    </div>
  </React.Fragment>
)

```

通过多个 `&` 来提升 CSS 选择器权重，来避免使用 `!important`：

```tsx
onst Thing = styled.div`
   && {
     color: blue;
   }
 `

 const GlobalStyle = createGlobalStyle`
   div${Thing} {
     color: red;
   }
 `

 render(
   <React.Fragment>
     <GlobalStyle />
     <Thing>
       I'm blue, da ba dee da ba daa
     </Thing>
   </React.Fragment>
 )
```

注入默认属性：

```tsx
const Input = styled.input.attrs<{ $size?: string }>((props) =>
  // 此处属性会注入到 DOM 中
  ({
    // 静态类型不会被复写
    type: "text",
    // 动态类型会可被外层复写
    $size: props.$size || "1em",
  })
)`
  color: #bf4f74;
  font-size: 1em;
  border: 2px solid #bf4f74;
  border-radius: 3px;

  /* here we use the dynamically computed prop */
  margin: ${(props) => props.$size};
  padding: ${(props) => props.$size};
`;

render(
  <div>
    <Input placeholder="A small text input" />
    <br />
    <Input placeholder="A bigger text input" $size="2em" />
  </div>
);
```

可通过 styled(Component) 的方式来覆盖静态属性：

```tsx
const Input = styled.input.attrs<{ $size?: string }>((props) => ({
  type: "text",
  $size: props.$size || "1em",
}))`
  border: 2px solid #bf4f74;
  margin: ${(props) => props.$size};
  padding: ${(props) => props.$size};
`;

const PasswordInput = styled(Input).attrs({
  type: "password",
})`
  border: 2px solid aqua;
`;

render(
  <div>
    <Input placeholder="A bigger text input" $size="2em" />
    <br />
    {/* Notice we can still use the size attr from Input */}
    <PasswordInput placeholder="A bigger password input" $size="2em" />
  </div>
);
```

动画：

```tsx
const rotate = keyframes`
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
`;

const Rotate = styled.div`
  display: inline-block;
  animation: ${rotate} 2s linear infinite;
  padding: 2rem 1rem;
  font-size: 1.2rem;
`;

render(<Rotate>&lt; 💅🏾 &gt;</Rotate>);
```

要写动画片段的话，必须 `styled.css` 包裹：

```tsx
const rotate = keyframes``;

// ❌ This will throw an error!
const styles = `
  animation: ${rotate} 2s linear infinite;
`;

// ✅ This will work as intended
const styles = css`
  animation: ${rotate} 2s linear infinite;
`;
```

主题：

```tsx
const Button = styled.button`
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;

  /* Color the border and text with theme.main */
  color: ${(props) => props.theme.main};
  border: 2px solid ${(props) => props.theme.main};
`;

Button.defaultProps = {
  theme: {
    main: "#BF4F74",
  },
};

const theme = {
  main: "mediumseagreen",
};

render(
  <div>
    <Button>Normal</Button>

    <ThemeProvider theme={theme}>
      <Button>Themed</Button>
    </ThemeProvider>
  </div>
);
```

`theme` 支持函数类型，如果多层嵌套，可以通过函数中的参数获取上一个主题的值：

```tsx
/ Define our button, but with the use of props.theme this time
const Button = styled.button`
  color: ${props => props.theme.fg};
  border: 2px solid ${props => props.theme.fg};
  background: ${props => props.theme.bg};

  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;
`;

// Define our `fg` and `bg` on the theme
const theme = {
  fg: "#BF4F74",
  bg: "white"
};

// This theme swaps `fg` and `bg`
const invertTheme = ({ fg, bg }) => ({
  fg: bg,
  bg: fg
});

render(
  <ThemeProvider theme={theme}>
    <div>
      <Button>Default Theme</Button>

      <ThemeProvider theme={invertTheme}>
        <Button>Inverted Theme</Button>
      </ThemeProvider>
    </div>
  </ThemeProvider>
);
```

动态 CSS 属性名：

```tsx
const EqualDivider = styled.div`
  display: flex;
  margin: 0.5rem;
  padding: 1rem;
  background: papayawhip;
  ${props => props.$vertical && "flex-direction: column;"}

  > * {
    flex: 1;

    &:not(:first-child) {
      ${props => props.$vertical ? "margin-top" : "margin-left"}: 1rem;
    }
  }
`;

const Child = styled.div`
  padding: 0.25rem 0.5rem;
  background: #BF4F74;
`;

render(
  <div>
  <EqualDivider>
    <Child>First</Child>
    <Child>Second</Child>
    <Child>Third</Child>
  </EqualDivider>
  <EqualDivider $vertical>
    <Child>First</Child>
    <Child>Second</Child>
    <Child>Third</Child>
  </EqualDivider>
  </div>
);
```

#### 3.3 注意

避免将 styled-components 生成的组件放在 render 中：

```tsx
// 🚫
const Header = () => {
  const Title = styled.h1`
    font-size: 10px;
  `;

  return (
    <div>
      <Title />
    </div>
  );
};
```

```tsx
// ✅
const Title = styled.h1`
  font-size: 10px;
`;

const Header = () => {
  return (
    <div>
      <Title />
    </div>
  );
};
```
