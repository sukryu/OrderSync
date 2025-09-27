# 🛠️ OrderSync 개발 환경 설정 가이드

> **Version**: 1.0.0  
> **Last Updated**: 2025-09-27  
> **Status**: Approved

-----

## 📋 **개요**

이 문서는 OrderSync 프로젝트의 로컬 개발 환경을 설정하는 방법을 안내합니다.

**목적**:

- ✅ 개발자가 5분 내에 로컬 환경 구축
- ✅ 일관된 개발 환경 보장 (모든 팀원 동일)
- ✅ 프로덕션과 최대한 유사한 환경 제공

**대상 독자**:

- 백엔드 개발자 (Go)
- 풀스택 개발자
- DevOps 엔지니어

-----

## 💻 **시스템 요구사항**

### **하드웨어**

|항목     |최소 사양     |권장 사양   |
|-------|----------|--------|
|**CPU**|2코어       |4코어 이상  |
|**RAM**|4GB       |8GB 이상  |
|**디스크**|10GB 여유 공간|20GB SSD|

### **운영체제**

- ✅ **macOS** (12.0 Monterey 이상)
- ✅ **Linux** (Ubuntu 20.04+, Fedora 35+, Arch)
- ✅ **Windows** (WSL2 필수 - Ubuntu 22.04 권장)

### **필수 소프트웨어**

|도구                |버전    |설치 확인                   |
|------------------|------|------------------------|
|**Go**            |1.21+ |`go version`            |
|**Docker**        |24.0+ |`docker --version`      |
|**Docker Compose**|v2.20+|`docker compose version`|
|**Git**           |2.30+ |`git --version`         |
|**Make**          |4.0+  |`make --version`        |

-----

## 🚀 **빠른 시작 (Quick Start)**

### **Step 1: 저장소 클론**

```bash
# HTTPS
git clone https://github.com/ordersync/ordersync.git
cd ordersync

# SSH (권장)
git clone git@github.com:ordersync/ordersync.git
cd ordersync
```

### **Step 2: 환경 변수 설정**

```bash
# .env 파일 생성
cp .env.example .env

# 필수 값 설정 (에디터로 열기)
vim .env  # 또는 nano, code
```

**.env 파일 내용**:

```bash
# 데이터베이스
DATABASE_URL=postgresql://ordersync:dev_password@localhost:5432/ordersync_dev

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=dev_secret_change_in_production_minimum_32_chars
JWT_EXPIRES_IN=3600

# 암호화
ENCRYPTION_KEY=dev_encryption_key_change_in_production_32_bytes

# 토스페이먼츠 (테스트 키)
TOSS_SECRET_KEY=test_sk_YOUR_TEST_SECRET_KEY
TOSS_CLIENT_KEY=test_ck_YOUR_TEST_CLIENT_KEY

# 서버
PORT=8080
ENV=development
LOG_LEVEL=debug

# 외부 서비스 (선택)
SENTRY_DSN=
AWS_REGION=ap-northeast-2
```

### **Step 3: Docker Compose로 인프라 실행**

```bash
# 데이터베이스 및 Redis 시작
docker compose up -d

# 로그 확인
docker compose logs -f

# 상태 확인
docker compose ps
```

**실행 결과**:

```
NAME                IMAGE               STATUS
ordersync-postgres  postgres:15-alpine  Up (healthy)
ordersync-redis     redis:7-alpine      Up (healthy)
```

### **Step 4: 데이터베이스 마이그레이션**

```bash
# 마이그레이션 도구 설치
go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest

# 마이그레이션 실행
make migrate-up

# 또는 직접 실행
migrate -path migrations -database "${DATABASE_URL}" up
```

### **Step 5: 의존성 설치**

```bash
# Go 모듈 다운로드
go mod download

# 개발 도구 설치
make install-tools
```

### **Step 6: 서버 실행**

```bash
# 핫 리로드 개발 서버 (권장)
make dev

# 또는 Air 직접 실행
air

# 또는 일반 실행
go run cmd/api/main.go
```

**서버 실행 확인**:

```bash
# 헬스 체크
curl http://localhost:8080/health

# 응답 예시
{"status":"ok","database":"connected","redis":"connected"}
```

### **Step 7: 초기 데이터 시드 (선택)**

