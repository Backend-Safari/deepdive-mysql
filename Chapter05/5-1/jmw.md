### 트랜잭션

- 트랜잭션은 작업의 완전성을 보장해 주는 것이다. 즉 논리적인 작업 셋을 모두 완벽하게 처리하거나, 처리하지 못할 경우에는 원 상태로 복구해서 작업의 일부만 적용되는 현상(Partial update)이 발생하지 않게 만들어주는 기능이다.


- 잠금은 동시성을 제어하기 위한 기능


- 트랜잭션은 데이터의 정합성을 보장하기 위한 기능이다.


- 격리 수준이라는 것은 하나의 트랜잭션 내에서 또는 여러 트랜잭션 간의 작업 내용을 어떻게 공유하고 차단할 것인지를 결정하는 레벨을 의미한다.

<br>

주의 사항

- 실제로 DBMS에 데이터를 저장하는 작업은 트랜잭션에 포함


- 데이터베이스 커넥션은 개수가 제한적이어서 각 단위 프로그램이 커넥션을 소유하는 시간이 길어질수록 사용 가능한 여유 커넥션의 개수는 줄어듬


- 단위 프로그램에서 커넥션을 가져가기 위해 기다려야 하는 상황이 발생할 수 있다.


- 메일 전송이나 FTP 파일 전송 작업 또는 네트워크를 통해 원격 서버와 통신하는 등과 같은 작업은 어떻게 해서든 DBMS의 트랜잭션 내에서 제거하는 것이 좋다.


- 프로그램이 실행되는 동안 메일 서버와 통신할 수 없는 상황이 발생한다면 웹 서버뿐 아니라 DBMS 서버까지 위험해지는 사오항이 발생할 것이다.