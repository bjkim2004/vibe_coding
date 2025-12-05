# YouTube STT 요약 + 영상 주요 장면 캡처 도구
# (최종 PRD & TRD — 실행 가능 설계서 버전)

---

# 1. 제품 개요 (Product Overview)

웹에서 YouTube URL을 입력하면 영상 다운로드 → 오디오 추출 → Whisper 로컬 STT 처리 → 한국어 요약 → 장면 캡처 이미지 생성까지 수행하는 **개인용 도구**.

※ 유의사항: 유튜브 영상 다운로드는 개인/연구 목적의 로컬 활용용이며, 공개 서비스 배포 시 저작권/ToS 검토가 필요함.

---

# 2. 시스템 특성 (아키텍처 원칙)

✔ 완전 로컬 STT (Whisper 모델 다운로드 후 오프라인 처리 가능)  
✔ 로그인/회원제 불필요  
✔ 단순 단일 사용자 기준 (동시다중 처리 미고려)  
✔ 영상 길이가 길어질 경우 처리시간 길어짐을 사용자에게 안내  
✔ GPU 있으면 더 빠르게, 없으면 느리지만 정상 작동

---

# 3. 목표 사용자

- 유튜브 강의/인터뷰/토론 영상의 내용을 텍스트로 빠르게 보고 싶은 개인  
- 개인 학습/연구용  
- UI/UX 최소한의 단순 기능 중심 도구 선호자

---

# 4. 기능 요구사항 (MVP)

## 4.1 URL 입력 및 검증
- 유효한 YouTube URL인지 체크
- 잘못된 URL일 경우 메시지 안내

---

## 4.2 영상 다운로드 (yt-dlp)
- 최고 음질(best audio) 가능
- 다운로드 실패 시 에러 안내
- 처리 중 로딩 상태 표시
- 다운로드 시간 사용자 안내 (예: 40초 예상)

---

## 4.3 오디오 추출 (ffmpeg)
- video.mp4 → audio.wav
- 16kHz, mono 변환 → Whisper 최적화

---

## 4.4 Whisper STT (로컬)
- small 모델을 기본값으로 사용
- medium 모델은 GPU 있을 때만 사용
- 모델 로드는 요청마다 하지 않고 **서버 실행 시 1회만 로딩**
- STT 결과:
  - `text`
  - `segments` (타임라인 포함)

---

## 4.5 한국어 요약 기능
- 요약 엔진 고정:
  - **로컬 요약 알고리즘(TextRank 기반)**  
    → API 미사용  
    → 완전 오프라인 가능  
- 요약 결과 형식
핵심 요약 포인트

핵심 주제

결론

yaml
코드 복사

---

## 4.6 장면 캡처 기능
- ffmpeg scene detection 사용  
- 기본 threshold = 0.3  
- 너무 많은 컷이 생길 경우 상한 제한:
  - 최대 20장까지만

---

## 4.7 다운로드 기능
- STT 텍스트(.txt)
- 자막(.srt)
- 요약(.txt)
- 캡처이미지(.zip)

---

# 5. 비기능 요구사항

## ✔ 성능
- Whisper small + GPU → 10분 영상 → ~20–40초
- Whisper small + CPU → 10분 영상 → ~2–4분  
→ 사용자에게 처리 시간 안내 필수

---

## ✔ 안정성
- 처리 실패 시:
  - 사용자에게 명확한 원인 메시지
- 예: “오디오 추출 실패 — 코덱 문제 가능”

---

## ✔ UX 사용성
- 단일 페이지 UI
- 진행 상태 표시
- "기다리는 중" 시뮬레이션 표시

---

## ✔ 프라이버시 및 저장 정책
- 모든 데이터는 `temp/{task_id}`에 저장
- 작업 종료 후:
  - 사용자 수동 삭제 버튼 제공
- 서버 자동 삭제 기능은 불필요(개인 사용 기준)

---

# 6. UI 흐름 (확정)

[ URL 입력 ]
↓
[ 처리중 — 단계별 표시 ]
↓
[ 요약 결과 표시 ]
[ 전체 텍스트 표시 ]
[ 이미지 썸네일 그리드 표시 ]
↓
[ 다운로드 버튼 ]

yaml
코드 복사

---

# 7. 기술 요구사항 (TRD)

---

# TRD — 기술 명세

# 1. 시스템 구조

React frontend → FastAPI backend → yt-dlp / ffmpeg / Whisper

yaml
코드 복사

---

# 2. Backend API 설계

## POST /api/process  (동기 처리)
**개인 사용 기준 → 비동기 큐 불필요**

요청:
```json
{
  "youtube_url": "https://youtu.be/xxxxx"
}
응답:

json
코드 복사
{
  "status": "ok",
  "task_id": "b821bcbb",
  "text": "...",
  "summary": "...",
  "captured_images": [
    "/static/captures/b821bcbb/frame_001.jpg",
    "/static/captures/b821bcbb/frame_002.jpg"
  ]
}
3. Whisper 모델 전략 (중요 업데이트)
서버 실행 시 모델 로딩

요청마다 로딩❌

small을 기본 모델로 하되 GPU 감지 시 medium 옵션 가능:

scss
코드 복사
if torch.cuda.is_available():
    model = whisper.load_model("medium")
else:
    model = whisper.load_model("small")
4. 요약 엔진 확정
기존: LLM or 요약 (미정)
업데이트:
✔ TextRank 기반 추출요약 사용
✔ 100% 로컬
✔ Whisper 자막 문장 단위 활용

5. 영상 처리 파이프라인
scss
코드 복사
yt-dlp(video) → video.mp4
ffmpeg(video.mp4) → audio.wav
Whisper(audio.wav) → text + segments
TextRank(text) → summary
ffmpeg(video.mp4, scene=0.3) → frames/*.jpg
6. temp 디렉토리 구조
bash
코드 복사
/temp/{task_id}/video.mp4
/temp/{task_id}/audio.wav
/temp/{task_id}/text.txt
/temp/{task_id}/subtitle.srt
/temp/{task_id}/frames/*.jpg
task_id = uuid4()

7. 디렉토리 구조 (최종)
cpp
코드 복사
project/
  backend/
    app.py
    whisper_model_loader.py
    stt.py
    summarize.py
    capture.py
    downloader.py
  frontend/
  temp/
  static/
    captures/
8. 예외 처리 규칙
상황	시스템 응답
URL 오류	“유효하지 않은 YouTube URL입니다.”
영상 다운로드 실패	“영상 다운로드에 실패했습니다.”
Whisper 오류	“STT 처리 중 오류 발생”
ffmpeg 캡처 오류	“장면 캡처 실패 — ffmpeg 문제”

9. 프론트엔드 반응형 UI
모바일에서도 사용 가능

화면 폭 < 900px → 2열 캡처 이미지 표시

10. 실행 환경 요구사항
CPU only OK
GPU 있으면 성능 향상
NVIDIA CUDA 세팅시 선택적 활용