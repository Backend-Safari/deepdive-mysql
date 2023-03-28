### B-Tree 인덱스 사용에 영향을 미치는 요소
#### 인덱스 키 값의 크기
- 디스크에 데이터를 저장하는 최소 단위를 페이지 또는 블록이라 하며, 디스크의 모든 읽기, 쓰기 작업의 최소 작업 단위이다.
- InnoDB 스토리지 엔진의 버퍼 풀에서 데이터를 버퍼링하는 기본 단위이기도 하다.
- 인덱스 키 값이 커지면 한 페이지에 저장할 수 있는 키 값이 감소한다.
    - 키 값에 따라 디스크 읽기 횟수가 변동하고, 많을수록 속도가 느려진다.

#### B-Tree 깊이
- 키 값이 커지면 하나의 인덱스 페이지에 담을 수 있는 인덱스 키 값의 개수가 적어지고, 같은 레코드 건수라 해도 B-Tree의 깊이가 깊어져서 디스크 읽기가 더 필요해진다.

#### 선택도(기수성)
- 인덱스에서 유니크한 값의 수를 선택도 또는 기수성 이라고 한다.
- 기수성이 클수록 검색할 대상이 줄어드므로 속도가 빨라진다.

#### 읽어야하는 레코드 건수
- DBMS의 옵티마이저는 인덱스를 통한 레코드 읽기가 테이블의 레코드 읽기보다 4~5배 정도 비용이 드는 것으로 예측한다.
    - 인덱스를 통해 읽어야 할 레코드 건수가 테이블의 20~25%를 넘으면 테이블 전체를 읽는 것이 효율적이다.

### B-Tree 인덱스를 통한 데이터 읽기
- 효율적인 인덱스 사용 판단을 위해 MySQL이 어떻게 인덱스를 이용해서 레코드를 읽는지 알아보자.

#### 인덱스 레인지 스캔
- 루트 노드, 브랜치 노드, 리프 노드를 거쳐 필요한 레코드의 시작 지점을 찾고, 오름 또는 내림차순으로 스캔하여 쿼리셋을 반환한다.
- 인덱스 리프 노드에서 조건에 일치하는 건들은 데이터 파일에서 레코드를 읽어오는데, 레코드 한 건 마다 I/O가 한 번씩 발생한다.
    - 그래서 인덱스를 통해 데이터 레코드를 읽는 작업은 비용이 많이 드는 작업으로 분류된다.
- 데이터 스캔 후 인덱스 키와 레코드 주소를 이용해 레코드가 지정된 페이지를 가져오는데, 쿼리가 필요로 하는 데이터에 따라 생략할 수 있다. 이를 커버링 인덱스라고 한다.

#### 인덱스 풀 스캔
- 인덱스 레인지 스캔과 달리, 인덱스의 처음부터 끝까지 읽는 방식을 인덱스 풀 스캔이라고 한다.
- 인덱스에 포함된 컬럼만으로 쿼리를 처리할 수 있는 경우 테이블 레코드 전체를 읽는 것보다 효율적이므로 사용한다.

#### 루스 인덱스 스캔
- 앞선 두 방식을 타이트 인덱스 스캔이라 부르고, 루스 인덱스 스캔은 이름 그대로 조건을 만족하는 일부 인덱스만 스캔하는 방식이다.
- 일반적으로 GROUP BY 또는 최솟값, 최댓값 함수에 대해 최적화하는 경우 사용된다.

#### 인덱스 스킵 스캔
- group by 절에만 적용할 수 있는 루스 인덱스 스캔과 달리, 인덱스 스킵 스캔은 where절에 사용 가능하도록 용도가 넓어졌다.
    - 기존에는 ix_gender_birthdate 인덱스를 생성하면 gender와 birthdate 컬럼이 있어야 인덱스를 사용할 수 있었지만, MySQL 8.0부터는 gender 컬럼이 없어도 건너뛰고 사용할 수 있게 되었다.
- select 절에만 존재하고 where절에는 없다면 인덱스 풀 스캔을 하게되는데, 인덱스 풀 스캔은 인덱스를 효율적으로 사용하지 못했다고 표현한다.
    - 인덱스 스킵 스캔 환경 변수를 on으로 전환하면 where절에 인덱스가 모두 없어라도 인덱스 레인지 스캔을 사용하여 효율적인 인덱스 사용이 가능하다.
- 8.0에 도입된 기능이라 사용에 아래와 같은 제한이 있다.
    - where 조건절에 없는 인덱스는 유니크한 값의 개수가 적어야함
        - 유니크한 값이 많다면 오히려 효율이 떨어질 수 있다.
    - 쿼리가 인덱스에 존재하는 컬럼만으로 처리 가능해야 한다.(커버링 인덱스)

### 다중 컬럼 인덱스
- 다중 컬럼 인덱스는 2개 이상의 컬럼이 연결된 구조이다.
- 인덱스 내 컬럼의 순서에 따라 이전 컬럼의 정렬값에 의존하므로 컬럼위 위치를 신중하게 정해야 한다.
    - 장고 ORM으로 생각하면 다중 filter를 걸은 효과와 같다.