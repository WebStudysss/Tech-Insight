# 🌐 무상태(Stateless) 설계의 핵심 정리

## 1️⃣ 개요
웹 애플리케이션에서 **Stateless(무상태)** 란,  
서버가 **이전 요청의 상태나 컨텍스트를 저장하지 않는 설계 방식**을 말한다.  
즉, **각 요청(Request)** 은 서로 완전히 **독립적**이며,  
서버는 매번 “처음 보는 요청”처럼 처리한다.

> 💡 REST, HTTP, MSA 구조 모두 이 ‘무상태성’을 기반으로 한다.

---

## 2️⃣ 상태(State)란?
- **State(상태)**: 클라이언트와 서버 간의 ‘대화 맥락’ 또는 ‘세션 정보’
- 예를 들어 로그인 후 사용자 정보를 유지하는 세션, 장바구니 내역, 진행 중인 트랜잭션 등이 상태 정보에 해당한다.

| 구분 | Stateful | Stateless |
|------|-----------|------------|
| 요청 간 맥락 | 유지됨 (세션/쿠키 등) | 없음 (독립 요청) |
| 서버 부하 | 높음 (메모리 관리 필요) | 낮음 (단순 처리) |
| 확장성 | 낮음 (세션 공유 필요) | 높음 (로드밸런싱 용이) |
| 장애 복구 | 어려움 (세션 일관성 문제) | 쉬움 (무상태 서버 교체 가능) |

---

## 3️⃣ 무상태(Stateless) 설계 원리

### (1) 요청은 항상 ‘완전해야 한다’
> 요청 하나로 서버가 필요한 모든 정보를 알아야 한다.

예
```http
GET /api/user/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```
- JWT를 이용해 클라이언트가 인증 정보를 직접 담아 보낸다.
- 서버는 이전 세션을 기억하지 않고, 토큰만으로 인증을 수행한다.

### (2) 서버는 ‘세션’을 저장하지 않는다
- 로그인 상태를 서버 메모리에 저장하지 않고,  
  **토큰 기반 인증(JWT)** 으로 대체한다.
- 장바구니, 임시 데이터 등은 **Redis 같은 외부 저장소**에 위임한다.

> 💬 즉, 서버는 오직 “입력 → 처리 → 응답”만 담당하며,  
> 상태관리는 외부로 위임한다.

---

### (3) 클라이언트 중심 설계
- 상태 유지는 클라이언트가 담당하거나,
- 필요 시 서버 간 공유 가능한 **Session Store (예: Redis)** 를 사용한다.

---

## 4️⃣ 동작 예시 (Stateless vs Stateful 시퀀스 다이어그램)

### 🧩 Stateful 구조 — 세션 기반 로그인

```mermaid
sequenceDiagram
    participant Client as 클라이언트
    participant LB as 로드 밸런서
    participant ServerA as 서버 A
    participant ServerB as 서버 B

    Client->>LB: 로그인 요청 (/login)
    LB->>ServerA: 요청 전달
    ServerA->>ServerA: 사용자 인증 후 세션 생성 (sessionId=abc123)
    ServerA-->>Client: 세션 쿠키 발급 (Set-Cookie: JSESSIONID=abc123)
    
    Note over Client,ServerA: 이후 요청에서 쿠키(JSESSIONID=abc123)를 포함
    
    Client->>LB: 프로필 요청 (/profile, 쿠키 포함)
    LB->>ServerA: 세션이 존재하는 서버로 라우팅됨
    ServerA->>ServerA: 세션 정보 조회
    ServerA-->>Client: 사용자 프로필 응답

    Note over LB,ServerB: 만약 LB가 ServerB로 보낸다면?
    Client->>LB: 프로필 요청 (/profile, 쿠키 포함)
    LB->>ServerB: 요청 전달
    ServerB->>ServerB: ❌ 세션 정보 없음
    ServerB-->>Client: 401 Unauthorized (로그인 만료)
```

