# useState로 상태 관리 로직 작성해 보기

> *모던 리액트 Deep Dive 책*의 상태 관리 로직 구현 파트를 정리한 포스팅입니다.

기본적으로 좋은 상태 관리 로직은 다음과 같은 조건을 만족해야 합니다.

1. 컴포넌트 외부 어딘가에 상태를 두고 여러 컴포넌트가 같이 쓸 수 있어야 한다.
2. 외부에 있는 상태를 사용하는 컴포넌트는 상태의 변화를 알아챌 수 있어야 한다. 즉, 상태가 변할 때마다 리렌더링이 일어나서 최신 상태 값을 기준으로 렌더링해야 한다. 그리고 이 상태 감지는 모든 컴포넌트에서 동일하게 작동해야 한다.
3. 상태가 객체인 경우, 객체 프로퍼티 중 컴포넌트가 사용하지 않는 값이 변했다면 리렌더링이 발생해선 안 된다.

위와 같은 조건을 만족할 수 있으며, 컴포넌트 레벨의 지역 상태를 벗어나는 상태 관리 코드를 작성해 보겠습니다.

---

전체적인 컨셉은 다음과 같습니다.

> 상태를 내부 변수로 갖는 스토어를 컴포넌트 바깥에 두고, 그 스토어에서 값을 가져와 useState로 관리하는 커스텀 훅을 만든다.
> 컴포넌트는 커스텀 훅이 반환하는 상태와 업데이트 함수를 사용하여 렌더링한다.

# 상태 관리 로직 작성하기

## 1. 스토어 만들기

먼저 전역적으로 사용할 스토어 코드를 작성합니다.

```tsx
// useState를 사용하는 것과 비슷하게 초기값 또는 게으른 초기화를 위한 함수를 받음
type Initializer<T> = T extends any ? T | ((prev: T) => T) : never;

type Store<State> = {
  get: () => State; // 스토어 내부의 변수 값을 리턴
  set: (action: Initializer<State>) => State; // 스토어 내부의 변수 값을 셋팅(변경)
  subscribe: (callback: () => void) => () => void; // 해당 스토어의 값을 참조하는 경우 호출하는 함수
};

export const createStore = <State extends unknown>(
  initialState: Initializer<State> // 스토어에 저장할 값 (또는 게으른 초기화를 위한 함수)
): Store<State> => {
  // State 타입을 갖는 값을 저장하고 있는 스토어를 리턴
  let state = typeof initialState !== 'function' ? initialState : initialState(); // initialState가 함수면 게으른 초기화, 함수가 아니면 값 저장
  const callbacks = new Set<() => void>(); // 스토어 내부 값이 변경되면 실행할 함수의 목록

  const get = () => state; // 스토어의 값을 리턴하는 함수
  const set = (nextState: State | ((prev: State) => State)) => {
    // nextState가 함수라면 함수가 리턴하는 값으로 변경하고 함수가 아니라면 해당 값으로 변경
    state = typeof nextState === 'function' ? (nextState as (prev: State) => State)(state) : nextState;

    callbacks.forEach((callback) => callback()); // 값을 변경한 다음 해당 스토어를 구독하는 컴포넌트 리렌더링
    return state; // 변경된 상태 리턴
  };

  const subscribe = (callback: () => void) => {
    // 스토어 값이 변경되면 실행할 콜백을 받아
    callbacks.add(callback); // 목록 집합에 넣어두고
    return () => {
      // 해당 상태 값으로 렌더링이 끝난 다음 해당 콜백 제거
      callbacks.delete(callback);
    };
  };

  return { get, set, subscribe };
};
```

## 2. 스토어를 활용하는 커스텀 훅 만들기

스토어에 저장한 상태 값을 참조하고 업데이트할 수 있는 로직을 추상화한 커스텀 훅 코드를 작성합니다.

```tsx
export const useStore = <State extends unknown>(store: Store<State>) => {
  const [state, setState] = useState<State>(() => store.get()); // 스토어에서 값을 가져와 게으른 초기화. 이 useState가 컴포넌트의 렌더링을 트리거

  useEffect(() => {
    // 스토어의 현재 값을 가져와 setState를 수행하는 함수를 스토어의 subscribe로 등록
    // createStore 내부 값이 변경될 때마다 subscribe에 등록된 함수를 실행하기 때문에
    // store 값이 변경될 때마다 state의 값이 변경되는 것을 보장받음
    const unsubscribe = store.subscribe(() => {
      // store 값이 변경됐을 때 실행될 콜백
      setState(store.get());
    });
    return unsubscribe;
  }, [store]);

  return [state, store.set] as const;
};
```

