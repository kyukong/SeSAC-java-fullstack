# [12/1] Spring (실행 계획, 인덱스, 힌트, ROWNUM)

## addFlashAttribute()

- addAttribute 의 경우 redirect 시 다음 페이지로 넘어갈 때 Model 에 데이터를 담기 위해 사용
- addFlashAttribute 는 내부적으로 HttpSession 을 이용해서 처리하기 때문에 일회성 데이터 전달

## XML 의 CDATA 섹션

- XML 에서는 `<` 를 이용하여 태그를 구분
- MyBatis 에서는 XML 또는 인터페이스 및 어노테이션을 사용하여 쿼리문 정의 가능
- 쿼리문에 있는 부등호를 사용하기 위해 CDATA 이용

## 실행 계획 (Execution Plan)

- SQL 이 데이터베이스에 전달되어 어떻게 실행될 것인지에 대한 계획 (절차)
- SQL 문은 다음과 같은 절차를 거쳐 실행됨
    - SQL 파싱
        - SQL 구문에 오류가 있는지 실행 객체에 대한 검증 (테이블, 제약 조건, 권한 등)
    - SQL 최적화
        - SQL 이 실행되는데 필요한 비용 (cost) 산정
        - 최적화된 방법 ⇒ **실행 계획**
        - 데이터의 양이나 제약 조건 등의 여러 상황에 따라 실행 계획이 달라짐
    - SQL 실행
        - 세워진 실행 계획을 통해서 메모리 상에서 데이터를 읽거나 물리적인 공간에서 데이터를 로딩하는 등의 작업 수행
- PK 컬럼을 기준으로 order by 한 경우의 실행 계획

    ```sql
    select * from tbl_board order by bno desc;
    ```

    1. PK_BOARD → FULL SCAN DESCENDING
        - 테이블은 물리적으로 PK 를 기준으로 정렬되어 있음
        - desc 정렬을 요청했으므로 descenging 으로 PK tree 를 순회
    2. TBL_BOARD → BY INDEX ROWID
        - PK 인 INDEX 조회

## 인덱스 (Index)

- 조회한 데이터를 정렬할 때 데이터의 수가 많으면 그 만큼 부하가 발생함
- 수만개 이상의 데이터를 처리해야 하는 경우 최대한 정렬을 안할 수 있는 방법에 대해 고민해야 함 ⇒ **인덱스**
- 원하는 데이터를 빠르게 조회하기 위해 별도의 디스크에 정렬되어 있는 상태로 저장되어 있는 데이터
- 인덱스는 B-Tree 구조로 되어 있기 때문에 FULL SCAN 하는 것보다 원하는 데이터에 빠르게 접근할 수 있음
- 즉, 정렬되어 있으며 빠르게 접근 가능한 구조라는 장점을 가지고 있음

## 오라클 힌트 (Hint)

- 경우에 따라 최적화된 실행 계획이 아닌 방법으로 처리해야 더 빠르게 처리될 수 있음
- 그럴 땐 select 문에 힌트를 전달하여 실행 계획에 대한 ‘힌트’ 를 줄 수 있음
- 힌트 구문은 에러가 발생하더라도 그대로 실행되므로 실행 계획 확인을 통해 동작 검증 필요
- 예를 들어 board 테이블에 bno 컬럼의 역순으로 모든 데이터를 조회한다고 가정해본다면
    - 힌트를 사용하지 않는 경우
        - 모든 데이터에 대해 order by 를 사용하여 조회 성능이 떨어짐
        - bno 에는 이미 index 가 걸려 있기 때문에 정렬되어 있는데, order by 또한 요청하여 다시한번 정렬이 발생함

        ```sql
        explain plan for
        select *
        from tbl_board
        order by bno desc;
        
        select *
        from table(dbms_xplan.display);
        ```

        ```sql
        Plan hash value: 1400135285
         
        --------------------------------------------------------------------------------------------
        | Id  | Operation                   | Name         | Rows  | Bytes | Cost (%CPU)| Time     |
        --------------------------------------------------------------------------------------------
        |   0 | SELECT STATEMENT            |              |   400K|    31M|  5581   (1)| 00:00:01 |
        |   1 |  TABLE ACCESS BY INDEX ROWID| TBL_BOARD    |   400K|    31M|  5581   (1)| 00:00:01 |
        |   2 |   INDEX FULL SCAN DESCENDING| PK_TBL_BOARD |   400K|       |   755   (1)| 00:00:01 |
        --------------------------------------------------------------------------------------------
        ```

    - 힌트를 사용하는 경우
        - 이미 정렬되어 있는 속성을 이용
        - 정렬되어 있는 인덱스를 거꾸로 탐색하라고 힌트를 주어 order by 를 사용했을 때보다 더 나은 실행 계획이 작성된 것을 확인할 수 있음

        ```sql
        explain plan for
        select /*+INDEX_DESC (tbl_board pk_board) */ *
        from tbl_board;
        
        select *
        from table(dbms_xplan.display);
        ```

        ```sql
        Plan hash value: 215559671
         
        -------------------------------------------------------------------------------
        | Id  | Operation         | Name      | Rows  | Bytes | Cost (%CPU)| Time     |
        -------------------------------------------------------------------------------
        |   0 | SELECT STATEMENT  |           |   400K|    31M|  1334   (1)| 00:00:01 |
        |   1 |  TABLE ACCESS FULL| TBL_BOARD |   400K|    31M|  1334   (1)| 00:00:01 |
        -------------------------------------------------------------------------------
         
        Hint Report (identified by operation id / Query Block Name / Object Alias):
        Total hints for statement: 1 (U - Unused (1))
        ---------------------------------------------------------------------------
         
        "   1 -  SEL$1 / ""TBL_BOARD""@""SEL$1"""
                 U -  INDEX_DESC (tbl_board pk_board) / index specified in the hint doesn't exist
        ```


