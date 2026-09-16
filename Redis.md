# Redis 정리

## 1. Redis란?

* **Redis(Remote Dictionary Server)**

  * 프로세스로 실행되는 **In-Memory 기반 Key-Value 데이터 저장소**
  * 데이터를 `Key`와 `Value` 형태로 저장
  * 관계형 DB처럼 테이블 간 관계를 중심으로 관리하지 않음
  * 빠른 데이터 읽기/쓰기에 최적화
  * 대부분의 데이터를 **메모리(RAM)**에서 처리
  * 일반적인 디스크 기반 DB보다 빠른 처리 속도 제공

* 주요 활용 분야

  * Database
  * Cache
  * Message Broker
  * Ranking
  * Session Storage
  * Shared Data Storage
  * 실시간 데이터 처리

---

# 2. Redis의 특징

## 2.1 In-Memory Database

* 데이터를 주로 **메모리에서 처리**
  -> 디스크보다 빠른 접근 속도
    -> 데이터 조회 및 수정 속도가 매우 빠름

* 활용 예시

  * 실시간 랭킹
  * 세션 관리
  * 캐시
  * 로그인 인증 정보
  * 게임 서버 임시 데이터
  * 실시간 통계
  * API 응답 캐싱

* 메모리 DB이지만 영속성 지원

  * **RDB Snapshot**
  * **AOF(Append Only File)**

* 필요 시 데이터를 디스크에 저장하여 복구 가능

---

## 2.2 Key-Value 구조

* 기본 저장 형태

  * `Key -> Value`

* 예시

  * `user:1 -> Alice`

* 데이터 저장

```redis
SET user:1 "Alice"
```

* 데이터 조회

```redis
GET user:1
```

* 특징

  * 구조가 단순함
  * Key를 이용한 빠른 데이터 접근 가능

---

# 3. Redis가 지원하는 자료구조

* **String**
  (일반적인 문자열)
* **List**
  (일반적인 리스트)
* **Set**
  (Key에 중복되지 않은 여러 Value들을 저장하는 자료구조)
* **Sorted Set**
  (Set과 특징은 같지만 score라는 필드가 추가되어 있음
   -> score순으로 오름차순으로 정렬되어 있냐의 차이)
* **Hash**
  (Field-Value형식, 한 Key에 여러 Field-Value가 들어갈 수 있음)
* **Stream**
  (이벤트 기록(log)을 저장하기 좋은 자료구조)
* **Bitmap**
  (String의 변형, Bit 단위 연산 가능)
* **HyperLogLogs**
  (매우 많은 데이터에서 고유한 카디널리티를 추정하는 자료구조, 오차범위 0.81%, 최대 12KB)

* 장점

  * 단순 Key-Value 저장뿐만 아니라 다양한 기능 구현 가능
  * 자료구조에 맞는 명령어 제공

* 게임 랭킹 예시

  * **Sorted Set** 활용
  * 플레이어와 점수를 함께 저장
  * 점수를 기준으로 자동 정렬
  * 전체 데이터를 따로 가져와 정렬할 필요 없이 순위 계산 가능

---

# 4. Redis 활용 방법

## 4.1 실시간 랭킹

* **Sorted Set** 활용
* 점수를 기준으로 데이터 정렬
* 플레이어 점수 변경 시 Redis에 즉시 반영
* 상위 랭킹을 빠르게 조회 가능
* 실시간으로 순위가 변경되는 서비스에 적합

---

## 4.2 Cache

* Redis의 대표적인 활용 방식
* 자주 조회되는 데이터를 메모리에 저장
* 동일한 DB 요청 반복을 줄일 수 있음

예시 쿼리:

```sql
SELECT * FROM ranking ORDER BY score DESC LIMIT 100;
```

* Redis를 사용하지 않는 경우

  * 요청마다 MySQL에서 조회 및 정렬
  * DB 부하 증가

