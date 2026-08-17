# 25장: 실시간 게임 순위표

## 소개

온라인 모바일 게임의 **순위표**를 설계한다.

<div style="margin-left:3rem">
    <img src="./images/leaderboard.png" alt="순위표" width="500" />
</div>

---

## 1단계: 문제 이해 및 설계 범위 확정

- 지원자: 순위표의 점수는 어떻게 계산하는가?
- 면접관: 사용자가 경기에서 승리할 때마다 1점을 얻는다.
- 지원자: 모든 플레이어가 순위표에 포함되는가?
- 면접관: 그렇다.
- 지원자: 순위표와 연관된 기간 구간이 있는가?
- 면접관: 매월 새로운 토너먼트가 시작되며, 이에 따라 새로운 순위표가 시작된다.
- 지원자: 상위 10명의 사용자만 고려하면 된다고 가정해도 되는가?
- 면접관: 상위 10명의 사용자와 특정 사용자의 순위를 함께 표시하려 한다. 시간이 허락한다면 순위표에서 특정 사용자 주변의 사용자들을 보여 주는 방안도 논의할 수 있다.
- 지원자: 토너먼트에는 사용자가 몇 명이나 참가하는가?
- 면접관: 500만 DAU와 2500만 MAU이다.
- 지원자: 토너먼트 기간에 평균적으로 경기가 몇 번 진행되는가?
- 면접관: 각 플레이어는 하루 평균 10경기를 한다.
- 지원자: 두 플레이어의 점수가 같으면 순위를 어떻게 결정하는가?
- 면접관: 이 경우 순위는 같다. 시간이 허락한다면 동점 처리 방안을 논의할 수 있다.
- 지원자: 순위표가 실시간이어야 하는가?
- 면접관: 그렇다. 실시간 결과 또는 최대한 실시간에 가까운 결과를 보여 주고자 한다. 일괄 처리된 결과 이력을 보여 주는 것은 허용되지 않는다.

### **기능 요구사항**

- 순위표에 상위 10명의 플레이어를 표시한다.
- 특정 사용자의 순위를 보여 준다.
- 주어진 사용자의 위아래 네 순위에 있는 사용자들을 표시한다(추가 요구사항).

### **비기능 요구사항**

- 점수를 실시간으로 갱신한다.
- 점수 갱신이 순위표에 실시간으로 반영된다.
- 전반적인 확장성, 가용성, 신뢰성을 보장한다.

### **개략적 규모 추정**

5000만 DAU가 24시간 동안 고르게 분포한다고 가정하면 초당 평균 사용자는 50명이다.
하지만 일반적으로 분포가 고르지 않으므로 피크 시간대의 온라인 사용자는 초당 250명으로 추정할 수 있다.

사용자가 1점을 얻는 요청의 QPS는 하루 평균 10경기를 기준으로 다음과 같이 계산한다. 50 users/s * 10 = 500 QPS. Peak QPS = 2500.

상위 10명의 순위표를 조회하는 QPS는 사용자가 하루 평균 한 번 연다고 가정하면 50이다.

---

## 2단계: 개략적 설계안 제시 및 동의 구하기

### **API 설계**

첫 번째로 필요한 API는 사용자의 점수를 갱신하는 API이다.

```
POST /v1/scores
```

이 API는 `user_id`와 경기 승리로 얻은 `points`라는 두 매개변수를 받는다.

이 API에는 최종 클라이언트가 아닌 게임 서버만 접근할 수 있어야 한다.

다음 API는 순위표의 상위 10명 플레이어를 조회한다.

```
GET /v1/scores
```

응답 예시는 다음과 같다.

```
{
  "data": [
    {
      "user_id": "user_id1",
      "user_name": "alice",
      "rank": 1,
      "score": 12543
    },
    {
      "user_id": "user_id2",
      "user_name": "bob",
      "rank": 2,
      "score": 11500
    }
  ],
  ...
  "total": 10
}
```

특정 사용자의 점수도 조회할 수 있다.

```
GET /v1/scores/{:user_id}
```

응답 예시는 다음과 같다.

```
{
    "user_info": {
        "user_id": "user5",
        "score": 1000,
        "rank": 6,
    }
}
```

### **개략적 아키텍처**

<div style="margin-left:3rem">
    <img src="./images/high-level-architecture.png" alt="개략적 아키텍처" width="500" />
</div>

