# [10/27] Oracle (집합 연산자, 여러 함수, GROUP BY, HAVING)

## 집합 연산자

- 집합 연산하려는 컬럼의 타입이 일치해야 함 (네이밍은 상관 X)
- union : 중복이 제거된 합집합
- union all : 중복 제거가 안된 합집합
- intersect : 교집합
- minus : 차집합

<details>
<summary>Quiz</summary>
<div>

- Q1. set operator를 이용하여 job ID가 ST_CLERK을 포함하지 않는 부서의 ID를 출력하세요.
- 부서의 전체 집합 **minus** job id 가 st_clerk 을 포함하지 않는 부서의 ID

    ```sql
    select department_id
    from departments
    minus
    select department_id
    from job_history
    where job_id = 'ST_CLERK'
    ;
    ```

- Q2. set operator를 이용하여 부서가 없는 지역의 Country ID와 이름을 출력하세요. (join)
- Q3. 부서가 10, 50 그리고 20의 순서로 부서의 업무리스트를 정렬하고 set operators를 이용하여 job ID와 department ID를 출력하세요.

    ```sql
    select job_id, department_id, 1  as a_dummy
    from employees
    where department_id = 10
    union
    select job_id, department_id, 2 as a_dummy
    from employees
    where department_id = 50
    union
    select job_id, department_id, 3 as a_dummy
    from employees
    where department_id = 20
    order by a_dummy
    ;
    ```

- Q4. 입사후 현재 업무와 같은 업무를 담당한 적이 있는 사원의 employee ID와 job ID를 출력하세요.

    ```sql
    select employee_id, job_id
    from employees
    intersect
    select employee_id, job_id
    from job_history
    order by employee_id
    ;
    ```
</div>
</details>

## 숫자 함수

- abs() : 절대값
- cos() : cosine 함수
- exp() : e 의 n 승
- floor() : 소수점 아래 자름 (버림)
- log() : log 값
- power(m, n) : m 의 n 승
- sign(n) : n < 0 이면 -1, n = 0 이면 0, n > 0 이면 1
- sin() : sine 함수
- tan() : tangent 함수
- round() : 특정 자릿수에서 반올림
    - round(34.5678, 2) → 34.57
    - round(34.5678, -1) → 30
- trunc() : 특정 자릿수에서 버림
    - trunc(34.5678, 2) → 34.56
    - trunc(34.5678, -1) → 30
    - trunc(34.5678) → 34
- mod() : 입력 받은 수를 나눈 나머지 값
    - mod(27, 2) → 1
    - mod(27, 5) → 2
    - mod(27, 7) → 6

## 문자 처리 함수

### 대소문자 변환함수

- upper() : 대문자로 변환
- lower() : 소문자로 변환
- initcap() : 첫글자만 대문자, 나머지 글자는 소문자

### 문자 길이를 구하는 함수

- length() : 문자의 길이 반환 (한글 1byte)

    ```sql
    select length('Oracle'), length('오라클')
    from dual
    ;
    
    -> 6	3
    ```

- lengthb() : 문자의 길이 반환 (한글 3byte) ⇒ 문자셋 utf-8 기준

    ```sql
    select lengthb('Oracle'), lengthb('오라클')
    from dual
    ;
    
    -> 6	9
    ```


### 문자 조작 함수

- concat() : 문자의 값 연결
- substr() : 문자를 잘라 추출 (한글 1byte) → 기준 : 문자
- substrb() : 문자를 잘라 추출 (한글 2byte) → 기준 : byte
    - 영문자는 1byte 로 관리되기 때문에 substr() 과 동일
    - 한글은 3byte 로 관리되기 때문에 다르게 처리
- instr() : 특정 문자의 위치 값을 반환 (한글 1byte)
    - instr(대상, 찾을 글자, 시작 위치 = 1, 몇 번째의 문자 = 1)

    ```sql
    select instr('WELCOME TO ORACLE', 'O')
    from dual;
    
    -> 5
    ```

    ```sql
    select instr('WELCOME TO **O**RACLE', 'O', 6, 2)
    from dual;
    
    -> 12
    ```