위 훅을 사용하는 컴포넌트는 전역 값을 변경하려면 `store.set`을 호출해야 하고, `store.set`을 호출하면 실행할 콜백으로 넘겨준 `setState(store.get())` 함수가 실행되기 때문에 (리렌더링을 트리거하는 useState의 setter 함수가 실행되어) 해당 컴포넌트가 리렌더링되는 로직입니다.

## 3. 원하는 값이 변경됐을 때만 리렌더링되도록 셀렉터 만들기

위 로직까지 작성하여 사용한다면 스토어의 값이 변경됐을 때 리렌더링이 발생하여 변경된 값으로 새로운 렌더링을 할 수 있으나, 컴포넌트에서 필요하지 않은 값이 변경됐을 때도 리렌더링이 발생합니다.

이처럼 불필요하게 발생하는 리렌더링을 줄이기 위해 셀렉터 코드를 작성합니다.

```tsx
export const useStoreSelector = <State extends unknown, Value extends unknown>(
  store: Store<State>,
  selector: (state: State) => Value // 스토어의 값에서 어떤 값을 가져올지 정의하는 함수
) => {
  const [state, setState] = useState(() => selector(store.get())); // 스토어의 값(store.get())에서 가져오려는 값으로 게으른 초기화

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      // 스토어의 값이 변경됐을 때 실행될 콜백
      const value = selector(store.get()); // 스토어의 변경된 값을 가져와서
      setState(value); // 지역 state 업데이트를 통해 리렌더링 트리거
    });
    return unsubscribe;
  }, [store, selector]);

  return state;
};
```

# 컴포넌트에서 위 로직 사용하기

스토어를 만들어 전역적으로 관리할 값을 관리하고 `useStoreSelector` 훅을 사용해 컴포넌트에서 필요로 하는 값만 사용하도록 합니다.

`useStoreSelector` 훅 내부에서 useEffect를 통해 `store.subscribe`를 호출하고 있기 때문에 `store.set` 함수를 통해 스토어의 값이 업데이트된 경우 리렌더링이 발생합니다.

```tsx
const store = createStore({ count: 0, test: 'hi' }); // { count: 0, text: 'hi' }를 초기값으로 갖는 객체를 관리하는 스토어 생성

function Counter() {
  const counter = useStoreSelector(
    store, // 스토어에서
    useCallback((store) => store.count), // count 값만 선택해 옴
    []
  );

  function handleClick() {
    store.set((prev) => ({ ...prev, count: prev.count + 1 })); // 스토어에서 관리하는 내부 값 업데이트
    // 내부 로직에 따라 리렌더링이 트리거될 것 (useSelector 훅 내부에서 useEffect를 통해 subscribe 호출)
  }

  return (
    <>
      <h3>{counter}</h3>
      <button onClick={handleClick}>+</button>
    </>
  );
}

const textSelector = (state: ReturnType<typeof store.get>) => state.text; // 스토어에서 text 값만 선택해 오는 셀렉터 함수

function TextEditor() {
  const text = useStoreSelector(store, textSelector); // text만 선택해 옴

  function handleChange(e: ChangeEvent<HTMLInputElement>) {
    store.set((prev) => ({ ...prev, text: e.target.value })); // 스토어 내부 값 업데이트
    // 내부 로직에 따라 리렌더링이 트리거될 것 (useSelector 훅 내부에서 useEffect를 통해 subscribe 호출)
  }

  return (
    <>
      <h3>{text}</h3>
      <input value={text} onChange={handleChange} />
    </>
  );
}
```

이때 selector 함수는 `useStoreSelector` 훅 내부의 useEffect의 의존성 배열에 들어가 있기 때문에 함수의 참조값이 변경된다면 effect가 계속 실행되어 `store.subscribe`가 여러 번 호출됩니다. 그렇기 때문에 컴포넌트 외부에 함수를 정의하거나 useCallback 훅을 활용하여 참조 값이 변경되지 않도록 해야 합니다.

# 마치며

리액트 외부에서 관리되는 값에 대한 변경을 추적하고, 이를 리렌더링까지 할 수 있는 로직에 대해 알아봤습니다.

useState 훅을 마치 상태 관리 라이브러리처럼 사용하는 예제였는데요.

위 로직의 경우 훅과 스토어를 사용하는 구조는 반드시 하나의 스토어만을 가져야하기 때문에 여러 개의 스토어를 사용하려면 `createStore`와 `useStore`(또는 `useStoreSelector`)를 여러 번 호출해야 한다는 단점이 있습니다.

때문에 프로덕션에 도입하기 보단 상태 관리 로직을 어떻게 구현할 수 있는지 생각해 보는 용도로 활용하면 되겠습니다.
