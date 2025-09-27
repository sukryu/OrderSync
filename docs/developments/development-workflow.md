# 🔄 OrderSync 개발 워크플로우 가이드

> **Version**: 1.0.0  
> **Last Updated**: 2025-09-27  
> **Status**: Approved

-----

## 📋 **개요**

이 문서는 OrderSync 프로젝트의 Git 브랜치 전략, PR 프로세스, 코드 리뷰 규칙을 정의합니다.

**목적**:

- ✅ 일관된 개발 프로세스 확립
- ✅ 코드 품질 유지 및 버그 최소화
- ✅ 효율적인 협업 환경 구축
- ✅ 안전한 배포 프로세스 보장

**대상 독자**:

- 모든 개발자 (백엔드, 프론트엔드, DevOps)
- 프로젝트 매니저
- QA 엔지니어

-----

## 🌳 **Git 브랜치 전략 (Git Flow 기반)**

### **브랜치 구조**

```
main (프로덕션)
  │
  ├── release/* (릴리스 준비)
  │     │
  │     └── develop (개발 통합)
  │           │
  │           ├── feature/* (기능 개발)
  │           ├── bugfix/* (버그 수정)
  │           └── hotfix/* (긴급 수정)
```

### **브랜치 설명**

#### **1. main (프로덕션)**

- **목적**: 프로덕션 환경에 배포되는 안정적인 코드
- **보호 수준**: 최고 (직접 푸시 금지)
- **병합 조건**: Release 브랜치에서만 병합 가능
- **태그**: 모든 배포 시 버전 태그 생성 (v1.0.0)

```bash
# main 브랜치 보호 규칙
- ✅ PR 리뷰 필수 (최소 2명 승인)
- ✅ CI/CD 테스트 통과 필수
- ✅ 직접 푸시 금지
- ✅ Force push 금지
- ✅ 브랜치 삭제 금지
```

#### **2. develop (개발 통합)**

- **목적**: 개발 중인 기능들을 통합하는 브랜치
- **보호 수준**: 높음 (PR 필수)
- **병합 조건**: Feature/Bugfix 브랜치에서 병합
- **배포**: 스테이징 환경에 자동 배포

```bash
# develop 브랜치 보호 규칙
- ✅ PR 리뷰 필수 (최소 1명 승인)
- ✅ CI 테스트 통과 필수
- ✅ 직접 푸시 금지
- ✅ Force push 금지
```

#### **3. feature/* (기능 개발)**

- **목적**: 새로운 기능 개발
- **생성 위치**: develop 브랜치에서 분기
- **병합 대상**: develop 브랜치
- **네이밍**: `feature/ISSUE_NUMBER-brief-description`

```bash
# 예시
feature/123-add-user-authentication
feature/456-implement-payment-webhook
feature/789-create-order-dashboard
```

#### **4. bugfix/* (버그 수정)**

- **목적**: develop 브랜치의 버그 수정
- **생성 위치**: develop 브랜치에서 분기
- **병합 대상**: develop 브랜치
- **네이밍**: `bugfix/ISSUE_NUMBER-brief-description`

```bash
# 예시
bugfix/234-fix-login-timeout
bugfix/567-resolve-order-calculation-error
```

#### **5. release/* (릴리스 준비)**

- **목적**: 프로덕션 배포 준비
- **생성 위치**: develop 브랜치에서 분기
- **병합 대상**: main 및 develop 브랜치
- **네이밍**: `release/v1.2.0`

```bash
# 예시
release/v1.0.0
release/v1.1.0
release/v2.0.0-beta
```

#### **6. hotfix/* (긴급 수정)**

- **목적**: 프로덕션 긴급 버그 수정
- **생성 위치**: main 브랜치에서 분기
- **병합 대상**: main 및 develop 브랜치
- **네이밍**: `hotfix/ISSUE_NUMBER-critical-fix`

```bash
# 예시
hotfix/999-fix-payment-critical-bug
hotfix/888-resolve-database-connection-issue
```

-----

## 🔀 **브랜치 워크플로우**

### **1. 새로운 기능 개발 플로우**