- 플레이어가 경기에서 승리하면 클라이언트가 게임 서비스에 요청을 보낸다.
- 게임 서비스는 승리가 유효한지 검증하고 순위표 서비스를 호출해 플레이어의 점수를 갱신한다.
- 순위표 서비스는 순위표 저장소에서 사용자의 점수를 갱신한다.
- 플레이어는 순위표 서비스를 호출해 상위 10명의 플레이어와 해당 플레이어의 순위 같은 순위표 데이터를 조회한다.

고려할 수 있는 대안 설계는 클라이언트가 순위표 서비스에서 자신의 점수를 직접 갱신하는 방식이다.

<div style="margin-left:3rem">
    <img src="./images/alternative-design.png" alt="대안 설계" width="500" />
</div>

이 방식은 중간자 공격에 취약하므로 안전하지 않다. 플레이어가 프록시를 두고 원하는 대로 자신의 점수를 변경할 수 있다.

한 가지 추가로 유의할 점은 게임 로직을 서버에서 관리하는 게임의 경우, 클라이언트가 승리를 기록하기 위해 서버를 명시적으로 호출할 필요가 없다는 것이다.
서버가 게임 로직에 따라 이를 자동으로 처리한다.

추가로 게임 서버와 순위표 서비스 사이에 메시지 큐를 둘지 고려해야 한다. 다른 서비스도 게임 결과에 관심이 있다면 유용하지만, 현재까지 면접에서 명시적으로 요구하지 않았으므로 설계에는 포함하지 않는다.

<div style="margin-left:3rem">
    <img src="./images/message-queue-based-comm.png" alt="메시지 큐 기반 통신" width="500" />
</div>

### **데이터 모델**

순위표 데이터를 저장할 수 있는 관계형 DB, Redis, NoSQL 방안을 논의한다.

NoSQL 방안은 상세 설계 절에서 다룬다.

#### 관계형 데이터베이스 방안

규모가 중요하지 않고 사용자 수가 많지 않다면 관계형 DB가 요구사항을 충분히 충족한다.

매월 하나씩 두는 간단한 순위표 테이블에서 시작할 수 있다(개인 메모: 이 방식은 타당하지 않다. 단순히 `month` 열을 추가하면 매월 새 테이블을 관리해야 하는 번거로움을 피할 수 있다).

<div style="margin-left:3rem">
    <img src="./images/leaderboard-table.png" alt="순위표 테이블" width="500" />
</div>

여기에 포함할 추가 데이터가 있지만 실행할 쿼리와는 무관하므로 생략한다.

사용자가 1점을 얻으면 어떻게 되는가?

<div style="margin-left:3rem">
    <img src="./images/user-wins-point.png" alt="사용자가 점수를 얻는 과정" width="500" />
</div>

사용자가 아직 테이블에 없다면 먼저 삽입해야 한다.

```
INSERT INTO leaderboard (user_id, score) VALUES ('mary1934', 1);
```

이후 호출부터는 점수만 갱신한다.

```
UPDATE leaderboard set score=score + 1 where user_id='mary1934';
```

순위표의 상위 플레이어를 어떻게 찾는가?

<div style="margin-left:3rem">
    <img src="./images/find-leaderboard-position.png" alt="순위표 순위 찾기" width="500" />
</div>

다음 쿼리를 실행할 수 있다.

```
SELECT (@rownum := @rownum + 1) AS rank, user_id, score
FROM leaderboard
ORDER BY score DESC;
```

하지만 데이터베이스 테이블의 모든 레코드를 정렬하기 위해 테이블 스캔을 수행하므로 성능이 좋지 않다.

`score`에 인덱스를 추가하고 `LIMIT` 연산을 사용해 전체 스캔을 피하도록 최적화할 수 있다.

```
SELECT (@rownum := @rownum + 1) AS rank, user_id, score
FROM leaderboard
ORDER BY score DESC
LIMIT 10;
```

하지만 이 접근법은 사용자가 순위표 상단에 있지 않고 그 순위를 찾으려 할 때 확장성이 좋지 않다.

#### Redis 방안

복잡한 데이터베이스 쿼리에 의존하지 않으면서 수백만 명의 플레이어에게도 잘 작동하는 방안을 찾아야 한다.

Redis는 메모리에서 동작하므로 빠르며, 요구사항에 적합한 자료 구조인 정렬 집합(sorted set)을 제공한다.

정렬 집합은 프로그래밍 언어의 집합과 비슷하지만, 주어진 기준에 따라 자료 구조를 정렬된 상태로 유지할 수 있다.
내부적으로는 키(user_id)와 값(score)의 매핑을 유지하는 해시 맵과, 점수에 따라 사용자를 정렬된 순서로 매핑하는 스킵 리스트(skip list)로 구현된다.

