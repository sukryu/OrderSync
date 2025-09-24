# OrderSync Domain Design

<div align="center">

**🎯 Domain-Driven Design • 🔐 Security-First Modeling • ⚡ High-Performance Architecture**

![DDD](https://img.shields.io/badge/DDD-Domain%20Driven%20Design-blue?style=for-the-badge)
![Go](https://img.shields.io/badge/Go-Clean%20Architecture-00ADD8?style=for-the-badge)
![Security](https://img.shields.io/badge/Security-Encrypted%20By%20Design-green?style=for-the-badge)

</div>

-----

## 🎯 **도메인 설계 철학**

OrderSync는 **도메인 주도 설계(DDD)**를 기반으로, **보안과 성능을 최우선**으로 하는 도메인 모델을 구축합니다.

### **핵심 설계 원칙**

1. **🔐 암호화 우선**: 모든 민감 데이터는 도메인 레벨에서 암호화
1. **📊 감사 추적**: 모든 비즈니스 액션은 불변 로그로 기록
1. **⚡ 성능 최적화**: 도메인 객체 설계 시 캐싱과 인덱싱 고려
1. **🎯 경계 명확화**: 각 도메인의 책임과 의존성을 명확히 분리
1. **🔄 이벤트 기반**: 도메인 간 통신은 이벤트를 통해 느슨한 결합 유지

-----

## 🏗️ **도메인 구조 개요**

```
📦 OrderSync Domain Architecture

🏪 Core Business Domains
├── Restaurant Domain    # 매장 및 메뉴 관리
├── Order Domain        # 주문 생성 및 상태 관리  
├── Payment Domain      # 결제 처리 및 정산
└── User Domain         # 사용자 인증 및 권한

🛡️ Security & Infrastructure Domains  
├── Encryption Domain   # 데이터 암호화/복호화
├── Audit Domain       # 감사 로그 및 추적
├── Metrics Domain     # 성능 및 보안 메트릭
└── Notification Domain # 실시간 알림 처리
```

-----

## 🏪 **Restaurant Domain**

매장 정보와 메뉴 관리를 담당하는 핵심 도메인입니다.

### **🏢 Restaurant Aggregate**

```go
// internal/restaurant/domain/restaurant.go
package domain

import (
    "context"
    "time"
    "github.com/ordersync/internal/shared/types"
    "github.com/ordersync/internal/encryption/domain"
)

// Restaurant는 매장을 나타내는 Aggregate Root입니다
type Restaurant struct {
    // 기본 식별자
    id       RestaurantID
    ownerID  UserID
    
    // 암호화된 매장 정보 (PII)
    name        encryption.EncryptedString // 매장명
    address     encryption.EncryptedString // 주소 정보
    phoneNumber encryption.EncryptedString // 연락처
    businessNumber encryption.EncryptedString // 사업자등록번호
    
    // 운영 정보 (암호화하지 않음)
    status         RestaurantStatus
    businessHours  BusinessHours
    settings       RestaurantSettings
    
    // 위치 정보 (검색용 - 암호화하지 않음)
    location       GeographicLocation
    
    // 도메인 메타데이터
    createdAt time.Time
    updatedAt time.Time
    version   int64 // 낙관적 잠금용
    
    // 도메인 이벤트 (메모리에만 존재)
    events []DomainEvent
}

// RestaurantID는 매장의 고유 식별자입니다
type RestaurantID string

// RestaurantStatus는 매장 운영 상태를 나타냅니다
type RestaurantStatus string
const (
    RestaurantStatusActive    RestaurantStatus = "ACTIVE"     // 정상 운영
    RestaurantStatusSuspended RestaurantStatus = "SUSPENDED"  // 일시 중단
    RestaurantStatusClosed    RestaurantStatus = "CLOSED"     // 폐점
    RestaurantStatusDeleted   RestaurantStatus = "DELETED"    // 삭제됨
)

// BusinessHours는 매장 운영시간을 관리합니다
type BusinessHours struct {
    Monday    DaySchedule `json:"monday"`
    Tuesday   DaySchedule `json:"tuesday"`
    Wednesday DaySchedule `json:"wednesday"`
    Thursday  DaySchedule `json:"thursday"`
    Friday    DaySchedule `json:"friday"`
    Saturday  DaySchedule `json:"saturday"`
    Sunday    DaySchedule `json:"sunday"`
}

// DaySchedule은 특정 요일의 운영시간입니다
type DaySchedule struct {
    IsOpen    bool      `json:"is_open"`
    OpenTime  time.Time `json:"open_time"`
    CloseTime time.Time `json:"close_time"`
}

// GeographicLocation은 매장 위치 정보입니다
type GeographicLocation struct {
    Latitude  float64 `json:"latitude"`  // 위도
    Longitude float64 `json:"longitude"` // 경도
    Address   string  `json:"address"`   // 검색용 주소 (암호화되지 않음)
}

// RestaurantSettings는 매장별 설정 정보입니다
type RestaurantSettings struct {
    AcceptAnonymousOrders bool          `json:"accept_anonymous_orders"` // 익명 주문 허용
    MinOrderAmount        int64         `json:"min_order_amount"`        // 최소 주문 금액
    MaxOrderItems         int           `json:"max_order_items"`         // 최대 주문 수량
    EstimatedPrepTime     time.Duration `json:"estimated_prep_time"`     // 기본 조리 시간
    AutoAcceptOrders      bool          `json:"auto_accept_orders"`      // 자동 주문 승인
    NotificationSettings  NotificationPreferences `json:"notification_settings"`
}

// Restaurant 생성자 및 팩토리 메서드
func NewRestaurant(
    ownerID UserID,
    name, address, phoneNumber string,
    location GeographicLocation,
    encryptionService encryption.Service,
) (*Restaurant, error) {
    // 새로운 매장 인스턴스를 생성합니다
    // - 필수 필드 검증
    // - 민감 정보 암호화 (name, address, phoneNumber)
    // - 도메인 이벤트 생성 (RestaurantCreated)
    // - 기본 설정값 초기화
}

// Restaurant 도메인 메서드들
func (r *Restaurant) ID() RestaurantID {
    // 매장 ID를 반환합니다
}

func (r *Restaurant) OwnerID() UserID {
    // 매장 소유자 ID를 반환합니다
}

func (r *Restaurant) UpdateBusinessHours(hours BusinessHours) error {
    // 매장 운영시간을 업데이트합니다
    // - 시간 유효성 검증 (open < close)
    // - 24시간 운영 여부 확인
    // - 도메인 이벤트 생성 (BusinessHoursUpdated)
    // - 버전 증가 (낙관적 잠금)
}

func (r *Restaurant) UpdateSettings(settings RestaurantSettings) error {
    // 매장 설정을 업데이트합니다
    // - 설정값 유효성 검증
    // - 최소/최대 제한값 확인
    // - 알림 설정 검증
    // - 도메인 이벤트 생성 (SettingsUpdated)
}

func (r *Restaurant) Suspend(reason string) error {
    // 매장 운영을 일시 중단합니다
    // - 현재 상태 확인 (ACTIVE -> SUSPENDED만 허용)
    // - 진행 중인 주문 확인
    // - 도메인 이벤트 생성 (RestaurantSuspended)
    // - 고객 알림 준비
}

func (r *Restaurant) Activate() error {
    // 매장 운영을 재개합니다
    // - 필수 정보 완성도 확인
    // - 메뉴 활성화 상태 확인
    // - 도메인 이벤트 생성 (RestaurantActivated)
}

func (r *Restaurant) IsOpenNow() bool {
    // 현재 시점의 매장 운영 여부를 확인합니다
    // - 현재 요일과 시간 확인
    // - BusinessHours와 비교
    // - 특별 휴일 처리
}

func (r *Restaurant) CanAcceptOrder(orderTime time.Time, orderAmount int64) bool {
    // 주문 수락 가능 여부를 검증합니다
    // - 매장 운영 상태 확인
    // - 운영시간 내 여부 확인
    // - 최소 주문 금액 확인
    // - 주문 수락 설정 확인
}

func (r *Restaurant) GetDecryptedInfo(decryptionService encryption.Service) (*RestaurantInfo, error) {
    // 암호화된 매장 정보를 복호화하여 반환합니다
    // - 필요한 권한 확인
    // - 개별 필드 복호화
    // - 감사 로그 기록
    // - 복호화 실패 시 에러 처리
}

// 도메인 이벤트 관리
func (r *Restaurant) DomainEvents() []DomainEvent {
    // 발생한 도메인 이벤트들을 반환합니다
}

func (r *Restaurant) ClearEvents() {
    // 처리 완료된 도메인 이벤트를 제거합니다
}

func (r *Restaurant) addEvent(event DomainEvent) {
    // 새로운 도메인 이벤트를 추가합니다
    // - 중복 이벤트 방지
    // - 이벤트 타임스탬프 설정
    // - 이벤트 순서 보장
}
```

### **🍽️ Menu Aggregate**

```go
// internal/restaurant/domain/menu.go
package domain

// Menu는 메뉴 항목을 나타내는 Aggregate Root입니다
type Menu struct {
    // 기본 식별자
    id           MenuID
    restaurantID RestaurantID
    
    // 암호화된 메뉴 정보
    name        encryption.EncryptedString // 메뉴명
    description encryption.EncryptedString // 메뉴 설명
    
    // 가격 정보 (계산 필요하므로 암호화하지 않음)
    basePrice    int64 // 기본 가격 (원 단위)
    discountRate int   // 할인율 (%)
    
    // 메뉴 속성
    category     MenuCategory
    tags         []MenuTag
    options      []MenuOption
    
    // 재고 및 상태
    isAvailable     bool
    dailyLimit      *int // nil이면 무제한
    soldCount       int  // 일일 판매량
    estimatedTime   time.Duration // 조리 예상 시간
    
    // 영양 정보 (선택적)
    nutritionInfo *NutritionInfo
    
    // 메타데이터
    sortOrder int
    createdAt time.Time
    updatedAt time.Time
    version   int64
    
    events []DomainEvent
}

type MenuID string

type MenuCategory string
const (
    MenuCategoryAppetizer MenuCategory = "APPETIZER" // 전채
    MenuCategoryMainDish  MenuCategory = "MAIN_DISH" // 메인
    MenuCategoryDessert   MenuCategory = "DESSERT"   // 디저트
    MenuCategoryBeverage  MenuCategory = "BEVERAGE"  // 음료
    MenuCategorySide      MenuCategory = "SIDE"      // 사이드
)

type MenuTag string
const (
    MenuTagSpicy      MenuTag = "SPICY"      // 매운맛
    MenuTagVegetarian MenuTag = "VEGETARIAN" // 채식
    MenuTagHalal      MenuTag = "HALAL"      // 할랄
    MenuTagPopular    MenuTag = "POPULAR"    // 인기메뉴
    MenuTagNew        MenuTag = "NEW"        // 신메뉴
    MenuTagSignature  MenuTag = "SIGNATURE"  // 대표메뉴
)

// MenuOption은 메뉴의 옵션을 나타냅니다 (맵기, 크기 등)
type MenuOption struct {
    ID          string           `json:"id"`
    Name        string           `json:"name"`        // 옵션명 (예: "맵기 선택")
    Type        MenuOptionType   `json:"type"`        // 선택 타입
    Required    bool             `json:"required"`    // 필수 선택 여부
    Choices     []MenuChoice     `json:"choices"`     // 선택 가능한 항목들
    MaxChoices  int              `json:"max_choices"` // 최대 선택 개수
}

type MenuOptionType string
const (
    MenuOptionTypeSingle   MenuOptionType = "SINGLE"   // 단일 선택
    MenuOptionTypeMultiple MenuOptionType = "MULTIPLE" // 다중 선택
)

type MenuChoice struct {
    ID    string `json:"id"`
    Name  string `json:"name"`  // 선택지명 (예: "보통맛", "매운맛")
    Price int64  `json:"price"` // 추가 가격 (0이면 무료)
}

type NutritionInfo struct {
    Calories     int     `json:"calories"`      // 칼로리
    Protein      float64 `json:"protein"`       // 단백질 (g)
    Carbs        float64 `json:"carbs"`         // 탄수화물 (g)
    Fat          float64 `json:"fat"`           // 지방 (g)
    Sodium       float64 `json:"sodium"`        // 나트륨 (mg)
    Allergens    []string `json:"allergens"`    // 알레르기 유발 요소
}

// Menu 생성자 및 팩토리 메서드
func NewMenu(
    restaurantID RestaurantID,
    name, description string,
    basePrice int64,
    category MenuCategory,
    encryptionService encryption.Service,
) (*Menu, error) {
    // 새로운 메뉴 아이템을 생성합니다
    // - 필수 필드 검증 (name, price > 0)
    // - 메뉴명과 설명 암호화
    // - 기본 설정값 초기화
    // - 도메인 이벤트 생성 (MenuCreated)
}

// Menu 도메인 메서드들
func (m *Menu) ID() MenuID {
    // 메뉴 ID를 반환합니다
}

func (m *Menu) RestaurantID() RestaurantID {
    // 소속 매장 ID를 반환합니다
}

func (m *Menu) UpdatePrice(newPrice int64, discountRate int) error {
    // 메뉴 가격을 업데이트합니다
    // - 가격 유효성 검증 (> 0)
    // - 할인율 유효성 검증 (0-100%)
    // - 가격 변경 이력 생성
    // - 도메인 이벤트 생성 (MenuPriceUpdated)
}

func (m *Menu) AddOption(option MenuOption) error {
    // 메뉴 옵션을 추가합니다
    // - 옵션 유효성 검증
    // - 중복 옵션 확인
    // - 최대 옵션 개수 제한 확인
    // - 도메인 이벤트 생성 (MenuOptionAdded)
}

func (m *Menu) RemoveOption(optionID string) error {
    // 메뉴 옵션을 제거합니다
    // - 기존 주문에 미치는 영향 확인
    // - 필수 옵션 제거 방지
    // - 도메인 이벤트 생성 (MenuOptionRemoved)
}

func (m *Menu) SetAvailability(available bool) error {
    // 메뉴 판매 가능 여부를 설정합니다
    // - 상태 변경 권한 확인
    // - 재고 확인 (available = true일 때)
    // - 도메인 이벤트 생성 (MenuAvailabilityChanged)
}

func (m *Menu) SetDailyLimit(limit *int) error {
    // 일일 판매 한도를 설정합니다
    // - 한도 유효성 검증 (nil 또는 > 0)
    // - 현재 판매량과 비교
    // - 한도 초과 시 자동 비활성화
    // - 도메인 이벤트 생성 (MenuDailyLimitSet)
}

func (m *Menu) IncrementSoldCount(quantity int) error {
    // 판매량을 증가시킵니다
    // - 일일 한도 확인
    // - 판매량 업데이트
    // - 한도 도달 시 자동 비활성화
    // - 도메인 이벤트 생성 (MenuSold)
}

func (m *Menu) ResetDailySoldCount() {
    // 일일 판매량을 초기화합니다 (매일 자정 실행)
    // - 판매량을 0으로 초기화
    // - 한도 내 메뉴 자동 활성화
    // - 도메인 이벤트 생성 (DailySalesReset)
}

func (m *Menu) CalculateTotalPrice(options []SelectedOption) int64 {
    // 선택된 옵션을 포함한 총 가격을 계산합니다
    // - 기본 가격에서 시작
    // - 각 옵션의 추가 가격 합산
    // - 할인율 적용
    // - 최종 가격 반환 (원 단위)
}

func (m *Menu) ValidateOptions(selectedOptions []SelectedOption) error {
    // 선택된 옵션의 유효성을 검증합니다
    // - 필수 옵션 선택 여부 확인
    // - 선택 가능한 옵션인지 확인
    // - 최대 선택 개수 제한 확인
    // - 존재하지 않는 옵션 선택 방지
}

func (m *Menu) IsAvailableForOrder() bool {
    // 주문 가능 여부를 확인합니다
    // - 메뉴 활성화 상태 확인
    // - 일일 판매 한도 확인
    // - 매장 운영시간 확인
    // - 재고 상태 확인
}

func (m *Menu) GetEstimatedPrepTime() time.Duration {
    // 예상 조리 시간을 반환합니다
    // - 기본 조리 시간 반환
    // - 복잡한 옵션에 따른 시간 추가 고려
    // - 현재 주문량에 따른 지연 시간 고려
}
```

### **🛠️ Restaurant Domain Services**

```go
// internal/restaurant/domain/services.go
package domain

// RestaurantDomainService는 매장 도메인의 복잡한 비즈니스 로직을 처리합니다
type RestaurantDomainService interface {
    // 매장 등록 시 사업자등록번호 중복 확인
    ValidateBusinessRegistration(ctx context.Context, businessNumber string) error
    
    // 메뉴 가격 정책 검증 (최소가격, 최대가격 등)
    ValidateMenuPricing(ctx context.Context, restaurantID RestaurantID, menu *Menu) error
    
    // 매장 운영시간과 현재 시간을 비교하여 주문 가능 여부 판단
    CanAcceptOrderAtTime(ctx context.Context, restaurant *Restaurant, orderTime time.Time) bool
    
    // 매장의 일일 매출 목표 대비 현재 성과 계산
    CalculateDailySalesPerformance(ctx context.Context, restaurantID RestaurantID, targetDate time.Time) (*SalesPerformance, error)
}

// MenuRecommendationService는 AI 기반 메뉴 추천 로직을 처리합니다
type MenuRecommendationService interface {
    // 시간대, 날씨, 고객 패턴을 고려한 메뉴 추천
    GetRecommendedMenus(ctx context.Context, restaurantID RestaurantID, criteria RecommendationCriteria) ([]MenuID, error)
    
    // 메뉴 인기도 기반 정렬
    SortMenusByPopularity(ctx context.Context, menus []Menu, timeRange TimeRange) ([]Menu, error)
    
    // 재고 예측 기반 메뉴 활성화/비활성화 추천
    GetInventoryBasedRecommendations(ctx context.Context, restaurantID RestaurantID) ([]MenuAvailabilityRecommendation, error)
}

// PricingService는 동적 가격 책정 로직을 처리합니다
type PricingService interface {
    // 수요와 공급을 고려한 동적 가격 제안
    SuggestDynamicPricing(ctx context.Context, menuID MenuID, demandData DemandAnalysis) (*PricingSuggestion, error)
    
    // 경쟁업체 가격 분석 기반 가격 추천
    AnalyzeCompetitorPricing(ctx context.Context, restaurantID RestaurantID, category MenuCategory) (*CompetitorAnalysis, error)
    
    // 할인 전략 최적화
    OptimizeDiscountStrategy(ctx context.Context, restaurantID RestaurantID, salesGoal SalesGoal) (*DiscountStrategy, error)
}
```

### **📅 Restaurant Repository Interface**

```go
// internal/restaurant/domain/repository.go
package domain

// RestaurantRepository는 매장 데이터 영속성을 담당합니다
type RestaurantRepository interface {
    // 매장 생성 (트랜잭션 내에서 실행)
    Create(ctx context.Context, restaurant *Restaurant) error
    
    // ID로 매장 조회 (캐시 우선)
    GetByID(ctx context.Context, id RestaurantID) (*Restaurant, error)
    
    // 소유자 ID로 매장 목록 조회
    GetByOwnerID(ctx context.Context, ownerID UserID) ([]*Restaurant, error)
    
    // 지리적 위치 기반 매장 검색 (반경 내)
    SearchByLocation(ctx context.Context, center GeographicLocation, radiusKm float64) ([]*Restaurant, error)
    
    // 매장 정보 업데이트 (낙관적 잠금 적용)
    Update(ctx context.Context, restaurant *Restaurant) error
    
    // 매장 삭제 (소프트 삭제)
    Delete(ctx context.Context, id RestaurantID) error
    
    // 사업자등록번호 중복 확인
    ExistsByBusinessNumber(ctx context.Context, businessNumber encryption.EncryptedString) (bool, error)
}

// MenuRepository는 메뉴 데이터 영속성을 담당합니다
type MenuRepository interface {
    // 메뉴 생성
    Create(ctx context.Context, menu *Menu) error
    
    // ID로 메뉴 조회
    GetByID(ctx context.Context, id MenuID) (*Menu, error)
    
    // 매장 ID로 메뉴 목록 조회 (정렬, 필터링 지원)
    GetByRestaurantID(ctx context.Context, restaurantID RestaurantID, filter MenuFilter) ([]*Menu, error)
    
    // 카테고리별 메뉴 조회
    GetByCategory(ctx context.Context, restaurantID RestaurantID, category MenuCategory) ([]*Menu, error)
    
    // 활성화된 메뉴만 조회 (고객용)
    GetAvailableMenus(ctx context.Context, restaurantID RestaurantID) ([]*Menu, error)
    
    // 메뉴 정보 업데이트
    Update(ctx context.Context, menu *Menu) error
    
    // 메뉴 삭제 (하드 삭제 - 주문 이력 확인 후)
    Delete(ctx context.Context, id MenuID) error
    
    // 일일 판매량 일괄 초기화
    ResetDailySoldCounts(ctx context.Context, restaurantID RestaurantID) error
    
    // 인기 메뉴 통계 조회
    GetPopularMenus(ctx context.Context, restaurantID RestaurantID, timeRange TimeRange, limit int) ([]*Menu, error)
}
```

-----

## 🛒 **Order Domain**

주문의 생성, 상태 관리, 생명주기를 담당하는 핵심 도메인입니다.

### **📋 Order Aggregate**

```go
// internal/order/domain/order.go
package domain

import (
    "context"
    "time"
    restaurantDomain "github.com/ordersync/internal/restaurant/domain"
    userDomain "github.com/ordersync/internal/user/domain"
    "github.com/ordersync/internal/encryption/domain"
)

// Order는 주문을 나타내는 Aggregate Root입니다
type Order struct {
    // 기본 식별자
    id           OrderID
    restaurantID restaurantDomain.RestaurantID
    customerID   *userDomain.UserID // nil이면 익명 주문
    
    // 주문 위치 정보 (암호화)
    tableInfo encryption.EncryptedString // 테이블 번호 또는 위치 정보
    
    // 주문 항목들 (암호화된 JSON)
    items encryption.EncryptedJSON // []OrderItem의 JSON
    
    // 가격 정보 (계산 필요하므로 암호화하지 않음)
    subtotal      int64 // 소계
    discountAmount int64 // 할인 금액
    taxAmount      int64 // 세금
    totalAmount    int64 // 최종 금액
    
    // 주문 상태 및 이력
    status        OrderStatus
    statusHistory []StatusChange // 상태 변경 이력
    
    // 특별 요청사항 (암호화)
    specialRequests encryption.EncryptedString
    
    // 시간 정보
    createdAt    time.Time
    acceptedAt   *time.Time // 매장에서 승인한 시간
    preparingAt  *time.Time // 조리 시작 시간
    readyAt      *time.Time // 픽업 준비 완료 시간
    completedAt  *time.Time // 주문 완료 시간
    cancelledAt  *time.Time // 취소 시간
    
    // 예상 시간
    estimatedReadyTime *time.Time // 예상 완료 시간
    
    // 익명 주문용 추적 정보
    anonymousToken *string // 익명 주문 추적 토큰
    
    // 도메인 메타데이터
    version int64 // 낙관적 잠금
    events  []DomainEvent
}

type OrderID string

type OrderStatus string
const (
    OrderStatusPending   OrderStatus = "PENDING"   // 주문 접수 대기
    OrderStatusAccepted  OrderStatus = "ACCEPTED"  // 매장 승인
    OrderStatusPreparing OrderStatus = "PREPARING" // 조리 중
    OrderStatusReady     OrderStatus = "READY"     // 픽업 가능
    OrderStatusCompleted OrderStatus = "COMPLETED" // 완료
    OrderStatusCancelled OrderStatus = "CANCELLED" // 취소됨
    OrderStatusRefunded  OrderStatus = "REFUNDED"  // 환불됨
)

// OrderItem은 주문에 포함된 메뉴 아이템을 나타냅니다
type OrderItem struct {
    MenuID      restaurantDomain.MenuID     `json:"menu_id"`
    MenuName    string                      `json:"menu_name"`    // 주문 시점 메뉴명 저장
    UnitPrice   int64                       `json:"unit_price"`   // 주문 시점 단가
    Quantity    int                         `json:"quantity"`
    Options     []SelectedMenuOption        `json:"options"`      // 선택된 옵션들
    Subtotal    int64                       `json:"subtotal"`     // 아이템별 소계
    
    // 조리 관련 정보
    EstimatedPrepTime time.Duration         `json:"estimated_prep_time"` // 예상 조리시간
    SpecialInstructions string             `json:"special_instructions"` // 특별 요청
}

// SelectedMenuOption은 고객이 선택한 메뉴 옵션입니다
type SelectedMenuOption struct {
    OptionID   string `json:"option_id"`
    OptionName string `json:"option_name"`
    ChoiceID   string `json:"choice_id"`
    ChoiceName string `json:"choice_name"`
    ExtraPrice int64  `json:"extra_price"` // 추가 가격
}

// StatusChange는 주문 상태 변경 이력을 기록합니다
type StatusChange struct {
    FromStatus  OrderStatus `json:"from_status"`
    ToStatus    OrderStatus `json:"to_status"`
    ChangedBy   *userDomain.UserID `json:"changed_by"`   // 상태 변경한 사용자
    Reason      string      `json:"reason"`              // 변경 사유
    Timestamp   time.Time   `json:"timestamp"`
    Message     string      `json:"message"`             // 고객용 메시지
}

// Order 생성자 및 팩토리 메서드
func NewOrder(
    restaurantID restaurantDomain.RestaurantID,
    customerID *userDomain.UserID,
    items []OrderItem,
    tableInfo string,
    specialRequests string,
    encryptionService encryption.Service,
) (*Order, error) {
    // 새로운 주문을 생성합니다
    // - 주문 항목 유효성 검증 (수량 > 0, 가격 > 0)
    // - 메뉴 존재 여부 및 활성화 상태 확인
    // - 총 금액 계산 및 검증
    // - 민감 정보 암호화 (tableInfo, specialRequests, items)
    // - 익명 주문 시 anonymousToken 생성
    // - 도메인 이벤트 생성 (OrderCreated)
}

func NewAnonymousOrder(
    restaurantID restaurantDomain.RestaurantID,
    items []OrderItem,
    tableInfo string,
    customerPhone string,
    encryptionService encryption.Service,
) (*Order, error) {
    // 익명 주문을 생성합니다
    // - 일반 주문 생성과 동일한 검증
    // - 익명 추적 토큰 생성 (UUID + 타임스탬프)
    // - 임시 고객 정보 암호화 저장
    // - 도메인 이벤트 생성 (AnonymousOrderCreated)
}

// Order 도메인 메서드들
func (o *Order) ID() OrderID {
    // 주문 ID를 반환합니다
}

func (o *Order) RestaurantID() restaurantDomain.RestaurantID {
    // 소속 매장 ID를 반환합니다
}

func (o *Order) CustomerID() *userDomain.UserID {
    // 고객 ID를 반환합니다 (익명 주문 시 nil)
}

func (o *Order) Status() OrderStatus {
    // 현재 주문 상태를 반환합니다
}

func (o *Order) Accept(acceptedBy userDomain.UserID, estimatedTime time.Duration) error {
    // 매장에서 주문을 승인합니다
    // - 현재 상태 검증 (PENDING -> ACCEPTED만 허용)
    // - 승인자 권한 확인 (매장 소유자 또는 직원)
    // - 예상 완료 시간 계산 및 설정
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (OrderAccepted)
    // - 고객 알림 준비
}

func (o *Order) Reject(rejectedBy userDomain.UserID, reason string) error {
    // 매장에서 주문을 거절합니다
    // - 현재 상태 검증 (PENDING -> CANCELLED만 허용)
    // - 거절자 권한 확인
    // - 거절 사유 기록 (필수)
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (OrderRejected)
    // - 자동 환불 처리 준비 (결제 완료 시)
}

func (o *Order) StartPreparing(startedBy userDomain.UserID) error {
    // 조리를 시작합니다
    // - 현재 상태 검증 (ACCEPTED -> PREPARING)
    // - 조리 시작자 권한 확인
    // - 조리 시작 시간 기록
    // - 예상 완료 시간 재계산
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (OrderPreparationStarted)
}

func (o *Order) MarkReady(completedBy userDomain.UserID, message string) error {
    // 주문 준비 완료를 표시합니다
    // - 현재 상태 검증 (PREPARING -> READY)
    // - 완료 처리자 권한 확인
    // - 실제 조리 시간 계산 및 기록
    // - 고객용 메시지 설정 (선택적)
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (OrderReady)
    // - 고객 픽업 알림 발송
}

func (o *Order) Complete(completedBy userDomain.UserID) error {
    // 주문을 완료 처리합니다 (픽업 또는 서빙 완료)
    // - 현재 상태 검증 (READY -> COMPLETED)
    // - 완료 처리자 권한 확인
    // - 완료 시간 기록
    // - 전체 주문 소요 시간 계산
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (OrderCompleted)
    // - 고객 만족도 조사 알림 준비
}

func (o *Order) Cancel(cancelledBy *userDomain.UserID, reason string) error {
    // 주문을 취소합니다 (고객 또는 매장에서 취소 가능)
    // - 취소 가능 상태 확인 (COMPLETED, CANCELLED 제외)
    // - 취소자 권한 확인 (고객 본인 또는 매장 관계자)
    // - 취소 사유 기록
    // - 취소 수수료 계산 (진행 상태에 따라)
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (OrderCancelled)
    // - 환불 처리 준비
}

func (o *Order) UpdateEstimatedTime(newEstimatedTime time.Duration, updatedBy userDomain.UserID) error {
    // 예상 완료 시간을 업데이트합니다
    // - 업데이트 권한 확인 (매장 관계자만)
    // - 현재 상태에서 시간 변경 가능 여부 확인
    // - 예상 시간 유효성 검증 (1분~180분)
    // - 예상 완료 시간 재계산
    // - 도메인 이벤트 생성 (EstimatedTimeUpdated)
    // - 고객 알림 발송 (시간 변경이 큰 경우)
}

func (o *Order) CalculateTotalAmount() int64 {
    // 주문의 총 금액을 계산합니다
    // - 모든 주문 항목의 소계 합산
    // - 할인 금액 차감
    // - 세금 계산 및 추가
    // - 배달비 추가 (배달 주문인 경우)
    // - 최종 금액 반환
}

func (o *Order) ValidateOrderItems(menuRepository restaurantDomain.MenuRepository) error {
    // 주문 항목의 유효성을 검증합니다
    // - 메뉴 존재 여부 확인
    // - 메뉴 활성화 상태 확인
    // - 선택된 옵션의 유효성 검증
    // - 가격 일치 여부 확인
    // - 재고 수량 확인 (일일 한도)
}

func (o *Order) GetDecryptedItems(decryptionService encryption.Service) ([]OrderItem, error) {
    // 암호화된 주문 항목을 복호화하여 반환합니다
    // - 접근 권한 확인 (고객 본인 또는 매장 관계자)
    // - 항목별 복호화 수행
    // - 복호화 실패 시 에러 처리
    // - 감사 로그 기록
}

func (o *Order) GetStatusHistory() []StatusChange {
    // 주문 상태 변경 이력을 반환합니다
    // - 시간순 정렬
    // - 민감 정보 마스킹 (필요시)
}

func (o *Order) IsAnonymous() bool {
    // 익명 주문 여부를 확인합니다
}

func (o *Order) GetAnonymousToken() *string {
    // 익명 주문 추적 토큰을 반환합니다
}

func (o *Order) CanBeCancelled() bool {
    // 주문 취소 가능 여부를 확인합니다
    // - 현재 상태 확인
    // - 조리 진행 상황 확인
    // - 취소 정책 적용
}

func (o *Order) GetElapsedTime() time.Duration {
    // 주문 후 경과 시간을 계산합니다
}

func (o *Order) GetRemainingEstimatedTime() *time.Duration {
    // 남은 예상 시간을 계산합니다
    // - 현재 시간과 예상 완료 시간 비교
    // - nil 반환 (예상 시간 미설정 시)
}

// 도메인 이벤트 관리
func (o *Order) DomainEvents() []DomainEvent {
    // 발생한 도메인 이벤트들을 반환합니다
}

func (o *Order) ClearEvents() {
    // 처리 완료된 도메인 이벤트를 제거합니다
}
```

### **🔔 Order Domain Services**

```go
// internal/order/domain/services.go
package domain

// OrderValidationService는 주문 생성/수정 시 복잡한 검증 로직을 처리합니다
type OrderValidationService interface {
    // 주문 생성 전 종합적인 유효성 검증
    ValidateNewOrder(ctx context.Context, order *Order, restaurant *restaurantDomain.Restaurant) error
    
    // 매장의 현재 상황을 고려한 주문 수락 가능 여부 판단
    CanAcceptOrder(ctx context.Context, restaurantID restaurantDomain.RestaurantID, orderTime time.Time) (*AcceptabilityResult, error)
    
    // 주문 수정 가능 여부 및 수정 범위 검증
    ValidateOrderModification(ctx context.Context, originalOrder *Order, modifications OrderModifications) error
    
    // 주문 취소 정책 적용 및 수수료 계산
    ValidateCancellation(ctx context.Context, order *Order, cancellationRequest CancellationRequest) (*CancellationResult, error)
}

// OrderPricingService는 주문 가격 계산 로직을 처리합니다
type OrderPricingService interface {
    // 주문 항목별 가격 계산 (옵션, 할인 포함)
    CalculateItemPricing(ctx context.Context, items []OrderItem, discounts []Discount) (*PricingBreakdown, error)
    
    // 동적 할인 적용 (시간대, 수량, 단골고객 등)
    ApplyDynamicDiscounts(ctx context.Context, order *Order, customerProfile *CustomerProfile) (*DiscountResult, error)
    
    // 세금 계산 (매장 위치별 세율 적용)
    CalculateTax(ctx context.Context, restaurantID restaurantDomain.RestaurantID, subtotal int64) (*TaxCalculation, error)
    
    // 배달비 계산 (거리, 날씨, 시간대 고려)
    CalculateDeliveryFee(ctx context.Context, restaurantLocation, deliveryLocation GeographicLocation, orderTime time.Time) (*DeliveryFeeCalculation, error)
}

// OrderStatusService는 주문 상태 전이 로직을 처리합니다
type OrderStatusService interface {
    // 상태 전이 가능 여부 검증
    ValidateStatusTransition(currentStatus OrderStatus, targetStatus OrderStatus, context StatusTransitionContext) error
    
    // 자동 상태 전이 처리 (시간 기반)
    ProcessAutomaticTransitions(ctx context.Context, orders []Order) ([]StatusTransitionResult, error)
    
    // 상태별 비즈니스 규칙 적용
    ApplyStatusBusinessRules(ctx context.Context, order *Order, newStatus OrderStatus) (*BusinessRuleResult, error)
    
    // 상태 변경 알림 대상자 결정
    DetermineNotificationRecipients(ctx context.Context, order *Order, statusChange StatusChange) ([]NotificationRecipient, error)
}

// OrderAnalyticsService는 주문 데이터 분석을 처리합니다
type OrderAnalyticsService interface {
    // 주문 패턴 분석 (시간대, 요일, 계절별)
    AnalyzeOrderPatterns(ctx context.Context, restaurantID restaurantDomain.RestaurantID, timeRange TimeRange) (*OrderPatternAnalysis, error)
    
    // 고객 주문 이력 분석
    AnalyzeCustomerOrderHistory(ctx context.Context, customerID userDomain.UserID) (*CustomerOrderAnalysis, error)
    
    // 주문 완료 시간 예측 모델링
    PredictOrderCompletionTime(ctx context.Context, order *Order, restaurantWorkload RestaurantWorkload) (*CompletionTimePrediction, error)
    
    // 주문 취소 위험도 평가
    AssessCancellationRisk(ctx context.Context, order *Order) (*CancellationRiskAssessment, error)
}
```

### **📊 Order Repository Interface**

```go
// internal/order/domain/repository.go
package domain

// OrderRepository는 주문 데이터 영속성을 담당합니다
type OrderRepository interface {
    // 주문 생성 (트랜잭션 내에서 실행)
    Create(ctx context.Context, order *Order) error
    
    // ID로 주문 조회
    GetByID(ctx context.Context, id OrderID) (*Order, error)
    
    // 익명 토큰으로 주문 조회
    GetByAnonymousToken(ctx context.Context, token string) (*Order, error)
    
    // 매장별 주문 목록 조회 (페이징, 필터링, 정렬 지원)
    GetByRestaurantID(ctx context.Context, restaurantID restaurantDomain.RestaurantID, filter OrderFilter) (*OrderList, error)
    
    // 고객별 주문 이력 조회
    GetByCustomerID(ctx context.Context, customerID userDomain.UserID, filter OrderFilter) (*OrderList, error)
    
    // 상태별 주문 조회 (배치 처리용)
    GetByStatus(ctx context.Context, status OrderStatus, limit int) ([]*Order, error)
    
    // 특정 시간 범위 내 주문 조회
    GetByTimeRange(ctx context.Context, startTime, endTime time.Time, filter OrderFilter) ([]*Order, error)
    
    // 주문 정보 업데이트 (낙관적 잠금 적용)
    Update(ctx context.Context, order *Order) error
    
    // 주문 상태 일괄 업데이트 (배치 처리용)
    UpdateStatusBatch(ctx context.Context, orderIDs []OrderID, newStatus OrderStatus, updatedBy userDomain.UserID) error
    
    // 주문 삭제 (소프트 삭제, 법적 보관 기간 고려)
    Delete(ctx context.Context, id OrderID) error
    
    // 매장별 실시간 주문 통계 조회
    GetRestaurantOrderStats(ctx context.Context, restaurantID restaurantDomain.RestaurantID, timeRange TimeRange) (*OrderStatistics, error)
    
    // 주문 검색 (고객명, 메뉴명, 주문번호 등)
    Search(ctx context.Context, criteria SearchCriteria) (*OrderSearchResult, error)
}

// OrderFilter는 주문 조회 시 사용하는 필터링 옵션입니다
type OrderFilter struct {
    Status      []OrderStatus `json:"status"`
    StartDate   *time.Time    `json:"start_date"`
    EndDate     *time.Time    `json:"end_date"`
    MinAmount   *int64        `json:"min_amount"`
    MaxAmount   *int64        `json:"max_amount"`
    IsAnonymous *bool         `json:"is_anonymous"`
    
    // 페이징
    Page    int `json:"page" validate:"min=1"`
    Limit   int `json:"limit" validate:"min=1,max=100"`
    
    // 정렬
    SortBy    string `json:"sort_by" validate:"oneof=created_at total_amount status"`
    SortOrder string `json:"sort_order" validate:"oneof=asc desc"`
}

// OrderList는 페이징된 주문 목록을 나타냅니다
type OrderList struct {
    Items      []*Order `json:"items"`
    TotalCount int64    `json:"total_count"`
    Page       int      `json:"page"`
    Limit      int      `json:"limit"`
    HasNext    bool     `json:"has_next"`
}
```

-----

## 💳 **Payment Domain**

결제 처리, 환불, 정산을 담당하는 보안이 중요한 도메인입니다.

### **💰 Payment Aggregate**

```go
// internal/payment/domain/payment.go
package domain

import (
    "context"
    "time"
    orderDomain "github.com/ordersync/internal/order/domain"
    "github.com/ordersync/internal/encryption/domain"
)

// Payment는 결제를 나타내는 Aggregate Root입니다
type Payment struct {
    // 기본 식별자
    id      PaymentID
    orderID orderDomain.OrderID
    
    // 결제 금액 정보 (암호화하지 않음 - 계산 필요)
    amount         int64 // 결제 금액
    fee            int64 // 결제 수수료
    netAmount      int64 // 실수령액 (amount - fee)
    
    // 결제 방법 및 제공업체 정보
    method         PaymentMethod
    provider       PaymentProvider // TOSS, KAKAO 등
    
    // 암호화된 외부 결제 정보
    externalPaymentID encryption.EncryptedString // 토스페이 결제 ID
    cardInfo         encryption.EncryptedJSON    // 마스킹된 카드 정보
    
    // 결제 상태 및 이력
    status         PaymentStatus
    statusHistory  []PaymentStatusChange
    
    // 결제 세부 정보 (암호화)
    paymentDetails encryption.EncryptedJSON // 결제 상세 메타데이터
    
    // 정산 정보
    settlementDate    *time.Time // 정산 예정일
    settlementStatus  SettlementStatus
    settlementID      *string    // 정산 배치 ID
    
    // 시간 정보
    createdAt     time.Time
    authorizedAt  *time.Time // 승인 시간
    capturedAt    *time.Time // 매입 시간
    cancelledAt   *time.Time // 취소 시간
    failedAt      *time.Time // 실패 시간
    
    // 실패 정보
    failureReason *string // 실패 사유
    
    // 도메인 메타데이터
    version int64 // 낙관적 잠금
    events  []DomainEvent
}

type PaymentID string

type PaymentMethod string
const (
    PaymentMethodCard     PaymentMethod = "CARD"     // 신용/체크카드
    PaymentMethodKakaoPay PaymentMethod = "KAKAOPAY" // 카카오페이
    PaymentMethodTossPay  PaymentMethod = "TOSSPAY"  // 토스페이
    PaymentMethodBank     PaymentMethod = "BANK"     // 계좌이체
    PaymentMethodCash     PaymentMethod = "CASH"     // 현금
)

type PaymentProvider string
const (
    PaymentProviderToss   PaymentProvider = "TOSS"   // 토스페이먼츠
    PaymentProviderKakao  PaymentProvider = "KAKAO"  // 카카오페이
    PaymentProviderNice   PaymentProvider = "NICE"   // 나이스페이
    PaymentProviderKCP    PaymentProvider = "KCP"    // KCP
)

type PaymentStatus string
const (
    PaymentStatusPending    PaymentStatus = "PENDING"    // 결제 대기
    PaymentStatusAuthorized PaymentStatus = "AUTHORIZED" // 승인됨
    PaymentStatusCaptured   PaymentStatus = "CAPTURED"   // 매입됨
    PaymentStatusFailed     PaymentStatus = "FAILED"     // 실패
    PaymentStatusCancelled  PaymentStatus = "CANCELLED"  // 취소됨
    PaymentStatusRefunded   PaymentStatus = "REFUNDED"   // 환불됨
)

type SettlementStatus string
const (
    SettlementStatusPending   SettlementStatus = "PENDING"   // 정산 대기
    SettlementStatusProcessed SettlementStatus = "PROCESSED" // 정산 완료
    SettlementStatusFailed    SettlementStatus = "FAILED"    // 정산 실패
)

// PaymentStatusChange는 결제 상태 변경 이력을 기록합니다
type PaymentStatusChange struct {
    FromStatus PaymentStatus `json:"from_status"`
    ToStatus   PaymentStatus `json:"to_status"`
    Timestamp  time.Time     `json:"timestamp"`
    Reason     string        `json:"reason"`
    ExternalID *string       `json:"external_id"` // 외부 시스템 참조 ID
}

// MaskedCardInfo는 마스킹된 카드 정보입니다 (PCI DSS 준수)
type MaskedCardInfo struct {
    MaskedNumber string `json:"masked_number"` // **** **** **** 1234
    Brand        string `json:"brand"`         // VISA, MASTER, AMEX 등
    ExpiryMonth  string `json:"expiry_month"`  // MM
    ExpiryYear   string `json:"expiry_year"`   // YY
    IssuerName   string `json:"issuer_name"`   // 발급사명
    CardType     string `json:"card_type"`     // CREDIT, CHECK
}

// PaymentDetails는 결제 세부 정보입니다
type PaymentDetails struct {
    Currency          string            `json:"currency"`           // KRW
    PaymentMethodFlow string            `json:"payment_method_flow"` // DIRECT, REDIRECT
    CustomerIP        string            `json:"customer_ip"`        // 고객 IP (암호화됨)
    UserAgent         string            `json:"user_agent"`         // 브라우저 정보
    Metadata          map[string]string `json:"metadata"`           // 추가 메타데이터
}

// Payment 생성자 및 팩토리 메서드
func NewPayment(
    orderID orderDomain.OrderID,
    amount int64,
    method PaymentMethod,
    provider PaymentProvider,
    encryptionService encryption.Service,
) (*Payment, error) {
    // 새로운 결제를 생성합니다
    // - 금액 유효성 검증 (> 0)
    // - 결제 방법과 제공업체 조합 검증
    // - 결제 수수료 계산
    // - 실수령액 계산 (amount - fee)
    // - 도메인 이벤트 생성 (PaymentCreated)
}

// Payment 도메인 메서드들
func (p *Payment) ID() PaymentID {
    // 결제 ID를 반환합니다
}

func (p *Payment) OrderID() orderDomain.OrderID {
    // 연관된 주문 ID를 반환합니다
}

func (p *Payment) Amount() int64 {
    // 결제 금액을 반환합니다
}

func (p *Payment) Status() PaymentStatus {
    // 현재 결제 상태를 반환합니다
}

func (p *Payment) Authorize(
    externalPaymentID string,
    cardInfo MaskedCardInfo,
    paymentDetails PaymentDetails,
    encryptionService encryption.Service,
) error {
    // 결제를 승인합니다
    // - 현재 상태 검증 (PENDING -> AUTHORIZED)
    // - 외부 결제 ID 및 카드 정보 암호화 저장
    // - 승인 시간 기록
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (PaymentAuthorized)
}

func (p *Payment) Capture(capturedAmount *int64) error {
    // 결제를 매입합니다 (실제 돈 이동)
    // - 현재 상태 검증 (AUTHORIZED -> CAPTURED)
    // - 매입 금액 검증 (원래 금액 이하)
    // - 부분 매입 처리 (전체 금액과 다른 경우)
    // - 매입 시간 기록
    // - 정산 스케줄 생성
    // - 도메인 이벤트 생성 (PaymentCaptured)
}

func (p *Payment) Fail(reason string) error {
    // 결제를 실패 처리합니다
    // - 실패 가능한 상태 확인 (PENDING, AUTHORIZED)
    // - 실패 사유 기록
    // - 실패 시간 기록
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (PaymentFailed)
    // - 주문 취소 프로세스 시작
}

func (p *Payment) Cancel(reason string, cancellationFee int64) error {
    // 결제를 취소합니다
    // - 취소 가능한 상태 확인
    // - 취소 수수료 계산 및 적용
    // - 환불 예정 금액 계산
    // - 취소 시간 기록
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (PaymentCancelled)
}

func (p *Payment) InitiateRefund(refundAmount int64, reason string) (*Refund, error) {
    // 환불을 시작합니다
    // - 환불 가능한 상태 확인 (CAPTURED, REFUNDED)
    // - 환불 금액 검증 (남은 금액 이하)
    // - 환불 수수료 계산
    // - 부분 환불인지 전체 환불인지 확인
    // - 환불 객체 생성 및 연결
    // - 상태를 REFUNDED로 변경 (전체 환불 시)
    // - 도메인 이벤트 생성 (RefundInitiated)
}

func (p *Payment) CalculateFee() int64 {
    // 결제 수수료를 계산합니다
    // - 결제 방법별 수수료율 적용
    // - 결제 제공업체별 수수료 구조 고려
    // - 최소/최대 수수료 한도 적용
    // - 할인 혜택 적용 (대량 거래, VIP 등)
}

func (p *Payment) GetNetAmount() int64 {
    // 실수령액을 계산합니다 (결제금액 - 수수료)
    // - 현재 결제 금액에서 수수료 차감
    // - 환불된 금액 고려
    // - 취소 수수료 차감
}

func (p *Payment) GetDecryptedCardInfo(decryptionService encryption.Service) (*MaskedCardInfo, error) {
    // 암호화된 카드 정보를 복호화하여 반환합니다
    // - 접근 권한 확인 (매장 소유자, 관리자만)
    // - 카드 정보 복호화
    // - 감사 로그 기록 (PCI DSS 요구사항)
    // - 마스킹된 정보만 반환 (보안 유지)
}

func (p *Payment) CanBeRefunded() bool {
    // 환불 가능 여부를 확인합니다
    // - 현재 상태 확인 (CAPTURED 상태만 환불 가능)
    // - 환불 정책 확인 (시간 제한, 사유 제한 등)
    // - 이미 환불된 금액 고려
    // - 분쟁 상태 확인
}

func (p *Payment) GetRefundableAmount() int64 {
    // 환불 가능한 금액을 계산합니다
    // - 원래 결제 금액에서 이미 환불된 금액 차감
    // - 환불 수수료 고려
    // - 최소 환불 금액 정책 적용
}

func (p *Payment) IsSettled() bool {
    // 정산 완료 여부를 확인합니다
    // - 정산 상태 확인
    // - 정산 완료 시간 확인
}

func (p *Payment) GetSettlementInfo() *SettlementInfo {
    // 정산 정보를 반환합니다
    // - 정산 예정일, 정산 금액, 정산 수수료 등
    // - 정산 지연 사유 (있는 경우)
}

// 도메인 이벤트 관리
func (p *Payment) DomainEvents() []DomainEvent {
    // 발생한 도메인 이벤트들을 반환합니다
}

func (p *Payment) ClearEvents() {
    // 처리 완료된 도메인 이벤트를 제거합니다
}
```

### **💸 Refund Aggregate**

```go
// internal/payment/domain/refund.go
package domain

// Refund는 환불을 나타내는 Aggregate Root입니다
type Refund struct {
    // 기본 식별자
    id        RefundID
    paymentID PaymentID
    
    // 환불 금액 정보
    refundAmount    int64 // 환불 요청 금액
    refundFee       int64 // 환불 수수료
    netRefundAmount int64 // 실제 환불 금액 (refundAmount - refundFee)
    
    // 환불 사유 (암호화)
    reason encryption.EncryptedString
    
    // 환불 상태 및 이력
    status        RefundStatus
    statusHistory []RefundStatusChange
    
    // 외부 환불 정보 (암호화)
    externalRefundID encryption.EncryptedString // 토스페이 환불 ID
    
    // 환불 처리 정보
    processedAt *time.Time // 환불 처리 완료 시간
    failureReason *string  // 환불 실패 사유
    
    // 환불 세부 정보 (암호화)
    refundDetails encryption.EncryptedJSON
    
    // 시간 정보
    createdAt time.Time
    
    // 도메인 메타데이터
    version int64
    events  []DomainEvent
}

type RefundID string

type RefundStatus string
const (
    RefundStatusPending   RefundStatus = "PENDING"   // 환불 요청됨
    RefundStatusProcessed RefundStatus = "PROCESSED" // 환불 처리됨
    RefundStatusFailed    RefundStatus = "FAILED"    // 환불 실패
    RefundStatusCancelled RefundStatus = "CANCELLED" // 환불 취소됨
)

type RefundStatusChange struct {
    FromStatus RefundStatus `json:"from_status"`
    ToStatus   RefundStatus `json:"to_status"`
    Timestamp  time.Time    `json:"timestamp"`
    Reason     string       `json:"reason"`
    ExternalID *string      `json:"external_id"`
}

type RefundDetails struct {
    RefundMethod     string            `json:"refund_method"`      // 환불 방법
    ExpectedDays     int               `json:"expected_days"`      // 예상 소요일
    BankAccountInfo  *BankAccountInfo  `json:"bank_account_info"`  // 계좌 환불 시
    Metadata         map[string]string `json:"metadata"`
}

type BankAccountInfo struct {
    BankCode    string `json:"bank_code"`
    AccountNumber string `json:"account_number"` // 마스킹된 계좌번호
    HolderName  string `json:"holder_name"`     // 예금주명
}

// Refund 생성자 및 팩토리 메서드
func NewRefund(
    paymentID PaymentID,
    refundAmount int64,
    reason string,
    encryptionService encryption.Service,
) (*Refund, error) {
    // 새로운 환불을 생성합니다
    // - 환불 금액 유효성 검증
    // - 환불 사유 암호화
    // - 환불 수수료 계산
    // - 실제 환불 금액 계산
    // - 도메인 이벤트 생성 (RefundCreated)
}

// Refund 도메인 메서드들
func (r *Refund) ID() RefundID {
    // 환불 ID를 반환합니다
}

func (r *Refund) PaymentID() PaymentID {
    // 연관된 결제 ID를 반환합니다
}

func (r *Refund) Process(externalRefundID string, refundDetails RefundDetails) error {
    // 환불을 처리합니다
    // - 현재 상태 검증 (PENDING -> PROCESSED)
    // - 외부 환불 ID 암호화 저장
    // - 환불 세부정보 암호화 저장
    // - 처리 완료 시간 기록
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (RefundProcessed)
}

func (r *Refund) Fail(failureReason string) error {
    // 환불을 실패 처리합니다
    // - 현재 상태 검증 (PENDING -> FAILED)
    // - 실패 사유 기록
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (RefundFailed)
    // - 재시도 스케줄링 검토
}

func (r *Refund) Cancel(cancellationReason string) error {
    // 환불을 취소합니다
    // - 취소 가능한 상태 확인 (PENDING만)
    // - 취소 사유 기록
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (RefundCancelled)
}

func (r *Refund) GetNetRefundAmount() int64 {
    // 실제 환불 금액을 반환합니다
    // - 환불 수수료를 차감한 금액
}

func (r *Refund) IsCompleted() bool {
    // 환불 완료 여부를 확인합니다
    // - PROCESSED 상태인지 확인
}

func (r *Refund) GetEstimatedCompletionDate() *time.Time {
    // 예상 환불 완료일을 계산합니다
    // - 환불 방법별 처리 소요일 고려
    // - 주말, 공휴일 제외
    // - 은행 업무 시간 고려
}
```

### **💳 Payment Domain Services**

```go
// internal/payment/domain/services.go
package domain

// PaymentProcessingService는 결제 처리의 복잡한 비즈니스 로직을 담당합니다
type PaymentProcessingService interface {
    // 결제 방법별 수수료 계산
    CalculateProcessingFee(ctx context.Context, amount int64, method PaymentMethod, provider PaymentProvider) (*FeeCalculation, error)
    
    // 결제 제공업체별 한도 확인
    ValidatePaymentLimits(ctx context.Context, amount int64, method PaymentMethod, customerID *userDomain.UserID) error
    
    // 결제 위험도 평가 (사기 탐지)
    AssessFraudRisk(ctx context.Context, paymentRequest PaymentRequest) (*FraudAssessment, error)
    
    // 결제 재시도 로직 (네트워크 오류, 일시적 장애 시)
    ShouldRetryPayment(ctx context.Context, payment *Payment, error PaymentError) (*RetryDecision, error)
}

// PaymentReconciliationService는 결제 대사 및 정산 로직을 처리합니다
type PaymentReconciliationService interface {
    // 일일 결제 대사 수행
    PerformDailyReconciliation(ctx context.Context, date time.Time) (*ReconciliationResult, error)
    
    // 결제 제공업체와 데이터 일치성 확인
    ValidatePaymentConsistency(ctx context.Context, payments []Payment) (*ConsistencyReport, error)
    
    // 정산 대상 결제 선별 및 처리
    ProcessSettlement(ctx context.Context, settlementDate time.Time) (*SettlementResult, error)
    
    // 미정산 결제 추적 및 알림
    TrackPendingSettlements(ctx context.Context) (*PendingSettlementsReport, error)
}

// PaymentAnalyticsService는 결제 데이터 분석을 처리합니다
type PaymentAnalyticsService interface {
    // 결제 성공률 분석 (방법별, 시간대별)
    AnalyzePaymentSuccessRate(ctx context.Context, criteria AnalyticsCriteria) (*PaymentAnalytics, error)
    
    // 결제 수수료 최적화 분석
    OptimizePaymentFees(ctx context.Context, restaurantID restaurantDomain.RestaurantID) (*FeeOptimizationSuggestion, error)
    
    // 고객 결제 패턴 분석
    AnalyzeCustomerPaymentPatterns(ctx context.Context, customerID userDomain.UserID) (*PaymentPatternAnalysis, error)
    
    // 결제 실패 원인 분석
    AnalyzePaymentFailures(ctx context.Context, timeRange TimeRange) (*FailureAnalysis, error)
}

// RefundPolicyService는 환불 정책 관련 비즈니스 로직을 처리합니다
type RefundPolicyService interface {
    // 환불 정책 적용 가능성 검증
    ValidateRefundEligibility(ctx context.Context, payment *Payment, refundRequest RefundRequest) (*RefundEligibility, error)
    
    // 환불 수수료 계산 (정책별, 시간별)
    CalculateRefundFee(ctx context.Context, payment *Payment, refundAmount int64, refundReason string) (*RefundFeeCalculation, error)
    
    // 자동 환불 가능 여부 판단
    CanProcessAutomaticRefund(ctx context.Context, payment *Payment, refundRequest RefundRequest) (bool, error)
    
    // 환불 승인 프로세스 결정
    DetermineRefundApprovalProcess(ctx context.Context, refund *Refund) (*ApprovalProcess, error)
}
```

### **📊 Payment Repository Interface**

```go
// internal/payment/domain/repository.go
package domain

// PaymentRepository는 결제 데이터 영속성을 담당합니다
type PaymentRepository interface {
    // 결제 생성
    Create(ctx context.Context, payment *Payment) error
    
    // ID로 결제 조회
    GetByID(ctx context.Context, id PaymentID) (*Payment, error)
    
    // 주문 ID로 결제 조회
    GetByOrderID(ctx context.Context, orderID orderDomain.OrderID) (*Payment, error)
    
    // 외부 결제 ID로 조회 (콜백 처리용)
    GetByExternalPaymentID(ctx context.Context, externalID encryption.EncryptedString) (*Payment, error)
    
    // 상태별 결제 조회 (배치 처리용)
    GetByStatus(ctx context.Context, status PaymentStatus, limit int) ([]*Payment, error)
    
    // 정산 대상 결제 조회
    GetSettlementTargets(ctx context.Context, settlementDate time.Time) ([]*Payment, error)
    
    // 결제 정보 업데이트 (낙관적 잠금 적용)
    Update(ctx context.Context, payment *Payment) error
    
    // 결제 상태 일괄 업데이트
    UpdateStatusBatch(ctx context.Context, paymentIDs []PaymentID, newStatus PaymentStatus) error
    
    // 결제 삭제 (법적 보관 기간 후)
    Delete(ctx context.Context, id PaymentID) error
    
    // 매장별 결제 통계 조회
    GetRestaurantPaymentStats(ctx context.Context, restaurantID restaurantDomain.RestaurantID, timeRange TimeRange) (*PaymentStatistics, error)
    
    // 결제 실패 이력 조회
    GetFailureHistory(ctx context.Context, criteria FailureCriteria) ([]*Payment, error)
}

// RefundRepository는 환불 데이터 영속성을 담당합니다
type RefundRepository interface {
    // 환불 생성
    Create(ctx context.Context, refund *Refund) error
    
    // ID로 환불 조회
    GetByID(ctx context.Context, id RefundID) (*Refund, error)
    
    // 결제 ID로 환불 목록 조회
    GetByPaymentID(ctx context.Context, paymentID PaymentID) ([]*Refund, error)
    
    // 상태별 환불 조회
    GetByStatus(ctx context.Context, status RefundStatus, limit int) ([]*Refund, error)
    
    // 처리 대기 중인 환불 조회
    GetPendingRefunds(ctx context.Context) ([]*Refund, error)
    
    // 환불 정보 업데이트
    Update(ctx context.Context, refund *Refund) error
    
    // 환불 삭제
    Delete(ctx context.Context, id RefundID) error
    
    // 환불 통계 조회
    GetRefundStatistics(ctx context.Context, timeRange TimeRange) (*RefundStatistics, error)
}
```

-----

## 👤 **User Domain**

사용자 인증, 권한 관리, 세션 관리를 담당하는 보안 핵심 도메인입니다.

### **🔐 User Aggregate**

```go
// internal/user/domain/user.go
package domain

import (
    "context"
    "time"
    "github.com/ordersync/internal/encryption/domain"
)

// User는 시스템 사용자를 나타내는 Aggregate Root입니다
type User struct {
    // 기본 식별자
    id UserID
    
    // 암호화된 개인정보 (PII)
    email       encryption.EncryptedString // 이메일 주소
    phoneNumber encryption.EncryptedString // 전화번호
    name        encryption.EncryptedString // 실명
    
    // 인증 정보 (암호화)
    passwordHash encryption.EncryptedString // bcrypt 해시
    salt         string                     // 솔트값 (암호화하지 않음)
    
    // 권한 및 역할
    role        UserRole        // 주 역할
    permissions []Permission    // 세부 권한
    
    // 계정 상태
    status           UserStatus
    emailVerified    bool
    phoneVerified    bool
    twoFactorEnabled bool
    
    // 보안 정보
    lastLoginAt       *time.Time
    passwordChangedAt time.Time
    failedLoginCount  int
    lockedUntil       *time.Time // 계정 잠금 해제 시간
    
    // 개인 설정
    preferences UserPreferences
    
    // 프로필 정보 (암호화)
    profileInfo encryption.EncryptedJSON // 추가 프로필 정보
    
    // 도메인 메타데이터
    createdAt time.Time
    updatedAt time.Time
    version   int64 // 낙관적 잠금
    events    []DomainEvent
}

type UserID string

type UserRole string
const (
    UserRoleCustomer        UserRole = "CUSTOMER"         // 일반 고객
    UserRoleRestaurantOwner UserRole = "RESTAURANT_OWNER" // 매장 사장님
    UserRoleStaff          UserRole = "STAFF"            // 매장 직원
    UserRoleAdmin          UserRole = "ADMIN"            // 시스템 관리자
    UserRoleSuperAdmin     UserRole = "SUPER_ADMIN"      // 최고 관리자
)

type Permission string
const (
    // 매장 권한
    PermissionRestaurantRead   Permission = "restaurant:read"
    PermissionRestaurantWrite  Permission = "restaurant:write"
    PermissionRestaurantDelete Permission = "restaurant:delete"
    
    // 주문 권한  
    PermissionOrderRead   Permission = "order:read"
    PermissionOrderWrite  Permission = "order:write"
    PermissionOrderCancel Permission = "order:cancel"
    
    // 결제 권한
    PermissionPaymentRead   Permission = "payment:read"
    PermissionPaymentRefund Permission = "payment:refund"
    
    // 메뉴 권한
    PermissionMenuRead   Permission = "menu:read"
    PermissionMenuWrite  Permission = "menu:write"
    PermissionMenuDelete Permission = "menu:delete"
    
    // 분석 권한
    PermissionAnalyticsRead Permission = "analytics:read"
    
    // 관리 권한
    PermissionUserManage   Permission = "user:manage"
    PermissionSystemConfig Permission = "system:config"
)

type UserStatus string
const (
    UserStatusActive     UserStatus = "ACTIVE"     // 활성 상태
    UserStatusInactive   UserStatus = "INACTIVE"   // 비활성 상태  
    UserStatusSuspended  UserStatus = "SUSPENDED"  // 정지 상태
    UserStatusDeleted    UserStatus = "DELETED"    // 삭제 상태
)

type UserPreferences struct {
    Language             string                    `json:"language"`              // ko, en
    Timezone             string                    `json:"timezone"`              // Asia/Seoul
    NotificationSettings NotificationPreferences  `json:"notification_settings"`
    PrivacySettings      PrivacySettings          `json:"privacy_settings"`
    DisplaySettings      DisplaySettings          `json:"display_settings"`
}

type NotificationPreferences struct {
    OrderNotifications    bool `json:"order_notifications"`     // 주문 알림
    PaymentNotifications  bool `json:"payment_notifications"`   // 결제 알림
    MarketingNotifications bool `json:"marketing_notifications"` // 마케팅 알림
    EmailEnabled          bool `json:"email_enabled"`           // 이메일 알림
    SMSEnabled            bool `json:"sms_enabled"`             // SMS 알림
    PushEnabled           bool `json:"push_enabled"`            // 푸시 알림
}

type PrivacySettings struct {
    ProfileVisibility string `json:"profile_visibility"` // public, private
    DataSharing       bool   `json:"data_sharing"`        // 데이터 활용 동의
    LocationTracking  bool   `json:"location_tracking"`   // 위치 추적 동의
}

type DisplaySettings struct {
    Theme      string `json:"theme"`       // light, dark
    FontSize   string `json:"font_size"`   // small, medium, large
    Currency   string `json:"currency"`    // KRW, USD
    DateFormat string `json:"date_format"` // YYYY-MM-DD, MM/DD/YYYY
}

// User 생성자 및 팩토리 메서드
func NewUser(
    email, phoneNumber, name, password string,
    role UserRole,
    encryptionService encryption.Service,
) (*User, error) {
    // 새로운 사용자를 생성합니다
    // - 이메일 형식 유효성 검증
    // - 전화번호 형식 검증 (국제 형식)
    // - 비밀번호 강도 검증 (최소 8자, 대소문자, 숫자, 특수문자)
    // - 개인정보 암호화 (email, phoneNumber, name)
    // - 비밀번호 해시 생성 및 암호화
    // - 기본 권한 설정 (역할에 따라)
    // - 기본 설정값 초기화
    // - 도메인 이벤트 생성 (UserCreated)
}

func NewCustomer(email, phoneNumber, name, password string, encryptionService encryption.Service) (*User, error) {
    // 고객 사용자를 생성합니다
    // - 고객 역할로 사용자 생성
    // - 기본 고객 권한 설정
    // - 마케팅 동의 기본값 설정
}

func NewRestaurantOwner(email, phoneNumber, name, password string, encryptionService encryption.Service) (*User, error) {
    // 매장 소유자를 생성합니다  
    // - 매장 소유자 역할로 사용자 생성
    // - 매장 관리 권한 설정
    // - 사업자 인증 필요 상태로 설정
}

// User 도메인 메서드들
func (u *User) ID() UserID {
    // 사용자 ID를 반환합니다
}

func (u *User) Role() UserRole {
    // 사용자 역할을 반환합니다
}

func (u *User) Status() UserStatus {
    // 계정 상태를 반환합니다
}

func (u *User) ChangePassword(oldPassword, newPassword string, encryptionService encryption.Service) error {
    // 비밀번호를 변경합니다
    // - 기존 비밀번호 검증
    // - 새 비밀번호 강도 검증
    // - 기존 비밀번호와 다른지 확인
    // - 새 비밀번호 해시 생성 및 암호화
    // - 비밀번호 변경 시간 업데이트
    // - 모든 세션 무효화 (보안)
    // - 도메인 이벤트 생성 (PasswordChanged)
}

func (u *User) VerifyEmail(verificationToken string) error {
    // 이메일 인증을 처리합니다
    // - 인증 토큰 유효성 검증
    // - 이메일 인증 상태 업데이트
    // - 계정 활성화 (필요시)
    // - 도메인 이벤트 생성 (EmailVerified)
}

func (u *User) VerifyPhoneNumber(verificationCode string) error {
    // 전화번호 인증을 처리합니다
    // - 인증 코드 유효성 검증
    // - 전화번호 인증 상태 업데이트
    // - 도메인 이벤트 생성 (PhoneVerified)
}

func (u *User) EnableTwoFactor(secretKey string) error {
    // 2단계 인증을 활성화합니다
    // - 시크릿 키 검증
    // - 2FA 활성화 상태 업데이트
    // - 백업 코드 생성 (암호화 저장)
    // - 도메인 이벤트 생성 (TwoFactorEnabled)
}

func (u *User) DisableTwoFactor() error {
    // 2단계 인증을 비활성화합니다
    // - 현재 2FA 활성화 상태 확인
    // - 2FA 비활성화 처리
    // - 백업 코드 삭제
    // - 도메인 이벤트 생성 (TwoFactorDisabled)
}

func (u *User) RecordLoginAttempt(success bool, ipAddress, userAgent string) error {
    // 로그인 시도를 기록합니다
    // - 성공 시: 로그인 시간 업데이트, 실패 카운트 초기화
    // - 실패 시: 실패 카운트 증가, 계정 잠금 검토
    // - IP 주소와 User-Agent 기록 (보안 분석용)
    // - 도메인 이벤트 생성 (LoginAttempt)
}

func (u *User) LockAccount(duration time.Duration, reason string) error {
    // 계정을 잠금 처리합니다
    // - 잠금 해제 시간 계산 및 설정
    // - 계정 상태 업데이트
    // - 모든 활성 세션 무효화
    // - 잠금 사유 기록
    // - 도메인 이벤트 생성 (AccountLocked)
}

func (u *User) UnlockAccount() error {
    // 계정 잠금을 해제합니다
    // - 잠금 상태 확인
    // - 잠금 해제 시간 확인
    // - 실패 카운트 초기화
    // - 계정 상태 정상화
    // - 도메인 이벤트 생성 (AccountUnlocked)
}

func (u *User) UpdatePermissions(newPermissions []Permission, updatedBy UserID) error {
    // 사용자 권한을 업데이트합니다
    // - 권한 변경 권한 확인 (관리자만)
    // - 권한 유효성 검증
    // - 역할과 권한의 일관성 확인
    // - 권한 변경 이력 기록
    // - 도메인 이벤트 생성 (PermissionsUpdated)
}

func (u *User) UpdatePreferences(newPreferences UserPreferences) error {
    // 사용자 설정을 업데이트합니다
    // - 설정값 유효성 검증
    // - 언어/시간대 지원 여부 확인
    // - 알림 설정 유효성 검증
    // - 도메인 이벤트 생성 (PreferencesUpdated)
}

func (u *User) Deactivate(reason string) error {
    // 계정을 비활성화합니다
    // - 현재 상태 확인 (ACTIVE -> INACTIVE)
    // - 비활성화 사유 기록
    // - 모든 활성 세션 무효화
    // - 관련된 매장/주문 처리 상태 확인
    // - 개인정보 익명화 스케줄링
    // - 도메인 이벤트 생성 (UserDeactivated)
}

func (u *User) Delete(deletionReason string) error {
    // 계정을 삭제합니다 (GDPR 준수)
    // - 삭제 가능 상태 확인
    // - 진행 중인 주문/결제 확인
    // - 법적 보관 의무 데이터 식별
    // - 개인정보 완전 삭제 스케줄링
    // - 상태를 DELETED로 변경
    // - 도메인 이벤트 생성 (UserDeleted)
}

func (u *User) HasPermission(permission Permission) bool {
    // 특정 권한 보유 여부를 확인합니다
    // - 직접 부여된 권한 확인
    // - 역할 기반 기본 권한 확인
    // - 상속된 권한 확인
}

func (u *User) CanAccessResource(resourceType string, resourceID string, action string) bool {
    // 특정 리소스에 대한 접근 권한을 확인합니다
    // - 리소스별 접근 정책 적용
    // - 소유권 기반 접근 제어
    // - 시간 기반 접근 제한 확인
    // - 지역 기반 접근 제한 확인
}

func (u *User) IsAccountLocked() bool {
    // 계정 잠금 상태를 확인합니다
    // - 잠금 해제 시간과 현재 시간 비교
    // - 자동 잠금 해제 처리
}

func (u *User) GetDecryptedEmail(decryptionService encryption.Service) (string, error) {
    // 암호화된 이메일을 복호화하여 반환합니다
    // - 접근 권한 확인 (본인 또는 관리자)
    // - 이메일 복호화
    // - 감사 로그 기록
}

func (u *User) GetDecryptedPhoneNumber(decryptionService encryption.Service) (string, error) {
    // 암호화된 전화번호를 복호화하여 반환합니다
    // - 접근 권한 확인
    // - 전화번호 복호화
    // - 감사 로그 기록
}

func (u *User) GetDecryptedName(decryptionService encryption.Service) (string, error) {
    // 암호화된 이름을 복호화하여 반환합니다
    // - 접근 권한 확인
    // - 이름 복호화
    // - 감사 로그 기록
}

func (u *User) ValidatePassword(inputPassword string, decryptionService encryption.Service) (bool, error) {
    // 입력된 비밀번호를 검증합니다
    // - 저장된 해시 복호화
    // - bcrypt 비교 수행
    // - 검증 결과 반환 (성공/실패만)
    // - 감사 로그 기록
}

// 도메인 이벤트 관리
func (u *User) DomainEvents() []DomainEvent {
    // 발생한 도메인 이벤트들을 반환합니다
}

func (u *User) ClearEvents() {
    // 처리 완료된 도메인 이벤트를 제거합니다
}
```

### **🎫 Session Aggregate**

```go
// internal/user/domain/session.go
package domain

// Session은 사용자 세션을 나타내는 Aggregate Root입니다
type Session struct {
    // 기본 식별자
    id     SessionID
    userID UserID
    
    // 토큰 정보 (해시값만 저장)
    accessTokenHash  string // JWT 액세스 토큰 해시
    refreshTokenHash string // 리프레시 토큰 해시
    
    // 세션 메타데이터 (암호화)
    ipAddress     encryption.EncryptedString // 접속 IP
    userAgent     encryption.EncryptedString // 브라우저/앱 정보
    deviceInfo    encryption.EncryptedJSON   // 디바이스 정보
    
    // 세션 상태
    status        SessionStatus
    isActive      bool
    lastActivity  time.Time
    
    // 만료 정보
    accessExpiresAt  time.Time
    refreshExpiresAt time.Time
    
    // 보안 정보
    loginMethod    LoginMethod // 로그인 방법
    twoFactorUsed  bool        // 2FA 사용 여부
    riskScore      float64     // 세션 위험도 점수 (0.0-1.0)
    
    // 지리적 정보 (암호화)
    location encryption.EncryptedJSON // 접속 위치 정보
    
    // 도메인 메타데이터
    createdAt time.Time
    version   int64
    events    []DomainEvent
}

type SessionID string

type SessionStatus string
const (
    SessionStatusActive   SessionStatus = "ACTIVE"   // 활성 세션
    SessionStatusExpired  SessionStatus = "EXPIRED"  // 만료됨
    SessionStatusRevoked  SessionStatus = "REVOKED"  // 취소됨
    SessionStatusSuspicious SessionStatus = "SUSPICIOUS" // 의심스러운 활동
)

type LoginMethod string
const (
    LoginMethodPassword    LoginMethod = "PASSWORD"     // 비밀번호 로그인
    LoginMethodTwoFactor   LoginMethod = "TWO_FACTOR"   // 2FA 로그인
    LoginMethodSSO         LoginMethod = "SSO"          // Single Sign-On
    LoginMethodBiometric   LoginMethod = "BIOMETRIC"    // 생체인증
    LoginMethodSocialLogin LoginMethod = "SOCIAL_LOGIN" // 소셜 로그인
)

type DeviceInfo struct {
    DeviceID       string `json:"device_id"`       // 디바이스 고유 ID
    DeviceType     string `json:"device_type"`     // mobile, desktop, tablet
    OS             string `json:"os"`              // iOS, Android, Windows
    OSVersion      string `json:"os_version"`      // 운영체제 버전
    AppVersion     string `json:"app_version"`     // 앱 버전
    BrowserName    string `json:"browser_name"`    // Chrome, Safari, etc
    BrowserVersion string `json:"browser_version"` // 브라우저 버전
}

type LocationInfo struct {
    Country     string  `json:"country"`      // 국가
    Region      string  `json:"region"`       // 지역/주
    City        string  `json:"city"`         // 도시
    Latitude    float64 `json:"latitude"`     // 위도
    Longitude   float64 `json:"longitude"`    // 경도
    Timezone    string  `json:"timezone"`     // 시간대
    ISP         string  `json:"isp"`          // 인터넷 서비스 제공업체
}

// Session 생성자 및 팩토리 메서드
func NewSession(
    userID UserID,
    accessTokenHash, refreshTokenHash string,
    ipAddress, userAgent string,
    deviceInfo DeviceInfo,
    loginMethod LoginMethod,
    encryptionService encryption.Service,
) (*Session, error) {
    // 새로운 세션을 생성합니다
    // - 토큰 해시값 저장 (원본 토큰은 저장하지 않음)
    // - IP 주소와 User-Agent 암호화
    // - 디바이스 정보 암호화
    // - 세션 위험도 초기 평가
    // - 만료 시간 계산 및 설정
    // - 도메인 이벤트 생성 (SessionCreated)
}

// Session 도메인 메서드들
func (s *Session) ID() SessionID {
    // 세션 ID를 반환합니다
}

func (s *Session) UserID() UserID {
    // 연관된 사용자 ID를 반환합니다
}

func (s *Session) IsValid() bool {
    // 세션 유효성을 확인합니다
    // - 활성 상태 확인
    // - 만료 시간 확인
    // - 위험도 점수 확인
}

func (s *Session) UpdateActivity() {
    // 마지막 활동 시간을 업데이트합니다
    // - 현재 시간으로 lastActivity 갱신
    // - 자동 만료 타이머 리셋
    // - 활동 패턴 분석을 위한 이벤트 생성
}

func (s *Session) RefreshTokens(newAccessHash, newRefreshHash string, newExpiryTimes TokenExpiryTimes) error {
    // 토큰을 갱신합니다
    // - 현재 세션 유효성 확인
    // - 새로운 토큰 해시값 업데이트
    // - 만료 시간 업데이트
    // - 도메인 이벤트 생성 (TokensRefreshed)
}

func (s *Session) Revoke(reason string, revokedBy *UserID) error {
    // 세션을 취소합니다
    // - 세션 상태를 REVOKED로 변경
    // - 취소 사유 기록
    // - 취소한 주체 기록
    // - 도메인 이벤트 생성 (SessionRevoked)
}

func (s *Session) MarkSuspicious(reason string, riskScore float64) error {
    // 세션을 의심스러운 것으로 표시합니다
    // - 위험도 점수 업데이트
    // - 상태를 SUSPICIOUS로 변경
    // - 의심스러운 활동 사유 기록
    // - 자동 보안 조치 트리거
    // - 도메인 이벤트 생성 (SessionMarkedSuspicious)
}

func (s *Session) UpdateLocation(locationInfo LocationInfo, encryptionService encryption.Service) error {
    // 세션의 위치 정보를 업데이트합니다
    // - 위치 정보 암호화
    // - 위치 변경 패턴 분석
    // - 비정상 위치 접근 탐지
    // - 도메인 이벤트 생성 (LocationUpdated)
}

func (s *Session) CalculateRiskScore(riskFactors []RiskFactor) float64 {
    // 세션 위험도를 계산합니다
    // - 위치 기반 위험도 평가
    // - 디바이스 신뢰도 평가
    // - 접근 패턴 이상 탐지
    // - 시간대별 접근 패턴 분석
    // - 종합 위험도 점수 반환 (0.0-1.0)
}

func (s *Session) IsAccessibleFromLocation(currentLocation LocationInfo) bool {
    // 현재 위치에서 접근 가능한지 확인합니다
    // - 지역 제한 정책 확인
    // - VPN/프록시 사용 탐지
    // - 비정상적인 위치 이동 탐지
}

func (s *Session) GetDecryptedIPAddress(decryptionService encryption.Service) (string, error) {
    // 암호화된 IP 주소를 복호화하여 반환합니다
    // - 접근 권한 확인 (관리자만)
    // - IP 주소 복호화
    // - 감사 로그 기록
}

func (s *Session) GetDecryptedDeviceInfo(decryptionService encryption.Service) (*DeviceInfo, error) {
    // 암호화된 디바이스 정보를 복호화하여 반환합니다
    // - 접근 권한 확인
    // - 디바이스 정보 복호화
    // - 감사 로그 기록
}

func (s *Session) HasExpired() bool {
    // 세션 만료 여부를 확인합니다
    // - 액세스 토큰 만료 확인
    // - 리프레시 토큰 만료 확인
    // - 절대 만료 시간 확인
}

func (s *Session) TimeUntilExpiry() time.Duration {
    // 세션 만료까지 남은 시간을 반환합니다
}

// 도메인 이벤트 관리
func (s *Session) DomainEvents() []DomainEvent {
    // 발생한 도메인 이벤트들을 반환합니다
}

func (s *Session) ClearEvents() {
    // 처리 완료된 도메인 이벤트를 제거합니다
}
```

### **🔐 User Domain Services**

```go
// internal/user/domain/services.go
package domain

// AuthenticationService는 인증 관련 복잡한 비즈니스 로직을 처리합니다
type AuthenticationService interface {
    // 비밀번호 강도 검증
    ValidatePasswordStrength(password string) (*PasswordStrengthResult, error)
    
    // 이메일/전화번호 중복 확인
    CheckEmailAvailability(ctx context.Context, email string) (bool, error)
    CheckPhoneAvailability(ctx context.Context, phoneNumber string) (bool, error)
    
    // 로그인 시도 분석 및 보안 조치
    AnalyzeLoginAttempt(ctx context.Context, userID UserID, loginAttempt LoginAttempt) (*SecurityAssessment, error)
    
    // 계정 잠금 정책 적용
    ShouldLockAccount(ctx context.Context, user *User, failedAttempts []LoginAttempt) (*LockDecision, error)
    
    // 2FA 시크릿 키 생성 및 검증
    GenerateTwoFactorSecret(ctx context.Context, userID UserID) (*TwoFactorSecret, error)
    ValidateTwoFactorCode(ctx context.Context, userID UserID, code string) (bool, error)
}

// AuthorizationService는 권한 관리 로직을 처리합니다
type AuthorizationService interface {
    // 역할 기반 기본 권한 할당
    AssignDefaultPermissions(ctx context.Context, role UserRole) ([]Permission, error)
    
    // 리소스 접근 권한 확인
    CheckResourceAccess(ctx context.Context, userID UserID, resource Resource, action Action) (*AccessResult, error)
    
    // 권한 상속 및 위임 처리
    ResolveEffectivePermissions(ctx context.Context, user *User) ([]Permission, error)
    
    // 권한 변경 영향도 분석
    AnalyzePermissionChangeImpact(ctx context.Context, userID UserID, newPermissions []Permission) (*PermissionImpactAnalysis, error)
}

// SessionManagementService는 세션 관리 로직을 처리합니다
type SessionManagementService interface {
    // 세션 생성 및 토큰 발급
    CreateSession(ctx context.Context, user *User, loginContext LoginContext) (*SessionResult, error)
    
    // 토큰 갱신 처리
    RefreshSession(ctx context.Context, refreshToken string) (*TokenRefreshResult, error)
    
    // 세션 위험도 실시간 평가
    AssessSessionRisk(ctx context.Context, session *Session, currentActivity ActivityContext) (*RiskAssessment, error)
    
    // 동시 세션 관리 (중복 로그인 정책)
    ManageConcurrentSessions(ctx context.Context, userID UserID, newSession *Session) (*ConcurrentSessionResult, error)
    
    // 의심스러운 세션 탐지 및 처리
    DetectAnomalousSession(ctx context.Context, session *Session) (*AnomalyDetectionResult, error)
}

// UserProfileService는 사용자 프로필 관리를 처리합니다
type UserProfileService interface {
    // 프로필 완성도 평가
    EvaluateProfileCompleteness(ctx context.Context, user *User) (*ProfileCompletenessScore, error)
    
    // 개인정보 익명화 처리
    AnonymizeUserData(ctx context.Context, userID UserID, retentionPolicy RetentionPolicy) error
    
    // GDPR 준수 데이터 삭제
    ProcessDataDeletionRequest(ctx context.Context, userID UserID, deletionRequest DeletionRequest) (*DeletionResult, error)
    
    // 사용자 활동 패턴 분석
    AnalyzeUserActivity(ctx context.Context, userID UserID, timeRange TimeRange) (*ActivityAnalysis, error)
}
```

### **📊 User Repository Interface**

```go
// internal/user/domain/repository.go
package domain

// UserRepository는 사용자 데이터 영속성을 담당합니다
type UserRepository interface {
    // 사용자 생성
    Create(ctx context.Context, user *User) error
    
    // ID로 사용자 조회
    GetByID(ctx context.Context, id UserID) (*User, error)
    
    // 암호화된 이메일로 사용자 조회 (로그인용)
    GetByEncryptedEmail(ctx context.Context, encryptedEmail encryption.EncryptedString) (*User, error)
    
    // 암호화된 전화번호로 사용자 조회
    GetByEncryptedPhoneNumber(ctx context.Context, encryptedPhone encryption.EncryptedString) (*User, error)
    
    // 역할별 사용자 목록 조회
    GetByRole(ctx context.Context, role UserRole, filter UserFilter) (*UserList, error)
    
    // 상태별 사용자 조회 (관리용)
    GetByStatus(ctx context.Context, status UserStatus, limit int) ([]*User, error)
    
    // 사용자 정보 업데이트 (낙관적 잠금 적용)
    Update(ctx context.Context, user *User) error
    
    // 사용자 삭제 (소프트 삭제)
    Delete(ctx context.Context, id UserID) error
    
    // 이메일 중복 확인 (암호화된 상태로)
    ExistsByEncryptedEmail(ctx context.Context, encryptedEmail encryption.EncryptedString) (bool, error)
    
    // 전화번호 중복 확인 (암호화된 상태로)  
    ExistsByEncryptedPhoneNumber(ctx context.Context, encryptedPhone encryption.EncryptedString) (bool, error)
    
    // 사용자 검색 (관리자용)
    Search(ctx context.Context, criteria UserSearchCriteria) (*UserSearchResult, error)
    
    // 비활성 사용자 조회 (정리용)
    GetInactiveUsers(ctx context.Context, inactiveDuration time.Duration) ([]*User, error)
}

// SessionRepository는 세션 데이터 영속성을 담당합니다
type SessionRepository interface {
    // 세션 생성
    Create(ctx context.Context, session *Session) error
    
    // ID로 세션 조회
    GetByID(ctx context.Context, id SessionID) (*Session, error)
    
    // 토큰 해시로 세션 조회
    GetByTokenHash(ctx context.Context, tokenHash string) (*Session, error)
    
    // 사용자별 활성 세션 목록 조회
    GetActiveSessionsByUserID(ctx context.Context, userID UserID) ([]*Session, error)
    
    // 만료된 세션 조회 (정리용)
    GetExpiredSessions(ctx context.Context, limit int) ([]*Session, error)
    
    // 의심스러운 세션 조회
    GetSuspiciousSessions(ctx context.Context, riskThreshold float64) ([]*Session, error)
    
    // 세션 정보 업데이트
    Update(ctx context.Context, session *Session) error
    
    // 세션 삭제 (하드 삭제)
    Delete(ctx context.Context, id SessionID) error
    
    // 사용자의 모든 세션 무효화
    RevokeAllUserSessions(ctx context.Context, userID UserID, reason string) error
    
    // 세션 통계 조회
    GetSessionStatistics(ctx context.Context, timeRange TimeRange) (*SessionStatistics, error)
}

// UserFilter는 사용자 조회 시 사용하는 필터링 옵션입니다
type UserFilter struct {
    Status        []UserStatus `json:"status"`
    Role          []UserRole   `json:"role"`
    EmailVerified *bool        `json:"email_verified"`
    PhoneVerified *bool        `json:"phone_verified"`
    CreatedAfter  *time.Time   `json:"created_after"`
    CreatedBefore *time.Time   `json:"created_before"`
    
    // 페이징
    Page  int `json:"page" validate:"min=1"`
    Limit int `json:"limit" validate:"min=1,max=100"`
    
    // 정렬
    SortBy    string `json:"sort_by" validate:"oneof=created_at last_login_at"`
    SortOrder string `json:"sort_order" validate:"oneof=asc desc"`
}

// UserList는 페이징된 사용자 목록을 나타냅니다
type UserList struct {
    Items      []*User `json:"items"`
    TotalCount int64   `json:"total_count"`
    Page       int     `json:"page"`
    Limit      int     `json:"limit"`
    HasNext    bool    `json:"has_next"`
}
```

-----

## 🛡️ **보안 지원 도메인들**

OrderSync의 보안 우선 설계를 지원하는 핵심 도메인들입니다.

## 🔐 **Encryption Domain**

모든 민감 데이터의 암호화/복호화를 담당하는 보안 핵심 도메인입니다.

### **🔑 Encryption Service**

```go
// internal/encryption/domain/encryption.go
package domain

import (
    "context"
    "time"
)

// EncryptionService는 데이터 암호화/복호화를 담당하는 도메인 서비스입니다
type EncryptionService interface {
    // 문자열 데이터 암호화
    EncryptString(ctx context.Context, plaintext string, keyType KeyType) (*EncryptedString, error)
    
    // 문자열 데이터 복호화
    DecryptString(ctx context.Context, encrypted EncryptedString) (string, error)
    
    // JSON 데이터 암호화
    EncryptJSON(ctx context.Context, data interface{}, keyType KeyType) (*EncryptedJSON, error)
    
    // JSON 데이터 복호화
    DecryptJSON(ctx context.Context, encrypted EncryptedJSON, target interface{}) error
    
    // 키 회전 (정기적 보안 강화)
    RotateKey(ctx context.Context, keyType KeyType) (*KeyRotationResult, error)
    
    // 암호화 키 상태 확인
    GetKeyStatus(ctx context.Context, keyType KeyType) (*KeyStatus, error)
    
    // 대량 데이터 재암호화 (키 회전 시)
    ReencryptData(ctx context.Context, oldKeyID, newKeyID string, batchSize int) (*ReencryptionProgress, error)
}

// EncryptedString은 암호화된 문자열을 나타냅니다
type EncryptedString struct {
    Value     string    `json:"value" db:"encrypted_value"`     // 암호화된 데이터
    KeyID     string    `json:"key_id" db:"key_id"`             // 사용된 암호화 키 ID  
    Algorithm string    `json:"algorithm" db:"algorithm"`       // 암호화 알고리즘 (AES-256-GCM)
    Nonce     string    `json:"nonce" db:"nonce"`               // 암호화 논스
    CreatedAt time.Time `json:"created_at" db:"created_at"`     // 암호화 시간
}

// EncryptedJSON은 암호화된 JSON 데이터를 나타냅니다
type EncryptedJSON struct {
    Value     string    `json:"value" db:"encrypted_value"`
    KeyID     string    `json:"key_id" db:"key_id"`
    Algorithm string    `json:"algorithm" db:"algorithm"`
    Nonce     string    `json:"nonce" db:"nonce"`
    CreatedAt time.Time `json:"created_at" db:"created_at"`
}

// KeyType은 암호화 키의 용도를 구분합니다
type KeyType string
const (
    KeyTypePII        KeyType = "pii"         // 개인정보 (이름, 이메일, 전화번호)
    KeyTypePayment    KeyType = "payment"     // 결제정보 (카드정보, 결제 ID)
    KeyTypeBusiness   KeyType = "business"    // 사업정보 (사업자등록번호, 매장정보)
    KeyTypeAuth       KeyType = "auth"        // 인증정보 (비밀번호 해시)
    KeyTypeSession    KeyType = "session"     // 세션정보 (IP, User-Agent)
    KeyTypeOrder      KeyType = "order"       // 주문정보 (주문 상세, 특별 요청)
    KeyTypeAudit      KeyType = "audit"       // 감사정보 (로그 데이터)
)

// KeyStatus는 암호화 키의 현재 상태를 나타냅니다
type KeyStatus struct {
    KeyID        string    `json:"key_id"`
    KeyType      KeyType   `json:"key_type"`
    Status       string    `json:"status"`        // ACTIVE, ROTATING, RETIRED
    CreatedAt    time.Time `json:"created_at"`
    LastUsedAt   time.Time `json:"last_used_at"`
    NextRotation time.Time `json:"next_rotation"` // 다음 키 회전 예정일
    Version      int       `json:"version"`       // 키 버전
}

// EncryptedString 도메인 메서드들
func (e *EncryptedString) IsValid() bool {
    // 암호화된 데이터의 유효성을 검증합니다
    // - 필수 필드 존재 여부 확인
    // - 알고리즘 지원 여부 확인
    // - 데이터 형식 유효성 확인
}

func (e *EncryptedString) NeedsReencryption(maxAge time.Duration) bool {
    // 재암호화 필요 여부를 확인합니다
    // - 암호화 후 경과 시간 확인
    // - 키 회전 정책에 따른 재암호화 필요성 판단
}

func (e *EncryptedString) GetKeyAge() time.Duration {
    // 암호화 키 사용 기간을 반환합니다
}

// EncryptedJSON 도메인 메서드들
func (e *EncryptedJSON) IsValid() bool {
    // 암호화된 JSON 데이터의 유효성을 검증합니다
}

func (e *EncryptedJSON) GetDataSize() int {
    // 암호화된 데이터의 크기를 반환합니다 (성능 모니터링용)
}

func (e *EncryptedJSON) EstimateDecryptionCost() int {
    // 복호화 예상 비용을 계산합니다 (데이터 크기 기반)
}
```

### **🔑 Key Management Service**

```go
// internal/encryption/domain/key_management.go
package domain

// KeyManagementService는 암호화 키의 생명주기를 관리합니다
type KeyManagementService interface {
    // 새로운 암호화 키 생성
    GenerateKey(ctx context.Context, keyType KeyType) (*EncryptionKey, error)
    
    // 키 활성화 (사용 가능 상태로 변경)
    ActivateKey(ctx context.Context, keyID string) error
    
    // 키 비활성화 (새로운 암호화 중단, 복호화만 가능)
    DeactivateKey(ctx context.Context, keyID string, reason string) error
    
    // 키 폐기 (완전 삭제, 복구 불가)
    RetireKey(ctx context.Context, keyID string, retentionPeriod time.Duration) error
    
    // 키 회전 실행 (새 키 생성 후 기존 키 교체)
    RotateKeys(ctx context.Context, keyType KeyType) (*KeyRotationResult, error)
    
    // 키 백업 및 복구
    BackupKey(ctx context.Context, keyID string) (*KeyBackup, error)
    RestoreKey(ctx context.Context, backup KeyBackup) error
    
    // 키 사용량 통계 조회
    GetKeyUsageStatistics(ctx context.Context, keyID string, timeRange TimeRange) (*KeyUsageStats, error)
    
    // 키 보안 감사
    AuditKeyAccess(ctx context.Context, keyID string) (*KeyAccessAudit, error)
}

// EncryptionKey는 암호화 키 정보를 나타냅니다
type EncryptionKey struct {
    ID          string    `json:"id"`           // 키 고유 식별자
    Type        KeyType   `json:"type"`         // 키 용도
    Algorithm   string    `json:"algorithm"`    // 암호화 알고리즘
    KeySize     int       `json:"key_size"`     // 키 크기 (비트)
    Status      KeyStatus `json:"status"`       // 키 상태
    CreatedAt   time.Time `json:"created_at"`   // 생성 시간
    ActivatedAt *time.Time `json:"activated_at"` // 활성화 시간
    ExpiresAt   *time.Time `json:"expires_at"`   // 만료 시간
    Metadata    map[string]interface{} `json:"metadata"` // 추가 메타데이터
}

// KeyRotationResult는 키 회전 작업 결과를 나타냅니다
type KeyRotationResult struct {
    OldKeyID          string    `json:"old_key_id"`
    NewKeyID          string    `json:"new_key_id"`
    RotationStartedAt time.Time `json:"rotation_started_at"`
    RotationCompletedAt *time.Time `json:"rotation_completed_at"`
    ReencryptedRecords int64     `json:"reencrypted_records"`
    FailedRecords     int64     `json:"failed_records"`
    Status            string    `json:"status"` // IN_PROGRESS, COMPLETED, FAILED
}

// KeyUsageStats는 키 사용 통계를 나타냅니다
type KeyUsageStats struct {
    KeyID           string `json:"key_id"`
    EncryptionCount int64  `json:"encryption_count"`  // 암호화 작업 횟수
    DecryptionCount int64  `json:"decryption_count"`  // 복호화 작업 횟수
    DataVolume      int64  `json:"data_volume"`       // 처리된 데이터 볼륨 (바이트)
    AvgResponseTime int64  `json:"avg_response_time"` // 평균 응답시간 (밀리초)
    ErrorCount      int64  `json:"error_count"`       // 오류 발생 횟수
}

// EncryptionKey 도메인 메서드들
func (k *EncryptionKey) IsActive() bool {
    // 키가 활성 상태인지 확인합니다
    // - 현재 상태 확인 (ACTIVE)
    // - 만료 시간 확인
    // - 사용 가능 여부 판단
}

func (k *EncryptionKey) NeedsRotation(rotationPolicy RotationPolicy) bool {
    // 키 회전이 필요한지 확인합니다
    // - 마지막 회전 후 경과 시간 확인
    // - 사용량 기반 회전 정책 확인
    // - 보안 정책 준수 여부 확인
}

func (k *EncryptionKey) CanBeRetired() bool {
    // 키 폐기 가능 여부를 확인합니다
    // - 현재 사용 중인 암호화 데이터 존재 여부
    // - 법적 보관 의무 기간 확인
    // - 의존성 분석 결과 확인
}

func (k *EncryptionKey) GetAge() time.Duration {
    // 키 생성 후 경과 시간을 반환합니다
}

func (k *EncryptionKey) EstimateReencryptionCost(dataVolume int64) *ReencryptionCostEstimate {
    // 재암호화 예상 비용을 계산합니다
    // - 처리해야 할 데이터 볼륨
    // - 예상 소요 시간
    // - 시스템 리소스 사용량
    // - 다운타임 예상 시간
}
```

### **📊 Encryption Repository Interface**

```go
// internal/encryption/domain/repository.go
package domain

// EncryptionKeyRepository는 암호화 키 메타데이터 영속성을 담당합니다
type EncryptionKeyRepository interface {
    // 키 메타데이터 저장 (실제 키는 KMS에 저장)
    Create(ctx context.Context, key *EncryptionKey) error
    
    // 키 ID로 메타데이터 조회
    GetByID(ctx context.Context, keyID string) (*EncryptionKey, error)
    
    // 키 타입별 활성 키 조회
    GetActiveKeyByType(ctx context.Context, keyType KeyType) (*EncryptionKey, error)
    
    // 키 타입별 모든 키 조회 (회전 이력 포함)
    GetAllKeysByType(ctx context.Context, keyType KeyType) ([]*EncryptionKey, error)
    
    // 만료 예정 키 조회
    GetExpiringKeys(ctx context.Context, within time.Duration) ([]*EncryptionKey, error)
    
    // 회전 필요 키 조회
    GetKeysNeedingRotation(ctx context.Context, policy RotationPolicy) ([]*EncryptionKey, error)
    
    // 키 상태 업데이트
    UpdateStatus(ctx context.Context, keyID string, status string) error
    
    // 키 메타데이터 업데이트
    Update(ctx context.Context, key *EncryptionKey) error
    
    // 키 사용량 기록
    RecordKeyUsage(ctx context.Context, keyID string, operation string, dataSize int64) error
    
    // 키 삭제 (메타데이터만, 실제 키는 별도 처리)
    Delete(ctx context.Context, keyID string) error
}

// EncryptionAuditRepository는 암호화 작업 감사 로그를 담당합니다
type EncryptionAuditRepository interface {
    // 암호화 작업 로그 기록
    LogEncryptionOperation(ctx context.Context, log *EncryptionAuditLog) error
    
    // 복호화 작업 로그 기록 (민감한 작업)
    LogDecryptionOperation(ctx context.Context, log *DecryptionAuditLog) error
    
    // 키 관리 작업 로그 기록
    LogKeyManagementOperation(ctx context.Context, log *KeyManagementAuditLog) error
    
    // 사용자별 암호화 작업 이력 조회
    GetEncryptionHistoryByUser(ctx context.Context, userID string, timeRange TimeRange) ([]*EncryptionAuditLog, error)
    
    // 키별 사용 이력 조회
    GetKeyUsageHistory(ctx context.Context, keyID string, timeRange TimeRange) ([]*EncryptionAuditLog, error)
    
    // 의심스러운 복호화 시도 조회
    GetSuspiciousDecryptionAttempts(ctx context.Context, criteria SuspiciousActivityCriteria) ([]*DecryptionAuditLog, error)
    
    // 감사 로그 통계
    GetEncryptionAuditStatistics(ctx context.Context, timeRange TimeRange) (*EncryptionAuditStats, error)
}
```

-----

## 📊 **Audit Domain**

모든 시스템 활동을 추적하고 기록하는 감사 도메인입니다.

### **📝 Audit Log Aggregate**

```go
// internal/audit/domain/audit_log.go
package domain

import (
    "context"
    "time"
    "github.com/ordersync/internal/encryption/domain"
    userDomain "github.com/ordersync/internal/user/domain"
)

// AuditLog는 시스템 감사 로그를 나타내는 Aggregate Root입니다
type AuditLog struct {
    // 기본 식별자
    id      AuditLogID
    traceID string     // 분산 추적 ID (요청 전체 추적)
    
    // 액션 정보
    action       AuditAction // 수행된 액션
    resourceType string      // 대상 리소스 타입 (Order, User, Payment 등)
    resourceID   string      // 대상 리소스 ID
    
    // 주체 정보 (암호화)
    userID    *userDomain.UserID           // 액션 수행자 (nil이면 시스템)
    ipAddress encryption.EncryptedString   // 접속 IP 주소
    userAgent encryption.EncryptedString   // User-Agent 정보
    
    // 세션 정보
    sessionID *userDomain.SessionID // 관련 세션 ID
    
    // 변경 내용 (암호화)
    oldValue encryption.EncryptedJSON // 변경 전 값
    newValue encryption.EncryptedJSON // 변경 후 값
    
    // 액션 결과
    success      bool     // 성공/실패 여부
    errorMessage *string  // 오류 메시지 (실패 시)
    statusCode   *int     // HTTP 상태 코드
    
    // 성능 정보
    duration time.Duration // 액션 수행 시간
    
    // 보안 정보
    riskLevel    RiskLevel // 위험도 수준
    sensitivityLevel SensitivityLevel // 민감도 수준
    
    // 컨텍스트 정보 (암호화)
    contextData encryption.EncryptedJSON // 추가 컨텍스트 정보
    
    // 지리적 정보 (암호화)
    geoLocation encryption.EncryptedJSON // 지리적 위치 정보
    
    // 메타데이터
    timestamp time.Time // 액션 수행 시간
    version   int64     // 로그 버전
    checksum  string    // 무결성 검증용 체크섬
    
    // 도메인 이벤트 (메모리에만 존재)
    events []DomainEvent
}

type AuditLogID string

type AuditAction string
const (
    // 인증 관련 액션
    AuditActionLogin            AuditAction = "LOGIN"
    AuditActionLogout           AuditAction = "LOGOUT"
    AuditActionPasswordChange   AuditAction = "PASSWORD_CHANGE"
    AuditActionTwoFactorEnable  AuditAction = "TWO_FACTOR_ENABLE"
    
    // 사용자 관리 액션
    AuditActionUserCreate       AuditAction = "USER_CREATE"
    AuditActionUserUpdate       AuditAction = "USER_UPDATE"
    AuditActionUserDelete       AuditAction = "USER_DELETE"
    AuditActionUserDeactivate   AuditAction = "USER_DEACTIVATE"
    
    // 주문 관련 액션
    AuditActionOrderCreate      AuditAction = "ORDER_CREATE"
    AuditActionOrderAccept      AuditAction = "ORDER_ACCEPT"
    AuditActionOrderCancel      AuditAction = "ORDER_CANCEL"
    AuditActionOrderComplete    AuditAction = "ORDER_COMPLETE"
    
    // 결제 관련 액션
    AuditActionPaymentCreate    AuditAction = "PAYMENT_CREATE"
    AuditActionPaymentCapture   AuditAction = "PAYMENT_CAPTURE"
    AuditActionPaymentRefund    AuditAction = "PAYMENT_REFUND"
    
    // 데이터 접근 액션
    AuditActionDataAccess       AuditAction = "DATA_ACCESS"
    AuditActionDataExport       AuditAction = "DATA_EXPORT"
    AuditActionDataImport       AuditAction = "DATA_IMPORT"
    
    // 시스템 관리 액션
    AuditActionConfigChange     AuditAction = "CONFIG_CHANGE"
    AuditActionKeyRotation      AuditAction = "KEY_ROTATION"
    AuditActionSystemMaintenance AuditAction = "SYSTEM_MAINTENANCE"
    
    // 보안 관련 액션
    AuditActionSuspiciousActivity AuditAction = "SUSPICIOUS_ACTIVITY"
    AuditActionSecurityBreach     AuditAction = "SECURITY_BREACH"
    AuditActionAccessDenied       AuditAction = "ACCESS_DENIED"
)

type RiskLevel string
const (
    RiskLevelLow      RiskLevel = "LOW"      // 일반적인 활동
    RiskLevelMedium   RiskLevel = "MEDIUM"   // 주의 필요
    RiskLevelHigh     RiskLevel = "HIGH"     // 위험한 활동
    RiskLevelCritical RiskLevel = "CRITICAL" // 긴급 대응 필요
)

type SensitivityLevel string
const (
    SensitivityLevelPublic     SensitivityLevel = "PUBLIC"     // 공개 정보
    SensitivityLevelInternal   SensitivityLevel = "INTERNAL"   // 내부 정보
    SensitivityLevelConfidential SensitivityLevel = "CONFIDENTIAL" // 기밀 정보
    SensitivityLevelRestricted SensitivityLevel = "RESTRICTED" // 제한 정보
)

// AuditLog 생성자 및 팩토리 메서드
func NewAuditLog(
    traceID string,
    action AuditAction,
    resourceType, resourceID string,
    userID *userDomain.UserID,
    ipAddress, userAgent string,
    encryptionService encryption.Service,
) (*AuditLog, error) {
    // 새로운 감사 로그를 생성합니다
    // - 필수 필드 검증 (traceID, action, timestamp)
    // - IP 주소와 User-Agent 암호화
    // - 위험도 및 민감도 수준 자동 분류
    // - 무결성 검증용 체크섬 생성
    // - 도메인 이벤트 생성 (AuditLogCreated)
}

// AuditLog 도메인 메서드들
func (a *AuditLog) ID() AuditLogID {
    // 감사 로그 ID를 반환합니다
}

func (a *AuditLog) TraceID() string {
    // 분산 추적 ID를 반환합니다
}

func (a *AuditLog) SetChangeData(oldValue, newValue interface{}, encryptionService encryption.Service) error {
    // 변경 전후 데이터를 암호화하여 저장합니다
    // - 민감 정보 필터링 (비밀번호, 토큰 등 제외)
    // - JSON 직렬화 후 암호화
    // - 데이터 크기 제한 확인 (성능 고려)
    // - 변경 내용 요약 생성
}

func (a *AuditLog) SetContextData(context map[string]interface{}, encryptionService encryption.Service) error {
    // 추가 컨텍스트 정보를 암호화하여 저장합니다
    // - HTTP 헤더 정보
    // - 요청 파라미터 (민감정보 제외)
    // - 환경 정보 (브라우저, OS 등)
    // - 비즈니스 컨텍스트 (매장 ID, 캠페인 ID 등)
}

func (a *AuditLog) MarkAsSuccess(duration time.Duration) {
    // 액션 성공으로 표시합니다
    // - 성공 상태 설정
    // - 수행 시간 기록
    // - 성공 메트릭 업데이트
}

func (a *AuditLog) MarkAsFailure(errorMessage string, statusCode int, duration time.Duration) {
    // 액션 실패로 표시합니다
    // - 실패 상태 설정
    // - 오류 메시지 및 상태 코드 기록
    // - 실패 원인 분류 (인증, 권한, 시스템 오류 등)
    // - 실패 메트릭 업데이트
}

func (a *AuditLog) AssessRiskLevel(riskAssessmentService RiskAssessmentService) error {
    // 액션의 위험도를 평가합니다
    // - 액션 유형별 기본 위험도
    // - 사용자 권한 수준 고려
    // - 접근 패턴 분석 (시간, 위치, 빈도)
    // - 비정상 행동 패턴 탐지
    // - 위험도 수준 업데이트
}

func (a *AuditLog) ClassifySensitivity() SensitivityLevel {
    // 로그 데이터의 민감도를 분류합니다
    // - 리소스 타입별 민감도 매핑
    // - 개인정보 포함 여부 확인
    // - 비즈니스 중요도 평가
    // - 법적 규제 대상 여부 확인
}

func (a *AuditLog) VerifyIntegrity() bool {
    // 로그 무결성을 검증합니다
    // - 체크섬 재계산 및 비교
    // - 타임스탬프 유효성 확인
    // - 데이터 변조 탐지
}

func (a *AuditLog) GenerateChecksum() string {
    // 로그 데이터의 체크섬을 생성합니다
    // - SHA-256 해시 계산
    // - 주요 필드들을 조합하여 해시 생성
    // - 변조 탐지용 무결성 검증 코드
}

func (a *AuditLog) IsHighRisk() bool {
    // 고위험 로그인지 확인합니다
    // - 위험도 수준 확인 (HIGH, CRITICAL)
    // - 보안 관련 액션 여부
    // - 실패한 민감 작업 여부
}

func (a *AuditLog) RequiresImmediateAlert() bool {
    // 즉시 알림이 필요한 로그인지 확인합니다
    // - CRITICAL 위험도 수준
    // - 보안 침해 의심 액션
    // - 시스템 장애 관련 액션
    // - 대량 데이터 접근 시도
}

func (a *AuditLog) GetRetentionPeriod() time.Duration {
    // 로그 보관 기간을 반환합니다
    // - 민감도 수준별 보관 정책
    // - 법적 요구사항 (개인정보보호법, 금융법 등)
    // - 비즈니스 요구사항
    // - 스토리지 비용 고려
}

func (a *AuditLog) CanBeArchived() bool {
    // 아카이브 가능 여부를 확인합니다
    // - 최소 보관 기간 경과 확인
    // - 진행 중인 감사/조사 연관성 확인
    // - 법적 보관 의무 확인
}

func (a *AuditLog) ShouldBeRedacted() bool {
    // 개인정보 삭제 필요 여부를 확인합니다
    // - GDPR 삭제 권리 적용 대상
    // - 개인정보 보관 기간 초과
    // - 사용자 계정 삭제 요청
}

// 도메인 이벤트 관리
func (a *AuditLog) DomainEvents() []DomainEvent {
    // 발생한 도메인 이벤트들을 반환합니다
}

func (a *AuditLog) ClearEvents() {
    // 처리 완료된 도메인 이벤트를 제거합니다
}
```

### **🔍 Audit Domain Services**

```go
// internal/audit/domain/services.go
package domain

// AuditTrailService는 감사 증적 생성 및 관리를 담당합니다
type AuditTrailService interface {
    // 자동 감사 로그 생성 (AOP 방식)
    CreateAuditLog(ctx context.Context, auditContext AuditContext) (*AuditLog, error)
    
    // 감사 증적 체인 생성 (연관된 액션들의 순차적 추적)
    CreateAuditTrail(ctx context.Context, traceID string, actions []AuditAction) (*AuditTrail, error)
    
    // 감사 로그 무결성 검증
    VerifyAuditIntegrity(ctx context.Context, logID AuditLogID) (*IntegrityVerificationResult, error)
    
    // 감사 로그 체인 검증 (시간 순서, 논리적 일관성)
    VerifyAuditChain(ctx context.Context, traceID string) (*ChainVerificationResult, error)
    
    // 감사 요구사항 준수 검증
    ValidateComplianceRequirements(ctx context.Context, auditLogs []AuditLog, requirements ComplianceRequirements) (*ComplianceReport, error)
}

// RiskAssessmentService는 액션별 위험도 평가를 담당합니다
type RiskAssessmentService interface {
    // 실시간 위험도 평가
    AssessActionRisk(ctx context.Context, auditContext AuditContext) (*RiskAssessment, error)
    
    // 사용자 행동 패턴 분석
    AnalyzeBehaviorPattern(ctx context.Context, userID userDomain.UserID, timeWindow time.Duration) (*BehaviorAnalysis, error)
    
    // 이상 행동 탐지
    DetectAnomalousBehavior(ctx context.Context, auditLog *AuditLog, historicalData []AuditLog) (*AnomalyDetection, error)
    
    // 위험 점수 계산 (머신러닝 기반)
    CalculateRiskScore(ctx context.Context, features RiskFeatures) (*RiskScore, error)
    
    // 위험 임계값 업데이트 (적응형 학습)
    UpdateRiskThresholds(ctx context.Context, feedbackData []RiskFeedback) error
}

// AuditAnalyticsService는 감사 데이터 분석을 담당합니다
type AuditAnalyticsService interface {
    // 감사 로그 통계 분석
    GenerateAuditStatistics(ctx context.Context, criteria AnalyticsCriteria) (*AuditStatistics, error)
    
    // 보안 이벤트 트렌드 분석
    AnalyzeSecurityTrends(ctx context.Context, timeRange TimeRange) (*SecurityTrendAnalysis, error)
    
    // 사용자별 활동 패턴 리포트
    GenerateUserActivityReport(ctx context.Context, userID userDomain.UserID, period ReportPeriod) (*ActivityReport, error)
    
    // 시스템 접근 패턴 분석
    AnalyzeAccessPatterns(ctx context.Context, resourceType string, timeRange TimeRange) (*AccessPatternAnalysis, error)
    
    // 규정 준수 리포트 생성
    GenerateComplianceReport(ctx context.Context, regulation ComplianceRegulation, period ReportPeriod) (*ComplianceReport, error)
}

// AuditRetentionService는 감사 로그 보관 정책을 관리합니다
type AuditRetentionService interface {
    // 보관 정책 적용
    ApplyRetentionPolicy(ctx context.Context, logs []AuditLog, policy RetentionPolicy) (*RetentionResult, error)
    
    // 자동 아카이브 처리
    ArchiveOldLogs(ctx context.Context, archiveCriteria ArchiveCriteria) (*ArchiveResult, error)
    
    // 개인정보 삭제 처리 (GDPR 준수)
    RedactPersonalData(ctx context.Context, userID userDomain.UserID, redactionPolicy RedactionPolicy) (*RedactionResult, error)
    
    // 법적 보관 의무 확인
    CheckLegalHold(ctx context.Context, logID AuditLogID) (*LegalHoldStatus, error)
    
    // 보관 기간 연장 처리
    ExtendRetentionPeriod(ctx context.Context, logIDs []AuditLogID, extension time.Duration, reason string) error
}
```

### **📊 Audit Repository Interface**

```go
// internal/audit/domain/repository.go
package domain

// AuditLogRepository는 감사 로그 데이터 영속성을 담당합니다
type AuditLogRepository interface {
    // 감사 로그 생성 (불변 저장)
    Create(ctx context.Context, log *AuditLog) error
    
    // 감사 로그 일괄 생성 (성능 최적화)
    CreateBatch(ctx context.Context, logs []*AuditLog) error
    
    // ID로 감사 로그 조회
    GetByID(ctx context.Context, id AuditLogID) (*AuditLog, error)
    
    // 추적 ID로 관련 로그들 조회 (분산 추적)
    GetByTraceID(ctx context.Context, traceID string) ([]*AuditLog, error)
    
    // 사용자별 감사 로그 조회
    GetByUserID(ctx context.Context, userID userDomain.UserID, filter AuditLogFilter) (*AuditLogList, error)
    
    // 리소스별 감사 로그 조회
    GetByResource(ctx context.Context, resourceType, resourceID string, filter AuditLogFilter) ([]*AuditLog, error)
    
    // 액션별 감사 로그 조회
    GetByAction(ctx context.Context, actions []AuditAction, filter AuditLogFilter) ([]*AuditLog, error)
    
    // 시간 범위별 감사 로그 조회 (성능 최적화된 파티션 쿼리)
    GetByTimeRange(ctx context.Context, startTime, endTime time.Time, filter AuditLogFilter) ([]*AuditLog, error)
    
    // 위험도별 감사 로그 조회 (보안 모니터링용)
    GetByRiskLevel(ctx context.Context, riskLevels []RiskLevel, filter AuditLogFilter) ([]*AuditLog, error)
    
    // 실패한 액션 로그 조회 (장애 분석용)
    GetFailedActions(ctx context.Context, filter AuditLogFilter) ([]*AuditLog, error)
    
    // 감사 로그 검색 (풀텍스트 검색 지원)
    Search(ctx context.Context, criteria AuditSearchCriteria) (*AuditSearchResult, error)
    
    // 감사 로그 통계 조회
    GetStatistics(ctx context.Context, criteria StatisticsCriteria) (*AuditStatistics, error)
    
    // 아카이브 대상 로그 조회 (보관 정책 기반)
    GetArchiveCandidates(ctx context.Context, archivePolicy ArchivePolicy) ([]*AuditLog, error)
    
    // 삭제 대상 로그 조회 (보관 기간 만료)
    GetDeletionCandidates(ctx context.Context, retentionPolicy RetentionPolicy) ([]*AuditLog, error)
    
    // 감사 로그 아카이브 (콜드 스토리지 이동)
    Archive(ctx context.Context, logIDs []AuditLogID, archiveLocation string) error
    
    // 감사 로그 삭제 (법적 보관 기간 만료 후)
    Delete(ctx context.Context, logIDs []AuditLogID) error
    
    // 무결성 검증을 위한 체크섬 조회
    GetChecksums(ctx context.Context, logIDs []AuditLogID) (map[AuditLogID]string, error)
    
    // 데이터 내보내기 (규정 준수 리포트용)
    Export(ctx context.Context, criteria ExportCriteria, format ExportFormat) (*ExportResult, error)
}

// AuditLogFilter는 감사 로그 조회 시 사용하는 필터링 옵션입니다
type AuditLogFilter struct {
    Actions      []AuditAction    `json:"actions"`
    RiskLevels   []RiskLevel      `json:"risk_levels"`
    Success      *bool            `json:"success"`
    StartTime    *time.Time       `json:"start_time"`
    EndTime      *time.Time       `json:"end_time"`
    IPAddress    *string          `json:"ip_address"`    // 검색용 (암호화되지 않은 해시)
    UserAgent    *string          `json:"user_agent"`    // 검색용
    
    // 페이징
    Page         int              `json:"page" validate:"min=1"`
    Limit        int              `json:"limit" validate:"min=1,max=1000"`
    
    // 정렬
    SortBy       string           `json:"sort_by" validate:"oneof=timestamp risk_level duration"`
    SortOrder    string           `json:"sort_order" validate:"oneof=asc desc"`
}

// AuditLogList는 페이징된 감사 로그 목록을 나타냅니다
type AuditLogList struct {
    Items        []*AuditLog `json:"items"`
    TotalCount   int64       `json:"total_count"`
    Page         int         `json:"page"`
    Limit        int         `json:"limit"`
    HasNext      bool        `json:"has_next"`
}
```

-----

## 📈 **Metrics Domain**

성능, 보안, 비즈니스 메트릭을 수집하고 분석하는 도메인입니다.

### **📊 Metric Aggregate**

```go
// internal/metrics/domain/metric.go
package domain

import (
    "context"
    "time"
    userDomain "github.com/ordersync/internal/user/domain"
    restaurantDomain "github.com/ordersync/internal/restaurant/domain"
)

// Metric은 시스템 메트릭을 나타내는 Aggregate Root입니다
type Metric struct {
    // 기본 식별자
    id        MetricID
    
    // 메트릭 분류
    name      string     // 메트릭 이름 (예: api_response_time, order_count)
    type_     MetricType // 메트릭 유형
    category  MetricCategory // 메트릭 카테고리
    
    // 메트릭 값
    value     float64   // 메트릭 값
    unit      string    // 단위 (ms, count, percent, bytes 등)
    
    // 라벨 및 차원 (태그)
    labels    map[string]string // 메트릭 라벨 (endpoint, method, status 등)
    
    // 시간 정보
    timestamp time.Time  // 메트릭 수집 시간
    interval  time.Duration // 측정 간격 (집계 메트릭의 경우)
    
    // 컨텍스트 정보
    source    MetricSource // 메트릭 소스 (API, Database, Queue 등)
    
    // 임계값 및 알림
    thresholds []Threshold // 알림 임계값들
    alertLevel AlertLevel  // 현재 알림 수준
    
    // 메타데이터
    description string                 // 메트릭 설명
    metadata    map[string]interface{} // 추가 메타데이터
    
    // 도메인 메타데이터
    version int64 // 버전 (스키마 변경 추적용)
    events  []DomainEvent
}

type MetricID string

type MetricType string
const (
    MetricTypeCounter   MetricType = "COUNTER"   // 누적 카운터 (요청 수, 에러 수)
    MetricTypeGauge     MetricType = "GAUGE"     // 현재 값 (CPU 사용률, 메모리)
    MetricTypeHistogram MetricType = "HISTOGRAM" // 분포 메트릭 (응답시간 분포)
    MetricTypeSummary   MetricType = "SUMMARY"   // 요약 통계 (평균, 백분위수)
)

type MetricCategory string
const (
    // 성능 메트릭
    MetricCategoryPerformance MetricCategory = "PERFORMANCE"
    
    // 비즈니스 메트릭
    MetricCategoryBusiness    MetricCategory = "BUSINESS"
    
    // 보안 메트릭
    MetricCategorySecurity    MetricCategory = "SECURITY"
    
    // 시스템 메트릭
    MetricCategorySystem      MetricCategory = "SYSTEM"
    
    // 사용자 경험 메트릭
    MetricCategoryUX          MetricCategory = "USER_EXPERIENCE"
)

type MetricSource string
const (
    MetricSourceAPI        MetricSource = "API"
    MetricSourceDatabase   MetricSource = "DATABASE"
    MetricSourceQueue      MetricSource = "QUEUE"
    MetricSourceCache      MetricSource = "CACHE"
    MetricSourceApplication MetricSource = "APPLICATION"
    MetricSourceInfrastructure MetricSource = "INFRASTRUCTURE"
)

type AlertLevel string
const (
    AlertLevelNone     AlertLevel = "NONE"     // 정상
    AlertLevelInfo     AlertLevel = "INFO"     // 정보성 알림
    AlertLevelWarning  AlertLevel = "WARNING"  // 경고
    AlertLevelCritical AlertLevel = "CRITICAL" // 긴급
    AlertLevelEmergency AlertLevel = "EMERGENCY" // 응급
)

type Threshold struct {
    Level     AlertLevel `json:"level"`      // 알림 수준
    Operator  string     `json:"operator"`   // 비교 연산자 (>, <, >=, <=, ==)
    Value     float64    `json:"value"`      // 임계값
    Duration  time.Duration `json:"duration"` // 지속 시간 (임계값 초과 지속 기간)
    Message   string     `json:"message"`    // 알림 메시지 템플릿
}

// Metric 생성자 및 팩토리 메서드
func NewPerformanceMetric(
    name string,
    value float64,
    unit string,
    labels map[string]string,
    source MetricSource,
) (*Metric, error) {
    // 성능 메트릭을 생성합니다
    // - API 응답시간, 처리량, 에러율 등
    // - 기본 성능 임계값 설정
    // - 도메인 이벤트 생성 (PerformanceMetricCreated)
}

func NewBusinessMetric(
    name string,
    value float64,
    unit string,
    labels map[string]string,
    restaurantID *restaurantDomain.RestaurantID,
) (*Metric, error) {
    // 비즈니스 메트릭을 생성합니다
    // - 주문 수, 매출, 고객 만족도 등
    // - 매장별 메트릭 라벨 추가
    // - 도메인 이벤트 생성 (BusinessMetricCreated)
}

func NewSecurityMetric(
    name string,
    value float64,
    unit string,
    labels map[string]string,
    riskLevel RiskLevel,
) (*Metric, error) {
    // 보안 메트릭을 생성합니다
    // - 로그인 실패 횟수, 의심스러운 접근 등
    // - 위험도에 따른 알림 수준 설정
    // - 도메인 이벤트 생성 (SecurityMetricCreated)
}

// Metric 도메인 메서드들
func (m *Metric) ID() MetricID {
    // 메트릭 ID를 반환합니다
}

func (m *Metric) Name() string {
    // 메트릭 이름을 반환합니다
}

func (m *Metric) Value() float64 {
    // 메트릭 값을 반환합니다
}

func (m *Metric) UpdateValue(newValue float64) error {
    // 메트릭 값을 업데이트합니다 (GAUGE 타입만)
    // - 메트릭 타입 확인 (GAUGE만 업데이트 가능)
    // - 값 유효성 검증
    // - 임계값 확인 및 알림 수준 업데이트
    // - 도메인 이벤트 생성 (MetricValueUpdated)
}

func (m *Metric) IncrementCounter(delta float64) error {
    // 카운터 메트릭을 증가시킵니다 (COUNTER 타입만)
    // - 메트릭 타입 확인 (COUNTER만)
    // - 델타값 유효성 검증 (양수만)
    // - 누적값 계산
    // - 도메인 이벤트 생성 (CounterIncremented)
}

func (m *Metric) AddHistogramSample(sample float64) error {
    // 히스토그램에 샘플을 추가합니다 (HISTOGRAM 타입만)
    // - 메트릭 타입 확인 (HISTOGRAM만)
    // - 샘플값 유효성 검증
    // - 버킷별 집계 업데이트
    // - 분위수 계산 업데이트
    // - 도메인 이벤트 생성 (HistogramSampleAdded)
}

func (m *Metric) CheckThresholds() []ThresholdViolation {
    // 임계값 위반 여부를 확인합니다
    // - 설정된 모든 임계값과 현재 값 비교
    // - 위반 지속 시간 확인
    // - 위반된 임계값들 반환
}

func (m *Metric) UpdateAlertLevel() {
    // 현재 알림 수준을 업데이트합니다
    // - 임계값 위반 상황 분석
    // - 가장 높은 위반 수준으로 설정
    // - 알림 수준 변경 시 이벤트 생성
}

func (m *Metric) AddLabel(key, value string) {
    // 메트릭에 라벨을 추가합니다
    // - 기존 라벨 중복 확인
    // - 라벨 키/값 유효성 검증
    // - 라벨 추가 및 메타데이터 업데이트
}

func (m *Metric) RemoveLabel(key string) {
    // 메트릭에서 라벨을 제거합니다
    // - 라벨 존재 여부 확인
    // - 필수 라벨 보호 (제거 방지)
    // - 라벨 제거 및 메타데이터 업데이트
}

func (m *Metric) GetLabelValue(key string) (string, bool) {
    // 특정 라벨의 값을 반환합니다
    // - 라벨 존재 여부와 값 반환
}

func (m *Metric) MatchesLabels(labelSelector map[string]string) bool {
    // 라벨 셀렉터와 일치하는지 확인합니다
    // - 모든 셀렉터 라벨이 메트릭 라벨과 일치하는지 확인
    // - 메트릭 쿼리 및 필터링에 사용
}

func (m *Metric) IsStale(maxAge time.Duration) bool {
    // 메트릭이 오래된 것인지 확인합니다
    // - 현재 시간과 타임스탬프 비교
    // - 스탤 메트릭 정리에 사용
}

func (m *Metric) GetAge() time.Duration {
    // 메트릭 생성 후 경과 시간을 반환합니다
}

func (m *Metric) ShouldTriggerAlert() bool {
    // 알림 트리거 필요 여부를 확인합니다
    // - 임계값 위반 여부
    // - 알림 수준 변경 여부
    // - 알림 쿨다운 시간 고려
}

func (m *Metric) GenerateAlertMessage() string {
    // 알림 메시지를 생성합니다
    // - 임계값 정보 포함
    // - 현재 값과 임계값 비교
    // - 컨텍스트 정보 추가 (라벨, 소스 등)
}

// 도메인 이벤트 관리
func (m *Metric) DomainEvents() []DomainEvent {
    // 발생한 도메인 이벤트들을 반환합니다
}

func (m *Metric) ClearEvents() {
    // 처리 완료된 도메인 이벤트를 제거합니다
}
```

### **📊 Metrics Domain Services**

```go
// internal/metrics/domain/services.go
package domain

// MetricCollectionService는 메트릭 수집 로직을 담당합니다
type MetricCollectionService interface {
    // 성능 메트릭 자동 수집
    CollectPerformanceMetrics(ctx context.Context, source MetricSource) ([]Metric, error)
    
    // 비즈니스 메트릭 계산 및 수집
    CollectBusinessMetrics(ctx context.Context, restaurantID restaurantDomain.RestaurantID, timeRange TimeRange) ([]Metric, error)
    
    // 시스템 리소스 메트릭 수집
    CollectSystemMetrics(ctx context.Context) ([]Metric, error)
    
    // 사용자 경험 메트릭 수집
    CollectUXMetrics(ctx context.Context, userID userDomain.UserID) ([]Metric, error)
    
    // 실시간 메트릭 스트리밍
    StreamMetrics(ctx context.Context, filter MetricFilter) (<-chan Metric, error)
    
    // 메트릭 집계 (시간 윈도우별)
    AggregateMetrics(ctx context.Context, metrics []Metric, window AggregationWindow) ([]Metric, error)
}

// MetricAnalysisService는 메트릭 분석 로직을 담당합니다
type MetricAnalysisService interface {
    // 메트릭 트렌드 분석
    AnalyzeTrends(ctx context.Context, metricName string, timeRange TimeRange) (*TrendAnalysis, error)
    
    // 이상치 탐지
    DetectAnomalies(ctx context.Context, metrics []Metric) ([]AnomalyDetection, error)
    
    // 성능 병목 지점 식별
    IdentifyBottlenecks(ctx context.Context, performanceMetrics []Metric) (*BottleneckAnalysis, error)
    
    // 용량 계획 분석
    AnalyzeCapacityPlanning(ctx context.Context, resourceMetrics []Metric) (*CapacityPlanningReport, error)
    
    // 메트릭 상관관계 분석
    AnalyzeCorrelations(ctx context.Context, metricNames []string, timeRange TimeRange) (*CorrelationMatrix, error)
    
    // 예측 분석 (머신러닝 기반)
    PredictMetricValues(ctx context.Context, metricName string, forecastPeriod time.Duration) (*ForecastResult, error)
}

// AlertManagementService는 알림 관리 로직을 담당합니다
type AlertManagementService interface {
    // 알림 규칙 평가
    EvaluateAlertRules(ctx context.Context, metric *Metric, rules []AlertRule) ([]Alert, error)
    
    // 알림 억제 (중복 알림 방지)
    SuppressDuplicateAlerts(ctx context.Context, alerts []Alert, suppressionRules []SuppressionRule) ([]Alert, error)
    
    // 알림 에스컬레이션
    EscalateAlerts(ctx context.Context, alert Alert, escalationPolicy EscalationPolicy) error
    
    // 알림 그룹핑 (관련 알림들을 묶어서 처리)
    GroupRelatedAlerts(ctx context.Context, alerts []Alert, groupingRules []GroupingRule) ([]AlertGroup, error)
    
    // 알림 자동 해결
    AutoResolveAlerts(ctx context.Context, resolvedMetrics []Metric) ([]Alert, error)
}

// MetricVisualizationService는 메트릭 시각화를 담당합니다
type MetricVisualizationService interface {
    // 대시보드 메트릭 준비
    PrepareDashboardMetrics(ctx context.Context, dashboardConfig DashboardConfig) (*DashboardData, error)
    
    // 실시간 차트 데이터 생성
    GenerateChartData(ctx context.Context, chartConfig ChartConfig) (*ChartData, error)
    
    // 메트릭 요약 생성
    GenerateMetricSummary(ctx context.Context, metricNames []string, timeRange TimeRange) (*MetricSummary, error)
    
    // 성능 리포트 생성
    GeneratePerformanceReport(ctx context.Context, reportConfig ReportConfig) (*PerformanceReport, error)
    
    // 커스텀 메트릭 쿼리 실행
    ExecuteMetricQuery(ctx context.Context, query MetricQuery) (*QueryResult, error)
}
```

### **📊 Metrics Repository Interface**

```go
// internal/metrics/domain/repository.go
package domain

// MetricRepository는 메트릭 데이터 영속성을 담당합니다
type MetricRepository interface {
    // 메트릭 저장 (시계열 데이터)
    Save(ctx context.Context, metric *Metric) error
    
    // 메트릭 일괄 저장 (성능 최적화)
    SaveBatch(ctx context.Context, metrics []Metric) error
    
    // 메트릭 이름으로 조회
    GetByName(ctx context.Context, name string, timeRange TimeRange) ([]Metric, error)
    
    // 라벨 셀렉터로 메트릭 조회
    GetByLabels(ctx context.Context, labelSelector map[string]string, timeRange TimeRange) ([]Metric, error)
    
    // 카테고리별 메트릭 조회
    GetByCategory(ctx context.Context, category MetricCategory, timeRange TimeRange) ([]Metric, error)
    
    // 소스별 메트릭 조회
    GetBySource(ctx context.Context, source MetricSource, timeRange TimeRange) ([]Metric, error)
    
    // 시간 범위별 메트릭 조회 (성능 최적화)
    GetByTimeRange(ctx context.Context, startTime, endTime time.Time, filter MetricFilter) ([]Metric, error)
    
    // 메트릭 집계 쿼리 (AVG, SUM, MAX, MIN, COUNT)
    AggregateMetrics(ctx context.Context, aggregation AggregationQuery) (*AggregationResult, error)
    
    // 분위수 계산 (P50, P95, P99)
    CalculatePercentiles(ctx context.Context, metricName string, percentiles []float64, timeRange TimeRange) ([]PercentileResult, error)
    
    // 메트릭 검색 (이름, 라벨 기반)
    SearchMetrics(ctx context.Context, searchCriteria MetricSearchCriteria) ([]Metric, error)
    
    // 메트릭 메타데이터 조회 (스키마 정보)
    GetMetricMetadata(ctx context.Context, metricName string) (*MetricMetadata, error)
    
    // 고유 라벨값 조회 (필터링용)
    GetUniqueLabelValues(ctx context.Context, labelKey string) ([]string, error)
    
    // 메트릭 이름 목록 조회
    GetMetricNames(ctx context.Context, filter MetricNameFilter) ([]string, error)
    
    // 오래된 메트릭 정리 (보관 정책 적용)
    DeleteOldMetrics(ctx context.Context, retentionPolicy MetricRetentionPolicy) (int64, error)
    
    // 메트릭 내보내기 (백업, 분석용)
    ExportMetrics(ctx context.Context, exportCriteria MetricExportCriteria) (*MetricExportResult, error)
    
    // 메트릭 통계 정보
    GetStatistics(ctx context.Context, timeRange TimeRange) (*MetricStatistics, error)
}

// AlertRepository는 알림 데이터 영속성을 담당합니다
type AlertRepository interface {
    // 알림 저장
    Save(ctx context.Context, alert *Alert) error
    
    // 알림 조회
    GetByID(ctx context.Context, alertID string) (*Alert, error)
    
    // 활성 알림 목록 조회
    GetActiveAlerts(ctx context.Context, filter AlertFilter) ([]Alert, error)
    
    // 알림 상태 업데이트
    UpdateStatus(ctx context.Context, alertID string, status AlertStatus) error
    
    // 알림 해결 처리
    ResolveAlert(ctx context.Context, alertID string, resolvedAt time.Time, resolvedBy string) error
    
    // 알림 이력 조회
    GetAlertHistory(ctx context.Context, filter AlertHistoryFilter) ([]Alert, error)
    
    // 알림 통계
    GetAlertStatistics(ctx context.Context, timeRange TimeRange) (*AlertStatistics, error)
    
    // 알림 삭제
    Delete(ctx context.Context, alertID string) error
}

// MetricFilter는 메트릭 조회 시 사용하는 필터입니다
type MetricFilter struct {
    Names        []string            `json:"names"`
    Categories   []MetricCategory    `json:"categories"`
    Sources      []MetricSource      `json:"sources"`
    Labels       map[string]string   `json:"labels"`
    AlertLevels  []AlertLevel        `json:"alert_levels"`
    
    // 시간 범위
    StartTime    *time.Time          `json:"start_time"`
    EndTime      *time.Time          `json:"end_time"`
    
    // 값 범위
    MinValue     *float64            `json:"min_value"`
    MaxValue     *float64            `json:"max_value"`
    
    // 페이징 및 정렬
    Limit        int                 `json:"limit" validate:"min=1,max=10000"`
    Offset       int                 `json:"offset" validate:"min=0"`
    SortBy       string              `json:"sort_by" validate:"oneof=timestamp value name"`
    SortOrder    string              `json:"sort_order" validate:"oneof=asc desc"`
}
```

-----

## 🔔 **Notification Domain**

실시간 알림 및 메시지 발송을 담당하는 도메인입니다.

### **📮 Notification Aggregate**

```go
// internal/notification/domain/notification.go
package domain

import (
    "context"
    "time"
    userDomain "github.com/ordersync/internal/user/domain"
    "github.com/ordersync/internal/encryption/domain"
)

// Notification은 알림을 나타내는 Aggregate Root입니다
type Notification struct {
    // 기본 식별자
    id          NotificationID
    templateID  *NotificationTemplateID // 사용된 템플릿 ID
    
    // 발송 대상 (암호화)
    recipientID   userDomain.UserID            // 수신자 사용자 ID
    recipientInfo encryption.EncryptedString   // 수신자 연락처 (이메일/전화번호)
    
    // 알림 내용 (암호화)
    title   encryption.EncryptedString // 알림 제목
    content encryption.EncryptedString // 알림 내용
    
    // 알림 유형 및 채널
    type_    NotificationType  // 알림 유형
    channel  NotificationChannel // 발송 채널
    priority NotificationPriority // 우선순위
    
    // 발송 상태
    status       NotificationStatus
    statusHistory []NotificationStatusChange
    
    // 시간 정보
    scheduledAt time.Time  // 발송 예정 시간
    sentAt      *time.Time // 실제 발송 시간
    deliveredAt *time.Time // 배달 확인 시간
    readAt      *time.Time // 읽음 확인 시간
    expiresAt   *time.Time // 만료 시간
    
    // 발송 결과
    deliveryResult *DeliveryResult // 발송 결과 상세
    
    // 재시도 정보
    retryCount    int              // 재시도 횟수
    maxRetries    int              // 최대 재시도 횟수
    retryInterval time.Duration    // 재시도 간격
    
    // 메타데이터
    metadata map[string]interface{} // 추가 메타데이터
    tags     []string               // 태그 (분류, 필터링용)
    
    // 도메인 메타데이터
    createdAt time.Time
    updatedAt time.Time
    version   int64
    events    []DomainEvent
}

type NotificationID string
type NotificationTemplateID string

type NotificationType string
const (
    NotificationTypeOrder    NotificationType = "ORDER"    // 주문 관련
    NotificationTypePayment  NotificationType = "PAYMENT"  // 결제 관련
    NotificationTypeAuth     NotificationType = "AUTH"     // 인증 관련
    NotificationTypeMarketing NotificationType = "MARKETING" // 마케팅
    NotificationTypeSystem   NotificationType = "SYSTEM"   // 시스템 공지
    NotificationTypeSecurity NotificationType = "SECURITY" // 보안 알림
    NotificationTypePromotion NotificationType = "PROMOTION" // 프로모션
)

type NotificationChannel string
const (
    NotificationChannelPush    NotificationChannel = "PUSH"    // 푸시 알림
    NotificationChannelEmail   NotificationChannel = "EMAIL"   // 이메일
    NotificationChannelSMS     NotificationChannel = "SMS"     // 문자메시지
    NotificationChannelWebSocket NotificationChannel = "WEBSOCKET" // 웹소켓 (실시간)
    NotificationChannelInApp   NotificationChannel = "IN_APP"  // 앱 내 알림
)

type NotificationPriority string
const (
    NotificationPriorityLow      NotificationPriority = "LOW"      // 낮음
    NotificationPriorityNormal   NotificationPriority = "NORMAL"   // 보통
    NotificationPriorityHigh     NotificationPriority = "HIGH"     // 높음
    NotificationPriorityCritical NotificationPriority = "CRITICAL" // 긴급
)

type NotificationStatus string
const (
    NotificationStatusPending    NotificationStatus = "PENDING"    // 발송 대기
    NotificationStatusScheduled  NotificationStatus = "SCHEDULED"  // 예약됨
    NotificationStatusSending    NotificationStatus = "SENDING"    // 발송 중
    NotificationStatusSent       NotificationStatus = "SENT"       // 발송됨
    NotificationStatusDelivered  NotificationStatus = "DELIVERED"  // 배달됨
    NotificationStatusRead       NotificationStatus = "READ"       // 읽음
    NotificationStatusFailed     NotificationStatus = "FAILED"     // 실패
    NotificationStatusExpired    NotificationStatus = "EXPIRED"    // 만료됨
    NotificationStatusCancelled  NotificationStatus = "CANCELLED"  // 취소됨
)

type NotificationStatusChange struct {
    FromStatus NotificationStatus `json:"from_status"`
    ToStatus   NotificationStatus `json:"to_status"`
    Timestamp  time.Time          `json:"timestamp"`
    Reason     string             `json:"reason"`
    Metadata   map[string]interface{} `json:"metadata"`
}

type DeliveryResult struct {
    Success       bool                   `json:"success"`
    ErrorCode     string                 `json:"error_code"`     // 에러 코드
    ErrorMessage  string                 `json:"error_message"`  // 에러 메시지
    ExternalID    string                 `json:"external_id"`    // 외부 서비스 ID
    ResponseTime  time.Duration          `json:"response_time"`  // 응답 시간
    Metadata      map[string]interface{} `json:"metadata"`       // 추가 정보
}

// Notification 생성자 및 팩토리 메서드
func NewNotification(
    recipientID userDomain.UserID,
    recipientInfo string, // 이메일 또는 전화번호
    title, content string,
    type_ NotificationType,
    channel NotificationChannel,
    priority NotificationPriority,
    scheduledAt time.Time,
    encryptionService encryption.Service,
) (*Notification, error) {
    // 새로운 알림을 생성합니다
    // - 수신자 정보 검증 (이메일/전화번호 형식)
    // - 제목과 내용 암호화
    // - 수신자 연락처 정보 암호화
    // - 발송 예정 시간 유효성 검증
    // - 우선순위에 따른 만료 시간 설정
    // - 도메인 이벤트 생성 (NotificationCreated)
}

func NewFromTemplate(
    templateID NotificationTemplateID,
    recipientID userDomain.UserID,
    recipientInfo string,
    templateVariables map[string]string,
    scheduledAt time.Time,
    encryptionService encryption.Service,
) (*Notification, error) {
    // 템플릿을 사용하여 알림을 생성합니다
    // - 템플릿 존재 여부 확인
    // - 템플릿 변수 치환
    // - 템플릿 설정에 따른 채널 및 우선순위 적용
    // - 알림 생성 (위와 동일한 프로세스)
}

// Notification 도메인 메서드들
func (n *Notification) ID() NotificationID {
    // 알림 ID를 반환합니다
}

func (n *Notification) RecipientID() userDomain.UserID {
    // 수신자 ID를 반환합니다
}

func (n *Notification) Status() NotificationStatus {
    // 현재 알림 상태를 반환합니다
}

func (n *Notification) Send(deliveryService NotificationDeliveryService) error {
    // 알림을 발송합니다
    // - 발송 가능 상태 확인 (PENDING, SCHEDULED)
    // - 현재 시간이 예정 시간 이후인지 확인
    // - 상태를 SENDING으로 변경
    // - 외부 발송 서비스 호출
    // - 발송 결과에 따라 상태 업데이트
    // - 도메인 이벤트 생성 (NotificationSent 또는 NotificationFailed)
}

func (n *Notification) MarkAsDelivered(deliveredAt time.Time, externalID string) error {
    // 알림 배달 완료로 표시합니다
    // - 현재 상태 확인 (SENT -> DELIVERED)
    // - 배달 시간 기록
    // - 외부 서비스 ID 기록
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (NotificationDelivered)
}

func (n *Notification) MarkAsRead(readAt time.Time) error {
    // 알림 읽음 처리합니다
    // - 현재 상태 확인 (DELIVERED -> READ)
    // - 읽은 시간 기록
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (NotificationRead)
}

func (n *Notification) MarkAsFailed(errorCode, errorMessage string, responseTime time.Duration) error {
    // 알림 발송 실패로 표시합니다
    // - 실패 상태로 변경
    // - 에러 정보 기록
    // - 재시도 가능 여부 확인
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (NotificationFailed)
}

func (n *Notification) Schedule(newScheduledAt time.Time) error {
    // 알림 발송 시간을 예약/변경합니다
    // - 현재 상태 확인 (PENDING 또는 SCHEDULED만 가능)
    // - 예약 시간 유효성 검증 (미래 시간)
    // - 만료 시간 재계산
    // - 상태를 SCHEDULED로 변경
    // - 도메인 이벤트 생성 (NotificationScheduled)
}

func (n *Notification) Cancel(reason string) error {
    // 알림 발송을 취소합니다
    // - 취소 가능 상태 확인 (SENT, DELIVERED, READ 제외)
    // - 취소 사유 기록
    // - 상태를 CANCELLED로 변경
    // - 상태 변경 이력 추가
    // - 도메인 이벤트 생성 (NotificationCancelled)
}

func (n *Notification) CanRetry() bool {
    // 재시도 가능 여부를 확인합니다
    // - 현재 상태 확인 (FAILED 상태만)
    // - 최대 재시도 횟수 확인
    // - 재시도 간격 확인
    // - 만료 시간 확인
}

func (n *Notification) Retry() error {
    // 알림 재발송을 시도합니다
    // - 재시도 가능 여부 확인
    // - 재시도 횟수 증가
    // - 상태를 PENDING으로 변경
    // - 다음 발송 시간 계산 (백오프 전략 적용)
    // - 도메인 이벤트 생성 (NotificationRetryScheduled)
}

func (n *Notification) IsExpired() bool {
    // 알림 만료 여부를 확인합니다
    // - 만료 시간과 현재 시간 비교
    // - 만료된 경우 자동으로 상태 변경
}

func (n *Notification) GetTimeUntilExpiry() *time.Duration {
    // 만료까지 남은 시간을 반환합니다
    // - 만료 시간이 설정되지 않은 경우 nil 반환
    // - 이미 만료된 경우 0 반환
}

func (n *Notification) UpdatePriority(newPriority NotificationPriority, reason string) error {
    // 알림 우선순위를 변경합니다
    // - 우선순위 변경 가능 상태 확인 (SENT, DELIVERED, READ 제외)
    // - 우선순위에 따른 발송 순서 재조정
    // - 변경 사유 기록
    // - 도메인 이벤트 생성 (NotificationPriorityUpdated)
}

func (n *Notification) AddTag(tag string) {
    // 알림에 태그를 추가합니다
    // - 중복 태그 확인
    // - 태그 유효성 검증
    // - 태그 목록에 추가
}

func (n *Notification) RemoveTag(tag string) {
    // 알림에서 태그를 제거합니다
    // - 태그 존재 여부 확인
    // - 태그 목록에서 제거
}

func (n *Notification) HasTag(tag string) bool {
    // 특정 태그 보유 여부를 확인합니다
}

func (n *Notification) GetDecryptedTitle(decryptionService encryption.Service) (string, error) {
    // 암호화된 제목을 복호화하여 반환합니다
    // - 접근 권한 확인
    // - 제목 복호화
    // - 감사 로그 기록
}

func (n *Notification) GetDecryptedContent(decryptionService encryption.Service) (string, error) {
    // 암호화된 내용을 복호화하여 반환합니다
    // - 접근 권한 확인
    // - 내용 복호화
    // - 감사 로그 기록
}

func (n *Notification) GetDecryptedRecipientInfo(decryptionService encryption.Service) (string, error) {
    // 암호화된 수신자 정보를 복호화하여 반환합니다
    // - 접근 권한 확인 (관리자 또는 본인만)
    // - 수신자 정보 복호화
    // - 감사 로그 기록
}

// 도메인 이벤트 관리
func (n *Notification) DomainEvents() []DomainEvent {
    // 발생한 도메인 이벤트들을 반환합니다
}

func (n *Notification) ClearEvents() {
    // 처리 완료된 도메인 이벤트를 제거합니다
}
```

### **📋 Notification Template Aggregate**

```go
// internal/notification/domain/template.go
package domain

// NotificationTemplate은 알림 템플릿을 나타내는 Aggregate Root입니다
type NotificationTemplate struct {
    // 기본 식별자
    id   NotificationTemplateID
    name string // 템플릿 이름 (고유)
    
    // 템플릿 분류
    type_    NotificationType    // 알림 유형
    category TemplateCategory    // 템플릿 카테고리
    
    // 템플릿 내용 (암호화)
    titleTemplate   encryption.EncryptedString // 제목 템플릿
    contentTemplate encryption.EncryptedString // 내용 템플릿
    
    // 채널별 설정
    channels []ChannelConfig // 지원하는 채널들과 설정
    
    // 템플릿 변수
    variables []TemplateVariable // 템플릿에서 사용하는 변수들
    
    // 발송 설정
    defaultPriority NotificationPriority // 기본 우선순위
    defaultExpiry   *time.Duration       // 기본 만료 시간
    
    // 재시도 설정
    retryConfig RetryConfig // 재시도 정책
    
    // 템플릿 상태
    isActive bool   // 활성화 여부
    version  string // 템플릿 버전
    
    // 국제화 지원
    localizations map[string]LocalizedTemplate // 언어별 템플릿
    
    // 메타데이터
    description string                 // 템플릿 설명
    tags        []string               // 태그
    metadata    map[string]interface{} // 추가 메타데이터
    
    // 도메인 메타데이터
    createdAt time.Time
    updatedAt time.Time
    createdBy userDomain.UserID
    updatedBy userDomain.UserID
    version   int64
    events    []DomainEvent
}

type TemplateCategory string
const (
    TemplateCategoryTransactional TemplateCategory = "TRANSACTIONAL" // 거래 관련
    TemplateCategoryMarketing     TemplateCategory = "MARKETING"     // 마케팅
    TemplateCategorySystem        TemplateCategory = "SYSTEM"        // 시스템
    TemplateCategoryAlert         TemplateCategory = "ALERT"         // 알림
)

type ChannelConfig struct {
    Channel     NotificationChannel `json:"channel"`
    IsEnabled   bool                `json:"is_enabled"`
    Priority    NotificationPriority `json:"priority"`
    Template    ChannelTemplate     `json:"template"` // 채널별 템플릿 설정
}

type ChannelTemplate struct {
    Subject     *string `json:"subject"`      // 이메일 제목 (이메일만)
    Header      *string `json:"header"`       // 헤더 텍스트
    Footer      *string `json:"footer"`       // 푸터 텍스트
    ButtonText  *string `json:"button_text"`  // 버튼 텍스트
    ButtonURL   *string `json:"button_url"`   // 버튼 URL
    ImageURL    *string `json:"image_url"`    // 이미지 URL
}

type TemplateVariable struct {
    Name        string `json:"name"`         // 변수명
    Type        string `json:"type"`         // 변수 타입 (string, number, date, etc.)
    Required    bool   `json:"required"`     // 필수 여부
    DefaultValue *string `json:"default_value"` // 기본값
    Description string `json:"description"`  // 설명
    Validation  *VariableValidation `json:"validation"` // 검증 규칙
}

type VariableValidation struct {
    MinLength   *int     `json:"min_length"`   // 최소 길이
    MaxLength   *int     `json:"max_length"`   // 최대 길이
    Pattern     *string  `json:"pattern"`      // 정규식 패턴
    AllowedValues []string `json:"allowed_values"` // 허용값 목록
}

type RetryConfig struct {
    MaxRetries    int           `json:"max_retries"`    // 최대 재시도 횟수
    InitialDelay  time.Duration `json:"initial_delay"`  // 초기 지연 시간
    BackoffFactor float64       `json:"backoff_factor"` // 백오프 계수
    MaxDelay      time.Duration `json:"max_delay"`      // 최대 지연 시간
}

type LocalizedTemplate struct {
    Language        string                  `json:"language"`         // 언어 코드 (ko, en, etc.)
    TitleTemplate   encryption.EncryptedString `json:"title_template"`   // 현지화된 제목 템플릿
    ContentTemplate encryption.EncryptedString `json:"content_template"` // 현지화된 내용 템플릿
    ChannelConfigs  []ChannelConfig         `json:"channel_configs"`  // 현지화된 채널 설정
}

// NotificationTemplate 생성자 및 팩토리 메서드
func NewNotificationTemplate(
    name string,
    type_ NotificationType,
    titleTemplate, contentTemplate string,
    channels []ChannelConfig,
    variables []TemplateVariable,
    createdBy userDomain.UserID,
    encryptionService encryption.Service,
) (*NotificationTemplate, error) {
    // 새로운 알림 템플릿을 생성합니다
    // - 템플릿 이름 중복 확인
    // - 제목/내용 템플릿 암호화
    // - 템플릿 변수 유효성 검증
    // - 채널 설정 유효성 검증
    // - 도메인 이벤트 생성 (TemplateCreated)
}

// NotificationTemplate 도메인 메서드들
func (t *NotificationTemplate) ID() NotificationTemplateID {
    // 템플릿 ID를 반환합니다
}

func (t *NotificationTemplate) Name() string {
    // 템플릿 이름을 반환합니다
}

func (t *NotificationTemplate) IsActive() bool {
    // 템플릿 활성화 상태를 반환합니다
}

func (t *NotificationTemplate) RenderTitle(variables map[string]string, language string, decryptionService encryption.Service) (string, error) {
    // 템플릿 변수를 사용하여 제목을 렌더링합니다
    // - 언어별 템플릿 선택 (기본값: 영어)
    // - 암호화된 템플릿 복호화
    // - 필수 변수 존재 여부 확인
    // - 변수 유효성 검증
    // - 템플릿 엔진을 통한 렌더링
}

func (t *NotificationTemplate) RenderContent(variables map[string]string, language string, decryptionService encryption.Service) (string, error) {
    // 템플릿 변수를 사용하여 내용을 렌더링합니다
    // - 언어별 템플릿 선택
    // - 암호화된 템플릿 복호화
    // - 변수 치환 및 렌더링
    // - HTML/텍스트 형식 처리
}

func (t *NotificationTemplate) ValidateVariables(variables map[string]string) []VariableError {
    // 제공된 변수들의 유효성을 검증합니다
    // - 필수 변수 누락 확인
    // - 변수 타입 및 형식 검증
    // - 허용값 범위 확인
    // - 검증 오류 목록 반환
}

func (t *NotificationTemplate) AddChannel(config ChannelConfig) error {
    // 새로운 채널 설정을 추가합니다
    // - 중복 채널 확인
    // - 채널 설정 유효성 검증
    // - 채널 목록에 추가
    // - 도메인 이벤트 생성 (TemplateChannelAdded)
}

func (t *NotificationTemplate) RemoveChannel(channel NotificationChannel) error {
    // 채널 설정을 제거합니다
    // - 채널 존재 여부 확인
    // - 최소 채널 요구사항 확인 (적어도 1개 채널 유지)
    // - 채널 목록에서 제거
    // - 도메인 이벤트 생성 (TemplateChannelRemoved)
}

func (t *NotificationTemplate) UpdateChannel(channel NotificationChannel, config ChannelConfig) error {
    // 채널 설정을 업데이트합니다
    // - 채널 존재 여부 확인
    // - 새 설정 유효성 검증
    // - 설정 업데이트
    // - 도메인 이벤트 생성 (TemplateChannelUpdated)
}

func (t *NotificationTemplate) AddVariable(variable TemplateVariable) error {
    // 새로운 템플릿 변수를 추가합니다
    // - 변수명 중복 확인
    // - 변수 유효성 검증
    // - 변수 목록에 추가
    // - 도메인 이벤트 생성 (TemplateVariableAdded)
}

func (t *NotificationTemplate) RemoveVariable(variableName string) error {
    // 템플릿 변수를 제거합니다
    // - 변수 사용 여부 확인 (템플릿에서 참조 중인지)
    // - 변수 목록에서 제거
    // - 도메인 이벤트 생성 (TemplateVariableRemoved)
}

func (t *NotificationTemplate) AddLocalization(language string, localized LocalizedTemplate, encryptionService encryption.Service) error {
    // 언어별 현지화 템플릿을 추가합니다
    // - 언어 코드 유효성 검증
    // - 현지화된 템플릿 암호화
    // - 현지화 데이터 저장
    // - 도메인 이벤트 생성 (TemplateLocalizationAdded)
}

func (t *NotificationTemplate) UpdateRetryConfig(config RetryConfig) error {
    // 재시도 설정을 업데이트합니다
    // - 설정값 유효성 검증
    // - 백오프 전략 유효성 확인
    // - 설정 업데이트
    // - 도메인 이벤트 생성 (TemplateRetryConfigUpdated)
}

func (t *NotificationTemplate) Activate() error {
    // 템플릿을 활성화합니다
    // - 필수 설정 완성도 확인
    // - 최소 1개 채널 활성화 확인
    // - 활성화 상태로 변경
    // - 도메인 이벤트 생성 (TemplateActivated)
}

func (t *NotificationTemplate) Deactivate(reason string) error {
    // 템플릿을 비활성화합니다
    // - 사용 중인 알림 확인
    // - 비활성화 상태로 변경
    // - 비활성화 사유 기록
    // - 도메인 이벤트 생성 (TemplateDeactivated)
}

func (t *NotificationTemplate) CreateNewVersion(updatedBy userDomain.UserID) (*NotificationTemplate, error) {
    // 템플릿의 새 버전을 생성합니다
    // - 현재 템플릿을 복사
    // - 버전 번호 증가
    // - 수정자 정보 업데이트
    // - 새 버전 반환
}

func (t *NotificationTemplate) GetSupportedChannels() []NotificationChannel {
    // 지원하는 채널 목록을 반환합니다
    // - 활성화된 채널만 필터링
}

func (t *NotificationTemplate) GetSupportedLanguages() []string {
    // 지원하는 언어 목록을 반환합니다
    // - 현지화 데이터가 있는 언어만 반환
}

func (t *NotificationTemplate) EstimateDeliveryCost(channel NotificationChannel, recipientCount int) (*DeliveryCostEstimate, error) {
    // 예상 발송 비용을 계산합니다
    // - 채널별 단가 적용
    // - 수신자 수량 고려
    // - 우선순위별 추가 비용 계산
}

// 도메인 이벤트 관리
func (t *NotificationTemplate) DomainEvents() []DomainEvent {
    // 발생한 도메인 이벤트들을 반환합니다
}

func (t *NotificationTemplate) ClearEvents() {
    // 처리 완료된 도메인 이벤트를 제거합니다
}
```

### **🔔 Notification Domain Services**

```go
// internal/notification/domain/services.go
package domain

// NotificationDeliveryService는 실제 알림 발송을 담당합니다
type NotificationDeliveryService interface {
    // 푸시 알림 발송
    SendPushNotification(ctx context.Context, notification *Notification) (*DeliveryResult, error)
    
    // 이메일 발송
    SendEmail(ctx context.Context, notification *Notification) (*DeliveryResult, error)
    
    // SMS 발송
    SendSMS(ctx context.Context, notification *Notification) (*DeliveryResult, error)
    
    // 웹소켓 실시간 알림
    SendWebSocketNotification(ctx context.Context, notification *Notification) (*DeliveryResult, error)
    
    // 인앱 알림
    SendInAppNotification(ctx context.Context, notification *Notification) (*DeliveryResult, error)
    
    // 발송 상태 확인 (외부 서비스)
    CheckDeliveryStatus(ctx context.Context, externalID string, channel NotificationChannel) (*DeliveryStatus, error)
    
    // 발송 가능 여부 확인
    CanDeliverTo(ctx context.Context, recipientInfo string, channel NotificationChannel) (bool, error)
}

// NotificationSchedulingService는 알림 스케줄링을 담당합니다
type NotificationSchedulingService interface {
    // 알림 스케줄링
    ScheduleNotification(ctx context.Context, notification *Notification) error
    
    // 스케줄된 알림 실행
    ProcessScheduledNotifications(ctx context.Context, batchSize int) ([]NotificationID, error)
    
    // 대량 알림 스케줄링 (배치 처리)
    ScheduleBulkNotifications(ctx context.Context, notifications []*Notification) (*BulkScheduleResult, error)
    
    // 스케줄 취소
    CancelScheduledNotification(ctx context.Context, notificationID NotificationID) error
    
    // 스케줄 변경
    RescheduleNotification(ctx context.Context, notificationID NotificationID, newTime time.Time) error
    
    // 반복 알림 스케줄링
    ScheduleRecurringNotification(ctx context.Context, notification *Notification, recurrence RecurrencePattern) error
}

// NotificationPreferenceService는 사용자 알림 설정을 관리합니다
type NotificationPreferenceService interface {
    // 사용자 알림 설정 조회
    GetUserPreferences(ctx context.Context, userID userDomain.UserID) (*NotificationPreferences, error)
    
    // 알림 설정 업데이트
    UpdatePreferences(ctx context.Context, userID userDomain.UserID, preferences *NotificationPreferences) error
    
    // 채널별 수신 동의 확인
    IsChannelEnabled(ctx context.Context, userID userDomain.UserID, channel NotificationChannel) (bool, error)
    
    // 알림 유형별 수신 동의 확인
    IsTypeEnabled(ctx context.Context, userID userDomain.UserID, type_ NotificationType) (bool, error)
    
    // 조용한 시간 확인 (Do Not Disturb)
    IsQuietHours(ctx context.Context, userID userDomain.UserID, timestamp time.Time) (bool, error)
    
    // 수신 거부 처리 (Opt-out)
    OptOutFromNotifications(ctx context.Context, userID userDomain.UserID, types []NotificationType) error
    
    // 수신 동의 복구 (Opt-in)
    OptInToNotifications(ctx context.Context, userID userDomain.UserID, types []NotificationType) error
}

// NotificationAnalyticsService는 알림 분석을 담당합니다
type NotificationAnalyticsService interface {
    // 발송 통계 분석
    AnalyzeDeliveryStatistics(ctx context.Context, criteria AnalyticsCriteria) (*DeliveryStatistics, error)
    
    // 채널별 성과 분석
    AnalyzeChannelPerformance(ctx context.Context, timeRange TimeRange) (*ChannelPerformanceReport, error)
    
    // 사용자 참여도 분석
    AnalyzeEngagementMetrics(ctx context.Context, userID userDomain.UserID, timeRange TimeRange) (*EngagementAnalysis, error)
    
    // 템플릿 효과성 분석
    AnalyzeTemplateEffectiveness(ctx context.Context, templateID NotificationTemplateID, timeRange TimeRange) (*TemplateAnalytics, error)
    
    // 최적 발송 시간 분석
    AnalyzeOptimalSendTimes(ctx context.Context, userSegment UserSegment) (*OptimalTimingReport, error)
    
    // A/B 테스트 결과 분석
    AnalyzeABTestResults(ctx context.Context, testID string) (*ABTestResult, error)
}
```

### **📊 Notification Repository Interface**

```go
// internal/notification/domain/repository.go
package domain

// NotificationRepository는 알림 데이터 영속성을 담당합니다
type NotificationRepository interface {
    // 알림 생성
    Create(ctx context.Context, notification *Notification) error
    
    // 알림 일괄 생성 (대량 발송용)
    CreateBatch(ctx context.Context, notifications []*Notification) error
    
    // ID로 알림 조회
    GetByID(ctx context.Context, id NotificationID) (*Notification, error)
    
    // 수신자별 알림 목록 조회
    GetByRecipientID(ctx context.Context, recipientID userDomain.UserID, filter NotificationFilter) (*NotificationList, error)
    
    // 상태별 알림 조회 (배치 처리용)
    GetByStatus(ctx context.Context, status NotificationStatus, limit int) ([]*Notification, error)
    
    // 예약된 알림 조회 (스케줄러용)
    GetScheduledNotifications(ctx context.Context, beforeTime time.Time, limit int) ([]*Notification, error)
    
    // 재시도 대상 알림 조회
    GetRetryableNotifications(ctx context.Context, limit int) ([]*Notification, error)
    
    // 만료된 알림 조회 (정리용)
    GetExpiredNotifications(ctx context.Context, limit int) ([]*Notification, error)
    
    // 채널별 알림 조회
    GetByChannel(ctx context.Context, channel NotificationChannel, filter NotificationFilter) ([]*Notification, error)
    
    // 우선순위별 알림 조회 (발송 큐 관리용)
    GetByPriority(ctx context.Context, priority NotificationPriority, filter NotificationFilter) ([]*Notification, error)
    
    // 알림 정보 업데이트
    Update(ctx context.Context, notification *Notification) error
    
    // 알림 상태 일괄 업데이트
    UpdateStatusBatch(ctx context.Context, notificationIDs []NotificationID, newStatus NotificationStatus) error
    
    // 알림 삭제 (보관 기간 만료 후)
    Delete(ctx context.Context, id NotificationID) error
    
    // 알림 검색 (제목, 내용 기반)
    Search(ctx context.Context, criteria NotificationSearchCriteria) (*NotificationSearchResult, error)
    
    // 알림 통계 조회
    GetStatistics(ctx context.Context, criteria StatisticsCriteria) (*NotificationStatistics, error)
    
    // 발송 성과 리포트
    GetDeliveryReport(ctx context.Context, timeRange TimeRange, groupBy GroupByDimension) (*DeliveryReport, error)
}

// NotificationTemplateRepository는 알림 템플릿 데이터 영속성을 담당합니다
type NotificationTemplateRepository interface {
    // 템플릿 생성
    Create(ctx context.Context, template *NotificationTemplate) error
    
    // ID로 템플릿 조회
    GetByID(ctx context.Context, id NotificationTemplateID) (*NotificationTemplate, error)
    
    // 이름으로 템플릿 조회
    GetByName(ctx context.Context, name string) (*NotificationTemplate, error)
    
    // 유형별 템플릿 목록 조회
    GetByType(ctx context.Context, type_ NotificationType) ([]*NotificationTemplate, error)
    
    // 활성 템플릿 목록 조회
    GetActiveTemplates(ctx context.Context) ([]*NotificationTemplate, error)
    
    // 템플릿 검색
    Search(ctx context.Context, criteria TemplateSearchCriteria) ([]*NotificationTemplate, error)
    
    // 템플릿 업데이트
    Update(ctx context.Context, template *NotificationTemplate) error
    
    // 템플릿 삭제
    Delete(ctx context.Context, id NotificationTemplateID) error
    
    // 템플릿 사용 통계
    GetUsageStatistics(ctx context.Context, templateID NotificationTemplateID, timeRange TimeRange) (*TemplateUsageStats, error)
}

// NotificationFilter는 알림 조회 시 사용하는 필터입니다
type NotificationFilter struct {
    Types       []NotificationType    `json:"types"`
    Channels    []NotificationChannel `json:"channels"`
    Priorities  []NotificationPriority `json:"priorities"`
    Statuses    []NotificationStatus   `json:"statuses"`
    Tags        []string               `json:"tags"`
    
    // 시간 범위
    CreatedAfter  *time.Time `json:"created_after"`
    CreatedBefore *time.Time `json:"created_before"`
    ScheduledAfter *time.Time `json:"scheduled_after"`
    ScheduledBefore *time.Time `json:"scheduled_before"`
    
    // 페이징 및 정렬
    Page      int    `json:"page" validate:"min=1"`
    Limit     int    `json:"limit" validate:"min=1,max=1000"`
    SortBy    string `json:"sort_by" validate:"oneof=created_at scheduled_at priority status"`
    SortOrder string `json:"sort_order" validate:"oneof=asc desc"`
}

// NotificationList는 페이징된 알림 목록을 나타냅니다
type NotificationList struct {
    Items      []*Notification `json:"items"`
    TotalCount int64           `json:"total_count"`
    Page       int             `json:"page"`
    Limit      int             `json:"limit"`
    HasNext    bool            `json:"has_next"`
}
```

-----

## 🔄 **도메인 이벤트 시스템**

모든 도메인에서 발생하는 이벤트들을 정의하고 처리하는 시스템입니다.

### **📡 Domain Events**

```go
// internal/shared/domain/events.go
package domain

import (
    "time"
    userDomain "github.com/ordersync/internal/user/domain"
    restaurantDomain "github.com/ordersync/internal/restaurant/domain"
    orderDomain "github.com/ordersync/internal/order/domain"
    paymentDomain "github.com/ordersync/internal/payment/domain"
)

// DomainEvent는 모든 도메인 이벤트가 구현해야 하는 인터페이스입니다
type DomainEvent interface {
    // 이벤트 고유 식별자
    ID() EventID
    
    // 이벤트 타입 (예: OrderCreated, PaymentCompleted)
    Type() EventType
    
    // 이벤트 발생 시간
    OccurredAt() time.Time
    
    // 이벤트를 발생시킨 Aggregate ID
    AggregateID() string
    
    // Aggregate 타입 (Order, Payment, User 등)
    AggregateType() string
    
    // 이벤트 버전 (스키마 진화 대응)
    Version() int
    
    // 이벤트 데이터 (JSON 직렬화 가능)
    Data() map[string]interface{}
    
    // 이벤트 메타데이터
    Metadata() EventMetadata
}

type EventID string
type EventType string

type EventMetadata struct {
    CorrelationID string                 `json:"correlation_id"` // 분산 추적 ID
    CausationID   *EventID               `json:"causation_id"`   // 이 이벤트를 유발한 이벤트 ID
    UserID        *userDomain.UserID     `json:"user_id"`        // 이벤트를 발생시킨 사용자
    IPAddress     string                 `json:"ip_address"`     // 발생 위치
    UserAgent     string                 `json:"user_agent"`     // 클라이언트 정보
    Source        string                 `json:"source"`         // 이벤트 소스 (API, Scheduler, etc.)
    Tags          []string               `json:"tags"`           // 이벤트 태그
    Context       map[string]interface{} `json:"context"`        // 추가 컨텍스트
}

// BaseEvent는 모든 도메인 이벤트의 공통 구현체입니다
type BaseEvent struct {
    id            EventID
    type_         EventType
    occurredAt    time.Time
    aggregateID   string
    aggregateType string
    version       int
    data          map[string]interface{}
    metadata      EventMetadata
}

// 주문 도메인 이벤트들
type OrderCreated struct {
    BaseEvent
    OrderID      orderDomain.OrderID               `json:"order_id"`
    RestaurantID restaurantDomain.RestaurantID     `json:"restaurant_id"`
    CustomerID   *userDomain.UserID                `json:"customer_id"`
    TotalAmount  int64                            `json:"total_amount"`
    ItemCount    int                              `json:"item_count"`
    IsAnonymous  bool                             `json:"is_anonymous"`
}

type OrderAccepted struct {
    BaseEvent
    OrderID         orderDomain.OrderID      `json:"order_id"`
    AcceptedBy      userDomain.UserID        `json:"accepted_by"`
    EstimatedTime   int                      `json:"estimated_time_minutes"`
    AcceptedAt      time.Time                `json:"accepted_at"`
}

type OrderStatusChanged struct {
    BaseEvent
    OrderID    orderDomain.OrderID      `json:"order_id"`
    OldStatus  orderDomain.OrderStatus  `json:"old_status"`
    NewStatus  orderDomain.OrderStatus  `json:"new_status"`
    ChangedBy  *userDomain.UserID       `json:"changed_by"`
    Reason     string                   `json:"reason"`
    ChangedAt  time.Time                `json:"changed_at"`
}

// 결제 도메인 이벤트들
type PaymentCreated struct {
    BaseEvent
    PaymentID paymentDomain.PaymentID   `json:"payment_id"`
    OrderID   orderDomain.OrderID       `json:"order_id"`
    Amount    int64                     `json:"amount"`
    Method    paymentDomain.PaymentMethod `json:"method"`
}

type PaymentCompleted struct {
    BaseEvent
    PaymentID   paymentDomain.PaymentID `json:"payment_id"`
    OrderID     orderDomain.OrderID     `json:"order_id"`
    Amount      int64                   `json:"amount"`
    NetAmount   int64                   `json:"net_amount"`
    CompletedAt time.Time               `json:"completed_at"`
}

type PaymentFailed struct {
    BaseEvent
    PaymentID    paymentDomain.PaymentID `json:"payment_id"`
    OrderID      orderDomain.OrderID     `json:"order_id"`
    Amount       int64                   `json:"amount"`
    ErrorCode    string                  `json:"error_code"`
    ErrorMessage string                  `json:"error_message"`
    FailedAt     time.Time               `json:"failed_at"`
}

// 사용자 도메인 이벤트들
type UserCreated struct {
    BaseEvent
    UserID userDomain.UserID   `json:"user_id"`
    Role   userDomain.UserRole `json:"role"`
}

type UserLoggedIn struct {
    BaseEvent
    UserID    userDomain.UserID    `json:"user_id"`
    SessionID userDomain.SessionID `json:"session_id"`
    IPAddress string               `json:"ip_address"`
    LoginAt   time.Time            `json:"login_at"`
}

type SuspiciousActivity struct {
    BaseEvent
    UserID      *userDomain.UserID `json:"user_id"`
    ActivityType string            `json:"activity_type"`
    RiskScore   float64            `json:"risk_score"`
    Details     map[string]interface{} `json:"details"`
}

// 매장 도메인 이벤트들
type RestaurantCreated struct {
    BaseEvent
    RestaurantID restaurantDomain.RestaurantID `json:"restaurant_id"`
    OwnerID      userDomain.UserID              `json:"owner_id"`
    Name         string                         `json:"name"` // 암호화되지 않은 버전
}

type MenuUpdated struct {
    BaseEvent
    RestaurantID restaurantDomain.RestaurantID `json:"restaurant_id"`
    MenuID       restaurantDomain.MenuID       `json:"menu_id"`
    ChangeType   string                        `json:"change_type"` // CREATED, UPDATED, DELETED
    OldPrice     *int64                        `json:"old_price"`
    NewPrice     *int64                        `json:"new_price"`
}
```

### **🎯 Event Handlers**

```go
// internal/shared/domain/event_handlers.go
package domain

// EventHandler는 도메인 이벤트를 처리하는 핸들러 인터페이스입니다
type EventHandler interface {
    // 처리 가능한 이벤트 타입들을 반환합니다
    SupportedEventTypes() []EventType
    
    // 이벤트를 처리합니다
    Handle(ctx context.Context, event DomainEvent) error
    
    // 핸들러 우선순위를 반환합니다 (낮을수록 먼저 실행)
    Priority() int
    
    // 핸들러가 활성화되어 있는지 확인합니다
    IsEnabled() bool
}

// EventBus는 도메인 이벤트 발행 및 구독을 관리합니다
type EventBus interface {
    // 이벤트를 발행합니다
    Publish(ctx context.Context, events ...DomainEvent) error
    
    // 이벤트 핸들러를 등록합니다
    Subscribe(handler EventHandler) error
    
    // 이벤트 핸들러를 해제합니다
    Unsubscribe(handler EventHandler) error
    
    // 이벤트 처리 현황을 조회합니다
    GetProcessingStats() *EventProcessingStats
}

// 알림 발송 이벤트 핸들러 예시
type NotificationEventHandler struct {
    notificationService NotificationService
}

func (h *NotificationEventHandler) SupportedEventTypes() []EventType {
    return []EventType{
        "OrderCreated",
        "OrderAccepted", 
        "OrderReady",
        "PaymentCompleted",
        "PaymentFailed",
    }
}

func (h *NotificationEventHandler) Handle(ctx context.Context, event DomainEvent) error {
    switch event.Type() {
    case "OrderCreated":
        return h.handleOrderCreated(ctx, event.(*OrderCreated))
    case "OrderAccepted":
        return h.handleOrderAccepted(ctx, event.(*OrderAccepted))
    case "OrderReady":
        return h.handleOrderReady(ctx, event.(*OrderStatusChanged))
    case "PaymentCompleted":
        return h.handlePaymentCompleted(ctx, event.(*PaymentCompleted))
    case "PaymentFailed":
        return h.handlePaymentFailed(ctx, event.(*PaymentFailed))
    default:
        return nil // 지원하지 않는 이벤트는 무시
    }
}

func (h *NotificationEventHandler) handleOrderCreated(ctx context.Context, event *OrderCreated) error {
    // 새 주문 생성 시 매장 사장님에게 알림 발송
    // - 매장 소유자 조회
    // - 푸시 알림 또는 SMS 발송
    // - 실시간 웹소켓 알림
}

func (h *NotificationEventHandler) handleOrderAccepted(ctx context.Context, event *OrderAccepted) error {
    // 주문 승인 시 고객에게 알림 발송
    // - 고객 연락처 조회
    // - 예상 시간 포함한 알림 생성
    // - 선호하는 채널로 발송
}

// 감사 로그 이벤트 핸들러 예시
type AuditLogEventHandler struct {
    auditService AuditService
}

func (h *AuditLogEventHandler) SupportedEventTypes() []EventType {
    // 모든 이벤트 타입 지원 (전체 감사 로그 기록)
    return []EventType{"*"}
}

func (h *AuditLogEventHandler) Handle(ctx context.Context, event DomainEvent) error {
    // 모든 도메인 이벤트를 감사 로그로 기록
    // - 이벤트 데이터를 감사 로그 형식으로 변환
    // - 민감 정보 마스킹
    // - 감사 로그 저장
    return h.auditService.LogDomainEvent(ctx, event)
}

// 메트릭 수집 이벤트 핸들러 예시
type MetricsEventHandler struct {
    metricsService MetricsService
}

func (h *MetricsEventHandler) Handle(ctx context.Context, event DomainEvent) error {
    // 도메인 이벤트를 기반으로 비즈니스 메트릭 생성
    switch event.Type() {
    case "OrderCreated":
        return h.incrementOrderCount(ctx, event.(*OrderCreated))
    case "PaymentCompleted":
        return h.recordRevenue(ctx, event.(*PaymentCompleted))
    case "UserLoggedIn":
        return h.recordActiveUser(ctx, event.(*UserLoggedIn))
    }
    return nil
}
```

-----

## 🎯 **도메인 설계 요약**

### **🏗️ 도메인 아키텍처 특징**

1. **🔐 보안 우선 설계**
- 모든 민감 데이터는 컬럼 레벨에서 암호화
- 완전한 감사 추적으로 모든 변경사항 기록
- 도메인 레벨에서 접근 제어 및 권한 검증
1. **⚡ 고성능 설계**
- 도메인 객체 설계 시 캐싱과 인덱싱 고려
- 이벤트 기반 비동기 처리로 성능 최적화
- 대용량 데이터 처리를 위한 배치 처리 지원
1. **🎯 명확한 경계**
- 각 도메인의 책임과 의존성을 명확히 분리
- 도메인 간 통신은 이벤트를 통해 느슨한 결합 유지
- Aggregate 경계를 통한 일관성 보장
1. **🔄 확장 가능한 구조**
- 도메인 이벤트를 통한 새로운 기능 추가 용이성
- 인터페이스 기반 설계로 구현체 교체 가능
- 마이크로서비스 전환 준비된 경계 설정

### **📋 도메인별 핵심 특징**

|도메인             |핵심 기능     |보안 요소      |성능 고려사항     |
|----------------|----------|-----------|------------|
|**Restaurant**  |매장/메뉴 관리  |사업자정보 암호화  |위치 기반 검색 최적화|
|**Order**       |주문 생명주기 관리|주문 상세 암호화  |실시간 상태 추적   |
|**Payment**     |결제/환불 처리  |PCI DSS 준수 |고가용성 결제 처리  |
|**User**        |인증/권한 관리  |개인정보 암호화   |세션 관리 최적화   |
|**Encryption**  |데이터 암호화   |AES-256-GCM|키 회전 자동화    |
|**Audit**       |감사 추적     |불변 로그      |시계열 데이터 최적화 |
|**Metrics**     |성능/보안 메트릭 |메트릭 데이터 보호 |실시간 집계 처리   |
|**Notification**|실시간 알림    |알림 내용 암호화  |대량 발송 최적화   |

### **🔗 도메인 간 의존성 그래프**

```
User Domain ← Auth/Session
    ↓
Restaurant Domain ← Menu Management  
    ↓
Order Domain ← Payment Domain
    ↓
Notification Domain

# 보안 지원 도메인 (모든 도메인에서 사용)
All Domains → Encryption Domain
All Domains → Audit Domain  
All Domains → Metrics Domain
```

### **🎯 다음 구현 단계**

1. **🔐 보안 기반**: Encryption Domain → Audit Domain
1. **👤 사용자 관리**: User Domain → Session 관리
1. **🏪 비즈니스 로직**: Restaurant → Order → Payment
1. **🔔 지원 기능**: Notification → Metrics
1. **🔄 통합**: Domain Events → Event Handlers

-----

<div align="center">

**OrderSync Domain Design Complete** ✨

**보안 우선 • 성능 최적화 • 확장 가능한 아키텍처**

*Built for Scale, Designed for Security, Optimized for Performance*

**Ver. 1.0.0** | **Domain Count 8**: 8개 | **Aggregates**: 15개 | **Domain Services**: 25개
**Last Update. 2025.09.25**
</div>
