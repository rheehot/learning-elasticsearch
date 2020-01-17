# ES 온보딩 교육 5회차 & ES 튜토리얼 5-1

<hr>

### 검색엔진으로 ES 활용하기 - 분석기와 색인
* 색인과 역색인
  * 색인의 필수 조건  
      A. 프라이머리 샤드가 할상 제일 먼저 쓰여야한다.  
      B. 프라이머리 샤드가 전부 쓰여진 이후에 리플리카 샤드로 복제를 한다.  

  * 특징  
      A. 색인은 특정한 데이터가 어느 위치에 있는지 미리 저장해두어 검색 시에 로찾을 수 있는 것으로 색인은 데이터의 위치를 순서대로 기억  
      B. 역색인은 데이터를 색인할 때 Term(단어) 기준으로 색인을 수행하는 것으로 인간의 사고에 가깝고 성능이 좋음  
      C. 사전 정의된 `Analyzer(분석기)`에 의해 토크나이징 된 단어를 색인함  


* `분석기(Analyzer)`
  * 구성요소
    1. Character filter: 원본 텍스트를 사전에 `가공`하는 과정  
      ex) html 태그 제거, 패턴 매칭 ....  
    1. Tokenizer: 어떤 방식으로 원본 Text를 토크나이징하여 `Term`을 만들지 결정하는 과정  
      ex) 공백 기준 토크나이징  
    1. Token filter: Tokenizer에 의해 결정된 Token들을 `필터링`하는 과정  
      ex) stop word 제거(필요없는 word)  
      
  * 특징  
    1. 기본은 standard analyzer이고 직접 정의한 또는 외부 구성요소를 통한 사용자 정의 분석기로 변경 가능
    1. `_analyze API`를 통해 분석되는 토큰 확인 가능(테스트용)
    1. 인덱스의 analysis를 정의하고 해당 인덱스를 대상으로 `_analyze` API 실행 가능
    1. 특정 text type field에 대해 분석기를 변경하려면 반드시 `_reindex` 필요
    1. index close/open 이후 적용
    1. 토큰 생성과 `토큰 검색 시에도` 사용
    1. 분석기를 구성할 때 Tokenizer는 단일 적용이지만 나머지는 복수 적용할 수 있음
  
  * Standard Analyzer  
    1. character filter 미사용  
    1. standard tokenizer 사용  
    1. standard toekn filter, lowercase token filter, stop token filter 사용  
  
  * Whitespace Analyzer  
    1. character filter 미사용  
    1. whitespace tokenizer 사용  
    1. token filter 미사용  
  
  * Nori Analzyer  
    1. character filter 미사용  
    1. nori_tokenizer 사용  
    1. nori_part_of_speech token filter, nori_readingform token filter 사용  
      cf) nori_tokenizer는 decompound_mode 별로 복합어에 대한 다른 토큰을 가져갈 수 있는 설정 존재  
        - none: 단어를 분리하지 앟고 그대로 제공  
        - discard: 복합어는 버리고 봅합어를 나눈 토큰으로 설정  
        - mixed: 복합어와 복합어를 나눈 토큰으로 설정  
  
  * 분석기 정의하는 방법
    * A. 이미 정의되어 있는 ES 제공 analyzer를 그대로 사용하는 방법  
        cf) `analyzer` 필드를 정의하면 인덱스 정의하듯이 입력해도 analyzer로 인식  
        ex)  
        ```json
        PUT index_analyzer_settings2 {"settings": { "analysis": {"analyzer": { "my_analyzer" : {"type": "custom", "char_filter": [ "html_strip" ], "tokenizer": "standard", "filter": ["uppercase" ]} }} },"mappings": { "properties": {"comment": {"type": "text","analyzer": "my_analyzer"} }} }
        ```  

    * B. 이미 정의되어 있는 character filter, tokenizer, token filter를 조합하여 사용하는 방식  
        ex)  
        ```json
        PUT index_analyzer_settings2 {"settings": { "analysis": {"analyzer":  "my_analyzer" : {"type": "custom", "char_filter": [ "html_strip" ], "tokenizer": "standard", "filter": [ "uppercase" ]} }} },"mappings": { "properties": {"comment": {"type": "text", "analyzer": "my_analyzer }} }
        ```

    * C. 플러그인 형태로 제공되는 외부 analyer를 사용하는 방식  
        cf) 분석기에 의한 Token이나 frequency를 확인하고 싶을 때는 `terms vector` 이용  
        ex)  
        ```json
        POST /bank/account/25/_termvectors {"fields": ["address"], "offsets" : true, "payloads" : true, "positions" : true, "term_statistics" : true, "field_statistics" : true}
        ```

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
  * `Fetch Phase` - 문서 ID를 기반으로 문서를 받아오는 과정
    1. 클라이언트로 리턴할 Document를 가지고 있는 샤드들에게만 요청
    2. 요청을 받은 샤드들은 전체 문서 내용(_source) 등의 정보를 요청 노드로 전달하고, 해당 노드는 클라이언트로 최종 결과를 리턴
    