```bash
# 테스트용 데이터 생성
make seed

# 또는
go run scripts/seed.go
```

**생성되는 데이터**:

- 테스트 사용자 (customer@example.com / owner@example.com)
- 샘플 매장 (테스트 치킨집)
- 샘플 메뉴 (후라이드 치킨, 양념 치킨)

-----

## 🔧 **상세 설치 가이드**

### **1. Go 설치**

#### **macOS**

```bash
# Homebrew 사용
brew install go@1.21

# 버전 확인
go version
# 출력: go version go1.21.x darwin/amd64
```

#### **Linux (Ubuntu/Debian)**

```bash
# 공식 바이너리 다운로드
wget https://go.dev/dl/go1.21.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.5.linux-amd64.tar.gz

# PATH 설정 (~/.bashrc 또는 ~/.zshrc)
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# 적용
source ~/.bashrc

# 버전 확인
go version
```

#### **Windows (WSL2)**

```bash
# WSL2 Ubuntu에서 실행
wget https://go.dev/dl/go1.21.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.5.linux-amd64.tar.gz

# .bashrc에 추가
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```

-----

### **2. Docker & Docker Compose 설치**

#### **macOS**

```bash
# Docker Desktop 다운로드 및 설치
# https://docs.docker.com/desktop/install/mac-install/

# 또는 Homebrew
brew install --cask docker

# 설치 확인
docker --version
docker compose version
```

#### **Linux (Ubuntu)**

```bash
# Docker 공식 저장소 추가
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 현재 사용자를 docker 그룹에 추가
sudo usermod -aG docker $USER
newgrp docker

# Docker Compose 설치
sudo apt-get update
sudo apt-get install docker-compose-plugin

# 설치 확인
docker --version
docker compose version
```

#### **Windows**

```bash
# Docker Desktop for Windows 설치 필수
# WSL2 백엔드 활성화 필요
# https://docs.docker.com/desktop/install/windows-install/
```

-----

### **3. 개발 도구 설치**

```bash
# Makefile을 통한 일괄 설치
make install-tools
```

**설치되는 도구**:

#### **Air (핫 리로드)**

```bash
go install github.com/cosmtrek/air@latest

# .air.toml 설정 파일 생성
cat > .air.toml << 'EOF'
root = "."
tmp_dir = "tmp"

[build]
  bin = "./tmp/main"
  cmd = "go build -o ./tmp/main ./cmd/api"
  delay = 1000
  exclude_dir = ["assets", "tmp", "vendor", "testdata"]
  exclude_file = []
  exclude_regex = ["_test.go"]
  exclude_unchanged = false
  follow_symlink = false
  include_dir = []
  include_ext = ["go", "tpl", "tmpl", "html"]
  kill_delay = "0s"
  log = "build-errors.log"
  send_interrupt = false
  stop_on_error = true

[color]
  app = ""
  build = "yellow"
  main = "magenta"
  runner = "green"
  watcher = "cyan"

[log]
  time = false

[misc]
  clean_on_exit = true
EOF
```

#### **golangci-lint (코드 린터)**

```bash
# 설치
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

# 설정 파일 (.golangci.yml)
cat > .golangci.yml << 'EOF'
linters:
  enable:
    - gofmt
    - govet
    - errcheck
    - staticcheck
    - unused
    - gosimple
    - structcheck
    - varcheck
    - ineffassign
    - deadcode
    - typecheck
    - gosec
    - dupl

linters-settings:
  govet:
    check-shadowing: true
  golint:
    min-confidence: 0.8
  gocyclo:
    min-complexity: 15

run:
  timeout: 5m
  tests: true

issues:
  exclude-use-default: false
  max-issues-per-linter: 0
  max-same-issues: 0
EOF

# 실행
golangci-lint run
```

#### **migrate (DB 마이그레이션)**

```bash
go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest
```

#### **sqlc (SQL 코드 생성)**

```bash
go install github.com/sqlc-dev/sqlc/cmd/sqlc@latest

# sqlc.yaml 설정
cat > sqlc.yaml << 'EOF'
version: "2"
sql:
  - engine: "postgresql"
    queries: "internal/infrastructure/database/queries"
    schema: "migrations"
    gen:
      go:
        package: "database"
        out: "internal/infrastructure/database"
        emit_json_tags: true
        emit_interface: true
EOF
```