- instrb() : 특정 문자의 위치 값을 반환 (한글 2byte)

    ```sql
    select instr('데이터베이스', '이', 4, 1), instrb('데이터베이스', '이', 4, 1)
    from dual;
    
    -> 5  4
    ```

    - 1, 2, 3(데), 4, 5, 6(이), 7, 8, 9(터), …
    - 4byte 이후 처음 등장하는 ‘이’ 는 4~6바이트 사이의 값이므로 4 반환
- lpad(), rpad() : 입력받은 문자열과 기호를 정렬하여 특정 길이의 문자열로 반환
    - lpad : left padding

    ```sql
    select lpad('Oracle', 20, '#')
    from dual;
    
    -> ##############Oracle
    ```

    ```sql
    select rpad('Oracle', 20, '#')
    from dual;
    
    -> Oracle##############
    ```


## 형변환 함수

- to_char() : 날짜형 혹은 숫자형을 **문자형**으로 변환
    - 날짜형을 문자형으로 변환

        <details>
        <summary>형 변환 함수의 날짜 출력 형식</summary>
        <div>
    
            - YYYY : 년도 표현 (4자리)
            - YY : 년도 표현 (2자리)
            - MM : 월을 숫자로 표현
            - MON : 월을 알파벳으로 표현
            - DAY : 요일 표현
            - DY : 요일을 약어로 표현

        </div>
        </details>

        ```sql
        select hiredate, to_char(hiredate, 'MON/DD DAY DY')
        from emp;
        
        -> 
        23/10/23	10월/23 월요일 월
        07/03/01	3월 /01 목요일 목
        07/04/02	4월 /02 월요일 월
        ```

        <details>
        <summary>형 변환 함수의 시간 출력 형식</summary>
        <div>

            - AM 또는 PM : 오전, 오후 시각 표시
            - HH 또는 HH12 : 시간 (1~12)
            - HH24 : 24시간으로 표현 (0~23)
            - MI : 분 표현
            - SS : 초 표현

        </div>
        </details>

        ```sql
        select to_char(sysdate, 'YYYY/MM/DD, AM HH:MI:SS')
        from dual;
        
        -> 2023/10/27, 오전 05:39:23
        ```

    - 숫자형을 문자형으로 변환

        <details>
        <summary>형 변환 함수의 숫자 출력 형식</summary>
        <div>

            - 0 : 자릿수를 나타내며 자릿수가 맞지 않을 경우 0으로 채움
            - 9 : 자릿수를 나타내며 자릿수가 맞지 않아도 채우지 않음
            - L : 각 지역별 통화 기호를 앞에 표시
            - . : 소수점
            - , : 천 단위 자리 구분

        </div>
        </details>

        ```sql
        select to_char(1230000), to_char(1230000, 'L999,999,999')
        from dual;
        
        -> 1230000	          ￦1,230,000
        
        select to_char(123456, '000000000'), to_char(123456, '999,999,999')
        from dual;
        
        -> 000123456	     123,456
        ```

- to_date() : 문자형을 **날짜형**으로 변환
    - DATE 형과 비교할 때에는 DATE 형으로 변환한 후 사용

    ```sql
    select ename, hiredate
    from emp
    where hiredate = to_date('2007/04/02', 'YYYY/MM/DD');
    
    -> 한예슬	07/04/02
    ```

- to_number() : 문자형을 **숫자형**으로 변환

    ```sql
    select to_number('20,000', '99,999') - to_number('10,000', '99,999')
    from dual;
    
    -> 10000
    ```


## 날짜 함수

- sysdate() : 시스템에 저장된 현재 날짜
- months_between() : 두 날짜 사이가 몇 개월인지 반환

    ```sql
    select ename, sysdate as 오늘, 
                to_char(hiredate, 'YYYY/MM/DD') as 입사일,
                trunc(months_between(sysdate, hiredate)) as 근무달수
    from emp;
    
    -> 
    소민이	23/10/27	2023/10/23	0
    김사랑	23/10/27	2007/03/01	199
    한예슬	23/10/27	2007/04/02	198
    
    select trunc(hiredate, 'MONTH')
    from emp;
    
    -> 
    23/10/01
    07/03/01
    07/04/01
    ```

