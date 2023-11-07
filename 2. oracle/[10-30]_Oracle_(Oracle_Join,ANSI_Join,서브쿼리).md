# [10/30] Oracle (Oracle Join, ANSI Join, 서브쿼리)

<details>
<summary>Quiz</summary>
<div>

- Q1. tester1 계정에서 부서별 연봉 합계 구하기

```sql
select job, 
          sum(decode(deptno, '10', sal)) as d10,
          sum(decode(deptno, '20', sal)) as d20,
          sum(decode(deptno, '30', sal)) as d30,
          sum(sal)
from emp
group by job
;
```

- Q2. ace 계정에서 부서별 연봉 합계 구하기

```sql
select job_id,
                sum(decode(department_id, 10, salary)) as d10,
                sum(decode(department_id, 20, salary)) as d20,
                sum(decode(department_id, 30, salary)) as d30,
                sum(decode(department_id, 40, salary)) as d40,
                sum(decode(department_id, 50, salary)) as d50,
                sum(decode(department_id, 60, salary)) as d60,
                sum(decode(department_id, 70, salary)) as d70,
                sum(decode(department_id, 80, salary)) as d80,
                sum(decode(department_id, 90, salary)) as d90,
                sum(decode(department_id, 100, salary)) as d100,
                sum(decode(department_id, 110, salary)) as d110,
                sum(salary)
from employees
group by job_id
;
```
</div>
</details>

## Join

- 조인 시 테이블 별칭 붙일 때 as 사용하면 에러 발생

## Oracle Join

- ANSI Join 에 비해 간결함

### Cross Join (Cartesian Join)

- 특별한 키워드 없이 select 문의 from 절에 여러 테이블을 콤마로 연결해서 연속해서 기술하는 것
- 조건없이 조회했기 때문에 행의 데이터가 복사됨

```sql
select e.ename, e.deptno, d.deptno, d.dname
from emp e, dept d
order by ename
;
```

### Equi Join

- 조인 대상이 되는 두 테이블에 공통으로 존재하는 컬럼의 값이 일치되는 행을 연결하여 결과를 생성하는 조인 방법 (=)

```sql
select e.ename, d.dname
from emp e, dept d
where e.deptno = d.deptno
order by ename
;
```

### Non Equi Join

- 조인 조건 설정 시 (=) 가 아닌 다른 조건을 거는 조인 방법

```sql
select e.ename as 사원명,
            e.sal as 급여,
            s.grade as 등급
from emp e, salgrade s
where s.losal <= e.sal and e.sal <= s.hisal
;

select e.ename as 사원명,
            e.sal as 급여,
            s.grade as 등급
from emp e, salgrade s
where e.sal between s.losal and s.hisal
;
```

### Outer Join

- 두 테이블에서 온전히 일치하지 않는 값도 포함하여 읽어오고 싶을 때 사용하는 조인 조건
    - ex) null 을 포함하는 조인
- Left Outer Join

    ```sql
    select e.ename, d.dname
    from emp e, dept d
    where e.deptno = d.deptno(+)
    order by ename
    ;
    ```

    - d 테이블에 없는 정보도 포함하여 읽고 싶은 경우
    - from 절과 where 절의 순서는 바꾸어도 되나 위와 같은 순서를 권장
- Right Outer Join

    ```sql
    select e.last_name, d.department_name
    from employees e, departments d
    where e.employee_id(+) = d.manager_id
    ;
    ```

    - e 테이블에 없는 정보도 포함하여 읽고 싶은 경우 (null 등)
    - from 절과 where 절의 순서는 바꾸어도 되나 위와 같은 순서를 권장
- Full Outer Join
    - Oracle Join 에서는 Full Outer Join 을 지원하지 않음
    - Full Outer Join 구현 시 Left Outer Join 과 Right Outer Join 을 Union 하는 등으로 구현해야 함

### Self Join

- 자기가 자기 자신의 테이블과 조인하는 것

```sql
select e.ename as 사원이름, m.ename as 직속상관이름
from emp e, emp m
where e.mgr = m.empno(+)
;

select e.last_name as 사원명, m.last_name as 상사명
from employees e, employees m
where e.manager_id = m.employee_id(+)
;
```

### Threeways Join

- 3개의 테이블 조인

```sql
-- threeways join (oracle)
select e.ename as 사원명, 
            d.dname as 부서명,
            s.grade as 등급
from emp e, dept d, salgrade s
where e.deptno = d.deptno(+)
and e.sal between s.losal and s.hisal
;

-- threeways join (oracle)
select e.last_name as 사원명,
            d.department_name as 부서명,
            j.grade_level as 등급
from employees e, departments d, job_grades j
where e.department_id = d.department_id(+)
    and e.salary between j.lowest_sal and j.highest_sal
order by e.salary, e.last_name
;
```

## ANSI Join

- Oracle Join 에 비해 명확함 → 새로 작성하는 Join 문의 경우 ANSI Join 으로 작성할 것

### Cross Join

