## 4-2 InnoDB 스토리리지  엔진  아키텍쳐
- 가장 보편적으로 사용되는 InnoDB에 대해 알아보자.

### 프라이머리 키에 의한 클러스터링
- 기본적으로 PK를 기준으로 클러스터링되어 저장된다.
- PK가 클러스터링 인덱스이기 때문에 레코드 주소 대신 PK를 이용할 경우 더 빨리 처리된다.
- 오라클의 IOT(Index organized table)과 동일한 구조이다.

### 외래 키 지원
- MyISAM과 달리 외래키는 InnoDB에서만 지원한다.
- 외래키는 상호체크를 해야하므로 잠금이 여러 테이블로 전파되고, 그로 인한 데드락 발생에 주의해야 한다.
- 스키마 변경 등 관리가 복잡하다. 작성한 코드의 로직을 정확히 이해하고 사용해야 하므로 서비스의 급한 상황 대처에 불리할 수 있다.
- 외래키 체크를 해제하면 로직에 상관없이 수정할 수 있다.
    - ```mysql> SET foreign+key_checks=ON/OFF;```
    - 물론 부모자식 관계를 해칠 수 있는 것이 아닌, 온오프 시 관계는 일관성을 갖게 수정해야 한다.

### MVCC
- 레코드 레벨의 트랜잭션을 지원하는 DBMS가 제공하는 기능이며, 목적은 잠금을 사용하지 않는 일관된 읽기를 제공하는 데 있다.
- sql 격리 레벨에 따라 select 문은 다른 동작을 한다. 총 4가지 레벨이 존재하며, ~가 있다. InnoDB를 예시도 동작을 알아보자
    - sql 동작 시 InnoDB 버퍼풀, 언두 로그, 데이터 파일의 역할을 보자.
    - read_uncommite의 경우 update를 명령하면 언두 로그에 이전 데이터가 기록되고, 커밋 전일 경우 select문을 동작하면 버퍼 풀을 불러온다. read_commited 이상의 레벨에서는 언두 로그를 반환한다.
    언두 영역은 commit되기 전까지 즉, 트랜잭션이 끝날 때 삭제된다.

위와 같은 과정을 DBMS에서는 MVCC라고 표현한다.

### 잠금없는 일관된 읽기
- InnoDB 엔진은 MVCC 기술을 이용해 잠금을 걸지 않고 읽기 작업을 수행한다. serializeable이 아닌 이상 insert와 연결되지 않은 순수한 select문은 즉시 동작한다.
- 오랜 시간 활성상태인 트랜잭션으로 인해 서버가 느려지는 경우가 있는데, 이러한 일관된 읽기를 위해 언두 로그가 쌓여서 발생하는 문제다.
- 트랜잭션이 시작되면 가능한 빨리 롤백 / 커밋하는 것이 좋다.

### 자동 데드락 감지
- InnoDB 스토리지 엔진은 내부적으로 데드락 감지 스레드를 가지고 있어서 주기적으로 강제 종료시킨다. 종료 기준은 언두 로그 양이며, 더 적은 양을 가진 트랜잭션이 롤백된다.
- 상위 레이어인 MySQL 엔진에서 관리되는 테이블 잠금도 innodb_table_lock 옵션을 통해 활성화 가능하다.
- 동시 처리 서비스의 경우 데드락 감지 스레드는 느려지며, 더 많은 CPU 자원을 소모할 수 있다.
- 위 문제를 innodb_deadlock_detect 시스템 변수로 통제 가능하며, 더 이상 데드락 감지 스레드는 동작하지 않는다. 이 경우, 2개 이상의 트랜잭션으로 인해 데드락을 요구하는 상황이 발생해도 동작하지 않는데, innodb_lock_wait_timeout 시스템 변수로 일정 시간이 지나면 에러 메세지를 반환한다. 권장 시간은 기본값이 50초보다 훨씬 낮은 시간이다.

### 자동되된 장애 복구
- InnoDB는 장애로부터 손실이나 데이터를 보호하기 위해 MySQL서버가 시작될 때 완료되지 못한 트랜잭션이나 디스크에 일부만 기록된 데이터 페이지 등에 대한 복구 작업이 자동으로 진행된다.
- InnoDB는 견고하여 데이터 파일 손상 발생이 적으나, 서버와 무관하게 디스크나 서버 하드웨어 이슈로 문제가 발생하는 경우 복구가 쉽지 않다. InnoDB 데이터 파일은 MySQL 서버가 시작될 때 자동 복구가 진행되지만, 복구 불가능한 손상 시 서버가 종료된다.
- 이 때, 데이터 파일이나 로그 파일의 손상 여부 검사 과정을 선별적으로 진행 가능하다.
    - innodb_force_recovery 설정은 0~6까지 있으며, 숫자가 클수록 심각한 상황이며 복구 가능성은 낮아진다. (0에서만 쿼리 수행 가능)
    > InnoDB 로그 파일 손상 -> 6으로 설정<br>
    InnoDB 테이블 데이터 파일 손상 -> 1로 설정<br>
    후 MySQL 서버 실행