* Redis를 사용하는 경우

  * 조회 결과를 Redis에 저장
  * 이후 요청은 Redis에서 빠르게 조회
  * Redis에 데이터가 없을 때만 DB 조회
  * DB 부하와 응답 시간 감소

---

## 4.3 Session Storage

* 로그인 세션을 Redis에 저장 가능
* 여러 애플리케이션 서버가 동일한 Redis를 공유 가능
* 어느 서버로 요청이 들어와도 동일한 세션 정보 사용 가능
* 서버 수평 확장 시 유용

---

## 4.4 Pub/Sub

* **Publish / Subscribe** 기능 지원

* Publisher

  * 메시지를 발행하는 쪽

* Subscriber

  * 메시지를 구독하고 수신하는 쪽

* Channel

  * Publisher와 Subscriber가 메시지를 주고받는 통로

* 활용

  * 실시간 알림
  * 채팅
  * 서버 간 이벤트 전달
  * 실시간 상태 업데이트

* 주의점

  * 기본 Pub/Sub은 메시지를 영구 저장하지 않음
  * Subscriber가 연결되지 않은 동안 발생한 메시지는 다시 받을 수 없음

* 높은 메시지 신뢰성이 필요한 경우

  * Redis Streams
  * Kafka
  * RabbitMQ 등 고려

---

# 5. Redis CLI 기본 명령어

## 명령어 종류

* `SET` : 데이터 삽입
* `GET` : 데이터 조회
* `DEL` : 데이터 삭제
* `EXISTS` : 데이터 존재 여부 확인
* `MGET` : 여러 Key의 값 한 번에 조회
* `EXPIRE` : Key에 만료 시간 설정
* `TTL` : Key의 남은 만료 시간 확인

---

## 명령어 사용 예시

### SET

데이터를 삽입할 때 사용

```redis
SET key "value"
```

예시:

```redis
SET user:1 "Alice"
```

---

### GET

Key에 저장된 데이터를 조회할 때 사용

```redis
GET key
```

예시:

```redis
GET user:1
```

결과:

```text
Alice
```

---

### DEL

Key와 데이터를 삭제할 때 사용

```redis
DEL key
```

예시:

```redis
DEL user:1
```

---

### EXISTS

해당 Key가 존재하는지 확인할 때 사용

```redis
EXISTS key
```

예시:

```redis
EXISTS user:1
```

결과:

```text
1 → 존재함
0 → 존재하지 않음
```

---

### MGET

여러 Key의 값을 한 번에 조회할 때 사용

```redis
MGET key1 key2 key3
```

예시:

```redis
MGET user:1 user:2 user:3
```

---

### EXPIRE

Key에 만료 시간을 설정할 때 사용.

```redis
EXPIRE key seconds
```

예시:

```redis
EXPIRE user:1 60
```

`user:1`은 60초 후 자동으로 삭제된다.

이러한 데이터의 남은 생존 시간을 **TTL(Time To Live)**이라고 함.

---

### TTL

Key의 남은 만료 시간을 확인할 때 사용

```redis
TTL key
```

예시:

```redis
TTL user:1
```

---

# 6. Redis의 장점

## 빠른 속도

* 데이터를 메모리에서 처리
* 빠른 읽기/쓰기 성능
* 실시간 처리가 필요한 서비스에 적합

---

## 다양한 자료구조

* String
* List
* Set
* Sorted Set
* Hash
* Stream 등 지원

* 활용

  * 캐시
  * 랭킹
  * 메시징
  * 세션
  * 실시간 데이터 처리

---

## 다양한 프로그래밍 언어 지원

* C#
* Java
* Python
* JavaScript / Node.js
* Go
* C / C++
* PHP

* 대부분의 서버 프레임워크에서 Redis 사용 가능

---

## 확장성

* Redis Cluster 지원
  (RedisCluster란: 실행중인 여러 Redis에 데이터를 분산하는 기능 -> 장애 발생시에도 작동 & 고 성능 보장)

