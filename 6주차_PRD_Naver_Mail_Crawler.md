# 네이버 메일 크롤링 앱 PRD (Product Requirements Document)

## 1. 제품 개요

### 1.1 제품명
**네이버 메일 자동 수집 및 관리 시스템**

### 1.2 제품 목적
네이버 메일 계정에 로그인하여 최신 메일을 자동으로 수집하고, 웹 인터페이스를 통해 메일 내용을 조회하고 관리할 수 있는 시스템

### 1.3 타겟 사용자
- 네이버 메일을 주로 사용하는 개인 사용자
- 메일 내용을 체계적으로 관리하고 싶은 사용자
- 메일 데이터를 백업하고 보관하고 싶은 사용자

## 2. 핵심 기능

### 2.1 메일 자동 수집
- **기능**: 네이버 메일 계정에 자동 로그인하여 최신 10개 메일 수집
- **대상**: 발송자, 제목, 본문 내용, 첨부 이미지
- **주기**: 사용자가 요청할 때마다 실행
- **보안**: 로그인 정보는 암호화하여 안전하게 저장

### 2.2 메일 데이터 저장
- **저장소**: Supabase PostgreSQL 데이터베이스
- **저장 데이터**: 
  - 메일 ID, 발송자, 제목, 본문 내용
  - 첨부 이미지 파일명 목록
  - 수집 시간
- **이미지 저장**: 로컬 서버에 이미지 파일 다운로드 및 저장

### 2.3 웹 인터페이스
- **메일 목록 조회**: 수집된 메일 목록을 테이블 형태로 표시
- **메일 상세 보기**: 메일 제목 클릭 시 팝업으로 상세 내용 표시
- **이미지 표시**: 메일 본문의 이미지를 팝업에서 확인 가능
- **반응형 디자인**: 다양한 화면 크기에 대응

## 3. 사용자 스토리

### 3.1 메일 수집
**As a** 네이버 메일 사용자  
**I want to** 최신 메일을 자동으로 수집하고 싶다  
**So that** 메일 내용을 체계적으로 관리할 수 있다

**Acceptance Criteria:**
- 네이버 계정 정보를 입력하여 로그인
- 최신 10개 메일을 자동으로 수집
- 메일 본문과 이미지를 정확하게 추출
- 수집된 데이터를 데이터베이스에 저장

### 3.2 메일 조회
**As a** 사용자  
**I want to** 수집된 메일 목록을 확인하고 싶다  
**So that** 원하는 메일을 쉽게 찾을 수 있다

**Acceptance Criteria:**
- 메일 목록을 최신순으로 정렬하여 표시
- 발송자, 제목, 수집 시간 정보 표시
- 메일 제목 클릭 시 상세 내용 팝업 표시

### 3.3 메일 상세 보기
**As a** 사용자  
**I want to** 메일의 상세 내용을 확인하고 싶다  
**So that** 메일 내용을 완전히 파악할 수 있다

**Acceptance Criteria:**
- 발송자, 제목, 본문 내용 표시
- 메일 본문의 이미지 표시
- 이미지 확대 및 네비게이션 기능
- 팝업 닫기 기능

## 4. 비기능 요구사항

### 4.1 성능
- 메일 수집 시간: 10개 메일 기준 2분 이내
- 페이지 로딩 시간: 3초 이내
- 이미지 로딩 시간: 5초 이내

### 4.2 보안
- 로그인 정보 암호화 저장
- HTTPS 통신
- 세션 관리

### 4.3 사용성
- 직관적인 사용자 인터페이스
- 반응형 웹 디자인
- 에러 메시지 명확성

### 4.4 호환성
- 최신 브라우저 지원 (Chrome, Firefox, Safari, Edge)
- 모바일 디바이스 지원

## 5. 제약사항

### 5.1 기술적 제약
- 네이버 메일의 DOM 구조 변경 시 크롤링 로직 수정 필요
- 이미지 파일 크기 제한 (10MB)
- 동시 사용자 수 제한

### 5.2 법적/윤리적 제약
- 개인정보보호법 준수
- 네이버 서비스 이용약관 준수
- 사용자 동의 하에 데이터 수집

## 6. 성공 지표

