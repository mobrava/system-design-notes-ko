# 18장: Google Maps

## 소개

**Google Maps**의 간단한 버전을 설계한다.

Google Maps에 대한 몇 가지 사실:
 * 2005년에 시작했다.
 * 위성 이미지, 도로 지도, 실시간 교통 상황, 경로 계획 등 다양한 서비스를 제공한다.
 * 2021년 기준 일일 활성 사용자 수는 10억 명이고, 전 세계 99%를 서비스 범위에 포함하며, 실시간 위치 정보를 매일 2,500만 건 업데이트한다.

---

## 1단계: 문제 이해 및 설계 범위 설정

지원자와 면접관의 예시 질의응답:
 * 지원자: 일일 활성 사용자 수는 몇 명인가?
 * 면접관: 일일 활성 사용자 10억 명
 * 지원자: 어떤 기능에 중점을 두어야 하는가?
 * 면접관: 위치 업데이트, 내비게이션, ETA, 지도 렌더링
 * 지원자: 도로 데이터의 크기는 어느 정도인가? 이 데이터에 접근할 수 있는가?
 * 면접관: 다양한 소스에서 확보한 원시 도로 데이터가 있으며, 그 크기는 수 TB이다.
 * 지원자: 교통 상황을 고려해야 할까?
 * 면접관: 그렇다. 시간을 정확히 추정하려면 교통 상황을 고려해야 한다.
 * 지원자: 도보, 자전거, 자동차 등 다양한 이동 수단은 어떠한가?
 * 면접관: 모두 지원해야 한다.
 * 지원자: 경유지가 여러 개인 경로 안내는 어떠한가?
 * 면접관: 면접 범위에서는 다루지 말자.
 * 지원자: 사업체 정보와 사진은 어떠한가?
 * 면접관: 좋은 질문이지만 고려할 필요는 없다.

사용자 위치 업데이트, ETA를 포함한 내비게이션 서비스, 지도 렌더링이라는 세 가지 주요 기능에 중점을 둔다.

### **비기능 요구 사항**

- **정확성**: 사용자에게 잘못된 경로를 안내해서는 안 된다.
- **원활한 내비게이션**: 사용자에게 지도가 매끄럽게 렌더링되어야 한다.
- **데이터 및 배터리 사용량**: 클라이언트는 데이터와 배터리를 최대한 적게 사용해야 한다. 모바일 기기에서는 특히 중요하다.
- 일반적인 가용성 및 확장성 요구 사항

### **지도 기초**

설계를 시작하기 전에 몇 가지 지도 관련 개념을 이해해야 한다.

#### 위치 결정 시스템

지구는 자전축을 중심으로 회전하는 구형이다. 위치는 위도(얼마나 북쪽이나 남쪽에 있는지)와 경도(얼마나 동쪽이나 서쪽에 있는지)로 정의한다.

<div style="margin-left:3rem">
    <img src="./images/partitioning-system.png" alt="위치 결정 시스템" width="500" />
</div>

#### 3D에서 2D로 변환

3차원 공간의 점을 2차원 평면으로 옮기는 과정을 "지도 투영"이라고 한다.

이를 수행하는 방법에는 여러 가지가 있으며 각 방법에는 장단점이 있다. 거의 모두 실제 형상을 왜곡한다.

<div style="margin-left:3rem">
    <img src="./images/map-projections.png" alt="지도 투영법" width="500" />
</div>

Google Maps는 "Web Mercator"라는 Mercator 투영의 수정된 버전을 선택했다.

#### 지오코딩

지오코딩은 주소를 지리적 좌표로 변환하는 과정이다.

반대 과정을 "역지오코딩"이라고 한다.

이를 구현하는 한 가지 방법은 도로망을 지리 좌표 공간에 매핑한 여러 소스(예: GIS)의 데이터를 활용해 보간하는 것이다.

#### 지오해싱

지오해싱(Geohashing)은 지리적 영역을 문자와 숫자로 이루어진 문자열로 표현하는 인코딩 시스템이다.

지오해싱은 세계를 평면으로 나타내고 이를 재귀적으로 네 개의 사분면으로 세분한다.

<div style="margin-left:3rem">
    <img src="./images/geohashing.png" alt="지오해싱" width="500" />
</div>

#### 지도 렌더링

지도 렌더링은 타일링을 통해 이루어진다. 전체 지도를 하나의 큰 맞춤형 이미지로 렌더링하는 대신 세계를 더 작은 타일로 나눈다.