#### **swag (Swagger 문서 생성)**

```bash
go install github.com/swaggo/swag/cmd/swag@latest

# Swagger 문서 생성
swag init -g cmd/api/main.go -o docs
```

-----

## 📦 **Docker Compose 설정**

### **docker-compose.yml**

```yaml
version: '3.9'

services:
  # PostgreSQL 데이터베이스
  postgres:
    image: postgres:15-alpine
    container_name: ordersync-postgres
    environment:
      POSTGRES_DB: ordersync_dev
      POSTGRES_USER: ordersync
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql
    command:
      - "postgres"
      - "-c"
      - "max_connections=200"
      - "-c"
      - "shared_buffers=256MB"
      - "-c"
      - "log_statement=all"  # 개발 환경에서만
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ordersync"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - ordersync

  # Redis 캐시
  redis:
    image: redis:7-alpine
    container_name: ordersync-redis
    ports:
      - "6379:6379"
    command: redis-server --maxmemory 512mb --maxmemory-policy allkeys-lru
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    networks:
      - ordersync

  # API 서버 (개발용 - 선택)
  api:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: ordersync-api
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgresql://ordersync:dev_password@postgres:5432/ordersync_dev
      REDIS_URL: redis://redis:6379
      JWT_SECRET: dev_secret_change_in_production_minimum_32_chars
      ENCRYPTION_KEY: dev_encryption_key_change_in_production_32_bytes
      LOG_LEVEL: debug
      ENV: development
    volumes:
      - .:/app
      - /app/tmp  # Air 핫 리로드용
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    command: air
    networks:
      - ordersync

  # Prometheus (메트릭 수집)
  prometheus:
    image: prom/prometheus:latest
    container_name: ordersync-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    networks:
      - ordersync

  # Grafana (모니터링 대시보드)
  grafana:
    image: grafana/grafana:latest
    container_name: ordersync-grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
      GF_USERS_ALLOW_SIGN_UP: false
    volumes:
      - grafana_data:/var/lib/grafana
      - ./config/grafana-datasources.yml:/etc/grafana/provisioning/datasources/datasources.yml
    depends_on:
      - prometheus
    networks:
      - ordersync

volumes:
  postgres_data:
  redis_data:
  prometheus_data:
  grafana_data:

networks:
  ordersync:
    driver: bridge
```

### **Dockerfile.dev (개발용)**

```dockerfile
FROM golang:1.21-alpine AS base

# 빌드 도구 설치
RUN apk add --no-cache git ca-certificates tzdata make

# Air 설치 (핫 리로드)
RUN go install github.com/cosmtrek/air@latest

WORKDIR /app

# Go 모듈 의존성 복사
COPY go.mod go.sum ./
RUN go mod download

# 소스 코드 복사
COPY . .

# Air 실행
CMD ["air", "-c", ".air.toml"]
```

-----

## 🗄️ **데이터베이스 설정**

### **초기 스키마 생성 (scripts/init.sql)**

```sql
-- scripts/init.sql
-- 개발 환경 초기화 스크립트

-- Extension 설치
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- 개발용 주석
COMMENT ON DATABASE ordersync_dev IS 'OrderSync Development Database';

-- 초기 설정 완료 로그
DO $$
BEGIN
    RAISE NOTICE 'OrderSync development database initialized successfully';
END $$;
```

### **마이그레이션 실행**

```bash
# 최신 버전으로 마이그레이션
migrate -path migrations -database "${DATABASE_URL}" up

# 특정 버전으로 마이그레이션
migrate -path migrations -database "${DATABASE_URL}" goto 3

# 롤백 (1단계)
migrate -path migrations -database "${DATABASE_URL}" down 1

# 전체 롤백 (주의!)
migrate -path migrations -database "${DATABASE_URL}" down -all

# 현재 버전 확인
migrate -path migrations -database "${DATABASE_URL}" version
```

### **시드 데이터 생성 (scripts/seed.go)**

