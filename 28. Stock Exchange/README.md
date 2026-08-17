# 28장: 증권 거래소

## 소개
이 장에서는 **전자 증권 거래소**를 설계한다.

기본 기능은 매수자와 매도자를 효율적으로 연결하는 것이다.

주요 증권 거래소로는 **NYSE**, **NASDAQ** 등이 있다.

<div style="margin-left:3rem">
    <img src="./images/world-stock-exchanges.png" alt="세계 증권 거래소" width="500" />
</div>

---

## 1단계: 문제 이해 및 설계 범위 확정
 * 지원자: 주식, 옵션, 선물 중 어떤 증권을 거래할 것인가?
 * 면접관: 단순화를 위해 주식만 거래한다.
 * 지원자: 주문 제출, 취소, 정정 중 어떤 주문 작업을 지원하는가? 지정가, 시장가, 조건부 주문은 어떻게 하는가?
 * 면접관: 주문 제출과 취소를 지원해야 한다. 주문 유형은 지정가 주문만 고려하면 된다.
 * 지원자: 시스템이 시간 외 거래를 지원해야 하는가?
 * 면접관: 아니다. 정규 거래 시간만 지원한다.
 * 지원자: 거래소의 기본 기능을 설명해 줄 수 있는가?
 * 면접관: 고객은 지정가 주문을 제출하거나 취소하고 매칭된 체결 결과를 실시간으로 받을 수 있다. 호가창도 실시간으로 볼 수 있어야 한다.
 * 지원자: 거래소의 규모는 어느 정도인가?
 * 면접관: 수만 명의 사용자가 동시에 거래하고 종목은 약 100개이다. 하루 주문은 수십억 건이다. 규정 준수를 위한 위험도 검사도 지원해야 한다.
 * 지원자: 어떤 위험도 검사를 수행하는가?
 * 면접관: 간단한 위험도 검사를 수행하자. 예를 들어 사용자 한 명이 하루에 Apple 주식을 최대 100만 주까지 거래하도록 제한한다.
 * 지원자: 사용자 지갑은 어떻게 연계하는가?
 * 면접관: 고객이 주문을 제출하기 전에 충분한 자금이 있는지 확인해야 한다. 대기 중인 주문에 사용할 자금은 주문이 완료될 때까지 보류해야 한다.

### **비기능 요구사항**
면접관이 언급한 규모는 중소 규모 거래소를 설계해야 함을 시사한다.
향후 더 많은 종목과 사용자를 지원할 수 있는 유연성도 보장해야 한다.

그 밖의 비기능 요구사항은 다음과 같다.
 * 가용성 - 최소 99.99%이다. 중단 시간은 평판을 해칠 수 있다.
 * 내결함성 - 운영 사고의 영향을 제한하려면 내결함성과 빠른 복구 메커니즘이 필요하다.
 * 지연 시간 - 왕복 지연 시간은 밀리초(ms) 수준이어야 하며 99번째 백분위수에 중점을 둔다. 99p 지연 시간이 지속적으로 높으면 소수의 사용자에게 좋지 않은 경험을 준다.
 * 보안 - 계정 관리 시스템이 있어야 한다. 법적 규정 준수를 위해 사용자 신원을 확인하는 고객확인제도(KYC)를 지원해야 한다. 공개 리소스에 대한 DDoS 공격도 방어해야 한다.

### **개략적 규모 추정**
 * 종목 100개, 하루 주문 10억 건
 * 정규 거래 시간은 09:30부터 16:00까지이다(6.5시간).
 * QPS = 1bil / 6.5 / 3600 = 43000
 * Peak QPS = 5*QPS = 215000
 * 시장 개장 시점에는 거래량이 훨씬 많다.

---

## 2단계: 개략적 설계안 제시 및 동의 구하기

### **비즈니스 기초 지식 101**
거래소와 관련된 몇 가지 기본 개념을 논의한다.

브로커는 거래소와 최종 사용자 사이의 상호작용을 중개한다. Robinhood, Fidelity 등이 있다.

기관 고객은 전문 거래 소프트웨어를 사용해 대량으로 거래한다. 이들에게는 전문적인 처리가 필요하다.
예를 들어 대량 거래로 시장에 영향을 주지 않도록 주문을 분할한다.