> 요약
서버가 세션을 메모리에 저장하므로, 요청이 항상 같은 서버로 가야 정상 처리됨.
서버 A가 죽거나 LB가 Server B로 라우팅하면 세션 일관성 깨짐.
확장성 낮고, Sticky Session 필요.

### 🧩 Stateless 구조 — JWT 기반 로그인
```mermaid
sequenceDiagram
    participant Client as 클라이언트
    participant LB as 로드 밸런서
    participant ServerA as 서버 A
    participant ServerB as 서버 B

    Client->>LB: 로그인 요청 (/login)
    LB->>ServerA: 요청 전달
    ServerA->>ServerA: 사용자 인증 후 JWT 생성 (서명 포함)
    ServerA-->>Client: JWT 반환 (Authorization: Bearer eyJhbGciOiJIUzI1Ni...)

    Note over Client: 이후 요청마다 JWT를 헤더에 포함시킴
    
    Client->>LB: 프로필 요청 (/profile, Authorization 헤더 포함)
    LB->>ServerB: 요청 전달 (무작위 라우팅 가능)
    ServerB->>ServerB: JWT 서명 검증 (서버 상태 필요 없음)
    ServerB-->>Client: 사용자 프로필 응답 (정상 처리)

    Note over Client,ServerB: 모든 서버가 같은 JWT 서명 키를 공유하므로,<br>서버 간 상태 공유 불필요

```
>요약
서버는 세션을 전혀 저장하지 않음.
클라이언트가 JWT로 상태를 직접 유지하므로,
요청이 어느 서버로 가든 인증 가능하고 수평 확장(Scale-out)에 유리하다.


✅ 요청 간 맥락이 없어도, 토큰 기반으로 어디서나 처리 가능
✅ 서버 간 세션 공유나 Sticky Session이 필요 없음


#### ⚖️ 구조적 비교 요약

| 항목 | Stateful (세션 기반) | Stateless (JWT 기반) |
|------|----------------------|----------------------|
| 상태 저장 위치 | 서버 메모리 | 클라이언트(JWT) |
| 요청 독립성 | 낮음 | 높음 |
| 확장성 | 세션 공유 필요 → 낮음 | 서버 간 독립 → 높음 |
| 장애 복구 | 세션 유실 위험 | 즉시 처리 가능 |
| REST 철학 부합 | ❌ | ✅ |
| 적용 예시 | 전통적인 JSP/Servlet, Spring Session | REST API, MSA, Serverless, Gateway |

---

> 🌐 **핵심 정리**  
> Stateless 구조는 요청 간 독립성을 유지하고,  
> 서버를 ‘기억하지 않아도 되는 노드’로 만들어  
> 확장성과 복원력을 극대화한다.



--- 

## 5️⃣ 왜 중요한가? (Stateless의 가치)

| 항목 | 설명 |
|------|------|
| 🔁 **확장성(Scalability)** | 서버 간 세션 공유 필요가 없어 수평 확장(Scale-out)에 유리 |
| 💥 **복원력(Fault Tolerance)** | 서버 다운 시에도 다른 인스턴스가 즉시 요청 처리 가능 |
| 🚀 **성능(Performance)** | 세션 조회나 복제 비용이 없음 |
| 🧩 **단순성(Simplicity)** | 요청-응답 구조 단순, 유지보수 용이 |
| 🧱 **표준성(RESTful)** | REST API의 근간이자 HTTP의 본래 철학에 부합 |

---

## 6️⃣ 실무 적용 포인트

| 상황 | 권장 설계 |
|------|------------|
| 인증 유지 | JWT + Refresh Token |
| 로그인 상태 표시 | 클라이언트에서 상태 관리 (e.g. localStorage) |
| 장바구니, 임시 데이터 | Redis, DB 등 외부 저장소 활용 |
| 서버 확장 | 무상태 서버로 구성, Sticky Session 피하기 |
| MSA 간 인증 | Access Token 공유 or API Gateway 단에서 검증 |