```bash
# 1. develop 브랜치 최신화
git checkout develop
git pull origin develop

# 2. 기능 브랜치 생성
git checkout -b feature/123-add-oauth-login

# 3. 개발 작업 수행
# ... 코드 작성 ...

# 4. 변경사항 커밋
git add .
git commit -m "feat: Add OAuth2.0 Google login"

# 5. 원격 브랜치에 푸시
git push origin feature/123-add-oauth-login

# 6. GitHub에서 PR 생성 (develop ← feature/123-add-oauth-login)

# 7. 코드 리뷰 및 수정 반영

# 8. PR 승인 후 병합 (Squash and Merge)

# 9. 로컬 브랜치 정리
git checkout develop
git pull origin develop
git branch -d feature/123-add-oauth-login
```

### **2. 버그 수정 플로우**

```bash
# 1. develop 최신화
git checkout develop
git pull origin develop

# 2. 버그 수정 브랜치 생성
git checkout -b bugfix/456-fix-order-validation

# 3. 버그 수정
# ... 코드 수정 ...

# 4. 테스트 작성 및 실행
go test ./...

# 5. 커밋 및 푸시
git add .
git commit -m "fix: Resolve order item validation bug"
git push origin bugfix/456-fix-order-validation

# 6. PR 생성 및 리뷰

# 7. 병합 후 정리
```

### **3. 릴리스 플로우**

```bash
# 1. develop에서 릴리스 브랜치 생성
git checkout develop
git pull origin develop
git checkout -b release/v1.2.0

# 2. 버전 정보 업데이트
# - CHANGELOG.md 작성
# - version.go 버전 변경
# - package.json 버전 업데이트

# 3. 릴리스 테스트
make test
make test-integration
make test-e2e

# 4. 문서 업데이트
# - API 문서 재생성
# - README 업데이트

# 5. 릴리스 브랜치 푸시
git add .
git commit -m "chore: Prepare release v1.2.0"
git push origin release/v1.2.0

# 6. main으로 PR 생성 (main ← release/v1.2.0)

# 7. PR 승인 후 병합

# 8. main에 태그 생성
git checkout main
git pull origin main
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin v1.2.0

# 9. develop으로 역병합 (변경사항 반영)
git checkout develop
git merge main
git push origin develop

# 10. 릴리스 브랜치 삭제
git branch -d release/v1.2.0
git push origin --delete release/v1.2.0
```

### **4. 핫픽스 플로우 (긴급 수정)**

```bash
# 1. main에서 핫픽스 브랜치 생성
git checkout main
git pull origin main
git checkout -b hotfix/999-fix-payment-crash

# 2. 긴급 수정
# ... 최소한의 변경 ...

# 3. 테스트
go test ./internal/payment/...

# 4. 커밋 및 푸시
git add .
git commit -m "fix: Resolve critical payment crash (CVE-2025-1234)"
git push origin hotfix/999-fix-payment-crash

# 5. main으로 즉시 PR 생성 (긴급)

# 6. 긴급 승인 후 병합

# 7. 핫픽스 태그 생성
git checkout main
git pull origin main
git tag -a v1.1.1 -m "Hotfix v1.1.1 - Payment crash fix"
git push origin v1.1.1

# 8. develop으로 역병합
git checkout develop
git merge main
git push origin develop

# 9. 핫픽스 브랜치 삭제
git branch -d hotfix/999-fix-payment-crash
git push origin --delete hotfix/999-fix-payment-crash
```

-----

## 📝 **커밋 메시지 컨벤션 (Conventional Commits)**

### **커밋 메시지 형식**

```
<type>(<scope>): <subject>

<body>

<footer>
```

### **Type (필수)**

|Type        |설명               |예시                                           |
|------------|-----------------|---------------------------------------------|
|**feat**    |새로운 기능 추가        |`feat(auth): Add Google OAuth login`         |
|**fix**     |버그 수정            |`fix(order): Resolve calculation error`      |
|**docs**    |문서 수정            |`docs(api): Update authentication guide`     |
|**style**   |코드 포맷팅 (동작 변경 없음)|`style: Format code with gofmt`              |
|**refactor**|코드 리팩토링          |`refactor(payment): Simplify webhook handler`|
|**test**    |테스트 추가/수정        |`test(user): Add login test cases`           |
|**chore**   |빌드/설정 변경         |`chore: Update Go to 1.21`                   |
|**perf**    |성능 개선            |`perf(db): Optimize order query with index`  |
|**ci**      |CI/CD 설정 변경      |`ci: Add security scan to pipeline`          |
|**revert**  |커밋 되돌리기          |`revert: Revert "feat: Add feature X"`       |

### **Scope (선택)**

