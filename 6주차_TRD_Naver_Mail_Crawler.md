# 네이버 메일 크롤링 앱 TRD (Technical Requirements Document)

## 1. 시스템 아키텍처

### 1.1 전체 아키텍처
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Database      │
│   (React)       │◄──►│   (Express.js)  │◄──►│   (Supabase)    │
│   Port: 3000    │    │   Port: 3001    │    │   PostgreSQL    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Browser       │    │   Playwright    │    │   File Storage  │
│   (Chrome)      │    │   (Automation)  │    │   (Images)      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 1.2 기술 스택
- **Frontend**: React 18, Vite, CSS3
- **Backend**: Node.js, Express.js
- **Database**: Supabase (PostgreSQL)
- **Web Automation**: Playwright
- **File Storage**: 로컬 파일 시스템
- **Encryption**: AES-256-CBC

## 2. 데이터베이스 설계

### 2.1 Supabase PostgreSQL 접속 정보
```env
# 데이터베이스 연결 정보
SUPABASE_USER=postgres.dktrdivmekmioqqkclzx
SUPABASE_PASSWORD=*********
SUPABASE_HOST=aws-1-ap-northeast-2.pooler.supabase.com
SUPABASE_PORT=6543
SUPABASE_DB=postgres
```

### 2.2 연결 문자열
```javascript
// PostgreSQL 연결 문자열
const connectionString = `postgresql://postgres.dktrdivmekmioqqkclzx:libero201!@aws-1-ap-northeast-2.pooler.supabase.com:6543/postgres`;
```

### 2.3 테이블 구조
```sql
-- emails 테이블 생성
CREATE TABLE emails (
    id SERIAL PRIMARY KEY,
    sender VARCHAR(255) NOT NULL,
    subject TEXT NOT NULL,
    content TEXT,
    images TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 인덱스 생성
CREATE INDEX idx_emails_created_at ON emails(created_at DESC);
CREATE INDEX idx_emails_sender ON emails(sender);
```

### 2.4 데이터 타입 및 제약사항
- **id**: 자동 증가 기본키
- **sender**: 발송자명 (최대 255자)
- **subject**: 메일 제목 (텍스트)
- **content**: 메일 본문 내용 (텍스트)
- **images**: 이미지 파일명 목록 (JSON 문자열)
- **created_at**: 수집 시간 (타임스탬프)

## 3. 네이버 메일 사이트 Selector 정보

### 3.1 로그인 관련 Selector
```javascript
// 로그인 버튼
'#account > div > a'
'a.link_login'

// 로그인 폼
'#id'                    // 아이디 입력 필드
'#pw'                    // 비밀번호 입력 필드
'.btn_login'            // 로그인 버튼
```

### 3.2 메일 목록 관련 Selector
```javascript
// 메일 목록 컨테이너
'#mail_list_scroll_view'
'.mail_list_container'

// 개별 메일 아이템
'.mail_item'
'.mail_list_item'
'.mail_row'

// 메일 링크 (제목)
'a[href*="mail"]'
'.mail_subject a'
'.subject_link'
```

### 3.3 메일 상세보기 관련 Selector
```javascript
// 메일 상세보기 컨테이너
'#mail_read_scroll_view'

// 메일 본문 영역 (우선순위 순)
'#mail_read_scroll_view .mail_view_body'
'#mail_read_scroll_view .mail_content'
'#mail_read_scroll_view .mail_view_body .mail_content'

// 메일 본문 내부 구조
'#mail_read_scroll_view .mail_view_body > div'
'#mail_read_scroll_view .mail_content > div'
'#mail_read_scroll_view .mail_view_body div[class*="content"]'
'#mail_read_scroll_view .mail_content div[class*="content"]'

// 테이블 기반 메일 본문
'#mail_read_scroll_view .mail_view_body table'
'#mail_read_scroll_view .mail_content table'
'#mail_read_scroll_view .mail_view_body table tbody'
'#mail_read_scroll_view .mail_content table tbody'

// 중첩된 테이블 구조
'#mail_read_scroll_view .mail_view_body table tbody tr td'
'#mail_read_scroll_view .mail_content table tbody tr td'
'#mail_read_scroll_view .mail_view_body table tbody tr td div'
'#mail_read_scroll_view .mail_content table tbody tr td div'

// 메일 미리보기/요약
'#mail_read_scroll_view .mail_view_body .mail_preview'
'#mail_read_scroll_view .mail_content .mail_preview'
'#mail_read_scroll_view .mail_view_body .preview'
'#mail_read_scroll_view .mail_content .preview'
'#mail_read_scroll_view .mail_view_body .summary'
'#mail_read_scroll_view .mail_content .summary'

// 일반적인 메일 본문 영역
'#mail_read_scroll_view [class*="mail"] [class*="body"]'
'#mail_read_scroll_view [class*="mail"] [class*="content"]'
'#mail_read_scroll_view [class*="body"]'
'#mail_read_scroll_view [class*="content"]'
```

### 3.4 발송자 정보 관련 Selector
```javascript
// 발송자 이름
'.sender_name'
'.sender'
'[class*="sender"]'

// 발송자에서 제외할 요소들 (날짜, 용량)
// Playwright evaluate() 함수로 DOM 조작하여 제거
// 날짜 패턴: \d{2}\.\d{2,4}
// 용량 패턴: \d+\.?\d*\s*KB
```

### 3.5 이미지 관련 Selector
```javascript
// 메일 본문 이미지 (포함)
'#mail_read_scroll_view .mail_view_body img'
'#mail_read_scroll_view .mail_content img'
'#mail_read_scroll_view .mail_view_body .mail_content img'
'img[src*="data:image"]'     // 인라인 이미지 (base64)
'img[src*="http"]'           // 외부 이미지
'img[src*="//"]'             // 프로토콜 없는 이미지
'img[data-src]'              // 지연 로딩 이미지
'img[srcset]'                // 반응형 이미지

// 첨부 이미지 (제외)
'.mail_attachment img'
'.attachment img'
'img[src*="attachment"]'
'img[src*="inline"]'
'.attachment_area img'
'.file_attachment img'
```

### 3.6 제거할 UI 요소 Selector
```javascript
// 제거할 불필요한 요소들
const elementsToRemove = [
  'script', 'style', 'noscript', 'iframe', 'embed', 'object',
  '.mail_header', '.mail_footer', '.mail_navigation', '.mail_sidebar',
  '.mail_attachment', '.attachment', '.attachment_area', '.file_attachment',
  '.mail_signature', '.signature', '.mail_footer_info', '.footer_info',
  '.mail_tracking', '.tracking', '.mail_ads', '.ads', '.advertisement',
  '.mail_banner', '.banner', '.mail_promo', '.promo',
  '.mail_unsubscribe', '.unsubscribe', '.mail_opt_out', '.opt_out',
  '.mail_privacy', '.privacy', '.mail_legal', '.legal',
  '.mail_social', '.social', '.mail_share', '.share',
  '.mail_related', '.related', '.mail_suggestions', '.suggestions'
];
```

### 3.7 Selector 우선순위 전략
1. **정확한 ID/Class**: `#mail_read_scroll_view .mail_view_body`
2. **일반적인 클래스**: `[class*="mail"] [class*="body"]`
3. **태그 기반**: `div`, `p`, `span`
4. **속성 기반**: `img[src*="http"]`, `img[data-src]`

### 3.8 Selector 안정성 고려사항
- **다중 Selector**: 하나의 요소에 대해 여러 Selector 시도
- **Fallback 전략**: 주요 Selector 실패 시 대체 Selector 사용
- **동적 요소 대응**: `data-src`, `srcset` 등 지연 로딩 요소 고려
- **중복 제거**: Set을 사용한 URL 중복 방지
- **크기 필터링**: 너무 작은 이미지 (10x10px 미만) 제외

## 4. API 설계

### 4.1 REST API 엔드포인트

#### 4.1.1 메일 수집 API
```
POST /api/crawl
Content-Type: application/json

Response:
{
  "success": boolean,
  "message": string,
  "data": {
    "count": number,
    "emails": array
  }
}
```

#### 4.1.2 메일 목록 조회 API
```
GET /api/emails
Content-Type: application/json

Response:
{
  "success": boolean,
  "data": [
    {
      "id": number,
      "sender": string,
      "subject": string,
      "content": string,
      "images": array,
      "created_at": string
    }
  ]
}
```

#### 4.1.3 이미지 서빙 API
```
GET /images/{filename}
Content-Type: image/*

Response: 이미지 파일 바이너리
```

### 4.2 에러 처리
```javascript
// 표준 에러 응답 형식
{
  "success": false,
  "error": {
    "code": string,
    "message": string,
    "details": object
  }
}
```

## 5. 보안 설계

### 5.1 인증 및 권한
- **환경 변수**: 민감한 정보는 `.env` 파일에 저장
- **암호화**: AES-256-CBC로 비밀번호 암호화
- **CORS**: 허용된 도메인에서만 API 접근

### 5.2 데이터 보호
```javascript
// 암호화 설정
const crypto = require('crypto');
const algorithm = 'aes-256-cbc';
const key = process.env.ENCRYPTION_KEY; // 32바이트 키
const iv = crypto.randomBytes(16);
```

### 5.3 입력 검증
- **SQL Injection 방지**: Parameterized Query 사용
- **XSS 방지**: 입력 데이터 sanitization
- **파일 업로드 제한**: 파일 크기 및 타입 검증

## 6. 크롤링 시스템

### 6.1 Playwright 설정
```javascript
const { chromium } = require('playwright');

const browser = await chromium.launch({
  headless: true,
  args: ['--no-sandbox', '--disable-setuid-sandbox']
});

const page = await browser.newPage();
await page.setViewportSize({ width: 1920, height: 1080 });
```

### 6.2 DOM 선택자 전략
```javascript
// 우선순위 기반 선택자
const contentSelectors = [
  '#mail_read_scroll_view .mail_view_body',
  '#mail_read_scroll_view .mail_content',
  '#mail_read_scroll_view .mail_view_body .mail_content',
  // ... 추가 선택자들
];
```

### 6.3 이미지 처리
```javascript
// 이미지 다운로드 및 저장
async downloadImage(url, filename) {
  const response = await axios.get(url, {
    responseType: 'stream',
    timeout: 15000,
    maxRedirects: 5
  });
  
  const writer = fs.createWriteStream(path.join('images', filename));
  response.data.pipe(writer);
}
```

## 7. 성능 최적화

### 7.1 데이터베이스 최적화
- **연결 풀링**: PostgreSQL 연결 풀 사용
- **쿼리 최적화**: 인덱스 활용 및 효율적인 쿼리
- **페이징**: 대용량 데이터 처리 시 페이징 구현

### 7.2 프론트엔드 최적화
- **이미지 지연 로딩**: Intersection Observer API 사용
- **상태 관리**: React useState 최적화
- **메모이제이션**: 불필요한 리렌더링 방지

### 7.3 크롤링 최적화
- **병렬 처리**: 여러 메일 동시 처리
- **캐싱**: 중복 요청 방지
- **타임아웃**: 무한 대기 방지

## 8. 모니터링 및 로깅

### 8.1 로깅 시스템
```javascript
// 구조화된 로깅
console.log(`메일 ${index + 1} 처리 중...`);
console.log(`메일 ${index + 1} 수집 완료: ${subject}`);
console.error(`메일 ${index + 1} 처리 오류:`, error);
```

### 8.2 에러 추적
- **크롤링 에러**: DOM 선택자 실패, 네트워크 오류
- **데이터베이스 에러**: 연결 실패, 쿼리 오류
- **파일 시스템 에러**: 이미지 다운로드 실패

### 8.3 성능 모니터링
- **응답 시간**: API 응답 시간 측정
- **메모리 사용량**: Node.js 메모리 사용량 모니터링
- **CPU 사용률**: 크롤링 작업 중 CPU 사용률

## 9. 배포 및 운영

### 9.1 환경 설정
```bash
# 개발 환경
NODE_ENV=development
PORT=3001

# 프로덕션 환경
NODE_ENV=production
PORT=3001
```

### 9.2 의존성 관리
```json
{
  "dependencies": {
    "express": "^4.18.2",
    "playwright": "^1.40.0",
    "pg": "^8.11.3",
    "dotenv": "^16.3.1",
    "cors": "^2.8.5",
    "axios": "^1.6.0"
  }
}
```

### 9.3 프로세스 관리
```bash
# PM2를 사용한 프로세스 관리
pm2 start server.js --name "mail-crawler"
pm2 start "npm run dev" --name "frontend" --cwd frontend
```

## 10. 테스트 전략

### 10.1 단위 테스트
- **API 테스트**: Jest를 사용한 엔드포인트 테스트
- **유틸리티 테스트**: 암호화, 데이터 변환 함수 테스트
- **크롤링 테스트**: Mock 데이터를 사용한 크롤링 로직 테스트

### 10.2 통합 테스트
- **데이터베이스 테스트**: 실제 DB 연결 테스트
- **크롤링 테스트**: 실제 네이버 메일 페이지 테스트
- **API 통합 테스트**: 전체 워크플로우 테스트

### 10.3 성능 테스트
- **부하 테스트**: 동시 사용자 시뮬레이션
- **메모리 테스트**: 장시간 실행 시 메모리 누수 확인
- **크롤링 성능**: 대량 메일 처리 성능 측정

## 11. 확장성 고려사항

### 11.1 수평적 확장
- **로드 밸런서**: 다중 서버 인스턴스 지원
- **데이터베이스 샤딩**: 대용량 데이터 분산 저장
- **캐싱 레이어**: Redis를 사용한 캐싱 시스템

### 11.2 수직적 확장
- **리소스 증설**: CPU, 메모리, 스토리지 확장
- **최적화**: 알고리즘 및 데이터 구조 최적화
- **병렬 처리**: 멀티스레딩 및 비동기 처리

## 12. 유지보수 계획

### 12.1 정기 점검
- **일일**: 로그 확인, 에러 모니터링
- **주간**: 성능 지표 분석, 용량 확인
- **월간**: 보안 업데이트, 의존성 업데이트

### 12.2 백업 전략
- **데이터베이스 백업**: 일일 자동 백업
- **파일 백업**: 이미지 파일 정기 백업
- **설정 백업**: 환경 설정 및 코드 백업

### 12.3 업데이트 계획
- **보안 패치**: 즉시 적용
- **기능 업데이트**: 분기별 릴리스
- **의존성 업데이트**: 월별 점검 및 업데이트