주문 유형은 다음과 같다.
 * 지정가 - 고정 가격에 매수하거나 매도한다. 즉시 일치하는 주문을 찾지 못하거나 일부만 체결될 수 있다.
 * 시장가 - 가격을 지정하지 않는다. 현재 시장 가격으로 즉시 체결된다.

가격은 다음과 같다.
 * 매수호가 - 매수자가 주식을 사려고 하는 가장 높은 가격
 * 매도호가 - 매도자가 주식을 팔려고 하는 가장 낮은 가격

미국 시장의 시세는 L1, L2, L3의 세 단계로 나뉜다.

L1 시장 데이터에는 최우선 매수호가와 매도호가 및 수량이 포함된다.

<div style="margin-left:3rem">
    <img src="./images/l1-price.png" alt="L1 가격" width="500" />
</div>

L2에는 더 많은 가격 단계가 포함된다.

<div style="margin-left:3rem">
    <img src="./images/l2-price.png" alt="L2 가격" width="500" />
</div>

L3는 각 가격 단계와 해당 단계의 대기 수량을 보여 준다.

<div style="margin-left:3rem">
    <img src="./images/l3-price.png" alt="L3 가격" width="500" />
</div>

캔들스틱은 주어진 구간의 시가와 종가뿐 아니라 최고가와 최저가도 보여 준다.

<div style="margin-left:3rem">
    <img src="./images/candlestick.png" alt="캔들스틱" width="500" />
</div>

FIX는 대부분의 공급업체가 사용하는 증권 거래 정보 교환 프로토콜이다. 증권 거래 예시는 다음과 같다.
```
8=FIX.4.2 | 9=176 | 35=8 | 49=PHLX | 56=PERS | 52=20071123-05:30:00.000 | 11=ATOMNOCCC9990900 | 20=3 | 150=E | 39=E | 55=MSFT | 167=CS | 54=1 | 38=15 | 40=2 | 44=15 | 58=PHLX EQUITY TESTING | 59=0 | 47=C | 32=0 | 31=0 | 151=15 | 14=0 | 6=0 | 10=128 |
```

### **개략적 설계**

<div style="margin-left:3rem">
    <img src="./images/high-level-design.png" alt="개략적 설계" width="500" />
</div>

거래 흐름은 다음과 같다.
 * 고객이 거래 인터페이스를 통해 주문을 제출한다.
 * 브로커가 주문을 거래소로 보낸다.
 * 주문은 고객 게이트웨이를 통해 거래소로 들어온다. 고객 게이트웨이는 검증, 요청률 제한, 인증 등을 수행한다. 주문은 주문 관리자로 전달된다.
 * 주문 관리자가 위험도 관리자가 정한 규칙에 따라 위험도 검사를 수행한다.
 * 위험도 검사를 통과하면 주문 관리자가 지갑에 주문을 위한 충분한 자금이 있는지 확인한다.
 * 주문이 매칭 엔진으로 전송된다. 일치하는 주문을 찾으면 매칭 엔진이 매수와 매도에 대해 체결(fill) 두 개를 내보낸다. 두 주문에 순서를 부여해 결정성을 보장한다.
 * 체결 결과를 고객에게 반환한다.

시장 데이터 흐름(M1-M3)은 다음과 같다.
 * 매칭 엔진이 체결 스트림을 생성해 시장 데이터 게시자로 보낸다.
 * 시장 데이터 게시자가 캔들스틱 차트를 구성해 데이터 서비스로 보낸다.
 * 시장 데이터는 실시간 분석을 위한 전문 저장소에 저장된다. 브로커는 시의적절한 시장 데이터를 얻기 위해 데이터 서비스에 연결한다.

보고 흐름(R1-R2)은 다음과 같다.
 * 리포터가 주문과 체결에서 필요한 모든 보고 필드를 수집해 DB에 기록한다.
 * 보고 필드 - client_id, price, quantity, order_type, filled_quantity, remaining_quantity

거래 흐름은 핵심 경로에 있지만 나머지 흐름은 그렇지 않으므로 흐름별 지연 시간 요구사항이 다르다.

