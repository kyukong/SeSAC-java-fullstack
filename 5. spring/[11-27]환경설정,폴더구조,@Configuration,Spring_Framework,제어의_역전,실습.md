# [11/27] Spring (환경설정, 폴더구조, @Configuration, Spring Framework, 제어의 역전, 실습)

## 환경설정

- IntelliJ Spring Framework, lombok 설치
- Tomcat 9 셋팅

## 프로젝트 폴더 구조

- src
    - main
        - webapp
            - WEB_INF
                - spring
                    - appServlet
                        - servlet-context.xml : 웹과 관련된 스프링 설정 파일
                    - root-context.xml : 스프링 설정 파일
                - views : 템플릿 프로젝트의 jsp 파일 경로
                    - home.jsp
                - web.xml : **Tomcat 의 web.xml 파일**
- target
- pom.xml : **Maven 이 사용하는 pom.xml**

## @Configuration

- Java 에 대한 설정을 하는 경우 `@Configuration` 어노테이션을 이용해서 쉽게 설정할 수 있음
- 기존의 web.xml 파일 또한 @Configuration 을 이용하여 처리할 수 있음
    - AbstractAnnotationConfigDispatcherServletInitializer 클래스 상속

## Spring Framework

- 특정 기능을 위주로 간단한 jar 파일 등을 이용해서 모든 개발이 가능하도록 구성된 프레임워크
- Spring 2.5 버전 : 어노테이션(Annotation) 을 활용하는 설정을 토입하면서 편리한 설정과 개발이 가능하도록 지원
- Spring 3.0 버전 : 별도의 설정 없이도 Java 클래스만으로 설정 파일을 대신할 수 있게 지원
    - @Configuration
- Spring 4.0 버전 : 모바일 환경과 웹 환경에서 많이 사용되는 REST 방식의 컨트롤러 지원
    - @RestController
- Spring 5.0 버전 : Reactor 를 이용한 Reactive 스타일의 개발 환경 지원

## 스프링의 주요 특징

- POHO(Plain Old Java Object) 기반의 구성
- 의존성 주입(DI) 과 스프링
    - Spring 내 ApplicationContext 가 관리
- AOP(Aspect Oriented Programming) 의 지원
    - 핵심 비즈니스 로직에만 집중해서 코드 개발
    - 각 프로젝트마다 다른 관심사를 적용할 때 코드의 수정을 최소화함
    - 원하는 관심사의 유지보수가 수월한 코드로 구성
- 트랜잭션의 지원
- 편리한 MVC 구조
- WAS 의 종속적이지 않은 개발 환경

## 제어의 역전과 의존성 주입

- 제어의 역전(IoC, Inversion of Control)
    - 사용자의 제어권을 다른 주체에게 넘여 의존 관계의 방향을 바꾸는 것
    - 설계 원칙
- 의존성 주입(Dependency Injection)
    - 클래스 외부에서 의존되는 것을 대상 객체의 인스턴스 변수에 주입하는 기술
    - 구현 방법

## Servlet Project → Spring Framework 실습

- web.xml 에 DispatcherServlet 설정 추가
- servlet-context.xml 에 Controller 관련 빈 등록
