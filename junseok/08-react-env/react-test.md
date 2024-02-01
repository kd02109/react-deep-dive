## React Testing Library
Dom Testing Library를 기반으로 만들어진 테스팅 라이브러리입니다. 리액트를 기반으로 한 테스트를 수행하기 위해 만들어졌습니다. 따라서 Dom Testing Library에 대한 이해가 있어야 합니다. 

Dom Testing Library는 jsdom을 기반으로 하고 있습니다. jsdom은 순수하게 자바스크립트로 작성된 라이브러리로, nodejs 환경에서 HTML과 DOM을 사용할 수 있도록 해주는 라이브러리입니다. 아래와 같이 nodejs 환경에서 마치 HTML이 있는 것 처럼 DOM 요소에 접근해서 값을 불러오거나, 수정할 수 있습니다.

```js
const jsdom = require("jsdom");
const { JSDOM } = jsdom;

const dom = new JSDOM(`<!DOCTYPE html><p>Hello world</p>`);
console.log(dom.window.document.querySelector("p").textContent); // "Hello world"
```

jsDOM을 사용해서 nodejs 환경에서 HTML을 사용할 수 있는 DOM Testing Library를 기반으로, 동일한 원리로 리엑트 기반 환경에서 리엑트 컴포넌트를 테스팅할 수 있는 라이브러리가 React Testing Library라이브러리입니다. 이를 통해 실제 리액트 컴포넌트를 렌더링 하지 않고도 리액트 컴ㅁ포넌트가 원하는 대로 렌더링되고 있는지 확인할 수 있습니다. 

또한 컴포넌트 뿐만 아니라 Provider, Hook 등 리액트를 구성하는 다양한 요소들을 테스트 할 수 있습니다. 

## 자바스크립트 테스트의 기초
기본적인 테스트 코드의 작성 방식은 다음과 같은 과정을 따릅니다.
1. 테스트할 함수나 모듈을 선정합니다.
2. 함수나 모듈이 반환하길 기대하는 값을 적습니다.
3. 함수나 모듈의 실제 반환 값을 적습니다. 
4. 3번의 기대에 따라 2번의 결과가 일치하는지 확인합니다. 
5. 기대하는 결과를 반환하면, 테스트는 성공이며, 만약 기대와 다른 결과를 반환하면 에러를 던집니다. 

이러한 예상 결과를 확인하는 방법으로 nodejs는 assert라는 모듈을 기본적으로 제공합니다. 

```js
const assert = require('assert');
const sum = (a,b) => a+b;
assert.equal((1,2),3);
assert.equal((1,1),2);
assert.equal((1,2),2); // AssertionError [ERR_ASSERTION] [ERR_ASSERTION] : 3 == 2
```

이처럼 테스트 결과를 확인할 수 있도록 도와주는 라이브러리를 어선셜(assertion) 라이브러리라고 합니다. 이러한 어설션 라이브러리에는 Node.js가 제공하는 assert 외에도 should.js, expect.js, chai 등 다양합니다. 또한 동등 비교 `equal`이 외에 객체 동일을 비교하는 `deepEqual`, 에러 반환 여부를 확인하는 `throws`등 다양한 메서드를 제공하고 있습니다. 

올바른 테스트 코드는 가능한 사람이 읽기 쉽게, 테스트의 목적이 분명하게 작성되는 것이 중요합니다. 즉 무엇을 테스트 했고, 무슨 테스트를 어떻게 수행했는지 등의 모든 테스트 정보를 일목요연 하게 보여주어야 합니다. 이러한 테스트의 기승전결을 완성해 주는 것이 테스팅 프레임워크 입니다.
`Jest`, `Mocha`, `Karma`, `Jasmine`등이 있습니다. 특히 리액트 진영에서는 페이스북에서 개발한 오픈소스 라이브러리인 Jest가 널리 쓰이고 있습니다. Jest의 경우 자체 제작한 expect 패키지를 사용해 어설션을 수행합니다.

