## 다중 칼럼(Multi-column) 인덱스
- 2개 이상의 칼럼이 연결됐다고 해서 "Concatenated Index"라고도 한다.
- 루트 노드와 리프 노드는 항상 존재한다.
- 인덱스의 두 번째 칼럼은 첫 번째 칼럼에 의존해서 정렬돼 있다.
- 다중 칼럼 인덱스에서는 인덱스 내에서 각 칼럼의 위치가 상당히 중요하다.

## B-Tree 인덱스의 정렬 및 스캔방향
- 인덱스의 키 값은 항상 오름차순이거나 내림차순으로 정렬되어 저장된다.
- 인덱스를 어느 방향으로 읽을지는 쿼리에 따라 옵티마이저가 실시간으로 만들어내는 실행 계획에 따라 결정된다.

## 인덱스의 정렬
- MySQL 5.7 버전까지는 칼럼 단위로 정렬 순서를 혼합해서 인덱스를 생성할 수 없었다.
- 이러한 문제점을 해결하기 위해 숫자 칼럼의 경우 -1을 곱합 값을 저장해 우회방법으로 했었다.

## 인덱스 스캔 방향
```
SELECT * FROM employees
  OREDER BY first_name DESC
  LIMIT 1;
```
- 인덱스는 항상 오름차순으로만 정렬돼 있지만 인덱스를 최솟값부터 읽으면 오름차순으로 값을 가져올 수 있다.
- 쿼리의 ORDER BY 처리나 MIN() 또는 MAX() 함수 등의 최적화가 필요한 경우에도 MySQL 옵티마이저는 인덱스의 읽기 방향을 전환해서 사용하도록 실행계획을 만들어 낸다.

## 내림차순 인덱스
- 오름차순 인덱스(Ascending index): 작은 값의 인덱스 키가 B-Tree의 왼쪽으로 정렬된 인덱스
- 내림차순 인덱스(Descending index): 큰 값의 인덱스 키가 B-Tree의 왼쪽으로 정렬된 인덱스
- 인덱스 정순 인덱스(Forward index scan): 인덱스 키의 크고 작음에 관계없이 인덱스 리프 노드의 왼쪽 페이지부터 오른쪽으로 스캔
- 인덱스 역순 인덱스(Backward index scan): 인덱스 키의 크고 작음에 관계없이 인덱스 리프 노드의 오른쪽 페이지부터 왼쪽으로 스캔
- 인덱스 역순 스캔이 인덱스 정순 스캔에 비해 느릴수 밖에 없는 이유는 두가지이다.
  - 페이지 잠금이 인덱스 정순 스캔에 적합한 구조
  - 페이지 내에서 인덱스 레코드가 단방향으로만 연결된 구조

- 쿼리에서 자주 사용되는 정렬 순서대로 인덱스를 생성하는 것이 잠금 병목 형상을 완화하는데 도움이 된다.

## 인덱스의 가용성
- B-Tree 인덱스의 특징은 왼쪽 값에 기준해서 (Left-most) 오른쪽 값이 정렬돼 있다는 것이다.
- 여기서 왼쪽이란 하나의 칼럼 내에서뿐만 아니라 다중 칼럼 인덱스의 칼럼에 대해서도 함께 적용된다.

## 가용성과 효율성 판단
- B-Tree 인덱스는 칼럼의 값을 변형하지 않고, 원래의 값을 이용해 인덱싱하는 알고리즘 이다.