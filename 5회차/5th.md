# ES 온보딩 교육 5회차 & ES 튜토리얼 5

<hr>

### 검색엔진으로 ES 활용하기 - 분석기와 색인
* 색인과 역색인
  * 색인의 필수 조건
    1. 프라이머리 샤드가 할상 제일 먼저 쓰여야한다.
    2. 프라이머리 샤드가 전부 쓰여진 이후에 리플리카 샤드로 복제를 한다.
  * 특징
    * 색인은 특정한 데이터가 어느 위치에 있는지 미리 저장해두어 검색 시에 로찾을 수 있는 것으로 색인은 데이터의 위치를 순서대로 기억
    * 역색인은 데이터를 색인할 때 Term(단어) 기준으로 색인을 수행하는 것으로 인간의 사고에 가깝고 성능이 좋음
    * 사전 정의된 `Analyzer(분석기)`에 의해 토크나이징 된 단어를 색인함

* `분석기(Analyzer)`
  * 구성요소
    1. Character filter: 원본 텍스트를 사전에 가공하는 과정  
    ex) html 태그 제거, 패턴 매칭 ....
    2. Tokenizer: 어떤 방식으로 원본 Text를 토크나이징하여 Term을 만들지 결정  
    ex) 공백 기준 토크나이징
    3. Token filter: Tokenizer에 의해 결정된 Token들을 가공하는 과정  
    ex) stop word 제거
  * 특징
    1. 기본은 standard analyzer이고 직접 정의한 또는 외부 구성요소를 통한 사용자 정의 분석기로 변경 가능
    2. `_analyze API`를 통해 분석되는 토큰 확인 가능(테스트용)
    3. 인덱스의 analysis를 정의하고 해당 인덱스를 대상으로 `_analyze` API 실행 가능
    4. 특정 text type field에 대해 분석기를 변경하려면 반드시 `_reindex` 필요
    5. index close/open 이후 적용
    6. 토큰 생성과 `토큰 검색 시에도` 사용
  * Standard Analyzer
    1. character filter 미사용
    2. standard tokenizer 사용
    3. standard toekn filter, lowercase token filter, stop token filter 사용
  * Whitespace Analyzer
    1. character filter 미사용
    2. whitespace tokenizer 사용
    3. token filter 미사용
  * Nori Analzyer
    1. character filter 미사용
    2. nori_tokenizer 사용
    3. nori_part_of_speech token filter, nori_readingform token filter 사용
    cf) nori_tokenizer는 decompound_mode 별로 복합어에 대한 다른 토큰을 가져갈 수 있는 설정 존재
      none: 단어를 분리하지 앟고 그대로 제공  
      discard: 복합어는 버리고 봅합어를 나눈 토큰으로 설정  
      mixed: 복합어와 복합어를 나눈 토큰으로 설정  
  * 분석기 정의하는 방법
    1. 이미 정의되어 있는 ES 제공 analyzer를 그대로 사용하는 방법  
    ex)`PUT index_analyzer_settings2 {"settings": { "analysis": {"analyzer": { "my_analyzer" : {"type": "custom", "char_filter": [ "html_strip" ], "tokenizer": "standard", "filter": ["uppercase" ]} }} },"mappings": { "properties": {"comment": {"type": "text","analyzer": "my_analyzer"} }} }`  
    cf) `Analyzer` 필드를 정의하면 익데스 정의하듯이 정의해도 analyzer로 인식 
    2. 이미 정의되어 있는 character filter, tokenizer, token filter를 조합하여 사용하는 방식
    ex)`PUT index_analyzer_settings2 {"settings": { "analysis": {"analyzer":  "my_analyzer" : {"type": "custom", "char_filter": [ "html_strip" ], "tokenizer": "standard", "filter": [ "uppercase" ]} }} },"mappings": { "properties": {a  "comment": {"type": "text", "analyzer": "my_analyzer }} }`
    3. 플러그인 형태로 제공되는 외부 analyer를 사용하는 방식  
    cf) 분석기에 의한 Token이나 frequency를 확인하고 싶을 때는 `terms vector` 이용  
    ex)`POST /bank/account/25/_termvectors {"fields": ["address"], "offsets" : true, "payloads" : true, "positions" : true, "term_statistics" : true, "field_statistics" : true}`

### 검색엔진으로 ES 활용하기 - 검색
* 검색
  * `_search` API를 통해서 문서를 검색
  * 쿼리 요청 구조에 따라 `URI` 혹은 `HTTP request body` 2가지 방식의 요청이 있음
  * 쿼리 안의 쿼리 개수에 따라 `Leaf query 절`과 `Compound query 절`로 나눠짐  
  * Query Phase와 Fetch Phase로 나눠짐

* 검색 과정
  * `Query Phase` - 쿼리를 받아 문서가 어느 노드의 어떤 샤드에 있는지를 찾는 과정
    1. 요청을 받은 노드에서 from, size를 계산한 빈 Queue를 생성
    2. 다른 노드들에 Broadcast하여 다른 노드들도 비어 있는 로컬 Queue를 생성
    3. 노드들은 Queue에 문서의 ID를 넣고 리턴하고 score 기준으로 sorting
  * `Fetch Phase` - 문서 id를 기반으로 문서를 받아오는 과정
    1. 클라이언트로 리턴할 Document를 가지고 있는 샤드들에게만 요청
    2. 요청을 받은 샤드들은 전체 문서 내용(_source) 등의 정보를 요청 노드로 전달하고, 해당 노드는 클라이언트로 최종 결과를 리턴

* 쿼리 안의 쿼리 개수에 따른 분류
  * `Lead query caluse` - 자체적으로 쿼리를 할 수 있는 완성된 검색 쿼리(단일 쿼리)  
    ex) match, term, range
  * `Compound query clause` - leaf query 혹은 compound query를 혼합해주는 검색 쿼리(여러개 혼합)  
    ex) bool, boosting
    
* 쿼리 요청 구조에 따른 분류
  * URI 검색
    1. URI에 request parameter를 통해서 검색 질의
    2. 한정된 옵션의 검색만 가능  
    ex)`GET bank/_search?from=0&size=100&q=address:Fleet&sort=age:asc`

  * Request Body 검색
    1. `Query DSL(Domain Spectific Language)`을 이용해 HTTP Body를 정의한 이후 질의
    2. 여러가지 옵션을 넣어서 질의 가능  
    ex)`GET bank/_search {"query" : { "term" : {"city.keyword": "Mulino" }}}`

* Query DSL
  * JSON 기반의 ES 쿼리를 정의하는 언어