Jest등의 테스트 라이브러리를 전역에서 사용할 수 있는 이유는 Jest CLI에 있습니다. 테스팅 프레임워크에서는 global이라 해서 실행 시에 전역 스코프에 기본적으로 넣어주는 값들이 있습니다. 그리고 Jest는 해당 값을 실제 테스트 직전에 미리 전역 스코프에 넣어줍니다. 이를 통해 간결하고 빠른 테스트 코드 작성에 도움을 줍니다. 

## 리엑트 컴포넌트 테스트 코드 작성하기
기본적으로 리액트에서 컴포넌트 테스트는 다음과 같은 순서로 진행됩니다. 
1. 컴포넌트를 렌더링한다.
2. 필요하다면 컴포넌트에서 특정 액션을 수행한다.
3. 컴포넌트 렌더링과 2번의 액션을 통해 기대하는 결과와 실제 결과를 비교한다.

현재 리엑트 기반 프레임워크를 사용하지 않고, 순수 리엑트 라이브러리로 코드를 작성할 경우 쉽게 번틀러 setting을 기반으로 한 시작 명령어를 제공합니다.
1. vite
2. CRA(Create-React-App)


### vitest
이때 vite 번들러 환경에서 리엑트를 시작하는 경우, Jest 보다는 vitest를 사용하는 것을 권장합니다. 또한 vite 번들러 환경에서 리엑트를 사용할 경우 별도로 react-testing-library를 설치해야 합니다. vitest의 특징을 간단하게 정리했습니다.
- vite 기반에서 쉽게 테스트를 실행할 수 있는 테스팅 프레임워크입니다.
- Jest 호환 API 및 가이드를 제공합니다.
- 추가 설정 없이 ESM, Typescript, Jsx를 사용할 수 있습니다. 


### CRA
npx create-react-app 명령어를 통해 리엑트 프로젝트를 생성할 경우 내부에 react-testing-library가 포함돼 있으므로 별도로 설치할 필요가 없습니다.
이때 기초적인 App.test.tsx가 설치되어 있습니다. 
```tsx
import React from 'react';
import { render, screen } from '@testing-library/react';
import App from './App';

test('renders learn react link', () => {
  render(<App />);
  const linkElement = screen.getByText(/learn react/i);
  expect(linkElement).toBeInTheDocument();
});
```

1. <App/> 을 렌더링합니다.
2. 렌더링 하는 컴포넌트 내부에서 "learn react"라는 문자열을 가진 DOM 요소를 찾습니다.
3. `expect(linkElement).toBeInTheDocument();` 어설션을 활용해 2번에서 찾은 요소가 docuement 내부에 있는지 확인합니다. 