```go
// scripts/seed.go
package main

import (
    "context"
    "log"
    "github.com/google/uuid"
    "github.com/ordersync/ordersync/internal/infrastructure/database"
)

func main() {
    ctx := context.Background()
    
    // 데이터베이스 연결
    db, err := database.Connect(ctx)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    
    // 1. 테스트 사용자 생성
    customerID := uuid.New()
    ownerID := uuid.New()
    
    db.Exec(`
        INSERT INTO users (id, email, password_hash, name, role, status)
        VALUES 
            ($1, 'customer@example.com', $2, '테스트고객', 'CUSTOMER', 'ACTIVE'),
            ($3, 'owner@example.com', $4, '테스트사장님', 'RESTAURANT_OWNER', 'ACTIVE')
    `, customerID, hashPassword("password123"), ownerID, hashPassword("password123"))
    
    // 2. 테스트 매장 생성
    restaurantID := uuid.New()
    db.Exec(`
        INSERT INTO restaurants (id, owner_id, name, table_ordering_enabled)
        VALUES ($1, $2, '테스트 치킨집', true)
    `, restaurantID, ownerID)
    
    // 3. 테스트 메뉴 생성
    db.Exec(`
        INSERT INTO menus (id, restaurant_id, name, price, is_available)
        VALUES 
            ($1, $2, '후라이드 치킨', 18000, true),
            ($3, $2, '양념 치킨', 19000, true)
    `, uuid.New(), restaurantID, uuid.New())
    
    log.Println("✅ 시드 데이터 생성 완료")
    log.Println("📧 고객 계정: customer@example.com / password123")
    log.Println("📧 사장님 계정: owner@example.com / password123")
}

func hashPassword(password string) string {
    // bcrypt 해싱 구현
    // ...
}
```

-----

## 🧪 **테스트 실행**

### **단위 테스트**

```bash
# 전체 테스트 실행
go test ./...

# 커버리지 포함
go test -cover ./...

# 상세 커버리지 리포트
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html
open coverage.html

# 특정 패키지 테스트
go test ./internal/domain/user/...

# Verbose 모드
go test -v ./...

# Race Condition 검사
go test -race ./...
```

### **통합 테스트**

```bash
# 통합 테스트 (DB 필요)
go test -tags=integration ./...

# 또는 Makefile
make test-integration
```

### **E2E 테스트**

```bash
# API 서버 실행 후
make test-e2e
```

-----

## 🎨 **IDE 설정**

### **Visual Studio Code**

#### **추천 확장 프로그램**

`.vscode/extensions.json`:

```json
{
  "recommendations": [
    "golang.go",
    "ms-azuretools.vscode-docker",
    "ms-vscode-remote.remote-containers",
    "eamodio.gitlens",
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "humao.rest-client",
    "redhat.vscode-yaml"
  ]
}
```

#### **워크스페이스 설정**

`.vscode/settings.json`:

```json
{
  "go.useLanguageServer": true,
  "go.lintTool": "golangci-lint",
  "go.lintOnSave": "package",
  "go.formatTool": "goimports",
  "go.testFlags": ["-v", "-race"],
  "go.coverOnSave": true,
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  },
  "[go]": {
    "editor.defaultFormatter": "golang.go"
  },
  "files.exclude": {
    "**/.git": true,
    "**/tmp": true,
    "**/.DS_Store": true
  }
}
```

#### **디버깅 설정**

`.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch API Server",
      "type": "go",
      "request": "launch",
      "mode": "debug",
      "program": "${workspaceFolder}/cmd/api",
      "env": {
        "DATABASE_URL": "postgresql://ordersync:dev_password@localhost:5432/ordersync_dev",
        "REDIS_URL": "redis://localhost:6379",
        "JWT_SECRET": "dev_secret_change_in_production_minimum_32_chars",
        "LOG_LEVEL": "debug"
      },
      "args": []
    },
    {
      "name": "Run Tests",
      "type": "go",
      "request": "launch",
      "mode": "test",
      "program": "${workspaceFolder}",
      "env": {},
      "args": ["-v"]
    }
  ]
}
```

### **GoLand / IntelliJ IDEA**

#### **Go 모듈 설정**

1. Settings → Go → Go Modules
1. ✅ Enable Go modules integration
1. Environment: `GOPROXY=https://proxy.golang.org,direct`

#### **Run Configuration**

- Name: `OrderSync API`
- Run kind: `Directory`
- Directory: `cmd/api`
- Environment: `.env` 파일 참조

