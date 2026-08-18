> **한국어 번역 안내 / Korean Translation Notice**
>
> 이 저장소는 [liquidslr/system-design-notes](https://github.com/liquidslr/system-design-notes)의 비공식 한국어 번역본이다.
> 원본 노트는 Alex Xu의 《System Design Interview - An Insider's Guide》 1·2권을 바탕으로 작성되었으며,
> 공식 한국어판으로는 인사이트 출판사의 《가상 면접 사례로 배우는 대규모 시스템 설계 기초》 1·2권이 있다.
> 모든 권리는 원저작자에게 있으며, 이 번역본은 학습 목적으로만 사용한다.
>
> This is an unofficial Korean translation of [liquidslr/system-design-notes](https://github.com/liquidslr/system-design-notes).
> All rights belong to the original authors. For learning purposes only.

# [시스템 설계 면접 - 내부자 가이드(1권 및 2권)](https://bytebytego.com/courses/system-design-interview)
이 노트는 시스템 설계 면접 도서인 [1권 및 2권 제2판](https://www.goodreads.com/book/show/54109255-system-design-interview-an-insider-s-guide)을 바탕으로 작성했다.

노트는 다음 링크에서 확인할 수 있다: https://pagefy.io/system-design/system-design-interview-by-alex-xu

**참고:** 이 노트는 현재 작성 중이다.


 * [1장: 사용자 0명에서 수백만 명까지 확장](./01.%20Scaling/)
 * [2장: 개략적 규모 추정](./02.%20Back%20Of%20the%20Envelope%20Estimation/)
 * [3장: 시스템 설계 면접을 위한 프레임워크](./03.%20System%20Design%20Framework/)
 * [4장: 처리율 제한기 설계](./04.%20Rate%20Limiter//)
 * [5장: 일관 해싱 설계](./05.%20Consistent%20Hashing/)
 * [6장: 키-값 저장소 설계](./06.%20Key-Value%20Store/)
 * [7장: 분산 시스템의 고유 ID 생성기 설계](./07.%20Unique-Id%20Generator/)
 * [8장: URL 단축기 설계](./08.%20URL%20Shortener/)
 * [9장: 웹 크롤러 설계](./09.%20Web%20Crawler/)
 * [10장: 알림 시스템 설계](./10.%20Notification%20System/)
 * [11장: 뉴스 피드 시스템 설계](./11.%20News%20Feed%20System/)
 * [12장: 채팅 시스템 설계](./12.%20Chat%20System/)
 * [13장: 검색어 자동 완성 시스템 설계](./13.%20Search%20Autocomplete/)
 * [14장: YouTube 설계](./14.%20Youtube/)
 * [15장: Google Drive 설계](./15.%20Google%20Drive/)
 * [16장: 근접성 서비스](./16.%20Proximity%20Service/)
 * [17장: 주변 친구](./17.%20Nearby%20Friends/)
 * [18장: Google Maps](./18.%20Google%20Maps/)
 * [19장: 분산 메시지 큐](./19.%20Distributed%20Message%20Queue/)
 * [20장: 지표 모니터링 및 경보 시스템](./20.%20Metrics%20Monitoring%20and%20Alerting%20System/)
 * [21장: 광고 클릭 이벤트 집계](./21.%20Ad%20Click%20Event%20Aggregation/)
 * [22장: 호텔 예약 시스템](./22.%20Hotel%20Reservation%20System/)
 * [23장: 분산 이메일 서비스](./23.%20Distributed%20Email%20Service/)
 * [24장: S3 유사 객체 저장소](./24.%20S3-like%20Object%20Storage/)
 * [25장: 실시간 게임 순위표](./25.%20Real-time%20Gaming%20Leaderboard/)
 * [26장: 결제 시스템](./26.%20Payment%20System/)
 * [27장: 전자 지갑](./27.%20%20Digital%20Wallet/)
 * [28장: 증권 거래소](./28.%20Stock%20Exchange/)


# 추가 자료

### 처리율 제한
- [서킷 브레이커 알고리즘](https://martinfowler.com/bliki/CircuitBreaker.html)
- [Uber 처리율 제한기](https://github.com/uber-go/ratelimit/blob/master/ratelimit.go)


### 일관 해싱
- [일관 해싱](https://tom-e-white.com/2007/11/consistent-hashing.html)
- [CS168: 개요와 일관 해싱]( http://theory.stanford.edu/~tim/s16/l/l1.pdf)
- [Apache Cassandra](http://www.cs.cornell.edu/Projects/ladis2009/papers/Lakshman-ladis2009.PDF)
- [Discord 확장](https://blog.discord.com/scaling-elixir-f9b8e1e7c29b)
- [Google Maglev](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44824.pdf)


### 키-값 저장소
- [Amazon Dynamo](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)
- [Cassandra 아키텍처](https://docs.datastax.com/en/archived/cassandra/3.0/cassandra/architecture/archIntro.html)
- [Google BigTable 아키텍처](https://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf)
- [Amazon Dynamo DB 내부 구조](https://www.allthingsdistributed.com/2007/10/amazons_dynamo.html)
- [Amazon Dynamo DB의 설계 패턴](https://www.youtube.com/watch?v=HaEPXoXVf2k)
- [Amazon Dynamo DB 내부 구조](https://www.youtube.com/watch?v=yvBR71D0nAQ)


### 고유 ID 생성기
- [티켓 서버: 저비용 분산 고유 기본 키](https://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap)
- [Snowflake](https://blog.twitter.com/engineering/en_us/a/2010/announcing-snowflake.html)


### 웹 크롤러
- [웹 크롤링](http://infolab.stanford.edu/~olston/publications/crawling_survey.pdf)
- [Google 동적 렌더링](https://developers.google.com/search/docs/guides/dynamic-rendering)



### 채팅 시스템
- [Discord가 수십억 개의 메시지를 저장하는 방법](https://discord.com/blog/how-discord-stores-billions-of-messages)
- [Flannel: Slack의 확장을 지원하는 애플리케이션 수준 엣지 캐시](https://slack.engineering/flannel-an-application-level-edge-cache-to-make-slack-scale/)


### 검색어 자동 완성
- [Prefixy를 구축한 방법](https://medium.com/@prefixyteam/how-we-built-prefixy-a-scalable-prefix-search-service-for-powering-autocomplete-c20f98e2eff1)
- [접두사 해시 트리](https://people.eecs.berkeley.edu/~sylvia/papers/pht.pdf)


### YouTube
- [YouTube 아키텍처](http://highscalability.com/youtube-architecture)
- [YouTube 확장성 2012](https://www.youtube.com/watch?v=w5WVu624fY8)
- [대규모 비디오 트랜스코딩](https://www.egnyte.com/blog/2018/12/transcoding-how-we-serve-videos-at-scale/)
- [Facebook 비디오 방송](https://engineering.fb.com/ios/under-the-hood-broadcasting-live-video-to-millions/)
- [Netflix의 대규모 비디오 인코딩](https://netflixtechblog.com/high-quality-video-encoding-at-scale-d159db052746)
- [Netflix의 샷 기반 인코딩](https://netflixtechblog.com/optimized-shot-based-encodes-now-streaming-4b9464204830)


### Google Drive
- [차등 동기화](https://neil.fraser.name/writing/sync/)
- [차등 동기화 영상](https://www.youtube.com/watch?v=S2Hp_1jqpY8)
- [Dropbox를 확장한 방법](https://www.youtube.com/watch?v=PE4gwstWhmc&feature=youtu.be)
