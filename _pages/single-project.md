---
title: "진행중인 개인 프로젝트"
permalink: /project/
layout: single
---

<br><br>

# 1. MSA 주문 클레임 시스템

Spring Boot와 Spring Cloud로 구축된 마이크로서비스 기반 주문 및 클레임 관리 시스템입니다.

## 사용 기술

### 핵심 기술
- **Java 21**: 최신 LTS 버전의 Java
- **Spring Boot 3.3.3**: 프로덕션 레디 애플리케이션 구축을 위한 프레임워크
- **Spring Cloud 2023.0.3**: 클라우드 네이티브 애플리케이션 구축을 위한 도구

### 마이크로서비스 아키텍처
- **Spring Cloud Netflix Eureka**: 서비스 디스커버리 및 등록
- **Spring Cloud Gateway**: 라우팅 및 필터링을 위한 API 게이트웨이
- **Spring Cloud Config**: 중앙 집중식 구성 관리
- **Spring Cloud Bus**: 구성 변경을 위한 분산 메시징

### 메시징 및 이벤트 기반 아키텍처
- **Apache Kafka**: 비동기 통신을 위한 이벤트 스트리밍 플랫폼
- **Outbox 패턴**: 트랜잭션 아웃박스를 통한 안정적인 메시지 전달

### 회복성 및 장애 허용
- **Resilience4j**: 서킷 브레이커, 속도 제한기 및 재시도 메커니즘
- **Spring Cloud Circuit Breaker**: 폴백 메커니즘을 통한 장애 허용

### 영속성
- **Spring Data JPA**: JPA를 사용한 데이터 액세스 계층
- **PostgreSQL**: 영구 저장소를 위한 관계형 데이터베이스

### 관측성 및 모니터링
- **Spring Boot Actuator**: 모니터링을 위한 프로덕션 레디 기능
- **Micrometer**: 애플리케이션 메트릭 파사드
- **Zipkin & Brave**: 요청 흐름을 위한 분산 추적 (traceId를 서비스 간에 전파하여 전체 요청 흐름을 추적)
- **Prometheus**: 메트릭 수집 및 알림 (구성됨)

### 보안 (계획됨)
- **Keycloak**: 신원 및 접근 관리
- **OAuth2/OpenID Connect**: 인증 및 권한 부여 프로토콜

### 개발 도구
- **Gradle**: 빌드 자동화 도구
- **Lombok**: 상용구 코드 감소

## 모듈

### 핵심 서비스
- **order-service**: 주문 생성, 처리 및 상태 업데이트 관리
- **payment-service**: 주문에 대한 결제 처리
- **inventory-service**: 제품 재고 및 재고 확인 관리
- **claim-service**: 주문 클레임 및 반품 처리 (개발 중)

### 인프라 서비스
- **api-gateway**: 라우팅 및 로드 밸런싱을 통한 모든 클라이언트 요청의 진입점
- **discovery**: 서비스 디스커버리 및 등록을 위한 Eureka 서버
- **config-server**: 중앙 집중식 구성 관리
- **outbox-relay**: 안정적인 이벤트 발행을 위한 아웃박스 패턴 구현

### 공통 컴포넌트
- **common-lib**: 서비스 전반에 걸쳐 사용되는 공유 모델, 이벤트 및 유틸리티

## 시스템 아키텍처

<a href="{{site.url}}/assets/images/flow_chart.svg" target="_blank"><img src="{{site.url}}/assets/images/flow_chart.svg" alt="flow_chart" title="flow_chart"></a>


시스템은 다음과 같은 흐름으로 이벤트 기반 마이크로서비스 아키텍처를 따릅니다:
1. 클라이언트가 Keycloak으로 인증하고 토큰을 받습니다
2. 요청은 API 게이트웨이를 통해 라우팅됩니다
3. 주문은 order-service에서 생성됩니다
4. 이벤트는 아웃박스 패턴을 통해 Kafka에 발행됩니다
5. 인벤토리 및 결제 서비스가 이벤트를 처리합니다
6. 결과는 order-service로 다시 전달됩니다
7. 주문 상태가 그에 따라 업데이트됩니다

