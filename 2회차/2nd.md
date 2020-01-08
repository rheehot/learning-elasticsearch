# ES 온보딩 교육 2회차 & ES 튜토리얼 1~2

<hr>

### ES와 HTTP
| 엘라스틱서치 HTTP 메소드 | 기능 |
|:--------|:--------|
| GET | 데이터 조회 |
| PUT | 데이터 생성 |
| POST | 인덱스 업데이트 or 데이터 조회 |
| DELETE | 데이터 삭제 |
| HEAD | 인덱스 정보 확인 |

API 요청 구조 = (메서드) (인덱스)/(타입)/(문서id) {json 데이터}

### 인덱스 생성 및 삭제, 조회
* 인덱스를 생성하는 방법
    * 인덱스의 `settings`를 정의
        * Static index setting - 원본 샤드 개수를 설정
        * Dynamic index setting - 복제본 샤드 개수를 설정
    * 인덱스의 `mappings`를 정의  
    ex) `PUT twitter[인덱스 이름] {"settings": {"index":{"number_of_shards":3,"number_of_replicas":1}}}`  
    ex) `PUT twitter[인덱스 이름] {"settings" : { "index.number_of_shards" : 3, "index.number_of_replicas" : 1}`
* 인덱스를 삭제하는 방법
    * DELETE 메소드를 사용  
    ex) `DELETE twitter[인덱스 이름]`
* 인덱스의 존재를 확인하는 방법
    * HEAD 메소드를 사용  
    ex) `HEAD twitter[인덱스 이름]` 
* 인덱스의 정보를 조회하는 방법 
    * GET 메소드를 사용
    * 인덱스의 셋팅을 확인  
    ex) `GET twitter[인덱스 이름]/_settings`
    * 인덱스의 매핑을 확인  
    ex) `GET twitter[인덱스 이름]/_mappings`
    * 인덱스의 상태를 확인  
    ex) `GET twitter[인덱스 이름]/_stats`
    * 인덱스의 샤드 및 세그먼트 정보를 확인  
    ex) `GET twitter[인덱스 이름]/_segments`
    * 인덱스의 요약 정보를 확인  
    ex) `GET _cat/indices/[인덱스 이름]?v`

### 문서 색인 및 조회
* 문서를 색인하는 방법
    * PUT 메소드를 사용  
    ex) `PUT twitter/_doc/1 {"user" : "kimchy","post_date" : "2009-11-15T14:12:12", "message" : "trying out Elasticsearch"}`  문서 1번으로 인덱싱  
    ex) `PUT twitter/_doc/1/_create {"user" : "kimchy", "post_date" : "2009-11-15T14:12:12", "message" : "trying out Elasticsearch"}`  id가 없을 때만 인덱싱  
    ex) `PUT twitter/_doc/?op_type=create {"user" : "kimchy", "post_date" : "2009-11-15T14:12:12", "message" : "trying out Elasticsearch"}`  id가 없을 때만 인덱싱  
    * POST 메소드를 사용  
    ex) `POST twitter/_doc {"user" : "kimchy","post_date" : "2009-11-15T14:12:12", "message" : "trying out Elasticsearch"}`  
* 문서를 조회하는 방법
    * GET 메소드를 사용  
    ex) `GET twitter/_doc/1` id 1번 문서 조회  
    ex) `GET twitter/_source/1` 실제 문서 데이터인 _source만 조회

### 문서 갱신 및 삭제
* 문서를 갱신하는 방법
    * PUT 메소드를 사용  
    ex) `PUT twitter/_doc/1 {"user" : "kimchy","post_date" : "2009-11-15T14:12:12", "message" : "trying out Elasticsearch"}`
* 문서를 삭제하는 방법
    * DELETE 메소드를 사용  
    ex) `DELETE twitter/_doc/1`

### 클러스터 정보 확인
* 클러스터 상태 정보를 확인하는 방법
    * GET 메소드를 사용  
    ex) `GET _cluster/health` 클러스터 health 확인  
    ex) `GET _cluster/settings` 클러스터 setting 확인

### ES 플러그인
* 플러그인은 ES 기능을 커스텀 설정에 의해 좀 더 강화하여 사용하는 방법으로 플러그인을 로딩하려면 `클러스터 재시작`이 필요
* Core Plugins - Elasticsearch 에서 공식적으로 지원하는 플러그인
* Community Plugins - Elasticsearch 에서 공식 지원이 아닌 플러그인

### 주요 플러그인
* HEAD 플러그인 - 한눈에 클러스터를 보기 위한 도구로 접속 포트 9100번 사용
* HQ 플러그인 - 한눈에 클러스터의 사용률을 보기 위한 도구로 접속 포트 5000번 사용

### 보충
* ES 7부터는 일부 명령어에서 type(_doc)을 생략하는 것이 디폴트로 되어 있음