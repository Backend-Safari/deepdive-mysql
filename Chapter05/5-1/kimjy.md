## 트랜잭션
- MyISAM은 트랜잭션 중에 부분 업데이트를 수행하는 partial update가 가능
- InnoDB는 partial update를 허용하지 않음
- 트랜잭션은 꼭 필요한 최소의 코드에만 적용이 필요
  - 너무 많은 작업을 한 트랜잭션내에서 수행할 경우 문제가 발생