<div style="margin-left:3rem">
    <img src="./images/sorted-set.png" alt="정렬 집합" width="500" />
</div>

스킵 리스트는 어떻게 작동하는가?
- 빠른 검색을 지원하는 연결 리스트이다.
- 정렬된 연결 리스트와 다단계 인덱스로 구성된다.

<div style="margin-left:3rem">
    <img src="./images/skip-list.png" alt="스킵 리스트" width="500" />
</div>

이 구조를 사용하면 데이터 집합이 충분히 클 때 특정 값을 빠르게 검색할 수 있다.
아래 예시에서는 노드가 64개일 때 주어진 값을 찾기 위해 기본 연결 리스트에서는 노드 62개를 순회해야 하지만, 스킵 리스트에서는 노드 11개만 순회하면 된다.

<div style="margin-left:3rem">
    <img src="./images/skip-list-performance.png" alt="스킵 리스트 성능" width="500" />
</div>

정렬 집합은 추가 및 검색 연산에 O(logN)의 비용을 들여 데이터를 항상 정렬된 상태로 유지하므로 관계형 데이터베이스보다 성능이 좋다.

반면 관계형 DB에서 특정 사용자의 순위를 찾으려면 다음과 같은 중첩 쿼리를 실행해야 한다.

```
SELECT *,(SELECT COUNT(*) FROM leaderboard lb2
WHERE lb2.score >= lb1.score) RANK
FROM leaderboard lb1
WHERE lb1.user_id = {:user_id};
```

Redis에서 순위표를 운영하려면 어떤 연산이 필요한가?
- **ZADD** - 사용자가 없다면 집합에 삽입하고, 있다면 점수를 갱신한다. 시간 복잡도는 O(logN)이다.
- **ZINCRBY** - 사용자의 점수를 주어진 양만큼 증가시킨다. 사용자가 없다면 점수는 0에서 시작한다. 시간 복잡도는 O(logN)이다.
- **ZRANGE/ZREVRANGE** - 점수에 따라 정렬된 사용자 범위를 조회한다. 정렬 순서(ASC/DESC), 오프셋, 결과 크기를 지정할 수 있다. 시간 복잡도는 O(logN+M)이며, M은 결과 크기이다.
- **ZRANK/ZREVRANK** - 주어진 사용자의 위치(순위)를 ASC/DESC 순서로 조회한다. 시간 복잡도는 O(logN)이다.

사용자가 1점을 얻으면 어떻게 되는가?

```
ZINCRBY leaderboard_feb_2021 1 'mary1934'
```

매월 새로운 순위표를 만들고 이전 순위표는 이력 저장소로 옮긴다.

사용자가 상위 10명의 플레이어를 조회하면 어떻게 되는가?

```
ZREVRANGE leaderboard_feb_2021 0 9 WITHSCORES
```

결과 예시는 다음과 같다.

```
[(user2,score2),(user1,score1),(user5,score5)...]
```

사용자가 자신의 순위를 조회하는 경우는 어떠한가?

<div style="margin-left:3rem">
    <img src="./images/leaderboard-position-of-user.png" alt="사용자의 순위표 순위" width="500" />
</div>

사용자의 순위표 위치를 알고 있다면 다음 쿼리로 쉽게 조회할 수 있다.

```
ZREVRANGE leaderboard_feb_2021 357 365
```

사용자의 위치는 `ZREVRANK <user-id>`로 조회할 수 있다.

저장 공간 요구사항을 살펴보자.
- 최악의 경우로 해당 월에 2500만 MAU 모두가 게임에 참여한다고 가정한다.
- ID는 24자 문자열이고 점수는 16비트 정수이므로 26 bytes * 25mil = ~650MB의 저장 공간이 필요하다.
- 스킵 리스트의 오버헤드 때문에 저장 비용이 두 배가 되더라도 최신 Redis 클러스터에는 충분히 들어간다.

고려해야 할 또 다른 비기능 요구사항은 초당 2500회의 갱신을 지원하는 것이다. 이는 단일 Redis 서버의 처리 능력으로 충분히 감당할 수 있다.

추가 유의사항은 다음과 같다.
- Redis 서버가 중단될 때 데이터가 손실되지 않도록 Redis 복제본을 가동할 수 있다.
- 장애 발생 시 데이터 손실을 막기 위해 Redis 영속성도 활용할 수 있다.
- 사용자 이름과 표시 이름 같은 사용자 상세 정보를 조회하고, 사용자가 언제 경기에서 승리했는지 등을 저장하기 위해 MySQL에 보조 테이블 두 개가 필요하다.
- MySQL의 두 번째 테이블은 인프라 장애 발생 시 순위표를 재구성하는 데 사용할 수 있다.
- 작은 성능 최적화로 자주 접근하는 상위 10명 플레이어의 사용자 상세 정보를 캐시할 수 있다.