---

## 7️⃣ 주의할 점 (Stateless의 오해)

| 오해 | 올바른 이해 |
|------|--------------|
| “무상태면 로그인 못한다” | ❌ → 상태를 서버가 아닌 **클라이언트/외부 저장소**에 둔다 |
| “모든 상태는 제거해야 한다” | ❌ → 단, **서버가 상태를 ‘기억하지 않는다’**가 핵심 |
| “Stateful이 나쁘다” | ❌ → 일부 실시간 서비스(게임, 트레이딩)는 상태 필요 |

---

## 8️⃣ 확장 개념 (더 깊게 가보자)

| 주제 | 설명 |
|------|------|
| 🧩 **Session Clustering** | 상태를 공유하기 위해 여러 서버가 세션 복제하는 구조 (비추천) |
| 🪶 **Sticky Session** | 같은 클라이언트를 항상 같은 서버로 라우팅하는 방식 |
| 🧱 **Token-based Auth (JWT)** | 클라이언트가 상태 정보를 포함해 요청 (무상태 핵심 구현체) |
| 🧭 **Idempotency (멱등성)** | 무상태 API의 중요한 속성, 같은 요청은 항상 같은 결과 |
| 🧰 **MSA 설계 원칙 중 Stateless** | 마이크로서비스 간 독립 배포·확장을 위한 핵심 원칙 |

---

## 9️⃣ 면접/스터디에서 자주 묻는 질문

| 질문 | 핵심 답변 |
|------|------------|
| “왜 Stateless가 중요한가요?” | 서버 확장성과 복원력을 높이고, RESTful 원칙을 유지하기 위해 |
| “로그인 상태를 Stateless하게 어떻게 유지하나요?” | JWT를 통해 클라이언트가 상태를 직접 전달 |
| “Stateful과 Stateless의 중간 설계가 가능한가요?” | 가능함. 외부 세션 스토리지(Redis)를 통한 하이브리드 구조 |
| “Sticky Session은 왜 피해야 하나요?” | 서버 간 부하 불균형 및 확장성 저하 유발 |

### 💬 Q1. “왜 Stateless가 중요한가요?”
**핵심 답변**  
Stateless는 서버 확장성과 복원력을 높이고, RESTful 원칙을 유지하기 위해 필요하다.

**설명**  
- **확장성(Scalability)**: 요청에 세션 의존이 없으므로 서버를 여러 대로 쉽게 나눌 수 있다.  
  예를 들어, 트래픽 증가 시 인스턴스를 추가해도 세션 공유가 필요 없어 바로 부하분산이 가능하다.  
- **복원력(Fault Tolerance)**: 특정 서버가 장애를 일으켜도, 다른 서버가 동일 요청을 바로 처리할 수 있다.  
- **유지보수성(Maintainability)**: 상태 관리 로직이 없으므로 서버 코드는 단순해지고, 배포 시에도 세션 데이터 손실 우려가 없다.  
- **REST 철학 부합**: REST는 HTTP 프로토콜의 기본 특성인 Stateless를 기반으로 한다.  
  “서버는 요청 간 컨텍스트를 기억하지 않는다”가 핵심 원리다.

---

### 💬 Q2. “로그인 상태를 Stateless하게 어떻게 유지하나요?”
**핵심 답변**  
서버가 세션을 저장하지 않고, **JWT(Json Web Token)** 을 이용해 클라이언트가 인증 상태를 직접 유지한다.

**설명**  
- 로그인 시 서버는 사용자 정보를 기반으로 JWT를 생성해 클라이언트에게 전달한다.  
- 이후 요청마다 `Authorization: Bearer <JWT>` 형태로 토큰을 함께 보낸다.  
- 서버는 토큰을 검증해 유효하면 해당 사용자를 인증한다.  
  (이때 DB 접근 없이 토큰 서명만으로 인증 가능)
- 토큰 만료 시 **Refresh Token** 으로 재발급하는 구조를 사용하면, 보안성과 유연성을 동시에 확보할 수 있다.  