클라이언트는 필요한 타일만 다운로드한 뒤 모자이크를 이어 붙이듯 렌더링한다.

확대/축소 수준마다 서로 다른 타일이 있다. 클라이언트는 현재 확대/축소 수준에 맞는 타일을 선택한다.

예를 들어, 전 세계를 축소하면 전 세계를 나타내는 단일 256x256 타일만 다운로드된다.

#### 내비게이션 알고리즘을 위한 도로 데이터 처리

대부분의 라우팅 알고리즘에서 교차로는 노드로, 도로는 간선으로 표현된다.

<div style="margin-left:3rem">
    <img src="./images/road-representation.png" alt="도로 표현" width="500" />
</div>

대부분의 내비게이션 알고리즘은 Dijkstra 또는 A* 알고리즘을 변형해 사용한다.

경로 탐색 성능은 그래프 크기에 민감하다. 대규모 환경에서는 전 세계를 하나의 그래프로 표현해 그 위에서 알고리즘을 실행할 수 없다.

대신 타일링과 유사한 기술을 사용해 세계를 점점 더 작은 그래프로 세분한다.

라우팅 타일은 인접 타일에 대한 참조를 보유하며, 알고리즘은 서로 연결된 타일을 탐색하면서 더 큰 도로 그래프를 이어 붙일 수 있다.

<div style="margin-left:3rem">
    <img src="./images/routing-tiles.png" alt="라우팅 타일" width="500" />
</div>

이 기술을 사용하면 메모리 대역폭을 크게 줄이고 주어진 출발지/목적지 쌍에 필요한 타일만 로드할 수 있다.

그러나 긴 경로에서는 작고 상세한 라우팅 타일을 이어 붙이는 데 여전히 시간과 메모리가 많이 든다. 따라서 세부 수준이 서로 다른 라우팅 타일을 두고, 알고리즘은 목적지에 따라 적절한 세부 수준의 타일을 사용한다.

<div style="margin-left:3rem">
    <img src="./images/map-routing-hierarchical.png" alt="계층적 지도 라우팅" width="500" />
</div>

### **개략적 추정**

다음 데이터를 저장해야 한다.
 * 전 세계 지도 - 저장해야 할 모든 타일을 기준으로 ~70pb로 추정하지만, 매우 유사한 타일(예: 광활한 사막)은 압축할 수 있다는 점을 고려한다.
 * 메타데이터 - 크기를 무시할 수 있으므로 계산에서 제외한다.
 * 도로 정보 - 라우팅 타일로 저장한다.

내비게이션 요청의 예상 QPS 계산은 다음과 같다(1bil DAU at 35min of usage per week -> 5bil minutes per day).
GPS 업데이트 요청을 배치 처리한다고 가정하면 평상시 200k QPS, 최대 부하에서는 1mil QPS가 된다.

---

## 2단계: 개략적 설계 제안 및 승인 받기

<div style="margin-left:3rem">
    <img src="./images/high-level-design.png" alt="개략적 설계" width="500" />
</div>

### **위치 서비스**

<div style="margin-left:3rem">
    <img src="./images/location-service.png" alt="위치 서비스" width="500" />
</div>

사용자의 위치 업데이트를 기록하는 역할을 담당한다.
 * 위치 업데이트는 `t`초마다 전송된다.
 * 위치 데이터 스트림은 시간이 지남에 따라 서비스를 개선하는 데 사용될 수 있다(예: 보다 정확한 ETA 제공, 교통 데이터 모니터링, 폐쇄된 도로 감지, 사용자 행동 분석 등).

위치 업데이트를 매번 서버로 보내는 대신 클라이언트 측에서 여러 업데이트를 모아 배치로 전송할 수 있다.

<div style="margin-left:3rem">
    <img src="./images/location-update-batches.png" alt="위치 업데이트 배치" width="500" />
</div>

이러한 최적화에도 Google Maps 규모의 시스템에서는 부하가 여전히 상당하다. 따라서 Cassandra처럼 쓰기 부하가 큰 워크로드에 최적화된 데이터베이스를 활용할 수 있다.

또한 추가 분석을 위한 위치 업데이트의 효율적인 스트림 처리를 위해 Kafka를 활용할 수도 있다.

위치 업데이트 요청 페이로드 예시:

```
POST /v1/locations
Parameters
  locs: JSON encoded array of (latitude, longitude, timestamp) tuples.
```

### **내비게이션 서비스**

이 컴포넌트는 합리적인 시간 안에 A와 B 사이의 빠른 경로를 찾는다(지연 시간이 약간 발생해도 괜찮다). 경로가 반드시 가장 빠를 필요는 없지만 정확성이 중요하다.

요청 페이로드 예시:

```
GET /v1/nav?origin=1355+market+street,SF&destination=Disneyland
```

예시 응답:

```json
{
  "distance": {"text":"0.2 mi", "value": 259},
  "duration": {"text": "1 min", "value": 83},
  "end_location": {"lat": 37.4038943, "Ing": -121.9410454},
  "html_instructions": "Head <b>northeast</b> on <b>Brandon St</b> toward <b>Lumin Way</b><div style=\"font-size:0.9em\">Restricted usage road</div>",
  "polyline": {"points": "_fhcFjbhgVuAwDsCal"},
  "start_location": {"lat": 37.4027165, "lng": -121.9435809},
  "geocoded_waypoints": [
    {
       "geocoder_status" : "OK",
       "partial_match" : true,
       "place_id" : "ChIJwZNMti1fawwRO2aVVVX2yKg",
       "types" : [ "locality", "political" ]
    },
    {
       "geocoder_status" : "OK",
       "partial_match" : true,
       "place_id" : "ChIJ3aPgQGtXawwRLYeiBMUi7bM",
       "types" : [ "locality", "political" ]
    }
  ],
  "travel_mode": "DRIVING"
}
```

교통 상황의 변화와 경로 재탐색은 아직 고려하지 않았으며 상세 설계에서 다룬다.

### **지도 렌더링**

지도 타일의 전체 데이터 세트는 페타바이트 규모이므로 클라이언트에 모두 저장하는 것은 현실적이지 않다.

클라이언트의 위치와 확대/축소 수준에 따라 필요할 때 서버에서 가져와야 한다.

사용자가 확대 또는 축소할 때와 내비게이션 중 새로운 타일을 향해 이동할 때 새 타일을 가져와야 한다.

지도 타일은 클라이언트에게 어떻게 제공되어야 하는가?
 * 동적으로 생성할 수 있지만 서버에 막대한 부하를 주고 캐싱도 어렵게 만든다.
 * 지도 타일은 클라이언트가 계산할 수 있는 지오해시를 기준으로 정적으로 제공한다. CDN에 정적으로 저장해 제공할 수 있다.

<div style="margin-left:3rem">
    <img src="./images/static-map-tiles.png" alt="정적 지도 타일" width="500" />
</div>

CDN을 사용하면 지연 시간을 최소화하도록 사용자와 가장 가까운 접속 지점(Point of Presence, POP) 서버에서 지도 타일을 가져올 수 있다.

<div style="margin-left:3rem">
    <img src="./images/cdn-vs-no-cdn.png" alt="CDN 사용 여부 비교" width="500" />
</div>

지도 타일을 결정할 때 고려해야 할 옵션:
 * 지도 타일의 지오해시는 클라이언트 측에서 계산할 수 있다. 다만 클라이언트의 업데이트를 강제하기 어렵기 때문에, 이 지도 타일 계산 방식을 장기간 유지하기로 결정할 때는 신중해야 한다.
 * 또는 API 호출이 하나 더 필요하다는 대가로 클라이언트를 대신해 지도 타일 URL을 계산하는 간단한 API를 둘 수 있다.

<div style="margin-left:3rem">
    <img src="./images/map-tile-url-calculation.png" alt="지도 타일 URL 계산" width="500" />
</div>

---

## 3단계: 상세 설계

### **데이터 모델**

다루어야 할 여러 유형의 데이터를 저장하는 방법을 살펴본다.

#### 라우팅 타일

초기 도로 데이터 세트는 다양한 소스에서 얻으며, 위치 업데이트 데이터를 바탕으로 시간이 지나면서 개선한다.

도로 데이터는 비정형이다. 이 원시 데이터를 앱에 필요한 그래프 기반 라우팅 타일로 변환하는 주기적인 오프라인 처리 파이프라인을 둔다.

데이터베이스 기능이 필요하지 않으므로 이 타일을 데이터베이스에 저장하지 않는다. 대신 S3 객체 스토리지에 저장하고 적극적으로 캐싱할 수 있다.