모든 서비스 간 통신에서 traceId가 전파되어 전체 요청 흐름을 추적할 수 있습니다. 이를 통해 복잡한 마이크로서비스 환경에서도 요청의 전체 경로를 쉽게 파악하고 문제를 진단할 수 있습니다.

<a href="{{site.url}}/assets/images/zipkin.png" target="_blank"><img src="{{site.url}}/assets/images/zipkin.png" alt="zipkin" title="zipkin"></a>

## API 엔드포인트

### API 게이트웨이 (포트: 8080)
- 모든 클라이언트 요청의 주요 진입점
- 적절한 서비스로 요청 라우팅
- 회복성을 위한 서킷 브레이커 패턴 구현

### 주문 서비스 (포트: 9001)
- **POST /orders**: 새 주문 생성
  - 요청 본문: `{ "userId": "string", "totalAmount": number }`
  - 응답: 주문 ID (UUID)
- **GET /orders/{id}**: ID로 주문 세부 정보 가져오기
  - 응답: 주문 세부 정보

### 클레임 서비스
- **POST /claims**: 새 클레임 생성
- **GET /claims/{id}**: ID로 클레임 세부 정보 가져오기

## 시작하기

### 사전 요구 사항
- Java 21
- Docker 및 Docker Compose
- Gradle

### 애플리케이션 실행
1. 인프라 서비스 시작:
```bash
docker-compose -f docker/docker-compose.yml up -d
```

2. 다음 순서로 서비스 시작:
  - discovery
  - config-server
  - order-service, payment-service, inventory-service
  - api-gateway

## 개발 로드맵

### 계획된 기능
- 인증 및 권한 부여를 위한 Keycloak 통합
- 보안 API 액세스를 위한 OAuth2/OpenID Connect 구현
- Prometheus 및 Grafana를 통한 모니터링 및 알림 강화
- SpringDoc OpenAPI를 통한 포괄적인 API 문서화
- claim-service의 완전한 구현


---

<br><br>

# 2. ASAP

미션 계획(1Day, 3Day, 5Day, 7Day)을 세우고 달성하기 위해, 동기부여를 만들어주는 사이트 제작

## 사용 기술

### 핵심 기술
- **Spring Boot 2.1.7**: 프로덕션 레디 애플리케이션 구축을 위한 프레임워크
- **JAVA 11**: 안정적인 LTS 버전의 Java
- **JPA**: 객체 관계 매핑을 위한 자바 퍼시스턴스 API

### 인증 및 보안
- **JWT 0.9.1**: JSON 웹 토큰을 통한 인증
- **OAuth2**: 소셜 로그인 지원 (Google, Naver, Kakao)

### 프론트엔드 및 API
- **Mustache**: 서버 사이드 템플릿 엔진
- **jQuery, JavaScript**: 클라이언트 사이드 스크립팅
- **Swagger 2.9.2**: API 문서화 및 테스트

### 데이터베이스
- **H2 Database**: 개발 및 테스트 환경용 인메모리 데이터베이스
- **MySQL**: 프로덕션 환경용 관계형 데이터베이스

### 인프라 및 배포
- **AWS**: 클라우드 인프라 (EC2, RDS, IAM, S3, CodeDeploy)
- **Travis CI**: 지속적 통합 (구버전)
- **Jenkins**: 지속적 통합 및 배포 (현재버전)
- **NGINX 1.20**: 웹 서버 및 리버스 프록시

## Git 저장소
- [ASAP v1](https://github.com/hyot88/ASAP)
- [ASAP v2 - API 백엔드](https://github.com/hyot88/ASAP-api)

## 시스템 아키텍처

### 구버전 (Travis CI 사용)
<a href="{{site.url}}/assets/images/old.png" target="_blank"><img src="{{site.url}}/assets/images/old.png" alt="old" title="old"></a>

### 현재버전 (Jenkins 사용)
<a href="{{site.url}}/assets/images/new.png" target="_blank"><img src="{{site.url}}/assets/images/new.png" alt="new" title="new"></a>
