### 기능
1. 대상사이트 : https://play.google.com/store/apps/details?id={앱ID}&hl=ko
2. playwright사용하여 크롤링
3. 앱 ID를 화면에 입력하면 상품정보(앱ID, 앱명, 리뷰수, 다운로드수) 수집
3.1 앱명 : #yDmH0d > c-wiz.SSPGKf.Czez9d > div > div > div:nth-child(1) > div > div:nth-child(1) > div > div > c-wiz > div.hnnXjf > div.Il7kR > div > div > div.Fd93Bb.ynrBgc.xwcR9d > h1 > span
3.2 리뷰수 : #yDmH0d > c-wiz.SSPGKf.Czez9d > div > div > div:nth-child(1) > div > div:nth-child(1) > div > div > c-wiz > div.hnnXjf > div.JU1wdd > div > div > div:nth-child(1) > div.g1rdde
3.3 다운로드수: #yDmH0d > c-wiz.SSPGKf.Czez9d > div > div > div:nth-child(1) > div > div:nth-child(1) > div > div > c-wiz > div.hnnXjf > div.JU1wdd > div > div > div:nth-child(2) > div.ClM7OO
4. 수집된 '앱정보'테이블에 저장
5. '#ow10 > section > div > div.Jwxk6d > div:nth-child(5) > div > div > button > span' 클릭
6. 앱리뷰(별점, 리뷰작성일, 리뷰내용) 정보 수집
6.1 별점: <path d="M12 17.27L18.18 21l-1.64-7.03L22 9.24l-7.19-.61L12 2 9.19 8.63 2 9.24l5.46 4.73L5.82 21z"></path>
6.1 리뷰내용 : #yDmH0d > div.VfPpkd-Sx9Kwc.cC1eCc.UDxLd.PzCPDd.HQdjr.VfPpkd-Sx9Kwc-OWXEXe-FNFY6c > div.VfPpkd-wzTsW > div > div > div > div > div.fysCi.Vk3ZVd > div > div:nth-child(2) > div:nth-child(1) > div.h3YV2d
6.2 리뷰작성일: #yDmH0d > div.VfPpkd-Sx9Kwc.cC1eCc.UDxLd.PzCPDd.HQdjr.VfPpkd-Sx9Kwc-OWXEXe-FNFY6c > div.VfPpkd-wzTsW > div > div > div > div > div.fysCi.Vk3ZVd > div > div:nth-child(2) > div:nth-child(1) > header > div.Jx4nYe > span
7. 수집된 '앱리뷰' 테이블에 저장
8. 앱정보와 앱리뷰 목록 조회 화면 출력
9. '앱리뷰 분석' 버튼을 클릭하면 gemini open api를 호출
10. gemini에 앱리뷰 전체 분석과 앱리뷰 개별 분석 요청
11. 앱리뷰 전체 분석 결과는 '앱정보' 테이블에 저장
12. 앱리뷰 개별 분석 결과는 '앱리뷰' 테이블에 저장
13. 앱리뷰 전체 분석 결과는 앱정보와 함께 출력
14. 앱리뷰 개별 분석 결과는 각 리뷰 하단에 출력
15. single page application으로 구성


### 기술
1. backend: python fast api 환경 구성
2. frontend: vite로 구성

### DB 접속 정보
postgresql://postgres.dktrdivmekmioqqkclzx:@aws-1-ap-northeast-2.pooler.supabase.com:6543/postgres

### Gemini OpenAPI Key 