- Oracle Join 의 Cross Join 과 동일함
- 단순 콤마로 구분하는 Oracle Join 보다 조건이 원래 없는 Cross Join 을 사용하고자 하는 의도가 명확히 드러남

```sql
select e.ename, e.deptno, d.deptno, d.dname
from emp e cross join dept d
order by ename
;
```

### Natural Join

- 두 테이블의 조인 조건을 자동으로 처리해줌
- 두 테이블간의 컬럼명이 동일할 때 조인 조건으로 사용
- 즉, 동일 컬럼명이 여러 개 있을 경우 조인 조건이 잘못 설정될 수 있음 → 지양할 것

```sql
select ename, dname
from emp natural join dept
;

select last_name, department_name
from employees natural join departments
;
-> employees 와 departments 테이블에 중복 컬럼명이 두 개 이상이라 결과가 예상과는 다르게 동작
```

### Join On

- 일반적인 SQL Join 문과 동일
- Oracle Join 과는 다르게 ANSI Join 에서는 Full Outer Join 이 있음
- join == **inner join**
- left outer join == **left join** / right outer join == **right join** / full outer join == **full join**

```sql
-- equi join
select e.ename, d.dname
from emp e inner join dept d
on e.deptno = d.deptno
order by ename
;

-- non equi join
select e.ename as 사원명,
            e.sal as 급여,
            s.grade as 등급
from emp e inner join salgrade s
on s.losal <= e.sal and e.sal <= s.hisal
;

select e.ename as 사원명,
            e.sal as 급여,
            s.grade as 등급
from emp e inner join salgrade s
on e.sal between s.losal and s.hisal
;

-- left outer join
select e.ename, d.dname
from emp e left join dept d
on e.deptno = d.deptno
order by ename
;

-- right outer join
select e.last_name, d.department_name
from employees e right join departments d
on e.employee_id = d.manager_id
;

-- full outer join
select e.ename, d.dname
from emp e full join dept d
on e.deptno = d.deptno
order by ename
;

-- self join
select e.ename as 사원이름, m.ename as 직속상관이름
from emp e left join emp m
on e.mgr = m.empno
;
```

### Threeways join

- 3개의 테이블 조인

```sql
-- threeways join (ansi)
select e.ename as 사원명,
            d.dname as 부서명,
            e.sal as 급여,
            s.grade as 등급
from emp e 
left join dept d
on e.deptno = d.deptno
inner join salgrade s
on e.sal between s.losal and s.hisal
;

-- threeways join (ansi)
select e.last_name as 사원명,
            d.department_name as 부서명,
            e.salary as 급여,
            j.grade_level as 등급
from employees e
left join departments d
on e.department_id = d.department_id
inner join job_grades j
on e.salary between j.lowest_sal and j.highest_sal
order by e.salary, e.last_name
;
```

## 서브쿼리

- 하나의 select 문장에서 그 문장 안에 포함된 또 하나의 select 문장

## 다중 행 서브 쿼리

- in : 메인 쿼리의 비교 조건(’=’ 연산자로 비교할 경우)이 서브 커리의 결과 중에서 하나라도 일치하면 참
    - 급여가 500 을 초과하는 사원과 같은 부서에 근무하는 사원 구하기

        ```sql
        select ename, sal, deptno
        from emp
        where deptno in (select distinct deptno
                          from emp
                          where sal > 500);
        ```

- any, some : 메인 쿼리의 비교 조건이 서브 쿼리의 검색 결과와 하나 이상이 일치하면 참
    - 30번 부서의 최소 급여보다 많은 급여를 받는 사원 출력하기

        ```sql
        select ename, sal
        from emp
        where sal > any (select sal
                          from emp
                          where deptno = 30);
        ```

- all : 메인 쿼리의 비교 조건이 서브 쿼리의 검색 결과와 모든 값이 일치하면 참
    - 30번 부서의 최대 급여보다 많은 급여를 받는 사원 출력하기

        ```sql
        select ename, sal
        from emp
        where sal > all (select sal
                          from emp
                          where deptno = 30);
        ```

- exists : 메인 쿼리의 비교 조건이 서브 쿼리의 결과 중에서 만족하는 값이 하나라도 존재하면 참
  - 서브 쿼리의 결과 값이 참이 나오면 메인 쿼리의 결과 값 리턴
  - 서브 쿼리의 결과 값이 존재하지 않는다면 메인 쿼리 값 리턴하지 않음

      ```sql
      select *
      from dept
      where exists (select *
                    from emp
                    where deptno = 10);
      ```

  - exists 연산자는 서브 쿼리에서 메인 쿼리의 정보를 사용할 수 있음 (상호연관 쿼리)

      ```sql
      select *
      from dept
      where exists (select *
                    from emp
                    where emp.deptno = dept.deptno);
      ```

      - 상호 연관 쿼리는 서브쿼리 → 메인쿼리 → 서브쿼리의 순으로 동작이 수행되기 때문에 성능을 저하시킴