> 💡 이 구조 덕분에 서버는 로그인 상태를 기억할 필요가 없으며,  
> 수평 확장 시에도 인증 정보가 서버 간에 공유될 필요가 없다.

---

### 💬 Q3. “Stateful과 Stateless의 중간 설계가 가능한가요?”
**핵심 답변**  
가능하다. 대표적인 방법은 **외부 세션 스토리지(Redis 등)** 를 사용하는 **하이브리드 구조**다.

**설명**  
- 완전 Stateless로 설계하기 어려운 경우(예: 장바구니, 결제 과정, 트랜잭션 세션 등),  
  세션 데이터를 서버 메모리가 아닌 **Redis** 같은 외부 인메모리 DB에 저장한다.  
- 이 방식은 클라이언트 요청이 어느 서버로 가든 동일한 세션 데이터를 조회할 수 있어,  
  **확장성과 일관성의 균형**을 맞춘다.  
- 예시
```
[Client] → [LoadBalancer]
├─▶ [Server A] → Redis에서 세션 조회
└─▶ [Server B] → 동일 세션 Redis에서 공유
```

> ⚙️ 실무에서는 “Stateless 서버 + Stateful 외부 저장소”로 혼합 설계하는 경우가 많다.

---

### 💬 Q4. “Sticky Session은 왜 피해야 하나요?”
**핵심 답변**  
Sticky Session은 **확장성 저하**와 **장애 복원성 저하**를 유발하기 때문에 지양해야 한다.

**설명**  
- **Sticky Session**: 클라이언트가 항상 같은 서버로 라우팅되도록 하는 방식  
(예: A 서버에서 로그인하면, 이후 요청도 항상 A 서버로 감)
- 단점:
1. 특정 서버에 트래픽이 몰려 **부하 불균형**이 발생한다.  
2. 해당 서버가 장애를 일으키면 세션이 사라져 사용자 로그인이 풀린다.  
3. 서버 추가 시 새로운 노드는 기존 세션을 모름 → 확장성 저하.
- 반면 Stateless는 모든 서버가 동일하게 요청을 처리하므로,  
장애 복구 및 오토스케일링이 훨씬 간단하다.

> 💬 Sticky Session은 “임시방편”으로만 쓰며,  
> 장기적으로는 세션 스토리지나 JWT로 전환하는 것이 이상적이다.

---

### 💬 Q5. “Stateless 설계 시 보안 문제는 없나요?”
**핵심 답변**  
있다. 특히 토큰 탈취, 재사용 공격 등에 대비해야 한다.

**설명**  
- **토큰 탈취**: 클라이언트 저장소(localStorage, sessionStorage)에 저장된 JWT가 노출될 위험  
→ 해결: HttpOnly 쿠키 사용, 토큰 암호화, 만료시간 단축  
- **토큰 위조**: 서명 검증 미흡 시 위조된 JWT로 인증 가능  
→ 해결: 공개키-비공개키(Public/Private Key) 기반 검증  
- **Refresh Token 도난**: 장기 토큰이 탈취되면 지속 인증 가능  
→ 해결: Refresh Token Rotation (재발급 시 이전 토큰 폐기)

> 🔒 Stateless 구조는 보안 로직을 토큰 단에서 강화해야 한다.

---

### 💬 Q6. “Stateless가 항상 옳은가요?”
**핵심 답변**  
아니다. 무상태는 대부분의 웹 서비스에 유리하지만, **실시간 연결이나 상태 기반 로직**에는 Stateful이 필요하다.

**설명**  
- **Stateful이 필요한 경우**
- 실시간 게임, 채팅, 트레이딩 시스템 등 “연결 지속성”이 중요한 서비스
- 세션 단위로 지속적인 데이터 교환이 필요한 경우
- **Stateless가 유리한 경우**
- REST API, 웹 백엔드, 인증 서버, API Gateway, 배치 등
- 실제로는 **Stateless + 일부 Stateful**을 혼합해 설계한다.