### InnoDB 버퍼 풀
- 가장 핵심적인 부분으로, 디스크의 데이터 파일이나 인덱스 정보를 메모리에 캐시해 두는 공간이다. 일반적인 앱에서 데이터를 변경하는 쿼리는 다양한 데이터 파일의 레코드를 변경하기 때문에 랜덤한 작업을 발생시킨다. 이러한 작업을 모아서 처리하여 작업 횟수를 줄이는 역할을 InnoDB 버퍼 풀이 한다.

#### 버퍼풀의 구조
- InnoDB 스토리지 엔진은 버퍼 풀이라는 메모리 공간을 시스템 변수로 설정한 크기의 크기로 나누어 InnoDB 스토리지 엔진이 데이터를 플요로 할 때 데이터를 읽어서 각 조각에 저장한다.

- 버퍼 풀의 조각을 관리하기 위해 LRU, 플러시, 프리 리스트라는 3개의 자료 구조로 관리한다.
  - 프리 리스트
    - InnoDB 버퍼 풀에서 실제 사용자 데이터로 채워지지 않은 비어 있는 페이지들의 목록이며, 사용자의 쿼리가 새롭게 디스크의 데이터를 읽어와야 하는 경우에 사용

#### 버퍼 풀과 리두 로그
- 버퍼 풀
    - 저장된 데이터를 모두 적용하는 공간(클린, 더디 페이지)
- 리두로그
    - 변경 사항을 저장하는 공간(더티 페이지)

<br>

- InnoDB의 버퍼 풀은 디스크의 모든 데이터 파일이 버퍼 풀에 적재되기 전 까지는, 서버의 메모리를 허용할수록 쿼리의 성능이 빨라진다. <br>
하지만 InnoDB 버퍼 풀은 데이터베이스 성능 향상을 위해 캐시와 쓰기 버퍼링 두 가지 용도가 있는데, 버퍼 풀의 메모리 공간만 늘리는 것은 캐시 기능만 향상시키는 것이다. <br>
쓰기 버퍼링 기능까지 향상시키기 위해서는 버퍼 풀과 리두 로그와의 관계를 이해해야 한다.

- InnoDB의 버퍼 풀은 디스크의 상태가 변경되지 않은 클린 페이지와 insert 등으로 변경된 데이터를 가진 더티 페이지를 모두 포함하고 있다.

- 더티 페이지는 디스크와 메모리(버퍼 풀)의 데이터 상태가 다르므로 언젠가는 디스크로 기록되어야 한다. InnoDB 스토리지 엔진에서 리두 로그는 1개 이상의 고정 크기 파일을 연결해서 순환 고리처럼 사용한다. 즉, 데이터 변경이 계속 발생하면 리두 로그 파일에 기록됐던 로그 엔트리는 새로운 로그 엔트리로 덮어 쓰인다. 그래서 전체 리두 로그 파일에서 재사용 가능한 공간과 당장 재사용 불가능한 공간을 구분해서 관리해야 하는데, 재사용 불가능한 공간을 활성 리두 로그 라고 한다.

#### 버퍼 풀 플러시
- 플러시 리스트 플러시
    - InnoDB 스토리지 엔진은 리두 로그 공간을 활용하기 위해 더티 페이지를 디스크에 동기화해야 한다. 오래된 리두 로그부터 동기화되며 시스템 변수로 조절 가능하다.

    - 시스템 변수를 조정하여 많은 더티 페이지를 한 번의 쓰기 요청으로 처리하면 효과를 극대화 할 수 있다. 하지만, 너무 많은 더티 페이지로 인한 디스크 쓰기 폭발 현상을 주의히야 한다.

- LRU 리스트 플러시
    - 사용 빈도가 낮은 데이터 페이지들을 제거하여 새로운 페이지를 읽을 공간을 확보한다.

#### 버퍼 풀 상태 백업 및 복구
- 서버 재시작하면 쿼리 성능이 떨어지는데, 버퍼 풀에 쿼리들이 사용할 데이터가 준비되어 있지 않기 때문이다. 이렇게 디스크의 데이터가 버퍼 풀에 적대돼 있는 상태를 워밍업 이라고 표현한다.

### Double Write Buffer
- 리두 로그는 공간 낭비를 막기 위해 변경된 사항만 기록한다. InnoDB의 스토리지 엔진에서 더티 페이지를 디스크에 플러시할 때 일부만 기록된다면 페이지 내용을 복구할 수 없을 수도 있으며, 파셜 페이지 또는 톤 페이지라고 한다. 그래서 나온 방식이 Double Write Buffer 기법이다.

#### 시스템 테이블스페이스
- InnoDB 스토리지 엔진은 데이터 파일 변경 내용을 기록하기 전에 더티 페이지 내용을 묶어 쓰기 방식으로 시스템 테이블스페이스에 기록한다.
- 해당 내용은 파셜 페이지 또는 톤 페이지 현상 발생 시에만 사용된다.