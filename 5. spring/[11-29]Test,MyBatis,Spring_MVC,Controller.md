# [11/29] Spring (Test, MyBatis, Spring MVC, Controller)

## Annotation 을 이용한 Controller 빈 등록

- `@Controller` : 컨트롤러 역할을 하는 빈 등록
- `@RequestMapping` : `@Controller` 내에 나누어지는 URL 정의
    - method 를 지정하지 않으면 모든 Method 를 포함하는 형식 제공 (GET, POST, PUT, DELETE 등)

## Robert C. Martin 의 단위테스트 작성 지침 (FIRST)

- Fast : 테스트는 완전히 자동화 되어야 하고 [빠르게] 결과 확인이 가능해야 한다.
- Independent : 테스트는 다른 테스트에 의존하지 않고 [독립적으로] 테스트 가능해야 한다.
- Repeatable : 운영환경외의 환경에서도 실행 가능해야 하고 [반복 실행]이 가능해야 한다.
- Self-validation : 테스트는 true, false중 하나의 값을 결과로 [자가 검증] 한다. 두 가지 외에 직접 실행해야 확인이 되는 주관적인 결과를 배제하기 위해서이다.
- Timely : 단위테스트는 제품 코드를 구현하기 전에 작성하는 것이 좋다.
  (논쟁의 여지가 있음)

## `@Test` 어노테이션

- @Test(timeout=5000)
    - 이 테스트 메소드가 결과를 반환하는데 5,000밀리 초를 넘긴다면 이 테스트는 실패
- @Test(expected=RuntimeException.class)
    - 이 테스트 메소드는 RuntimeException이 발생해야 테스트가 성공, 그렇지 않으면 실패

## HikariCP

- 가벼운 용량과 빠른 속도를 가지는 JDBC 의 커넥션 풀 프레임워크
- Spring Boot 의 기본 커넥션 풀

## MyBatis

- XML 파일 또는 인터페이스를 생성하여 사용할 SQL 문 정의
- XML 파일을 읽어와 해당 기능을 처리해주는 구현체 (DAO) 자동 생성
- 기존의 SQL 을 그대로 활용할 수 있음
- 내부적으로 JDBC 의 PreparedStatement 를 이용해서 SLQ 처리

### 인터페이스

```java
import org.apache.ibatis.annotations.Select;

public interface TimeMapper {

    @Select("SELECT sysdate FROM dual")
    String getTime();

    String getTime2();
}
```

- 인터페이스만 생성하면 MyBatis 가 자동으로 구현체 생성하여 실행
    - getClass().getName() 으로 구현체 생성 검증

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.zerock.sample.mapper.TimeMapper;

import lombok.extern.java.Log;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("file:**/*-context.xml")
@Log
public class TimeMapperTest {

    @Autowired
    private TimeMapper timeMapper;

    @Test
    public void testConnection() {
        log.info(timeMapper.getClass().getName());
        log.info(timeMapper.getTime());
        log.info(timeMapper.getTime2());
    }
}
```

- 인터페이스에서 `@Select` 등의 쿼리문 확인 후 쿼리문이 없을 경우 xml 파일 확인
    - 인터페이스 하나당 XML 설정 파일 하나 생성
    - resource 에 XML 파일 생성 시 src path 와 동일하게 설정해야 함
    - id 는 method 명과 동일해야 함

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.9//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.zerock.sample.mapper.TimeMapper">
    <select id="getTime2" resultType="string">
        SELECT to_char(sysdate, 'YYYY/MM/DD HH24:MI:SS') as sysdba FROM dual
    </select>
</mapper>
```

## JDBC vs MyBatis

- JDBC
    - 직접 Connection 을 맺고 마지막에 close()
    - PreparedStatement 직접 생성 및 처리
    - PreparedStatement 의 setXXX() 등에 대한 모든 작업을 개발자가 처리
    - SELECT 의 경우 직접 ResultSet 처리
- MyBatis
    - 자동으로 Connection close() 가능
    - MyBatis 내부적으로 PreparedStatement 처리
    - #{prop} 와 같이 속성을 지정하면 내부적으로 자동 처리
    - 리턴 타입을 지정하는 경우 자동으로 객체 생성 및 ResultSet 처리

## SQLSessionFactory

- SQLSession 을 관리하는 객체
- SQLSession 을 통해서 Connection 을 생성하거나 SQL 을 전달하고 결과를 리턴받음

## log4jdbc-log4j2

- MyBatis 는 내부적으로 PreparedStatement 로 처리하기 때문에 실행되는 쿼리문에 `?` 로 변수를 처리함
- 변수의 값이 나와있지 않기 때문에 로그를 추적하기 어려움
- SQL 로그를 제대로 확인하기 위한 라이브러리

```xml
<!-- https://mvnrepository.com/artifact/org.bgee.log4jdbc-log4j2/log4jdbc-log4j2-jdbc4.1 -->
<dependency>
	<groupId>org.bgee.log4jdbc-log4j2</groupId>
	<artifactId>log4jdbc-log4j2-jdbc4.1</artifactId>
	<version>1.16</version>
</dependency>
```