```bash
# 도메인별 스코프
feat(user): ...
feat(order): ...
feat(payment): ...
feat(restaurant): ...

# 계층별 스코프
fix(api): ...
fix(domain): ...
fix(infrastructure): ...

# 도구별 스코프
chore(docker): ...
chore(k8s): ...
ci(github-actions): ...
```

### **Subject (필수)**

```bash
# ✅ 좋은 예시
feat(auth): Add JWT token refresh endpoint
fix(order): Resolve race condition in order creation
docs(setup): Update development environment guide

# ❌ 나쁜 예시
feat: add stuff
fix: bug fix
update readme
```

**규칙**:

- 동사 원형으로 시작 (Add, Fix, Update)
- 첫 글자 대문자
- 마침표 없음
- 50자 이내
- 명령문 사용 (“Added”가 아닌 “Add”)

### **Body (선택)**

```bash
# 예시 1: 상세 설명
feat(payment): Integrate Toss Payments webhook

- Add webhook endpoint for payment confirmation
- Implement signature verification
- Add automatic order status update
- Handle payment failure scenarios

Closes #123

# 예시 2: Breaking Change
feat(api): Change order creation response format

BREAKING CHANGE: Order creation API now returns 
order_id instead of id field. Update all clients 
to use the new field name.

Closes #456
```

### **Footer (선택)**

```bash
# 이슈 참조
Closes #123
Fixes #456
Resolves #789

# 리뷰어 참조
Reviewed-by: @backend-dev
Co-authored-by: @frontend-dev

# Breaking Change
BREAKING CHANGE: API endpoint changed from /v1/orders to /v2/orders
```

### **완전한 커밋 메시지 예시**

```bash
feat(order): Add real-time order status notification

Implement WebSocket-based real-time order status updates:
- Create WebSocket endpoint /ws/orders/{order_id}
- Add Redis Pub/Sub for message broadcasting
- Implement automatic reconnection on client side
- Add connection authentication with JWT

This enables customers to see order status updates 
in real-time without polling the API.

Performance: Reduces API calls by 80%
Closes #234
Reviewed-by: @backend-lead
```

-----

## 🔍 **Pull Request (PR) 프로세스**

### **PR 생성 전 체크리스트**

```bash
# 1. 최신 develop 브랜치 병합
git checkout develop
git pull origin develop
git checkout feature/your-branch
git merge develop

# 2. 충돌 해결 (있는 경우)

# 3. 테스트 실행
go test ./...
go test -race ./...
golangci-lint run

# 4. 코드 포맷팅
gofmt -w .
goimports -w .

# 5. 커밋 정리 (필요시 squash)
git rebase -i develop

# 6. 푸시
git push origin feature/your-branch
```

### **PR 템플릿**

```markdown
## 📋 변경 사항 요약
<!-- 이 PR에서 무엇을 변경했는지 간략히 설명 -->

## 🎯 관련 이슈
Closes #123
Related to #456

## 🔄 변경 타입
- [ ] 새로운 기능 (Breaking change 아님)
- [ ] 버그 수정 (Breaking change 아님)
- [ ] Breaking change (기존 기능에 영향)
- [ ] 문서 업데이트
- [ ] 코드 리팩토링
- [ ] 성능 개선
- [ ] 테스트 추가/수정

## 📝 상세 설명
<!-- 변경 사항에 대한 상세한 설명 -->

### 변경된 내용
- 
- 

### 변경 이유
- 

### 구현 방법
- 

## 🧪 테스트 방법
<!-- 이 변경사항을 어떻게 테스트했는지 -->

```bash
# 테스트 명령어
go test ./internal/order/...
```

### 테스트 시나리오

1. 
1. 
1. 

## 📸 스크린샷 (UI 변경 시)

<!-- 변경 전/후 스크린샷 -->

## ✅ 체크리스트

- [ ] 코드가 프로젝트 코딩 스타일을 따름
- [ ] 자체 코드 리뷰 완료
- [ ] 주석 추가 (복잡한 로직에 대해)
- [ ] 문서 업데이트 (필요 시)
- [ ] 테스트 추가/수정
- [ ] 모든 테스트 통과
- [ ] Breaking change인 경우 마이그레이션 가이드 작성
- [ ] CI/CD 파이프라인 통과

## 📚 참고 자료

<!-- 관련 문서, 링크, 참고 자료 -->

## 💬 추가 코멘트

<!-- 리뷰어에게 전달할 추가 정보 -->

```
### **PR 제목 규칙**

