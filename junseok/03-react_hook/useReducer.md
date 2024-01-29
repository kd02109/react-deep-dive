## useReducer

```tsx
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

#### 매개 변수
- **reducer** : `useReducer`의 기본 action을 정의하는 함수입니다. 해당 `reducer`는 `useReducer`의 첫 번째 인수로 넘겨주어야 합니다. 
- **initialArg** | **initialState**: 두 번째 인수로, useReducer의 초깃값을 의미합니다. 
- **init** : `useState`의 인수로 함수를 넘겨 줄 떄 처럼, 초깃값을 지연 생성시키고 싶을 떄 사용하는 함수입니다. 이 함수는 필수 값이 아닙니다. 만약 인수로 넘겨주는 함수가 존재한다면, useState와 동일하게 **게으른 초기화**가 발생합니다. 이때 **initialArg | initialState를 인수로 init 함수**가 실행됩니다.  

#### 반환 값
- **state** : 현재 Reducer가 가지고 있는 값을 의미합니다. 배열의 첫 번째 요소입니다. 
- **dispatcher** : state를 업데이트하는 함수입니다. 배열의 두 번째 요소입니다. dispatcher는 인자로 action을 넘겨 줍니다. 해당 action은 state를 변경할 수 있는 액션을 의미합니다. 

## 예시 코드 
```tsx
import { useReducer } from 'react';

type State = number;
type Action = { type: 'ADD' | 'MINUS' | 'RESET' };

const initialState: State = 0;
function reducer(state: State, action: Action) {
  switch (action.type) {
    case 'ADD':
      return state + 1;
    case 'MINUS':
      return state - 1;
    case 'RESET':
      return 0;
  }
}

function App() {
  const [count, dispatch] = useReducer(reducer, initialState);
  const handlePlus = () => {
    dispatch({ type: 'ADD' });
  };

  const handleMinus = () => {
    dispatch({ type: 'MINUS' });
  };
  return (
    <>
      <h1>{count}</h1>
      <button onClick={handlePlus}>plus</button>
      <button onClick={handleMinus}>minus</button>
    </>
  );
}

export default App;
```

## useReducer로 useState 구현하기
```tsx
import { useReducer } from 'react';

// reducer 함수 생성
function reducer<T>(prevState: T, newState: T) {
  return typeof newState === 'function' ? newState(prevState) : newState;
}

// lazy initialzation을 위한 init 함수 생성
function init<T>(initialArg: T): T {
  return typeof initialArg === 'function' ? initialArg() : initialArg;
}

// useReducer 호출을 통한 useState 구현 
function useCustomState<T>(
  initialArg: T | (() => T),
): [T, React.Dispatch<React.SetStateAction<T>>] {
  return useReducer(reducer, initialArg, init);
}

// 실제 실행
function App() {
  const [number, setNumber] = useCustomState(() => 1);
  const handlePlus = () => {
    setNumber(prev => prev + 1);
  };
  return (
    <>
      <h1>{number}</h1>
      <button onClick={handlePlus}>plus</button>
    </>
  );
}

export default App;

```
- reducer는 전달된 setState에 전달된 인자가 함수일 경우 이전 prev 값으로 함수를 실행합니다. 그렇지 않은 경우 전달 받은 newState를 반환합니다. 

- init 함수는 lazy initialzation을 실행합니다. 전달된 initialArg가 함수인 경우 해당 함수를 실행하고, 그렇지 않은 경우 initialArg를 반환합니다. 

- useCustomState에서는 useReducre 실행을 반환합니다. 이때 해당 반환 타입을 useState와 동일하게 설정합니다.
