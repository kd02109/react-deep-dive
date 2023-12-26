## react hook의 규칙

훅의 사용에는 몇몇 규칙이 있습니다. 이러한 규칙을 react-of-hooks라고 하며, 이와 관련된 ESLing 규칙인 **react-hooks/rules-of-hoos**도 존재합니다.

1. 최상위에서만 훅을 호출해야 합니다. 반복문이나 조건문, 중첩된 함수 내에서 훅을 실행할 수 없습니다. 이 규칙을 따라야만 컴포넌트가 렌더링 될 때마다 항상 동일한 순서로 훅이 호출되는 것을 보장할 수 있습니다.

**리엑트에서 훅이 호출된 순서는 매우 매우 중요합니다.** 훅이 호출 된 순서에 따라 파이버 객체의 링크드 리스트의 호출 순서에 따라 저장되기 때문입니다. 하지만 만약 조건문, 반복문 내에서 훅이 호출되어 호출 순서가 일정하지 않다면, 리엑트 훅 호출 순서를 보장할 수 없게 됩니다. 

2. 훅을 호출할 수 있는 것은 리엑트 함수현 컴포넌트, 혹은 사용자 정의 훅의 두가지 경우 뿐입니다. 일반 자바스크립트 함수에서는 훅을 사용할 수 없습니다. 


## 사용자 정의 훅(Custom Hook)
사용자 정의 훅은 리엑트에서 제공하는 훅을 기반으로 해서 사용자의 편의와 필요에 따라 활용할 수 있는 훅을 만드는 것입니다. 이러한 사용자 훅은 반드시 use로 시작하는 함수를 만들어야 한다는 것입니다. 리엑트 훅의 이름은 use로 시작한다는 규칙이 있으며, 사용자 정의 훅도 이러한 규칙을 준수하여 개발 시 해당 함수가 리엑트 훅이라는 것을 바로 인식할 수 있다는 장점이 있습니다. 

사용자 정의 훅은 그 자체로는 렌더링에 영향을 미치지 못합니다. 따라서 컴포넌트 내부에 미치는 영향을 최소화해 개발자가 훅을 원하는 바양으로만 사용할 수 있게 하는 장점이 있습니다. **따라서 단순히 컴포넌트 전반에 걸쳐 동일한 로직으로 값을 제공하거나 특정한 훅의 작동을 취하게 하고 싶다면 사용자 정의 훅을 사용하는 것이 좋습니다.**


이러한 다양한 사용자 정의 훅을 제공하는 저장소로 [usereHooks](https://usehooks.com/), [react-use](https://github.com/streamich/react-use), [ahooks](https://ahooks.js.org/) 등이 있습니다. 


### 사용자 정의 훅 활용 예시 useResize
- 윈도우 화면 크기가 변경 될 때, 스크롤 이벤트를 진행하는 ref를 반환하는 훅입니다.
```tsx
'use client';
import { useState, useEffect, useRef } from 'react';
export default function useResize(answers: string[]) {
  // Ref for the chat container
  const chatDivRef = useRef<HTMLDivElement>(null);

  // State to track window size
  const [windowSize, setWindowSize] = useState({
    width: 0,
    height: 0,
  });

  // Effect to add the resize event listener
  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    window.addEventListener('resize', handleResize);

    // Clean up the event listener
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  // Effect to scroll to the bottom when window size changes
  useEffect(() => {
    if (chatDivRef.current) {
      chatDivRef.current.scrollTop = chatDivRef.current.scrollHeight;
    }
  }, [windowSize]); // Dependency on windowSize

  return chatDivRef;
}

```

## 고차 컴포넌트 (HOC, Higher Ordef Component)
**고차 컴포넌트는 컴포넌트 자체의 로직을 재사용하기 위한 방법입니다.** 해당 고차 함수 방법은 고차 함수의 일종입니다. 즉 자바스크립트 함수의 일급 객체 특징을 활용한 방법이기에, 자바스크립트 환경 어디서든 활용할 수 있습니다. 대표적인 예시로 **React.memo**가 있습니다.

### 고차 함수의 예시
고차함수는 함수를 인자로 받거나, 결과를 반환하는 함수를 의미합니다. 자바스크립트의 대표적인 고참함수로 배열에서 주로 사용하는 map, filter, find 등의 메서드가 있습니다. 해당 함수는 모두 함수를 인자로 받는 함수로 고참함수에 해당합니다.

```js
[0,1,2,3].map(item => item * 2)
```

### 고차 컴포넌트 만들기 
**고차 컴포넌트의 경우 with으로 시작하는 이름을 사용해야 합니다.** 이는 강제된 규칙은 아니지만, 리엑트 커뮤니티에 널리 퍼진 일종의 관습이라고 할 수 있습니다. \

함**수형 컴포넌트의 반환값, 즉 렌더링의 결과물에도 영향을 미치는 공통 로직이라면 고차 컴포넌트를 사용하는 것이 좋습니다.** 만약 api의 작업 혹은 로그인 여부 처리를 해야 할경우, 로그인 실패, 로딩 중 등의 별도의 컴포넌트가 필요로 하고 해당 훅을 사용하는 컴포넌트 별로 공통된 컴포넌트 관련 로직을 반복적으로 작성해야 합니다. 하지만, 고차 컴포넌트를 활용할 경우 이러한 .불필요한 과정을 줄일 수 있습니다. 

```tsx
/* eslint-disable @typescript-eslint/no-explicit-any */
import React, { useState } from 'react';

function withLogger(WrappedComponent: (data: any) => JSX.Element) {
  return function WithLogging(props: { url: string }) {
    const [data, setData] = useState('');
    const [isLoading, setIsLoading] = useState(true);
    React.useEffect(() => {
      const webAbort = new AbortController();
      (async () => {
        const json = await fetch(props.url, { signal: webAbort.signal });
        const data = await json.json();
        setData(data[0]);
        setIsLoading(false);
      })();
      return () => {
        webAbort.abort();
      };
    }, [props.url]);

    if (isLoading) {
      return <h1>Loading</h1>;
    }

    return <WrappedComponent data={data} />;
  };
}

// Usage
const MyComponent = ({ data }: { data: any }) => {
  return (
    <div>
      <img src={data.url} alt="image" />
    </div>
  );
};

const EnhancedComponent = withLogger(MyComponent);

export default function App() {
  return (
    <EnhancedComponent url={'https://api.thecatapi.com/v1/images/search'} />
  );
}
```

이러한 고차 컴포넌트를 사용할때 주의할 점은 부수 효과를 최소화 해야 합니다. **고차 컴포넌트는 반드시 컴포넌트를 인수로 받게 됩니다.** 이때 컴포넌트의 props를 임의로 수정, 추가, 삭제하는 일은 없어야 합니다. 만약 기존 props를 수정하거나 삭제한다면, 언제 props가 수정될지 모른다는 우려를 가지고 개발해야 하는 불편함이 발생합니다. 

**마지막으로 고차함수를 여러개 사용하게 될 경우, 복잡함이 증가하게 됩니다.**