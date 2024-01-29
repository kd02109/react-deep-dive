## 클래스형 컴포넌트의 getDerivedStateFromError 와 componentDidCatch

`getDerivedStateFromError`는 자식 컴포넌트에서 에러가 발생했을 때, 호출되는 에러 메서드입니다. 이 에러 메서드를 사용하면, 적절한 에러 처리 로직을 구현할 수 있습니다. 

`componentDidCatch`는 자식 컴포넌트에서 에러가 발생했을 때 실행이 됩니다. `componentDidCatch`는 두개의 인자를 받습니다. error와 어떤 컴포넌트가 에러를 발생시켰는지 정보를 가지고 있는 info입니다. 

### 사용 예제

```tsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Example "componentStack":
    //   in ComponentThatThrows (created by App)
    //   in ErrorBoundary (created by App)
    //   in div (created by App)
    //   in App
    logErrorToMyService(error, info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return this.props.fallback;
    }

    return this.props.children;
  }
}
```

```jsx
// 실제 사용예시
<ErrorBoundary fallback={<p>Something went wrong</p>}>
  <Profile />
</ErrorBoundary>
```

`ErrorBoundary`는 렌더링 중에 throw 된 error를 **`catch`** 하도록 동작합니다. 좀 더 친숙한 try-catch 를 생각해보면 좋을 것 같습니다. 이 동작과 비슷하게 **하위 트리 에서 발생하는 에러를 `catch` 합니다**. 만약 해당 `ErrorBoundary`에서 에러 처리를 실패하는 등을 이유로 에러를 re-throw 될 경우 가장 가까운 Error Boundary에서 catch 됩니다.

여기서 에러가 발생하게 된다면, 에러 화면을 보여주게 됩니다. 그렇다면, 해당 에러 처리를 다시 시도하고 싶을 경우에 어떻게 해야 할까요? 이는 `ErrorBoundary` 의 state 값 hasError 값을 false로 변경시켜 다시 하위 children에 리렌더링을 발생시키면 됩니다. 

```tsx
import React from 'react';
type Prop = {
  children: React.ReactNode;
};

type State = {
  hasError: boolean;
};

class ErrorBoundary extends React.Component<Prop, State> {
  constructor(props: Prop) {
    super(props);
    this.state = {
      hasError: false,
    };
  }

  // eslint-disable-next-line @typescript-eslint/no-explicit-any, @typescript-eslint/no-unused-vars
  static getDerivedStateFromError(error: any) {
    return { hasError: true };
  }

  #handleReStart() {
    this.setState({ hasError: false });
  }

  render(): React.ReactNode {
    if (this.state.hasError) {
      return (
        <div>
          Error 발생
          <button onClick={this.#handleReStart}>다시 실행하기</button>
        </div>
      );
    }
    return <>{this.props.children}</>;
  }
}

export default ErrorBoundary;
```

## react-error-boundary