또한 라이브러리를 활용해 인접 리스트를 바이너리 파일로 효율적으로 압축할 수 있다.

#### 사용자 위치 데이터

사용자 위치 데이터는 교통 상황을 갱신하고 여러 분석을 수행하는 데 매우 유용하다.

이 데이터는 쓰기 작업이 많으므로 Cassandra에 저장할 수 있다.

예시 행:

<div style="margin-left:3rem">
    <img src="./images/user-location-data-torw.png" alt="사용자 위치 데이터 행" width="500" />
</div>

#### 지오코딩 데이터베이스

이 데이터베이스는 위도/경도 쌍과 장소를 연결하는 키-값 쌍을 저장한다.

읽기는 잦고 쓰기는 드물기 때문에 빠른 읽기 액세스 속도를 위해 Redis를 사용할 수 있다.

#### 세계 지도의 미리 계산된 이미지

논의한 대로 지도 타일링 이미지를 미리 계산하여 CDN에 저장한다.

<div style="margin-left:3rem">
    <img src="./images/precomputed-map-tile-image.png" alt="미리 계산된 지도 타일 이미지" width="500" />
</div>

### **서비스**

#### 위치 서비스

이 서비스의 데이터베이스 설계와 사용자 위치 저장 방식을 자세히 살펴본다.

<div style="margin-left:3rem">
    <img src="./images/location-service-diagram.png" alt="위치 서비스 다이어그램" width="500" />
</div>

NoSQL 데이터베이스를 사용해 위치 업데이트의 높은 쓰기 부하를 처리할 수 있다. 사용자 위치 데이터는 자주 바뀌고 새 업데이트가 도착하면 기존 데이터가 낡으므로 일관성보다 가용성을 우선한다.

모든 요구 사항에 잘 맞는 Cassandra를 데이터베이스로 선택한다.

저장하려는 행의 예:

<div style="margin-left:3rem">
    <img src="./images/user-location-row-example.png" alt="사용자 위치 행 예시" width="500" />
</div>

 * `user_id`는 특정 사용자의 모든 위치 업데이트에 빠르게 액세스하기 위한 파티션 키이다.
 * `timestamp`는 위치 업데이트가 수신되는 시간별로 정렬된 데이터를 저장하기 위한 클러스터링 키이다.

또한 Kafka를 활용해 여러 목적으로 위치 업데이트가 필요한 다른 서비스에 데이터를 스트리밍한다.

<div style="margin-left:3rem">
    <img src="./images/location-update-streaming.png" alt="위치 업데이트 스트리밍" width="500" />
</div>

#### 지도 렌더링

지도 타일은 다양한 확대/축소 수준으로 저장된다. 가장 낮은 확대/축소 수준에서는 전 세계가 단일 256x256 타일로 표시된다.

확대/축소 수준이 한 단계 높아질 때마다 지도 타일 수는 네 배로 늘어난다.

<div style="margin-left:3rem">
    <img src="./images/zoom-level-increases.png" alt="확대 수준 증가" width="500" />
</div>

한 가지 최적화 방법은 전체 이미지 정보를 네트워크로 전송하는 대신 타일을 벡터(경로 및 다각형)로 표현하고 클라이언트가 타일을 동적으로 렌더링하게 하는 것이다.

이렇게 하면 상당한 대역폭이 절약된다.

#### 내비게이션 서비스

이 서비스는 가장 빠른 경로를 찾는 역할을 한다.

<div style="margin-left:3rem">
    <img src="./images/navigation-service.png" alt="내비게이션 서비스" width="500" />
</div>

이 하위 시스템의 각 컴포넌트를 살펴본다.

먼저 주소를 위도/경도 좌표로 변환하는 지오코딩 서비스가 있다.

요청 예시:

```
https://maps.googleapis.com/maps/api/geocode/json?address=1600+Amphitheatre+Parkway,+Mountain+View,+CA
```

예시 응답:

```json
{
   "results" : [
      {
         "formatted_address" : "1600 Amphitheatre Parkway, Mountain View, CA 94043, USA",
         "geometry" : {
            "location" : {
               "lat" : 37.4224764,
               "lng" : -122.0842499
            },
            "location_type" : "ROOFTOP",
            "viewport" : {
               "northeast" : {
                  "lat" : 37.4238253802915,
                  "lng" : -122.0829009197085
               },
               "southwest" : {
                  "lat" : 37.4211274197085,
                  "lng" : -122.0855988802915
               }
            }
         },
         "place_id" : "ChIJ2eUgeAK6j4ARbn5u_wAGqWA",
         "plus_code": {
            "compound_code": "CWC8+W5 Mountain View, California, United States",
            "global_code": "849VCWC8+W5"
         },
         "types" : [ "street_address" ]
      }
   ],
   "status" : "OK"
}
```

