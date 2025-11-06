# Technical Requirements Document (TRD)
## Google Sheets Gemini API 플러그인

**문서 버전**: 1.1  
**작성일**: 2024  
**최종 수정일**: 2024

---

## 1. 기술 개요

### 1.1 시스템 아키텍처

```
┌─────────────────────────────────────────┐
│         Google Sheets                    │
│  ┌───────────────────────────────────┐  │
│  │   Apps Script (Code.gs)          │  │
│  │                                   │  │
│  │  ┌─────────────┐  ┌───────────┐ │  │
│  │  │ Custom      │  │  Menu     │ │  │
│  │  │ Functions   │  │  Handlers │ │  │
│  │  └──────┬──────┘  └─────┬─────┘ │  │
│  │         │                │       │  │
│  │         └──────┬─────────┘       │  │
│  │                │                  │  │
│  │         ┌──────▼──────┐          │  │
│  │         │ Gemini API  │          │  │
│  │         │  Handler    │          │  │
│  │         └──────┬──────┘          │  │
│  └─────────────────┼────────────────┘  │
│                    │                     │
└────────────────────┼─────────────────────┘
                     │
         ┌───────────▼───────────┐
         │   Gemini API          │
         │   (External Service)  │
         └───────────────────────┘
```

### 1.2 기술 스택

| 구성 요소 | 기술 | 버전 |
|----------|------|------|
| 스크립트 언어 | Google Apps Script | V8 Runtime |
| API 통신 | UrlFetchApp | - |
| 데이터 저장 | PropertiesService | - |
| UI | SpreadsheetApp.getUi() | - |
| 외부 API | Gemini API | v1beta |

---

## 2. 시스템 구성 요소

### 2.1 파일 구조

```
sheets/
├── Code.gs              # 메인 스크립트 파일
├── appsscript.json      # 매니페스트 파일 (선택사항)
├── README.md            # 사용자 가이드
├── PRD.md               # 제품 요구사항 문서
└── TRD.md               # 기술 요구사항 문서
```

### 2.2 주요 모듈

#### 2.2.1 Code.gs 구조

```javascript
// 1. 상수 정의
- GEMINI_API_KEY
- DEFAULT_MODEL (gemini-2.0-flash)
- GEMINI_API_BASE_URL

// 2. 이벤트 핸들러
- onOpen()
- onInstall()

// 3. UI 관리
- createGeminiMenu()

// 4. 설정 관리
- setApiKey()
- getApiKey()
- setModel()
- getModel()

// 5. 응답 처리
- extractResultOnly(response) - 핵심 결과만 추출

// 6. API 통신
- callGeminiAPI(prompt, model, resultOnly)

// 7. 메뉴 기능
- askGeminiFromSelection()
- analyzeWithGemini()
- getGeminiResponseToNextCell()

// 8. 커스텀 함수 (셀 격리 처리)
- MyAI(prompt, cellReference) - 결과만 반환, 셀 격리
- AI(prompt) - 결과만 반환, 셀 격리
- MyAIFull(prompt, cellReference) - 전체 응답, 셀 격리

// 8. 유틸리티
- processRangeWithGemini()
```

---

## 3. API 명세

### 3.1 Gemini API 호출

#### 3.1.1 엔드포인트
```
POST https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent?key={apiKey}
```

#### 3.1.2 요청 형식
```json
{
  "contents": [{
    "parts": [{
      "text": "프롬프트 텍스트"
    }]
  }]
}
```

#### 3.1.3 응답 형식
```json
{
  "candidates": [{
    "content": {
      "parts": [{
        "text": "응답 텍스트"
      }]
    },
    "finishReason": "STOP"
  }]
}
```

#### 3.1.4 지원 모델
- `gemini-2.0-flash` (기본값, 최신 버전)
- `gemini-1.5-flash`
- `gemini-1.5-flash-latest`
- `gemini-pro`

### 3.2 내부 API

#### 3.2.1 callGeminiAPI(prompt, model, resultOnly)
**설명**: Gemini API 호출 핵심 함수

**파라미터**:
- `prompt` (string, 필수): Gemini에 보낼 프롬프트
- `model` (string, 선택): 사용할 모델 (기본값: getModel())
- `resultOnly` (boolean, 선택): 결과만 반환할지 여부 (기본값: false)