- add_months() : 특정 날짜에 개월 수 더함

    ```sql
    select ename, 
                to_char(hiredate, 'YYYY/MM/DD') as 입사일,
                to_char(add_months(hiredate, 6), 'YYYY/MM/DD') as "입사 6개월 후"
    from emp;
    
    select ename, 
                to_char(hiredate, 'YYYY/MM/DD') as 입사일,
                to_char(hiredate + 6, 'YYYY/MM/DD') as "입사 6개월 후"
    from emp;
    
    -> 
    소민이	2023/10/23	2024/04/23
    김사랑	2007/03/01	2007/09/01
    한예슬	2007/04/02	2007/10/02
    ```

- next_day() : 특정 날짜에 최초로 도래하는 인자로 받은 요일의 날짜 반환

    ```sql
    select to_char(sysdate, 'YYYY/MM/DD') as 오늘,
                to_char(next_day(sysdate, 'WEDNESDAY'), 'YYYY/MM/DD') as 수요일
    from dual;
    ```

- last_day() : 해당 달의 마지막 날짜 반환

    ```sql
    select ename, to_char(hiredate, 'YYYY/MM/DD') as 입사일,
                to_char(last_day(hiredate), 'YYYY/MM/YY') as "마지막 날짜"
    from emp;
    
    -> 
    소민이	2023/10/23	2023/10/23
    김사랑	2007/03/01	2007/03/07
    한예슬	2007/04/02	2007/04/07
    ```

- round() : 인자로 받은 날짜를 특정 기준으로 반올림

    <details>
    <summary>ROUND 함수의 포맷 모델</summary>
    <div>

      - CC, SCC : 4자리 연도의 끝 두 글자를 기준으로 반올림
      - SYYY, YYYY, YEAR, SYEAR, YYY, YY, Y : 년 (7월 1일부터 반올림)
      - DDD, D, J : 일을 기준으로 반올림
      - HH, HH12, HH24 : 시를 기준으로 반올림
      - Q : 한 분기의 두 번째 달의 16일을 기준으로 반올림
      - MONTH, MON, MM, RM : 월 (16일을 기준으로 반올림)
      - DAY, DY, D : 한 주가 시작되는 날짜를 기준으로 반올림
      - MI : 분을 기준으로 반올림

    </div>
    </details>
  
- trunc() : 인자로 받은 날짜를 특정 기준으로 버림

    ```sql
    select to_char(hiredate, 'YYYY/MM/DD') as 입사일, 
                to_char(trunc(hiredate, 'MONTH'), 'YYYY/MM/DD') as 입사일
    from emp;
    
    -> 
    2023/10/23	2023/10/01
    2007/03/01	2007/03/01
    2007/04/02	2007/04/01
    ```


## NVL 함수

- nvl(컬럼, 대체값)
- nvl2(컬럼, null이 아닌 경우 대체값, null인 경우 대체값)

    ```sql
    select ename, sal, comm, nvl2(comm, sal  *12 + comm, sal * 12)
    from emp;
    
    -> 
    소민이	500		6000
    김사랑	300		3600
    한예슬	250	80	3080
    ```

- nullif(첫번째값, 두번째값) : 두 표현식이 동일한 경우 null, 다른 경우 첫번째값 반환

    ```sql
    select nullif('A', 'A'), nullif('A', 'B')
    from dual;
    
    -> (null)  A
    ```

- coalesce(값1, 값2, …, 값n) : 인수 중에서 null 이 아닌 첫 번째 인수를 반환하는 함수

    ```sql
    select ename, sal, comm, coalesce(comm, sal, 0)
    from emp
    order by deptno;
    
    -> 
    조향기	280		280
    한예슬	250	80	80
    감우성	500	0	0
    ```

    - 앞의 모든 값이 모두 null 이면 마지막 값 반환

## DECODE 함수

- 여러 조건에 대해 적절한 값 반환 (Java 의 switch case 문과 유사)

    ```sql
    select ename, deptno, decode(deptno, 10, '경리부',
    	                                    20, '인사과',
    	                                    30, '영업부',
    	                                    40, '전산부') as dname
    from emp;
    
    -> 
    소민이	10	경리부
    김사랑	20	인사과
    한예슬	30	영업부
    ```

