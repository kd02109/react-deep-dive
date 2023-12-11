## TypeScript 

### in 
`property in object`로 주로 사용합니다. 어떤 객체에 키가 존재하는지 확인하는 용도로 사용됩니다. 
하나의 객체에 서로 다른 타입을 가질 수 있습니다. 이때 `in`을 사용해서 `object`의 타입을 좁히고 정확한 프로퍼티에 접근할 수 있습니다.

혹은 각 객체의 타입을 확인하는 프로퍼티를 통해서도 확인이 가능합니다. 저는 대화 박스를 의미하는 object와 날짜의 변동을 의미하는 타입을 각 객체에 `type` 프로퍼티를 만들어서 관리했습니다. 

```ts
export type Message = {
  type: 'message';
  user: string;
  time: string;
  profile: string;
  message?: string;
  messageFn?: (name: string) => string;
  picture?: string;
};
export type Date = {
  type: 'date';
  date: string;
  profile: string;
};
export type MessageOrDate = Message | Date;

if (item.type === 'message') {
    const time = changeKrTime(item.time);
    if (item.message && item.user === 'me') {
      return <Chat message={item.message} time={time} />;
    }
    if (item.message && item.user !== 'me') {
      return (
        <SpeechBubble
          profile={profile}
          name={item.user}
          message={item.message}
          time={time}
        />
      );
    }
}

// in 사용하기 
    if ('message' in item && item.user === 'me') {
      return <Chat message={item.message} time={time} />;
    }
```
### 인덱스 시그니처
```ts
type stringObj = { [key:string] : string}
```
인덱스 시그니처는 객체의 키를 정의하는 방식입니다. 하지만 인덱스 시그니처의 경우 키의 범위가 너무 넓습니다. 따라서 객체의 타입도 필요에 따라 좁혀야 합니다. 객체의 키를 좁히는 방식은 두가지가 있습니다. 

```ts
// Utility_type Record 사용하기 
type Key = "name" | "id" | "age" | "profile"

type User = Recore< Key, string>

// 타입을 사용한 인덱스 시그니처 
type User = {[key in Key] : string }

// object에서 추출한 키에 정확한 타입 할당하기
const getKeys = <T extends Object>(obj: T): Array<keyof T> =>
  Object.keys(obj) as Array<keyof T>;

// value 타입을 정확하게 반환하기 
export type ValueOf<T> = T[keyof T];
```