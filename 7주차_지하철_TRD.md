# TRD (Technical Requirements Document)

### 1. 시스템 아키텍처 ✅

#### 1.1 전체 구조 (구현됨)

```
[클라이언트 - Next.js 웹앱]
├── React 18 + TypeScript
├── Zustand (상태 관리)
├── TanStack Query (데이터 페칭)
├── Tailwind CSS (스타일링)
└── PWA (모바일 지원)
        ↓
[Next.js API Routes]
├── /api/subway (실시간 데이터)
├── 데이터 파싱 및 변환
└── 캐싱 시스템
        ↓
[서울 열린데이터 광장 API]
├── 실시간 열차 위치
├── 실시간 도착 정보
└── JSON 응답 처리
```

#### 1.2 구현된 기술 스택

**프론트엔드 (완료)**

- React 18 + TypeScript ✅
- Next.js 14 (App Router) ✅
- TanStack Query (React Query) - API 상태 관리 ✅
- Zustand - 전역 상태 관리 ✅
- Tailwind CSS - 스타일링 ✅

**웹 애플리케이션 (완료)**

- Next.js 14 (App Router) ✅
- PWA 설정 (모바일 지원) ✅
- Service Worker 구현 ✅
- 반응형 디자인 ✅

**API & 데이터 처리 (완료)**

- Next.js API Routes ✅
- 서울 열린데이터 광장 API 연동 ✅
- 실시간 데이터 캐싱 ✅
- 에러 처리 및 재시도 로직 ✅

### 2. API 명세 ✅

#### 2.1 서울 열린데이터 광장 API (구현됨)

**실시간 열차 위치 API**
- 엔드포인트: `http://swopenAPI.seoul.go.kr/api/subway/{인증키}/json/realtimePosition/0/50/{호선명}`
- 인증키: `7a70754c4a626a6b35374465557049` ✅
- 응답 형식: JSON ✅

**실시간 도착 정보 API**
- 엔드포인트: `http://swopenAPI.seoul.go.kr/api/subway/{인증키}/json/realtimeStationArrival/0/50/{역명}`
- 특정 역의 열차 도착 정보 조회 ✅

**지원 노선 (구현됨)**:

- 1~9호선 (1001~1009) ✅
- 광역철도: 신분당선(1077), 경강선(1081), 수인분당선(1075), 공항철도(1065) ✅
- GTX-A(1032), 우이신설선(1092) 등 ✅

**주요 응답 데이터 (구현됨)**:

```json
{
  "realtimeArrivalList": [
    {
      "subwayId": "1001",
      "btrainNo": "열차번호",
      "statnNm": "역명",
      "updnLine": "상행/하행/내선/외선",
      "arvlMsg2": "도착 메시지",
      "barvlDt": "도착 예정 시간(초)",
      "bstatnNm": "종착역"
    }
  ]
}
```

#### 2.2 클라이언트 API 함수 (구현됨) ✅

```typescript
// 실시간 열차 위치 조회 ✅
getRealtimePosition(line: string, stationName?: string): Promise<TrainPosition[]>

// 특정 역의 열차 도착 정보 ✅
getTrainArrivalInfo(stationName: string, line: string): Promise<TrainPosition[]>

// 최적 출발 시간 계산 ✅
calculateOptimalDeparture(
  currentStation: string,
  walkingTime: number,
  line: string,
  trains: TrainPosition[]
): DepartureRecommendation

// 노선별 색상 가져오기 ✅
getLineColor(lineId: string): string

// 노선명 가져오기 ✅
getLineName(lineId: string): string
```

### 3. 데이터 모델 (구현됨) ✅

```typescript
interface Station {
  id: string;
  name: string;
  line: string;
  order: number; // 노선 내 순서
}

interface TrainPosition {
  trainNo: string;
  stationName: string;
  status: 'approaching' | 'arrived' | 'departed';
  direction: string; // 상행, 하행, 내선, 외선 등
  isLastTrain: boolean;
  updatedAt: Date;
  arrivalTime?: string; // 도착 예정 시간
  destination?: string; // 종착역
}

// API에서 받아오는 실제 열차 데이터 타입 ✅
interface RealtimeTrainData {
  trainNo: string;
  currentStation: string;
  destination: string;
  arrivalTime: string;
  status: string;
  message: string;
  line: string;
  direction: string;
  rawArrivalTime: string;
}

interface UserSettings {
  homeStation: Station | null;
  destinationStation: Station | null;
  walkingTimeMinutes: number;
  notificationEnabled: boolean;
  notificationAdvanceMinutes: number;
}

interface DepartureRecommendation {
  shouldLeaveNow: boolean;
  waitingTimeIfLeaveNow: number; // 분
  optimalDepartureTime: Date;
  nextTrainArrival: Date;
  message: string;
  otherStations?: string; // 다른 역에 있는 열차 정보
  stationDetails?: Array<{
    station: string;
    count: number;
    nextTrain: string;
  }>;
}
```

### 4. 핵심 알고리즘 (구현됨) ✅

#### 4.1 최적 출발 시간 계산 ✅

```
1. 현재 시각 기준 출발역에 가장 가까운 열차 찾기 ✅
2. 해당 열차의 출발역 도착 예상 시간 계산 ✅
3. 사용자 도보 시간 고려 ✅
4. 최적 출발 시간 = 열차 도착 시간 - 도보 시간 - 여유 시간(1분) ✅
5. 현재 시각과 비교하여 메시지 생성 ✅
6. 다른 역에 있는 열차 정보도 함께 제공 ✅
```

#### 4.2 열차 도착 시간 예측 ✅

