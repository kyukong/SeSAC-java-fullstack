# [11/23] React (state, React DOM Router, SPA, Routing)

## JSX(JavaScript Syntax Extension, JavaScript XML)

- React 요소 트리를 생성하는 자바스크립트 확장 표현식
- React 요소를 편리하게 생성하기 위한 문법 설탕(syntactic sugar)
    - 대표적인 문법 설탕 : 람다, 삼항연산자 등
- 웹팩의 번들링 과정에서 바벨에 의해 자바스크립트 코드로 변환
- 바벨은 React.createElement() 를 호출하여 컴파일
    - ES6, JSX → 표준 코드로 변환

### 문법

- 기본적으로 XML 규칙을 따름
- 단일 요소가 아닌 경우 최상위 루트 요소가 있어야 함
- 코드는 소문자를 기본으로 하고, 리액트 사용자 컴포넌트는 대문자로 시작한느 파스칼 표기법 사용
- Boolean, null, undefined 는 출력되지 않음
- 표현식(Expression)은 `{ ... }` 를 이용하여 변수값, 객체, 연산자 함수 호출 등의 자바스크립트 구문ㅇ르 사용할 수 있음

## State

- 컴포넌트의 상태를 동적으로 관리하기 위한 기본 React Hooks 중 하나
- props 와 다르게 컴포넌트 내부에서 관리
    - props 는 부모 노드에서 자식 노드로 값 전달
    - state 는 자식 노드에서의 이벤트(CallBack)를 부모 노드에게 전달
    - 하나의 변수에 대해 상태(값)이 변하는 것을 관리

```javascript
import { useState } from 'react'; // react 의 hook import

const [mode, setMode] = useState('WELCOME');
```

- useState(초깃값) : 상태를 관리하고자 하는 값의 초깃값 설정
- const [mode, setMode]
    - useState 로 받아온 첫번째 요소(mode) : 상태의 값을 읽을 때 쓰는 데이터
    - useState 로 받아온 두번째 요소(setMode) : 상태의 값을 변경할 때 사용하는 함수

        ```javascript
        setMode('WRITE'); // mode == 'WRITE';
        setMode('READ'); // mode == 'READ';
        ```


## State 사용 시 이벤트의 전달

- State 를 사용하여 자식 노드에서 이벤트가 발생하면 부모 노드에게 해당 이벤트 전달
- 이벤트가 올 경우 부모 노드는 그에 맞는 상태값 변경 및 노드 관리
- 상태값 변경 함수를 사용하여 상태값이 변경되면 컴포넌트가 비동기적으로 리렌더링 됨
- 노드의 클릭 이벤트를 이용하여 형제 노드의 값을 변경하고자 할 때 state 사용
    - DOM Tree 는 단방향(부모 → 자식) 구조로 그 외의 방향으로 흐름을 제어하고 싶은 경우 state 사용
    - 자식 노드인 Nav 나 Header 에서 이벤트를 호출하여 setMode 가 요청되면
        - 자식 노드는 상태값, 상태값 변경 함수에 대한 정보가 전혀 없음
        - 부모 노드에서 이벤트 시 실행되는 함수를 관리하고, 자식 노드는 이벤트만을 호출 (Hook)
    - 부모 노드에서 관리하는 상태값이 변경되면 이후의 동작 수행
    - 자식 노드(Nav) 에는 props 로 넘어온 함수를 onClick 이벤트로 실행
        - props 로 넘어온 함수에 대한 정보는 전혀 없음

        ```javascript
        function Nav(props) {
        const lis = [];
        
        for (let i = 0, cnt = props.topics.length; i < cnt; i++) {
          let t = props.topics[i];
          lis.push(
            <li key={t.id}>
              <a
                id={t.id}
                href={'/read/' + t.id}
                onClick={(event) => {
                  event.preventDefault();
                  props.onChangeMode(Number(event.target.id));
                }}
              >
                {t.title}
              </a>
            </li>,
          );
        }
        ```

    - 자식 노드에서 이벤트가 발생하면 부모 노드에게 이벤트 훅 전달
    - 이벤트 훅이 오면 App 컴포넌트 다시 실행 (렌더링X)
        - 이벤트 발생 시 상태값을 변경하는 함수를 호출하여 상태값 변경
        - 원하는 노드 생성 → 해당 노드만 리렌더링

        ```javascript
        function App() {
          const [mode, setMode] = useState('WELCOME');
          const [id, setId] = useState(null);
        
          const topics = [
            { id: 1, title: 'html', body: 'html is ...' },
            { id: 2, title: 'css', body: 'css is ...' },
            { id: 3, title: 'javascript', body: 'javascript is ...' },
          ];
        
          // 부모 노드에서 새로운 자식 노드 생성
          let content;
          if (mode === 'WELCOME') {
            content = <Article title="Welcome" body="Hello, Web"></Article>;
          } else if (mode === 'READ') {
            let title,
              body = null;
            for (let i = 0; i < topics.length; i++) {
              console.log(topics[i].id, id);
              if (topics[i].id === id) {
                title = topics[i].title;
                body = topics[i].body;
              }
            }
            content = <Article title={title} body={body}></Article>;
          }
        
          return (
            <div className="App">
              <Header
                title="WEB"
                onChangeMode={() => {
                  setMode('WELCOME');
                }}
              ></Header>
              <Nav
                topics={topics}
                onChangeMode={(_id) => {
                  setMode('READ');
                  setId(_id);
                }}
              ></Nav>
              {content}
            </div>
          );
        }
        ```


