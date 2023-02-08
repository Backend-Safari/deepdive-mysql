# 04. 아키텍처
### 목적
> MySQL 서버는 MySQL 엔진과 스토리지 엔진로 구성되어 있고, 이 두 엔진의 차이점에 대해 설명할 수 있어야 한다.

---

## 4-1 MySQL 엔진 아키텍처
### MySQL의 전체 구조

<br>

#### <strong>MySQL 엔진</strong>
- 클라이언트로부터 접속 및 쿼리 요청을 처리하는 핸들러와 파서 및 전처리기, 쿼리의 최적화된 실행을 위한 옵티마이저가 중심을 이룬다.
#### <strong>스토리지 엔진</strong>
- MySQL 엔진이 분석 및 최적화를 담당한다면, 스토리지 엔진은 실제 데이터를 디스크에 저장하거나 읽어오는 역할을 한다.
<br>

> MySQL엔진은 하나지만, 스토리지 엔진은 여러 개를 동시에 사용 가능하다.

<br>

#### <strong>핸들러 API</strong>
- MySQL 엔진의 쿼리 실행기에서 데이터를 쓰거나 읽을 때 스토리지 엔진에 쓰기 또는 읽기를 요청하는데, 이 요청을 핸들러 요청이라 하고, 사용되는 API를 핸들러 API라고 부른다.

<br>

---

### <strong>MySQL 스레딩 구조</strong>

MySQL 서버는 프로세스 기반이 아닌 스레드 기반으로 작동하며, 크게 포그라운드와 백그라운드 스레드로 구분할 수 있다.
<br>

백그라운드의 스레드 개수는 MySQL 서버의 설정에 따라 가변적이며, 2개 이상은 설정 내용에 의해 여러 스레드가 동일 작업을 병렬 처리하는 경우이다.

<br>
<br>

#### <strong>포그라운드 스레드(클라이언트 스레드)</strong>
- 포그라운드 스레드는 주로 각 클라이언트 사용자가 요청하는 쿼리 문장을 처리한다. 데이터를 MySQL의 데이터 버퍼나 캐시로부터 가져오며, 캐시가 없는 경우 직접 디스크나 인덱스 파일에서 데이터를 읽어와서 작업을 처리한다.
> MyIZSAM - 디스크 쓰기 작업까지 포그라운드 스레드 처리<br>
InnoDB - 데이터 버퍼나 캐시까지만 포그라운드 스레드가 처리

<br>

#### <strong>백그라운드 스레드</strong>
MyISAM과 달리 InnoDB는 여러 작업이 백그라운드로 처리된다.
- 인서트 버퍼를 병합하는 스레드
- 로그를 디스크로 기록하는 스레드
- InnoDB 버퍼 풀의 데이터를 디스크에 기록하는 스레드
  - InnoDB 버퍼 풀이란?
- 데이터를 버퍼로 읽어 오는 스레드
- 잠금이나 데드락을 모니터링하는 스레드
  - 데드락이란?

모두 중요한 역할을 하지만 가장 중요한 것은 로그 스레드와 버퍼의 데이터를 디스크로 내려쓰는 쓰기 스레드일 것이다.<br>

사용자의 요청을 처리하는 도중 데이터의 쓰기 작업은 지연 처리될 수 있지만 익기 작업은 절대 지연될 수 없다. 그래서 삿용 DBMS에는 대부분 쓰기 작업을 버퍼링해서 일괄 처리하는 기능이 탑재돼 있으며, InnoDB도 이와 같이 처리한다.

<br>

---
### 메모리 할당 및 사용 구조
MySQL에서 사용되는 메모리 공간은 글로벌 메모리 영역과 로컬 메모리 영역으로 구분할 수 있으며, MySQL 서버 내에 존재하는 스레드가 공유해서 아용하는 공간인지 여부에 따라 구분된다.

<br>

#### <strong>글로벌 메모리 영역</strong>
일반적으로 클라이언트 스레드의 수와 무관하게 하나의 메모리만 할당된다. 예외도 있지만 클라이언트 스레드와 무관하며, 각 메모리가가 모든 스레드에 공유된다.
- 테이블 캐시
- InnoDB 버퍼 풀
- InnoDB 어댑티브 해시 인덱스
- InnoDB 라두 로그 버퍼

<br>

#### <strong>로컬 메모리 영역</strong>
MySQL 서버 상에 존재하는 클라이언트 스레드가 쿼리를 처리하는 데 사용하는 메모리 영역이다.
- 정렬 버퍼(소트)
- 조인 버퍼
- 바이너리 로그 캐시
- 네트워크 버퍼

로컬 메모리는 각 클라이언트 스레드별로 독립적으로 할당되며 절대 공유되지 않는다는 특성이 있다.

로컬 메모리 공간의 특징은 각 쿼리의 용도별로 필요할 때만 할당되고 필요하지 않은 경우 할당되지 않을 수 있다는 점이다. 대표적으로 소트 버퍼나 조인 버퍼 공간이 그렇다.
> 커넥션이 열려 있는 동안 계속 할당된 공간도 있고(= 커넥션 버퍼, 결과 버퍼)<br>
쿼리를 실행하는 순간에만 할당하는 공간도 있다.(= 소트 버퍼, 조인 버퍼)

<br>

---
### 플러그인 스토리지 엔진 모델
MySQL의 독특한 구조 중 대표적인 것이 바로 플러그인 모델이다. 스토리지 엔진 뿐만 아닌 검색 엔진, 사용자 인증 등도 모두 플러그인으로 구현되어 제공된다.