```
- API에서 받은 실제 도착 시간 정보 활용 ✅
- 도착 메시지 기반 상태 판단 (진입/도착/출발) ✅
- 초 단위 도착 시간을 분/초로 변환 ✅
- 실시간 상태 업데이트 (30초 간격) ✅
```

#### 4.3 방향별 색상 구분 ✅

```
- 상행: 파란색 그라데이션 (blue-100 to blue-200) ✅
- 하행: 빨간색 그라데이션 (red-100 to red-200) ✅
- 내선: 보라색 그라데이션 (purple-100 to purple-200) ✅
- 외선: 초록색 그라데이션 (green-100 to green-200) ✅
```

### 5. 저장소 구조 (구현됨) ✅

```
subway-timing-app/
├── src/
│   ├── app/                 # Next.js App Router
│   │   ├── layout.tsx       # 루트 레이아웃
│   │   ├── page.tsx         # 메인 페이지
│   │   ├── providers.tsx    # React Query Provider
│   │   ├── globals.css      # 글로벌 스타일
│   │   └── api/
│   │       └── subway/
│   │           └── route.ts # API 라우트
│   ├── components/          # React 컴포넌트
│   │   ├── LineSelector.tsx # 노선 선택
│   │   ├── TrainCard.tsx    # 열차 정보 카드
│   │   ├── DepartureRecommendation.tsx # 출발 시간 추천
│   │   ├── SettingsModal.tsx # 설정 모달
│   │   ├── LoadingSpinner.tsx # 로딩 스피너
│   │   └── TrainVisualization.tsx # 열차 시각화
│   ├── hooks/               # 커스텀 훅
│   │   └── useSubwayData.ts # 지하철 데이터 훅
│   ├── lib/                 # 유틸리티 및 API
│   │   ├── api.ts           # API 함수
│   │   ├── cache.ts         # 캐싱 시스템
│   │   ├── queryClient.ts   # React Query 설정
│   │   ├── storage.ts       # 로컬 스토리지
│   │   └── utils.ts         # 유틸리티 함수
│   ├── store/               # 상태 관리
│   │   └── useAppStore.ts   # Zustand 스토어
│   ├── types/               # 타입 정의
│   │   └── index.ts         # TypeScript 타입
│   └── data/                # 정적 데이터
│       └── stations.ts      # 역 정보
├── docs/
│   ├── PRD.md              # 제품 요구사항
│   └── TRD.md              # 기술 요구사항
├── public/                 # 정적 파일
│   ├── icon.svg            # PWA 아이콘
│   ├── manifest.json       # PWA 매니페스트
│   └── sw.js               # Service Worker
├── package.json            # 의존성 관리
├── next.config.js          # Next.js 설정
├── tailwind.config.js      # Tailwind CSS 설정
├── tsconfig.json           # TypeScript 설정
└── README.md               # 프로젝트 문서
```

### 6. 개발 단계 (현재 상태)

**Phase 1: MVP (웹 우선) ✅ 완료**

- 웹 애플리케이션 개발 ✅
- 9개 노선 지원 (1~9호선) + 광역철도 ✅
- 실시간 열차 위치 표시 ✅
- 최적 출발 시간 계산 ✅
- PWA 지원 ✅

**Phase 2: 기능 확장 ✅ 완료**

- 여러 노선 지원 ✅
- 사용자 설정 저장 ✅
- 방향별 색상 구분 ✅
- 캐싱 시스템 ✅
- 에러 처리 및 재시도 ✅

**Phase 3: 향후 확장 계획**

- React Native 모바일 앱
- Electron 데스크톱 앱
- 앱스토어 배포
- 추가 기능 (혼잡도, 경로 최적화 등)

### 7. 보안 및 성능 (구현됨) ✅

**보안**

- API 키 환경변수 관리 ✅
- HTTPS 통신 ✅
- XSS/CSRF 방어 (Next.js 기본 제공) ✅

**성능**

- API 응답 캐싱 (30초) ✅
- React Query 기반 데이터 페칭 최적화 ✅
- 코드 스플리팅 (Next.js 자동) ✅
- PWA 캐싱 (Service Worker) ✅

**모니터링**

- 콘솔 로깅으로 API 호출 추적 ✅
- 에러 처리 및 재시도 로직 ✅
- 성능 최적화 (캐싱, 상태 관리) ✅

### 8. 테스트 전략 (향후 구현)

- Unit Test: Jest + React Testing Library
- E2E Test: Playwright (웹)
- API Mock: MSW (Mock Service Worker)
- 테스트 커버리지 목표: 80% 이상

### 9. 배포 전략 (구현됨) ✅

**웹**: Vercel 또는 Netlify (CI/CD 자동화) ✅
- Next.js 빌드 최적화 ✅
- PWA 설정 완료 ✅
- 환경변수 관리 ✅

**향후 계획**:
- 모바일: App Store, Google Play
- 데스크톱: GitHub Releases, 자동 업데이트

### 10. 향후 확장 가능성

- 버스 실시간 정보 통합
- 경로 최적화 기능
- 혼잡도 정보 표시
- 소셜 기능 (친구와 위치 공유)
- 다국어 지원
- 알림 기능 강화
- 오프라인 지원

### 11. API 참조

**서울 열린데이터 광장 API**
- 문서: https://data.seoul.go.kr/dataList/OA-12764/F/1/datasetView.do
- 실시간 도착 정보: http://swopenAPI.seoul.go.kr/api/subway/{API_KEY}/json/realtimeStationArrival/0/50/서울
- 실시간 위치 정보: http://swopenAPI.seoul.go.kr/api/subway/{API_KEY}/json/realtimePosition/0/50/1