# [10/31] Oracle (DDL, 데이터 딕셔너리, DML, TCL, 서브쿼리를 이용한 DML 사용, 제약 조건)

## CREATE TABLE

```sql
create table emp01 (
    empno number(4),
    ename varchar2(14),
    sal number(7, 3)
);
```

## ALTER TABLE

### 컬럼 추가

```sql
alter table emp01
add(birth date);
```

### 컬럼 변경

```sql
alter table emp01
modify ename varchar2(30);
```

- 기존 컬럼에 데이터가 없는 경우 컬럼 타입이나 크기 변경이 자유로움
- 기존 데이터가 존재하는 경우 CHAR 와 VARCHAR2 만 허용
- 기존 데이터가 존재하는 경우 변경한 컬럼의 크기가 저장된 데이터의 크기보다 같거나 큰 경우만 가능

### 컬럼 제거

```sql
alter table emp01
drop column ename;
```

- 2개 이상의 컬럼이 존재하는 테이블에서만 가능
- 한 번에 하나의 컬럼만 삭제 가능
- 삭제된 컬럼은 복구할 수 없음

### SET UNUSED

```sql
alter table emp01
set unused (empno);

alter table emp01
drop unused columns;
```

- 삭제 예정인 컬럼을 미리 사용하지 않도록 명시
- 실제로 테이블에 해당 컬럼이 삭제되지는 않음 → select 가능
    - 컬럼 삭제의 경우 연관되어 있는 여러 처리를 해주어야 하기 때문에 시간이 걸리지만, unused 는 비교적 빠르게 처리
- describe 문으로도 조회되지 않음

## RENAME

```sql
rename emp01 to emp02;
```

## DROP TABLE AND TRUNCATE TABLE

```sql
drop table emp02;

flashback table emp02 to before drop; -> 복구
```

- drop table : 테이블과 테이블의 데이터 모두 제거 (테이블 구조 제거)

```sql
truncate table dept01;
```

- truncate table : 테이블의 데이터만 제거

## 데이터 딕셔너리

- 사용자와 데이터베이스 자원을 효율적으로 관리하기 위한 다양한 정보를 저장하는 시스템 테이블의 집합
- 오라클 서버는 데이터베이스의 이름이나 생성 시각, 사용자 권한 및 데이터의 변경 사항을 반영하기 위해 지속적으로 관리함
- 데이터베이스 서버에 의해 자동으로 갱신되는 테이블
- 읽기 전용 뷰 형태

### USER_ 데이터 딕셔너리

- 현재 사용자와 관련된 데이터 딕셔너리
- 자신이 생성한 테이블, 인덱스, 뷰, 동의어 등의 객체나 해당 사용자에게 부여된 권한 정보를 제공

### ALL_ 데이터 딕셔너리

- 전체 사용자와 관련된 데이터 딕셔너리
- 사용자가 접근할 수 있는 모든 객체에 대한 정보를 조회
- 조회중인 객체가 누구의 소유인지를 확인하도록 하기 위해 OWNER 컬럼을 제공

### DBA_ 데이터 딕셔너리

- DBA 나 시스템 권한을 가진 사용자만 접근 가능

## INSERT

- insert into table_name (column1, column2, …) values (value1, value2, …);
- insert into table_name values (value1, value2, …); ⇒ 모든 컬럼의 값 삽입 필수
- 되도록 테이블명 뒤에 컬럼 명시 할 것
    - insert 시 당장 입력하지 않아도 되는 값을 생략할 수 있음
- 삽입할 컬럼의 데이터 타입이 문자 (CHAR, VARCHAR2)와 날짜(DATE)일 경우에는 반드시 작은 따옴표(’’)를 사용
- 해당 컬럼에 null 을 허용하며 default 값이 없을 경우 값을 입력하지 않으면 null 삽입
- null 값을 갖는 컬럼에 null 대신 공백문자(’’)를 삽입하면 null 저장
    - 프로그램과 단순 쿼리문 실행의 결과는 다를 수 있음

## UPDATE

```sql
update dept01
set dname = '생산부'
where deptno = 10;

update dept01
set dname = '생산부2', loc = '부산'
where deptno = 40;
```

## DELETE

```sql
delete [from] dept01
where deptno = 10;
```

## 트랜잭션 관리

- 데이터 처리에서 논리적인 하나의 작업 단위
- commit : 트랜잭션 내 명령어 목록 실행
- rollback : 트랜잭션 내 명령어 목록 취소
    - rollback to savepoint {savepoint 이름}
    - 호출한 savepoint 이후의 지점으로는 돌아갈 수 없음
- savepoint : 트랜잭션 중간 저장
- DDL 혹은 DCL 문이 실행되는 경우에는 자동으로 commit 되고, 오라클이 종료되면 자동으로 rollback
- 트랜잭션이 종료됨과 동시에 새로운 트랜잭션 시작

```sql
select * from emp02;
savepoint a;
delete emp02 where empno = (select max(empno) from emp02);

savepoint b;
delete emp02 where empno = (select max(empno) from emp02);

savepoint c;
delete emp02 where empno = (select max(empno) from emp02);

rollback to c;
rollback to a;
```

## 서브 쿼리로 테이블 생성하기

```sql
create table emp02
as
select * from emp;

desc emp02;

create table emp03
as
select empno, ename from emp;

desc emp03;
```