#### 거래 흐름
거래 흐름은 핵심 경로에 있으므로 낮은 지연 시간을 위해 고도로 최적화해야 한다.

그 중심에는 크로스 엔진이라고도 하는 매칭 엔진이 있다. 주요 책임은 다음과 같다.
 * 각 종목의 호가창, 즉 한 종목에 대한 매수/매도 주문 목록을 유지한다.
 * 매수 주문과 매도 주문을 매칭한다. 매칭 결과 매수 측과 매도 측에 각각 하나씩 체결(fill) 두 개가 생긴다. 이 기능은 빠르고 정확해야 한다.
 * 체결 스트림을 시장 데이터로 배포한다.
 * 매칭 결과를 결정적인 순서로 생성해야 한다. 이는 고가용성의 기반이다.

다음은 시퀀서이다. 각 수신 주문과 송신 체결에 시퀀스 ID를 부여해 매칭 엔진을 결정적으로 만드는 핵심 구성 요소이다.

<div style="margin-left:3rem">
    <img src="./images/sequencer.png" alt="시퀀서" width="500" />
</div>

수신 주문과 송신 체결에 시퀀스를 부여하는 이유는 다음과 같다.
 * 적시성과 공정성
 * 빠른 복구/재생
 * 정확히 한 번 보장

개념적으로 Kafka는 사실상 수신 및 송신 메시지 큐이므로 시퀀서로 사용할 수 있다. 하지만 더 낮은 지연 시간을 달성하기 위해 직접 구현한다.

주문 관리자는 주문 상태를 관리한다. 또한 주문을 보내고 체결을 받으며 매칭 엔진과 상호작용한다.

주문 관리자의 책임은 다음과 같다.
 * 위험도 검사를 위해 주문을 보낸다. 예를 들어 사용자의 거래량이 100만 미만인지 확인한다.
 * 사용자 지갑과 대조해 주문을 검사하고, 주문 실행에 충분한 자금이 있는지 확인한다.
 * 주문을 시퀀서로 보낸 뒤 매칭 엔진으로 전달한다. 대역폭을 줄이기 위해 필요한 주문 정보만 매칭 엔진에 전달한다.
 * 시퀀서에서 체결 결과를 다시 받은 뒤 고객 게이트웨이를 통해 브로커에게 보낸다.

주문 관리자를 구현할 때의 주요 과제는 상태 전이 관리이다. 이벤트 소싱은 실현 가능한 방안 중 하나이다(상세 설계에서 다룬다).

마지막으로 고객 게이트웨이는 사용자에게서 주문을 받아 주문 관리자에게 보낸다. 책임은 다음과 같다.

<div style="margin-left:3rem">
    <img src="./images/client-gateway.png" alt="고객 게이트웨이" width="500" />
</div>

고객 게이트웨이는 핵심 경로에 있으므로 가볍게 유지해야 한다.

고객별로 여러 고객 게이트웨이가 있을 수 있다. 예를 들어 코로케이션(colo) 엔진은 브로커가 거래소의 데이터 센터에서 임대하는 거래 엔진 서버이다.

<div style="margin-left:3rem">
    <img src="./images/client-gateways.png" alt="고객 게이트웨이" width="500" />
</div>

#### 시장 데이터 흐름
시장 데이터 게시자는 매칭 엔진에서 체결 결과를 받아 체결 스트림으로 호가창과 캔들스틱 차트를 구성한다.

이 데이터는 집계된 데이터를 구독자에게 보여 줄 책임이 있는 데이터 서비스로 전송된다.

<div style="margin-left:3rem">
    <img src="./images/market-data.png" alt="시장 데이터" width="500" />
</div>

#### 보고 흐름
리포터는 핵심 경로에 있지 않지만 여전히 중요한 구성 요소이다.

<div style="margin-left:3rem">
    <img src="./images/reporting-flow.png" alt="보고 흐름" width="500" />
</div>

거래 이력, 세금 보고, 규정 준수 보고, 정산 등을 담당한다.
보고 흐름에서는 지연 시간이 핵심 요구사항이 아니다. 정확성과 규정 준수가 더 중요하다.

### **API 설계**
고객은 브로커를 통해 증권 거래소와 상호작용하며 주문 제출, 체결 조회, 시장 데이터 조회, 분석을 위한 과거 데이터 다운로드 등을 수행한다.