---

## 3단계: 상세 설계

### **클라우드 제공업체 사용 여부**

서비스를 직접 배포하고 관리하거나 클라우드 제공업체가 대신 관리하도록 할 수 있다.

서비스를 직접 관리한다면 순위표 데이터에는 Redis를, 사용자 프로필에는 MySQL을 사용하고, 데이터베이스를 확장해야 한다면 사용자 프로필용 캐시를 둘 수 있다.

<div style="margin-left:3rem">
    <img src="./images/manage-services-ourselves.png" alt="서비스 직접 관리" width="500" />
</div>

대안으로 클라우드 서비스를 활용해 많은 서비스를 대신 관리하도록 할 수 있다. 예를 들어 AWS API Gateway를 사용해 API 호출을 AWS Lambda 함수로 라우팅할 수 있다.

<div style="margin-left:3rem">
    <img src="./images/api-gateway-mapping.png" alt="API Gateway 매핑" width="500" />
</div>

AWS Lambda를 사용하면 서버를 직접 관리하거나 프로비저닝하지 않고도 코드를 실행할 수 있다. 필요할 때만 실행되며 자동으로 확장된다.

사용자가 점수를 얻는 예시는 다음과 같다.

<div style="margin-left:3rem">
    <img src="./images/user-scoring-point-lambda.png" alt="Lambda에서 사용자가 점수를 얻는 과정" width="500" />
</div>

사용자가 순위표를 조회하는 예시는 다음과 같다.

<div style="margin-left:3rem">
    <img src="./images/user-retrieve-leaderboard.png" alt="사용자의 순위표 조회" width="500" />
</div>

Lambda는 서버리스 아키텍처의 구현체이다. 확장과 환경 설정을 직접 관리할 필요가 없다.

저자는 게임을 처음부터 구축한다면 이 접근법을 사용할 것을 권장한다.

### **Redis 확장**

500만 DAU라면 저장 공간과 QPS 양쪽 측면에서 단일 Redis 인스턴스로 충분하다.

하지만 사용자 기반이 10배 증가해 5억 DAU가 된다고 가정하면 저장 공간은 65GB가 필요하고 QPS는 25만으로 증가한다.

이 정도 규모에는 샤딩이 필요하다.

이를 구현하는 한 가지 방법은 데이터를 범위 파티셔닝하는 것이다.

<div style="margin-left:3rem">
    <img src="./images/range-partition.png" alt="범위 파티션" width="500" />
</div>

이 예시에서는 사용자의 점수를 기준으로 샤딩한다. user_id와 샤드의 매핑은 애플리케이션 코드에서 유지한다.
이 매핑 자체에는 MySQL이나 다른 캐시를 사용할 수 있다.

상위 10명의 플레이어를 조회하려면 점수가 가장 높은 샤드(`[900-1000]`)에 쿼리한다.

사용자의 순위를 조회하려면 해당 사용자의 샤드 내 순위를 계산한 뒤, 다른 샤드에서 더 높은 점수를 가진 모든 사용자의 수를 더해야 한다.
후자는 샤드별 전체 레코드 수를 `info keyspace` 명령으로 빠르게 조회할 수 있으므로 O(1) 연산이다.

대안으로 Redis Cluster를 통한 해시 파티셔닝을 사용할 수 있다. Redis Cluster는 일관된 해싱과 비슷하지만 완전히 같지는 않은 파티셔닝 방식에 따라 Redis 노드 전체에 데이터를 분산하는 프록시이다.

<div style="margin-left:3rem">
    <img src="./images/hash-partition.png" alt="해시 파티션" width="500" />
</div>

이 구성에서는 상위 10명의 플레이어를 계산하기가 어렵다. 각 샤드에서 상위 10명의 플레이어를 가져온 뒤 애플리케이션에서 결과를 병합해야 한다.

<div style="margin-left:3rem">
    <img src="./images/top-10-players-calculation.png" alt="상위 10명 플레이어 계산" width="500" />
</div>

해시 파티셔닝에는 몇 가지 한계가 있다.
- K가 큰 상위 K명의 사용자를 조회해야 한다면 모든 샤드에서 많은 데이터를 가져와야 하므로 지연 시간이 증가할 수 있다.
- 파티션 수가 늘어날수록 지연 시간이 증가한다.
- 사용자의 순위를 결정하는 간단한 방법이 없다.

