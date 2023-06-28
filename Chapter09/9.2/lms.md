## GROUP BY 처리
인덱스 스캔 방식에 따라 처리 방식이 두 가지로 나뉜다.

### 타이트 인덱스 스캔
- 드라이빙 테이블에 속한 컬럼만 이용해 인덱스를 차례로 읽으며 조인한다.
- 이미 정렬된 인덱스를 읽으므로 추가 정렬 작업이 필요하지 않고, Extra 컬럼에 별도 코멘트가 표시되지 않는다.

### 루스 인덱스 스캔
- where절에서 테이블 인덱스의 첫 번째 인덱스를 스캔하지 않아도 group by 절에 첫 번째 인덱스가 있다면, 인덱스 레인지 스캔이 가능하다.
    - 실행 순서
    1. 첫 번째 인덱스의 유니크(그룹 키)값을 찾는다.
    2. group by 절과 where 절의 조건으로 인덱스를 검색하는 것과 흡사하게 동작
    3. 인덱스에서 첫 번째 인덱스의 다음 유니크 값을 찾는다.
    4. 3번에서 결과가 더 없으면 종료, 있으면 2번으로 돌아간다.

- 루스 인덱스 방식은 단일 테이블에 대해 수행되는 group by 처리에만 사용 가능
- 인덱스 레인지 스캔은 유니크 값이 많을수록, 루스 인덱스 스캔은 유니크 값이 적을수록 성능이 향상

### 임시 테이블 사용 GROUP BY
- group by 절이 인덱스를 사용하지 못할 때 임시 테이블을 사용
    ```
    # 예시 sql
    select e.last_name, AVG(s.salary)
    from emplyees e, salaries s
    where s.emp_no=e.emp_no
    group by e.last_name;
    ```
- MySQL 8.0 이전에는 group by 절을 사용하면 암묵적으로 order by 절도 자동 적용되었는데, 8.0부터는 별도 적용해야 한다.

### SELECT DISTINCT
- select 절에 사용되는 distinct는 조회하는 모든 select 절을 조합한 유니크한 레코드를 가져온다.
    ```
    # 두 sql은 같은 동작을 한다.
    select distinct first_name, last_name from employees;
    select distinct (first_name), last_name from employees;
    ```