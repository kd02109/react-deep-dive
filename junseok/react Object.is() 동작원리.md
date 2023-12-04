## 자바스크립트의 원시 값, 참조 값
- 자바스크립트의 원시 값은 총 7가지가 있습니다.
```js
typeof true // "boolean"
typeof 12 // "number"
typeof "string" // "string"
typeof null // "object"
typeof BigInt("1") // "bigint
typeof Symbol('key') // symbol
typeof undefined // undefined 
```


- 이외의 값들은 모두 참조값입니다. 
```js
typeof [] // "objec"
typeof {} // "object"
function hello(){}
function hello2(){}
typeof hello // "funvction"

hello === hello2 // false
```
원시 값들은 **불변 형태**로 저장이 됩니다. 반면 참조 값들은 프로퍼티를 수정, 삭제, 추가 할 수 있기 때문에 변경 가능한 형태오 저장이 됩니다. 따라서 값이 아닌 참조를 전달합니다. 

## Object.is 
`Object.is` 는 엄격한 동등 기호(`===`)와 같이 동작하지만, 다른 부분이 있습니다.
```js
0 === -0 // true
Object.is(-0, 0) // false

Number.NaN === NaN // false
Object.is(Number.NaN, NaN) // true

NaN === 0/0 // false
Object.is(NaN, 0/0) // true

const a = {};
const b = {};
a === b //false
Object.is(a,b) // false
```

하지만 `Object.is()`를 사용한다고 하더라도 객체 비교에는 차이가 없습니다. 객체 비교는 메모리 참조와 동일합니다. 
```js
Object.is({}, {}) // false
Object.is([], []) //false
```

이러한 `Object.is()`는 리엑트에서 state 값의 변경을 확인하기 위한 검사를 위해 사용됩니다. 
state의 값이 변경되었다면 `false`를 변경되지 않았다면 `true`를 반환합니다. `Object.is`를 바탕으로 해서 리엑트는 객체의 얕은 비교를 실행합니다. 
```jsx
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
매번 obj의 경우 프로퍼티 내부의 값이 동등해도 계속 새롭게 객체를 생성하기 때문에, useEffect의 참조에서 새로운 값으로 이를 파악합니다.
```
App.tsx:11 change Number : 100
App.tsx:14 Change objNumber : 100
App.tsx:11 change Number : 101
App.tsx:14 Change objNumber : 101
App.tsx:14 Change objNumber : 101
App.tsx:14 Change objNumber : 101
App.tsx:14 Change objNumber : 101
```


`shallowEqual`은 객체의 첫 번째 깊이 까지 비교를 진행합니다. 이러한 `shallowEqual`은 `props` 값의 비교를 위해 사용됩니다.

```jsx
import { memo, useEffect, useState } from 'react';

const Component = memo((props: { counter: number }) => {
  useEffect(() => {
    console.log('Component has been Rendered');
  });
  return <h1>{props.counter}</h1>;
});

const DeeperComponent = memo((props: { counter: { counter: number } }) => {
  useEffect(() => {
    console.log('DeeperComponent has been Rendered');
  });
  return <h1>{props.counter.counter}</h1>;
});

function App() {
  const [, setNumber] = useState(100);

  const handleClick = () => setNumber(prev => prev + 1);

  return (
    <>
      <DeeperComponent counter={{ counter: 100 }} />
      <Component counter={100} />
      <button onClick={handleClick}>+</button>
      <div>App</div>
    </>
  );
}

export default App;
```
`DeeperComponent`는 프로퍼티 값으로 객체를 전달합니다. 반면에 `Component`에는 객체의 프로퍼티 값으로 원시 값을 전달합니다. 첫 번째 깊이의 갚까지 얕은 비교를 하기 때문에, 버튼을 클릭할때, `Component`는 재 렌더링이 발생하지 않습니다. Prop으로 전달된 객체에 변경사항이 없는 것을 확인했기 때문입니다. 하지만 재귀적으로 깊이 비교를 해야 하는 `DeeperComponent`의 경우 `memo`가 정상적으로 동작하지 않습니다. 렌더링 마다 새로운 객체를 프로퍼티에 할당하기 때문에, 전달된 Prop이 참조하는 메모가 변경되기 때문입니다. 따라서 버튼을 클릭할 때 마다 재 렌더링이 발생하게 됩니다. 
```
DeeperComponent has been Rendered
Component has been Rendered
DeeperComponent has been Rendered
DeeperComponent has been Rendered
DeeperComponent has been Rendered
```

리엑트에서 첫번째 깊이의 값들까지 비교를 실행하는 이유는 리엑트의 성능과 연관이 있습니다. 객체 내부에 객체의 존재는 재귀적으로 탐색하기 전까지는 몇 단계 깊이인지 알 수 없기 때문에, 성능에 악영향을 주게 됩니다.