[npm: react-error-boundary](https://www.npmjs.com/package/react-error-boundary)

해당 클래스형 컴포넌트를 어떻게 사용할 수 있을까 고민을 해보았을 때, 가장 유용한 사용 방법은 ajax 요청에서 요청에 실패했을 때 해당 방법을 사용하는 것이었습니다. 요청이 실패한 경우 실패한 사실을 알리고, 다시금 요청을 날리는 방법을 사용하는 것입니다. 해당 방법을 어떻게 적용하면 좋을지 고민하는 과정에서 유용한 라이브러리를 발견할 수 있었습니다. 바로 `**[react-error-boundary]**`입니다. 아직 까지 함수형 컴포넌트가 제공되지 않기 때문에, 해당 라이브러리를 사용했습니다.

해당 라이브러리를 활용해서 프로젝트에 적용을 해보았습니다. 이는 `Suspense`와 유사한 사용방법을 가지고 있습니다. 

```jsx
"use client";

import { ErrorBoundary } from "react-error-boundary";

<ErrorBoundary fallback={<div>Something went wrong</div>}>
  <ExampleApplication />
</ErrorBoundary>
```

컴포넌트를 전달할 경우 다음과 같이 처리할 수 있습니다

```jsx
"use client";

import { ErrorBoundary } from "react-error-boundary";

function Fallback({ error, resetErrorBoundary }) {
  // Call resetErrorBoundary() to reset the error boundary and retry the render.

  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre style={{ color: "red" }}>{error.message}</pre>
    </div>
  );
}

<ErrorBoundary
  FallbackComponent={Fallback}
  onReset={(details) => {
    // Reset the state of your app so the error doesn't happen again
  }}>
  <ExampleApplication />
</ErrorBoundary>;
```

fallback 과 FallbackComponent는 모두 fallback 내부에서 에러 상태일 때 표시될 컴포넌트입니다. 만약 에러가 발생하고 해당 에러를 다시 처리 하고 싶다면, 다음과 같이 처리 할 수 있습니다. fallback 컴포넌트에 전달되는 컴포넌트의 prop에는  resetErrorBoundary 가 있습니다. 해당 prop에는 리셋 함수가 있어 가장 최근에 일어난 에러가 일어난 지점을 다시 시도해볼 수 있습니다.  저는 이를 다음과 같이 적용해 보았습니다.

1. **ErrorBoundary Layer 생성**

```jsx
'use client';
import { ErrorBoundary, FallbackProps } from 'react-error-boundary';
import ResultSection from '@/components/result/ResultSection';
export default function ErrorContainer({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <ErrorBoundary FallbackComponent={OurFallbackComponent}>
      {children}
    </ErrorBoundary>
  );
}

const OurFallbackComponent = ({ error, resetErrorBoundary }: FallbackProps) => {
  return (
    <ResultSection title={'내가 한 답변'}>
      <h1 className="font-bold text-medium text-bgBrown">{error.message}</h1>
      <button
        onClick={resetErrorBoundary}
        className="border-2 border-solid px-4 py-2 rounded-xl mt-2">
        다시 시도하기
      </button>
    </ResultSection>
  );
};
```

1. **AJAX 요청시에 Error를 catch 할 수 있는 useErrorBoundary 훅을 사용**

```jsx
'use client';
import { useSearchParams } from 'next/navigation';
import { useEffect, useState } from 'react';
import { useErrorBoundary } from 'react-error-boundary';
import { getApiWhitToken } from '@/api/clientApi';
import { END_POINT } from '@/api/url';
import ResultChatContainer from '@/components/result/ResultChatContainer';
import ResultSection from '@/components/result/ResultSection';
import Spinner from '@/components/Spinner';
import { QUESTIONS_MAN, QUESTIONS_WOMAN } from '@/data/question';
import { AnswerData, Data } from '@/types/types';

export default function ResultMyChat() {
  const searchParams = useSearchParams();
  const token = searchParams.get('token');
  const [questions, setQuestion] = useState<Data[] | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const handleError = useErrorBoundary();

  useEffect(() => {
    getApiWhitToken<AnswerData>(END_POINT.getAnswerVisiting, token!)
      .then(data => {
        if (data) {
          const QUESTIONS =
            data?.user.gender === 'man' ? QUESTIONS_MAN : QUESTIONS_WOMAN;

          const questions = QUESTIONS.map(question => {
            const answers = data?.answer[question.id];
            question.answer = [...answers!];
            setIsLoading(false);
            return question;
          });
          setQuestion(questions);
        }
      })
      .catch(e => {
        // 에러 발생시 Fallback 컴포넌트를 렌더링 합니다.
        handleError.showBoundary(e);
      });

    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
    <ResultSection title={'내가 한 답변'}>
      {isLoading ? (
        <Spinner loading={isLoading} />
      ) : (
        questions?.map(question => (
          <ResultChatContainer
            key={question.id}
            chat={`${question.name} : ${question.answer.join(', ')}`}
          />
        ))
      )}
    </ResultSection>
  );
}
```

## 사용 예시 화면 

- **정상 출력**

<img src="./done.png" width="300" height="300">


- **오류 발생**

<img src="./error.png" width="300" height="300">
