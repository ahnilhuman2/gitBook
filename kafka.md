# 메시지 큐, 동기 방식 비교 및 Kafka 정리

## 📚 메시지 큐, 동기 방식 비교 및 Kafka 정리

이 문서는 **메시지 큐**의 개념과 특징, 동기 방식과의 차이점, 그리고 분산 메시지 큐 시스템인 **Kafka**에 대해 정리한 내용.

***

### 1. 메시지 큐 (Message Queue)란?

메시지 큐는 **비동기적으로** 데이터를 주고받는 시스템입니다.

#### 주요 구성요소

* **Producer (생산자)**\
  메시지를 생성하여 큐에 저장하는 역할
* **Queue (큐)**\
  메시지를 임시로 저장하는 데이터 구조 (주로 FIFO 방식)
* **Consumer (소비자)**\
  큐에서 메시지를 꺼내 처리하는 역할

#### 특징

* **비동기 처리**
  * Producer는 메시지를 큐에 넣고 즉시 다음 작업을 수행
  * Consumer는 나중에 메시지를 꺼내 처리
* **부하 분산**
  * 트래픽이 몰리더라도 메시지를 큐에 저장해두고 순차적 또는 병렬적으로 처리 가능
* **확장성**
  * 여러 Consumer를 추가하여 처리량을 확장할 수 있음
* **장애 복구**
  * 메시지가 큐에 남아 있으면 장애 발생 시 재처리 가능

***

### 2. 동기 방식 vs. 메시지 큐 (비동기 방식)

#### 동기 방식 (Synchronous)

* **동작 원리**:\
  요청 후 응답이 완료될 때까지 기다림.
* **특징**:
  * 사용자가 결과를 기다려야 하므로 응답 지연 발생
  * 서버가 요청을 순차적으로 처리 → 병목 현상 발생 가능
  * 구현 구조는 단순하지만 확장성에 한계가 있음.
* **예시**:\
  사용자가 주문을 요청하면, 주문 처리 완료까지 기다려야 하는 시스템.

#### 메시지 큐를 이용한 비동기 방식 (Asynchronous)

* **동작 원리**:\
  요청 시 메시지를 큐에 저장한 후 즉시 응답을 반환하고, 나중에 별도의 Consumer가 메시지를 처리.
* **특징**:
  * 사용자에게 즉시 응답 제공
  * 서버 부하 분산 및 확장이 용이
  * 장애 복구 가능 (큐에 저장된 메시지로 재처리)
* **예시**:\
  주문 요청 시 주문 정보가 메시지 큐에 저장되고, 백그라운드에서 별도의 프로세스가 주문을 처리하는 시스템.

#### 비교 표

| 항목        | 동기 방식                      | 메시지 큐 (비동기 방식)            |
| --------- | -------------------------- | ------------------------- |
| **요청 처리** | 요청 → 즉시 처리 후 결과 반환 (대기 필요) | 요청 → 메시지 큐에 저장 → 나중에 처리   |
| **응답 시간** | 처리 완료까지 대기 (느림)            | 즉시 응답 (빠름)                |
| **서버 부하** | 스레드 블로킹으로 부하 증가            | 부하 분산 (여러 Consumer 추가 가능) |
| **확장성**   | 한계가 있음                     | Consumer 추가로 확장 용이        |
| **장애 복구** | 서버 장애 시 데이터 손실 위험          | 큐에 남은 메시지로 재처리 가능         |

***

### 3. Kafka란?

**Kafka**는 대용량 데이터를 빠르고 안정적으로 처리하기 위한 **분산 메시지 큐 시스템**.

#### Kafka의 주요 특징

* **분산 시스템**\
  여러 서버(브로커)로 구성되어 수평 확장이 매우 용이하다.
* **내구성**\
  메시지를 디스크에 저장하여 영구 보존, 데이터 유실 최소화.
* **높은 처리량**\
  초당 수십만 건의 메시지를 처리할 수 있어, 대규모 로그 및 이벤트 스트리밍에 최적이.
* **구성 요소**:
  * **Producer**: 메시지를 생성하여 Kafka 토픽에 전송.
  * **Broker**: 메시지를 저장하고 관리하는 Kafka 서버.
  * **Topic**: 메시지가 전송되는 논리적 채널.
  * **Consumer**: 토픽에서 메시지를 구독하여 처리.

#### Kafka vs. Redis 메시지 큐 비교

| 비교 항목      | Redis 메시지 큐              | Kafka 메시지 큐                  |
| ---------- | ------------------------ | ---------------------------- |
| **처리 방식**  | 메모리 기반의 간단한 비동기 큐        | 분산 시스템 기반의 로그/이벤트 스트리밍 처리    |
| **데이터 보존** | 기본적으로 휘발성 (옵션에 따라 보존 가능) | 디스크 기반 저장으로 영구 보존, 재처리 용이    |
| **확장성**    | 단일 서버 또는 소규모 클러스터에 적합    | 분산 환경에서 수평 확장이 용이            |
| **사용 예시**  | 빠른 주문 처리, 실시간 알림         | 로그 분석, 대용량 이벤트 스트리밍, 트랜잭션 처리 |

***

### 4. 결론

* **동기 방식**은 단순하지만, 사용자가 결과를 기다려야 하고 서버 부하가 증가하면 병목 현상이 발생할 수 있습니다.
* \*\*메시지 큐(비동기 방식)\*\*는 사용자에게 빠른 응답을 제공하며, 작업을 백그라운드에서 처리하여 부하 분산과 확장성이 뛰어납니다.
* **Kafka**는 대규모 데이터 처리와 로그/이벤트 스트리밍에 적합한 분산 메시지 큐 시스템으로, 높은 처리량과 내구성을 자랑합니다.

👉 **시스템 요구사항과 규모에 따라 동기 방식과 비동기 메시지 큐를 적절히 선택하며, 대용량 처리가 필요할 경우 Kafka를 활용하는 것이 효율적입니다.**
