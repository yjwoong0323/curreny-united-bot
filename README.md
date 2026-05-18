# Cuunit — 환전 재테크를 위한 P2P 금융 서비스

> 핀테크 인턴십 코스 개발자 과정 2회차 — 기업 연계 프로젝트 (2025.06 ~ 2025.08)
>
> **One Click to Profit** — 버튼 한 번으로 누구나 쉽고 확실하게 환차익을 낼 수 있는 환전 플랫폼

<br>

## 프로젝트 소개

**Cuunit**은 다중 은행의 실시간 환율을 통합 수집해 사용자에게 최적의 환율로 환전 기회를 제공하는 P2P 기반 환전 플랫폼입니다.

환전 재테크에 관심이 있더라도, 매매기준율 · 전신환 매도/매입율 · 우대환율 · 관세 등 복잡한 개념과 은행별로 다른 환율을 비교해야 하는 정보 비대칭 문제 때문에 일반 사용자가 접근하기 어려웠습니다.
Cuunit은 이 과정을 버튼 한 번으로 자동화해, 비대면 · 수수료 없는 투명한 환전 구조를 만드는 것을 목표로 합니다.

<br>

## 핵심 기능

| 기능 | 설명 |
| --- | --- |
| 최적의 환전 | 다중 은행 환율 중 가장 유리한 가격을 자동 선택 (BUY / SELL) |
| 다중 은행 환율 크롤링 | 10분마다 크롤러가 실행되어 은행별 실시간 환율 데이터 수집 |
| 원클릭 환테크 | 외화 구매 → 판매까지 단일 트랜잭션으로 처리해 즉시 차익 실현 |

<br>

## 기술 스택

| 영역 | 기술 |
| --- | --- |
| Backend | Kotlin, Spring Boot, JWT |
| Crawler | Python, Selenium |
| Database | MySQL, Redis |
| Infra | Docker |
| Test | Kotest, pytest |
| Collaboration | GitHub, Notion |

<br>

## 프로젝트 구조

```
.
├── crawler/                    # Python 크롤러 모듈
│   ├── banks/                  # 은행별 커스텀 크롤러
│   ├── dto/
│   ├── utils/
│   ├── main.py
│   ├── scheduler.py            # 10분 단위 스케줄링
│   └── rate_plot.ipynb         # 환율 데이터 분석
│
└── backend/                    # Spring Boot 백엔드
    ├── auth/                   # JWT 인증/보안
    ├── common/
    ├── enums/
    ├── exception/
    ├── exchange/               # 환율·환전 도메인
    ├── finance/                # Account / Wallet / 입출금
    └── user/                   # 사용자 도메인
```

<br>

## 주요 구현 사항

### 1. JWT 기반 인증 & Redis 토큰 관리 (담당)

- 로그인 시 `JWTProvider`를 통해 AccessToken / RefreshToken을 발급, Redis에 저장
- `TokenProvider`와 `AuthenticationFilter`를 분리 구현해 인증 책임 명확화
- RefreshToken을 활용한 재발급 플로우로 사용자 세션 유지
- Redis TTL을 활용한 자동 만료 처리로 보안성 확보

### 2. User / Account / Wallet 도메인 API (담당)

- 회원가입 · 로그인 · 회원정보 관리
- 사용자별 원화/외화 계좌(Account) 및 지갑(Wallet) 관리
- 입출금 내역(`deposit_withdrawal`) 및 외화 거래 이력(`wallet_fx_history`) 추적

### 3. 환전 처리 로직

- BigDecimal + 통화별 소수점 단위(scale) 적용으로 금액 손실 없는 정확한 환전 계산
- Redis Queue 기반 순차 처리로 동시 주문 시에도 일관성 보장
- `buyOrder` → `sellOrder` 흐름으로 원클릭 환테크 구현

### 4. 다중 은행 환율 크롤링

