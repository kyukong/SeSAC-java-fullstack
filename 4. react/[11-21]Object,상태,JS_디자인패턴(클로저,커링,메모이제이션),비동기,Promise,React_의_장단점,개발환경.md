# [11/21] React (Object, 상태, JS 디자인패턴(클로저, 커링, 메모이제이션), 비동기, Promise, React 의 장단점, 개발환경)

## Heap Area

- Javascript 의 모든 요소는 객체(obj)
- 모든 요소는 Heap Area 에 저장됨
    - 함수, 메서드, 배열 등 모두 객체로 취급
- `scope` : 실행 Context 내에서 관리되는 값(변수)의 접근 범위

## 함수 지향형 프로그래밍

- 함수를 매개변수나 변수로 선언 가능한 프로그래밍
    - 함수 데이터 타입이 존재
- 생산성 향상 목적

## 객체 (Object)

- property 와 method 의 선언
- Javascript 의 객체는 기본적으로 전역 변수를 조회해옴
- 현재 객체의 값에 접근하고 싶은 경우 `this` 키워드 사용

    ```javascript
    let obj = {
      name: '다',
      name2: '라',
      test: function() {
        console.log(name); // 가
        console.log(this.name); // 다
        console.log(window.name); // 가
        console.log(name2); // error
      }
    }
    obj.test();
    ```

- 전역 기준이 아닌 객체를 기준으로 변수에 접근하고 싶은 경우 bind() 메서드 사용
    - bind() : 현재 객체의 부모 객체(노드)에 접근

    ```javascript
    let obj2 = {
      name: '다',
      test: function() {
        console.log(name); // 가
        console.log(this.name); // 가
        console.log(window.name); // 가
      }.bind(this) // 부모 노드의 this 를 가져옴 (obj2 -> window)
    }
    obj2.test();
    ```


## 상태 유지

- 함수를 호출할 때마다 1씩 증가된 값을 출력하는 함수 구현
    - 함수 내에서 지역 변수 선언 시 함수 호출 시마다 cnt 값 초기화
        - 상태가 유지되지 않음
    - 함수가 호출될 때 새로운 실행 Context 가 생성되며 내부에 변수 생성
    - 함수 종료 시 실행 Context 소멸되어 매번 초기화 진행

    ```javascript
    function count() {
      var cnt = 0;
      return ++cnt;
    }
    console.log(count()); // 1
    console.log(count()); // 1
    ```

- 상태를 유지하기 위해 함수 외부의 전역 변수로 선언
    - Global Context 에서 number 선언 후 이에 접근해서 count

    ```javascript
    var number = 0;
    function count() {
      number++;
      return number;
    }
    
    console.log(count()); // 1
    console.log(count()); // 2
    console.log(count()); // 3
    ```

- 다만 동일한 이름의 함수를 선언할 경우 예상과는 다르게 동작할 수 있음
    - 이름이 동일한 함수를 재선언 할 경우 함수 오버라이딩 처리됨
    - 위에 있는 console.log 에서 호출한 count 는 두번째로 선언한 함수
    - Global Context 에 변수 저장 시 초기화한 값이 아닌 undefined 로 초기화
    - number 값이 초기화되기 전에 count 를 실행하여 undefined 로 연산 처리 진행
    - 변수 실제 초기화는 스크립트를 읽는 순서대로 진행 (0으로 초기화)

    ```javascript
    function count() {
      var cnt = 0;
      return ++cnt;
    }
    console.log(count()); // NaN
    console.log(count()); // NaN
    
    var number = 0;
    
    function count() {
      number++;
      return number;
    }
    console.log(count()); // 1
    console.log(count()); // 2
    console.log(count()); // 3
    ```