-----

## 📝 **Makefile 명령어**

```makefile
# Makefile
.PHONY: help dev build test lint migrate-up migrate-down seed clean

help: ## 도움말 표시
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

install-tools: ## 개발 도구 설치
	go install github.com/cosmtrek/air@latest
	go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
	go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest
	go install github.com/sqlc-dev/sqlc/cmd/sqlc@latest
	go install github.com/swaggo/swag/cmd/swag@latest

dev: ## 개발 서버 실행 (핫 리로드)
	air

build: ## 프로덕션 빌드
	go build -ldflags="-s -w" -o bin/ordersync cmd/api/main.go

test: ## 단위 테스트 실행
	go test -v -race -cover ./...

test-integration: ## 통합 테스트 실행
	go test -v -tags=integration ./...

lint: ## 코드 린트
	golangci-lint run --timeout 5m

migrate-up: ## 데이터베이스 마이그레이션 (최신)
	migrate -path migrations -database "${DATABASE_URL}" up

migrate-down: ## 데이터베이스 롤백 (1단계)
	migrate -path migrations -database "${DATABASE_URL}" down 1

migrate-create: ## 새 마이그레이션 생성 (usage: make migrate-create name=add_users_table)
	migrate create -ext sql -dir migrations -seq $(name)

seed: ## 시드 데이터 생성
	go run scripts/seed.go

docker-up: ## Docker Compose 시작
	docker compose up -d

docker-down: ## Docker Compose 중지
	docker compose down

docker-logs: ## Docker 로그 확인
	docker compose logs -f

swagger: ## Swagger 문서 생성
	swag init -g cmd/api/main.go -o docs

clean: ## 빌드 파일 정리
	rm -rf bin/ tmp/ coverage.out coverage.html
	go clean -cache -testcache -modcache

.DEFAULT_GOAL := help
```

-----

## 🚨 **트러블슈팅**

### **문제 1: Docker Compose 실행 실패**

**증상**:

```
Error: Cannot connect to the Docker daemon
```

**해결**:

```bash
# Docker 서비스 시작
sudo systemctl start docker  # Linux
open -a Docker  # macOS

# Docker Desktop 실행 확인
docker ps
```

-----

### **문제 2: PostgreSQL 연결 실패**

**증상**:

```
Error: connection refused (localhost:5432)
```

**해결**:

```bash
# 컨테이너 상태 확인
docker compose ps

# PostgreSQL 로그 확인
docker compose logs postgres

# 포트 충돌 확인
lsof -i :5432

# 기존 PostgreSQL 중지
brew services stop postgresql  # macOS
sudo systemctl stop postgresql  # Linux
```

-----

### **문제 3: Go 모듈 다운로드 실패**

**증상**:

```
Error: go: github.com/... unknown revision
```

**해결**:

```bash
# 프록시 설정
export GOPROXY=https://proxy.golang.org,direct

# 모듈 캐시 정리
go clean -modcache

# 다시 다운로드
go mod download
```

-----

### **문제 4: Air 핫 리로드 작동 안 함**

**증상**:

```
코드 변경 시 서버가 자동 재시작되지 않음
```

**해결**:

```bash
# Air 재설치
go install github.com/cosmtrek/air@latest

# .air.toml 권한 확인
chmod +x .air.toml

# tmp 디렉토리 정리
rm -rf tmp/
mkdir tmp

# Air 수동 실행
air -c .air.toml
```

-----

### **문제 5: 마이그레이션 버전 충돌**

**증상**:

```
Error: Dirty database version 2. Fix and force version.
```

**해결**:

```bash
# 현재 버전 확인
migrate -path migrations -database "${DATABASE_URL}" version

# 강제로 버전 설정 (주의!)
migrate -path migrations -database "${DATABASE_URL}" force 1

# 다시 마이그레이션
migrate -path migrations -database "${DATABASE_URL}" up
```

-----

## ✅ **설치 확인 체크리스트**