## 지역 변수 vs State

- 2개 이상의 변수를 사용하여 관련 노드를 수정할 경우 렌더링 횟수에 대한 차이 발생
- 지역 변수를 사용하여 자식 노드 변경 시 선언된 변수가 변경될 때마다 렌더링 실행
    - 2개 변수 사용 시 2번 렌더링
- State 를 사용하여 자식 노드 변경 시 선언된 변수가 모두 변경된 이후에 렌더링 실행
    - 2개 상태 사용 시 1번 렌더링
    - 렌더링 되는 시점은 react 가 판단하기 때문에 정확히 언제 일어나는지 알 수 없음

## CRUD 예제 코드

<details>
<summary>Code</summary>
<div>

```javascript
import './App.css';
import { useState } from 'react';

function Header(props) {
  return (
    <header>
      <h1>
        <a
          href="/"
          onClick={function (event) {
            event.preventDefault();
            props.onChangeMode();
          }}
        >
          {props.title}
        </a>
      </h1>
    </header>
  );
}

function Nav(props) {
  const lis = [];

  for (let i = 0, cnt = props.topics.length; i < cnt; i++) {
    let t = props.topics[i];
    lis.push(
      <li key={t.id}>
        <a
          id={t.id}
          href={'/read/' + t.id}
          onClick={(event) => {
            event.preventDefault();
            props.onChangeMode(Number(event.target.id));
          }}
        >
          {t.title}
        </a>
      </li>,
    );
  }

  return (
    <nav>
      <ol>{lis}</ol>
    </nav>
  );
}

function Article(props) {
  return (
    <article>
      <h2>{props.title}</h2>
      {props.body}
    </article>
  );
}

function Create(props) {
  return (
    <article>
      <h2>Create</h2>
      <form
        onSubmit={(event) => {
          event.preventDefault();
          const title = event.target.title.value;
          const body = event.target.body.value;
          props.onCreate(title, body);
        }}
      >
        <p>
          <input type="text" name="title" placeholder="title" />
        </p>
        <p>
          <textarea name="body" placeholder="body"></textarea>
        </p>
        <p>
          <input type="submit" value="Create"></input>
        </p>
      </form>
    </article>
  );
}

function Update(props) {
  const [title, setTitle] = useState(props.title);
  const [body, setBody] = useState(props.body);
  return (
    <article>
      <h2>Update</h2>
      <form
        onSubmit={(event) => {
          event.preventDefault();
          const title = event.target.title.value;
          const body = event.target.body.value;
          props.onUpdate(title, body);
        }}
      >
        <p>
          <input
            type="text"
            name="title"
            placeholder="title"
            value={title}
            onChange={(event) => {
              setTitle(event.target.value);
            }}
          />
        </p>
        <p>
          <textarea
            name="body"
            placeholder="body"
            value={body}
            onChange={(event) => {
              setBody(event.target.value);
            }}
          ></textarea>
        </p>
        <p>
          <input type="submit" value="Update"></input>
        </p>
      </form>
    </article>
  );
}

function App() {
  const [mode, setMode] = useState('WELCOME');
  const [id, setId] = useState(null);
  const [nextId, setNextId] = useState(4);
  const [topics, setTopics] = useState([
    { id: 1, title: 'html', body: 'html is ...' },
    { id: 2, title: 'css', body: 'css is ...' },
    { id: 3, title: 'javascript', body: 'javascript is ...' },
  ]);

  // 자식 노드에서 발생한 이벤트(Hook)를 이용하여 부모 노드에서 새로운 자식 노드 생성
  let content;
  let contextControl;
  if (mode === 'WELCOME') {
    content = <Article title="Welcome" body="Hello, Web"></Article>;
  } else if (mode === 'READ') {
    let title,
      body = null;
    for (let i = 0; i < topics.length; i++) {
      if (topics[i].id === id) {
        title = topics[i].title;
        body = topics[i].body;
      }
    }
    content = <Article title={title} body={body}></Article>;
    contextControl = (
      <>
        <li>
          <a
            href={'/update/' + id}
            onClick={(event) => {
              event.preventDefault();
              setMode('UPDATE');
            }}
          >
            Update
          </a>
        </li>
        <li>
          <input
            type="button"
            value="Delete"
            onClick={() => {
              const newTopics = [];
              for (let i = 0; i < topics.length; i++) {
                if (topics[i].id !== id) {
                  newTopics.push(topics[i]);
                }
              }
              setTopics(newTopics);
            }}
          />
        </li>
      </>
    );
  } else if (mode === 'CREATE') {
    content = (
      <Create
        onCreate={(_title, _body) => {
          const newTopic = { id: nextId, title: _title, body: _body };
          const newTopics = [...topics];
          newTopics.push(newTopic);
          setTopics(newTopics);
          setMode('READ');
          setId(nextId);
          setNextId(nextId + 1);
        }}
      ></Create>
    );
  } else if (mode === 'UPDATE') {
    let title,
      body = null;
    for (let i = 0; i < topics.length; i++) {
      if (topics[i].id === id) {
        title = topics[i].title;
        body = topics[i].body;
      }
    }
    content = (
      <Update
        title={title}
        body={body}
        onUpdate={(title, body) => {
          const newTopics = [...topics];
          const updatedTopic = { id: id, title: title, body: body };
          for (let i = 0; i < newTopics.length; i++) {
            if (newTopics[i].id === id) {
              newTopics[i] = updatedTopic;
              break;
            }
          }
          setTopics(newTopics);
          setMode('READ');
        }}
      ></Update>
    );
  }

  return (
    <div className="App">
      <Header
        title="WEB"
        onChangeMode={() => {
          setMode('WELCOME');
        }}
      ></Header>
      <Nav
        topics={topics}
        onChangeMode={(_id) => {
          setMode('READ');
          setId(_id);
        }}
      ></Nav>
      {content}
      <ul>
        <li>
          <a
            href="/create"
            onClick={(event) => {
              event.preventDefault();
              setMode('CREATE');
            }}
          >
            Create
          </a>
        </li>
        {contextControl}
      </ul>
    </div>
  );
}

export default App;
```

