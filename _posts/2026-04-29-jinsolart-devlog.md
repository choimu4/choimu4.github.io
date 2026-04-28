---
layout: post
title: 진솔아트(JINSOLART) 홈페이지 개발 작업 일지
subtitle: Firebase Hosting + Functions + 네이버 블로그 RSS 연동
author: 민욱 최
excerpt_image: ../assets/images/jinsol.png
categories: Swift
banner:
  image: ../assets/images/jinsol.png
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 3.5em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
tags: Firebase JavaScript DevLog
---

> 작성일: 2026-04-29  
> 배포 URL: https://jinsolart-55b0d.web.app  
> 프로젝트 경로: `/portpolio/jinsolart/`

---

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [기술 스택](#2-기술-스택)
3. [디렉토리 구조](#3-디렉토리-구조)
4. [Phase 1 — 정적 사이트 구현](#4-phase-1--정적-사이트-구현)
5. [Phase 2 — Firebase 호스팅 전환](#5-phase-2--firebase-호스팅-전환)
6. [Phase 3 — 네이버 블로그 RSS 연동](#6-phase-3--네이버-블로그-rss-연동)
7. [Phase 4 — 문의 폼 + Gmail 발송](#7-phase-4--문의-폼--gmail-발송)
8. [Phase 5 — 버그 수정 및 UX 개선](#8-phase-5--버그-수정-및-ux-개선)
9. [트러블슈팅 기록](#9-트러블슈팅-기록)
10. [배포 방법](#10-배포-방법)
11. [남은 과제](#11-남은-과제)

---

## 1. 프로젝트 개요

간판·시공 업체 **진솔아트** 대표 박춘식 님의 사업자 홈페이지.  
네이버 블로그(pcs59190)의 실제 시공 사례 포스트를 자동으로 포트폴리오에 표시하고,  
방문객이 문의 폼을 제출하면 Gmail로 알림 메일이 발송된다.

| 항목 | 내용 |
|------|------|
| 업종 | 간판·채널 LED·필름·인테리어 시공 |
| 대표 연락처 | 010-4399-6426 |
| 블로그 | https://blog.naver.com/pcs59190 |
| 배포 환경 | Firebase Hosting + Functions (서울 리전) |

---

## 2. 기술 스택

| 레이어 | 선택 | 이유 |
|--------|------|------|
| 프론트엔드 | Vanilla JS + CSS (빌드 도구 없음) | 소규모 사이트, 빠른 로드 속도 우선 |
| 호스팅 | Firebase Hosting | CDN 자동 배포, 무료 플랜 충분 |
| API 서버 | Firebase Functions v2 (Node 20) | 호스팅과 동일 프로젝트, Secret Manager 통합 |
| 메일 발송 | Nodemailer + Gmail SMTP (STARTTLS) | 별도 메일 서버 불필요 |
| 인증 정보 | Firebase Secret Manager | 코드·환경 변수 파일에 노출 없음 |
| 이미지 프록시 | images.weserv.nl | 네이버 CDN hotlink 차단 우회 + 글로벌 캐싱 |

---

## 3. 디렉토리 구조

```
jinsolart/
├── public/                  # Firebase Hosting 배포 대상
│   ├── index.html           # 단일 페이지 HTML (SPA)
│   ├── style.css            # 전체 스타일 (다크 네온 테마)
│   └── main.js              # 클라이언트 JS (RSS·폼·애니메이션)
│
├── functions/               # Firebase Functions v2
│   ├── index.js             # API 라우터 (GET /api/rss, POST /api/contact)
│   ├── package.json         # functions 전용 의존성
│   └── node_modules/        # (gitignore)
│
├── firebase.json            # Hosting + Functions + rewrite 규칙
├── .firebaserc              # 연결된 Firebase 프로젝트 ID
├── server.js                # (참고용) 로컬 개발용 Express 원본
└── .gitignore               # .env, node_modules, .DS_Store 제외
```

---

## 4. Phase 1 — 정적 사이트 구현

### 4-1. 페이지 구성 (index.html)

단일 HTML에 모든 섹션을 배치하는 SPA 구조.

| 섹션 | 역할 |
|------|------|
| `#hero` | 풀스크린 배경 + CTA 버튼 2개 |
| `#services` | 서비스 카드 6종 (채널, 필름, LED, 아크릴, 현수막, 인테리어) |
| `#portfolio` | 네이버 블로그 RSS 기반 동적 카드 그리드 |
| `#strengths` | 강점 4카드 |
| `#stats` | 숫자 카운터 애니메이션 (경력·시공·만족도) |
| `#contact` | 문의 폼 (이름·연락처·이메일·서비스·위치·내용) |
| `footer` | 사업자 정보 + 연락처 |

### 4-2. 디자인 시스템 (style.css)

- **컬러**: 배경 `#0a0a0a`, 포인트 `#00f5a0` (네온 그린), 보조 `#00b4d8` (네온 블루)
- **타이포**: `'Apple SD Gothic Neo', Arial, sans-serif`
- **반응형**: 768px (태블릿), 480px (모바일) 두 개 브레이크포인트

### 4-3. 클라이언트 JS 주요 기능 (main.js)

```
1. 상수 정의        — BLOG_ID, RSS_URL, FALLBACK_GRADIENTS
2. NAVIGATION      — 스크롤 배경 전환 + 부드러운 스크롤 앵커 + 모바일 햄버거 메뉴
3. BLOG RSS 로드   — fetch('/api/rss') → XML 파싱 → 포트폴리오 동적 생성
4. PORTFOLIO FILTER — 카테고리 탭 (전체·채널·필름·LED·기타)
5. LIGHTBOX        — 카드 클릭 시 이미지 확대 팝업 + 블로그 링크 버튼
6. COUNTER         — IntersectionObserver 기반 숫자 카운터 애니메이션
7. CONTACT FORM    — 유효성 검사 + fetch POST + 발송 성공/실패 UI
```

---

## 5. Phase 2 — Firebase 호스팅 전환

### 5-1. 배경

초기에는 로컬 `server.js`(Express)로 개발. 배포를 위해 Firebase로 전환.

### 5-2. firebase.json 구성

```json
{
  "hosting": {
    "public": "public",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      { "source": "/api/**", "function": "api", "region": "asia-northeast3" }
    ]
  },
  "functions": { "source": "functions" }
}
```

- `/api/**` 경로는 Functions로 라우팅, 나머지는 Hosting이 정적 파일 서빙
- `region: asia-northeast3` = 서울 리전 (응답 속도 최소화)

### 5-3. functions/index.js 이식 포인트

| server.js (로컬) | functions/index.js (Firebase) |
|-----------------|-------------------------------|
| `express.static()` | 제거 (Hosting이 처리) |
| `app.listen(PORT)` | 제거 (Functions가 처리) |
| `process.env.GMAIL_USER` | `defineSecret('GMAIL_USER').value()` |
| `require('dotenv').config()` | 제거 (Secret Manager 사용) |

### 5-4. Secret Manager 등록 (최초 1회)

```bash
firebase functions:secrets:set GMAIL_USER   # 발신 Gmail 주소
firebase functions:secrets:set GMAIL_PASS   # Gmail 앱 비밀번호 (16자리)
firebase functions:secrets:set MAIL_TO      # 수신 이메일 주소
```

### 5-5. functions/package.json

```json
{
  "name": "jinsolart-functions",
  "engines": { "node": "20" },
  "dependencies": {
    "firebase-functions": "^6.0.0",
    "cors": "^2.8.5",
    "nodemailer": "^6.9.13"
  }
}
```

---

## 6. Phase 3 — 네이버 블로그 RSS 연동

### 6-1. 흐름

```
브라우저 → GET /api/rss → Firebase Function → rss.blog.naver.com → XML 반환
                                                         ↑
                                          서버 사이드 요청이므로 CORS 무관
```

### 6-2. GET /api/rss (functions/index.js)

```js
app.get('/api/rss', async (req, res) => {
  const response = await fetch('https://rss.blog.naver.com/pcs59190', {
    headers: { 'User-Agent': 'Mozilla/5.0 (compatible; RSS-Reader/1.0)' },
  });
  const xml = await response.text();
  res.set('Content-Type',  'application/xml; charset=utf-8');
  res.set('Cache-Control', 'public, max-age=3600');  // 1시간 캐시
  res.status(200).send(xml);
});
```

### 6-3. 클라이언트 RSS 파싱 (main.js)

```js
const RSS_URL = '/api/rss';  // 상대 경로 → 같은 도메인 Functions 호출

async function loadPortfolio() {
  const res  = await fetch(RSS_URL);
  const xml  = await res.text();
  const doc  = new DOMParser().parseFromString(xml, 'text/xml');
  const items = Array.from(doc.querySelectorAll('item'));
  items.slice(0, 12).forEach((post, i) => {
    grid.appendChild(createPortfolioItem(post, i));
  });
}
```

### 6-4. 이미지 프록시 (images.weserv.nl)

네이버 CDN은 외부 도메인에서의 이미지 직접 로드를 403으로 차단(hotlink protection).  
`images.weserv.nl`을 통해 URL을 래핑하면 프록시 서버가 네이버에서 가져와 캐싱·전달.

```js
// extractImage()가 반환한 네이버 원본 URL
const rawUrl   = extractImage(description);
// images.weserv.nl 프록시로 래핑
const imageUrl = rawUrl
  ? `https://images.weserv.nl/?url=${encodeURIComponent(rawUrl)}`
  : null;
```

### 6-5. 카테고리 자동 분류

제목·설명 텍스트에 키워드를 검색해 카테고리 자동 부여:

| 카테고리 | 주요 키워드 |
|---------|-----------|
| 채널 | 채널, 입체, 돌출, 3D, 볼륨 |
| 필름 | 필름, 시트, 래핑, 유리, 시공 |
| LED | LED, 전광판, 조명, 발광 |
| 아크릴 | 아크릴, 투명, 판넬 |
| 현수막 | 현수막, 배너, 플랙스 |
| 인테리어 | 인테리어, 리모델링, 인테리어필름 |

---

## 7. Phase 4 — 문의 폼 + Gmail 발송

### 7-1. 폼 필드 및 유효성 검사

| 필드 | 타입 | 필수 | 검사 규칙 |
|------|------|------|-----------|
| 이름 | text | ✅ | 한글·영문·공백만 허용 (IME 조합 중 필터 제외) |
| 연락처 | tel | ✅ | 숫자 자동 포맷 (010-XXXX-XXXX / 02-XXXX-XXXX) |
| 이메일 | email | ✅ | 표준 이메일 형식 정규식 |
| 서비스 | select | ✅ | 선택 안 함 방지 |
| 시공 위치 | text | ✗ | 선택 입력 |
| 문의 내용 | textarea | ✗ | 500자 제한 + 실시간 글자 수 표시 |

### 7-2. 한국어 IME 처리

모바일에서 한글 입력 시 `compositionstart` / `compositionend` 이벤트로  
조합 중인 문자(ㅎ → 하 → 한)를 필터에서 제외:

```js
let isComposing = false;
nameInput.addEventListener('compositionstart', () => { isComposing = true;  });
nameInput.addEventListener('compositionend',   () => { isComposing = false; });
nameInput.addEventListener('input', () => {
  if (isComposing) return;  // 조합 중에는 필터 실행 안 함
  // 한글·영문·공백 외 문자 제거
  nameInput.value = nameInput.value.replace(/[^가-힣a-zA-Z\s]/g, '');
});
```

### 7-3. POST /api/contact (functions/index.js)

```
요청 바디: { name, phone, email, service, location, message }
처리 순서:
  1. 필수 필드 누락 검사 → 400 반환
  2. Secret Manager에서 Gmail 자격 증명 로드
  3. Nodemailer transporter 생성 (smtp.gmail.com:587, STARTTLS)
  4. HTML 이메일 구성 (네온 다크 테마 인라인 스타일)
  5. 발송 성공 → 200 { ok: true }
     발송 실패 → 500 { ok: false, error: "..." }
```

### 7-4. 발송 이메일 형식

- **From**: `"진솔아트 홈페이지" <gmail_user>`
- **To**: Secret Manager의 `MAIL_TO` 값
- **Reply-To**: 문의자 이메일 (Gmail 답장 버튼으로 바로 회신 가능)
- **Subject**: `[진솔아트 문의] {이름} - {서비스}`
- **HTML**: 네온 다크 테마, 연락처 `tel:` 링크, 이메일 `mailto:` 링크 포함

---

## 8. Phase 5 — 버그 수정 및 UX 개선

### 8-1. 모바일 반응형

```css
/* 태블릿 (≤ 768px) */
@media (max-width: 768px) {
  .services-grid { grid-template-columns: repeat(2, 1fr); }
  .stat-item     { padding: 28px 16px; }
}

/* 모바일 (≤ 480px) */
@media (max-width: 480px) {
  section        { padding: 48px 0; }
  .container     { padding: 0 16px; }
  .services-grid { grid-template-columns: 1fr; }
  .hero-btns     { flex-direction: column; }
}
```

### 8-2. 모바일 내비게이션 메뉴 숨김

`top: 64px` + `translateY(-120%)`로 설정했을 때 메뉴 하단 ~19px이 보이는 문제.  
완전히 뷰포트 위로 올리는 공식으로 수정:

```css
/* 수정 전 */
.nav-menu { transform: translateY(-120%); }

/* 수정 후 — 자신의 높이 + nav 높이(64px) 만큼 올려 완전히 숨김 */
.nav-menu { transform: translateY(calc(-100% - 64px)); }
```

### 8-3. 한국어 텍스트 줄바꿈

서비스 카드 설명 텍스트가 음절 단위로 잘려 가독성 저하.

```css
.service-card p {
  word-break: keep-all;  /* 어절 단위로 줄바꿈 */
}
```

---

## 9. 트러블슈팅 기록

### GQL 스키마 오류

**증상**: `firebase deploy` 시 `Cannot query field "id" on type "Artist_KeyOutput"` 오류  
**원인**: Firebase Data Connect의 `_insert` mutation은 `_KeyOutput` 타입을 반환하며 하위 필드 없음  
**해결**: `dataconnect/example/queries.gql`의 `artist_insert { id }`, `inquiry_insert { id }` 에서 `{ id }` 제거

---

### Secret 환경변수 충돌

**증상**: `functions/.env`에 `GMAIL_USER` 등이 정의된 상태에서 Secret Manager에도 동일 이름 등록 → Cloud Run 배포 실패  
**원인**: 같은 이름의 변수가 `.env`와 Secret Manager 양쪽에 존재하면 충돌  
**해결**: `functions/.env` 파일 삭제 (Secret Manager만 사용)

---

### RSS CORS 차단

**증상**: 공개 CORS 프록시(corsproxy.io, allorigins.win)가 Firebase Hosting 도메인을 차단 → 403  
**원인**: 일부 CORS 프록시 서비스가 Firebase 도메인을 악용 방지 차원에서 차단  
**해결**: Firebase Functions에 `GET /api/rss` 서버 사이드 프록시 구현

---

### 네이버 이미지 403

**증상**: 포트폴리오 이미지가 일부 환경에서 로드되지 않음  
**원인**: 네이버 CDN이 외부 도메인 Referer를 감지해 hotlink 차단  
**해결 시도 1**: `<img referrerpolicy="no-referrer">` → 일부 환경에서 여전히 실패  
**해결 시도 2**: `images.weserv.nl` 프록시로 URL 래핑 → 프록시가 네이버에서 가져와 전달하므로 안정적

```js
// 최종 적용 코드
const rawUrl   = extractImage(description);
const imageUrl = rawUrl
  ? `https://images.weserv.nl/?url=${encodeURIComponent(rawUrl)}`
  : null;
```

---

### 한국어 이름 입력 불가 (모바일)

**증상**: 모바일에서 이름 필드에 한글 입력 시 글자가 바로 삭제됨  
**원인**: 한글 IME 조합 중인 문자(e.g. `ㅎ`)가 정규식 필터와 충돌  
**해결**: `compositionstart` / `compositionend` 이벤트로 조합 중 필터 비활성화

---

## 10. 배포 방법

### 최초 배포 (Secret 등록 포함)

```bash
# 1. Secret 등록 (최초 1회)
firebase functions:secrets:set GMAIL_USER
firebase functions:secrets:set GMAIL_PASS
firebase functions:secrets:set MAIL_TO

# 2. Functions 의존성 설치
cd functions && npm install && cd ..

# 3. 전체 배포
npx firebase deploy
```

### 정적 파일만 업데이트

```bash
npx firebase deploy --only hosting
```

### Functions만 업데이트

```bash
npx firebase deploy --only functions
```

---

## 11. 남은 과제

| 우선순위 | 항목 | 비고 |
|---------|------|------|
| 높음 | 커스텀 도메인 연결 | Firebase Hosting 콘솔 → 도메인 추가 → DNS CNAME 설정 |
| 중간 | Open Graph 메타 태그 | SNS 공유 시 썸네일·설명 표시 |
| 중간 | Google Analytics 연동 | 방문자 통계 수집 |
| 낮음 | 이미지 Lazy Load 스켈레톤 | 로드 중 빈 카드 개선 |
| 낮음 | 폼 제출 후 감사 메시지 개선 | 현재 토스트 알림만 있음 |
