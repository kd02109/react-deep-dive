## 상태 관리의 필요성

컴퓨터 공학에서 상태를 관리한다는 것은 data의 변화를 추적하고 해당 data를 저장하고 변경내역을 추적하는 행위입니다.

이를 좁혀서 웹 개발에서 FE가 하는 상태관리에는 다음과 같은 데이터를 관리해야 합니다.
- UI: 상호 작용이 가능한 모든 요소 (다크모드, input, btn, form, 알림)
- URL: URL의 쿼리 또한 상태관리의 대상이 됩니다. URL의 주소에 따라 사용자에게 보여주어야 할 데이터 혹은 화면 구조가 변경되기 때문입니다. 
- api: 서버에서 가지고 오는 모든 값을 의미합니다. 

기본적으로 리엑트 만으로도 상태를 관리할 수 있습니다. 특히 함수형 컴포넌트에서 `useState` `useReducer`를 활용해서 상태를 관리할 수 있습니다. 하지만 이러한 상태는 전역에서 관리 되지 않는, 컴포넌트 별로 관리하는 지역변수입니다. 

드렇다면, 모든 컴포넌ㅌ느에서 공통적으로 관리되어야 하는 상태를 쉽게 관리하기 위해서 무엇을 고려해야 할까?

1. 컴포넌트 외부 어딘가에 상태를 두고 여러 컴포넌트에서 이를 함께 사용할 수 있어야 합니다. 
2. 외부에 있는 상태를 사용하는 컴포넌트는 상태의 변화를 알아챌 수 있어야 합니다. 상태가 변화될 때, 리렌더링이 일어나서 컴포넌트를 최신 상태 값 기준으로 렌더링 해야 합니다. 이는 상태를 변경시키는 컴포넌트 뿐만 아니라 이 상태를 참조하는 모든 컴포넌트에서 동일하게 작동해야 합니다. 
3. 상태가 원시 값이 아닌 객체일 경우, 객체에 본인이 참조하고 있는 값이 변하지 않는 경우에는, 컴포넌트의 리렌더링이 발생하면 안됩니다. (최적화)


## useState를 활용한 store 만들기 

```tsx
/* eslint-disable @typescript-eslint/no-unnecessary-type-constraint */
/* eslint-disable @typescript-eslint/no-explicit-any */

export type Initializer<T> = T extends any ? T | ((prev: T) => T) : never;

export type Store<State> = {
  get: () => State;
  set: (action: Initializer<State>) => State;
  subscribe: (callbak: () => void) => () => void;
};

export const createStore = <State extends unknown>(
  initializer: Initializer<State>,
): Store<State> => {
  let state = typeof initializer === 'function' ? initializer() : initializer;

  const callbacks = new Set<() => void>();

  const get = () => state;
  const set = (nextState: State | ((prev: State) => State)) => {
    state = typeof nextState === 'function' ? nextState(state) : nextState;

    callbacks.forEach(callback => callback());
    return state;
  };

  const subscribe = (callback: () => void) => {
    callbacks.add(callback);
    return () => {
      callbacks.delete(callback);
    };
  };
  return { get, set, subscribe };
};

```

1. store의 초깃 값은 lazy initialization, 또는 원시 값으로 받아 store의 기본 값을 초기화 합니다. 
2. 1번에서 받은 초기 값을 저장합니다. 
3. 컴포넌트로 넘겨받는 콜백함수를 저장하기 위해 callbacks를 Set으로 선언합니다. Set은 원시값이나 객체에 관계없이 유일한 갑ㅅ을 저장할 수 있어 중복 없이 콜백 함수를 저장하는 용도로 사용됩니다. 
4. get을 함수로 만들어 매번 최산값을 가져올 수 있도록 합니다. 
5. set을 만들어 새로운 값을 넣을 수 있도록 합니다.  useState의 두번째 인수와 마찬가지로 함수일 수도, 단순히 값을 받을 수 있습니다. 그리고 callbacks를 순회해 등록된 모든 콜백을 실행합니다. 
6. subcribe는 callbacks set에 callback을 등록할 수 있는 함수입니다. 반환 값으로 callback을 삭제하는 함수를 반환합니다. 이는 callbacks에 callback이 무한히 추가되는 것을 방지합니다. useEffect의 클린업 함수와 같은 역할을 합니다. 

###  useStore
해당 createStore을 활용한 useStore를 만들었습니다. 

```tsx
import { useEffect, useState } from 'react';
import { Store } from '@/store/store';

export const useStore = <State extends unknown>(store: Store<State>) => {
  const [state, setState] = useState(() => store.get());

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      setState(store.get());
    });
    return unsubscribe;
  }, [store]);

  return [state, store.set] as const;
};

```

1. hook의 인수로 사용할 store를 받습니다. 
2. 이 스토어 값을 초기 값으로 하는 useState를 만듭니다. 해당 state가 컴포넌트의 렌더링을 유도합니다.
3. useEffect는 store의 현재 값을 가져와 setState를 수행하는 함수를 store의 subscribe로 동록합니다.  creaeState 내부에서 값이 변경될 때마다 subscribe에 등록된 함수를 실행하므로 useStore 내부에서는 store의 값이 변경될 때만다 state의 값이 변경되는 것을 보장받을 수 있습니다. 
4. useEffect의 클린업 함수로 callbacks에서 해당 함수를 제거해 callback이 계속해서 쌓이는 문제를 해결합니다. 

### useStoreSelector
```tsx
import { useEffect, useState } from 'react';
import { Store } from '@/store/store';

export const useStoreSelector = <State extends unknown, Value extends unknown>(
  store: Store<State>,
  selector: (state: State) => Value,
) => {
  const [state, setState] = useState(() => selector(store.get()));

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      const value = selector(store.get());
      setState(value);
    });
    return unsubscribe;
  }, [store, selector]);

  return state
};

```
useStore와 달리 selector라는 callback 함수를 두 번째 인수로 받습니다. 해당 함수는 store의 상태에 있는 특정 값을 가져오는 함수입니다. 해당 함수는 store.get()을 실행합니다.

1. useStoreSelector를 사용할 경우 우선 어떤 store를 참고할지, 해당 stroe에서 어떤 값을 사용할지 결정하는 selector 값을 인자로 받습니다.

2. 인자로 전해진 selector를 통해서 store의 특정 값을 state에 저장합니다.
3. store, selector가 변경될 경우 useEffect가 동작합니다. 
    - subsirbe를 수행합니다. 이를 통해 callbacks에 callback을 저장합니다. 
    - 해당 callback 함수는 store.set에 의해 실행이 됩니다. 즉 store.set이 state 변화를 트리거하는 메서드입니다. 
    - 이때 모든 callback 함수가 실행되어도, 참조하고 있는 state의 변화가 없다면, 리렌더링이 발생하지 않습니다. 