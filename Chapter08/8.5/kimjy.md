R-Tree

- 공간 정보 저장을 위한 지오메트리 정보를 저장할 수 있는 데이터 타입
- 공간정보는 MBR을 통해 관리되며, MBR은 다시 여러 레벨로 나뉘어서 관리되며, 이 레벨을 통해 트리를 구성하고, 인덱싱을 수행한다.

전문 검색 인덱스

- 엘라스틱서치같은 건가..??
- 문서 내용 전체를 인덱스화 해서 특정 키워드가 포함된 문서를 검색하는 기능은 B-Tree 만을 사용해서는 사용이 불가능
- 이를 위해 전문 검색 인덱스를 사용해야 함
    - 문서의 키워드를 인덱싱하는 기법은, 어근 분석과 n-gram으로 분류할 수 있음.
- 어근 분석
    - working → work, worked → work 처럼 단어의 원형을 찾는 것
- 형태소 분석
    - 형태소: 언어에서 의미가 있는 최소의 단위(minimally meaningful unit)
    - 형태소 분석: 단어(또는 어절)를 구성하는 각 형태소 분리 및
분리된 형태소의 기본형 및 품사 정보 추출
  - 가능한 패키지는 MeCab이 존재하지만, MySQL에 적용하기에는 너무 많은 시간과 노력이 필요
  - 따라서 n-gram을 사용하기도 함.
- n-gram
  - n-gram은 n 개의 글자씩 잘라서 인덱싱하는 방법
  - n개씩 잘라서 인덱싱을 수행하나, 만약 중복된 토큰이면 병합하여 인덱싱에 저장
  - 불용어가 존재하면 걸러서 버림
  - 일반적으로 2-gram을 주로 사용 
- 불용어 처리
    - I, my, me 처럼 문장에는 자주 등장하나, 실제 분석에는 큰 의미가 없는 단어를 제거
    - 우리가 검색에서 I, my, me 등을 검색하는 경우는 크지 않음