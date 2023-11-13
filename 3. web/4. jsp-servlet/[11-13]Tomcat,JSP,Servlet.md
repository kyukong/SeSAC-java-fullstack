# [11/13] JSP (Tomcat, JSP, Servlet)

## Tomcat 설치 및 설정

- tomcat 9 버전 설치
    - tomcat 10 에서 패키지 구조가 많이 변경되어 9 버전 선택
- server.xml 설정 변경

    ```html
    <!-- A "Connector" represents an endpoint by which requests are received
         and responses are returned. Documentation at :
         Java HTTP Connector: /docs/config/http.html
         Java AJP  Connector: /docs/config/ajp.html
         APR (HTTP/AJP) Connector: /docs/apr.html
         Define a non-SSL/TLS HTTP/1.1 Connector on port 8080
    -->
    <Connector port="80" URIEncoding="UTF-8" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxParameterCount="1000"
               />
    ```


## JSP 동작 과정

- JSP 호출 시 Tomcat 이 Java 클래스로 변경하여 실행

## Servlet 생성하기 (초기의 방법)

### 1. XML 파일에 선언

1. java servlet 코드 작성
    - servlet : HttpServlet 를 상속받은 자바 클래스
    - HttpServletRequest : HTTP 요청
    - HttpServletResponse : HTTP 응답
        - 해당 내용을 브라우저에게 전달

    ```java
    package basic;
    
    import java.io.IOException;
    import java.io.PrintWriter;
    
    import javax.servlet.ServletException;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    
    public class HelloServlet extends HttpServlet {
    
    	@Override
    	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    		response.setContentType("text/html;charset=UTF-8");
    		PrintWriter out = response.getWriter();
    		out.println("<html>");
    		out.println("<head><title>first servlet</title></head>");
    		out.println("<body><h1>Hello Servlet~</h1></body>");
    		out.println("</html>");
    		out.flush();
    		out.close();
    	}
    }
    ```

2. xml 작성
    1. servlet 선언

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd" version="4.0">
          <display-name>xciwep01</display-name>
          <welcome-file-list>
            <welcome-file>index.html</welcome-file>
            <welcome-file>index.htm</welcome-file>
            <welcome-file>index.jsp</welcome-file>
            <welcome-file>default.html</welcome-file>
            <welcome-file>default.htm</welcome-file>
            <welcome-file>default.jsp</welcome-file>
          </welcome-file-list>
          
        	<!-- servlet 추가 -->
          <servlet>
          	<servlet-name>helloServlet</servlet-name>
          	<servlet-class>basic.HelloServlet</servlet-class>
          </servlet>
        </web-app>
        ```

    2. servlet mapping

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd" version="4.0">
          <display-name>xciwep01</display-name>
          <welcome-file-list>
            <welcome-file>index.html</welcome-file>
            <welcome-file>index.htm</welcome-file>
            <welcome-file>index.jsp</welcome-file>
            <welcome-file>default.html</welcome-file>
            <welcome-file>default.htm</welcome-file>
            <welcome-file>default.jsp</welcome-file>
          </welcome-file-list>
          
          <servlet>
          	<servlet-name>helloServlet</servlet-name>
          	<servlet-class>basic.HelloServlet</servlet-class>
          </servlet>
          
        	<!-- servlet-mapping 추가 -->
          <servlet-mapping>
          	<servlet-name>helloServlet</servlet-name>
          	<url-pattern>/helloServlet</url-pattern>
          </servlet-mapping>
        </web-app>
        ```


### 2. Servlet 에 선언

1. java servlet 코드 작성

    ```java
    package basic;
    
    import java.io.IOException;
    import java.io.PrintWriter;
    
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    
    @WebServlet(urlPatterns = "/helloServlet")
    public class HelloServlet extends HttpServlet {
    
    	@Override
    	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    		response.setContentType("text/html;charset=UTF-8");
    		PrintWriter out = response.getWriter();
    		out.println("<html>");
    		out.println("<head><title>first servlet</title></head>");
    		out.println("<body><h1>Hello Servlet~</h1></body>");
    		out.println("</html>");
    		out.flush();
    		out.close();
    	}
    }
    ```

2. xml 작성
    - url-mapping 을 Servlet 에게 주었기 때문에 별다른 설정은 없음

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd" version="4.0">
      <display-name>xciwep01</display-name>
      <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>default.html</welcome-file>
        <welcome-file>default.htm</welcome-file>
        <welcome-file>default.jsp</welcome-file>
      </welcome-file-list>
    </web-app>
    ```


## JSP Connection

- HTTP Request Message

    ```
    GET /index.jsp HTTP/1.1
    Host: localhost:80
    ```

- HTTP Response Message

    ```
    Http/1.1 200
    Set-Cookie: JSESSION=asdf; Path=/; HttpOnly
    Content-Type: text/html;charset=UTF-8
    Content-Length: 169
    Date: Mon, 13 Nov 2023 02:36:13 GMT
    
    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8">
    <title>Insert title here</title>
    </head>
    <body>
    	<h1>Hello Servlet/JSP</h1>
    	Hello World
    </body>
    </html>
    ```

    - JSESSION : session key
        - 첫 접속 시 발급
        - 이후의 통신 시 session key 값으로 connection

## XMl 설정값 load-on-startup

- load-on-startup : 0 이상일 경우 서버 구동 당시 초기화 후 메모리 로드
    - servlet 특성상 미리 로드하지 않고 처음 접근 시 생성 후 메모리 로드 → 해당 과정 생략
    - 여러 servlet 에게 설정 시 우선순위대로 초기화 진행(1이 높은 우선순위)
    - 음수일 경우 무조건 요청 후 생성 및 메모리 로드

    ```xml
    <servlet>
    	<servlet-name>helloServlet</servlet-name>
    	<servlet-class>basic.HelloServlet</servlet-class>
    	<load-on-startup>10</load-on-startup>
    </servlet>
    
    <servlet-mapping>
    	<servlet-name>helloServlet</servlet-name>
    	<url-pattern>/helloServlet</url-pattern>
    </servlet-mapping>
    
    <servlet>
    	<servlet-name>helloServlet2</servlet-name>
    	<servlet-class>basic.HelloServlet2</servlet-class>
    	<load-on-startup>1</load-on-startup>
    </servlet>
    
    <servlet-mapping>
    	<servlet-name>helloServlet2</servlet-name>
    	<url-pattern>/helloServlet2</url-pattern>
    </servlet-mapping>
    ```

  ![](docs/1.png)


## JSP 를 이용하여 화면에 결과 출력

### 1. 내장 객체 JspWriter out 이용

- JSP 에서는 자바를 몰라도 브라우저에 출력할 수 있도록 내장 객체인 out 을 미리 선언해두었음

```html
<%@ page import="java.util.Calendar" %>
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<%
		Calendar cal = Calendar.getInstance();
		
		int year = cal.get(Calendar.YEAR);
		int month = cal.get(Calendar.MONTH) + 1;
		
		// JspWriter out;
		out.println(year + ", " + month);
		...
	%>
</body>
</html>
```

### 2. JSP 문법 이용

- `<%=  %>`

```html
<%@ page import="java.util.Calendar" %>
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<%
		Calendar cal = Calendar.getInstance();
		
		int year = cal.get(Calendar.YEAR);
		int month = cal.get(Calendar.MONTH) + 1;
	%>
	<table>
		<caption><%= year + "," + month %></caption>
		<caption><%= year %>,<%= month %></caption>
	</table>
</body>
</html>
```