경로 플래너 서비스는 현재 교통 상황에 따라 이동 시간에 최적화된 추천 경로를 계산한다.

최단 경로 서비스는 객체 스토리지의 라우팅 타일에 변형된 A* 알고리즘을 실행해 최적 경로를 계산한다.
 * 출발지/목적지 쌍을 받아 위도/경도 쌍으로 변환하고, 여기서 지오해시를 구해 라우팅 타일을 결정한다.
 * 알고리즘은 최초 라우팅 타일에서 시작해 목적지 타일까지 충분히 좋은 경로를 찾을 때까지 탐색한다.

<div style="margin-left:3rem">
    <img src="./images/shortest-path-service.png" alt="최단 경로 서비스" width="500" />
</div>

경로 플래너는 ETA 서비스를 호출해 머신러닝 알고리즘과 교통 데이터를 바탕으로 예상 도착 시간을 구한다.

순위 지정 서비스는 사용자가 전달한 필터(예: 유료 도로나 고속도로를 피하는 플래그)를 바탕으로 여러 후보 경로의 순위를 매긴다.

업데이트 서비스는 일부 주요 데이터베이스를 비동기로 갱신해 최신 상태로 유지한다.

#### 개선 - 적응형 ETA 및 경로 재탐색

한 가지 개선 방법은 새로 들어온 교통 데이터를 바탕으로 진행 중인 경로를 적응적으로 갱신하는 것이다.

이를 구현하는 한 가지 방법은 현재 내비게이션 중인 사용자와 해당 경로에서 지나야 할 모든 타일을 데이터베이스에 저장하는 것이다.

데이터는 다음과 같은 형태다.

```
user_1: r_1, r_2, r_3, …, r_k
user_2: r_4, r_6, r_9, …, r_n
user_3: r_2, r_8, r_9, …, r_m
...
user_n: r_2, r_10, r21, ..., r_l
```

어떤 타일에서 교통사고가 발생하면 경로가 해당 타일을 통과하는 모든 사용자를 식별하고 경로를 다시 탐색할 수 있다.

데이터베이스에 저장하는 타일 수를 줄이려면 출발지 라우팅 타일부터 목적지 타일이 포함되는 수준까지 해상도가 서로 다른 여러 라우팅 타일을 저장할 수 있다.

```
user_1, r_1, super(r_1), super(super(r_1)), ...
```

<div style="margin-left:3rem">
    <img src="./images/adaptive-eta-data-storage.png" alt="적응형 ETA 데이터 저장소" width="500" />
</div>

이 구조에서는 사용자의 최종 타일에 교통사고가 발생한 타일이 포함되는지만 확인하면 사용자의 영향 여부를 알 수 있다.

또한 내비게이션 중인 사용자의 가능한 모든 경로를 추적하고, 더 빠른 경로로 다시 탐색할 수 있으면 이를 알려줄 수도 있다.

#### 전송 프로토콜

서버에서 클라이언트로 데이터를 선제적으로 푸시할 수 있는 몇 가지 옵션이 있다.
 * 모바일 푸시 알림은 페이로드가 제한적이고 웹 앱에서 사용할 수 없으므로 적합하지 않다.
 * WebSocket은 일반적으로 서버의 컴퓨팅 자원을 덜 사용하므로 롱 폴링보다 나은 선택이다.
 * 서버 전송 이벤트(SSE)도 사용할 수 있지만 WebSocket을 선호한다. WebSocket은 라스트 마일 배송 기능 등에 유용한 양방향 통신을 지원하기 때문이다.

---

## 4단계: 마무리

최종 설계는 다음과 같다.

<div style="margin-left:3rem">
    <img src="./images/final-design.png" alt="최종 설계" width="500" />
</div>

추가로 제공할 수 있는 기능 중 하나는 여러 위치를 방문하는 최적 경로를 결정하는 다중 경유지 내비게이션이다. 이 기능은 Uber나 Lyft 같은 기업 고객에게 판매할 수 있다.
