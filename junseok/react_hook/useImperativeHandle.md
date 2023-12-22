## useImperativeHandle

- **상위 컴포넌트에게 노출하고 싶은 메서드만을 선택**할 수 있습니다. 따라서 forwardRef와 함께 사용할 때 의미가 있습니다. 
따라서 전체 ref를 노출하지 않고, input에 연결된 Ref 중 focus와 scroll 기능만을 노출하고 싶을 수 있습니다. 이를 위해서 실제 브라우저 DOM을 별도의 ref에 유지해야 합니다. 그리고 이를 **useImperativeHandle을 사용해서 부모 컴포넌트에서 호출할 메서드만 있는 핸들**을 노출합니다.

- `useImperativeHandle`을 사용함으로서 더이상 forwardRef에서 전달된 input에 대한 ref는 전달되지 않습니다. `useImperativeHandle`의 두번째 인자인 callback의 반환 값들이 새로운 객체로서 전달되고 해당 메서드에만 접근할 수 있기 떄문입니다. 

- 아래의 예시 코드는  전체 `<input>` DOM 노드를 노출하지 않고` focus`와` scrollIntoView`의 두 메서드만을 노출 하는 경우입니다. **이를 위해서는 실제 브라우저의 DOM을 별도의 ref에 유지해야 합니다. useImperativeHandle을 사용하여 부모 컴포넌트에서 호출할 메서드만 있는 핸들을 노출합니다.**
```tsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

// 노출하고자 하는 메서드들의 타입을 지정합니다. 
interface HandleInput {
  focus: () => void;
  scrollIntoView: () => void;
}

const MyInput = forwardRef(function MyInput(_, ref) {
  const inputRef = useRef<HTMLInputElement>(null);

    // forwardRef를 통해 전달받은 상위 ref에서 접근할 수 있는 메서드를 새롭게 지정합니다. 이제는 부모 ref에서 접근할 수 있는 값은 focus와 scrollIntoView로 고정이 됩니다.
  useImperativeHandle(
    ref,
    () => {
      return {
        focus() {
          inputRef.current?.focus();
        },
        scrollIntoView() {
          inputRef.current?.scrollIntoView();
        },
      };
    },
    [],
  );

  return <input ref={inputRef} />;
});

function App() {
  const ref = useRef<HandleInput>(null);

  function handleClick() {
    ref.current?.focus();
  }

  function handleScroll() {
    ref.current?.scrollIntoView();
  }

  return (
    <>
      <button type="button" onClick={handleScroll}>
        ScrollIntoInput
      </button>
      <form
        style={{
          width: '1000px',
          height: '3000px',
          display: 'flex',
          flexDirection: 'column',
          justifyContent: 'end',
        }}>
        <MyInput ref={ref} />
        <button type="button" onClick={handleClick}>
          Edit
        </button>
      </form>
    </>
  );
}
```


- 해당 방식은 객체 지향적 방식과 유사합니다. Elment에 연결된 모든 Ref 값을 활용하는 것이 아닌, 사용하고자 하는 메서드만을 노출해서 이를 사용하도록 하고 있습니다. 