이러한 이유로 저자는 이 문제에 고정 파티션을 사용하는 쪽을 선호한다.

그 밖의 유의사항은 다음과 같다.
- 쓰기가 많은 Redis 노드에는 필요할 때 스냅샷을 수용할 수 있도록 요구량의 두 배에 해당하는 메모리를 할당하는 것이 모범 사례이다.
- Redis-benchmark라는 도구로 Redis 구성의 성능을 추적하고 데이터에 근거해 의사결정할 수 있다.

### **대안 방안: NoSQL**

다음 특성에 최적화된 적절한 NoSQL 데이터베이스를 사용하는 대안도 고려할 수 있다.
- 쓰기 부하가 많다.
- 동일한 파티션 내 항목을 점수별로 효율적으로 정렬한다.

DynamoDB, Cassandra, MongoDB 모두 적합하다.

이 장에서 저자는 DynamoDB를 사용하기로 했다. DynamoDB는 안정적인 성능과 뛰어난 확장성을 제공하는 완전 관리형 NoSQL 데이터베이스이다.
또한 기본 키에 포함되지 않은 필드를 쿼리해야 할 때 글로벌 보조 인덱스를 사용할 수 있다.

<div style="margin-left:3rem">
    <img src="./images/dynamo-db.png" alt="DynamoDB" width="500" />
</div>

체스 게임의 순위표를 저장하는 테이블부터 시작한다.

<div style="margin-left:3rem">
    <img src="./images/chess-game-leaderboard-table-1.png" alt="체스 게임 순위표 테이블 1" width="500" />
</div>

이 방식은 잘 작동하지만 점수를 기준으로 무언가를 쿼리해야 할 때 확장성이 좋지 않다. 따라서 점수를 정렬 키로 둘 수 있다.

<div style="margin-left:3rem">
    <img src="./images/chess-game-leaderboard-table-2.png" alt="체스 게임 순위표 테이블 2" width="500" />
</div>

이 설계의 또 다른 문제는 월을 기준으로 파티셔닝한다는 것이다. 최신 월에 다른 월보다 접근이 편중되므로 핫스팟 파티션이 발생한다.

각 키에 `user_id % num_partitions`로 계산한 파티션 번호를 덧붙이는 쓰기 샤딩(write sharding) 기법을 사용할 수 있다.

<div style="margin-left:3rem">
    <img src="./images/chess-game-leaderboard-table-3.png" alt="체스 게임 순위표 테이블 3" width="500" />
</div>

사용할 파티션 수를 정할 때 고려해야 할 중요한 트레이드오프는 다음과 같다.
- 파티션이 많을수록 쓰기 확장성이 높아진다.
- 하지만 집계 결과를 수집하기 위해 더 많은 파티션에 쿼리해야 하므로 읽기 확장성은 저하된다.

이 접근법을 사용하려면 앞에서 살펴본 "분산-수집(scatter-gather)" 기법을 사용해야 하며, 파티션을 추가할수록 시간 복잡도가 증가한다.

<div style="margin-left:3rem">
    <img src="./images/scatter-gather-2.png" alt="분산-수집 2" width="500" />
</div>

적절한 파티션 수를 평가하려면 벤치마킹을 수행해야 한다.

이 NoSQL 접근법에는 여전히 한 가지 큰 단점이 있다. 사용자의 정확한 순위를 계산하기 어렵다는 점이다.

샤딩이 필요할 만큼 규모가 크다면 사용자에게 자신이 속한 점수의 "백분위"를 알려 주는 방식을 고려할 수 있다.

cron 작업이 주기적으로 실행되어 점수 분포를 분석하고, 이를 바탕으로 다음과 같이 사용자의 백분위를 결정할 수 있다.

```
10th percentile = score < 100
20th percentile = score < 500
...
90th percentile = score < 6500
```

---

## 4단계: 마무리

시간이 허락한다면 다음 사항도 논의할 수 있다.
- **더 빠른 조회** - `user_id -> user object` 매핑을 가진 Redis 해시를 통해 사용자 객체를 캐시할 수 있다. 데이터베이스에 쿼리하는 것보다 빠르게 조회할 수 있다.
- **동점 처리** - 두 플레이어의 점수가 같으면 마지막으로 경기를 한 시간을 기준으로 정렬해 동점을 처리할 수 있다.
- **시스템 장애 복구** - 대규모 Redis 장애가 발생하면 MySQL WAL 항목을 순회하는 임시 스크립트로 순위표를 재생성할 수 있다.
