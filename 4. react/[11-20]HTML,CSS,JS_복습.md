# [11/20] React (HTML, CSS, JS 복습)

## 추상화

- 문제를 찾아 분석하여 핵심을 찾아가는 과정
- 프로그래밍의 추상화
    - 명사와 동사로 표현
    - 변수와 메서드로 표현
    - 상태와 상태변경으로 표현

## 객체지향

- 요구사항을 반영하기 간단
- 업무 분담 과정이 간단 (MSA)

## a tag

- `href` 속성에 정의한 URL 로 제어를 이동
    - URL 을 주소로 가지는 Web Server 에게 요청 및 화면 replace
- `#` : 화면 제어를 현재 페이지로 유지 → replace 되지 않음

## CSS 우선순위

- 기본 선택자 > 복합 선택자
- 동일한 수준일 경우 나중에 정의한 것이 우선순위가 더 높음

## CSS 선택자

- 속성 선택자  : 조건을 기준으로 요소 선택

```html
<style>
    /* 기본 선택자 */
    .class01 {
      background-color: silver;
    }
    #fruit {
      background-color: transparent;
    }

    /* 복합 선택자 : 기본 선택자들의 조합 */
    ul li { /* 자손 */
      color: blue;
    }
    ul > li { /* 자식 */
      background-color: pink;
    }
    li#기준 ~ li { /* 바로 뒤의 형제들 */
      color: plum;
    }
    li#기준 + li { /* 바로 뒤의 형제 */
      color: green;
    }

    /* 속성 선택자 */
    li[title] { /* 속성 존재 여부 */
      color: orange;
    }
    li[title=할인] { /* 속성 값이 같다면 */
      color: red;
    }
    li[title~="무료배송"]::after { /* 공백 기준으로 포함된다면 */
      content: " [무료배송]";
    }
    li[title$="품절"] { /* 속성 값이 끝난다면 */
      color: gray;
    }
    li[title^="hot"] { /* 시작된다면 */
      color: red;
    }
		li[title*="무료"] { /* 포함된다면 */
      text-decoration: underline;
    }

		/* 가상 선택자 */
    #fruit > ol > li:nth-child(2n) {
      background-color: lime;
    }
    #fruit > ol > li:hover {
      text-decoration: underline;
    }
  </style>
```

## jQuery

- 주로 css 요소 선택자를 사용하여 접근할 수 있어 사용

```html
<script src="https://code.jquery.com/jquery-3.7.1.min.js" integrity="sha256-/JqT3SQfawRcv/BIHPThkBvs0OEvtFFmqPF/lYI/Cxo=" crossorigin="anonymous"></script>
```

```html
<script>
  // jQuery
  $('li[title$="품절"]').on('click', function() {
    alert('품절입니다.');
  });

  // Javascript
  document.querySelector('li[title$="품절"]').addEventListener('click', function() {
    alert('품절입니다.');
  });
</script>
```

## 요소 생성

```html
<script>
    // jQuery
    $('h1').on('click', function() {
        $('body').append('<h1>Header</h1>');
    })
	
    // Javascript
    document.querySelector('h1').addEventListener('click', function() { 
        document.querySelector('body').appendChild(document.createElement('h1')).textContent = 'Header';
    })
</script>
```

- 브라우저가 렌더링 후 DOM 요소를 메모리에 로드 → CPU 접근
- Javascript(jQuery)를 이용하여 새로운 요소 생성 및 메모리 상에서 연결
- 새로운 요소 생성 시(DOM 에 변경이 있을 경우) 브라우저 Reload 하여 변경사항 적용

## Javascript 의 특징

- DOM 의 상속은 트리 구조로 이루어짐
- 상속은 자바와 같이 실제 상속이 아닌 정보를 복제하는 과정
- 타입 추론 과정이 있어 오버로딩이 없음

## Element vs Node 개념

- `Element` : 태그 기준으로 하나의 요소를 언급할 때 사용
- `Node` : DOM 트리 구조 관점에서 언급할 때 사용

## Javascript 의 동작 시점

- Javascript 는 DOM 트리의 Object 들이 메모리에 모두 로드된 이후에 사용 가능
    - 브라우저가 DOM 트리를 분석하여 Object 들을 메모리에 로드
    - Javascript 는 모든 것을 Object(객체)로 관리
- 메모리에 접근하여 Object 조작 → reload

## ECMA

- javascript 표준을 지정하는 기관
- ECMAScript(ES) : ECMA 에서 지정한 Script 기준
    - ES6 == ECMA2015

## 함수 지향형

- **데이터 타입이 함수인 변수를 매개변수로 사용하는 것**
    - 자바스크립트는 함수를 변수에 저장하여 관리할 수 있음
- 일급 시민이 함수일 경우 함수 지향형
- 일급 시민이 객체일 경우 객체 지향형

```html
<script>
    function add5(a, b, ...f) {
        var c = 0;
        f.forEach((g) => c += g(a, b));
        return c;
    }
    console.log(add5(1, 2, (a, b) => a + b, (a, b) => a - b));
</script>
```

## 일급 시민

- 자바스크립트의 함수는 일급 시민으로 함수형 프로그래밍 지원
- 함수를 변수처럼 사용할 수 있음
    - 변수에 함수를 대입할 수 있어야 함
    - 함수를 다른 함수에 인자로 넘길 수 있어야 함
    - 함수에서 함수를 생성하여 반환할 수 있어야 함

## Javascript 의 Heap Area

- Javascript 의 Memory Area 를 Heap Area 로 나타냄
- Javascript 실행 시 브라우저의 자바스크립트 엔진이 Global Context 생성
    - 변수와 함수, this 정보 Global Context 에 저장 → `호이스팅`
- 함수 호출 시 실행 Context 생성
    - 함수 호출 시 Global Context 에 있는지 확인 및 호출
    - 실행 Context 에 호출한 함수 정보와 변수, 호출된 arguments 정보 저장
        - 매개변수로 받아오지 않아도 함수 내에서 접근 가능

        ```javascript
        function add3() {
          var c = 0;
          for (i = 0, cnt = arguments.length; i < cnt; i++) {
              c += Number(arguments[i]);
          }
          return c;
        }
        
        console.log(add3(10)); // 10
        console.log(add3(10, 20)); // 30
        console.log(add3(10, 20, 30, 40, 50)); // 150
        ```

    - 함수 종료 시 CallStack 소멸 (Scope)
- `var` 키워드로 변수 선언 시 Global Context 에 변수만 저장 (값 X)
    - undefined 로 저장됨
    - undefined : 초기화되지 않음
- Global Context 에는 실제 값은 저장되지 않음

```html
<script>
    console.log(add(1, 2));
    function add(a, b) {
      return a + b;
    }
    console.log(add(1, 2));
</script>
```

- Global Area 에 function add 를 미리 저장해두었기 때문에 선언하기 전 호출 가능

## this 키워드

- root에서 this 키워드에 접근 시 window 객체를 의미
    - window == Global Context
    - window 는 생략 가능

    ```html
    <script>
    	var name = '가'; // window 객체의 property
    	let name1 = 'A'; // 일반 변수
    	
    	console.log(name, name1); // 가 A
    	console.log(this.name, this.name1); // 가 undefined
    	console.log(window.name, window.name1); // 가 undefined
    </script>
    ```

- 함수 내부의 this 키워드에 접근 시 함수를 호출한 객체를 의미

    ```html
    <script>
      function test() {
        var name = '나';
        console.log(name); // 나
        console.log(this.name); // 가, this == window
        console.log(window.name); // 가
      }
      test(); // window.test() 와 동일
    </script>
    ```