## FULL 힌트

- select 시 테이블 전체를 스캔

```sql
explain plan for
select /*+ FULL(tbl_board) */ *
from tbl_board
order by bno desc;

select *
from table(dbms_xplan.display);
```

```sql
Plan hash value: 3456581270
 
----------------------------------------------------------------------------------------
| Id  | Operation          | Name      | Rows  | Bytes |TempSpc| Cost (%CPU)| Time     |
----------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT   |           |   400K|    31M|       |  8976   (1)| 00:00:01 |
|   1 |  SORT ORDER BY     |           |   400K|    31M|    38M|  8976   (1)| 00:00:01 |
|   2 |   TABLE ACCESS FULL| TBL_BOARD |   400K|    31M|       |  1334   (1)| 00:00:01 |
----------------------------------------------------------------------------------------
```

- 위와 같이 설정한다면 테이블을 읽어올 때 FULL SCAN 후 order by 를 처리함

## INDEX_ASC, INDEX_DESC 힌트

- 미리 생성된 인덱스를 기준으로 스캔
- **주로 order by 를 위해서 사용함**
- 미리 정렬되어 있는 인덱스의 속성을 사용하여 full scan 을 통해 정렬과 같은 결과를 조회

```sql
explain plan for
select /*+ INDEX_ASC(tbl_board pk_board) */ *
from tbl_board
where bno > 0;

select *
from table(dbms_xplan.display);
```

```sql
Plan hash value: 215559671
 
-------------------------------------------------------------------------------
| Id  | Operation         | Name      | Rows  | Bytes | Cost (%CPU)| Time     |
-------------------------------------------------------------------------------
|   0 | SELECT STATEMENT  |           |   400K|    31M|  1334   (1)| 00:00:01 |
|*  1 |  TABLE ACCESS FULL| TBL_BOARD |   400K|    31M|  1334   (1)| 00:00:01 |
-------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
"   1 - filter(""BNO"">0)"
 
Hint Report (identified by operation id / Query Block Name / Object Alias):
Total hints for statement: 1 (U - Unused (1))
---------------------------------------------------------------------------
 
"   1 -  SEL$1 / ""TBL_BOARD""@""SEL$1"""
         U -  INDEX_ASC(tbl_board pk_board) / index specified in the hint doesn't exist
```

## ROWNUM

- SQL 이 실행된 결과에 붙여진 넘버링
- 실제 데이터가 아니라 테이블에서 데이터를 추출한 후에 처리되는 변수
- 상황에 따라서 값이 달라질 수 있음
- 모든 SELECT 문에는 ROWNUM 이라는 변수가 있어 몇번째로 읽어왔는지 알 수 있음
- DB 서버에서 SELECT 후 ORDER BY 등의 처리를 하기 때문에 데이터 조회 이후 가공하면 값이 정렬되지 않을 수 있음

## ROWNUM 을 이용한 페이징 처리 (인라인 뷰)

- SELECT 문 안쪽 FROM 에 다시 SELECT 문을 사용하는 것
- 어떤 결과를 구하는 SELECT 이 있고, 그 결과를 다시 대상으로 삼아서 SELECT 를 하는 것
- 외부의 SELECT 문은 인라인뷰로 작성된 결과를 하나의 테이블처럼 사용 가능 함
- 외부의 SELECT 문 또한 rownum 이 생성되지만 인라인뷰에서 선언한 rn 을 사용했기 때문에 내부의 SELECT 문의 rownum 에 접근 가능함

```sql
select
	bno, title, content
from ( // 인라인뷰
	select /*+INDEX_DESC(tbl_board pk_board) */
		rownum rn, bno, title, content
	from
		tbl_board
	where rownum <= 20)
where rn > 10;
```