**반환값**: 
- `string`: Gemini의 응답 텍스트 (resultOnly=true일 경우 핵심 결과만)

**에러 처리**:
- API 키 미설정
- 프롬프트 비어있음
- 네트워크 오류 (403, 429 등)
- 응답 파싱 실패

#### 3.2.2 extractResultOnly(response)
**설명**: 응답에서 핵심 결과만 추출

**파라미터**:
- `response` (string, 필수): Gemini의 전체 응답

**반환값**: 
- `string`: 설명 제거된 핵심 결과만

**처리 로직**:
- 설명 문구 자동 제거
- 코드 블록과 결과만 유지
- 줄바꿈 처리

#### 3.2.3 MyAI(prompt, cellReference)
**설명**: 커스텀 함수 - 결과만 반환, 셀 격리 처리

**파라미터**:
- `prompt` (string, 필수): 프롬프트
- `cellReference` (string|number|Range, 선택): 셀 참조 또는 값

**반환값**: 
- `string`: 결과만 반환 (해당 셀에만 표시)

**특징**:
- 활성 스프레드시트 컨텍스트 확인
- 셀 참조 오류 격리 처리
- 자동으로 결과만 추출

#### 3.2.4 MyAIFull(prompt, cellReference)
**설명**: 커스텀 함수 - 전체 응답 반환, 셀 격리 처리

**파라미터**:
- `prompt` (string, 필수): 프롬프트
- `cellReference` (string|number|Range, 선택): 셀 참조 또는 값

**반환값**: 
- `string`: 전체 응답 (해당 셀에만 표시)

---

## 4. 데이터 저장 및 관리

### 4.1 PropertiesService 구조

| 키 | 값 | 설명 |
|---|-----|------|
| GEMINI_API_KEY | string | Gemini API 키 |
| GEMINI_MODEL | string | 선택된 모델 이름 |

### 4.2 데이터 저장 방식
```javascript
// 저장
PropertiesService.getScriptProperties().setProperty('KEY', 'value');

// 조회
PropertiesService.getScriptProperties().getProperty('KEY');
```

### 4.3 보안
- API 키는 암호화되어 저장됨
- 사용자별 스크립트 속성으로 격리
- 스크립트 실행 시에만 접근 가능

---

## 5. 권한 및 인증

### 5.1 필요한 OAuth 스코프

```json
{
  "oauthScopes": [
    "https://www.googleapis.com/auth/spreadsheets.currentonly",
    "https://www.googleapis.com/auth/script.container.ui",
    "https://www.googleapis.com/auth/script.external_request"
  ]
}
```

### 5.2 권한 설명
- **spreadsheets.currentonly**: 현재 스프레드시트 읽기/쓰기
- **script.container.ui**: UI 메뉴 생성
- **script.external_request**: 외부 API 호출 (Gemini API)

---

## 6. 에러 처리

### 6.1 에러 타입 및 처리

| 에러 코드 | 원인 | 처리 방법 | 표시 위치 |
|----------|------|----------|----------|
| API 키 미설정 | PropertiesService에 키 없음 | 사용자에게 설정 안내 | 해당 셀 |
| 403 | 권한 오류 | API 키 또는 모델 확인 | 해당 셀 |
| 429 | 요청 한도 초과 | 사용자에게 대기 안내 | 해당 셀 |
| 네트워크 오류 | 연결 실패 | 재시도 안내 | 해당 셀 |
| 파싱 오류 | 응답 형식 불일치 | 에러 로그 기록 | 해당 셀 |
| 셀 참조 오류 | 잘못된 셀 참조 | 오류 무시하고 계속 진행 | 해당 셀 |
| 활성 시트 없음 | 스프레드시트 컨텍스트 없음 | 명확한 오류 메시지 | 해당 셀 |

### 6.2 에러 메시지 표시
- **커스텀 함수**: 셀에 에러 메시지 반환
- **메뉴 기능**: UI.alert() 사용
- **로그**: Logger.log() 사용

---

## 7. 성능 최적화

### 7.1 최적화 전략
1. **API 호출 최소화**: 불필요한 호출 방지
2. **에러 캐싱**: 동일 에러 반복 방지
3. **비동기 처리**: 가능한 경우 비동기 처리 (현재는 동기)
4. **결과 추출**: 설명 제거로 응답 크기 최소화
5. **셀 격리**: 각 셀 독립 처리로 성능 영향 최소화

