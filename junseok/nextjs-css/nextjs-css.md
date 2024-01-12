## Next.js css 적용하기 

### 전역 스타일 적용하기 
Next.js page Router를 사용할 경우 reset css와 같이 전역에 스타일을 적용해야 하는 경우가 있습니다. 이러한 경우 _app.tsx에 css 모듈을 적용해야 합니다. 
```tsx
// _app.tsx

import '../styles.css'

export default function MyApp{
    //...
}
```

### CSS 적용하기 
CSS-in-js, tailwindcss 등의 스타일을 활용하지 않는다면, 일반 css를 적용할 수 있습니다. 일반 css를 활용할 경우 `[name].module.css`와 같은 명명 규칙을 준수하여, 컴포넌트 별로 css를 적용할 수 있습니다. 이러한 경우 각 css 파일을 모듈로 관리하게 되어 동일한 className을 사용한다 하더라도 문제가 없습니다. 


```tsx
'use client';
import SearchForm from '@/app/(login)/_component/SearchForm';
// css 불러오기
import style from './search.module.css';
import { usePathname } from 'next/navigation';

export default function Search() {
  const pathName = usePathname();
  const onChangeFollow = () => {};
  const onChangeAll = () => {};

  if (pathName === '/explore' || pathName === '/compose/tweet') return null;

    return (
      <div>
        {/*css 적용하기*/}
        <h5 className={style.filterTitle}>검색 필터</h5>
        <div className={style.filterSection}>
          <div>
            <label>사용자</label>
            <div className={style.radio}>
              <label htmlFor="user">모든 사용자</label>
              <input type="radio" id="user" name="pf" defaultChecked onChange={onChangeAll} />
            </div>
            <div className={style.radio}>
              <label htmlFor="follow">내가 팔로우하는 사람들</label>
              <input type="radio" id="follow" name="pf" value="on" onChange={onChangeFollow} />
            </div>
          </div>
        </div>
      </div>
    );
  }
```
이는 실제 프로덕션 단계에서 별도의 CSS 파일로 관리하게 됩니다. 만약 css class로 `BfilterTitle`을 적용했다면,
실제 페이지에서 이를 확인해보면 `BfilterTitle__ewYiV` 이와 같이 적용된 것을 확인할 수 있습니다. 이는 컴포넌트 별로 css 충돌을 피하기 위해 Next.js의 최적화가 동작한 겻임을 확인할 수 있습니다. 

### SCSS & SASS
css룰 적용하는 방식과 동일합니다. 다만 SCSS에서 제공하는 variable을 컴포넌트에서 사용하고 싶다면 export 문법을 사용할 수 있습니다.
```scss
// scss 
$primary: blue;

:export {
    primary: $primary
}
```

```tsx

import styles from "./Button.module.scss";

export default function Button(){
    return <span style={{color: styles.primaty}}></span>
}
```

### CSS in JS

page router를 활용하는 경우 다음과 같이 설정을 하면 됩니다. 해당 설정은 다양한 CSS-IN-JS 라이브러리 중에서 `styled-components`를 활용하여 진행합니다. 



```tsx
import Document, {
  Html,
  Head,
  Main,
  NextScript,
  DocumentContext,
  DocumentInitialProps,
} from "next/document";
import { ServerStyleSheet } from "styled-components";
export default function MyDocument() {
  return (
    <Html lang="en">
      <Head />
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  );
}

MyDocument.getInitialProps = async (
  ctx: DocumentContext,
): Promise<DocumentInitialProps> => {
  const sheet = new ServerStyleSheet();
  const originalRenderPage = ctx.renderPage;

  try {
    ctx.renderPage = () =>
      originalRenderPage({
        enhanceApp: (App) => (props) => sheet.collectStyles(<App {...props} />),
      });

    const initialProps = await Document.getInitialProps(ctx);
    return {
      ...initialProps,
      styles: (
        <>
          {initialProps.styles}
          {sheet.getStyleElement()}
        </>
      ),
    };
  } finally {
    sheet.seal();
  }
};

```

