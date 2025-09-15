![](https://velog.velcdn.com/images/ohoh7391/post/fce4e9cd-5abd-4b17-9870-a42d8dafee5c/image.png)

# Proxy 정리 가이드

## 1. 개요
프록시(Proxy)는 **클라이언트와 서버 사이에서 통신을 대신 중계해주는 서버**다.  
단순히 요청을 전달하는 것뿐만 아니라 **보안, 성능 개선, 트래픽 제어** 등의 역할을 한다.  

- **Forward Proxy (정방향 프록시)**: 클라이언트 → 프록시 → 서버  
- **Reverse Proxy (역방향 프록시)**: 클라이언트 → 프록시 → 서버(실제 서버는 숨겨짐)

---

## 2. 아키텍처
```
[Client] ⇄ [Forward Proxy] ⇄ [Server]

[Client] ⇄ [Reverse Proxy] ⇄ [Backend Servers]
```

- Forward Proxy: 내부 사용자가 외부 서버에 직접 접근하지 않고, 프록시를 통해 요청

- Reverse Proxy: 외부 사용자가 프록시를 서버처럼 인식하고, 실제 서버는 백엔드에 숨겨짐


## 3. 특징
### Forward Proxy
- 내부망 사용자들의 외부 인터넷 접근 제어  
- 차단된 사이트 접근 가능 (IP 우회)  
- 클라이언트 신원 보호 (익명성 제공)  

### Reverse Proxy
- 외부 사용자가 직접 서버에 접근하지 않음  
- 서버 부하 분산(Load Balancing)  
- SSL 오프로딩(암호화/복호화 대리 처리)  
- 캐싱을 통한 성능 최적화  
- WAF(Web Application Firewall)로 보안 강화  

---

## 4. 활용 사례
- **Forward Proxy**
  - 회사 내부망 → 인터넷 접근 통제  
  - 특정 국가/지역에서 차단된 서비스 접근  
  - 연구·교육 환경에서 트래픽 모니터링  

- **Reverse Proxy**
  - **NGINX, Apache HTTP Server**: 트래픽 분산 및 캐싱  
  - **CDN (Cloudflare, Akamai 등)**: 콘텐츠 캐싱과 보안  
  - **API Gateway 대체**: 인증, 라우팅, 트래픽 제어  

---

## 5. Proxy vs API Gateway
| 구분            | Proxy                        | API Gateway                             |
|-----------------|------------------------------|-----------------------------------------|
| 주요 역할       | 요청 중계, 캐싱, 보안         | API 인증, 라우팅, 트래픽 관리            |
| 범위            | 네트워크 계층 전반            | 애플리케이션 계층(API 레벨)              |
| 사용 예시       | Nginx Reverse Proxy, Squid    | AWS API Gateway, Kong, Spring Cloud GW   |

---

## 6. 운영 및 실무 포인트
- Reverse Proxy는 **로드밸런서와 비슷하지만 더 다양한 기능**을 제공  
- SSL 인증서는 보통 Reverse Proxy 단에서 처리 → 서버 부담 감소  
- 캐싱 전략(정적 리소스, API 응답) 설계 시 성능 최적화 가능  
- 프록시 서버 로그는 **모니터링·보안 분석 포인트**로 활용됨  

---

## 7. 면접 질문
1. Forward Proxy와 Reverse Proxy의 차이점은?  
2. 프록시 서버를 사용하면 성능이 개선되는 이유는?  
3. CDN이 Reverse Proxy와 유사한 이유는?  
4. API Gateway와 Reverse Proxy는 언제 각각 사용하는가?  
5. SSL 종료(SSL Termination)를 Proxy에서 처리하는 이유는?  

---
## 7-1. 면접 질문 & 답변 예시

### Q1. Forward Proxy와 Reverse Proxy의 차이점은?
- **Forward Proxy**: 클라이언트 앞에 위치, 사용자가 직접 서버에 접근하지 않고 프록시를 통해 나감 → 클라이언트 보호/익명성/접근제어
- **Reverse Proxy**: 서버 앞에 위치, 사용자가 프록시를 서버로 인식 → 서버 보호/로드밸런싱/SSL 종료/캐싱

➡️ **한 줄 정리**: Forward Proxy는 "사용자 보호", Reverse Proxy는 "서버 보호"

---

### Q2. 프록시 서버를 사용했을 때 성능이 개선되는 이유는?
- 프록시 서버가 **자주 요청되는 리소스를 캐싱**해두면, 서버까지 가지 않고 빠르게 응답 가능  
- SSL 종료를 프록시에서 처리하면, **백엔드 서버 부하를 줄일 수 있음**  
- 로드밸런싱으로 **트래픽 분산 → 서버 과부하 방지**  

---

### Q3. CDN이 Reverse Proxy와 유사한 이유는?
- CDN도 **사용자와 서버 사이에서 요청을 대신 받아 처리**하는 구조  
- 전 세계 엣지 서버에 **캐싱된 콘텐츠를 제공** → 원본 서버에 부하가 덜 감  
- 클라이언트 입장에서는 CDN이 서버처럼 보이므로 Reverse Proxy와 원리가 유사함  

---

### Q4. API Gateway와 Reverse Proxy는 어떻게 다르며, 언제 각각 사용하는가?
- **공통점**: 클라이언트 요청을 받아 백엔드로 전달하고, 보안·라우팅·트래픽 제어 가능  
- **차이점**:
  - Reverse Proxy: 네트워크/서버 단위 중계, 주로 **로드밸런싱·SSL·캐싱** 목적
  - API Gateway: API 단위 관리, **인증·권한 관리·버전 관리·요율 제한(Rate limiting)** 목적
- **선택 기준**:
  - 단순히 서버 트래픽 분산 → Reverse Proxy
  - API 엔드포인트를 세밀하게 제어하고 외부 개발자에게 공개 → API Gateway

---

### Q5. SSL 종료(SSL Termination)를 Proxy에서 수행하는 이유는?
- SSL 암복호화 작업은 **CPU 리소스를 많이 소모**  
- Proxy(예: Nginx)가 SSL을 대신 처리하면, 백엔드 서버는 **평문 HTTP 트래픽만 처리** → 성능 최적화  
- 인증서도 Proxy 단에서 관리하므로 **운영 편의성** 향상  

---

## 추가 예상 질문
1. 프록시 서버를 도입할 때 단점은?  
   - 단일 장애 지점(SPOF, Single Point of Failure) 위험  
   - 설정/운영 복잡도 증가  
   - 잘못된 캐싱 정책 → 데이터 최신성 문제  

2. Nginx/HAProxy/Envoy 같은 도구는 어떤 기준으로 선택하나?  
   - 서비스 규모, 필요 기능(gRPC 지원 여부, 로드밸런싱 전략), 운영 환경(Kubernetes 여부)에 따라 선택  

3. 실제 프로젝트에서 프록시를 사용해본 경험이 있는가?  
   - (예시 답) "Nginx를 Reverse Proxy로 두어 SSL 종료와 로드밸런싱을 했습니다. 이를 통해 서버 부하를 줄이고, HTTPS 트래픽을 효율적으로 관리했습니다."



---

![](https://velog.velcdn.com/images/ohoh7391/post/7d40a459-5806-49d5-8c5c-b29d2c077f72/image.png)


![](https://velog.velcdn.com/images/ohoh7391/post/45b5ebb4-9738-4ba7-8897-18e74c4a36f8/image.png)

![](https://velog.velcdn.com/images/ohoh7391/post/12841a7b-ffbf-40bc-83db-925880d7d496/image.png)

출처 : 코딩문, Proxy

## 유용한 강의
- https://youtu.be/dThqHi8-MiQ?si=I7zNVNxftA4T3J0L
- https://youtu.be/sSw2sFA4JP0?si=vcpfu_Q4cnB5yf1M
- https://www.youtube.com/watch?v=Rt-KdCpsmdc
엔진엑스 + 프록시, 로드밸런스도 엔진엑스를 쓰는 넷플릭스
- https://www.youtube.com/watch?v=1WfdUtMxTLE

## 포스팅
- https://aws.amazon.com/ko/compare/the-difference-between-proxy-and-vpn/
- https://maker5587.tistory.com/47#google_vignette
- https://engineer-mole.tistory.com/288

---

## 8. 실무에서 자주 쓰이는 Proxy 도구 비교

| 도구      | 주요 특징 | 활용 사례 |
|-----------|-----------|-----------|
| **Nginx** | Reverse Proxy & Web Server 겸용, 가볍고 빠름, 설정이 직관적 | 정적 파일 서빙, 로드밸런싱, SSL 종료 |
| **HAProxy** | 고성능 TCP/HTTP 로드밸런서, 세밀한 트래픽 제어 가능 | 대규모 서비스 트래픽 분산, 금융/통신사 |
| **Envoy** | 최신 Proxy, gRPC 지원, 서비스 메쉬용으로 설계됨 | 마이크로서비스(MSA) 환경, Istio와 함께 사용 |

---

## 9. 실무 포인트 (발표용 강조)
- **Nginx**: 단순 Reverse Proxy로 시작하기 가장 쉬움 → 스타트업/개인 프로젝트에서 필수  
- **HAProxy**: 대규모 트래픽 제어에 강점 → 세밀한 설정과 성능 최적화 가능  
- **Envoy**: 마이크로서비스 아키텍처(MSA) 필수 → 서비스 메쉬(Service Mesh)에서 표준처럼 사용  

---

## 10. 발표/면접에서 어필할 수 있는 포인트
1. "Forward Proxy는 클라이언트 보호, Reverse Proxy는 서버 보호"라고 기억해 두면 설명하기 좋음  
2. API Gateway와 Reverse Proxy는 기능이 겹치지만, **API Gateway는 인증·트래픽 정책·버전 관리**에 초점  
3. CDN, SSL 종료, 로드밸런싱 모두 Reverse Proxy의 확장 기능으로 연결 가능  
4. 실무에서는 **Nginx → HAProxy → Envoy** 순으로 학습하면 성장 곡선이 자연스러움  