</div>
</details>


## React DOM Router

- 리액트 라우터는 클라이언트 내에서 자원을 탐색하는 클라이언트측 라우팅(Client-Side Routing) 을 지원
- 라우팅(Routing)은 지정된 자원을 경로를 이용하여 탐색하는 과정
    - 일반적으로 서버에 존재하는 자우너에 접근할 때 많이 이용
    - 최근에는 물리적으로 존재하지 않는 서비스를 찾아가는 탐색 과정도 포함
- 리액트 라우터는 리액트 내에서 **컴포넌트와 같은 자원에 경로를 설정**할 수 있는 다양항 방법 제공
    - Network 장비 중 하나인 라우터의 라우팅 테이블(Route table) 과 비슷
- React Router 는 react-router-dom 과 react-route-native 를 제공
- 다양한 경로를 관리할 수 있는 다양한 라우터와 라우터 컴포넌트, 기본 컴포넌트, 후크(Hooks) 등을 제공
- 2021년에 발표된 v6 는 리액트 v16.8 이상을 지원
- 최근에 유행하는 SPA(Single Page Application) 제작에 필수적인 기술
- Page 새로 고침을 피하기 위해 `<a />` 보단 `<Link />` 컴포넌트 사용을 권장

## SPA(Single Page Application)

- 단일 페이지로 구성된 어플리케이션
- 빠르고 자연스러운 화면 전환을 할 수 있고, 재사용 높은 컴포넌트를 이용하여 구성하기 수월
- UI 처리에 관련된 서버 요청이 줄어 들어 네트워크 트래픽 및 서버 자원의 사용을 줄일 수 있음
- 전체적인 화면 구현 자원을 한 번에 모두 다운로드 후 출력하게 되어 초기화면이 늦게 출력될 수 있음
- 크롤링(Crawling) 기반의 검색엔진 최적화(Search Engine Optimization, SEO) 에 불리
- 원활한 유지보수 및 업무 분담을 위해 신중한 설계 필요