이렇게 HTML 요소가 있는 여부를 확인하는 방법은 크게 3가지가 있습니다. [참고](https://testing-library.com/docs/queries/about)

- `getBy...` : 인수의 조건에 맞는 요소를 반환하며, **해당 요소가 없거나 두개 이상이면 에러를 발생**시킵니다. 복수 개를 찾고 싶다면 `getAllBy...`를 사용합니다. 
- `findBy...` : `getBy...`와 거의 유사하나 한 가지 큰 차이점은 **Promise를 반환**합니다. 즉, 비동기로 찾는다는 것을 의미합니다. 기본값으로 1000ms의 타임아웃을 가지고 있습니다. 마찬가지로 두 개 이상이면 에러를 발생시킵니다. 마찬가지로 복수개를 찾고 싶다면, `findAllBy...`를 사용합니다. 이러한 특징으로 **`findBy`는 비동기 액션 이후에 요소를 찾을 때 사용**합니다.
- `queryBy...` : 인수의 조건에 맞는 요소를 반환하는 대신, 찾지 못한다면 null을 반환합니다. `findBy...`, `getBy...`는 찾지 못하면 에러를 발생시키기 때문에 찾지 못해도 에러를 발생시키지 않고 싶다면 `queryBy...`를 사용합니다. 이또한 복수개에서는 에러를 발생시킵니다. 따라서 복수개를 찾을 경우 `queryAllBy...`를 사용합니다. 

추가적으로 컴포넌트 테스트 파일은 같은 디렉터리 상에 위치하는 것이 일반적입니다.


### 정적 컴포넌트 테스트

별도의 상태가 존재하지 않는 항상 같은 결과를 반환하는 컴포넌트 요소를 테스트하는 방법입니다. 이는 크게 어렵지 않습니다. 테스트를 원하는 컴포넌트를 렌더링 한 다음, 테스트를 원하는 요소를 찾아 수행하면 됩니다. 

간단한 Input Component를 구성하고 정적 컴포넌트 테스트를 진행해보았습니다. 
```tsx
// Input.tsx
import { ChangeEvent, useState, KeyboardEvent } from 'react';
import cn from 'classnames';
import './input.css';

/**
 * placeholder 설정 - 테스트 작성 완료
 * className에 따른 css class 설정 - 테스트 작성 완료
 * 텍스트를 입력할 때마다 onChange 핸들러 호출
 * focus 시 border 스타일 변경
 * focus 시 onFocus 핸들러 호출
 * Enter 키 입력 시 onEnter 핸들러 호출
 */

interface Prop {
    placeholder? :string;
    className? : string;
    onFocus? : ()=>void;
    onChange? : (value:string)=>void;
    onEnter? : (value:string)=>void;
}

export default function TextField({
  placeholder,
  className,
  onFocus,
  onChange,
  onEnter,
} : Prop) {
  const [value, setValue] = useState('');
  const [focused, setFocused] = useState(false);

  const changeValue = (ev : ChangeEvent<HTMLInputElement>) => {
    setValue(ev.target.value);
    onChange?.(ev.target.value);
  };
  const focus = () => {
    setFocused(true);
    onFocus?.();
  };
  const blur = () => {
    setFocused(false);
  };
  const pressEnter = (ev:KeyboardEvent<HTMLInputElement>) => {
    if (ev.key === 'Enter' && !ev.nativeEvent.isComposing) {
      ev.preventDefault();
      onEnter?.(value);
    }
  };

  return (
    <input
      type="text"
      className={cn('text-input', className)}
      onChange={changeValue}
      onFocus={focus}
      onBlur={blur}
      onKeyDown={pressEnter}
      placeholder={placeholder || '텍스트를 입력해 주세요.'}
      value={value}
      style={
        focused ? { borderWidth: 2, borderColor: 'rgb(25, 118, 210)' } : {}
      }
    />
  );
}

```

```tsx
// Input.test.tsx
import { render, screen } from "@testing-library/react"
import Input from "./Input"

beforeEach(()=>{
    jest.clearAllMocks()
});

describe('placeholder', () => {
    it('기본 placeholder "텍스트를 입력해 주세요."가 노출된다.', async () => {
        render(<Input />);  
  
      const textInput = screen.getByPlaceholderText('텍스트를 입력해 주세요.');
  
      screen.debug();
  
      expect(textInput).toBeInTheDocument();
    });
  
    it('placeholder가 prop에 따라 변경된다.', async () => {
      render(<Input placeholder={'상품명을 입력해주세요.'} />);
  
      const textInput = screen.getByPlaceholderText('상품명을 입력해주세요.');
  
      screen.debug();
  
      expect(textInput).toBeInTheDocument();
    });
  });

```

### React Testing Library dataset

React Testing Library는 HTML dataSet에 접근해서 요소 값을 가지고 올 수 있습니다. 이는 assertion `getByTestId`, `findByTestId` 등으로 선택할 수 있습니다. 웹에서 사용하는 `querySelector([data-...] = "..")` 와 동일한 역할을 합니다.