* 여러 Redis 노드에 데이터를 분산 가능
* 하나의 서버에 모든 부하가 집중되는 문제 완화
* 데이터 증가에 따라 서버 확장 가능

---

# 7. Sharding

* **Sharding**

  * 하나의 데이터 집합을 여러 서버에 나누어 저장하는 방식
    -> 이를 이용해서 대용량의 데이터들을 처리할 수 있으며 관리 용이성이 증가

* 샤딩 전략
  - 수평 샤딩
    (튜플 단위로 나눠서 저장함)
  - 수직 샤딩
    (어트리뷰트 단위로 나눠서 저장함)

* 예시

  * Shard A → User 1 ~ 5000
  * Shard B → User 5001 ~ 10000
  * Shard C → User 10001 ~ 15000

* 장점

  * 서버 부하 분산
  * 저장 공간 확장
  * 대규모 데이터 처리 가능

* 단점
  - 복잡성: 데이터 관리의 복잡성 증가 -> 샤드 간의 데이터 일관성과 무결성을 유지하는 것이 중요
  - 재샤딩(Resharding): 데이터 분포가 불균형하게 되거나, 시스템이 성장함에 따라 샤드의 재구성이 필요할 수 있음
  - 크로스 샤드 트랜잭션: 여러 샤드에 걸쳐 있는 데이터를 처리하는 트랜잭션은 구현하기 어려움

* Redis

  * **Redis Cluster**를 통해 데이터를 여러 노드에 분산 가능

---

# 8. Replication

* Redis 데이터를 다른 Redis 서버에 복제하는 기능

* 기존 명칭

  * Master-Slave Replication

* 현재 주로 사용하는 명칭

  * **Primary-Replica**

* Primary

  * 기본 데이터를 관리하는 서버

* Replica

  * Primary의 데이터를 복제하는 서버

* 활용

  * 서버 장애 대비
  * 읽기 요청 분산
  * 가용성 향상

* 주의점

  * Redis 복제는 일반적으로 비동기 방식
  * 장애 발생 시점에 따라 일부 데이터 유실 가능

---

# 9. Redis Cluster

* 여러 Redis 노드를 묶어 하나의 시스템처럼 사용하는 환경

* 주요 목적

  * 데이터 분산
  * 부하 분산
  * 확장성 증가
  * 장애 대응
  * 높은 가용성 확보

* 단순한 **성능 부스터**의 개념은 아님

* 여러 서버가 역할과 데이터를 나누어 처리하는 구조

* 장점

  * 한 서버가 모든 데이터를 처리하지 않아도 됨
  * 데이터 증가에 따라 노드 추가 가능
  * 특정 노드 장애에 대응 가능

* 단점

  * 단일 Redis보다 구조가 복잡함
  * 서버 간 통신 필요
  * 복제 및 동기화 관리 필요
  * 네트워크 장애나 복제 지연 발생 가능

---

# 10. Redis의 단점

## 높은 메모리 비용

* 데이터를 주로 RAM에 저장

* 데이터 증가 시 많은 메모리 필요

* RAM은 SSD/HDD보다 상대적으로 비쌈

* 대용량 데이터 전체를 Redis에 저장하면 비용 증가

* 일반적인 사용 방식

  * 자주 사용하는 데이터 저장
  * 빠른 처리가 필요한 데이터 저장
  * 영구 데이터는 다른 DB와 함께 관리

---

# 11. 메모리 관리 방법

## TTL

* 데이터에 만료 시간 설정
* 불필요한 데이터가 계속 메모리를 차지하는 것을 방지

```redis
EXPIRE session:user:1 3600
```

* 1시간 후 자동 삭제

* 활용 예시

  * 로그인 세션
  * 인증 코드
  * 임시 데이터
  * API Cache
  * 게임 매칭 정보