## Routing 설정

- 라우팅 대상이 되는 컴포넌트를 라우팅 태그에 포함시켜 라우팅 태그를 부모로 설정
    - 보통 App

```javascript
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
	<BrowserRouter>
		<App />
	</BrowserRouter>
);
```

## 라우터 컴포넌트 (Router Component)

- 리액트 라우터는 라우팅 기능을 위해 다양항 라우터 컴포넌트를 제공
    - BrowserRouter, HashRouter, MemoryRouter, NativeRouter, StaticRouter 등
- 라우터(Router) 컴포넌트는 Router 컴포넌트들의 상위 인터페이스
- 상위 컴포넌트인 라우터 컴포넌트의 속성은 대부분의 라우트 컴포넌트에서 공통 속성으로 이용될 수 있음
    - basename : 공통 상위 루트 지정
        - App 생략 가능

    ```javascript
    function App() {
    	return (
    		<BrowserRouter basename='/root'>
    			<Routes>
    				<Route path='/intro' />
    			<Routes />
    		</BrowserRouter>
    	);
    }
    ```


## BrowserRouter

- 브라우저의 주소창에 출력되고 있는 URL 을 기준으로 동작

    ```javascript
    http://localhost:3000/board/read/1
    http://localhost:3000/board/edit/1
    http://localhost:3000/board/update/1
    ```

- 리액트 라우터의 도움을 받고 싶은 컴포넌트의 최상위 컴포넌트를 감싸는 래퍼 컴포넌트
- HTML5 History API 를 이용하여 History Stack 을 조작하여 URL 과 UI 를 동기화
    - window.history 객체
- URL 요청 시 가장 먼저 History Stack 을 조회하여 내역 확인
    - Stack 에 이력이 있으면 해당 내용 처리
    - 없으면 서버에게 요청 전송 ⇒ 서버에 API 가 없으면 404 Not Found 발생 가능성 있음
- URL 이 달라지면(요청하면) path 가 일치하는 컴포넌트 리렌더링
- 새로 고침을 하는 경우 서버에 요청이 전달되어 HTTP 404 Not Found 오류가 발생될 수 있음
    - History Stack 내역에서 커서를 이동하여 조작하다가, 새로운 요청을 보내 오류 발생
    - 서버에서 관리하는 URL 이 아니라 발생되는 것으로 서버에서 오류 처리를 설정하여 처리
    - Route 컴포넌트의 `path` 속성을 이용하여 처리

        ```javascript
        <Route path='*' element={<ErrorPage />} /> // ErrorPage 컴포넌트 정의 필요
        ```

        ```javascript
        <Route path='*' element={<Navigate to='/' />} /> // Navigate 컴포넌트 정의 필요
        ```

- URL 을 다양하게 응용하는 동적 페이지에 적합

## HashRouter

- URL 에 ‘#’ 을 이용하여 경로를 지정

    ```javascript
    http://localhost:3000/#/about
    http://localhost:3000/#/history
    ```

- ‘#’는 앵커(Anchor)로 같은 리소스 내에 이동하여 처리될 경로 지정
    - html 을 기준으로 동일 html 내에 `<a name=''/>` 로 북마크(bookmark) 하면 `#북마크` 로 이동
- ‘#’ 이후의 지정된 내용(fragment)은 서버로 전달되지 않아 같은 URL 인 경우 서버로 요청(request) 이 전달되지 않아 화면이 갱신(reflash) 되지 않음
    - History 객체에 접근하지도 않음
    - 서버에 요청을 보내지 않기 때문에 요청 경로에 대한 정의가 없어도 404 가 뜨지 않음
