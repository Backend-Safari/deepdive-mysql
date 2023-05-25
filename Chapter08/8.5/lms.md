## 전문 검색 인덱스
- 지금까지 알아본 B-Tree는 전체 일치 또는 좌측 일부 일치같은 키워드화한 작은 값에 대한 인덱싱 알고리즘이다. 반대로 전문 검색 인덱스는 문서 전체를 인덱스화해서 특정 키워드를 인덱싱한다.

### 인덱스 알고리즘
- 본문 내용 중 검색하게 될 키워드를 분석하고 인덱스로 구축한다.
- 인덱싱 기법에는 어근 분석과 n-gram 분석으로 구분할 수 있다.

#### 어근 분석
- 불용어 처리, 어근 분석 과정을 거쳐 색인 작업을 수행한다.
    - 불용어 처리 : 검색에서 의미없는 단어 제거
    - 어근 분석 : 검색어의 뿌리인 원형을 찾는 작업
- 한국어의 경우 어근 분석보다 형태소 분석을 통한 명사 동사 구분이 중요하다. 언어마다 문법이 다르므로 유사 언어인 일본어 분석 프로그램 MeCab으로 대체 가능하다.
- 불용어는 설정 파일을 통한 커스텀 적용 및 제거가 가능하고, 스토리지 엔진별 온오프 관리가 가능하다.
    - 커스텀 불용어는 목록 파일, 스토리지 엔진을 사용하는 테이블로 사용할 수 있다. 테이블의 경우 전문 검색 인덱스가 생성돼야만 적용된다.

#### n-gram 알고리즘
- 문장이 아닌 키워기 기반 인덱싱 알고리즘이다.
- 인덱싱할 최소 단위를 n에 대입하며, 2글자의 경우 2-gram이라고 한다.
- 공백과 마침표 기준으로 단어를 구분하고, n개만큼 중첩하여 토큰으로 구분하므로 2-gram 10단어의 경우 (10-1)개 토큰으로 구분한다.

### 전문 검색 인덱스
- 전문 검색 인덱스 사용 조건
    - 쿼리 문장이 전문 검색을 위한 문법(match ... against ...) 사용
        - 문법을 사용하지 않으면 풀 테이블 스캔으로 처리한다.
    - 테이블이 전문 검색 대상 컬럼에 대해서 전문 인덱스 보유