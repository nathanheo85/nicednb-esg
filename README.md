# NICE D&B ESG — Contact Us 연동 가이드

랜딩페이지 문의 → Google Sheets → Admin 페이지 실시간 반영

```
랜딩페이지 (GitHub Pages)
    │  POST (fetch)
    ▼
Google Apps Script  ←─────────────────┐
    │  appendRow / update              │
    ▼                                  │
Google Sheets (DB)                     │
    │  GET (fetch)          PATCH (상태변경)
    ▼                                  │
Admin 페이지 ──────────────────────────┘
(GitHub Pages)
```

---

## 📁 레포지토리 구조

```
nicednb-esg/
├── index.html                  ← 랜딩페이지 (나이스디앤비_ESG_Landing_Page)
├── admin.html                  ← Admin 페이지 (nicednb_esg_admin_full)
├── apps-script/
│   └── Code.gs                 ← Google Apps Script (복사해서 붙여넣기용)
└── README.md
```

---

## 🚀 배포 순서

### Step 1. GitHub 레포 생성

```bash
# 로컬에 폴더 만들기
mkdir nicednb-esg && cd nicednb-esg

# 파일 배치
# - 랜딩페이지 → index.html
# - Admin 페이지 → admin.html
# - Apps Script 코드 → apps-script/Code.gs (참고용)

git init
git add .
git commit -m "initial commit"
git remote add origin https://github.com/[계정명]/nicednb-esg.git
git push -u origin main
```

---

### Step 2. Google Sheets + Apps Script 설정

#### 2-1. Google Sheets 생성
1. [Google Sheets](https://sheets.google.com) 접속
2. 빈 스프레드시트 새로 만들기
3. 파일명: `NICE D&B ESG 문의 관리`

#### 2-2. Apps Script 코드 등록
1. 스프레드시트 상단 메뉴 → **확장 프로그램** → **Apps Script**
2. 기존 코드 전체 삭제
3. `apps-script/Code.gs` 내용을 전체 복사해서 붙여넣기
4. **저장** (Ctrl+S / Cmd+S)

#### 2-3. 웹앱으로 배포
1. 오른쪽 상단 **배포** 버튼 클릭
2. **새 배포** 선택
3. 설정:
   - 유형: **웹 앱**
   - 설명: `NICE D&B ESG Contact API`
   - 실행 계정: **나(본인 Google 계정)**
   - 액세스 권한: **모든 사용자**
4. **배포** 클릭
5. Google 계정 권한 허용
6. **배포 URL 복사** (이 URL이 API 엔드포인트)

> ⚠️ URL 형식 예시:
> `https://script.google.com/macros/s/AKfycby.../exec`

---

### Step 3. URL 두 곳에 붙여넣기

#### 3-1. 랜딩페이지 (`index.html`)
파일에서 아래 부분을 찾아 교체:

```js
// 기존 handleSubmit() 함수 전체를 contact-submit.js 내용으로 교체
// 그리고 상단 SCRIPT_URL 변수에 복사한 URL 붙여넣기

const SCRIPT_URL = 'https://script.google.com/macros/s/여기에_배포_URL_붙여넣기/exec';
                                              ↑ 여기를 실제 URL로 교체
```

**구체적인 교체 방법:**
1. `index.html` 에서 `function handleSubmit()` 검색
2. 기존 함수(alert만 하는 버전) 전체를 `contact-submit.js` 코드로 교체
3. 파일 최상단 `const SCRIPT_URL = '...'` 의 URL 수정

#### 3-2. Admin 페이지 (`admin.html`)
파일에서 아래 줄을 찾아 URL 교체:

```js
const SCRIPT_URL = 'https://script.google.com/macros/s/여기에_배포_URL_붙여넣기/exec';
                                              ↑ 여기를 실제 URL로 교체
```

---

### Step 4. GitHub Pages 배포

1. GitHub 레포 → **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: **main** / **/ (root)**
4. **Save**

배포 완료 후 URL:
- 랜딩페이지: `https://[계정명].github.io/nicednb-esg/`
- Admin: `https://[계정명].github.io/nicednb-esg/admin.html`

---

## ✅ 동작 확인 체크리스트

```
□ 랜딩페이지 Contact Us 폼에서 테스트 문의 제출
□ Google Sheets에 새 행이 추가되는지 확인
□ Admin 페이지 로그인 후 문의가 표시되는지 확인
□ Admin에서 상태 변경 후 Sheets에 반영되는지 확인
□ CSV 내보내기 동작 확인
```

---

## 🔐 Admin 로그인 정보

| 항목 | 값 |
|------|-----|
| 아이디 | `admin` |
| 비밀번호 | `password` |

> ⚠️ 실제 운영 전 `admin.html` 내 `CREDENTIALS` 변수에서 반드시 변경하세요.

---

## ❓ 자주 발생하는 문제

### 문의 제출 후 오류가 뜨는 경우
- Apps Script 배포 시 **액세스 권한: 모든 사용자** 로 설정했는지 확인
- SCRIPT_URL이 정확히 입력됐는지 확인 (`/exec` 로 끝나야 함)

### Admin에서 데이터가 안 뜨는 경우
- 브라우저 콘솔(F12) → Network 탭에서 SCRIPT_URL 요청 응답 확인
- Apps Script 편집기 → **실행** → `doGet` 함수 직접 테스트

### Apps Script 재배포가 필요한 경우
코드 수정 후에는 반드시 **새 배포** (버전 업)를 해야 반영됩니다.
기존 배포 URL은 그대로 유지됩니다.

---

## 📊 Google Sheets 컬럼 구조

| 컬럼 | 내용 | 예시 |
|------|------|------|
| id | 자동 생성 ID | INQ-001 |
| date | 접수일 | 2026-04-10 |
| company | 기업명 | Hanwha Group |
| name | 담당자 성명 | 김민준 |
| email | 이메일 | email@company.com |
| phone | 연락처 | 02-1234-5678 |
| country | 국가 (항상 한글) | 대한민국 |
| subject | 제목 | ESG 평가 도입 검토 |
| msg | 문의 내용 | 안녕하세요... |
| status | 처리 상태 | 신규 / 처리 중 / 완료 / 종결 |