```sql
Table EMP02이(가) 생성되었습니다.

이름       널? 유형           
-------- -- ------------ 
EMPNO       NUMBER(4)    
ENAME       VARCHAR2(10) 
JOB         VARCHAR2(9)  
MGR         NUMBER(4)    
HIREDATE    DATE         
SAL         NUMBER(7,2)  
COMM        NUMBER(7,2)  
DEPTNO      NUMBER(2)    

Table EMP03이(가) 생성되었습니다.

이름    널? 유형           
----- -- ------------ 
EMPNO    NUMBER(4)    
ENAME    VARCHAR2(10)
```

## 테이블의 구조만 복사하기

- 테이블 구조만 복사하는 명령은 따로 없음
- 서브쿼리의 where 조건을 항상 false 가 나오도록 설정해서 값 저장 X

```sql
create table emp05
as 
select * from emp where 1 = 0;

create table emp05
as 
select * from emp where 1 <> 1;
```

## 서브 쿼리로 데이터를 삽입하기

- insert into 의 values 절 대신 서브 쿼리 사용

```sql
create table dept01
as
select * from dept where 1 <> 1;

insert into dept01
select * from dept;
```

## 서브 쿼리를 이용한 데이터 변경

- update 문의 set 절 대신 서브 쿼리 사용

```sql
10	경리부	서울
20	인사부	인천
30	영업부	용인
40	전산부	수원
```

```sql
update dept01
set loc = (select loc
          from dept01
          where deptno = 40)
where deptno = 20;
```

```sql
10	경리부	서울
20	인사부	수원
30	영업부	용인
40	전산부	수원
```

## 서브 쿼리를 이용한 데이터 삭제

- delete 문의 where 절에 서브 쿼리 사용

```sql
1015	소민이	사원	1010	23/10/23	500		
1001	김사랑	사원	1013	07/03/01	300		20
1002	한예슬	대리	1005	07/04/02	250	80	30
1003	오지호	과장	1005	05/02/10	500	100	30
1004	이병헌	부장	1008	03/09/02	600		20
1005	신동협	과장	1005	05/04/07	450	200	30
1006	장동건	부장	1008	03/10/09	480		30
1007	이문세	부장	1008	04/01/08	520		10
1008	감우성	차장	1003	04/03/08	500	0	30
1009	안성기	사장		96/10/04	1000		20
1010	이병헌	과장	1003	05/04/07	500		10
1011	조향기	사원	1007	07/03/01	280		30
1012	강혜정	사원	1006	07/08/09	300		20
1013	박중훈	부장	1003	02/10/09	560		20
1014	조인성	사원	1006	07/11/09	250		10
```

```sql
delete from emp02
where deptno = (select deptno
                from dept
                where dname = '영업부');
```

```sql
1015	소민이	사원	1010	23/10/23	500		
1001	김사랑	사원	1013	07/03/01	300		20
1004	이병헌	부장	1008	03/09/02	600		20
1007	이문세	부장	1008	04/01/08	520		10
1009	안성기	사장		96/10/04	1000		20
1010	이병헌	과장	1003	05/04/07	500		10
1012	강혜정	사원	1006	07/08/09	300		20
1013	박중훈	부장	1003	02/10/09	560		20
1014	조인성	사원	1006	07/11/09	250		10
```

## Oracle 에서의 테이블 삭제

- 테이블을 삭제하더라도 내부적으로는 해당 테이블을 이름을 변경하고 다른 곳에서 관리
    - 개발자의 실수를 복구할 수 있도록 따로 관리
    - BIN${테이블명}

    ```sql
    drop table emp03;
    ```

- 다음과 같은 명령어를 사용하여 휴지통에도 저장하지 않고 바로 삭제할 수 있음

    ```sql
    drop table emp03 purge;
    ```

- 휴지통 비우기

    ```sql
    purge recyclebin;
    ```

    - 저장공간이 가득 차거나 시간이 지나면 자동으로 휴지통이 비워짐

## 무결성 제약조건

- 테이블에 부적절한 자료가 입력되는 것을 방지하기 위해 테이블 생성할 때 각 컬럼에 대해 규칙을 정의하는 것
- 무결성 == 정확성
- not null : null 허용 불가
    - not null 제약조건은 유일하게 컬럼 제약조건 → 컬럼 옆에 선언하여 정의
    - 컬럼 제약조건의 경우 임의로 설정하기 어려워 테이블 제약조건인 check 를 사용하는 추세
- unique : 중복된 값 허용 불가
    - null 은 값(value)이 아니므로 unique 체크가 되지 않아 null 허용
    - null 을 허용하고 싶지 않을 경우 not null 을 더하거나, primary key 설정할 것
- primary key : not null + unique
    - 추가 작업을 수행하기 전에 테이블 내 이미 저장된 정보를 모두 조회 후 이미 입력된 정보가 아닐 경우에 추가
- foreign key : 참조되는 테이블에서 컬럼의 값이 존재
- check : 저장 가능한 데이터 값의 범위나 조건을 지정하여 설정한 값만 허용
    - check(column is not null) ⇒ 테이블 단위 제약조건

```sql
select constraint_name, constraint_type, table_name
from user_constraints;
```

```sql
PK_DEPT	P	DEPT
PK_EMP	P	EMP
FK_DEPTNO	R	EMP
SYS_C008258	C	TEST01
SYS_C008259	C	TEST02
CH_MYCHECK	C	TEST03
```

- SYS_C008258 와 SYS_C008259 는 지정되지 않아 임의로 작성된 제약조건 이름