```bash
# 형식: [Type] Brief description
[Feat] Add OAuth2.0 Google login
[Fix] Resolve order calculation bug
[Docs] Update API documentation
[Refactor] Simplify payment webhook handler
[Perf] Optimize database queries with indexes
```

### **PR 크기 가이드라인**

|크기    |변경 라인 수 |리뷰 시간|권장사항    |
|------|--------|-----|--------|
|**XS**|< 50    |10분  |이상적 ✅   |
|**S** |50-200  |30분  |좋음 ✅    |
|**M** |200-500 |1시간  |허용 ⚠️    |
|**L** |500-1000|2시간+ |분할 고려 ⚠️ |
|**XL**|> 1000  |4시간+ |반드시 분할 ❌|

**큰 PR 분할 전략**:

```bash
# 1단계: 기본 구조 추가
feature/123-add-auth-structure

# 2단계: 비즈니스 로직 구현
feature/123-implement-auth-logic

# 3단계: 테스트 및 문서
feature/123-add-auth-tests
```

-----

## 👀 **코드 리뷰 가이드**

### **리뷰어 역할 및 책임**

#### **Primary Reviewer (필수)**

- ✅ 코드 로직 검증
- ✅ 아키텍처 패턴 준수 확인
- ✅ 성능 이슈 검토
- ✅ 보안 취약점 검사

#### **Secondary Reviewer (선택)**

- ✅ 코드 스타일 확인
- ✅ 테스트 커버리지 검토
- ✅ 문서화 확인

### **리뷰 우선순위**

|우선순위  |항목   |설명                       |
|------|-----|-------------------------|
|**P0**|보안   |SQL Injection, XSS, 인증/인가|
|**P0**|버그   |논리 오류, 데이터 손실            |
|**P1**|성능   |N+1 쿼리, 메모리 누수           |
|**P1**|아키텍처 |도메인 분리, 의존성 방향           |
|**P2**|코드 품질|가독성, 중복 코드               |
|**P3**|스타일  |포맷팅, 네이밍                 |

### **리뷰 코멘트 템플릿**

#### **1. Blocking (수정 필수)**

```markdown
🚨 **[BLOCKING]** 보안 취약점

이 코드는 SQL Injection에 취약합니다.

**문제**:
```go
query := fmt.Sprintf("SELECT * FROM users WHERE id = %s", userID)
```

**해결**:

```go
query := "SELECT * FROM users WHERE id = $1"
db.QueryRow(query, userID)
```

**참고**: https://owasp.org/www-community/attacks/SQL_Injection

```
#### **2. Suggestion (개선 제안)**
```markdown
💡 **[SUGGESTION]** 성능 최적화 기회

현재 코드는 N+1 쿼리 문제가 있습니다.

**현재**:
```go
for _, order := range orders {
    items := db.GetOrderItems(order.ID) // N번 쿼리
}
```

**개선**:

```go
orderIDs := extractIDs(orders)
itemsMap := db.GetOrderItemsBatch(orderIDs) // 1번 쿼리
```

**영향**: 100개 주문 조회 시 101개 쿼리 → 2개 쿼리 (50배 향상)

```
#### **3. Question (질문)**
```markdown
❓ **[QUESTION]** 의도 확인

이 로직의 의도가 명확하지 않습니다. 설명 부탁드립니다.

```go
if status == "PENDING" && time.Now().Unix()%2 == 0 {
    // 왜 짝수 초에만 처리하나요?
}
```

```
#### **4. Nit (사소한 지적)**
```markdown
🔹 **[NIT]** 코드 스타일

변수명을 더 명확하게 작성하면 좋겠습니다.

**현재**: `r := getR()`
**제안**: `restaurant := getRestaurant()`
```

### **리뷰 응답 가이드**

#### **리뷰 받는 사람 (Author)**

```markdown
# ✅ 수정 완료
수정했습니다. 확인 부탁드립니다.
Commit: abc123

# 🤔 대안 제시
제안해주신 방법도 좋지만, 다음 이유로 현재 방식을 유지하고 싶습니다:
1. ...
2. ...
의견 부탁드립니다.

# 📚 추가 설명
이 코드는 다음과 같은 이유로 작성되었습니다:
1. ...
2. ...
문서: [링크]

# ❌ 동의하지 않음 (근거 필수)
이 제안은 다음 이유로 적용하기 어렵습니다:
1. 성능 저하 우려 (벤치마크: ...)
2. 기존 API 호환성 문제
대안으로 ... 방법은 어떨까요?
```

### **리뷰 완료 체크리스트**