## Spring MVC 의 구조

- 스프링은 하나의 기능만을 포함하는 것이 아닌 ‘코어’ 라는 프레임워크에 여러 서브 프로젝트를 결합하여 사용하는 형태
    - 스프링 MVC 와 같은 서브 프로젝트 조합 가능
    - WebApplicationContext 내부에 MVC 설정을 포함하는 구조

## 프로젝트의 로딩 구조

- 프로젝트 구동 시 관여하는 XML 은 web.xml, root-context.xml, servlet-context.xml
    - web.xml : Tomcat 이 읽는 설정 파일
    - root-context.xml, servlet-context.xml : Spring 관련 파일
- 프로젝트의 구동은 **web.xml** 에서 시작하며 아래와 같은 순서로 동작
- `<listener>` : 가장 먼저 Context Listener 설정
    - ContextLoaderListener 는 해당 웹 어플리케이션 구동 시 같이 동작하므로 프로젝트를 실행하면 로그 중에서 가장 먼저 출력됨
- `<context-param>` : **root-context.xml** 을 읽어와 전역적인 스프링 빈 객체 생성
    - 스프링의 영역 안에 생성되며, 객체들 간의 의존성 처리
- `<servlet>` : 서블릿과 관련된 설정 동작 (DispatcherServlet)
    - `<init-param>` : **servlet-context.xml** 실행하여 스프링 빈 등록
    - DispatcherServlet 에서 XmlWebApplicationContext 를 이용해서 servlet-context.xml 을 로딩하고 해석
    - 이 과정에서 등록된 빈들은 기존에 만들어진 빈들과 같이 연동

## 스프링 MVC 의 기본 사상

- 기존 Servlet 의 계층
    - Servlet/JSP > 개발자의 코드 영역
- 스프링 MVC 의 계층
    - Servlet/JSP > Spring MVC > 개발자의 코드 영역
    - Spring MVC 는 내부적으로 Servlet API 를 활용
- 과거에는 스프링 MVC 의 **클래스를 상속**하거나 **인터페이스를 구현**하는 형태로 구현
- 스프링 2.5버전 부터 **어노테이션** 및 **XML** 로 구현

## `@InitBinder`

- Controller 에서 파라미터를 원하는 형태 및 타입으로 자동 바인딩하기 위해 사용
- ‘2023-11-27’ 과 같은 문자열을 Date 타입으로 변환하려고 할 때
    - Controller 쪽에서 정의 가능
    - WebDataBinder 에 CustomEditor 추가하여 원하는 포맷팅 설정 가능

        ```java
        import java.text.SimpleDateFormat;
        
        import org.springframework.beans.propertyeditors.CustomDateEditor;
        import org.springframework.stereotype.Controller;
        import org.springframework.web.bind.WebDataBinder;
        import org.springframework.web.bind.annotation.GetMapping;
        import org.springframework.web.bind.annotation.InitBinder;
        import org.springframework.web.bind.annotation.RequestMapping;
        import org.springframework.web.bind.annotation.RequestMethod;
        
        import net.developia.spring02.domain.SampleDTO;
        import net.developia.spring02.domain.TodoDTO;
        
        import lombok.extern.java.Log;
        
        @Controller
        @RequestMapping("/sample/*")
        @Log
        public class SampleController {
        
            @InitBinder
            public void initBinder(WebDataBinder binder) {
                SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
                binder.registerCustomEditor(java.util.Date.class, new CustomDateEditor(dateFormat, false));
            }
        
            @GetMapping("/ex03")
            public String ex03(TodoDTO dto) {
                log.info("todo: " + dto);
                return "ex03";
            }
        }
        ```

    - 날짜 변환은 간단하게 `@DateTimeFormat` 어노테이션을 사용해서 타입을 변환할 수 있음
        - DTO 에서 정의가능 (도메인)

        ```java
        import java.util.Date;
        
        import org.springframework.format.annotation.DateTimeFormat;
        
        import lombok.Data;
        
        @Data
        public class TodoDTO {
        
            private String title;
        
            @DateTimeFormat(pattern = "yyyy-MM-dd")
            private Date schedule;
        }
        ```


## `@ModelAttribute`

- 스프링 MVC 의 Controller 에서 객체를 이용하여 값을 받아오면 별도의 처리 없이도 화면에 해당 값이 전달됨
    - Java Beans 규칙에 맞는 객체인 경우 화면에 전달
    - 생성자가 없거나 빈 생성자를 가져야 하며, getter/setter 를 가진 클래스
- 다만 객체가 아닌 일반 변수의 경우 화면으로 전달되지 않음
- Java Beans 규칙에 맞지 않은 값 또한 화면으로 전달하기 위해 `@ModelAttribute` 사용
    - jsp 에서 SampleDTO 라는 이름으로 객체에 접근 가능
    - jsp 에서 page 라는 이름으로 변수에 접근 가능

    ```java
    @GetMapping("/ex04")
    public String ex04(SampleDTO dto, @ModelAttribute("page") int page) {
    	return "/sample/ex04";
    }
    ```