고객 게이트웨이와 브로커 간 통신에는 RESTful API를 사용한다.

기관 고객에게는 낮은 지연 시간 요구사항을 충족하기 위해 전용 프로토콜을 사용한다.

주문 생성:
```
POST /v1/order
```

매개변수:
 * symbol - 주식 종목 기호. String
 * side - buy 또는 sell. String
 * price - 지정가 주문의 가격. Long
 * orderType - limit 또는 market(설계에서는 limit 주문만 지원한다). String
 * quantity - 주문 수량. Long

응답:
 * id - 주문 ID. Long
 * creationTime - 시스템에서 주문을 생성한 시간. Long
 * filledQuantity - 성공적으로 체결된 수량. Long
 * remainingQuantity - 아직 체결되지 않은 수량. Long
 * status - new/canceled/filled. String
 * 나머지 속성은 입력 매개변수와 같다.

체결 조회:
```
GET /execution?symbol={:symbol}&orderId={:orderId}&startTime={:startTime}&endTime={:endTime}
```

매개변수:
 * symbol - 주식 종목 기호. String
 * orderId - 주문 ID. 선택 사항. String
 * startTime - epoch 형식의 조회 시작 시간 \[11\]. Long
 * endTime - epoch 형식의 조회 종료 시간. Long

응답:
 * executions - 범위에 포함되는 각 체결의 배열(아래 속성 참조). Array
 * id - 체결 ID. Long
 * orderId - 주문 ID. Long
 * symbol - 주식 종목 기호. String
 * side - buy 또는 sell. String
 * price - 체결 가격. Long
 * orderType - limit 또는 market. String
 * quantity - 체결 수량. Long

호가창 조회:
```
GET /marketdata/orderBook/L2?symbol={:symbol}&depth={:depth}
```

매개변수:
 * symbol - 주식 종목 기호. String
 * depth - 매수/매도 측별 호가창 깊이. Int

응답:
 * bids - 가격과 수량이 담긴 배열. Array
 * asks - 가격과 수량이 담긴 배열. Array

캔들스틱 조회:
```
GET /marketdata/candles?symbol={:symbol}&resolution={:resolution}&startTime={:startTime}&endTime={:endTime}
```

매개변수:
 * symbol - 주식 종목 기호. String
 * resolution - 캔들스틱 차트의 시간 구간 길이(초). Long
 * startTime - epoch 형식의 구간 시작 시간. Long
 * endTime - epoch 형식의 구간 종료 시간. Long

응답:
 * candles - 각 캔들스틱 데이터가 담긴 배열(속성은 아래에 나열). Array
 * open - 각 캔들스틱의 시가. Double
 * close - 각 캔들스틱의 종가. Double
 * high - 각 캔들스틱의 최고가. Double
 * low - 각 캔들스틱의 최저가. Double

### **데이터 모델**
거래소에는 세 가지 주요 데이터 유형이 있다.
 * 상품, 주문, 체결
 * 호가창
 * 캔들스틱 차트

#### 상품, 주문, 체결
상품은 거래되는 종목의 속성을 설명한다. 상품 유형, 거래 종목 기호, UI 표시 종목 기호 등이 있다.

이 데이터는 자주 변경되지 않으며 주로 UI 렌더링에 사용된다.

주문은 매수/매도 주문에 대한 지시를 나타낸다. 체결은 외부로 전달되는 매칭 결과이다.

데이터 모델은 다음과 같다.

<div style="margin-left:3rem">
    <img src="./images/product-order-execution-data-model.png" alt="상품, 주문, 체결 데이터 모델" width="500" />
</div>

세 가지 흐름 모두에서 주문과 체결을 접한다.
 * 핵심 경로에서는 높은 성능을 위해 인메모리로 처리한다. 시퀀서에 저장하고 시퀀서에서 복구한다.
 * 리포터는 보고 용도로 주문과 체결을 데이터베이스에 기록한다.
 * 호가창과 캔들스틱 차트를 재구성하도록 체결 결과를 시장 데이터 쪽으로 전달한다.

#### 호가창
호가창은 상품의 매수/매도 주문을 가격 단계별로 구성한 목록이다.

