# ES 온보딩 교육 4회차 & ES 튜토리얼 3-2

<hr>

### Rolling Restart
* 의미
    * 시스템 작업이나 ES의 버전 업그레이드를 해야하는 경우가 있을 때 노드들을 내리게 되면 주인을 잃은 샤드들이 기본 라우팅 설정에 의해 균등을 위해 자동으로 프라이머리 샤드와 레플리카 샤드가 다른 노드에 복구 되는데 이때 발생하는 다시 샤드를 재배치하고 리벨런싱하는 오버헤드를 줄이기 위한 작업
* 과정
    1. `_cluster API`로 라우팅 할당(`cluster.routing.allocation.enable`)을 `new_primaires`로 변경하여 Primary와 Replica 샤드 간 데이터 동기화
    2. 작업 하고자 하는 노드 중지 -> 제외된 노드 내 샤드들이 `unassignned` 상태가 됨(클러스터 상태 Yellow)
    3. 작업 실행(ES 버전 업 or 시스템 작업)
    4. 해당 노드의 ES 재시작
    5. _cluster API로 라우팅 할당을 `null`로 변경하여 라우팅 할당을 가능하게 변경

### Shard Allocation
* 의미
    * 생성되는 인덱스의 샤드가 노드의 수와 동일하면 큰 문제가 되지 않으나 노드 증설으로 인해 노드 간의 용량 이격이 벌어져 샤드 할당
* 방식
    * A. `_cluster API`로 reroute를 이용하여 샤드를 강제 분배
    * B. `_cluster API`의 settings의 `disk threshold`와 `watermark` 설정을 이용 (low < high < flood_stage)
        1. watermark.low: 더이상 커지지 않게할 디스크 볼륨 내 임계치(신규 생성 인덱스를 제외)
        2. watermark.high: 더이상 커지지 않게할 디스크 볼륨 내 임계치(모든 인덱스)
        3. watermark.flood_stage: 초과 즉시 모든 인덱스의 write 작업을 막는고 read-only로 사용되는 임계치
    * C. 데이터 노드를 그룹으로 샤드 할당
        1. 데이터 노드의 `node.attr.rack.id`를 통해 region 셋팅
        2. `cluster.routing.allocation.awareness.attributes`를 마스터 노드에 셋팅하는 순간부터 시작

### ES 환경 설정(Dynamic settings)
* 의미
    * static 설정과는 달리 운영 중에 인덱스의 `_settings`을 변경하는 방식으로 REST API로 변경 상태를 설정
* 방식 
    * A. `index.number_of_replicas` - 운영 중에 리플리카 샤드 개수를 변경
    * B. `index.refresh_interval` - 메모리 버퍼캐시로 쓰인 이후 검색이 되도록 디스크로 쓰이는 시간 간격(모든 인덱스에 적용도 가능)
    * C. `Routing Allocation` - `index.routing.allocation.enable`을 이용하여 `새롭게 할당된 데이터 노드`에 대해 샤드를 재할당하는 방식 결정
        1. all (default) - 모든 샤드들에게 할당을 허용
        2. none - 샤드가 할당되지 않도록 설정
        3. primaries - 프라이머리 샤드만 할당되도록 설정 
        4. new_primaries - 새롭게 생성되는 인덱스의 프라이머리 샤드만 할당되도록 설정
        5. null - default
    * D. `Routing Rebalance` - `index.routing.rebalance.enable`을 이용하여 `데이터 노드`에 샤드를 어떤 방식으로 재배치할지를 결정
        1. all (default) - 모든 샤드들에게 재배치 허용 
        2. none - 샤드가 재배치되지 않도록 설정
        3. primaries - 프라이머리 샤드만 재배치되도록 설정 
        4. replicas - 리플리카 샤드만 재배치되도록 설정 
        5. null - default로 설정

### ES Mapping
* 의미
    * ES의 문서 스키마(6.x부터 1 index - single mapping만 가능)
        1. Dynamic Mapping - 스키마가 없을 때 인입되는 문서를 보고 자동으로 매핑을 만듦
        2. Static Mapping - 사용자가 정의한 스키마를 기준으로 매핑
* 응용
    * `template`을 이용하여 인덱스가 생성될 때 사용자 정의된 세팅이나 매핑을 자동으로 적용 가능
        1. 인덱스 패턴, 인덱스 세팅, 인덱스 매핑 관련 사항 정의
        2. 인덱스가 생성될 때 패턴이 매칭되는 인덱스는 해당 정의
        3. order 가 높은 번호가 낮은 번호를 override 하여 merging

### ES 고급 설정 - Hot Node / Warm Node
* 의미
    * 최근 데이터를 더 자주 보는 경향을 이용한 매커니즘으로 최근 데이터는 고성능 HOT 노드로 오래된 데이터는 저렴한 Warm 노드로 이동
* 방식 - elasticsearch.yml 파일, Template, Curator를 이용하여 운영
    1. elasticsearch.yml 파일의 `node.attr.box_type`를 `hot` or `warm`으로 변경
    2. 템플릿의 `index.routing.allocation.require.box_type` 활용하여 새로 생성되는 인덱스의 샤드를 hot 쪽으로만 할당
    3. 일정 시간이 지나면 hot 노드를 warm 노드로 재할당(Curator로 자동화)


### ES 고급 API
* Cluster API
* Reindex API
* Bulk API