---

## 10️⃣ 요약

> 🌐 **Stateless = 요청 간 독립성 보장 + 서버의 단순화 + 확장성 극대화**

- 요청 하나로 완결되어야 한다.
- 상태는 클라이언트나 외부 스토리지로 옮긴다.
- 서버는 언제든 교체·확장 가능한 ‘가벼운 노드’로 설계한다.

---

## 💡 다음 확장 학습 주제 추천
1. **Idempotency (멱등성)** – 무상태 API에서 왜 중요한가  
2. **JWT vs Session 비교** – 인증의 상태 관리 차이  
3. **Sticky Session과 Redis 세션 클러스터링**  
4. **API Gateway의 Stateless 인증 흐름 설계**  
5. **서버리스(Serverless) 아키텍처에서의 무상태 개념**

---


# 📌 확장 학습 주제 심화
## 1) Idempotency(멱등성) — 무상태 API에서 재시도 안전성 보장

### 한 줄 정의
**같은 요청을 여러 번 보내도 최종 결과가 동일**하도록 만드는 속성.  
네트워크 재시도·중복 전송·타임아웃 재요청 상황에서도 **중복 부작용**(이중 결제, 이중 발송 등)을 막는다.

### 왜 중요한가?
- **무상태 서버**는 요청 간 맥락을 기억하지 않으므로, **클라이언트/중간자(게이트웨이/리트라이러)** 가 재시도할 때도 **안전해야 함**.
- 클라우드/분산 환경의 기본 전달 보장(at-least-once, 중복 가능)과 **자연스럽게 상호 보완**.

### HTTP 메서드와 멱등성
| 메서드 | 표준상 멱등성 | 비고 |
|---|---|---|
| GET/HEAD | ✅ | 조회만 수행해야 함 |
| PUT | ✅ | “리소스를 이 값으로 만들어라(Upsert)” 의미일 때 |
| DELETE | ✅ | 여러 번 호출해도 최종 상태는 “없음” |
| OPTIONS | ✅ | 프리플라이트 조회 |
| POST | ❌(기본) | **Idempotency-Key**로 멱등화 가능 |
| PATCH | ❌(기본) | 조건부 갱신(ETag/If-Match)로 멱등화 가능 |

### 멱등화 핵심 패턴
1. **리소스 키 기반(자연/비즈니스 키) Upsert**
   - `PUT /users/42` : “항상 42번 사용자를 이 본문으로 만든다”
   - DB 유니크 제약 + UPSERT로 중복을 구조적으로 제거

2. **POST + Idempotency-Key 헤더**
   - 클라이언트가 **요청 단위의 전역 고유키**를 생성: `Idempotency-Key: 7f2c-...`
   - 서버/게이트웨이는 (엔드포인트, 키) 조합으로 **처리 기록 + 응답 캐시** 저장
   - **동일 키 + 동일 페이로드** 재요청 → **이전 응답 재생산**(200/201 등)
   - **동일 키 + 상이한 페이로드** → **409 Conflict**(정책적으로 거부)

3. **조건부 갱신(Optimistic Locking)**
   - `If-Match: <ETag>`를 이용한 **버전 충돌 방지**
   - 실패 시 412 Precondition Failed → 재시도 루프에서 안전

4. **멱등 컨슈머(메시지 처리)**
   - 메시지/이벤트마다 **Message-ID** 저장(“inbox table”)
   - 이미 처리한 ID면 스킵(또는 안전한 재실행)
   - **Transactional Outbox/Inbox** 패턴으로 **정확히 한 번처럼** 보이게

### 구현 로드맵(POST 결제 예)
1) 스키마(예시)
```sql
CREATE TABLE idempotency (
  endpoint VARCHAR(120) NOT NULL,
  idem_key VARCHAR(64) NOT NULL,
  request_hash CHAR(64) NOT NULL,
  response_status SMALLINT NOT NULL,
  response_body TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (endpoint, idem_key)
);
```

