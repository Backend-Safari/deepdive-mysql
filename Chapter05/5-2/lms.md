## 5-2 MySQL 엔진의 잠금
MySQL의 잠금은 MySQL 레벨과 스토리지 레벨로 나눌 수 있다. MySQL 레벨의 잠금은 모든 스토리지 레벨에서 적용되지만, 스토리지 엔진 레벨의 잠금은 상호 간 영향을 미치지 않는다.

### 글로벌 락
- 잠금 범위가 MySQL 서버 전체이며, MySQL에서 제공하는 잠금 중 범위가 가장 크다.
- FLUSH TABLES WITH READ LOCK 명령어로 획득할 수 있으며, 일관된 백업을 받을 때 사용된다.
- mysqldump 같은 백업 프로그램을 사용할 때는 설정에 따른 글로벌 락 동작 여부를 확인해보는 것이 좋다.
> 명령이 시작되기 전에 DDL, DML 작업을 수행하고 있다면 읽기 잠금을 걸기 위해 해당 SQL과 트랜잭션이 완료될 때 까지 기다려야 한다. 최악의 상황을 가정하면 장시간 걸리는 쿼리와 글로벌 락이 완료될 때 까지 다른 쿼리가 실행되지 못할 수도 있다.

- 일반적으로 MySQL 서버 백업은 레플리카 서버에서 실행된다. 하지만 백업이 글로벌 락을 획득하면 복제는 백업 시간만큼 지연되며, 소스 서버에 문제가 생기면 그만큼 서비스를 멈춰야 할 수도 있다.
    - XtraBackup, Enterprise Backup은 복제가 진행되는 상태에서 백업을 만들 수 있다. ( 단, 스키마 변경 시 백업 실패 )
    - 백업 락은 위와 같은 상황을 방지하기 위해 만들어졌으며, 백업 실패를 막기 위해 DDL 명령이 실행되면 복제를 중지한다.

### 테이블 락
- 개별 테이블 단위로 설정되는 잠금이며, 명시적 / 묵시적으로 특정 테이블 락을 획득할 수 있다.
- 묵시적 락은 쿼리가 실행되는 동안 자동으로 획득했다가 쿼리 완료 후 자동 해제된다.
- 모든 스토리지 엔진에 적용할 수 있으나 InnoDB 엔진의 경우 레코드 기반의 잠금을 제공하므로 스키마를 변경하는 쿼리(DDL)에만 영향을 미친다.

### 네임드 락
- 네임드 락은 테이블이나 레코드, 데이터베이스 객체가 아닌 지정 문자열에 대해 잠금을 획득한다.
- 복잡한 요건으로 레코드를 변경하는 경우, 동일 데이터를 변경하거나 참조하는 프로그램끼리 분류해서 네임드 락을 걸고 쿼리를 실행하면 데드락을 최소화할 수 있다.

### 메타데이터 락
- 데이터베이스 객체의 이름이나 구조를 변경하는 경우 자동 획득하는 잠금이다.
- 테이블 이름 변경 시 원본과 변경될 테이블 모두 잠금 설정한다. 따로 잠금 설정하고 진행할 경우 테이블이 없는 찰나의 시간이 존재하여 테이블을 못 찾는 에러가 발생된다.
- 테이블 구조 변경의 경우, 오래 걸릴 때의 언두 로그 증가와 Online DDL이 실행되는 동안 누적된 Online DDL 버퍼 등 고려해야 할 문제가 많다. MySQL 서버의 DDL은 단일 스레드로 동작하므로 pk 구간별로 나누어 진행하여 시간을 단축해야 한다. 이후 남은 데이터( 최대한 최근일수록 좋다 )는 테이블 잠금 설정하여 서비스에 미치는 영향을 최소한으로 줄일 수 있다.