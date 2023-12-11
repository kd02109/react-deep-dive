## JSX

- JSX는 리엑트가 등장하면서 페이스북에서 소개한 새로운 구문이지만 리엑트에 종속되지는 않습니다. JSX는 XML과 유사한 내장형 구문으로 독자적인 문법 체계를 이루고 있습니다. 
- 따라서 이러한 JSX는 ESCMAScript의 표준이 아닙니다. 따라서 v8, Deno와 같은 자바스크립트 엔진이나 크롬, 웨일, 파이어폭스 같은 브라우저에 의해서 실행되지 않습니다. 
- 따라서 **jsx는 반드시 바벨과 같은 트랜스파일러를 통해 자바스크립트 코드로 변환하는 과정**을 거쳐야 합니다. JSX의 목적은 HTML, XML을 자바스크립트 내부에 표현하는 것에 국한되지 않습니다. 가장 중요한 기능은 자바스크립트에서 작성한 JSX 문법을 트리 구조로 토큰화 해서 ESCMAScript로 변환하는 것이 가장 중요한 목적입니다. 즉 JSX는 자바스크립트 내부에서 표현하기 까다로원ㅆ던 XML 스타일의 트리 구문을 작성하는 것에 도움을 주는 문법입니다. 

## JSX 구성요소 
JSX는 **JSXElement, JSXAttributes, JSXChildren, JSCStrings** 4가지 컴포넌트를 기반으로 구성됩니다. 

### JSXElement
가장 기본적인 요소로 HTML의 element와 유사한 역할을 합니다.
- JSXElement Naming 규칙
    - JSXIdentifier: JSX 내부에서 사용할 수 있는 식별자로 자바스크립트 식별자 규칙과 동일합니다.
    - JSXNamespaceName: `:` 통해 서로 다른 식별자를 연결할 수 있습니다. 이떄 묶을 수 있는 것은 한 개뿐입니다.
    ```jsx
    function vaild(){
        return <foo:bar></foo:bar>
    }
    ```
    - JSXMemberExpression : `JSXIdentifier.JSXIdentifier`의 조합입니다. 해당 표현은 많은 곳에서 사용이 됩니다. styledComponent, framer-motion 등의 라이브러리에서 사용을 하고 있습니다. 해당 표현식을 통해 객체에 다양한 JSX를 미리 저장하고 불러와서 사용할 수도 있습니다. 
    ```jsx
    import {motion} from "framer-motion"
    funtion Motion(){
        return <motion.div>가능</motion.div>
    }
    ```
### JSXAttributes
JSXElement에 부여할 수 있는 속성을 의미합니다. HTML의 attribute와 유사한 역할을 합니다. 따라서 필수적으로 작성해야 하는 요소는 아닙니다. 
    - JSXSpreadAttributes 자바스크립트의 object spread뿐만 아니라 AssignmentExpression으로 취급되는 모든 표현식이 가능합니다. 조건문 표현식, 화살표 함수, 할당식 등 다양한 것이 포함됩니다. 
    - JSXAttributeName: 속성의 키 값입니다. JSXIdentifier와 JSXNamespaceName 방식이 가능합니다.
    - JSXAttributeValue: 속성의 키에 할당된 값입니다. 큰 따움표로 구성된 문자, 작은따옴표호 구성된 문자, {AssignmentExpression}, JSXElement 이 값으로 할당할 수 있습니다. 

### JSXChildren
JSXElement의 자식 값을 나타냅니다. JSX는 트리 구조를 구현하기 위한 문법으로 JSXElement 간의 부모 자식 관계를 표현할 수 있습니다. 이때 자식을 JSXChildren이라고 합니다.
    - JSXChild JSXChildren을 이루는 기본 단위 입니다. 단어의 차이에서 알 수 있듯이 JSXChildren은 JSXChild를 0개 이상 가질 수 있습니다. 이때 자식으로 가질 수 있는 값은 다음과 같습니다.

    ```
    JSXText : "{", "}", "<",">" 를 제외한 문자열
    JSXElement
    JSXFragment
    {JSXChildExpression (optional)} : 이는 자바스크립트의 AssignmentExpression을 의미합니다. 즉 화살표함수, 조건문 표현식, 할당식 등을 활용할 수 있습니다.
    ```