- 전역 변수로 선언하면 전역적인 기능을 쉽게 구현할 수 있으나 side effect 가 발생할 수 있음
    - 여러 명이 개발하는 경우 이를 수정하는 대상이 자유롭기 때문에 이러한 문제가 발생할 수 있음
    - 이러한 문제를 해결하기 위해 지역 변수로 관리
    - 다만 지역 변수로 관리할 경우 함수를 호출할 때마다 초기화됨
    - 지역 변수를 사용하면서 상태 유지가 되는 함수를 구현 => 클로저
    - 이렇게 side effect 가 발생하지 않는 함수를 **순수 함수**라고 표현

    ```javascript
    var number = 0;
    
    function count1() {
      number++;
      return number;
    }
    number = 100;
    
    console.log(count1()); // 101
    console.log(count1()); // 102
    console.log(count1()); // 103
    ```


## 클로저

- 지역 변수를 사용하면서 상태 유지가 되는 함수를 구현하기 위해 클로저 디자인 패턴 사용
    - 지역 변수 : 외부의 변경에 영향을 미치지 않는 기능 구현
    - 상태 유지 : 함수를 호출한 이력이 유지되는 기능 구현
- 클로저 디자인 패턴 : 함수를 return 하는 함수 패턴
    - 선언한 함수를 변수에 저장하여 실행 Context 정보 유지
    - 함수 내에서 함수 생성하여 새로운 실행 Context 생성
    - **변수 접근 시 현재의 실행 Context 에 값이 없으면 상위의 실행 Context 에 접근하는 성질 이용**
    - 본인을 호출한 count2 의 함수 실행 Context 에 저장되어 있는 count 변수에 접근
    - Java 의 private static 변수와 동일한 역할 수행

```javascript
function count2() {
  var count = 0;
  return function() {
    return ++count;
  }
}

// 새로운 실행 Context 생성
// 해당 실행 Context 는 언제 끝날지 모르기 때문에 계속해서 유지
let cl = count2(); 
console.log(cl()); // 1
console.log(cl()); // 2

let cl2 = count2();
console.log(cl2()); // 1
console.log(cl2()); // 2
```

- 변수에 함수를 저장할 경우 해당 함수가 언제 종료될지 모르기 때문에 계속해서 메모리 점유
- html 기반으로 이루어져 있을 경우 다른 html 페이지로 이동할 때 메모리 초기화
    - 클로저 사용을 지향
- SPA 의 경우 html 기반이 아니기 때문에 메모리 점유 고려 필요

## 커링(currying) 디자인 패턴

- 일부 인자를 미리 전달 후 나머지 인자를 전달하는 함수 디자인 패턴
- 하나의 공용 함수가 있는 경우 이를 세부적인 기능을 하는 함수로 나누고 싶을 때 유용
- 클로저를 활용하여 커링 디자인 패턴 구현

    ```javascript
    function add(x, y) {
      return x + y;
    }
    
    function add2(x) {
      return function(y) {
        return x + y;
      }
    }
    
    console.log(add(1, 2)); // 3
    console.log(add2(1)(2)); // 3
    ```

- 커링 디자인 패턴을 이용하여 배열의 값을 제곱한 배열을 반환하는 함수 구현

    ```javascript
    function curry(fn) {
      return function(x) {
        return x.map(fn);
      }
    }
    console.log(curry((x) => x * x)([1, 2, 3, 4, 5])); // [1, 4, 9, 16, 25]
    ```


## 메모이제이션(Memoization) 디자인 패턴

- 알고리즘 중 DP 와 유사
- 진행되는 과정을 특정 배열에 기록(memory)하며 연산 횟수를 줄이는 방법
- Memoization 패턴을 이용하지 않았을 경우 15번 연산 진행

    ```javascript
    let cnt1 = 0;
    function fib(n) {
      cnt1++;
      if (n < 2) return n;
      return fib(n - 2) + fib(n - 1);
    }
    let r = fib(5);
    console.log(cnt1, r); // 15 5
    ```

- Memoization 패턴을 이용할 경우 9번 연산으로 연산 횟수 감소

    ```javascript
    let result = [];
    cnt1 = 0;
    function fib2(n) {
      cnt1++;
      if (n < 2) return result[n] = n;
    
      if (result[n] == undefined) {
        result[n] = fib2(n - 2) + fib2(n - 1);
      }
      return result[n];
    }
    r = fib2(5);
    console.log(cnt1, r, result); // 9 5 [0, 1, 1, 2, 3, 5]
    ```