- XHR(XMLHttpRequest) 기반 비동기 수집으로 빠르고 안정적인 데이터 확보
- 은행별 Response 구조에 맞춘 커스텀 크롤러 모듈 인터페이스 분리 설계
- 최근 고시 시간 기준 5분 이내 데이터만 환전에 활용해 신뢰성 보장

<br>

## DB 설계

핵심 엔티티 11개로 구성된 정규화 스키마

- **사용자/계좌** — `user`, `account`, `wallet`, `deposit_withdrawal`, `wallet_fx_history`
- **환율/은행** — `bank`, `currency`, `exchange_rate`
- **환전 거래** — `exchange_order`, `exchange_ledger`, `transaction`

금액 필드는 모두 `DECIMAL(18, 2)`, 환율은 `DECIMAL(18, 6)`로 정밀도 확보.

<br>

## 환전 프로세스 흐름

```
   [User] ──₩──▶ ┌─ buyOrder ─────────────────┐
                 │  유저 ⇄ 회사 ⇄ 은행 (₩→$)  │
                 └────────────┬───────────────┘
                              ▼
                 ┌─ sellOrder ────────────────┐
                 │  유저 ⇄ 회사 ⇄ 은행 ($→₩)  │
                 └────────────┬───────────────┘
   [User] ◀──₩── ─────────────┘
```

사용자는 버튼 한 번만 누르면, 내부에서 매수·매도 주문이 자동으로 체결되어 환차익이 정산 됨

<br>

## 트러블슈팅 & 개선 포인트

| 영역 | 현재 구현 | 개선 방향 |
| --- | --- | --- |
| 주문 처리 | Redis Queue로 순차 환전 처리 | 주문량 급증 시 지연 / Redis 장애 대비 메시지 브로커(Kafka 등) 도입 검토 |
| 예외 처리 | String 기반 Exception | `@RestControllerAdvice` + ErrorCode Enum으로 일관된 응답 포맷 통일 |
| 테스트 커버리지 | 일부 단위 테스트 | Kotest 기반 통합 테스트 보강, 환전 계산 로직 엣지 케이스 추가 |
| 코드 컨벤션 | 팀원별 스타일 차이 | Ktlint + 코드 컨벤션 문서화 |

<br>

## 성과 & 차별성

- 다중 은행 실시간 환율 통합 — 단일 은행 의존 환전 서비스 대비 우위
- 플랫폼 확장성 — 크롤러 모듈을 인터페이스로 분리해 신규 은행 추가 시 비용 최소화
- 사용자 경험 최적화 — 복잡한 환율 개념을 추상화한 원클릭 UX
- 금액 정확성 — BigDecimal + 통화별 scale로 부동소수점 오차 원천 차단

<br>

## 팀 구성

| 이름 | 역할 | 담당 |
| --- | --- | --- |
| **이재웅** | Backend | 인증/보안 (JWT, Redis), User · Account · Wallet API |
| 남현기 | Backend | 환율 크롤링 모듈, 환율 거래 API |
| 황내권 | Mentor | 기획 및 개발 피드백 |

<br>

## 회고

> 기획 의도가 명확한 주제로 첫 프로젝트를 진행하면서, 일반 도메인보다 금융 서비스 개발에서는 안정성과 보안 요소가 훨씬 중요하다는 것을 체감했습니다.
>
> 특히 JWT + Redis 기반 인증 구조를 처음 설계하며 토큰 만료 · 재발급 · 무효화 흐름을 직접 다뤄본 경험이 컸고, BigDecimal과 통화별 scale을 다루며 금융 계산의 정밀도가 왜 중요한지 코드 레벨에서 이해할 수 있었습니다.
>
> 현업 멘토링을 통해 앞으로 채워나가야 할 역량(테스트, 일관된 예외 처리, 메시지 큐 등)이 명확해졌고, 백엔드 개발자로서의 방향성을 잡을 수 있었던 것이 가장 큰 수확이었습니다.

<br>

## Contact

- **이재웅** — yjwoong0323@gmail.com
- **GitHub** — [yjwoong0323/curreny-united-bot](https://github.com/yjwoong0323/curreny-united-bot)
