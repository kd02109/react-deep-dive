## useMemo_useCallback_React.memo

아래와 같은 코드가 있습니다. 
```tsx
function FirstChild({ trigger }: { trigger: boolean }) {
  return <div>{`Trigger: ${trigger}`}</div>;
}

function SecondChild({
  handleClick,
}: {
  handleClick: () => void;
}) {
  return (
    <>
      <ul>
        {new Array(20).fill(0).map((_, idx) => (
          <li key={idx}>{idx}</li>
        ))}
      </ul>
      <button onClick={handleClick}>Trigger</button>
    </>
  );
}

function App() {
  const [trigger, setTriger] = useState(false);
  const handleClick = () => {
    setTriger(prev => !prev);
  };
  return (
    <>
      <FirstChild trigger={trigger} />
      <SecondChild handleClick={handleClick} />
    </>
  );
}

export default App;
```
해당 App은 두개의 자식 컴포넌트를 가지고 있습니다. secondChild에 `handleClick`함수를 전달하고 해당 함수는 trigger 값을 변경합니다. 이에 따라서 App의 setState가 실행이 되고 리렌더링이 발생합니다. 해당 리렌더링을 막기 위해 useCallback을 활용합니다. 


```tsx
function FirstChild({ trigger }: { trigger: boolean }) {
  return <div>{`Trigger: ${trigger}`}</div>;
}

function SecondChild({
  handleClick,
}: {
  handleClick: () => void;
}) {
  return (
    <>
      <ul>
        {new Array(20).fill(0).map((_, idx) => (
          <li key={idx}>{idx}</li>
        ))}
      </ul>
      <button onClick={handleClick}>Trigger</button>
    </>
  );
}

function App() {
  const [trigger, setTriger] = useState(false);
  const handleClick = useCallback(() => {
    setTriger(prev => !prev);
  }, []);
  return (
    <>
      <FirstChild trigger={trigger} />
      <SecondChild handleClick={handleClick} />
    </>
  );
}

export default App;
```
useCallback을 통해 함수의 메모리 값이 변화하지 않고 유지가 되지만, 상위 state가 변동되어 변화와 연관이 없는 secondChild에도 리렌더링은 여전히 발생합니다. 해당 문제는 react의 Render Phase 와 Commit Phase를 이해해야 합니다. 
- Render phase는  **VDOM을 재조정(Reconciliation)하는 일련의 과정**입니다. 컴포넌트 호출은 Render phase에서 실행되며 호출이 곧 화면 반영을 나타내는 것은 아닙니다. 이전 포스트에서 언급한 용어인 렌더링에 빗대어 보자면 컴포넌트가 리-렌더링 된다는 말은 컴포넌트가 호출되고 그 결과가 VDOM에 반영된다는 것이지 DOM에 마운트되어 페인트 된다는 뜻은 아닙니다.
- Commit phase는 **Render phase에서 재조정된 VDOM을 DOM에 적용하고 라이프 사이클을 실행하는 단계**입니다. 여기서도 마찬가지로 DOM에 마운트된다는 것이지 페인트 된다는 건 아닙니다. **이 단계가 끝나고 리액트에서 콜 스택을 비워줘야지만 브라우저에서 화면을 페인트** 할 수 있게 됩니다.

즉 useCallback을 적용하였어도 Render phase는 진행이 되고 있는 상황입니다. 하지만 virtual DOM의 변화는 없기 때문에 DOM 상의 변화는 발생하지 않습니다.

따라서 위의 문제를 막기 위해서는 React.memo를 통해 prop의 변화가 없다면 리렌더링이 발생하지 않도록 해야합니다.
```tsx
import React from 'react';
const SecondChild = ({ handleClick }: { handleClick: () => void }) => {
  console.log('SecondChild render');
  return (
    <>
      <ul onClick={handleClick}>
        {new Array(20).fill(0).map((_, idx) => (
          <li key={idx}>{idx}</li>
        ))}
      </ul>
    </>
  );
};
export default React.memo(SecondChild);
```
이를 통해 버튼으로 인해서 상위 state가 변경되어도 변화가 필요없는 `Second Child`에 리렌더링이 발생하지 않게 됩니다.

이러한 useCallback은 함수를 재사용하기 위함입니다. useMemo는 값을 재사용하기 위함인데, useCallback은 useMemo를 활용해서 구현할 수 있습니다.

```tsx
export function useCallback(callback, agrs){
    currentHook = 8
    return useMemo(()=>callback, args)
}
```

react에서 useCallback을 별도로 구현한 이유는 사용의 편의성을 위해 구현한 것으로 보입니다.