하지만 대부분의 작업이 MySQL 엔진에서 처리되고, 마지막 데이터 읽기/쓰기 작업만 스토리지 엔진에서 처리되므로 일부분의 기능을 수행하는 엔진을 작성하게 된다. 실질적인 GROUP BY / ORDER BY 등 복잡한 처리는 MySQL 엔진의 쿼리 실행기에서 처리되며, MyISAM이나 InnoDB의 차이는 크게 없다.

하지만 스토리지 엔진은 무엇을 사용하든 차이가 없는 것이 아닌가? 싶지만 그렇지 않다. **중요한 것은 하나의 쿼리 작업은 여러 하위 작업으로 나뉘는데, 각 하위 작업이 어떤 엔진 영역에서 처리되는지 구분할 줄 알아야 한다.**

<br>

---

### 컴포넌트
MySQL 8.0 부터는 기존의 플러그인 아키텍처를 대체하기 위해 컴포넌트 아키텍처가 지원된다. MySQL 서버의 플러그인은 몇 가지 단점이 있는데, 아래와 같다.
- 플러그인은 오직 MySQL 서버와 인터페이스할 수 있고, 플러그인끼리는 통신할 수 없음
- 플러그인은 MySQL 서버의 변수나 함수를 직접 호출하기 때문에 안전하지 않다(캠슐화 안 됨)
- 플러그인은 상호 의존 관게를 설정할 수 없어서 초기화가 어려움

<br>

---

### 쿼리 실행 구조
#### <strong>쿼리 파서</strong>
- 사용자 요청으로 들어온 쿼리 문장을 토큰(MySQL이 인식 가능한 최소 단위)으로 분리해 트리 형태의 구조로 만들어 내는 작업을 의미한다. 쿼리 문장의 문법 오류는 이 과정에서 발견되고, 오류 메세지를 전달한다.

#### <strong>전처리기</strong>
- 파서 과정에서 만들어진 파서 트리를 기반으로 쿼리 문장에 구조적인 문제가 있는지 확인한다. 각 토큰을 테이블 이름, 내장 함수와 같은 개체와 매핑해 해당 객체의 존재 여부와 접근 권한 등을 확인하는 과정을 수행한다.

#### <strong>옵티마이저</strong>
- 사용자 요청으로 들어온 쿼리 문장을 저렴한 비용으로 가장 빠르게 처리할지를 결정하는 역할을 담당한다. 옵티마이저의 역할은 중요하고 영향 범우 ㅣ또한 아주 넓다.

#### <strong>실행 엔진</strong>
- 회사로 비유하면 옵티마이저는 경영진, 실행 엔진은 중간 관리자, 핸들러는 각 업무의 실무자로 비유할 수 있다. 옵티마이저가 GROUP BY를 처리하기 위해 임시 테이블을 사용할 때의 예시이다.

1. 실행 엔진이 핸들러에게 임시 테이블을 만들라고 요청
2. 다시 실행 엔진이 WHERE 절에 일치하는 레코드를 읽어오라고 핸들러에 요청
3. 읽어온 레코드들을 준비한 임시 테이블로 저장하라고 다시 핸들러에게 요청
4. 데이터가 준비된 임시 테이블에서 필요한 방식으로 데이터를 읽어 오라고 핸들러에게 다시 요청
5. 최종적으로 실행 엔진은 결과를 사용자나 다른 모듈로 넘김

즉, 실행 엔진은 만들어진 계획대로 각 핸들러에게 요청해서 받은 결과를 다른 핸들러 요청의 입력으로 연결하는 역할을 수행한다.

#### <strong>핸들러(스토리지 엔진)</strong>
- 앞서 언급한 것처럼 핸들러는 MySQL 서버의 가장 밑단에서 실행 엔진의 요청에 따라 데이터를 디스크로 저장하고, 읽어 오는 역할을 한다. 핸들러는 결국 스토리지 엔진을 의미하며, 조작하는 테이블에 따라 MyISAM / InnoDB 스토리지 엔진이 된다.

<br>

---

### 쿼리 캐시
- 쿼리 캐시는 빠른 응답을 위해 사용된다. SQL의 실행 결과를 메모리에 캐시하고, 동일 쿼리가 실행되면 테이블을 읽지 않고 즉시 결과를 반환하기 때문에 매우 빠른 성능을 보인다. 하지만 테이블 데이터가 변경되면 캐시에 저자된 결과 중 변경된 테이블과 관계가 있으면 모두 삭제된다. 이는 심각한 동시 처리 성능 저하를 유발한다.

- 결국 MySQL 8.0이 나오면서 쿼리 캐시는 MySQL 서버의 기능에서 완전히 제거되고, 관련 시스템 변수도 모두 제거됐다.
> 독튼한 환경(= 데이터 변경없이 읽기만 하는 서비스)에서만 효율적이다.<br>
실제로 도움은 안되지만 많은 버그를 유발하므로 좋은 선택인 것 같다.

<br>

---
### 명령어 모음

#### MySQL 구조
- 스토리지 엔진 지정
  - mysql> create table test_table (f1 int, f2 int) engine=innodb;
- 핸들러 API 작업 카운트
  - show global status like 'handlerr%';

#### 스레딩 구조
- 실행 중인 스레드 목록
  - select thread_id, name, type, processlist_user, processlist_host<br>
  from performance_schema, threads order by type, thread_id;

#### 플러그인 스토리지 엔진
- 스토리지 엔진 목록
  - show engines;
#### 컴포넌트
- 설치
  - mysql> install component 'file://component)validate_password';
- 확인
  - mysql> select * from mysql.component;

#### 트랜잭션 메타데이터
- mysql 테이블 딕셔너리 데이터 구조
  - linux> idb2sdi mysql_data_dir/mysql.idb > mysql_schema.json
  - linux> cat mysql_schema.json