---

## Eviction Policy

* Redis 메모리가 설정된 한도에 도달했을 때 적용되는 데이터 제거 정책

* 대표적인 방식

  * **LRU(Least Recently Used)**

* LRU

  * 최근에 가장 적게 사용된 데이터를 우선 제거

* Redis에서는 요구사항에 따라 다양한 Eviction Policy 설정 가능

---

# 12. 데이터 영속성

* Redis는 In-Memory DB이지만 디스크 저장 기능도 지원
* 서버 종료 또는 재시작 후 데이터 복구 가능

## RDB Snapshot

* 특정 시점의 Redis 데이터를 파일로 저장
* 일반적으로 `dump.rdb` 파일 사용
* 주기적인 백업에 적합

---

## AOF(Append Only File)

* Redis에서 발생한 데이터 변경 명령을 파일에 기록

예시:

```redis
SET user:1 "Alice"
SET user:2 "Bob"
DEL user:1
```

* 서버 재시작 시 기록된 명령을 다시 적용하여 데이터 복구

* 장점

  * RDB보다 데이터 유실 가능성을 줄일 수 있음

* 단점

  * 디스크 사용량 증가 가능
  * 설정에 따라 성능 비용 발생

---

# 13. 장애 대응 방법

## Replication

* Primary 데이터를 Replica에 복제
* Primary 장애 시 다른 노드 활용 가능

## Redis Cluster

* 데이터와 요청을 여러 노드에 분산
* 특정 서버 장애의 영향 감소

## RDB Snapshot

* Redis 데이터를 주기적으로 디스크에 저장
* 장애 발생 후 복구에 활용

## AOF

* Redis 데이터 변경 내용을 디스크에 기록
* 서버 재시작 시 데이터 복구에 활용

## 외부 DB와 함께 사용

* 중요한 데이터를 Redis에만 저장하지 않음

* MySQL, PostgreSQL 등의 영구 저장 DB와 함께 사용

* 게임 랭킹 예시

  * Redis → 빠른 실시간 랭킹 처리
  * MySQL/PostgreSQL → 실제 점수 영구 저장

---

# 14. Redis를 사용하기 좋은 상황

## Cache

* API 요청 결과
* DB 조회 결과

## Session

* 로그인 세션
* Access Token 관련 임시 데이터

## 게임 서버

* 실시간 랭킹
* 매칭 정보
* 접속 사용자 정보
* 게임 세션
* 임시 상태 데이터

## 인증

## 실시간 데이터

## Messaging
---

# 15. Redis와 관계형 DB

* Redis

  * 빠른 접근이 필요한 데이터 처리
  * 실시간 랭킹
  * Cache
  * Session
  * 인증 코드
  * 임시 데이터

* MySQL / PostgreSQL

  * 영구적으로 보존해야 하는 데이터 관리
  * 회원 정보
  * 점수 기록
  * 서비스 핵심 데이터

* 일반적으로 Redis가 관계형 DB를 완전히 대체하는 것이 아니라 함께 사용

---

# 핵심 정리

* Redis

  * **In-Memory 기반 Key-Value 데이터 저장소**
  * 빠른 읽기/쓰기 성능

* 주요 특징

  * 다양한 자료구조 지원
  * 실시간 처리에 적합
  * Cache, Session, Ranking, Messaging 등에 활용
  * TTL을 이용한 자동 데이터 삭제
  * RDB/AOF를 통한 데이터 영속성 지원
  * Replication을 통한 데이터 복제
  * Redis Cluster를 통한 확장 및 분산 처리

* 단점

  * 높은 메모리 비용
  * 장애 시 데이터 유실 가능성
  * 클러스터 환경 구성 시 관리 복잡성 증가

* 일반적인 사용 방식

  * 중요한 영구 데이터 → MySQL / PostgreSQL
  * 빠른 접근이 필요한 데이터 → Redis