```bash
# 코드 품질
- [ ] 코드가 DRY 원칙을 따르는가?
- [ ] 함수가 단일 책임을 가지는가?
- [ ] 에러 처리가 적절한가?
- [ ] 로그가 충분한가?

# 보안
- [ ] 입력 검증이 이루어지는가?
- [ ] SQL Injection 방어되는가?
- [ ] 민감 정보가 암호화되는가?
- [ ] 인증/인가가 올바른가?

# 성능
- [ ] N+1 쿼리가 없는가?
- [ ] 불필요한 반복문이 없는가?
- [ ] 캐싱이 적절한가?
- [ ] 인덱스가 활용되는가?

# 테스트
- [ ] 단위 테스트가 작성되었는가?
- [ ] 엣지 케이스가 커버되는가?
- [ ] 테스트 커버리지가 85% 이상인가?
- [ ] 통합 테스트가 필요한가?

# 문서화
- [ ] 복잡한 로직에 주석이 있는가?
- [ ] API 변경 시 문서가 업데이트되었는가?
- [ ] README가 최신 상태인가?
- [ ] CHANGELOG가 업데이트되었는가?
```

-----

## ⚡ **자동화 및 CI/CD**

### **GitHub Actions 워크플로우**

#### **1. PR 검증 워크플로우**

```yaml
# .github/workflows/pr-check.yml
name: PR Validation

on:
  pull_request:
    branches: [develop, main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      
      - name: Run golangci-lint
        uses: golangci/golangci-lint-action@v3
        with:
          version: latest
          args: --timeout 5m

  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      
      - name: Run tests
        run: |
          go test -v -race -coverprofile=coverage.out ./...
          go tool cover -func=coverage.out
      
      - name: Check coverage
        run: |
          coverage=$(go tool cover -func=coverage.out | grep total | awk '{print $3}' | sed 's/%//')
          if (( $(echo "$coverage < 85" | bc -l) )); then
            echo "Coverage is ${coverage}%, minimum is 85%"
            exit 1
          fi

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run gosec
        uses: securego/gosec@master
        with:
          args: '-severity medium ./...'
      
      - name: Run Trivy
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          severity: 'CRITICAL,HIGH'
```

#### **2. 자동 병합 조건**

```yaml
# .github/workflows/auto-merge.yml
name: Auto Merge

on:
  pull_request_review:
    types: [submitted]

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.event.review.state == 'approved'
    steps:
      - name: Auto merge
        uses: pascalgn/automerge-action@v0.15.6
        env:
          GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
          MERGE_LABELS: "automerge,!work-in-progress"
          MERGE_METHOD: "squash"
          MERGE_COMMIT_MESSAGE: "pull-request-title"
          MERGE_RETRIES: 6
          MERGE_RETRY_SLEEP: 10000
```

### **Pre-commit Hook**

```bash
# .git/hooks/pre-commit
#!/bin/bash

echo "🔍 Running pre-commit checks..."

# 1. 코드 포맷팅
echo "📝 Formatting code..."
gofmt -w .
goimports -w .

# 2. 린트 검사
echo "🔎 Running linter..."
if ! golangci-lint run --fix; then
    echo "❌ Linter failed"
    exit 1
fi

# 3. 단위 테스트
echo "🧪 Running tests..."
if ! go test -short ./...; then
    echo "❌ Tests failed"
    exit 1
fi

# 4. 커밋 메시지 검증
commit_msg=$(cat .git/COMMIT_EDITMSG)
if ! echo "$commit_msg" | grep -qE "^(feat|fix|docs|style|refactor|test|chore|perf|ci|revert)(\(.+\))?: .+"; then
    echo "❌ Invalid commit message format"
    echo "Use: <type>(<scope>): <subject>"
    exit 1
fi

echo "✅ All checks passed!"
```

-----

## 🚀 **배포 워크플로우**

### **배포 환경**