- ServerStyleSheet는 styled-components의 스타일을 서버에서 초기화해 사용되는 클래스입니다. 해당 클래스를 인스턴스로 초기화 하면 서버에서 styled-components가 작동하기 위한 다양한 기능을 가지고 있습니다. \

- originalRenderPage는 ctx.renderPage를 담아두고 있습니다. 기존의 ctx.renderPage가 하는 작업에 추가적으로 styled-components 관련 작업을 하기 위해 별도 변수로 분리하였습니다. 

- ctx.renderPage에는 기존에 해야 하는 작업과 함께, App을 렌더링할때 추가로 수행하고 싶은 작업을 진행합니다. 
    
    여기에서 `sheet.collectStyles(<App {...props} />)` 작업을 합니다. 이는 App 컴포넌트 위에 styled-components의 Context.API로 감싸는 구조입니다. 

- `const initialProps = await Document.getInitialProps(ctx);`는 기존의 _document.tsx가 렌더링을 수행할 깨 필요한 getInitialProps를 생성하는 작업을 합니다. 

- 마지막 반환 문구에서는 기존에 기본적으로 내려주는 props에 추가적으로 styled-components가 모아둔 자바스크립트 파일 내 스타일을 반환합니다. 이렇게 되면 서버 사이드 렌더링 시에 최초로 _document 렌더링 될때 styled-components에서 수집한 스타일도 함께 내려줄 수 있습니다. 

### CSS-in-JS App route

app router에서 css-in-js 방식을 사용하고 싶다면, [해당 Next.js 공식페이지](https://nextjs.org/docs/app/building-your-application/styling/css-in-js#configuring-css-in-js-in-app)에서 확인할 수 있습니다. 

```tsx
'use client';

import React, { useState } from 'react';
import { useServerInsertedHTML } from 'next/navigation';
import { StyleRegistry, createStyleRegistry } from 'styled-jsx';

export default function StyledJsxRegistry({ children }: { children: React.ReactNode }) {
  // Only create stylesheet once with lazy initial state
  // x-ref: https://reactjs.org/docs/hooks-reference.html#lazy-initial-state
  const [jsxStyleRegistry] = useState(() => createStyleRegistry());

  useServerInsertedHTML(() => {
    const styles = jsxStyleRegistry.styles();
    jsxStyleRegistry.flush();
    return <>{styles}</>;
  });

  return <StyleRegistry registry={jsxStyleRegistry}>{children}</StyleRegistry>;
}

```

하지만 해당 방법은 context API를 사용해야 하기 때문에 page route와 달리 client Component에서만 적용할 수 있습니다. 

### Context 객체

- [공식문서 링크](https://nextjs.org/docs/pages/api-reference/functions/get-initial-props)

getInitialProps는 context라는 하나의 인자를 받습니다. 이 인자는 다음과 같은 속성을 가진 객체입니다:

- pathname - 현재 라우트입니다. 즉, /pages에서 페이지의 경로입니다.
- query - URL의 쿼리 문자열 부분을 객체로 파싱한 값입니다.
- asPath - 브라우저에 표시되는 실제 경로(쿼리를 포함한) 문자열입니다.
- req - HTTP 요청 객체(서버 전용)
- res - HTTP 응답 객체(서버 전용)
- err - 렌더링 중 오류가 발생하면 오류 객체가 포함됩니다.
이러한 속성을 활용하여 필요한 데이터를 수집하고 처리할 수 있습니다.

getInitialProps는 자식 컴포넌트에서는 사용할 수 없으며, 각 페이지의 기본 내보내기에서만 사용할 수 있습니다.
getInitialProps 내에서 서버 측 모듈만 사용하는 경우 적절하게 가져와야 합니다. 그렇지 않으면 앱의 성능이 느려질 수 있습니다.
렌더링 유형과 관계없이 모든 props는 페이지 컴포넌트로 전달되며 초기 HTML에서 클라이언트 측에서 볼 수 있습니다. 이렇게 함으로써 페이지를 올바르게 물들일 수 있습니다. props에 클라이언트에서 사용할 수 없어야 할 중요한 정보가 없도록 주의하세요.