2) 처리 알고리즘
- 요청 진입 → `(endpoint, idem_key)` 조회  
  - **없음** → 처리 시작 전 **행 선점(INSERT)** 후 실제 비즈니스 수행 → **응답 저장**
  - **있음** → `request_hash` 같으면 **캐시된 응답 반환**, 다르면 **409 Conflict**
- **TTL/보존 기간** 운영(예: 결제·주문은 24h~72h, 일반 POST는 수 시간)

3) 동시성/경쟁 해결
- **DB 유니크키** + 트랜잭션으로 “이 키는 내가 먼저 잡았다” 보장
- 분산락(필요 시)보다 **단일 DB 제약**이 간단·견고

### 멱등화 체크리스트
- [ ] 클라이언트가 **Idempotency-Key** 생성/전달
- [ ] 서버가 키별 **요청 해시/응답** 저장 및 **동일 응답 리턴**
- [ ] 키 재사용 정책(동일/상이한 페이로드 처리 기준) 명시
- [ ] 키 **보존 기간/정리 배치** 운영
- [ ] 외부 부작용(이메일, 푸시, 결제 승인)도 **키 단위로 중복 방지**
- [ ] 장애/타임아웃 재시도 경로에 모두 적용(게이트웨이, 워커, MQ 컨슈머)


<hr>

## 2) API Gateway 무상태 인증 흐름 설계 — “인증은 게이트웨이에서, 상태는 토큰에”

### 목표
- **게이트웨이 레벨에서 JWT 검증**으로 인증을 **완전 무상태**로 처리
- 내부 서비스는 **신원/권한이 주입된 요청**만 신뢰하고 비즈니스에 집중

### 역할 분담
- **클라이언트**: Access Token(JWT) 보관·전달  
- **IdP/Auth 서버**: 로그인/권한 부여, **JWT 발급 + JWKS 공개**  
- **API Gateway**: 서명·클레임 검증, **정책/권한 결정(PDP/PEP)**, 컨텍스트 주입  
- **마이크로서비스**: 게이트웨이가 보장한 **주체/스코프** 신뢰, **세션 저장 없음**

### 무상태 설계 키 포인트
- **서명 검증만으로 인증**: `iss/aud/exp/nbf` 체크 + **JWKS 캐시**
- **Key Rotation**: `kid` 기반으로 **JWKS 자동 갱신**, 실패 시 **백오프 + 로컬 캐시 유지**
- **Clock Skew** 허용(예: ±2분), 토큰 만료 오차 최소화
- **Claims→권한 매핑**: `scope`, `roles`, `tenant`, `sub` 등으로 RBAC/ABAC
- **컨텍스트 주입**: 내부로 **원본 JWT** 전달 또는 **정제된 헤더**(예: `X-User-Id`, `X-Scopes`)
- **Opaque 토큰** 사용 시**만** IdP **Introspection**(네트워크 의존) — 가능하면 **JWT 권장**
- **게이트웨이 측 상태 저장 금지**: 세션/로그인 컨텍스트 보관 ❌ (레이트리밋 카운터는 예외적 저장소 사용 가능)

### 시퀀스(로그인 후 보호 API 호출)
```mermaid
sequenceDiagram
    participant Client as 클라이언트
    participant IdP as 인증 서버(IdP)
    participant GW as API 게이트웨이
    participant Svc as 내부 서비스

    Client->>IdP: 로그인 (OIDC/OAuth 2.1)
    IdP-->>Client: Access Token(JWT) 발급 + (선택)Refresh Token

    Client->>GW: 보호 API 호출 (Authorization: Bearer <JWT>)
    GW->>GW: JWKS 캐시에서 kid로 공개키 가져와 JWT 서명 검증
    GW->>GW: iss/aud/exp/nbf/Scope 등 클레임 검증 + 정책 평가(RBAC/ABAC)
    alt 검증 성공
        GW->>Svc: 요청 전달 (원본 JWT 또는 주입 헤더 포함)
        Svc-->>GW: 비즈니스 응답
        GW-->>Client: 200 OK (응답)
    else 검증 실패
        GW-->>Client: 401/403 에러
    end
```

