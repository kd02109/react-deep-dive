# Next.js의 page/app router의 주요 특징 비교하기

# 라우트

## app router

폴더명이 라우팅에 사용되며, 파일명은 page로 작성돼야 한다는 특징이 있다.

### /folder

- 폴더명이 route segment가 됨
- 중첩 라우팅의 경우 /folder/folder 형식으로 폴더를 중첩하여 생성
- ex. /posts/1 → https://localhost:3000/posts/1
- 해당 폴더에 `page`라는 이름의 파일명으로 페이지를 만들어야 함

### [folder]

- 어떠한 문자도 올 수 있다는 뜻으로 동적 라우팅 표현
- ex. /posts/[slug] → https://localhost:3000/posts/abcdefghijfis
- 해당 폴더에 `page`라는 이름의 파일명으로 페이지를 만들어야 함

## page router

폴더명과 파일명이 라우팅에 사용된다.

### /pages/index.js

- 웹사이트의 루트 페이지

### /pages/hello.js

- 파일명이 route segment가 됨
- [https://localhost:3000/hello](https://localhost:3000/hello) 페이지

### /pages/hello/[greeting].tsx

- 어떠한 문자도 올 수 있다는 뜻으로 동적 라우팅 표현

# 에러 페이지

## app router

### error.js

- route segment에 대한 error UI boundary를 정의하는 파일
- 컴포넌트에서 발생하는 예상치 못한 오류를 포착하고 대체 UI를 표시하기 위해 구현

### not-found.js

- 찾을 수 없는 파일(404 에러)일 때 렌더링할 UI를 정의하는 파일

## page router

### _error.js

- Next.js 프로젝트 전역에서 발생하는 에러를 적절하게 처리하고자 할 경우 구현
- 프로덕션 모드에서 확인 가능

### 404.js

- 404 페이지를 정의하는 파일

### 500.js

- 서버에서 발생하는 에러를 핸들링하는 페이지
- `_error.js`와 `500.js`가 모두 구현됐다면, `500.js`가 우선적으로 실행됨

# 데이터 페칭

## app router

### SSG 구현하기 - generateStaticParams

- 정적으로 결정된 페이지를 보여주고자 할 때 사용되는 함수
- 접근 가능한 주소를 정의하는 함수로, 리턴 값이 페이지 컴포넌트에게 전달됨
- ex
    
    ```jsx
    export async function generateStaticParams() {
      const posts = await fetch('https://.../posts').then((res) => res.json())
     
      return posts.map((post) => ({
        slug: post.slug, // 접근 가능한 post 하위 페이지 결정
      }))
    }
     
    export default async function Page({ params }) {
      const { slug } = params
    	const post = await fetchPost(slug); // 페칭에 활용
    }
    ```
    

### SSR 구현하기 - async 페이지와 fetch

- 서버 컴포넌트에 async를 붙이고, (확장된) fetch를 사용하여 SSR을 구현할 수 있음
- ex
    
    ```jsx
    export default async function Page() {
      // This request should be cached until manually invalidated.
      // Similar to `getStaticProps`.
      // `force-cache` is the default and can be omitted.
      const staticData = await fetch(`https://...`, { cache: 'force-cache' })
     
      // This request should be refetched on every request.
      // Similar to `getServerSideProps`.
      const dynamicData = await fetch(`https://...`, { cache: 'no-store' })
     
      // This request should be cached with a lifetime of 10 seconds.
      // Similar to `getStaticProps` with the `revalidate` option.
      const revalidatedData = await fetch(`https://...`, {
        next: { revalidate: 10 },
      })
     
      return <div>...</div>
    }
    ```
    

## page router

### SSG 구현하기 - getStaticPaths & getStaticProps

- 정적으로 결정된 페이지를 보여주고자 할 때 사용되는 함수
- 두 함수는 함께 구현돼야 함
- `getStaticPaths`: 동적 라우팅 시 접근 가능한 주소를 정의하는 함수이고
- `getStaticProps`: 접근 가능한 주소(`getStaticPaths`의 리턴값)를 바탕으로 해당 페이지에 요청이 왔을 때 제공할 props를 반환하는 함수
- ex
    
    ```jsx
    // post/[slug].js
    export const getStaticPaths = async () => {
    	return {
    		paths: [{ params: { id: '1'} }, { params: { id: '2'} }],
    		fallback:false,
    } // post/1, post2에 접근 가능
    
    export const getStaticProps = async ({ params }) => { // getStaticPaths에 정의한 값을 받아서
    	const { id } = params;
    
    	const post = await fetchPost(id); // 페칭에 활용한 다음
    
    	return {
    		props: { post }, // 페칭 결과를 해당 페이지의 props로 전달
    	}
    }
    
    ㅊ
    ```
    

### SSR 구현하기 - getServerSideProps

- 서버에서 실행되는 함수로, 페이지 진입 전에 실행되는 함수
    - 서버에서 실행되기 때문에 클라이언트에서만 실행 가능한 코드는 별도로 처리해야 함
- 응답값에 따라 props를 반환하거나 다른 페이지로 리다이렉트 가능
- 이 함수를 구현했다면 꼭 서버에서 실행해야 하는 페이지로 분류됨
- props의 결과를 HTML에 정적으로 작성해서 내려주기 때문에, props는 JSON.stringify로 직렬화할 수 있는 값만 제공해야 함
- 이 함수의 실행이 끝나기 전까지는 사용자에게 HTML을 제공할 수 없기 때문에 반드시 최초에 보여줘야 하는 데이터가 아니라면 클라이언트에서 호출해 사용하는 것이 효율적
- ex
    
    ```jsx
    export const getServerSideProps = async (context) => {
    	const { query: { id = '' } } = context;
    	const post = await fetchPost(id.toString());
    	return {
    		props: { post }
    	}
    }
    
    export default function Post({ post }){
    	// post로 렌더링
    }
    ```