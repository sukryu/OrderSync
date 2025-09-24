# OrderSync Architecture Overview

<div align="center">

**🏗️ 보안 우선 설계 • 📈 확장 가능한 구조 • ⚡ 고성능 실시간 시스템**

![Architecture Status](https://img.shields.io/badge/Status-In%20Development-orange?style=for-the-badge)
![Security Level](https://img.shields.io/badge/Security-PCI%20DSS%20Ready-green?style=for-the-badge)
![Performance Target](https://img.shields.io/badge/P99-<500ms-blue?style=for-the-badge)

</div>

-----

## 🎯 **아키텍처 설계 철학**

OrderSync는 **소상공인의 디지털 혁신**을 위한 실시간 주문 관리 시스템으로, 다음 원칙을 기반으로 설계되었습니다:

### 🔐 **보안 우선 (Security First)**

- **모든 민감 데이터 암호화**: PII, 결제정보, 사업자정보 컬럼 레벨 암호화
- **완전한 감사 추적**: 모든 데이터 변경사항 불변 로그로 기록
- **제로 트러스트 모델**: 모든 API 요청에 대한 인증, 인가, 검증

### ⚡ **성능 우선 (Performance First)**

- **P99 응답시간 목표**: 인증 200ms, 기타 API 500ms
- **동시 처리량**: 초당 10,000건 주문 처리 가능
- **실시간 알림**: 100ms 이내 WebSocket 알림 전달

### 🚀 **점진적 확장 (Progressive Scaling)**

- **모듈러 모놀리스**: MVP 단계에서 단일 바이너리 배포
- **마이크로서비스 준비**: 도메인별 명확한 경계 설정
- **무중단 확장**: 트래픽 증가에 따른 선택적 서비스 분리

-----

## 🏗️ **시스템 아키텍처 개요**

### **전체 구조**

```
┌─────────────────────────────────────────────────────────────┐
│                    Load Balancer                            │
│              (Nginx + Rate Limiting)                       │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                   API Gateway                               │
│  • JWT 인증/인가 (P99 < 200ms)                              │
│  • 요청 검증 & 암호화                                        │
│  • 감사 로그 & 메트릭                                        │
│  • Rate Limiting & DDoS 보호                               │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                Business Logic Layer                        │
│                  (Go Services)                             │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │    Auth     │  │   Order     │  │  Payment    │         │
│  │  Service    │  │  Service    │  │  Service    │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ Restaurant  │  │ Analytics   │  │Notification │         │
│  │  Service    │  │  Service    │  │  Service    │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                   Data Layer                                │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ PostgreSQL  │  │    Redis    │  │   Encryption│         │
│  │ (트랜잭션)   │  │  (캐시+세션) │  │   Service   │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### **핵심 설계 결정**

|구분        |선택        |근거                       |
|----------|----------|-------------------------|
|**언어**    |Go        |높은 동시성, 빠른 컴파일, 메모리 효율성  |
|**아키텍처**  |모듈러 모놀리스  |MVP 단계 적합, 점진적 마이크로서비스 전환|
|**데이터베이스**|PostgreSQL|ACID 보장, 암호화 지원, 복잡한 쿼리  |
|**캐시**    |Redis     |세션 관리, 실시간 알림, 고성능 캐싱    |
|**보안**    |컬럼 레벨 암호화 |PCI DSS 준수, 개인정보보호법 대응   |

-----

## 📋 **도메인 구조**

OrderSync는 **도메인 주도 설계(DDD)**를 기반으로 다음과 같이 구성됩니다:

### **핵심 비즈니스 도메인**

```
🏪 Restaurant Domain
├── 매장 관리 (사업자 정보 암호화)
├── 메뉴 관리 (동적 가격, 재고 관리)
└── 운영 시간 및 설정

🛒 Order Domain  
├── 주문 생성 및 상태 관리
├── 실시간 상태 추적
└── 주문 이력 및 분석

💳 Payment Domain
├── 토스페이 연동 결제
├── 환불 및 정산 관리  
└── PCI DSS 준수 보안

👤 User Domain
├── 사용자 인증 (JWT + 2FA)
├── 권한 관리 (RBAC)
└── 세션 관리
```

### **보안 지원 도메인**

```
🔐 Encryption Domain
├── AES-256-GCM 암호화
├── 키 관리 (KMS 연동)
└── 암호화/복호화 서비스

📊 Audit Domain
├── 완전한 감사 추적
├── 불변 로그 저장
└── 보안 이벤트 모니터링

📈 Metrics Domain
├── 실시간 성능 메트릭
├── 보안 위협 탐지
└── 비즈니스 인사이트
```

**📋 상세 도메인 설계**: [domain.md](./domain.md)

-----

## 🗄️ **데이터베이스 설계**

### **보안 중심 스키마**

```sql
-- 암호화 컬럼 예시
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email_encrypted TEXT NOT NULL,      -- 개인정보 암호화
    email_key_id VARCHAR(100),          -- 암호화 키 ID
    password_hash_encrypted TEXT,       -- 인증정보 암호화
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 감사 로그 (모든 변경사항 추적)
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY,
    trace_id VARCHAR(32),               -- 분산 추적
    action VARCHAR(50),                 -- CREATE_ORDER, LOGIN 등
    user_id UUID,
    old_value_encrypted TEXT,           -- 변경 전 값 (암호화)
    new_value_encrypted TEXT,           -- 변경 후 값 (암호화)
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### **성능 최적화**

- **복합 인덱스**: 주요 조회 패턴에 최적화된 인덱스 설계
- **파티셔닝**: 시계열 데이터(감사로그, 메트릭)의 월별 파티셔닝
- **Row Level Security**: PostgreSQL RLS로 데이터 접근 제어

**🗄️ 완전한 스키마**: [schema.md](./schema.md)

-----

## 🔌 **API 설계**

### **보안 우선 API**

모든 API는 **보안과 성능**을 최우선으로 설계됩니다:

```go
// 인증 & 인가 (P99 < 200ms)
POST /api/v1/auth/login          // JWT 로그인
POST /api/v1/auth/refresh        // 토큰 갱신
GET  /api/v1/auth/me            // 사용자 정보

// 주문 관리 (P99 < 500ms)  
POST /api/v1/orders             // 주문 생성 (익명 지원)
GET  /api/v1/orders/{id}        // 주문 조회
WebSocket /ws/orders/{id}/status // 실시간 상태

// 매장 관리
GET  /api/v1/restaurants/me/orders    // 매장 주문 목록
POST /api/v1/restaurants/me/orders/{id}/accept // 주문 승인
```

### **보안 기능**

- **JWT + RBAC**: 역할 기반 접근 제어
- **Rate Limiting**: IP/사용자별 요청 제한
- **입력 검증**: SQL Injection, XSS 방지
- **감사 로그**: 모든 API 호출 추적

**🔌 상세 API 명세**: [api.md](./api.md)

-----

## ⚡ **기술 스택**

### **백엔드 핵심**

|기술            |버전   |용도        |선택 근거         |
|--------------|-----|----------|--------------|
|**Go**        |1.21+|메인 언어     |높은 동시성, 빠른 성능 |
|**Gin**       |v1.9+|HTTP 프레임워크|가볍고 빠른 라우팅    |
|**PostgreSQL**|15+  |메인 데이터베이스 |ACID, 암호화 지원  |
|**Redis**     |7+   |캐시 + 세션   |높은 성능, Pub/Sub|

### **보안 & 모니터링**

|기술               |용도     |특징             |
|-----------------|-------|---------------|
|**JWT**          |인증 토큰  |Stateless, 확장성 |
|**AES-256-GCM**  |데이터 암호화|NIST 승인, 무결성 보장|
|**Prometheus**   |메트릭 수집 |시계열 데이터베이스     |
|**OpenTelemetry**|분산 추적  |성능 모니터링        |

### **외부 연동**

```
📱 토스페이먼츠  → 결제 처리
🔐 AWS KMS      → 암호화 키 관리  
📧 센드그리드    → 이메일 발송
📱 알리고       → SMS 발송
```

**🛠️ 상세 기술 선택 근거**: [tech.md](./tech.md)

-----

## 📊 **성능 및 확장성**

### **성능 목표**

|메트릭           |목표        |현재 상태    |
|--------------|----------|---------|
|**인증 API P99**|< 200ms   |🚧 개발 중   |
|**주문 API P99**|< 500ms   |🚧 개발 중   |
|**실시간 알림**    |< 100ms   |🚧 개발 중   |
|**동시 사용자**    |10,000명   |🚧 테스트 예정 |
|**처리량**       |10,000 TPS|🚧 벤치마크 예정|

### **확장 전략**

```
Phase 1: 모듈러 모놀리스 (현재)
├── 단일 Go 바이너리
├── PostgreSQL + Redis
└── 수직/수평 스케일링

Phase 2: 선택적 분리 (트래픽 증가 시)  
├── 주문 서비스 분리
├── 알림 서비스 분리
└── gRPC 내부 통신

Phase 3: 완전한 마이크로서비스
├── 도메인별 독립 배포
├── Event-Driven Architecture  
└── Kubernetes 오케스트레이션
```

-----

## 🛡️ **보안 아키텍처**

### **다층 보안 모델**

```
🔒 애플리케이션 계층
├── JWT 인증 + RBAC 인가
├── 입력 검증 & 출력 인코딩
└── API Rate Limiting

🔒 데이터 계층  
├── 컬럼 레벨 암호화 (AES-256-GCM)
├── Row Level Security (PostgreSQL)
└── 감사 로그 & 불변 기록

🔒 네트워크 계층
├── TLS 1.3 암호화 통신
├── VPC & Security Group  
└── DDoS 보호 (Cloudflare)

🔒 인프라 계층
├── 암호화 키 관리 (AWS KMS)
├── 컨테이너 보안 스캔
└── 취약점 자동 패치
```

### **규정 준수**

- **PCI DSS**: 결제 데이터 보안 표준
- **개인정보보호법**: 개인정보 암호화 및 접근 제어
- **GDPR**: 데이터 삭제 권리 및 처리 동의

-----

## 📈 **모니터링 & 관측성**

### **실시간 모니터링**

```
📊 비즈니스 메트릭
├── 주문 수, 매출, 전환율
├── 매장별 성과 분석
└── 실시간 대시보드

⚡ 성능 메트릭  
├── API 응답시간 (P50, P95, P99)
├── 데이터베이스 쿼리 성능
└── 캐시 히트율

🛡️ 보안 메트릭
├── 실패한 로그인 시도
├── 비정상 접근 패턴
└── 위협 탐지 알림
```

### **알림 시스템**

- **즉시 알림**: 시스템 장애, 보안 위협
- **일일 리포트**: 성능 요약, 비즈니스 지표
- **주간 분석**: 트렌드 분석, 최적화 제안

-----

## 🚀 **개발 및 배포**

### **개발 프로세스**

```
개발환경 (Local)
├── Go + PostgreSQL + Redis
├── Docker Compose  
└── 핫 리로드 개발

스테이징 (Staging)
├── 실제 환경과 동일한 구성
├── 성능 테스트 & 보안 검증
└── 자동화된 E2E 테스트

프로덕션 (Production)  
├── 무중단 배포 (Blue-Green)
├── 자동 롤백 시스템
└── 실시간 모니터링
```

### **CI/CD 파이프라인**

1. **코드 품질**: 린트, 테스트, 보안 스캔
1. **빌드**: Go 바이너리 최적화 컴파일
1. **배포**: 점진적 배포 (Canary)
1. **검증**: 헬스 체크, 성능 확인

-----

## 📚 **관련 문서**

### **아키텍처 상세**

- 🛠️ **[기술 스택 상세](./tech.md)**: 기술 선택 근거, 라이브러리 정보
- 📋 **[도메인 설계](./domain.md)**: DDD 모델링, 비즈니스 로직
- 🗄️ **[데이터베이스 스키마](./schema.md)**: 테이블 구조, 인덱스, 보안
- 🔌 **[API 명세](./api.md)**: 엔드포인트, 보안, 성능 최적화

### **개발 가이드**

- 🚀 **[개발 환경 설정](../docs/development.md)**: 로컬 개발 환경
- 🧪 **[테스트 전략](../docs/testing.md)**: 단위/통합/E2E 테스트
- 🔒 **[보안 가이드라인](../docs/security.md)**: 보안 코딩 규칙
- 📊 **[모니터링 가이드](../docs/monitoring.md)**: 메트릭, 로그, 알림

-----

## 🎯 **다음 단계**

### **개발 우선순위**

1. **🔐 보안 기반 구축**
   - JWT 인증 시스템
   -  데이터 암호화 서비스
   -   감사 로그 시스템
   -   
2. **⚡ 핵심 기능 구현**
   - 주문 생성 및 관리
   -  실시간 알림 (WebSocket)
   -  토스페이 결제 연동
   -  
3. **📊 모니터링 및 최적화**
   - 성능 메트릭 수집
   - 자동화된 테스트
   - 부하 테스트 및 튜닝

4. **🚀 MVP 출시 준비**
   - 보안 감사
   -  성능 벤치마크
   -   문서화 완료

-----

<div align="center">

**OrderSync Architecture**  
*Built with Security, Performance, and Scalability in Mind*

![Go](https://img.shields.io/badge/Go-1.21+-00ADD8?style=flat&logo=go)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-336791?style=flat&logo=postgresql)
![Redis](https://img.shields.io/badge/Redis-7+-DC382D?style=flat&logo=redis)
![Security](https://img.shields.io/badge/Security-AES--256--GCM-green?style=flat&logo=security)

**Ver. 1.0.0** | **Last Updated**: 2025-09-24

</div>