### JSXSTring
JSXAttributeValue와 JSXText는 HTML과 JSX 사이에 복사와 붙여넣기를 쉽게 할 수 있도록 설계되어 있습니다. HTML에서 가능한 문자열은 모두 사용가능합니다. 따라서 큰따옴표로 구성된 문자열, 작은 따옴표로 구성된 문자열, JSXText가 JSXString에서 사용가능합니다. 

여기서 자바스크립트와 HTML의 중요한 차이점이 발생합니다. 탈출문자 `\`는 자바스크립트에서 특수 문자를 처리하기 위해 사용되기 때문에 별도의 처리를 해야 합니다.
```
"\\" 
```

### JSX의 변환 방법 
리엑트에서 JSX를 변환하기 위해서 @babel/plugin-transform-react-jsx 플러그인을 사용합니다. [참고](https://ko.legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)

바벨은 JSX를 react가 트리 구성을 하기 위한 객체로 변경합니다. 이러한 특성 때문에 반환되는 JSX는 모두 단일 부모 JSX로 감싸주어야 합니다. 그렇지 않을 경우 두 개의 객체가 반환되는 형식으로 변환되기 떄문입니다.
```jsx
// 원본 jsx 코드
import { useState, useEffect } from 'react';

function App() {
  const [number, setNumber] = useState(100);
  const [objNumber, setObjNumber] = useState({ counter: 100 });

  const handleClick = () => setNumber(101);
  const handleObjClick = () => setObjNumber({ counter: 101 });

  useEffect(() => {
    console.log(`change Number : ${number}`);
  }, [number]);
  useEffect(() => {
    console.log(`Change objNumber : ${objNumber.counter}`);
  }, [objNumber]);
  return (
    <>
      <div>objNumber : {objNumber.counter}</div>
      <div>number : {number}</div>
      <button onClick={handleClick}>원시값 숫자 증가하기</button>
      <button onClick={handleObjClick}>참조값 숫자 증가하기</button>
      <div>App</div>
    </>
  );
}

export default App;
```

```jsx
// @babel/plugin-transform-react-jsx 로 반환된 코드 
// 바벨 v 7.23.5
import { useState, useEffect } from 'react';
import { jsxs as _jsxs } from "react/jsx-runtime";
import { jsx as _jsx } from "react/jsx-runtime";
import { Fragment as _Fragment } from "react/jsx-runtime";

function App() {
  const [number, setNumber] = useState(100);
  const [objNumber, setObjNumber] = useState({
    counter: 100,
  });
  const handleClick = () => setNumber(101);

  const handleObjClick = () =>
    setObjNumber({
      counter: 101,
    });

  useEffect(() => {
    console.log(`change Number : ${number}`);
  }, [number]);
  useEffect(() => {
    console.log(`Change objNumber : ${objNumber.counter}`);
  }, [objNumber]);

  return _jsxs(_Fragment, {
    children: [
      _jsxs('div', {
        children: ['objNumber : ', objNumber.counter],
      }),
      _jsxs('div', {
        children: ['number : ', number],
      }),
      _jsx('button', {
        onClick: handleClick,
        children: '\uC6D0\uC2DC\uAC12 \uC22B\uC790 \uC99D\uAC00\uD558\uAE30',
      }),
      jsx('button', {
        onClick: handleObjClick,
        children: '\uCC38\uC870\uAC12 \uC22B\uC790 \uC99D\uAC00\uD558\uAE30',
      }),
      _jsx('div', {
        children: 'App',
      }),
    ],
  });
}
export default App;

```

## 리엑트에서 사용되지 않는  jsx 문법
- JSXNamespacedName
- JSXMemberExpression