```bash
# 모든 항목이 ✅ 표시되어야 함

# 1. Go 설치
go version
# ✅ go version go1.21.x

# 2. Docker 설치
docker --version
# ✅ Docker version 24.x

# 3. Docker Compose 설치
docker compose version
# ✅ Docker Compose version v2.x

# 4. PostgreSQL 실행
docker compose ps postgres
# ✅ STATUS: Up (healthy)

# 5. Redis 실행
docker compose ps redis
# ✅ STATUS: Up (healthy)

# 6. 데이터베이스 연결
psql "${DATABASE_URL}" -c "SELECT 1"
# ✅ 1

# 7. Redis 연결
redis-cli -u "${REDIS_URL}" ping
# ✅ PONG

# 8. API 서버 실행
curl http://localhost:8080/health
# ✅ {"status":"ok"}

# 9. 마이그레이션 상태
migrate -path migrations -database "${DATABASE_URL}" version
# ✅ 5 (현재 버전 번호)

# 10. 테스트 실행
go test ./... -short
# ✅ PASS
```

-----

## 📚 **다음 단계**

설치가 완료되었다면:

1. ✅ **[개발 워크플로우](./development-workflow.md)** - Git 브랜치 전략
1. ✅ **[코딩 표준](./development-coding-standards.md)** - Go 코딩 컨벤션
1. ✅ **[API 문서](../architecture/api.md)** - API 명세서
1. ✅ **[테스트 가이드](./development-testing.md)** - 테스트 작성 방법
1. ✅ **[데이터베이스 스키마](../architecture/database-schema.md)** - DB 구조
1. ✅ **[보안 가이드](./security-guidelines.md)** - 보안 코딩 규칙

-----

## 🔗 **유용한 링크**

### **공식 문서**