이 모델을 위한 효율적인 자료 구조는 다음 조건을 충족해야 한다.
 * 상수 시간 조회 - 특정 가격 단계 또는 가격 단계 사이의 거래량 조회
 * 빠른 추가/체결/취소 연산
 * 최우선 매수호가/매도호가 조회
 * 가격 단계 순회

호가창 체결 예시는 다음과 같다.

<div style="margin-left:3rem">
    <img src="./images/order-book-execution.png" alt="호가창 체결" width="500" />
</div>

이 대량 주문을 체결한 후 매수호가와 매도호가의 스프레드가 벌어지면서 가격이 오른다.

호가창 구현의 의사 코드 예시는 다음과 같다.
```
class PriceLevel{
    private Price limitPrice;
    private long totalVolume;
    private List<Order> orders;
}

class Book<Side> {
    private Side side;
    private Map<Price, PriceLevel> limitMap;
}

class OrderBook {
    private Book<Buy> buyBook;
    private Book<Sell> sellBook;
    private PriceLevel bestBid;
    private PriceLevel bestOffer;
    private Map<OrderID, Order> orderMap;
}
```

더 효율적으로 구현하려면 표준 리스트 대신 이중 연결 리스트를 사용할 수 있다.
 * 리스트의 꼬리에 주문을 추가하므로 새 주문 제출은 O(1)이다.
 * 리스트의 머리에서 주문을 삭제하므로 주문 매칭은 O(1)이다.
 * 주문 취소는 호가창에서 주문을 삭제하는 것을 의미한다. O(1) 조회에는 `orderMap`을 사용하며, `Order`가 리스트의 이전 요소를 참조하므로 삭제도 O(1)이다.

<div style="margin-left:3rem">
    <img src="./images/order-book-impl.png" alt="호가창 구현" width="500" />
</div>

이 자료 구조는 시장 데이터 서비스에서 호가창을 재구성할 때도 사용한다.

#### 캔들스틱 차트
캔들스틱 데이터는 시장 데이터 서비스가 시간 구간 내 주문을 처리한 결과를 바탕으로 계산한다.
```
class Candlestick {
    private long openPrice;
    private long closePrice;
    private long highPrice;
    private long lowPrice;
    private long volume;
    private long timestamp;
    private int interval;
}

class CandlestickChart {
    private LinkedList<Candlestick> sticks;
}
```

메모리를 지나치게 많이 사용하지 않기 위한 최적화는 다음과 같다.
 * 캔들스틱을 보관할 때 미리 할당한 링 버퍼를 사용해 할당 횟수를 줄인다.
 * 메모리에 두는 캔들스틱 수를 제한하고 나머지는 디스크에 영속화한다.

실시간 분석에는 인메모리 열 지향 데이터베이스(예: KDB)를 사용한다. 시장이 마감되면 데이터를 이력 데이터베이스에 영속화한다.

---

## 3단계: 상세 설계
현대 거래소에 관해 알아둘 흥미로운 점 하나는 대부분의 다른 소프트웨어와 달리 일반적으로 모든 것을 하나의 거대한 서버에서 실행한다는 것이다.

세부 사항을 살펴본다.

### **성능**
거래소에서는 모든 백분위수에 걸쳐 전반적인 지연 시간이 짧은 것이 매우 중요하다.

지연 시간을 어떻게 줄일 수 있는가?
 * 핵심 경로의 작업 수를 줄인다.
 * 네트워크/디스크 사용량 또는 작업 실행 시간을 줄여 각 작업에 드는 시간을 단축한다.

첫 번째 목표를 달성하기 위해 핵심 경로에서 관련 없는 책임을 모두 제거했으며, 최적의 지연 시간을 달성하고자 로깅까지 제거한다.

원래 설계를 따르면 서비스 간 네트워크 지연 시간과 시퀀서의 디스크 사용이라는 몇 가지 병목이 있다.

이러한 설계로는 종단 간 지연 시간이 수십 밀리초에 이를 수 있다. 대신 수십 마이크로초를 달성하려 한다.

따라서 모든 것을 한 서버에 두고 프로세스는 mmap을 이벤트 저장소로 사용해 통신한다.