### 6.1 기능적 지표
- 메일 수집 성공률: 95% 이상
- 이미지 다운로드 성공률: 90% 이상
- 데이터 정확성: 98% 이상

### 6.2 사용자 경험 지표
- 페이지 로딩 시간: 3초 이내
- 사용자 만족도: 4.0/5.0 이상
- 에러 발생률: 5% 이하

## 7. 네이버 메일 사이트 Selector 정보

### 7.1 로그인 관련 Selector
```javascript
// 로그인 버튼
'#account > div > a'
'a.link_login'

// 로그인 폼
'#id'                    // 아이디 입력 필드
'#pw'                    // 비밀번호 입력 필드
'.btn_login'            // 로그인 버튼
```

### 7.2 메일 목록 관련 Selector
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

### 7.3 메일 상세보기 관련 Selector
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

### 7.4 발송자 정보 관련 Selector
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

### 7.5 이미지 관련 Selector
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

### 7.6 제거할 UI 요소 Selector
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

### 7.7 Selector 우선순위 전략
1. **정확한 ID/Class**: `#mail_read_scroll_view .mail_view_body`
2. **일반적인 클래스**: `[class*="mail"] [class*="body"]`
3. **태그 기반**: `div`, `p`, `span`
4. **속성 기반**: `img[src*="http"]`, `img[data-src]`

### 7.8 Selector 안정성 고려사항
- **다중 Selector**: 하나의 요소에 대해 여러 Selector 시도
- **Fallback 전략**: 주요 Selector 실패 시 대체 Selector 사용
- **동적 요소 대응**: `data-src`, `srcset` 등 지연 로딩 요소 고려
- **중복 제거**: Set을 사용한 URL 중복 방지
- **크기 필터링**: 너무 작은 이미지 (10x10px 미만) 제외

## 8. 데이터베이스 접속 정보

### 8.1 Supabase PostgreSQL 설정
```env
# config.env 파일
SUPABASE_USER=postgres.dktrdivmekmioqqkclzx
SUPABASE_PASSWORD=*********
SUPABASE_HOST=aws-1-ap-northeast-2.pooler.supabase.com
SUPABASE_PORT=6543
SUPABASE_DB=postgres
```

### 8.2 데이터베이스 스키마
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

### 8.3 연결 문자열
```javascript
// PostgreSQL 연결 문자열
const connectionString = `postgresql://${SUPABASE_USER}:${SUPABASE_PASSWORD}@${SUPABASE_HOST}:${SUPABASE_PORT}/${SUPABASE_DB}`;
```

## 9. 환경 설정

### 9.1 필수 환경 변수
```env
# 네이버 계정 정보
NAVER_ID=bjkim2004
NAVER_PASSWORD=libero281(

# 암호화 키 (32바이트)
ENCRYPTION_KEY=79e2ae930639e93e5b51f3d67aef025267c914c90a1f1edab90b691d8bb8c66f

# Supabase 데이터베이스 정보
SUPABASE_USER=postgres.dktrdivmekmioqqkclzx
SUPABASE_PASSWORD=libero201!
SUPABASE_HOST=aws-1-ap-northeast-2.pooler.supabase.com
SUPABASE_PORT=6543
SUPABASE_DB=postgres
```

### 9.2 프로젝트 구조
```
playwright2/
├── backend/
│   ├── server.js          # Express 서버
│   ├── crawler.js         # Playwright 크롤링 로직
│   ├── db.js             # 데이터베이스 연결
│   ├── encryption.js     # 암호화 유틸리티
│   ├── images/           # 다운로드된 이미지 저장
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── App.jsx
│   │   └── components/
│   └── package.json
├── config.env            # 환경 변수
└── README.md
```

## 10. 향후 계획

### 10.1 단기 계획 (1-3개월)
- 메일 필터링 기능 추가
- 검색 기능 구현
- 메일 분류 기능

### 10.2 중기 계획 (3-6개월)
- 다중 계정 지원
- 메일 자동 분류
- 통계 및 분석 기능

### 10.3 장기 계획 (6개월 이상)
- AI 기반 메일 분석
- 클라우드 저장소 연동
- 모바일 앱 개발