- [Go 공식 문서](https://go.dev/doc/)
- [PostgreSQL 문서](https://www.postgresql.org/docs/)
- [Redis 문서](https://redis.io/docs/)
- [Docker 문서](https://docs.docker.com/)

### **내부 문서**

- [프로젝트 README](../README.md)
- [아키텍처 개요](../architecture/architecture-readme.md)
- [도메인 모델](../architecture/architecture-domain.md)
- [기술 스택](../architecture/architecture-tech.md)

### **외부 도구**

- [토스페이먼츠 개발자 센터](https://docs.tosspayments.com/)
- [Swagger UI](http://localhost:8080/swagger/index.html) (로컬 실행 시)
- [Prometheus](http://localhost:9090) (로컬 실행 시)
- [Grafana](http://localhost:3000) (로컬 실행 시)

-----

## 💡 **개발 팁**

### **생산성 향상**

#### **1. 자주 사용하는 명령어 Alias 설정**

```bash
# ~/.bashrc 또는 ~/.zshrc에 추가
alias gdev='make dev'
alias gtest='go test -v ./...'
alias glint='golangci-lint run'
alias gmig='migrate -path migrations -database "${DATABASE_URL}"'
alias dcu='docker compose up -d'
alias dcd='docker compose down'
alias dcl='docker compose logs -f'

# 적용
source ~/.bashrc  # 또는 source ~/.zshrc
```

#### **2. 환경 변수 자동 로드 (direnv)**

```bash
# direnv 설치
brew install direnv  # macOS
sudo apt install direnv  # Linux

# 셸 설정에 추가
echo 'eval "$(direnv hook bash)"' >> ~/.bashrc  # bash
echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc    # zsh

# .envrc 파일 생성
cat > .envrc << 'EOF'
# .env 파일 자동 로드
dotenv

# Go 환경 변수
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# 프로젝트별 설정
export LOG_LEVEL=debug
export ENV=development
EOF

# 허용
direnv allow .
```

#### **3. Git Pre-commit Hook**

```bash
# .git/hooks/pre-commit 생성
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash

# 포맷팅 자동 실행
echo "🔄 Running gofmt..."
gofmt -w .

# 린트 검사
echo "🔍 Running golangci-lint..."
golangci-lint run --fix

# 테스트 실행
echo "🧪 Running tests..."
go test -short ./...

if [ $? -ne 0 ]; then
    echo "❌ Tests failed. Commit aborted."
    exit 1
fi

echo "✅ All checks passed!"
EOF

# 실행 권한 부여
chmod +x .git/hooks/pre-commit
```

### **디버깅 팁**

#### **1. 상세 로깅 활성화**

```bash
# 환경 변수로 로그 레벨 변경
export LOG_LEVEL=debug

# 또는 .env 파일 수정
LOG_LEVEL=debug
```

#### **2. PostgreSQL 쿼리 로깅**

```sql
-- psql 접속
psql "${DATABASE_URL}"

-- 쿼리 로깅 활성화
ALTER SYSTEM SET log_statement = 'all';
SELECT pg_reload_conf();

-- 느린 쿼리만 로깅 (1초 이상)
ALTER SYSTEM SET log_min_duration_statement = 1000;
```

#### **3. Redis 명령어 모니터링**

```bash
# Redis CLI 접속
redis-cli -u "${REDIS_URL}"

# 실시간 명령어 모니터링
MONITOR

# 특정 키 검색
KEYS order:*

# 키 TTL 확인
TTL menu:restaurant_id_123
```

-----

## 🔐 **보안 주의사항**

### **개발 환경에서 절대 하지 말아야 할 것**

❌ **프로덕션 데이터베이스에 연결하지 말 것**

```bash
# 잘못된 예시
DATABASE_URL=postgresql://prod_user:prod_pass@prod.example.com:5432/ordersync_prod
```

❌ **실제 고객 데이터를 로컬에 복사하지 말 것**

```bash
# 절대 금지!
pg_dump prod_db > local_dump.sql
psql dev_db < local_dump.sql
```

❌ **실제 결제 키를 개발 환경에 사용하지 말 것**

```bash
# 위험!
TOSS_SECRET_KEY=live_sk_real_secret_key
```

### **안전한 개발 환경**

✅ **항상 테스트 키 사용**

```bash
TOSS_SECRET_KEY=test_sk_YOUR_TEST_KEY
TOSS_CLIENT_KEY=test_ck_YOUR_TEST_KEY
```

✅ **민감한 정보는 .env에만 저장**

```bash
# .gitignore에 반드시 포함
echo ".env" >> .gitignore
echo ".env.local" >> .gitignore
```

✅ **시크릿 스캔 도구 사용**

```bash
# gitleaks 설치 및 실행
brew install gitleaks
gitleaks detect --source . --verbose
```

-----

## 📊 **성능 모니터링**

### **로컬 개발 중 성능 확인**

#### **1. Prometheus 메트릭 확인**

```bash
# 브라우저에서 열기
open http://localhost:9090

# 주요 메트릭 쿼리
# API 요청 수
sum(rate(api_requests_total[5m])) by (path)

# 응답 시간 P99
histogram_quantile(0.99, rate(api_request_duration_bucket[5m]))

# 에러율
sum(rate(api_requests_total{status=~"5.."}[5m])) / sum(rate(api_requests_total[5m]))
```

#### **2. Grafana 대시보드**

```bash
# Grafana 접속
open http://localhost:3000

# 로그인 (초기 비밀번호)
# Username: admin
# Password: admin

# 대시보드 임포트
# config/grafana-dashboard.json 사용
```

#### **3. Go 프로파일링**

```bash
# CPU 프로파일링
go test -cpuprofile=cpu.prof ./...
go tool pprof cpu.prof

# 메모리 프로파일링
go test -memprofile=mem.prof ./...
go tool pprof mem.prof

# 웹 UI로 보기
go tool pprof -http=:8081 cpu.prof
```

-----

## 🆘 **도움말 및 지원**

### **문제 해결 순서**

1. **문서 확인**: 이 가이드의 [트러블슈팅](#🚨-트러블슈팅) 섹션
1. **로그 확인**: `docker compose logs -f`
1. **이슈 검색**: [GitHub Issues](https://github.com/ordersync/ordersync/issues)
1. **팀 문의**: Slack #dev-support 채널
1. **이슈 등록**: 해결되지 않으면 새 이슈 생성

### **연락처**

- **개발팀 이메일**: dev@ordersync.com
- **Slack**: #dev-general, #dev-support
- **GitHub Discussions**: [토론 포럼](https://github.com/ordersync/ordersync/discussions)

-----

## 📝 **변경 이력**

|버전   |날짜        |변경 내용   |작성자           |
|-----|----------|--------|--------------|
|1.0.0|2025-09-27|초기 문서 작성|OrderSync Team|

-----

<div align="center">

**OrderSync Development Setup Guide v1.0.0**

*Built with ❤️ by OrderSync Team*

**Happy Coding! 🚀**

</div>
