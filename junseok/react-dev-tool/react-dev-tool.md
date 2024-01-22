## react-dev-tool

### 설치하기 
[구글 extension](https://chromewebstore.google.com/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi), [엣지](https://microsoftedge.microsoft.com/addons/detail/react-developer-tools/gpphkfbcpidddadnkolkpfckpihlkkil), [파이어폭스](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/)

해당 react-dev-tool이 회색이라면, 리엑트 개발 도구가 정상적으로 접근할 수 없는 페이지거나 리엑트로 개발되지 않은 페이지입니다. 

리엑트 로고에 색이 표시된다면, react-dev-tool이 해당 웹 페이지(실제 프로덕션)에 정상적으로 접근할 수 있다는 의미입니다.

react-dev-tool에 빨간 불이 들어온다면, 현재 웹페이지가 개발자 모드로 실행되고 있다는 의미입니다.


## Component
Component 탭에서는 현제 리액트 애플리케이션의 컴포넌트 트리를 확인할 수 있습니다. 또한 props와 내부 hooks 등 다양한 정보를 확인할 수 있습니다.

Componentd의 왼쪽 영역은 해당 리엑트 페이지의 컴포넌트 트리를 의미합니다. 기명 함수로 선언되어 컴포넌트 명을 알 수 있다면 해당 컴포넌트의 이름을 보여줍니다. 익명함수라면, Anonymous라는 이름으로 컴포넌트를 보여줍니다. 
- 익명 함수를 default로 export 한 경우 함수의 명칭을 추론할 수 없습니다. 따라서 _default, 혹은 Anoymous로 선언됩니다. 
- memo로 감싼 컴포넌트 또한 함수명을 추론하지 못합니다. Anoymous로 추롷합니다.
- 고차 컴포넌트의 두 함수 모두 Anoymous로 선언됩니다. 

컴포넌트의 함수 명을 명확히 하여 추적을 원한다면 다음과 같이 `displayName`을 설정합니다. 이는 디버깅에 많은 도움을 줍니다. 하지만 실제 프로덕션에서는 빌드 과정에서 각 컴포넌트 이름을 난독화를 진행하고 웹 동작에 필요없는 코드를 제거합니다. 따라서 개발 중에서만 유용한 방법입니다.
```tsx
const LinkBtn = () => <button></button>;
LinkBtn.displayName = "LinkBtn";
```

### 리엑트 개발자 도구 Component탭에서 확인할 수 있는 훅 목록 
- State: `useState`
- Reducer: `useReducer`
- Context: `useContext`
- Memo: `useMemo`
- Callback: `useCallback`
- Ref : `useRef`
- id : `useId`
- LayoutEffect: `useLayoutEffect`
- Effect : `useEffect` effect의 콜백함수에 기명 함수를 전달한다면, 해당 기명 함수를 명확하게 알 수 있습니다.
- 사용자 정의 훅: use를 제외한 나머지 이름 

## Profiler

리엑트가 렌더링하는 과정에서 발생하는 상황을 확인하기 위한 도구입니다. 리액트 애플리케이션이 렌더링되는 과정에서 어떤 컴포넌트가 렌더링됐는지, 몇 차례나 렌더링이 일어났으며, 어떤 작업에서 오래 걸렸는지 등 컴포넌트 렌더린 과정에서 발생하는 일을 확인할 수 있습니다. 

### 유용한 옵션 사용하기 
- General Highlight updates when components render : 컴포넌트가 헨더링 될 때마다 해당 컴포넌트에 하이라이트를 표시합니다.
- Debugging Hide logs during second render in Strict Mode : react Strict Mode에서 useEffect가 두 번 실행 되어 useEffect 콜백 내부의 console.log가 두번 씩 찍히는 것을 막습니다. 프로덕션 모드에서는 문제가 없습니다.
Profiler Record why each component rendered while profiling : 프로파일링 도중 무엇 때문에 컴포넌트가 렌더링 됐는지 기록합니다. 

### 활용 방법 
- Start Profiling : 좌측 상단에 위치해 있습니다. 프로파일링 시작이 되면 적색 버튼으로 변경이 됩니다. 이후 다시 적색 버튼을 클릭하면, 프로파일링이 중단되고 프로파일링 결과가 나타납니다. 
- Reload and Start Profiling : 해당 버튼을 누르면 웹페이지가 새로고침되면서 동시에 프로파일링이 시작됩니다. 이후 Start Profiling 버튼이 적색으로 변경됩니다. 프로파일링 중단은 위와 같습니다. 
- Clear Profiling Data : 프로파일링된 기록을 모두 삭제합니다. 

### Flamegraph
렌더 커밋별로 어떠한 작업이 일었났는지 알 수 있습니다. 너비가 넓을수록 해당 컴포넌트를 렌더링하는 데 오래 결렸다는 것을 의미합니다. 

렌더링 되지 않는 컴포넌트는 회색으로 표시되며, 'Did not render' 라는 메시지가 표시되는 것을 확인할 수 있습니다. 이를 활용하면, 개발자가 의도한 대로 메모제이션이 작동하고 있는지, 혹은 특정 상태 변화에 따라서 렌더링이 의도한 대로 제한적으로 발생하는지 확인 할 수 있습니다. 

오른쪽 상단의 화살표를 누르거나, 세로 막대 그래프를 클릭하면, 각 렌더 커밋별로 리액트 트리에서 발생한 렌더링 정보를 확인할 수 있습니다. 여기서는 렌더링 발생 횟수도 확인할 수 있어, 개발자가 의도한 횟수만큼 렌더링이 발생했는지도 알 수 있습니다.

### Ranked
해당 커밋에서 헨더링하는 데 오랜 시간이 걸린 컴포넌트를 순서대로 나열한 그래프입니다. 모든 컴포넌트를 보여주지 않고 렌더링이 발생한 컴포넌트만 보여줍니다. 

### Timeline
시간이 지남에 따라 컴포넌트에서 어떤 일이 일어났는지를 확인할 수 있습니다. Timeline은 리액트 18 이상의 환경에서만 확인할 수 있습니다. 

Timeline은 시간의 흐름에 따라 리엑트가 작동하는 내용을 추적하는 것에 유용합니다. 