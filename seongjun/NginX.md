
![](https://velog.velcdn.com/images/ohoh7391/post/509183f4-5e72-49e5-aae7-07afbcf29d6a/image.png)

# 🌐 Nginx 정리 가이드

## 1. 개요
- **Nginx(Engine-X)**는 고성능 **웹 서버(HTTP Server)**이자 **리버스 프록시 서버(Reverse Proxy Server)**, **로드 밸런서(Load Balancer)** 역할을 수행하는 오픈소스 소프트웨어다.  
- 2004년 Igor Sysoev에 의해 공개되었으며, 높은 동시 연결 처리 성능으로 아파치(Apache)와 함께 가장 많이 사용되는 웹 서버다.  
- 특히 **비동기 이벤트 기반 아키텍처**를 통해 수만 개의 동시 연결을 효율적으로 처리할 수 있다.

---

## 2. 아키텍처
### 기본 구조
1. **Master Process**  
   - 설정 파일 로드, Worker 프로세스 관리
2. **Worker Process**  
   - 클라이언트 요청을 실제로 처리
   - 이벤트 기반으로 다수의 연결을 동시에 처리
3. **모듈 구조**  
   - HTTP 모듈, Stream 모듈(TCP/UDP), Mail 모듈 등 플러그인처럼 확장 가능

### 처리 흐름
```
클라이언트 요청 → Nginx(Reverse Proxy, Load Balancer) 
             → 백엔드 서버(WAS, Spring Boot, Node.js 등)
             → 응답 반환
```

---

## 3. 특징
- **가벼움** : 메모리 효율적, 작은 리소스로 많은 연결 처리  
- **비동기 이벤트 기반 처리** : 동시성 처리 강점  
- **리버스 프록시 지원** : 백엔드 서버 앞단에서 보안·캐싱·로드 밸런싱 담당  
- **로드 밸런싱** : Round Robin, Least Connections, IP Hash 등 다양한 방식  
- **정적 파일 서비스** : 이미지, CSS, JS 같은 정적 리소스를 빠르게 전달  
- **SSL/TLS 종단 처리** : HTTPS 트래픽 종료(Offloading)  
- **유연한 설정 파일** : 블록 기반(`server`, `location`) 설정 구조  

---

## 4. 활용 사례
- **정적 리소스 캐싱** : S3/CDN 대체 혹은 보완  
- **리버스 프록시** : WAS(Spring, Node.js 등) 앞단에서 보안·트래픽 관리  
- **로드 밸런싱** : 다수의 애플리케이션 서버 분산 처리  

---

## 5. 실무 운영 포인트
- `worker_processes auto;` : CPU 코어 개수에 맞게 자동 할당  
- `worker_connections` : 동시 연결 수 조절 (ex. 1024 → 수만 개 커넥션 가능)  
- `upstream` 블록 : 로드 밸런싱 서버 그룹 정의  
- `location` 블록 : 요청 라우팅 세부 제어  
- `proxy_pass` : 백엔드 서버로 요청 전달  
- 로그 관리 (`access.log`, `error.log`) 및 모니터링 필수  

---

## 6. 심화 학습 주제 (스터디 발표용)
- **Event-driven 모델 vs Multi-threading 모델** 비교  
- **Apache vs Nginx** 성능/구조 차이  
- **Nginx + Redis 캐싱 아키텍처**  
- **Kubernetes Ingress Controller로서의 Nginx**  
- **Nginx와 API Gateway(Envoy, Kong, Spring Cloud Gateway) 비교**  
- **Failover/High Availability 구성 (Keepalived + Nginx)**  

---

## 7. 면접 질문 예시
1. Nginx와 Apache의 가장 큰 차이점은 무엇인가요?  
2. Nginx에서 리버스 프록시 역할은 어떤 이점을 주나요?  
3. 로드 밸런싱 전략(Round Robin, Least Connections, IP Hash)의 차이점은?  
4. 정적 리소스를 Nginx로 서빙하면 WAS보다 좋은 이유는?  
5. `location` 블록에서 prefix match와 regex match 차이는?  
6. Nginx로 무중단 배포를 구현하려면 어떤 방식이 있나요?  
7. Kubernetes 환경에서 Ingress Controller로 Nginx를 사용하는 이유는?  

---

## 8. 추가 특징 (심화)
- **커넥션 처리 모델**  
  - 이벤트 루프 기반 (epoll, kqueue 활용)  
  - Blocking I/O vs Non-blocking I/O vs Event-driven 차이 설명 가능  
- **Keep-Alive**  
  - 클라이언트와 지속적인 연결 유지 → 성능 향상, 단 연결 자원 관리 필요  
- **Gzip / Brotli 압축**  
  - 응답 사이즈 줄여 네트워크 전송 최적화  
- **Rate Limiting**  
  - `limit_req_zone`을 이용한 요청 속도 제한 → DDoS 완화  
- **Caching**  
  - Proxy 캐싱, FastCGI 캐싱 지원 → 백엔드 부하 줄임  
- **Rewrite/Redirect 규칙**  
  - `rewrite`, `return` 지시어로 URL 구조 제어 가능  

---

## 9. 운영/실무 포인트 (추가)
- **Zero-downtime Reload**  
  - `nginx -s reload` : 설정 반영 시 무중단 적용 가능  
- **Health Check**  
  - `proxy_next_upstream`, `fail_timeout`으로 비정상 서버 자동 제외  
- **보안**  
  - TLS 설정 (`ssl_protocols`, `ssl_ciphers`)  
  - 보안 헤더 추가 (`add_header X-Frame-Options`, `Content-Security-Policy`)  
- **Log 분석**  
  - `access.log`, `error.log`를 ELK(Elasticsearch + Logstash + Kibana)와 연동  
- **실패 대비**  
  - `proxy_cache_use_stale` 옵션으로 백엔드 장애 시 캐싱된 응답 제공  

---

## 10. 아키텍처 연계
- **Nginx + Spring Boot** : WAS 앞단에서 리버스 프록시 + 로드 밸런싱  
- **Nginx + Docker** : 다중 컨테이너 환경에서 서비스 라우팅  
- **Nginx + Kubernetes** : Ingress Controller 역할 (클러스터 외부 요청을 내부 서비스로 라우팅)  
- **Nginx + CDN** : 글로벌 트래픽 분산 및 정적 리소스 최적화  

---

## 11. 면접에서 자주 묻는 심화 질문
1. Nginx는 멀티 프로세스 기반인데, 동시성을 어떻게 확보하나요?  
2. 이벤트 루프 방식(epoll)이 성능에 어떤 이점을 주나요?  
3. Nginx에서 캐싱을 설정하면 어떤 장점과 단점이 있나요?  
4. Nginx와 HAProxy의 차이는 무엇인가요? (둘 다 로드밸런서 역할 가능)  
5. Kubernetes에서 Ingress Controller로 Nginx를 왜 많이 쓸까요?  
6. `worker_connections`와 `worker_processes` 설정이 성능에 미치는 영향은?  
7. HTTPS를 Nginx에서 종료(SSL Termination)하면 어떤 이점이 있나요?  
8. Rate Limiting(속도 제한)을 어떻게 구현할 수 있나요?  
9. Blue-Green / Canary 배포를 Nginx로 어떻게 지원하나요?  
10. 대규모 트래픽에서 Nginx와 CDN은 어떻게 협력하나요?  
---

## 12. 비교 정리: Nginx vs Apache vs HAProxy vs Envoy

| 구분 | **Nginx** | **Apache HTTP Server** | **HAProxy** | **Envoy** |
|------|-----------|-------------------------|--------------|------------|
| **주요 역할** | 웹 서버, 리버스 프록시, 로드 밸런서 | 전통적인 웹 서버 (정적/동적 콘텐츠 처리) | 고성능 TCP/HTTP 로드 밸런서 | 클라우드 네이티브 프록시, 서비스 메시 |
| **아키텍처** | 이벤트 기반(Non-blocking, epoll/kqueue) | 프로세스/스레드 기반 (MPM) | 이벤트 기반, 고성능 I/O | 이벤트 기반, gRPC/HTTP2 지원 |
| **강점** | 가벼움, 정적 리소스 처리, 리버스 프록시 | 다양한 모듈, .htaccess로 세밀한 설정 | 로드밸런싱 특화, 세션 지속성, Health Check | 서비스 메시용, Observability, gRPC/HTTP3 |
| **로드 밸런싱** | Round Robin, Least Connections, IP Hash | 기본 지원(성능 한계) | 전문적, 다양한 알고리즘 지원 | 고급 기능 + 서비스 디스커버리 연계 |
| **캐싱** | Proxy 캐싱, FastCGI 캐싱 | 모듈 기반 캐싱 (mod_cache 등) | 기본 캐싱 없음 (단순 프록시) | 캐싱보단 서비스 메시 중심 |
| **확장성** | Kubernetes Ingress Controller로 많이 사용 | 전통적인 단일 서버 중심 | L4/L7 로드밸런서, 고성능 환경 최적 | Istio 등 서비스 메시와 통합 |
| **주요 사용처** | 정적/동적 웹 서비스, 리버스 프록시, CDN, Ingress | 전통적 웹 호스팅, PHP/Perl 기반 서비스 | 대규모 트래픽 분산, 금융/통신 | 마이크로서비스 아키텍처, 서비스 메시 |

---

## 13. 면접 꼬리 질문 (비교 관련)
1. Apache와 Nginx의 동시 연결 처리 방식 차이는?  
2. HAProxy가 Nginx보다 로드밸런싱에서 유리한 점은?  
3. Envoy가 Kubernetes/마이크로서비스 환경에서 각광받는 이유는?  
4. CDN 환경에서는 Nginx와 HAProxy 중 어떤 선택이 더 적합할까요?  
5. 서비스 메시(Service Mesh)와 API Gateway 차이를 설명해보세요.  


--- 
## 참고하면 좋은 포스팅
- https://nginx.org/en/docs/
- https://soonmin.tistory.com/88
- https://jammdev.tistory.com/217