<div style="margin-left:3rem">
    <img src="./images/mmap-bus.png" alt="mmap 버스" width="500" />
</div>

또 다른 최적화는 애플리케이션 루프(핵심 작업을 실행하는 while 루프)를 동일한 CPU에 고정해 컨텍스트 스위칭을 피하는 것이다.

<div style="margin-left:3rem">
    <img src="./images/application-loop.png" alt="애플리케이션 루프" width="500" />
</div>

애플리케이션 루프를 사용하면 여러 스레드가 같은 리소스를 차지하려고 다투는 잠금 경합도 사라진다.

이제 mmap의 작동 방식을 살펴본다. mmap은 디스크의 파일을 애플리케이션 메모리에 매핑하는 UNIX 시스템 호출이다.

사용할 수 있는 한 가지 기법은 "공유 메모리(shared memory)"를 뜻하는 `/dev/shm`에 파일을 생성하는 것이다. 따라서 디스크 접근이 전혀 발생하지 않는다.

### **이벤트 소싱**
이벤트 소싱은 [전자 지갑 장](../chapter28)에서 자세히 논의한다. 모든 세부 사항은 해당 장을 참조한다.

간단히 말해 현재 상태를 저장하는 대신 불변 상태 전이를 저장한다.

<div style="margin-left:3rem">
    <img src="./images/event-sourcing.png" alt="이벤트 소싱" width="500" />
</div>

 * 왼쪽 - 전통적인 스키마
 * 오른쪽 - 이벤트 소싱 스키마

지금까지의 설계는 다음과 같다.

<div style="margin-left:3rem">
    <img src="./images/design-so-far.png" alt="현재까지의 설계" width="500" />
</div>

 * 외부 도메인이 FIX 프로토콜을 사용해 고객 게이트웨이와 상호작용한다.
 * 주문 관리자가 새 주문 이벤트를 받아 검증하고 내부 상태에 추가한다. 그런 다음 주문을 매칭 코어로 보낸다.
 * 주문이 매칭되면 `OrderFilledEvent`를 생성해 mmap으로 전송한다.
 * 다른 구성 요소가 이벤트 저장소를 구독하고 각자 맡은 처리를 수행한다.

한 가지 추가 최적화로 모든 구성 요소가 별도 주문 관리 호출을 피하도록 라이브러리로 패키징한 주문 관리자의 복사본을 보유한다.

이 설계에서 시퀀서는 더 이상 이벤트 저장소가 아니라 이벤트 저장소로 전달하기 전에 이벤트에 순서를 부여하는 단일 쓰기 주체가 된다.

<div style="margin-left:3rem">
    <img src="./images/sequencer-deep-dive.png" alt="시퀀서 상세 설계" width="500" />
</div>

### **고가용성**
목표 가용성은 99.99%로, 하루 중단 시간은 8.64초에 불과하다.

이를 달성하려면 거래소 아키텍처의 단일 장애점을 식별해야 한다.
 * 대기 상태로 둘 핵심 서비스(예: 매칭 엔진)의 백업 인스턴스를 구성한다.
 * 장애 감지와 백업 인스턴스로의 장애 조치를 적극적으로 자동화한다.

고객 게이트웨이 같은 무상태 서비스는 서버를 추가해 쉽게 수평 확장할 수 있다.

상태 저장 구성 요소에서는 리더가 아닐 때 수신 이벤트를 처리하되 송신 이벤트는 게시하지 않을 수 있다.

<div style="margin-left:3rem">
    <img src="./images/leader-election.png" alt="리더 선출" width="500" />
</div>

주 복제본의 중단 여부를 감지하려면 하트비트를 보내 작동하지 않는지 확인할 수 있다.

이 메커니즘은 단일 서버의 경계 안에서만 작동한다.
이를 확장하려면 전체 서버를 핫/웜 복제본으로 구성하고 장애 발생 시 장애 조치할 수 있다.

복제본 간 이벤트 저장소 복제에는 더 빠른 통신을 위해 신뢰할 수 있는 UDP를 사용할 수 있다.

### **내결함성**
웜 인스턴스까지 중단되면 어떻게 하는가? 발생 확률은 낮지만 대비해야 한다.