#### Spring Cloud Gateway 간단 예시(YAML)
```yml
spring:
  cloud:
    gateway:
      default-filters:
        - RemoveRequestHeader=Cookie
        - DedupeResponseHeader=Access-Control-Allow-Origin Access-Control-Allow-Credentials
      routes:
        - id: user-api
          uri: http://user-service
          predicates:
            - Path=/api/users/**
          filters:
            - name: JwtAuth
              args:
                jwk-set-uri: https://idp.example.com/.well-known/jwks.json
                expected-issuer: https://idp.example.com
                expected-audience: user-api
                clock-skew: 120
                inject-headers:
                  X-User-Id: "#{claims['sub']}"
                  X-Scopes: "#{claims['scope']}"

```


#### 운영시 체크하면 좋은 체크리스트
- [ ] **JWKS 캐시** 및 **키 회전(kid)** 대응  
- [ ] **iss/aud/exp/nbf** 검증 + **leeway** 허용  
- [ ] **권한 정책**(경로/메서드/스코프) 중앙화  
- [ ] **멀티 테넌트**라면 `iss`/`aud` 테넌트별 매핑  
- [ ] **원본 JWT 전달/마스킹** 정책(PII 최소화)  
- [ ] **레이트리밋/서킷브레이커/CORS**는 게이트웨이에서 일괄 처리  
- [ ] **토큰 크기/헤더 크기 제한** 주의 (프록시/로드밸런서 설정 일치)

<hr>

### NGINX/OpenResty(Lua) 컨셉
- `access_by_lua*`에서 **Authorization** 파싱 → **JWKS 캐시**로 서명 검증  
- 실패 시 즉시 **401 Unauthorized**, 성공 시 `X-User-*` 헤더 주입 후 **프록시 패스**


<hr>

좀더 상세히 들어간다면 ?

## 🧩 NGINX / OpenResty(Lua) 기반 무상태 인증 게이트웨이 설계

### 🎯 목적
JWT 기반 인증을 **NGINX 레이어에서 직접 수행**해,  
내부 서비스에는 **검증된 요청만 전달**하고,  
서버 자체는 **완전 무상태(Stateless)** 로 유지한다.

---

### ⚙️ 구조 개요

```mermaid
sequenceDiagram
    participant Client as 클라이언트
    participant NGINX as NGINX(OpenResty + Lua)
    participant JWKS as JWKS 서버 (Auth Provider)
    participant Service as 내부 서비스

    Client->>NGINX: Authorization: Bearer <JWT>
    NGINX->>NGINX: access_by_lua*에서 JWT 파싱
    NGINX->>JWKS: kid 기반 JWKS 키 요청 (캐시 확인)
    JWKS-->>NGINX: JWK 공개키 응답
    NGINX->>NGINX: 서명 검증 + iss/aud/exp/nbf 검증
    alt 성공
        NGINX->>NGINX: X-User-Id, X-Scopes 등 헤더 주입
        NGINX->>Service: 프록시 패스 (내부 API 전달)
        Service-->>NGINX: 응답
        NGINX-->>Client: 200 OK
    else 실패
        NGINX-->>Client: 401 Unauthorized
    end

```

## 🧠 핵심 로직 (Lua 단계별 설명)

1. Authorization 헤더 추출

```lua
local jwt_token = ngx.var.http_authorization
if not jwt_token or not jwt_token:find("Bearer") then
    return ngx.exit(ngx.HTTP_UNAUTHORIZED)
end
local token = jwt_token:gsub("Bearer ", "")
```

