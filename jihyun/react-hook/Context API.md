# Context를 사용할 때, 값과 업데이트 함수를 분리하는 이유 파헤쳐보기

# Context API의 리렌더링

Context API에는 리렌더링에 관한 이슈가 있다는 얘기를 한 번 쯤은 들어보셨을 것이라 생각합니다.

그리고 리렌더링 이슈를 해결하기 위해 상태와 상태 변경 함수를 각각의 context로 사용하는 경우도 있는데요.

먼저 예제를 통해 context를 사용할 때 **언제 리렌더링이 발생**하고, **왜 리렌더링이 발생**하는지 이해한 다음, 두 개의 context 사용하는 이유에 대해 알아보겠습니다.

## 프로바이더로 감싼 컴포넌트 중 리렌더링되는 컴포넌트 찾기

저번에 작성했던 [리액트에서 리렌더링의 조건 이해하기](https://sungjihyun.vercel.app/blog/rerendering-conditions-of-react) 글의 3번째 예제입니다.

`Parent` 컴포넌트는 `ChildA` 컴포넌트와 `children`, `lastChild` props를 context 프로바이더로 감싸 리턴합니다.

그리고 `children`, `lastChild`에 각각 대응되는 `ChildB`는 context 값을 참조하지 않고 있으며 `ChildC`는 context 값을 참조하고 있습니다.

```jsx
const Context = createContext();

export default function App() {
  return (
    <Parent lastChild={<ChildC />}>
      <ChildB />
    </Parent>
  );
}

function Parent({ children, lastChild }) {
  const contextValue = {};
  console.log('Parent is rendered');
  return (
    <div className="parent">
      <Context.Provider value={contextValue}>
        <ChildA />
        {children}
        {lastChild}
      </Context.Provider>
    </div>
  );
}

function ChildA() {
  console.log('ChildA is rendered');
  return <div className="childA"></div>;
}

function ChildB() {
  console.log('ChildB is rendered');
  return <div className="childB"></div>;
}

function ChildC() {
  console.log('ChildC is rendered');
  const value = useContext(Context);
  return <div className="childC"></div>;
}
```

위 상황에서 어떠한 이유로 `Parent` 컴포넌트가 리렌더링돼야 하는 상황이라고 생각해 보겠습니다.

리렌더링이 발생하면 참조 자료형인 `contextValue`의 참조값이 새롭게 생성되고 결국 **context 프로바이더가 제공하고 있는 value가 달라지게 됩니다.**

그럼 해당 프로바이더로 감싸진 `ChildA`, `ChildB`(children), `ChildC`(lastChild) 컴포넌트는 모두 리렌더링되는 것일까요?

![스크린샷 2023-12-23 오후 4.28.10.png](/jihyun/react-hook/img/4.28.10.png)

Context를 사용하면 불필요한 렌더링을 발생시키니 감싸진 컴포넌트들은 다 리렌더링 되려나라고 생각할 수도 있지만 결과를 보면 A와 C만 렌더링되는 것을 알 수 있습니다.

- `ChildA`의 경우
  - `ChildA`는 context의 값을 사용하진 않지만 **부모가 리렌더링되면 자식도 리렌더링 된다** 라는 조건에 의해 리렌더링되었습니다. 즉, `Parent`의 리렌더링으로 인해 함께 리렌더링된 것입니다.
- `ChildB`의 경우
  - `ChildB`는 context의 값을 사용하지도 않고, `Parent`의 자식 컴포넌트도 아닙니다. 따라서 리렌더링이 발생하지 않습니다.
- `ChildC`의 경우
  - `ChildC`는 `Parent`의 자식 컴포넌트는 아니지만 context 값을 사용하고 있고 해당 값이 변경되었기 때문에 리렌더링이 발생했습니다.

이를 종합해 봤을 때, 프로바이더로 감싼 컴포넌트 중에서 **context 값을 사용하는 컴포넌트만이 리렌더링된다는 것**을 알 수 있습니다.

## 불필요한 렌더링이 발생하는 예시

이번에는 context를 사용할 때 불필요한 렌더링이 발생하는 예시 코드를 보겠습니다.

`ThemeProvider`는 theme 상태를 가지고 있으며 context 프로바이더를 통해 state와 setState 함수를 하나의 객체형태로 children에게 제공하고 있습니다.

이때 value는 매 렌더링마다 재생성되지 않도록 메모이제이션되었으며 theme 상태의 영향을 받습니다.

`ConsumeTheme` 컴포넌트는 context 프로바이더가 제공하는 theme을 참조하여 렌더링합니다.

`Nothing` 컴포넌트는 context 값을 참조하지 않고 단순 문자열을 렌더링합니다.

`SetDarkmodeButton` 컴포넌트는 context 프로바이더가 제공하는 setTheme 함수를 사용하는 버튼을 리턴하며 theme을 참조하진 않습니다.

```jsx
const ThemeContext = createContext({ theme: 'light', setTheme: (theme: string) => {} });
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const value = useMemo(() => ({ theme, setTheme }), [theme]);

  console.log('ThemeProvider 컴포넌트 렌더링됨');
  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

function ConsumeTheme() {
  const { theme } = useContext(ThemeContext);

  console.log('ConsumeTheme 컴포넌트 렌더링됨');
  return <div>current theme is {theme}</div>;
}

function SetDarkmodeButton() {
  const { setTheme } = useContext(ThemeContext);

  console.log('SetDarkmodeButton 컴포넌트 렌더링됨');
  return <button onClick={() => setTheme('dark')}>다크 모드로 변경</button>;
}

function Nothing() {
  console.log('Nothing 컴포넌트 렌더링됨');
  return <div>nothing...</div>;
}

export default function App() {
  return (
    <ThemeProvider>
      <ConsumeTheme />
      <Nothing />
      <SetDarkmodeButton />
    </ThemeProvider>
  );
}
```

위 상황에서 `SetDarkmodeButton`을 클릭하여 theme을 변경해 보겠습니다.

버튼을 클릭하면 theme이 light에서 dark로 변경되고 이에 따라 프로바이더가 제공하는 value가 달라지게 됩니다.

이러한 상황에서 어떤 컴포넌트가 렌더링되는지 실행해 확인해 보겠습니다.

![Dec-23-2023 18-38-30.gif](/jihyun/react-hook/img/Dec-23-2023_18-38-30.gif)

버튼을 클릭하면, `Nothing` 컴포넌트를 제외하고 모두 리렌더링되는 것을 알 수 있습니다.

- `ThemeProvider`의 경우
  - `ThemeProvider`가 가진 theme 상태가 업데이트되었기 때문에 리렌더링이 발생합니다. 상태가 변했기 때문에 메모이제이션 했던 value도 새로운 참조값으로 변경됩니다.
- `ConsumeTheme`의 경우
  - 참조하고 있는 context 값이 변경되었고 이를 사용하는 컴포넌트기 때문에 리렌더링이 발생합니다.
- `Nothing`의 경우
  - context와 상관없는 컴포넌트기 때문에 프로바이더로 감싸졌더라도 리렌더링이 발생하지 않습니다.
- `SetDarkmodeButton`의 경우
  - setTheme은 setState function으로 참조값이 변경되지 않지만, useContext에서 반환하는 value의 참조값이 변경된 상태기 때문에 리렌더링이 발생합니다.
    > React guarantees that `setState` function identity is stable and won’t change on re-renders. This is why it’s safe to omit from the `useEffect` or `useCallback`dependency list. - [react 공식 문서](https://legacy.reactjs.org/docs/hooks-reference.html)

추가로, 버튼을 2번 클릭한다면 theme이 dark에서 dark로 변경 즉, 유지되기 때문에 의도된 동작대로 `ThemeProvider`만 한 번 실행되며, 3번째 이상 클릭할 때는 아무것도 리렌더링되지 않습니다.

> If your update function returns the exact same value as the current state, the subsequent rerender will be skipped completely. - [react 공식 문서](https://legacy.reactjs.org/docs/hooks-reference.html?fbclid=IwAR3rAXffa8s5dx1DSJgf_KNnvm9Roz39goXCinDpD3QJ1zl76OeKds7pMTs#functional-updates)

그런데, theme의 값과 관계 없이 항상 동일하게 렌더링되는 `SetDarkmodeButton`이 theme이 변경될 때마다 함께 리렌더링된다는 문제가 있습니다.

만약 Context에서 관리하는 상태가 자주 변경되는 상태라면 해당 컴포넌트는 계속 자주 불필요하게 렌더링이 발생할 것입니다.

이런 부분이 **Context를 사용할 때 불필요한 렌더링이 발생한다는 단점**으로 보여집니다.

# 불필요한 리렌더링을 막기 위한 값과 업데이트 분리하기

위 문제점을 해결하기 위해 `값을 위한 context`와 `업데이트를 위한 context` 이렇게 2가지로 분리하여 코드를 작성하면 **변경된 값을 사용하는 컴포넌트만 리렌더링 되도록** 할 수 있습니다.

위 코드를 리팩토링한 코드를 보겠습니다.

이번에는 useState가 반환한 배열의 각 요소를 `ThemeValueContext`와 `ThemeUpdateContext` 이렇게 두 개의 context로 분리하여 children에게 제공합니다.

```jsx
const ThemeValueContext = createContext('light');
const ThemeUpdateContext = createContext((theme: string) => {});
function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState('light');

  console.log('ThemeProvider 컴포넌트 렌더링됨');
  return (
    <ThemeValueContext.Provider value={theme}>
      <ThemeUpdateContext.Provider value={setTheme}>{children}</ThemeUpdateContext.Provider>
    </ThemeValueContext.Provider>
  );
}

function ConsumeTheme() {
  const theme = useContext(ThemeValueContext);

  console.log('ConsumeTheme 컴포넌트 렌더링됨');
  return <div>current theme is {theme}</div>;
}

function SetDarkmodeButton() {
  const setTheme = useContext(ThemeUpdateContext);

  console.log('SetDarkmodeButton 컴포넌트 렌더링됨');
  return <button onClick={() => setTheme('dark')}>다크 모드로 변경</button>;
}

function Nothing() {
  console.log('Nothing 컴포넌트 렌더링됨');
  return <div>nothing...</div>;
}

export default function App() {
  return (
    <ThemeProvider>
      <ConsumeTheme />
      <Nothing />
      <SetDarkmodeButton />
    </ThemeProvider>
  );
}
```

위 상황에서 `SetDarkmodeButton`을 클릭하여 theme을 변경해 보겠습니다.

![Dec-23-2023 22-13-06.gif](/jihyun/react-hook/img/Dec-23-2023_22-13-06.gif)

버튼을 클릭하면, `ThemeProvider`와 `ConsumeTheme` 컴포넌트만 리렌더링 됩니다. 이전 예시와는 달리 `SetDarkmodeButton` 컴포넌트는 리렌더링되지 않았어요! 🎉

`SetDarkmodeButton` 컴포넌트가 리렌더링되지 않은 이유는 `ThemeUpdateContext`에 변경이 없었기 때문입니다. 해당 context는 setTheme 함수를 값으로 사용하고 있는데 setTheme 함수는 상태의 setter 함수이므로 렌더링 간 그 참조값이 변경되지 않고 유지됩니다.

# 결론

Context API를 사용할 때 불필요한 리렌더링이 발생하는 상황에 대해 살펴봤습니다. 상태가 몇 번 변경되지 않는다면 큰 문제가 되지 않을 수 있지만, 상태가 자주 변경되거나 SetDarkmodeButton 컴포넌트가 리턴하는 자식 컴포넌트가 아주 많은 경우 **불필요한** 리렌더링이 발생해 성능 이슈로 연결될 수 있겠습니다.

위처럼 context를 분리하여 사용한다면 값의 변경으로 리렌더링이 필요한 컴포넌트만 리렌더링될테니 성능도 함께 챙길 수 있을 것 같네요.

Context를 사용하는 코드를 보고 한 눈에 이해하기 어려웠는데, 이제는 왜 메모이제이션이 들어가고 context를 분리해 사용하기도 하는지 그 이유를 알 것 같습니다.