<details>
<summary>Quiz</summary>
<div>
  
- Q1. 체육대회에서 사번으로 홀수는 '청군', 짝수는 '백군' 으로 출력

```sql
select ename as 사원명, decode(mod(empno, 2), 1, '청군', 0, '백군') as 팀
from emp
;
```
    
- Q2. 부서에 따른 연봉의 차등 인상

```sql
select ename, deptno, sal as 올해연봉,
    decode(
        deptno, 10, sal * 1.2,
                20, sal * 1.1,
                    sal
    ) as 내년연봉
from emp;
        
-- searched case
select ename, deptno, sal as 올해연봉,
    case deptno when 10 then sal * 1.2
                when 20 then sal * 1.1
                        else sal
    end as 내년연봉
from emp;
    
-- simpled case
select ename, deptno, sal as 올해연봉,
    case when deptno = 10 then sal * 1.2
         when deptno = 20 then sal * 1.1
                          else sal
    end as 내년연봉
from emp;
    
-> 
소민이	10	500	600
김사랑	20	300	330
한예슬	30	250	250
```

- Q3. 연봉 등급 구분

```sql
select ename, sal, 
    decode(
        floor((sal - 1) / 300), 0, 'LOW',
                                1, 'MID', 
                                2, 'HIGH',
                                3, 'TOP'
    ) as grade
from emp;
        
-- searched case
select ename, sal, 
    case ceil(sal / 300) - 1 when 0 then 'LOW'
                             when 1 then 'MID'
                             when 2 then 'HIGH'
                                    else 'TOP'
    end as grade
from emp;
    
-- simpled case
select ename, sal, 
    case when sal <= 300 then 'LOW'
         when sal <= 600 then 'MID'
         when sal <= 900 then 'HIGH'
                         else 'TOP'
    end as grade
from emp;
        
-> 
소민이	500	MID
김사랑	300	LOW
한예슬	250	LOW
```

</div>
</details>

## CASE 함수

- decode 함수 + 다양한 비교 연산자를 이용한 조건 설정 (Java 의 if 문과 유사)

    ```sql
    	select ename, deptno,
        case when deptno = 10 then '경리부'
                when deptno = 20 then '인사과'
                when deptno = 30 then '영업부'
                when deptno = 40 then '전산부'
        end as dname
    from emp;
    
    -> 
    소민이	10	경리부
    김사랑	20	인사과
    한예슬	30	영업부
    ```


## 그룹 함수 (집계 함수)

- sum() : 그룹의 누적 합계
- avg() : 그룹의 평균
- count() : 그룹의 총 개수
- max() : 그룹의 최댓값
- min() : 그룹의 최솟값
- stddev() : 그룹의 표준편차
- variance() : 그룹의 분산
- 집계함수는 두번까지 중첩해서 사용 가능

## GROUP BY

- null 이 포함되어 있을 경우 asc 는 가장 마지막에, desc 는 가장 앞에 위치함
- 그룹 함수는 컬럼의 값이 null 일 경우 제외
    - count(*) 는 row 의 수를 카운트하므로 제외
- select 문에 그룹 함수를 사용하는 경우, 그룹 함수를 적용하지 않은 단순 컬럼은 올 수 없음 → 서브 쿼리
    - 그룹 함수를 적용하지 않을 컬럼을 group by 에 포함할 경우 group by 를 하지 않는 결과와 동일

## HAVING

- group by 한 결과의 조건 제한
- where 는 grouping 하기 전, having 은 grouping 한 후에 실행하므로 having 절에만 작성할 수 있는 조건은 where 절에 작성 불가능 (ex. 집계함수 등)
- where 절의 조건은 having 절에 작성 가능 → 성능 이슈가 있으므로 지양
    - 데이터 선별 후 grouping 하는 것이 적절

## SQL 구문 실행 순서

- from → where → group by or select → having → order by
- where : 조건에 맞는 레코드 선별
- select : 조건에 맞는 컬럼 선별
- group by 를 하면서 메모리 상에 동시에 select 실행
- group by, having, select 는 메모리상에서 동시에 발생하여 동작 순서가 정해져있지 않음