|환경             |브랜치      |트리거         |배포 방법     |
|---------------|---------|------------|----------|
|**Development**|develop  |자동 (PR 병합 시)|자동 배포     |
|**Staging**    |release/*|수동 (승인 후)   |자동 배포     |
|**Production** |main     |수동 (승인 후)   |Blue-Green|

### **배포 체크리스트**

#### **스테이징 배포 전**

```bash
- [ ] 모든 테스트 통과
- [ ] 코드 리뷰 승인
- [ ] 문서 업데이트
- [ ] CHANGELOG 작성
- [ ] 데이터베이스 마이그레이션 검증
- [ ] 환경 변수 확인
```

#### **프로덕션 배포 전**

```bash
- [ ] 스테이징 환경에서 충분한 테스트 (최소 24시간)
- [ ] 성능 테스트 통과 (부하 테스트)
- [ ] 보안 감사 완료
- [ ] 롤백 계획 수립
- [ ] 모니터링 대시보드 준비
- [ ] 이해관계자 승인 (PO, Tech Lead)
- [ ] 배포 공지 (내부 팀, 고객)
- [ ] 백업 완료 (데이터베이스)
```

### **배포 단계**

#### **1. 개발 환경 (자동)**

```bash
# develop 브랜치에 PR 병합 시 자동 배포
1. PR 병합 → develop 브랜치 업데이트
2. CI/CD 파이프라인 트리거
3. 테스트 실행
4. Docker 이미지 빌드
5. 개발 환경 배포 (dev.ordersync.com)
6. 헬스 체크
7. Slack 알림
```

#### **2. 스테이징 환경 (수동 승인)**

```bash
# release 브랜치 생성 시
1. release/v1.2.0 브랜치 생성
2. CI/CD 파이프라인 트리거
3. 통합 테스트 실행
4. 승인 대기 (Tech Lead)
5. 승인 후 배포 (staging.ordersync.com)
6. 스모크 테스트 자동 실행
7. 성능 메트릭 수집
8. 배포 보고서 생성
```

#### **3. 프로덕션 환경 (Blue-Green)**

```bash
# main 브랜치에 병합 시
1. 릴리스 태그 생성 (v1.2.0)
2. 프로덕션 승인 대기 (CTO, PO)
3. Blue 환경에 배포 (ordersync.com)
4. 헬스 체크 (5분간 모니터링)
5. 카나리 배포 (5% 트래픽)
6. 메트릭 확인 (에러율, 응답시간)
7. 100% 트래픽 전환 (Green → Blue)
8. Green 환경 대기 (롤백용)
9. 24시간 후 Green 환경 정리
```

### **롤백 프로세스**

#### **즉시 롤백 (5분 내)**

```bash
# 심각한 버그 발견 시
1. 배포 중단 명령
   kubectl rollout undo deployment/ordersync-api

2. 트래픽 이전 버전으로 전환
   # Blue-Green 스위치
   kubectl patch svc ordersync -p '{"spec":{"selector":{"version":"v1.1.0"}}}'

3. 모니터링 확인
   # 에러율, 응답시간 정상화 확인

4. 포스트모템 이슈 생성
   # 롤백 이유, 영향 범위, 향후 대책
```

#### **계획된 롤백 (1시간 내)**

```bash
# 성능 저하 등 심각하지 않은 이슈
1. 이슈 분석 (30분)
2. 롤백 결정
3. 데이터 정합성 확인
4. 롤백 실행
5. 검증
6. 공지 (고객, 팀)
```

-----

## 📊 **메트릭 및 모니터링**

### **개발 메트릭**

#### **코드 품질 메트릭**

```bash
# 목표 기준
- 테스트 커버리지: ≥ 85%
- 코드 복잡도: < 15 (gocyclo)
- 중복 코드: < 5%
- 린트 이슈: 0개
```

#### **PR 메트릭**

```bash
# 건강한 개발 속도 지표
- PR 크기: < 300 LOC (평균)
- 리뷰 시간: < 4시간 (P95)
- 병합까지 시간: < 24시간 (평균)
- 재작업률: < 30%
```

#### **배포 메트릭**

```bash
# DevOps 성숙도 지표
- 배포 빈도: 주 2-3회
- 변경 실패율: < 5%
- 복구 시간 (MTTR): < 1시간
- 변경 리드 타임: < 1일
```

### **모니터링 대시보드**

#### **개발 대시보드**

```bash
# Grafana 패널
1. PR 현황 (Open/In Review/Merged)
2. 테스트 커버리지 트렌드
3. 빌드 성공률
4. 코드 품질 점수
5. 기술 부채 지표
```

#### **배포 대시보드**

```bash
# 실시간 모니터링
1. 배포 진행 상태
2. 헬스 체크 결과
3. 에러율 (현재 vs 이전 버전)
4. 응답 시간 (P50, P95, P99)
5. 트래픽 분포 (Blue/Green)
```

-----

## 🔧 **도구 및 설정**

### **Git 설정 (.gitconfig)**

```ini
[user]
    name = Your Name
    email = your.email@ordersync.com

[core]
    editor = vim
    autocrlf = input
    whitespace = trailing-space,space-before-tab

[push]
    default = current
    autoSetupRemote = true

[pull]
    rebase = true

[alias]
    # 자주 쓰는 명령어 단축
    co = checkout
    br = branch
    ci = commit
    st = status
    
    # 유용한 명령어
    lg = log --graph --oneline --decorate --all
    undo = reset --soft HEAD^
    amend = commit --amend --no-edit
    
    # 브랜치 정리
    cleanup = !git branch --merged | grep -v '\\*\\|main\\|develop' | xargs -n 1 git branch -d
    
    # 커밋 통계
    stats = shortlog -sn --no-merges

[fetch]
    prune = true

[rebase]
    autoStash = true
```

### **GitHub CLI (gh) 명령어**

```bash
# 설치
brew install gh  # macOS
gh auth login

# PR 생성
gh pr create --title "feat: Add feature" --body "Description" --base develop

# PR 체크아웃
gh pr checkout 123

# PR 리뷰 요청
gh pr review 123 --approve
gh pr review 123 --request-changes --body "Please fix..."
gh pr review 123 --comment --body "LGTM!"

# PR 병합
gh pr merge 123 --squash --delete-branch

# PR 목록 확인
gh pr list --state open --assignee @me
gh pr status

# 이슈 관리
gh issue create --title "Bug: ..." --body "..."
gh issue list --label bug --state open
```

### **유용한 Git Alias**

```bash
# ~/.gitconfig 또는 프로젝트별 설정
[alias]
    # 브랜치 워크플로우
    feature = "!f() { git checkout develop && git pull && git checkout -b feature/$1; }; f"
    bugfix = "!f() { git checkout develop && git pull && git checkout -b bugfix/$1; }; f"
    hotfix = "!f() { git checkout main && git pull && git checkout -b hotfix/$1; }; f"
    
    # PR 준비
    prepare = "!git fetch origin && git rebase origin/develop && git push --force-with-lease"
    
    # 커밋 관리
    fixup = "!git log --oneline -n 20 | fzf | awk '{print $1}' | xargs git commit --fixup"
    squash-all = "!git rebase -i $(git merge-base HEAD develop)"
    
    # 브랜치 동기화
    sync = "!git fetch origin && git rebase origin/$(git branch --show-current)"
    
    # 코드 리뷰
    review = "!f() { gh pr checkout $1 && git pull; }; f"
```

-----

## 📚 **참고 자료**

### **내부 문서**

- [개발 환경 설정](./development-setup.md) - 로컬 환경 구축
- [코딩 표준](./development-coding-standards.md) - Go 코딩 컨벤션
- [테스트 가이드](./development-testing.md) - 테스트 작성 방법
- [API 문서](../architecture/api.md) - API 명세서

### **외부 자료**

- [Conventional Commits](https://www.conventionalcommits.org/) - 커밋 메시지 규칙
- [GitHub Flow](https://docs.github.com/en/get-started/quickstart/github-flow) - Git 워크플로우
- [Code Review Best Practices](https://google.github.io/eng-practices/review/) - Google 코드 리뷰 가이드
- [The Twelve-Factor App](https://12factor.net/) - 애플리케이션 개발 원칙

### **도구 문서**

- [golangci-lint](https://golangci-lint.run/) - Go 린터
- [GitHub Actions](https://docs.github.com/en/actions) - CI/CD 자동화
- [Conventional Changelog](https://github.com/conventional-changelog/conventional-changelog) - 자동 CHANGELOG 생성

-----

## 🚨 **일반적인 문제 해결**

### **문제 1: Merge Conflict**

```bash
# 증상: PR에 충돌 발생
# 해결:

1. develop 브랜치 최신화
git checkout develop
git pull origin develop

2. feature 브랜치로 이동
git checkout feature/your-branch

3. develop 병합 (또는 rebase)
git merge develop
# 또는
git rebase develop

4. 충돌 해결
# 파일 수정 후
git add <resolved-files>
git commit  # merge의 경우
# 또는
git rebase --continue  # rebase의 경우

5. 푸시
git push --force-with-lease origin feature/your-branch
```

### **문제 2: 잘못된 커밋 수정**

```bash
# 마지막 커밋 메시지 수정
git commit --amend -m "fix: Correct commit message"
git push --force-with-lease

# 마지막 커밋에 파일 추가
git add forgotten-file.go
git commit --amend --no-edit
git push --force-with-lease

# 여러 커밋 수정 (interactive rebase)
git rebase -i HEAD~3
# pick/edit/squash/drop 선택
git push --force-with-lease
```

### **문제 3: 실수로 main에 푸시**

```bash
# 즉시 되돌리기
git checkout main
git reset --hard HEAD~1
git push --force origin main

# 새로운 커밋으로 되돌리기 (권장)
git revert HEAD
git push origin main

# 알림
# Slack #dev-alerts 채널에 즉시 공지
```

### **문제 4: PR이 너무 큼**

```bash
# 해결: PR 분할

1. 원본 브랜치 백업
git checkout feature/large-pr
git checkout -b feature/large-pr-backup

2. 첫 번째 PR용 브랜치 생성
git checkout develop
git checkout -b feature/large-pr-part1

3. 필요한 커밋만 체리픽
git cherry-pick <commit-hash-1>
git cherry-pick <commit-hash-2>

4. 첫 번째 PR 생성
git push origin feature/large-pr-part1
# PR 생성: Part 1 - 기본 구조

5. 두 번째 PR용 브랜치 (첫 번째 PR 기반)
git checkout -b feature/large-pr-part2 feature/large-pr-part1
git cherry-pick <commit-hash-3>
git cherry-pick <commit-hash-4>

6. 두 번째 PR 생성
git push origin feature/large-pr-part2
# PR 생성: Part 2 - 비즈니스 로직 (depends on #123)
```

### **문제 5: CI/CD 실패**

```bash
# 1. 로그 확인
gh run view <run-id>

# 2. 로컬에서 재현
docker compose up -d
make test

# 3. 캐시 문제 해결
git commit --allow-empty -m "ci: Clear cache"
git push

# 4. 워크플로우 재실행
gh run rerun <run-id>
```

-----

## 📝 **체크리스트 요약**

### **개발 시작 전**

```bash
□ 이슈 티켓 확인 및 할당
□ 브랜치 명명 규칙 확인
□ develop 브랜치 최신화
□ 로컬 환경 테스트
```

### **개발 중**

```bash
□ 커밋 메시지 규칙 준수
□ 자주 커밋 (작은 단위)
□ 정기적으로 develop 동기화
□ 테스트 작성 (TDD)
```

### **PR 생성 전**

```bash
□ 자체 코드 리뷰
□ 테스트 전체 실행
□ 린트 통과
□ 충돌 해결
□ PR 템플릿 작성
```

### **PR 리뷰 중**

```bash
□ 리뷰 코멘트 신속 대응
□ 변경 사항 재테스트
□ 문서 업데이트
□ CI/CD 통과 확인
```

### **병합 후**

```bash
□ 로컬 브랜치 삭제
□ 배포 확인
□ 모니터링 체크
□ 이슈 종료
```

-----

## 🎯 **목표 및 KPI**

### **팀 목표**

```bash
✅ PR 리뷰 시간 < 4시간 (P95)
✅ 배포 빈도 주 2-3회
✅ 변경 실패율 < 5%
✅ 테스트 커버리지 ≥ 85%
✅ 기술 부채 < 10% (SonarQube)
```

### **개인 목표**

```bash
✅ 주간 PR 2-3개 병합
✅ 코드 리뷰 참여 주 5회 이상
✅ 버그 발생률 < 5%
✅ 문서 기여 월 1회 이상
```

-----

## 📞 **지원 및 문의**

### **문제 해결 순서**

1. 이 문서의 [문제 해결](#🚨-일반적인-문제-해결) 섹션 확인
1. Slack #dev-support 채널 질문
1. Tech Lead 또는 Senior Developer 문의
1. GitHub Discussion 이슈 생성

### **연락처**

- **Slack**: #dev-general, #dev-support
- **이메일**: dev-team@ordersync.com
- **GitHub**: [@ordersync/developers](https://github.com/orgs/ordersync/teams/developers)

-----

## 📝 **변경 이력**

|버전   |날짜        |변경 내용         |작성자           |
|-----|----------|--------------|--------------|
|1.0.0|2025-09-27|초기 워크플로우 문서 작성|OrderSync Team|
|1.0.1|2025-09-28|자동 병합 규칙 추가   |DevOps Team   |

-----

<div align="center">

**OrderSync Development Workflow v1.0.0**

*Collaborate Efficiently, Ship Safely* 🚀

**Happy Coding!**

</div>