- URL 이 고정적인 정적 페이지에 적합

```javascript
function App() {
	return (
		<HashRouter>
			<Routes>
				<Route paht='/intro' /> // /#/about
				<Route path='/fag' /> // /#/fag 
			</Routes>
		</HashRouter>
	);
}
```

## Routes

- 현재 경로를 자식 컴포넌트 경로의 상위 경로로 설정
- 라우트(Route) 컴포넌트들의 부모 컴포넌트로 URL 을 이용하여 자식 라우트 컴포넌트 검색
- Routes 에 정의된 정보를 이용하여 테이블 형식으로 Route Config 생성
    - Route Config 는 라우트 컴포넌트의 트리 구조를 이용하여 라우트 컴포넌트의 트리상 위치(현재 위치)를 기준으로 route matches 를 생성
    - 일치하는 URL 이 있으면 Match 객체를 전달
    - Match 객체는 URL, URL Params, Pathname 등의 정보를 가지고 있음
- 경로가 변경될 때마다 자식 컴포넌트들의 경로와 UI 경로와의 연결을 최적화
- Routes 가 여러 개 구현되어 있으면 서로의 자식 Route 에 접근 불가능

## Route

- 리액트 라우터에서 가장 많이 사용되는 컴포넌트이며 중첩 허용
- URL 경로에 연결될 컴포넌트를 지정하고 렌더링 수행
- Route  컴포넌트의 중첩을 허용하여 Tree 구조 생성
- 속성
    - path : 문자열, 매칭될 URL 경로 지정
    - element : URL 에 매칭된 리액트 컴포넌트를 이용하여 리액트 요소 생성
    - Component : URL 에 매칭을 이용하여 리액트 요소 생성, RouterProvider 를 이용하여 생성
    - index : index 라우트로 지정되며 **페이지가 없을 경우에도 기본**으로 출력, 형제 라우트들이 있을 경우 **기본 라우트로 지정**
        - depth 별로 지정

    ```javascript
    <Routes>
    	<Route index path='/' element={<Home />} />
    	<Route path='/article:id' element={<Article />} />
    		<Route index path='/list' element={<ArticleList />} />
    		<Route path='/content' element={<ArticleContent />} />
    </Routes>
    ```


## Link

- 서버에 요청을 보내 화면이 새로고침 되는 `<a href='' />` 와 다르게 **History API 를 이용하여 브라우저 주소만 갱신**하여 새로고침 없이 이동
- Routes 없이 단독으로 사용 가능
- 속성
    - to : 문자열, 이동할 URL 경로 지정

    ```javascript
    <ul>
    	<li><Link to='/'>Home</Link></li>
    	<li><a href='/'>Home</Link></li>
    </ul>
    ```


## NavLink

- 기본적으로 Link 와 비슷하지만, `active`, `pending`, `transitioning` 상태를 알 수 있음
- 인식할 수 있는 상태를 이용하여 CSS 를 적용할 수 있음
- Routes 없이 단독적으로 사용 가능
- 상태별로 기본 클래스명이 지정되어 있음
- 속성
    - style : CSS 를 선언
    - className : 클래스명 지정
    - to : 이동할 URL 지정

    ```javascript
    // class 명 이용
    <NavLink to='about' className={
    	({isActive}) => isActive ? 'activated' : 'deactivated';
    }>
    	About
    </NavLink>
    ```

    ```javascript
    // class 객체 이용
    const activeStyle = {color: 'read'};
    const deactiveStyle = {color: 'gray'};
    
    <NavLink to='about' className={
    	({isActive}) => isActive ? activeStyle : deactiveStyle;
    }>
    	About
    </NavLink>
    ```


## Navigate

- useNavigate hook 의 속성 및 기능이 동일한 Wrapper 컴포넌트
- 렌더링이 일어나면 현재 경로를 변경
    - Route 사용 시 요청한 경로를 root 로 지정 → URL 고정
- 다른 리액트 컴포넌트와 잘 호환되는 useNavigate 사용 권장
- 속성
    - replace : true 면 이전 페이지로 이동할 수 없음
    - state : useLocation 등에서 사용될 상태 객체 전달
    - to : 이동할 URL 지정