## Javascript 의 비동기

- 첫번째 줄의 console.log 는 data 를 받아오기 전에 출력하여 빈 칸으로 출력됨
- 두번째 console.log 까지 도달한 후에야 값을 받아와 정상 출력됨

```javascript
let result = "";
$.get('msg.json', function(data) {
  result = data;
  console.log(result); // ""
});
console.log(result); // Object [ text: "Hello World" ]
```

- 자바스크립트는 Single Thread 로 이루어짐
- 화면과 관련된 기능을 제공하기 때문에 사용자가 제어하는 마우스의 움직임과 화면 단의 움직임을 동시에 처리해야 함
- Multi Thread 와 같이 동작을 처리하기 위해 비동기 처리 선택
- 실행 순서를 보장하지 않음 → Promise 패턴 등장

## Promise 패턴

```javascript
let result = "";
$.get('msg.json', {})
.done(function(data) {
  result = data;
  console.log(result); // Object [ text: "Hello World" ]
})
.fail(function(data) {
  console.log("오류" + data);
})
console.log(result);
```

- done 메서드를 이용해 현재 처리를 모두 마친 후 다음 내용 처리 진행
- 실패할 경우를 대비하기 위해 fail 메서드 사용

## React

- Javascript 는 자유도가 매우 높아 side effect 가 발생할 가능성이 높음
    - 개인 프로젝트가 아닌 팀 프로젝트의 경우 문제가 발생할 수 있음
- 문제가 발생하는 상황을 줄이기 위해 Javascript 의 기능을 제한한 react 사용
- 기능과 UI 를 묶어놓은 **Component 를 선언하여 재사용**하기 위해 react 사용
    - 반복적인 작업이 없으면 굳이 React 를 사용하지 않아도 됨
    - UI 유연성, 확장성 확보
- 페이스북 주도로 개발된 오픈 소스 경량 라이브러리 (프레임워크 X)
    - https://github.com/facebook/react
- 최근 유행하는 SPA(Single Page Application) 개발이 편리
- 리액트 네이티브를 이용하면 안드로이드, iOS 와 같은 모바일 앱 개발이 가능
    - 다만 안드로이드와 iOS 의 공통 부분에서만 사용 가능 (어느 정도 제한이 있음)
- UI 에 관련되지 않은 기능들은 외부 라이브러리(Axios, Redux 등)를 이용해야 함

## React 의 특징

- 컴포넌트 기반으로 코드의 재활용성을 높일 수 있음
- **가상돔(Virtual DOM)을 제공하여 효율적인 DOM 관리**와 화면 처리 지원
    - HTML 문서의 요소들 기반으로 DOM 이 생성
    - 다만 DOM 의 요소가 변경될 때마다 렌더링 과정이 실행됨 (rerendering, reload)
    - 가상돔은 메모리에 DOM 구조를 만들고, 요소 변경이 발생되면 가상돔 변경 후 한번에 모아 DOM 에 적용 (React Element)
        - 렌터링 부하를 줄여 성능 향상 (렌더링 프로세스 횟수 감소)
    - 브라우저에 대한 의존성이 없어, 다양한 브라우저에 이용 가능
- JSX(JavaScript Syntax Extension, JavaScript XML)를 이용하여 DOM Tree 생성
    - VDOM 에게 전달하기 위한 요소(element) 생성
- 모델(Model)과 뷰(View)사이에 단방향 데이터 바인딩을 제공
    - Source(Model) 코드가 변경되면 Target(View)이 변경
    - 양방향 데이터 바인딩 : Target 의 변경사항을 감지하여 Source 에 반영하는 옵저버 관리

## React 의 단점

- 브라우저에 의해 처리되는 HTML 보다 무겁고 처리 속도가 느림
    - HTML : Javascript 기반의 view 기술 (Tag 기반)
    - 렌더링 한 번을 기준으로는 JS 보다 React 가 느림