* 쿼리 요청 방식에 따른 분류
  * URI 검색
    1. URI에 request parameter를 통해서 검색 질의  
    2. 한정된 옵션의 검색만 가능  
      ex)`GET bank/_search?from=0&size=100&q=address:Fleet&sort=age:asc`  

  * Request Body 검색  
    1. `Query DSL(Domain Spectific Language)`을 이용해 HTTP Body를 정의한 이후 query 절로 질의  
    1. 여러가지 옵션을 넣어서 질의 가능  
      ex)`GET bank/_search {"query" : { "term" : {"city.keyword": "Mulino" }}}`  

* Request Body 검색(Query DSL)
  * JSON 기반의 ES 쿼리를 정의하는 언어
  * Query DSL 특징
    1. from, size parameter로 pagination 설정 가능  
      ex)`GET bank/_search {"from":0, "size": 2, "query" : {"match" : { "address": "Fleet"} }}`  
    1. `index.max_result_window` 필드로 조정 가능  
      ex)`PUT bank/_settings {"index.max_result_window": 10001 }`  
    1. key field 기준으로 `sort` 가능 - 오름차순(asc), 내림차순(desc)  
      ex)`GET bank/_search {"sort": {"age": "desc"} }`  
    1. score 가 계산되는 과정을 확인  
      ex)`GET bank/_search {"explain": true, "from":0, "size": 2, "query" : {"match" : { "address": "Fleet"} }}`  
    1. 원하는 데이터만 필터링  
      ex)`GET bank/_search {"_source": false,"sort": { "age": "desc" } }`  

* 쿼리 요청 결과에 따른 분류
  * Full Text Query(Query Context)
    1. 쿼리 문을 분석하여 검색하고 유사성을 기준으로 스코어(`_score`)가 가장 높은 문서 순으로 결과를 리턴
    1. match, match_phrase, match_phrase_prefix, query_string의 쿼리를 이용  
      cf) 스코어링 요소 - TF 가 높을수록, IDF 가 낮을수록, Field Length 가 낮을수록 스코어가 높아짐
    1. analyze해서 검색
        1. `TF(Term Frequency)` : Term이 해당 Document에 등장하는 빈도  
        2. `IDF(Inverse Document Frequency)` : Term이 전체 Index에서 등장하는 빈도  
        3. `Field Length` : Term이 포함된 Field의 길이  
    
  * Term Level Query(Filter Context)  
    1. 정확히 일치하는 용어만 검색하고 analyze되지 않은 결과만 리턴(NoSQL을 쓰는 듯한 느낌)
    1. 정확히 일치하는 문서만 검색되기 때문에 _score 무의미(계산은 됨)
    1. keyword field 기반으로만 검색 ({field}.keyword로 쿼리를 정의하든 mapping 시 keyword 로 미리 설정해야함)
    1. term, terms, range의 쿼리를 이용