### 7.2 성능 목표
- API 호출 응답 시간: 3초 이내
- 함수 실행 시간: 5초 이내
- 메모리 사용: 최소화

### 7.3 제한사항
- Google Apps Script 실행 시간: 6분
- 동시 실행 제한: 순차 처리
- API 호출 제한: Gemini 무료 티어 제한 적용

---

## 8. 보안 요구사항

### 8.1 API 키 보안
- ✅ PropertiesService 암호화 저장
- ✅ 코드에 하드코딩 금지
- ✅ 사용자별 격리

### 8.2 데이터 보안
- ✅ 외부 API 호출 시 HTTPS 사용
- ✅ 사용자 데이터 로깅 금지
- ✅ 민감 정보 전송 시 암호화

### 8.3 접근 제어
- ✅ 스크립트 소유자만 접근 가능
- ✅ 공유 스크립트의 경우 권한 확인

---

## 9. 테스트 계획

### 9.1 단위 테스트
- API 키 설정/조회
- 모델 선택/조회
- 프롬프트 검증
- 에러 처리

### 9.2 통합 테스트
- Gemini API 호출
- 응답 파싱
- 셀 업데이트
- 메뉴 생성

### 9.3 사용자 테스트
- 설치 프로세스
- 기본 기능 사용
- 에러 시나리오
- 성능 테스트

### 9.4 테스트 시나리오

#### 시나리오 1: 커스텀 함수 테스트
```
1. API 키 설정
2. =MyAI("테스트", A1) 입력
3. 응답 확인
```

#### 시나리오 2: 메뉴 기능 테스트
```
1. 셀에 텍스트 입력
2. 메뉴에서 "선택한 셀 분석하기" 선택
3. 결과 확인
```

#### 시나리오 3: 에러 처리 테스트
```
1. API 키 제거
2. 함수 실행
3. 에러 메시지 확인
```

---

## 10. 배포 및 운영

### 10.1 배포 방법
1. Google Apps Script 프로젝트 생성
2. Code.gs 파일 업로드
3. appsscript.json 설정 (선택사항)
4. 권한 승인
5. 스프레드시트에 연결

### 10.2 버전 관리
- Git 사용 (선택사항)
- 버전 태깅
- 변경 이력 관리

### 10.3 모니터링
- 에러 로그 확인
- 사용량 모니터링
- 성능 모니터링

### 10.4 업데이트 프로세스
1. 코드 수정
2. 테스트
3. 배포
4. 사용자 안내

---

## 11. 기술적 제약사항

### 11.1 Google Apps Script 제약
- 실행 시간 제한: 6분
- 메모리 제한: 250MB
- 동시 실행 제한: 있음

### 11.2 Gemini API 제약
- 무료 티어 일일 요청 제한
- 응답 길이 제한
- 모델별 제한사항

### 11.3 네트워크 제약
- 인터넷 연결 필수
- 방화벽 설정 고려
- 타임아웃 처리

---

## 12. 향후 개선 사항

### 12.1 기술적 개선
- [x] 결과만 추출 기능 (extractResultOnly)
- [x] 셀 격리 처리
- [ ] 배치 처리 기능
- [ ] 응답 캐싱
- [ ] 비동기 처리
- [ ] 재시도 로직
- [ ] 사용량 모니터링

### 12.2 기능 개선
- [ ] 여러 셀 동시 처리
- [ ] 템플릿 기능
- [ ] 자동화 워크플로우
- [ ] 다른 AI 모델 지원

### 12.3 성능 개선
- [ ] 응답 시간 단축
- [ ] 메모리 사용 최적화
- [ ] 에러 처리 개선

---

## 13. 참고 자료

### 13.1 공식 문서
- [Google Apps Script 문서](https://developers.google.com/apps-script)
- [Gemini API 문서](https://ai.google.dev/docs)
- [PropertiesService 문서](https://developers.google.com/apps-script/reference/properties/properties-service)

### 13.2 API 참조
- [Gemini API Reference](https://ai.google.dev/api)
- [Google Sheets API](https://developers.google.com/sheets/api)

---

## 14. 문서 승인

| 역할 | 이름 | 승인일 | 서명 |
|------|------|--------|------|
| 기술 리더 | | | |
| 개발자 | | | |
| QA 엔지니어 | | | |