- 컴포넌트 기반으로 다양하고 복잡한 구성의 화면일 경우 오히려 생산성이 떨어질 수 있음
- 복잡한 상태 관리

## 개발환경

- React 는 라이브러리이기 때문에 어플리케이션 개발 및 배포를 위해서는 여러 프로그램을 이용하여 구축해야 함
    - node.js, 바벨(Bable), 웹팩(Webpack), Visual Studio Code 등 사용

### node.js

- `.js` 파일을 실행하기 위한 환경 (Compiler)
- `.class` → JVM
- `.html` → Browser
- Chrome V8 JavaScript 엔진을 이용하여 만든 **JavaScript 애플리케이션**의 실행환경(Runtime)
- libuv 라이브러리를 이용하여 이벤트기반, 논블로킹 I/O (비동기) 방식 환경을 제공
- **npm(Node Package Manager)를 제공**하여 다양한 JavaScript 패키지의 의존성 설치 및 관리
    - 모듈의 모임 > 패키지 / 패키지의 모임 > 라이브러리 / 라이브러리의 모임 > 프레임워크

### 바벨(Babel)

- Node.js 기반으로 실행되며, 일반적으로 Webpack 등 다른 애플리케이션과 함께 사용
- ES6, JSX 같은 최신 버전의 코드를 **ES5 이하 또는 특정 버전으로 변환**해주는 ~~컴파일러 (compiler)~~ / 트랜스파일러(transpiler)
    - JSX(JavaScript XML) : XML 과 유사한 구문을 사용하여 DOM 트리를 생성할 수 있는 Javascript 를 확장한 문법 (JavaScript + XML)
    - 트랜스파일러라는 표현이 더 적합
- 자바스크립트의 브라우저 호환성 확보를 위해 사용

### 웹팩(Webpack) == maven

- Node.js 기반으로 실행
- 어플리케이션 제작에 이용되는 의존성들을 가지고 있는 자원(javascript, css, 이미지 등의 모듈)들을 분류하고 정리하여 하나로 묶어 (Binding) 정적 자원(Static Asset)으로 만들어 **배포 및 실행을 편리**하게 함
- 배포 가능한 형태의 하나의 파일로 생성
- 다양한 플러그인(Plug-in)을 제공하여 개발에 사용되는 다양한 모듈을 처리

### Visual Studio Code

- Microsoft에서 제공하는 Electron 프레임워크와 TypeScript 기반의 무료 통합 개발 환경
- 애플리케이션 제작에 필요한 **환경 설정, 모듈 관리, 코딩, 디버깅, 테스트, 배포** 등 전반적인 과정 처리

### 리액트 생태계(React Ecosystem)

- 리액트는 UI 중심의 자유도가 높은 라이브러리
- 기본 기능 외의 기능은 다른 라이브러리 또는 프레임워크를 이용하거나 통합
- 리액트와 관련 기술들을 포함하여 생태계라고 부름
- 주요 라이브러리
    - 리액트 라우터(React router), 리액트 리덕스(React Redux), 스타일드 컴포넌트(Styled Components), Babel, Polyfill, Webpack, Axios, Next.js 등

## Proejct 생성 관련 명령어

- npm install -g npm
    - npm 관련 업데이트
- npx create-react-app react-app
    - 프로젝트 생성 및 Git 저장소 생성
    - npx : Node Package eXecute
    - react-app : 프로젝트 이름
- npm start
    - 프로젝트 시작
    - 생성한 프로젝트 폴더에 접속한 이후 시작

## Project 내 주요 폴더 및 파일

- package.json
    - 프로젝트에 추가한 패키지들의 버전 기록 및 관리 파일
    - 프로젝트에서 npm 을 사용하려면 package.json 파일이 있어야 함
    - npm init 으로 파일 생성
- node_modules
    - 프로젝트에서 사용하는 패키지들
    - 현재 패키지를 생성할 때 활용된 패키지들

## Index.js

- index.html 과 index.js 는 이름 고정
    - 변경 시 webpack 수정
- index.js 의 `<App />` : App.js 파일 로드 및 반영
- import : 자원 참조
