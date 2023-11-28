# [11/26] Spring (xml 설정 파일, 스프링 빈 등록 및 조회, 의존성 주입, Spring 빈 등록 동작 과정)

## pom.xml vs jar 파일 직접 이동

- 런타임 시에 사용되는 라이브러리는 pom.xml 에 선언
- 컴파일 시에 필요한 라이브러리는 jar 파일 직접 이동

## DispatcherServlet

```jsx
org.springframework.web.servlet.DispatcherServlet
```

## root-context vs servlet-context

- root-context : 모든 경로에서 접근할 수 있는 context 파일
    - 외부에서도 접근 가능
    - 스프링 프레임워크에서 관리하는 객체
- servlet-context : 현재 servlet 에서만 접근할 수 있는 context 파일

## xml 을 이용한 빈 등록 및 조회

- beaninit.xml 파일 생성

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    		<!-- 빈 등록 -->
        <bean id="tv" class="net.developia.spring01.di101.SamsungTV" />
    </beans>
    ```

- 빈 등록 시 `<bean>` 태그를 활용하여 빈에 접근할 때 사용하는 아이디와 클래스를 직접 매핑
    - 속성으로 `id` 와 `name` 모두 가능
- Spring 에서 관리하는 ApplicationContext 를 읽어와 getBean 으로 접근

    ```java
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.GenericXmlApplicationContext;
    
    public class TVUser {
    
        public static void main(String[] args) {
    
            ApplicationContext context = new GenericXmlApplicationContext(TVUser.class, "beaninit.xml");
    
            TV tv = (TV) context.getBean("tv"); // new SamsungTV();
        }
    }
    ```

    - 빈 등록을 정의한 파일을 기준으로 ApplicationContext 접근

## 어노테이션을 이용한 빈 등록 및 조회

- `@Component` 를 이용하여 등록하고 싶은 빈 선정
    - value 로 빈의 이름 지정
    - value 를 지정하지 않을 경우 첫번째 문자가 소문자인 이름으로 자동 지정
    - 클래스의 이름이 대문자가 연속으로 있을 경우 다른 규칙으로 처리되므로 되도록 직접 지정할 것
        - 대문자 그대로 처리
- ApplicationContext 조회 시 AnnotationApplicationContext 로 base-package 선택
- xml 사용 시 `<component:scan>` 으로 base-package 선택
- 조회 시 설정한 빈 이름으로 접근 가능
    - 클래스 타입으로도 접근 가능

## setter 를 이용하여 의존성 주입

- property 를 정의하여 빈의 필드값을 채워넣을 수 있음
- 생성한 클래스의 setter 메서드를 호출하여 필드 값 저장
- property 시 일반 값이 아닌 빈을 등록할 경우 `ref` 속성 사용

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
        <bean id="messageBean" class="net.developia.spring01.di101e.MessageBeanImpl">
            <property name="name" value="김유빈" />
            <property name="greeting" value="안녕하세요" />
    				<property name="fileOutputter" ref="fileOutputter" />
        </bean>
    </beans>
    ```

    ```java
    public class MessageBeanImpl implements MessageBean {
    
        private String name;
        private String greeting;
    
    		@Override
        public void sayHello() {
            System.out.printf("%s님, %s%n", name, greeting);
            try {
                fileOutputter.outputter(String.format("%s님, %s", name, greeting));
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    
        public void setName(final String name) {
            this.name = name;
        }
    
        public void setGreeting(final String greeting) {
            this.greeting = greeting;
        }
    
    		public void setFileOutputter(final FileOutputter fileOutputter) {
            this.fileOutputter = fileOutputter;
        }
    }
    ```


## 생성자를 이용하여 의존성 주입

- constructor-arg 태그를 이용하여 생성자 주입 정의

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
        <bean id="fileOutputter" class="net.developia.spring01.di102e.FileOutputterImpl">
            <property name="name" value="outputter.txt" />
        </bean>
    
        <bean id="messageBean" class="net.developia.spring01.di102e.MessageBeanImpl">
            <constructor-arg value="김유빈" />
            <constructor-arg value="안녕하세요" />
            <constructor-arg ref="fileOutputter" />
        </bean>
    </beans>
    ```

    ```java
    import java.io.IOException;
    
    public class MessageBeanImpl implements MessageBean {
    
        private String name;
        private String greeting;
        private FileOutputter fileOutputter;
    
        public MessageBeanImpl(final String name, final String greeting, final FileOutputter fileOutputter) {
            this.name = name;
            this.greeting = greeting;
            this.fileOutputter = fileOutputter;
        }
    
        @Override
        public void sayHello() {
            System.out.printf("%s님, %s%n", name, greeting);
            try {
                fileOutputter.outputter(String.format("%s님, %s", name, greeting));
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    ```


## Spring 빈 등록 동작 과정

- 스프링 프레임워크가 시작되면 먼저 스프링이 사용하는 메모리 영역을 만듦 (ApplicationContext)
- 스프링이 객체를 생성하고 관리해야 하는 객체들에 대한 설정 (root-context.xml)
- root-context.xml 에 설정되어 있는 `<context:component-scan>` 태그의 내용을 통해서 설정한 패키지 스캔
- 해당 패키지에 있는 클래스들 중에서 스프링이 사용하는 `@Component` 라는 어노테이션이 존재하는 클래스의 인스턴스 생성
- 등록된 빈을 `@Autowired` 로 호출하는 경우 생성한 빈 객체 주입

## `@Autowired` vs `@Inject`

- `@Autowired` : Spring 의 의존성 주입 어노테이션
- `@Inject` : Java 의 의존성 주입 어노테이션