* Full Text Query의 쿼리 종류
  * `match Query` - 쿼리문의 analyze된 토큰으로 유사한 문서 검색  
  ex)`GET bank/_search {"query": { "match": {"address": {"query": "345 Fleet", "boost": 2.0} }} }`  
  cf) boost 옵션으로 스코어의 가중치를 부여할 수 있음  
  * `match_phrase Query` - analyze된 쿼리의 토큰 중 해당 value를 순서대로 합쳐서 쿼리 구문을 만들어 검색  
  ex)`GET bank/_search {"query": { "match_phrase": {"address": "Fleet Walk" }} }`  

  * `match_phrase_prefix Query` - match_phrase와 동일한 방식으로 문서 검색  
  ex)`GET bank/_search {"query": { "match_phrase_prefix": {"address": "425 Fleet W" }} }`

  * `multi_match Query` -  fields parameter에 정의된 field들에 대해서 match 쿼리 검색  
  ex)`GET multi_match_index/_search {"query": { "multi_match": {"query": "ks","fields": [ "first", "comment"] }} }`

  * `query_string Query` - match, multi_match, match_prase_prefix의 기능을 할 수 있는 쿼리
  ex)`GET bank/_search {"query": { "query_string" : {"query" : "Walk Flee*","fields": [ "address", "employer"] }} }`  

* Term Level Query의 쿼리 종류
  * `term Query` - 역색인 된 토큰 중에 term이 정확히 일치하는 것만 검색  
  ex)`GET bank/_search {"query": { "term" : {"gender.keyword": "M", "boost": 2.0 }} }`  
  cf) boost 옵션으로 스코어의 가중치를 부여할 수 있음  

  * `terms Query` - 여러개의 용어에 대해 검색  
  ex)`GET bank/_search {"query": { "terms" : {"gender.keyword": ["F","M"] }} }`

  * `range Query` - numeric, date, geo field에 대해서만 가능  
  cf) gte(greater than || equal), gt, lte(less than || equal), lt parameter 사용  

  * `wildcard Query` - term level 쿼리 중 와일드카드를 쓸 수 있는 쿼리  
  ex)`GET bank/_search {"query": { "wildcard" : {"lastname.keyword": "D*e" }} }`  
  cf) analyze가 되지 않음

* 쿼리 안의 구조에 따른 분류
  * `Lead query caluse` - 자체적으로 쿼리를 할 수 있는 완성된 검색 쿼리(단일 쿼리)  
    ex) match, term, range
  * `Compound query clause` - leaf query 혹은 compound query를 혼합해주는 검색 쿼리(혼합 쿼리)  
    ex) bool, boosting

* Compound query clause의 쿼리 종류 - bool 쿼리를 이용하고 그 하위 서브 쿼리 조합으로 사용
  * bool + `must` - 문서에 일치하는 항목, 스코어 계산  
    ex)`GET bank/_search {"query": { "bool": {"must": [ {"match": { "address": {"query": "Fleet" }} }] }} }`  
  * bool + `filter` - 문서에 일치하는 항목, 캐싱, 스코어 셋팅이 0이므로 보통 filter context(term 쿼리) 실행  
    ex)`GET bank/_search {"query": { "bool": {"filter": [ {"match": { "address": {"query": "Fleet" }} }] }} }`
  * bool + `should` - 문서에 일치하는 항목, must 나 filter 항목이 없으면 적어도 하나의 쿼리절과 일치되는 결과 리턴  
    1. must나 filter 항목이 없으면 적어도 하나의 쿼리절과 일치되는 결과 리턴(minimum_should_match가 default로 1)
    1. must, filter 항목이 있으면 모두 리턴(minimum_should_match가 default로 0)  
    ex)`GET bank/_search{"query": {"bool": { "should": [{"match": {"state": { "query": "MI","boost": 2 }} },{"term": {"gender.keyword": {"value": "M" }} }],"minimum_should_match" : 1 }} }`  
  * bool + `must_not` - 문서에 일치하지 않는 항목, 캐싱, 스코어 셋팅이 1이므로 보통 filter context 실행(term 쿼리)  
  ex)`GET bank/_search {"query": { "bool": {"must_not": [ {"match": { "address": {"query": "Fleet" }} }] }} }`