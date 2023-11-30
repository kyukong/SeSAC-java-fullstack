# [11/30] Spring(파일 업로드, MyBatis CRUD)

## `@RestController`

- `@Controller` + `@ResponseBody`
- `@ResponseBody` : 반환 객체를 json 형태로 응답

## `@ResponseEntity`

- 응답 시 body 내용 뿐만 아니라 헤더와 상태코드 등 그 외의 부가 정보도 포함하는 응답 객체

## 파일 업로드

- Servlet 3.0 전까지는 파일을 업로드 하기 위해 별도의 라이브러리를 필요로 함
- xml 파일을 이용하여 설정하는 경우
    - pom.xml 에 apache commons fileupload 라이브러리 추가

        ```xml
        <!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
        <dependency>
        	<groupId>commons-fileupload</groupId>
        	<artifactId>commons-fileupload</artifactId>
        	<version>1.5</version>
        </dependency>
        ```

    - servlet-context.xml 에 파일 관련 빈 등록 (root-context.xml 도 가능)
        - **파일 관련 객체 생성 시 아이디는 반드시 multipartResolver 로 설정**

        ```xml
        <beans:bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        	<beans:property name="defaultEncoding" value="utf-8" />
        	<beans:property name="maxUploadSize" value="104857560" />
        	<beans:property name="maxUploadSizePerFile" value="2097152" />
        	<beans:property name="uploadTempDir" value="file:/Users/yukong/uplaod/temp" />
        	<beans:property name="maxInMemorySize" value="10485756" />
        </beans:bean>
        ```

- Java 를 이용하여 설정하는 경우
    - WebMvcConfigurer 를 구현한 Java Config 객체 생성
        - CommonsMultipartResolver
        - `@Bean` 의 id 를 지정하여 설정

## Spring Core

- Spring MVC 와 외부 라이브러리를 연결하는 중간 단계
- POJO 영역
- 스프링의 의존성 주입을 이용해서 객체 간의 연관 구조를 연결

## MyBatis 를 이용한 CRUD

### insert

- sequence 를 직접 호출하여 insert
    - 하나의 쿼리문으로 insert 처리 가능
    - 결과값의 id 는 null 로 저장되어 있음

    ```xml
    <insert id="insert">
    	insert into tbl_board (bno, title, content, writer)
    	values (seq_tbl_board.nextval, #{title}, #{content}, #{writer})
    </insert>
    ```

- sequence 쿼리문을 별도로 호출하는 insert
    - sequence 를 먼저 호출하므로 두 번의 쿼리문 실행됨
    - 결과값의 id 는 이전에 호출한 sequence 값으로 채워져 있음

    ```xml
    <insert id="insertSelectKey">
    	<selectKey keyProperty="bno" order="BEFORE"
    		resultType="long">
    		select seq_tbl_board.nextval from dual
    	</selectKey>
    	insert into tbl_board(bno, title, content, writer)
    	values (#{bno}, #{title}, #{content}, #{writer})	
    </insert>
    ```

- 호출 코드

    ```java
    public void insert(BoardVO board);
    	
    public void insertSelectKey(BoardVO board);
    ```


### select

- 파라미터 값은 인터페이스의 메서드 파라미터로 정의되어 있으므로 생략 가능
- resultType 객체의 setter 메서드를 호출하여 필드 정보 저장
    - 테이블의 컬럼과 동일한 이름의 setter 찾아 호출
    - column1 → setColumn1

    ```xml
    <select id="read" resultType="BoardVO">
    	select * from tbl_board where bno = #{bno}
    </select>
    ```

- 호출 코드

    ```java
    public BoardVO read(Long bno);
    ```


### delete

- 삭제된 row 개수 반환
- 삭제된 데이터가 없을 경우 0 반환

    ```xml
    <delete id="delete">
    	delete from tbl_board where bno = #{bno}
    </delete>
    ```

- 호출 코드

    ```java
    public int delete(Long bno);
    ```


### update

- 파라미터로 입력한 객체의 getter 를 호출하여 쿼리문에 값 저장
- 반환값으로 수정된 컬럼의 개수 응답

    ```xml
    <update id="update">
    	update tbl_board
    		set title = #{title},
    		    content = #{content},
    		    writer = #{writer},
    		    updateDate = sysdate
    	where bno = #{bno}
    </update>
    ```

- 호출 코드

    ```java
    public int update(BoardVO board);
    ```