대형 기술 기업은 자연재해 등의 영향을 완화하기 위해 여러 도시에 있는 데이터 센터에 핵심 데이터를 복제하는 방식으로 이 문제에 대응한다.

고려해야 할 질문은 다음과 같다.
 * 주 인스턴스가 중단되면 언제 어떻게 백업 인스턴스로 장애 조치하는가?
 * 백업 인스턴스 중에서 리더를 어떻게 선택하는가?
 * 필요한 복구 시간은 얼마인가(복구 시간 목표, RTO)?
 * 어떤 기능을 복구해야 하는가? 시스템이 성능 저하 상태에서도 작동할 수 있는가?

해결 방법은 다음과 같다.
 * 버그 때문에 시스템이 중단될 수 있으며 주 인스턴스와 복제본 모두에 영향을 줄 수 있다. 카오스 엔지니어링으로 이러한 경계 사례와 치명적인 결과를 드러낼 수 있다.
 * 하지만 처음에는 시스템의 장애 유형에 관한 충분한 지식을 쌓을 때까지 수동으로 장애 조치할 수 있다.
 * 주 인스턴스 중단 시 어느 복제본이 리더가 될지 결정하기 위해 리더 선출(예: Raft)을 사용할 수 있다.

서로 다른 서버 간 복제 작동 방식의 예시는 다음과 같다.

<div style="margin-left:3rem">
    <img src="./images/replication-across-servers.png" alt="서버 간 복제" width="500" />
</div>

리더 선출 임기의 예시는 다음과 같다.

<div style="margin-left:3rem">
    <img src="./images/leader-election-terms.png" alt="리더 선출 임기" width="500" />
</div>