2. JWT 파싱 및 헤더 확인 (kid 확인)
```lua
local cjson = require "cjson"
local jwt = require "resty.jwt"
local jwt_obj = jwt:load_jwt(token)
local kid = jwt_obj and jwt_obj.header and jwt_obj.header.kid
if not kid then
    ngx.log(ngx.ERR, "JWT kid not found")
    return ngx.exit(ngx.HTTP_UNAUTHORIZED)
end
```

3. JWKS 키 캐시 확인

```lua
local jwks_cache = ngx.shared.jwks_cache
local jwk_key = jwks_cache:get(kid)
if not jwk_key then
    local http = require "resty.http"
    local httpc = http.new()
    local res, err = httpc:request_uri("https://idp.example.com/.well-known/jwks.json")
    if not res or res.status ~= 200 then
        ngx.log(ngx.ERR, "JWKS fetch failed: ", err)
        return ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)
    end
    local jwks = cjson.decode(res.body)
    for _, key in ipairs(jwks.keys) do
        jwks_cache:set(key.kid, cjson.encode(key), 3600)
    end
    jwk_key = jwks_cache:get(kid)
    if not jwk_key then
        return ngx.exit(ngx.HTTP_UNAUTHORIZED)
    end
end
```

4. 서명 검증 및 클레임 유효성 검사

```lua
local key_obj = cjson.decode(jwk_key)
local jwt = require "resty.jwt"
local validators = require "resty.jwt-validators"

local jwt_obj = jwt:verify_jwk_obj(key_obj, token, {
    lifetime_grace_period = 120,  -- exp 오차 허용
    iss = "https://idp.example.com",
    aud = "my-api"
})

if not jwt_obj.verified then
    ngx.log(ngx.ERR, "JWT invalid: ", jwt_obj.reason)
    return ngx.exit(ngx.HTTP_UNAUTHORIZED)
end
```

5. 검증된 정보 헤더 주입 후 내부 서비스로 전달

```lua
ngx.req.set_header("X-User-Id", jwt_obj.payload.sub)
ngx.req.set_header("X-Scopes", table.concat(jwt_obj.payload.scope or {}, " "))
ngx.req.set_header("X-Tenant", jwt_obj.payload.tenant or "default")
```

### 🧩 캐싱 & 성능 전략
| 항목 | 설명 |
|------|------|
| **JWKS 캐시** | `ngx.shared.DICT`에 `kid`별 JWK JSON 저장 (TTL: 1h~6h) |
| **백오프 로직** | JWKS 서버 장애 시 기존 캐시 유지 (`ngx.timer.at` 비동기 갱신) |
| **로컬 캐시 백업** | 초기 부팅 시 디스크 캐시 복원 (`lua-resty-lock` + 파일캐시) |
| **서명 알고리즘 제한** | `RS256`만 허용 (HS256 금지) |
| **exp/nbf/iat 오차 허용** | ±120초 leeway |

---

### 🔒 보안 포인트
| 항목 | 설명 |
|------|------|
| **공개키 검증** | JWKS는 TLS 위에서만 요청, HTTP 비허용 |
| **JWT 크기 제한** | `client_max_body_size`, `large_client_header_buffers` 설정 |
| **PII 최소화** | 민감정보(`email`, `name`)는 X-User-* 헤더로 전달하지 않음 |
| **CORS 정책 통합** | 게이트웨이에서 Origin/Headers/Methods 허용 제어 |
| **Rate Limiting / Circuit Breaker** | Lua 모듈(`lua-resty-limit-traffic`, `lua-resty-circuit-breaker`)로 처리 |



> 💡 **정리**
> - OpenResty는 “NGINX + Lua” 조합으로 **무상태 인증 로직을 네트워크 계층에서 수행**할 수 있는 강력한 도구다.  
> - 서버 애플리케이션이 인증을 직접 처리하지 않아도, 게이트웨이 단에서 **JWT 검증, 클레임 주입, 접근 정책 결정**을 전부 수행할 수 있다.  
> - 이로써 전체 아키텍처가 **Stateless + Secure + Scalable** 해진다.
