# ES 온보딩 교육 1회차

<hr>

### ES
* ES는 풀 텍스트 검색엔진이자 분석엔진 오픈소스
* `Lucene(루씬)`
    * 자바 기반의 고성능 정보 검색 라이브러리
    * API를 통해 전문 색인(full-text) 색인과 검색 기능 사용 가능
* 검색엔진으로서의 ES
    * Lucene 라이브러리를 기반으로 만든 검색 엔진
    * RESTFul API와 JSON 형태의 문서를 지원
    * 루씬이 가진 기능을 제공 하면서도 분산처리를 비롯한 대용량 데이터를 처리하기 위한 다양한 기능을 가짐
* 분석엔진으로서의 ES
    * Beats, Logstash, Kibana를 통해 분석 엔진으로 사용 가능
    * `Beats`: 데이터 수집기용 플랫폼으로 ES나 Logstash로 데이터를 전송하는 도구
    * `Logstash`: 필터링한 데이터(로그)를 ES에 문서로 저장하는 도구 
    * `Kibana`: 수집된 데이터를 통계/집계하는 웹 기반 시각화 도구

### ES 용어 및 개념 정리
* `필드`
    * 문서를 구성하기 위힌 속성
    * 동적인 데이터 타입을 지원하여 필드 안에 여러 속성이 들어갈 수 있음
* `문서`
    * JSON 형태의 Key-Value 쌍으로 실제 의미 있는 데이터를 가진 ES 저장의 기본 단위
    * 문서를 ES에 저장할 때 설정하거나 랜덤으로 고유한 ID를 가짐
    * 문서 ID가 문서 데이터를 찾아가는 열쇠
* `타입`
    * 인덱스의 논리적 구조를 의미
    * 1 Index - 1 Type이 된 현재는 타입을 분류 목적으로 사용하지 않고 인덱스를 분류 목적으로 사용
    * 1 Index - 1 Type이므로 한개의 테이블만 존재하는 데이터베이스라고 생각하면 됨
    * ES 7.x부터 _doc으로 이름 고정
* `인덱스`
    * 문서가 저장되는 가장 큰 단위
    * 하나의 인덱스에 여러 문서가 저장
    * 빠른 검색을 위해 데이터를 색인화(정렬 및 분류) 하는 것을 `인덱싱`이라고 함
    * `인덱싱`을 할 경우 Primary Shard가 제일 먼저 쓰여지고 이후 Replica Shard가 복제
* `노드`
    * 물리적인 인스턴스
    * node_name과 node_uuid를 가짐
    * 마스터 노드: 클러스터를 관리하는 노드
    * 데이터 노드: 실질적인 데이터를 저장하고 검색과 통계 관련 작업 수행하는 노드
    * 인제스트 노드: 색인에 앞서 전처리를 하기 위한 노드
    * 클라이언트 노드(코디네이팅 노드): 사용자의 쿼리를 받기 위한 노드로 데이터 노드가 리턴해준 데이터를 사용자에게 리턴하는 노드
* `클러스터`
    * 여러 개의 노드를 하나의 논리적인 개념으로 묶어 클러스터라고 부름
    * 클러스터를 대상으로 데이터를 저장을 요청하거나 검색 요청을 함
    * 클러스터 내에서 요청을 노드로 보낼 때 분산 처리함
    * cluster_name과 cluster_uuid를 가짐
* `샤드`
    * 물리적 공간에 존재하는 여러 개의 논리적인 파티션으로 `루씬`의 인덱스 역할을 함
    * 인덱스의 데이터를 샤드 단위로 나누고 복제 및 분산 저장하여 검색의 병렬성을 높이고 안정성을 추구
    * 인덱스를 샤드로 나눠 수평 분할하는 것을 `샤딩`이라고 함
    * 샤드 할당 알고리즘 = `shard = hash(routing) % number_of_primary_shards`
    * `원본 샤드(Primary Shard)`: 색인되어 저장되는 문서의 원본 샤드로 색인 생성 시점에 정의되고 이후 개수를 변경할 수 없음(기본 Primary Shard는 1개)
    * `복제본 샤드(Replica Shard)`: 색인되어 들어온 문서의 원본 샤드에 대한 복제본 샤드로 복제본의 개수를 변경 가능
    * Fail이 나는 경우 복제본 샤드를 원본 샤드로 승격
* `세그먼트`
    * 샤드 보다 더 작은 단위로 문서가 저장되는 최소의 단위로 물리적인 단위
    * 문서는 먼저 메모리(페이지 캐시/버퍼 캐시)에 저장되고 이후 Refresh 과정에서 디스크에 파일로 저장
    * 저장된 문서는 immutable 속성을 가져 update 할 수 없고 update 시엔 copy 이후 update
    * 백그라운드에서 세그먼트들의 병합을 진행

### ES와 관계형 데이터베이스 비교
| 엘라스틱서치 | 관계형 데이터베이스 |
|:--------|:--------|
| 인덱스 | 데이터베이스 |
| 매핑 | 스키마 |
| 타입 | 테이블 |
| 문서 | 행(레코드) |
| 필드 | 열(필드) |
| 샤드 | 파티션 |
| Query DSL | SQL |

### ES 설치하기
* YUM을 통해 설치
* RPM을 다운로드하여 설치
    * `/etc/sysconfig/`에 ES 환경변수 파일 존재 - `elasticsearch`
    * `/etc/elasticsearch`에 ES 환경변수 파일 존재 - `elasticsearch.yml` / `jvm.options` / `log4j2.properties` ...
* zip or tar를 다운로드하여 설치

### ES 실행하기 및 실행 확인하기
* 실행
    * init 
    ```bash
    sudo -i service elasticsearch start
    ```

    * systemd
    ```bash
    $ sudo systemctl start elasticsearch.service
    ```

* 재시작
    * init 
    ```bash
    sudo -i service elasticsearch restart
    ```

    * systemd
    ```bash
    $ sudo systemctl restart elasticsearch.service
    ```

* 종료
    * init 
    ```bash
    sudo -i service elasticsearch stop
    ```

    * systemd
    ```bash
    $ sudo systemctl stop elasticsearch.service
    ```

* 실행 확인하기
    * 프로세스 확인
    ```bash
    $ ps -ef | grep elasticsearch
    ```
    * 어플리케이션 반응 확인
    ```bash
    $ curl localhost:9200
    ```

### Kibana 설치 및 Dev Tools 활용하기
* YUM을 통해 설치하기
    * `/etc/kibana/`에 Kibana 환경변수 파일 존재 - `kibana.yml`
* Kibana 웹을 통해서 쿼리를 날릴 수 있는 Dev Tools가 존재

### 개념 보충
* 페이지 캐시: 디스크 접근을 최소화 하여 파일 I/O성능을 향상시키기 위해 사용되는 메모리 영역으로 한 번 읽은 파일의 내용을 이 페이지 캐시 영역에 저장하고 같은 파일의 접근이 일어나면 디스크에서 읽어오는 것이 아니라 페이지 캐시에서 읽음
* 버퍼 캐시: 디스크의 블록 단위로 데이터를 전송하는 블록 디바이스가 가지고 있는 블록 자체에 대한 캐시로 커널이 특정 블록에 접근하면 블록의 내용을 버퍼 캐시 영역에 저장하고 동일한 블록에 접근할 시에 버퍼 캐시에서 읽음(현재는 페이지 캐시가 버퍼 캐시 내에 포함되어 있음)
* init와 systemd 차이: RHEL7/CentOS7로 올라오면서 초기화 프로세스를 init 프로세스에서 systemd로 바꾸었음