Raft 작동 방식에 관한 자세한 내용은 [여기에서 확인할 수 있다](https://thesecretlivesofdata.com/raft/).

마지막으로 데이터 손실 허용 범위, 즉 상황이 심각해지기 전까지 얼마나 많은 데이터를 잃을 수 있는지도 고려해야 한다.
이 범위에 따라 데이터 백업 빈도가 결정된다.

증권 거래소에서는 데이터 손실을 허용할 수 없으므로 데이터를 자주 백업하고 Raft의 복제에 의존해 데이터 손실 확률을 줄여야 한다.

### **매칭 알고리즘**
의사 코드를 통해 매칭의 작동 방식을 잠시 살펴본다.
```
Context handleOrder(OrderBook orderBook, OrderEvent orderEvent) {
    if (orderEvent.getSequenceId() != nextSequence) {
        return Error(OUT_OF_ORDER, nextSequence);
    }

    if (!validateOrder(symbol, price, quantity)) {
        return ERROR(INVALID_ORDER, orderEvent);
    }

    Order order = createOrderFromEvent(orderEvent);
    switch (msgType):
        case NEW:
            return handleNew(orderBook, order);
        case CANCEL:
            return handleCancel(orderBook, order);
        default:
            return ERROR(INVALID_MSG_TYPE, msgType);

}

Context handleNew(OrderBook orderBook, Order order) {
    if (BUY.equals(order.side)) {
        return match(orderBook.sellBook, order);
    } else {
        return match(orderBook.buyBook, order);
    }
}

Context handleCancel(OrderBook orderBook, Order order) {
    if (!orderBook.orderMap.contains(order.orderId)) {
        return ERROR(CANNOT_CANCEL_ALREADY_MATCHED, order);
    }

    removeOrder(order);
    setOrderStatus(order, CANCELED);
    return SUCCESS(CANCEL_SUCCESS, order);
}

Context match(OrderBook book, Order order) {
    Quantity leavesQuantity = order.quantity - order.matchedQuantity;
    Iterator<Order> limitIter = book.limitMap.get(order.price).orders;
    while (limitIter.hasNext() && leavesQuantity > 0) {
        Quantity matched = min(limitIter.next.quantity, order.quantity);
        order.matchedQuantity += matched;
        leavesQuantity = order.quantity - order.matchedQuantity;
        remove(limitIter.next);
        generateMatchedFill();
    }
    return SUCCESS(MATCH_SUCCESS, order);
}
```

이 매칭 알고리즘은 한 가격 단계에서 어떤 주문을 매칭할지 결정하는 데 FIFO 알고리즘을 사용한다.

### **결정성**
기능적 결정성은 사용한 시퀀서 기법으로 보장한다.

이벤트가 실제로 발생하는 시간은 중요하지 않다.

<div style="margin-left:3rem">
    <img src="./images/determinism.png" alt="결정성" width="500" />
</div>

지연 시간 결정성은 추적해야 한다. 99 또는 99.99 백분위수 지연 시간을 모니터링해 계산할 수 있다.

예를 들어 Java의 가비지 컬렉터 이벤트가 지연 시간 급증을 일으킬 수 있다.

### **시장 데이터 게시자 최적화**
시장 데이터 게시자는 매칭 엔진에서 매칭 결과를 받아 이를 바탕으로 호가창과 캔들스틱 차트를 재구성한다.

메모리는 무한하지 않으므로 캔들스틱 일부만 유지한다. 고객은 원하는 정보의 세분화 수준을 선택할 수 있다. 더 세분화된 정보에는 더 높은 가격이 필요할 수 있다.

<div style="margin-left:3rem">
    <img src="./images/market-data-publisher.png" alt="시장 데이터 게시자" width="500" />
</div>

링 버퍼(순환 버퍼라고도 함)는 머리와 꼬리가 연결된 고정 크기 큐이다. 할당을 피하기 위해 공간을 미리 할당한다. 이 자료 구조는 잠금도 사용하지 않는다.

링 버퍼를 최적화하는 또 다른 기법은 패딩이다. 이를 통해 시퀀스 번호가 다른 어떤 것과도 동일한 캐시 라인에 놓이지 않도록 한다.

### **시장 데이터 배포의 공정성과 멀티캐스트**
한 구독자가 다른 구독자보다 먼저 데이터를 받으면 시장을 조작하는 데 사용할 수 있는 중요한 시장 정보를 얻게 되므로 구독자들이 동시에 데이터를 받도록 보장해야 한다.

이를 달성하기 위해 구독자에게 데이터를 게시할 때 신뢰할 수 있는 UDP를 사용하는 멀티캐스트를 이용할 수 있다.

인터넷에서 데이터는 세 가지 방식으로 전송할 수 있다.
 * 유니캐스트 - 하나의 출발지, 하나의 목적지
 * 브로드캐스트 - 하나의 출발지에서 전체 서브네트워크로 전송
 * 멀티캐스트 - 하나의 출발지에서 서로 다른 서브네트워크의 호스트 집합으로 전송

이론적으로 멀티캐스트를 사용하면 모든 구독자가 동시에 데이터를 받아야 한다.

하지만 UDP는 신뢰할 수 없으며 데이터가 모두에게 도달하지 않을 수 있다. 다만 재전송으로 보완할 수 있다.

### **코로케이션**
거래소는 브로커가 거래소와 동일한 데이터 센터에 서버를 함께 배치할 수 있게 한다.

이렇게 하면 지연 시간이 크게 줄어들며 VIP 서비스로 볼 수 있다.

### **네트워크 보안**
인터넷에 노출된 서비스가 일부 있으므로 DDoS는 거래소가 해결해야 할 과제이다. 선택지는 다음과 같다.
 * DDoS 공격이 가장 중요한 고객에게 영향을 주지 않도록 공개 서비스와 데이터를 비공개 서비스에서 격리한다.
 * 자주 갱신되지 않는 데이터를 저장하는 캐시 계층을 사용한다.
 * DDoS에 견디도록 URL을 강화한다. 예를 들어 `https://my.website.com/data/recent`를 `https://my.website.com/data?from=123&to=456`보다 선호한다. 전자가 캐시하기 더 쉽기 때문이다.
 * 효과적인 허용 목록/차단 목록 메커니즘이 필요하다.
 * 요청률 제한을 사용해 DDoS를 완화할 수 있다.

---

## 4단계: 마무리
그 밖의 흥미로운 내용은 다음과 같다.
 * 모든 거래소가 하나의 큰 서버에 모든 것을 두는 방식에 의존하는 것은 아니지만, 일부는 여전히 그렇게 한다.
 * 현대 거래소는 클라우드 인프라와 자동화된 시장 조성자(AMM)에 더 많이 의존해 호가창을 유지하는 일을 피하기도 한다.
