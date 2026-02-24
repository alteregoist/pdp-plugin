# PDP Plugin — 기술 분석 보고서

> **상품 상세페이지 자동 생성 플러그인 | 내부 개발팀 기술 분석 자료**
> 버전 0.1.0 · 작성자: Insoo · 2026-02-24

---

## 목차

1. [기술 아키텍처 및 모듈 구조](#1-기술-아키텍처-및-모듈-구조)
   - 1.1 플러그인 디렉토리 구조
   - 1.2 핵심 기술 스택
   - 1.3 8종 모듈 정의 및 HTML 구조
   - 1.4 멀티 플랫폼 지원 사양
   - 1.5 커맨드 & 워크플로우
2. [성능 및 최적화](#2-성능-및-최적화)
   - 2.1 병렬 빌드 전략
   - 2.2 이미지 생성 모델 선택 전략
   - 2.3 비용 최적화 — 3단계 이미지 활용 원칙
   - 2.4 상세페이지 1건당 예상 비용
3. [사용자 경험 (UX)](#3-사용자-경험-ux)
   - 3.1 대화형 정보 수집 플로우
   - 3.2 이미지 입력 방식 및 제약
   - 3.3 편집 워크플로우 (/수정하기)
   - 3.4 미리보기 & Playwright 캡처
4. [비즈니스 임팩트](#4-비즈니스-임팩트)
   - 4.1 비용 절감 효과
   - 4.2 생산성 향상
   - 4.3 확장성 및 개선 로드맵

---

## Executive Summary

PDP 플러그인은 Claude Cowork 환경 위에서 동작하는 **상품 상세페이지 자동 생성 도구**다. 판매자가 상품 이미지와 정보를 제공하면, AI가 카피라이팅·이미지 생성·HTML 빌드·스크린샷 캡처까지 일괄 처리하여 이커머스 플랫폼에 바로 업로드 가능한 상세페이지를 만들어낸다.

롯데마트몰, 쿠팡, 네이버 스마트스토어, 11번가 등 **4개 플랫폼**을 동시 지원하며, 복수 플랫폼 선택 시 Task 도구 기반 병렬 빌드로 생산성을 극대화한다. 상세페이지 1건당 이미지 생성 비용은 가성비 모드 기준 **약 $0.13(약 180원)** 수준이다.

| 항목 | 내용 |
|---|---|
| 플러그인명 | pdp (Product Detail Page Generator) |
| 버전 | 0.1.0 |
| 작성자 | Insoo |
| 배포 형태 | Claude Cowork 로컬 플러그인 |
| 핵심 키워드 | HTML+TailwindCSS, fal.ai MCP, Playwright, AI 카피라이팅, 병렬 빌드 |

---

## 1. 기술 아키텍처 및 모듈 구조

### 1.1 플러그인 디렉토리 구조

PDP 플러그인은 Claude Cowork 로컬 플러그인 포맷을 따르며, 아래와 같은 파일 구조로 구성된다.

```
pdp/
├── .claude-plugin/plugin.json      ← 플러그인 메타데이터 (이름, 버전, 설명)
├── README.md                        ← 기능 개요 및 사용법
├── commands/
│   ├── 상세페이지.md                ← /상세페이지 슬래시 커맨드 정의
│   ├── 수정하기.md                  ← /수정하기 슬래시 커맨드 정의
│   └── 미리보기.md                  ← /미리보기 슬래시 커맨드 정의
└── skills/
    └── detail-page-generator/
        ├── SKILL.md                 ← 핵심 워크플로우 정의
        └── references/
            ├── modules/
            │   ├── common.md        ← 공통 모듈 HTML 템플릿
            │   ├── 롯데마트몰.md
            │   ├── 쿠팡.md
            │   ├── 네이버스마트스토어.md
            │   └── 11번가.md
            ├── platform-configs.md  ← 플랫폼별 이미지 규격
            ├── copywriting-guide.md ← 카테고리별 카피라이팅 가이드
            └── image-generation.md  ← fal.ai 모델 레지스트리 & 프롬프트 전략
```

### 1.2 핵심 기술 스택

| 레이어 | 기술 |
|---|---|
| 프론트엔드 렌더링 | HTML5 + TailwindCSS (Play CDN) + Paperlogy 웹폰트 |
| AI 이미지 생성 | fal.ai MCP (`mcp__fal-ai__generate` / `result` / `upload` 등) |
| HTML → 이미지 캡처 | Node.js Playwright — Chromium 기반 풀페이지 스크린샷 |
| 이미지 후처리 | Node.js Sharp — 리사이즈 / 포맷 변환 (JPEG quality 90) |
| 상태 저장 | `product_info.yaml` (상품 정보) + `image_log.json` (이미지 이력) |
| 병렬 처리 | Claude Agent SDK **Task 도구** — 플랫폼별 HTML 빌드 동시 실행 |
| 런타임 요구사항 | Node.js 18+ (npm auto-install: playwright, sharp) |

> **설계 철학**: '코드를 직접 작성하지 않는 사용자'를 1순위 대상으로 설계되었다. Claude가 HTML 조립·Playwright 실행·fal.ai API 호출 등 모든 기술적 복잡성을 대신 처리하며, 사용자는 상품 정보 입력과 결과 확인만 담당한다.

### 1.3 8종 모듈 정의 및 HTML 구조

상세페이지는 **모듈(Module)** 단위로 조립된다. 각 모듈은 `<section data-module-type="X" data-module-order="N">` 구조를 가지며, 이 속성이 편집·캡처·순서 변경의 기준점이 된다.

| 모듈 | 이름 | 역할 | 필수 여부 |
|---|---|---|---|
| **A** | 상단배너 | 메인 비주얼 + 상품명 + 핵심 카피 + 가격 정보. 항상 최상단 배치. | 필수 (1순위) |
| **B** | 한눈에 보기 | 3~5개 핵심 특장점을 아이콘 카드 그리드로 표시. 빠른 스캔 유도. | 선택 |
| **C-1** | 상품 컷샷 | 이미지 + 텍스트 교대 배치(지그재그 패턴). 스토리텔링 역할. | 선택 |
| **C-2** | 상품 정보 | 텍스트 단독형. 스펙 테이블 또는 카드형. 롯데마트몰 필수. | 플랫폼 따라 |
| **D** | 인증 정보 | 인증마크 이미지 + 인증명 + 번호. 식품 카테고리 시 롯데마트몰 필수. | 식품 필수 |
| **E** | 인포그래픽 | 수치 하이라이트형 / 이미지형 / 비교형 3가지 변형 지원. | 선택 |
| **F** | 표시사항 | 법적 고지·주의사항·배송 안내. 항상 최하단. 전 플랫폼 필수. | 필수 (최하단) |
| **IMG** | 상품 이미지 | 풀너비 이미지 단독 모듈. 모듈 사이 삽입 가능. | 선택 |

**모듈 순서 규칙**: A(상단배너)는 항상 1번, F(표시사항)는 항상 마지막. 나머지 모듈은 플랫폼 권장 순서를 따르되, 사용자가 자유롭게 변경 가능하다.

#### 모듈 HTML 기본 구조

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/fonts-archive/Paperlogy/subsets/Paperlogy-dynamic-subset.css" />
  <title>{상품명} 상세페이지 — {플랫폼명}</title>
</head>
<body class="bg-white font-sans">
  <div class="max-w-[{MAX_WIDTH}px] mx-auto">
    <section data-module-type="A" data-module-order="1">...</section>
    <section data-module-type="B" data-module-order="2">...</section>
    <!-- ... -->
    <section data-module-type="F" data-module-order="N">...</section>
  </div>
</body>
</html>
```

### 1.4 멀티 플랫폼 지원 사양

플랫폼마다 이미지 규격·모듈 구성 순서·필수 모듈이 다르다. 공통 파일(`common.md`)을 베이스로 하고 플랫폼 전용 파일로 오버라이드하는 **계층적 상속 구조**를 사용한다.

| 플랫폼 | 최대 너비 | 배너 크기 | 필수 모듈 | 특이사항 |
|---|---|---|---|---|
| **롯데마트몰** | 860px | 860×480px | A, C-2, F (식품: +D) | 기본 플랫폼. F 모듈 반드시 최하단. |
| **쿠팡** | 780px | 780×440px | A, F | 이미지 중심. 모바일 트래픽 최적화 중요. |
| **네이버 스마트스토어** | 860px | 860×480px | A, F | SEO 친화적 텍스트·alt 텍스트 권장. |
| **11번가** | 830px | 830×470px | A, F | 모듈 순서: A→C-1→B→E→C-2→D→F |

**모듈 파일 로드 순서** (플랫폼별 수행):
1. `references/modules/common.md` — 공통 기본 템플릿 로드
2. `references/modules/{플랫폼명}.md` — 플랫폼 전용 오버라이드 로드
3. 플랫폼 파일에 정의된 모듈은 해당 템플릿 사용, 미정의 모듈은 common.md를 플랫폼 최대 너비에 맞게 적용

### 1.5 커맨드 & 워크플로우 개요

| 커맨드 | 동작 |
|---|---|
| `/상세페이지 [상품명]` | 신규 상세페이지 생성: 이미지 수집 → 카피라이팅 → 이미지 생성 → HTML 빌드 → 캡처 |
| `/수정하기 [HTML 경로]` | 기존 상세페이지 모듈 편집·추가·삭제·순서 변경 → 재캡처 |
| `/미리보기 [HTML 경로]` | Playwright로 전체 페이지 또는 모듈별 PNG 캡처 |

#### 5단계 생성 파이프라인 (`/상세페이지`)

```
Step 1  참조 파일 로드
        SKILL.md, modules/*.md, platform-configs.md,
        copywriting-guide.md, image-generation.md 로드

Step 2  이미지 수집 (최우선)
        ① URL 직접 제공  →  ② 작업폴더 파일 검색  →  ③ 채팅 첨부(불가, 안내)

Step 3  상품 정보 수집
        AskUserQuestion: 상품명 · 카테고리 · 플랫폼(복수 선택) · 핵심 특장점

Step 4  카피라이팅 + 이미지 생성
        copywriting-guide 기반 카피 생성 → product_info.yaml 저장
        → fal.ai 이미지 생성 (3단계 우선순위 적용)

Step 5  HTML 빌드 + Playwright 캡처
        단일 플랫폼: 순차 처리
        복수 플랫폼: Task 도구로 플랫폼별 병렬 실행
        → full.png + modules/*.png 캡처
```

#### 출력 파일 구조

**단일 플랫폼:**
```
{상품명}/
├── detail_page.html      ← 상세페이지 HTML
├── full.png              ← 전체 페이지 이미지
├── product_info.yaml     ← 상품 정보 (재생성/수정 시 참고)
├── image_log.json        ← 이미지 생성 이력
├── images/               ← 생성/사용된 이미지
├── originals/            ← 사용자 제공 원본 이미지
└── modules/              ← 모듈별 캡처 이미지
```

**복수 플랫폼:**
```
{상품명}/
├── product_info.yaml     ← 공통 상품 정보
├── image_log.json        ← 공통 이미지 이력
├── images/               ← 공통 생성 이미지
├── originals/            ← 공통 원본 이미지
├── 롯데마트몰/
│   ├── detail_page.html
│   ├── full.png
│   └── modules/
├── 쿠팡/
├── 네이버스마트스토어/
└── 11번가/
```

---

## 2. 성능 및 최적화

### 2.1 병렬 빌드 전략

복수 플랫폼 선택 시 **공통 작업(카피라이팅·이미지 생성)**은 단 한 번만 수행하고, 플랫폼별 HTML 빌드 + Playwright 캡처는 Claude Agent SDK의 **Task 도구**를 이용해 단일 메시지에서 N개를 동시 실행한다.

> **예시**: 4개 플랫폼(롯데마트몰+쿠팡+네이버+11번가) 선택 시 — 카피·이미지는 1회 생성, HTML 빌드+캡처는 4개 Task가 동시 실행된다. 순차 처리 대비 이론적으로 **최대 4배 속도 향상**.

| 플랫폼 수 | 처리 방식 | 소요 시간 (예상) | 비고 |
|---|---|---|---|
| 단일 | 순차 처리 | 기준값 (~2~3분) | 기본 케이스 |
| 2개 | 병렬 Task ×2 | ~1.5x 단축 | 공통 작업 공유 |
| 4개 | 병렬 Task ×4 | ~3~4x 단축 | 최대 병렬화 |

### 2.2 이미지 생성 모델 선택 전략

`image-generation.md`에 **승인된 모델 레지스트리**를 명시하여 outdated 모델 사용을 방지한다. 용도별 의사결정 트리에 따라 비용·품질·속도를 최적화한다.

#### 텍스트 → 이미지 (Text-to-Image)

| 모델 ID | 설명 | 가격 | 주 용도 |
|---|---|---|---|
| `fal-ai/nano-banana-pro` | Gemini 3 Pro 기반, 최고 품질 | $0.15/장 | 프리미엄 배너, 히어로 이미지 |
| `fal-ai/nano-banana` | Gemini 2.5 Flash, 고품질 가성비 | $0.039/장 | 라이프스타일 씬, 배경 이미지 |
| `fal-ai/kling-image/v3/text-to-image` | Kling V3, 시네마틱 특화 | $0.028/장 | 시네마틱 배너, 분위기 있는 씬 |
| `fal-ai/kling-image/o3/text-to-image` | Kling O3, 캐릭터 일관성 | $0.028/장 | 모델/캐릭터 시리즈 이미지 |
| `fal-ai/ideogram/v3` | 최고 텍스트 렌더링+포토리얼 | $0.03~$0.09/장 | 텍스트 포함 이미지, 로고 |
| `fal-ai/flux-2-pro` | FLUX 2 Pro, 최신 라인업 | ~$0.03/MP | 배너 배경, 범용 고품질 |
| `fal-ai/flux-2` | FLUX 2 기본, 빠르고 저렴 | ~$0.012/MP | 컷샷 배경, 반복 생성 |
| `fal-ai/flux/schnell` | 초고속 경량 모델 | ~$0.003/MP | 아이콘, 프로토타입 |

#### 이미지 → 이미지 (Image-to-Image) 및 유틸리티

| 모델 ID | 설명 | 가격 | 주 용도 |
|---|---|---|---|
| `fal-ai/kling-image/o3/image-to-image` | Kling O3 I2I, 캐릭터 일관성 최상 | $0.028/장 | 상품/모델 일관성 편집 |
| `fal-ai/nano-banana-pro/edit` | 자연어 지시 고품질 편집 | $0.15/장 | 프리미엄 이미지 편집 |
| `fal-ai/nano-banana/edit` | 자연어 지시 가성비 편집 | $0.039/장 | 일반 편집, 색감 조정 |
| `fal-ai/flux-pro/kontext` | 스타일 일관성 편집 | $0.04/장 | 배경 변경, 스타일 편집 |
| `fal-ai/ideogram/v3/edit` | 고정밀 텍스트 유지 편집 | $0.03~$0.09/장 | 텍스트 유지 편집 |
| `fal-ai/birefnet/v2` | 고정밀 배경 제거 | 무료~컴퓨트 | 상품 누끼 (배경 제거) |
| `fal-ai/aura-sr` | 고속 업스케일 2x/4x | 무료~컴퓨트 | 저해상도 이미지 업스케일 |

#### 모델 선택 의사결정 트리

```
텍스트→이미지 선택
├── 최고 품질 배너/히어로?       → nano-banana-pro ($0.15)
├── 시네마틱/분위기 있는 이미지?
│   ├── 캐릭터 일관성 필요?      → kling-image/o3 ($0.028)
│   └── 프롬프트 기반 시네마틱?  → kling-image/v3 ($0.028)
├── 고품질 가성비?               → nano-banana ($0.039)
├── 텍스트 포함 이미지/로고?     → ideogram/v3 ($0.03 TURBO)
├── 빠른 반복/프로토타입?        → flux-2 (~$0.012/MP)
└── 아이콘/심플 이미지?          → flux/schnell (~$0.003/MP)

이미지→이미지 선택
├── 캐릭터/상품 일관성 최우선?   → kling-image/o3/i2i ($0.028)
├── 자연어 편집 지시?
│   ├── 고품질                   → nano-banana-pro/edit ($0.15)
│   └── 가성비                   → nano-banana/edit ($0.039)
├── 스타일 일관성 편집?          → flux-pro/kontext ($0.04)
└── 배경 교체/inpainting?        → flux-2-pro/edit (~$0.03/MP)
```

### 2.3 비용 최적화 — 3단계 이미지 활용 원칙

원본 이미지 활용을 최우선으로 하여 불필요한 AI 이미지 생성을 최소화한다.

| 우선순위 | 방법 | 비용 | 설명 |
|---|---|---|---|
| **1순위** | 원본 이미지 직접 사용 | $0 | 제공된 이미지를 HTML에 그대로 배치 |
| **2순위** | 원본 기반 image-to-image 편집 | 낮음 | 원본을 참고하여 새 씬 생성 또는 배경 교체 |
| **3순위** | text-to-image 신규 생성 | 높음 | 아이콘·인포그래픽 등 원본으로 커버 불가한 경우만 |

### 2.4 상세페이지 1건당 예상 비용 (가성비 모드)

일반적인 상세페이지(모듈 A~F) 기준:

| 작업 | 모델 | 수량 | 예상 비용 |
|---|---|---|---|
| 배너 이미지 생성 | `fal-ai/nano-banana` | 1장 | $0.039 |
| 라이프스타일 씬 | `fal-ai/kling-image/v3` | 2장 | $0.056 |
| 컷샷 배경 | `fal-ai/flux-2` | 2장 | ~$0.024 |
| 아이콘 생성 | `fal-ai/flux/schnell` | 4장 | ~$0.012 |
| 배경 제거 (누끼) | `fal-ai/birefnet/v2` | 3장 | ~$0.00 |
| 업스케일 | `fal-ai/aura-sr` | 2장 | ~$0.00 |
| **합계** | | **~14장** | **~$0.13 (약 180원)** |

> **프리미엄 모드**: `nano-banana-pro`, `kontext/max` 등 고품질 모델 사용 시 약 $0.30~$0.50 수준. 플랫폼 요구 품질에 따라 모델을 선택하면 비용을 탄력적으로 관리할 수 있다.

#### 이미지 후처리 파이프라인 (Node.js Sharp)

```javascript
const sharp = require('sharp');

// 리사이즈 + JPEG 변환 (일반 사진)
await sharp('input.png')
  .resize({ width: 860 })
  .jpeg({ quality: 90 })
  .toFile('output.jpg');

// 리사이즈 + PNG 유지 (투명 배경)
await sharp('input.png')
  .resize({ width: 860 })
  .png()
  .toFile('output.png');
```

파일명 규칙: `{모듈타입}_{순서}_{설명}.{ext}` (예: `banner_main.png`, `cutshot_01_detail.jpg`)

---

## 3. 사용자 경험 (UX)

### 3.1 대화형 정보 수집 플로우

사용자가 `/상세페이지 [상품명]`을 입력하면, 플러그인은 **AskUserQuestion** 도구로 구조화된 질문을 순서대로 제시한다. 이미지가 제공된 경우 자동으로 상품 정보를 추출하여 질문을 최소화한다.

| 단계 | 항목 | 내용 |
|---|---|---|
| 1 | 이미지 확인 | 작업 폴더 스캔 또는 URL 감지 → `originals/` 폴더 보관 → fal.ai CDN 업로드 |
| 2 | 기본 정보 | 상품명(인자로 전달 시 생략), 카테고리 선택(식품/가전/생활용품/패션/건강) |
| 3 | 플랫폼 선택 | 다중 선택 가능(`multiSelect: true`). 선택한 플랫폼 수만큼 병렬 빌드 |
| 4 | 특장점 입력 | 3~5개 핵심 특장점. 이미지에서 자동 추출 가능 시 생략 |
| 5 | 생성 실행 | 카피 작성 → 이미지 생성 → HTML 빌드 → 캡처 → 결과 링크 제공 |

#### 카피라이팅 가이드라인 (카테고리별)

| 카테고리 | 톤 | 대표 키워드 | 색상 톤 |
|---|---|---|---|
| 식품 | 신선, 건강, 자연, 정직 | 엄선된, 100%, 무첨가 | 그린, 오렌지, 내추럴 브라운 |
| 가전/전자 | 혁신, 편리, 스마트 | 자동, 초절전, 올인원 | 다크 그레이, 블루, 실버 |
| 생활용품 | 따뜻, 실용, 안심 | 든든한, 간편한, 알뜰한 | 화이트, 민트, 라이트 블루 |
| 패션/뷰티 | 트렌디, 세련, 감각 | 데일리, 러블리, 글로우 | 핑크, 베이지, 블랙, 골드 |
| 건강/헬스 | 신뢰, 전문, 과학적 | 임상 검증, 특허 성분 | 화이트, 그린, 딥블루 |

### 3.2 이미지 입력 방식 및 제약

| 입력 방식 | 방법 | 처리 방식 | 지원 |
|---|---|---|---|
| **① URL 직접 제공 (최우선)** | `https://...` URL을 채팅에 입력 | fal.ai 모델에 `image_url`로 직접 전달. 별도 업로드 불필요. | ✅ |
| **② 작업폴더 파일** | 선택한 폴더에 이미지 파일 저장 | `originals/`로 복사 → fal.ai CDN 업로드 → URL 확보. | ✅ |
| **③ 채팅 첨부 (불가)** | 채팅창에 드래그·붙여넣기 | Cowork 보안 아키텍처상 파일 시스템 저장 불가. ①②로 안내. | ❌ |

> **개발 참고사항**: 채팅 첨부 이미지가 파일 시스템에 저장되지 않는 것은 Cowork 보안 아키텍처의 특성이다. 향후 버전에서 임시 저장 경로 지원 여부를 검토할 수 있다.

채팅 첨부 이미지 입력 시 사용자 안내 메시지:

```
채팅에 첨부된 이미지는 직접 사용할 수 없습니다. 아래 방법 중 하나를 이용해 주세요:
- 이미지 URL 붙여넣기 — 상품 이미지의 웹 주소(https://...)를 채팅에 입력해 주세요
- 작업 폴더에 파일 저장 — 이미지 파일을 선택하신 작업 폴더에 넣어주시면 바로 사용합니다
```

### 3.3 편집 워크플로우 (/수정하기)

생성된 HTML 파일은 `data-module-type` 속성 기반으로 파싱되어 모듈 단위 CRUD가 가능하다. 편집 전 `_backup` 접미사로 원본을 자동 백업한다.

**지원되는 편집 작업:**

- **텍스트 수정**: `copywriting-guide` 참조하여 카피 품질 유지하며 텍스트 변경
- **이미지 교체**: 새 이미지 경로 교체 또는 `image_log.json`의 기존 프롬프트 참조하여 재생성
- **모듈 순서 변경**: `data-module-order` 재배치. A(1st) / F(last) 규칙 강제
- **새 모듈 추가**: `modules.md` 템플릿 기반 HTML 생성 후 적절한 위치에 삽입
- **모듈 삭제**: 해당 `<section>` 제거 후 `data-module-order` 재번호
- **전체 스타일 조정**: TailwindCSS 클래스 일괄 변경 (색상·폰트·여백 등)

**상태 관리**: 편집 후 `product_info.yaml`의 수정일이 자동 갱신되고, 이미지 변경 시 `image_log.json`에 이력이 **누적 보존**된다(덮어쓰기 아님). 이력 기반으로 이전 프롬프트 참조 및 재생성이 가능하다.

#### 상태 파일 구조

**product_info.yaml** (상품 정보 저장):
```yaml
상품명: 유기농 꿀
생성일: "2026-02-24"
수정일: "2026-02-24"

기본정보:
  카테고리: 식품
  대상플랫폼: [롯데마트몰, 쿠팡]
  정가: 15,000원
  판매가: 12,000원

핵심특장점:
  - 국내산 100%
  - HACCP 인증

카피:
  롯데마트몰:
    A:
      헤드라인: 자연이 담긴 순수한 단맛
      서브카피: 국내산 아카시아 100%, 첨가물 없는 꿀
```

**image_log.json** (이미지 이력 관리):
```json
{
  "generated_images": [
    {
      "filename": "banner.png",
      "model": "fal-ai/nano-banana",
      "prompt": "Professional food photography of organic honey jar...",
      "source_url": "https://fal.media/files/...",
      "created_at": "2026-02-24T12:00:00Z"
    }
  ],
  "uploaded_originals": [
    {
      "original_filename": "상품사진.jpg",
      "saved_as": "originals/original_01.jpg",
      "uploaded_url": "https://fal.media/files/..."
    }
  ]
}
```

### 3.4 미리보기 & Playwright 캡처

`/미리보기` 커맨드는 Node.js Playwright를 통해 HTML을 Chromium으로 렌더링하고 스크린샷을 캡처한다.

| 캡처 종류 | 파일명 | 방법 |
|---|---|---|
| 전체 페이지 | `full.png` | `page.screenshot({ fullPage: true })` |
| 모듈별 | `modules/module_{N}_{type}.png` | `data-module-type` 별 `mod.screenshot()` |

```javascript
// Playwright 캡처 스크립트 핵심 로직
const browser = await chromium.launch();
const page = await browser.newPage({ viewport: { width: 860, height: 900 } });

await page.goto(`file://${htmlPath}`);
await page.waitForLoadState('networkidle');

// 전체 페이지 캡처
await page.screenshot({ path: 'full.png', fullPage: true });

// 모듈별 캡처
const modules = await page.$$('[data-module-type]');
for (const mod of modules) {
  const type = await mod.getAttribute('data-module-type');
  const order = await mod.getAttribute('data-module-order');
  await mod.screenshot({ path: `modules/module_${order}_${type}.png` });
}
```

Playwright + Chromium 미설치 시 자동으로 의존성을 해결한다:
```bash
npm install playwright && npx playwright install chromium
```

---

## 4. 비즈니스 임팩트

### 4.1 비용 절감 효과

기존 외주 제작 프로세스와 PDP 플러그인 비교:

| 항목 | 기존 외주 방식 | PDP 플러그인 |
|---|---|---|
| **제작 비용** | 30만~100만원/건 | ~$0.13~$0.50 (약 200~720원) |
| **제작 시간** | 3~7 영업일 | 10~30분 (자동화) |
| **수정 대응** | 추가 비용 발생 | 즉시 `/수정하기`로 재생성 |
| **플랫폼 대응** | 플랫폼별 별도 작업 | 단일 요청으로 4개 플랫폼 동시 |
| **카피라이팅** | 별도 전문가 필요 | 카테고리별 AI 자동 생성 |
| **이미지 생성** | 제품 촬영 비용 별도 | AI image-to-image로 확장 |

> **비용 절감 요약**: 상세페이지 1건당 이미지 생성 비용이 외주 대비 **99% 이상 절감**된다. 월 100건 운영 기준 연간 수천만 원 수준의 절감 효과가 가능하다.

### 4.2 생산성 향상

- **전문가 없이 운영 가능**: 카피라이팅·디자인·이미지 편집 전문 지식 없이 상품 정보만 입력하면 완성
- **즉각적인 수정 대응**: `/수정하기`로 특정 모듈만 선택적 수정. 전체 재작업 불필요
- **멀티 플랫폼 일괄 생성**: 4개 이커머스 플랫폼 규격에 맞는 상세페이지를 단 한 번의 명령으로 동시 생성
- **이력 관리 자동화**: `product_info.yaml` + `image_log.json` 자동 생성으로 재작업 기준점 확보
- **원본 이미지 자산 보존**: `originals/` 폴더에 원본 복사본 보관. 언제든 재활용 가능
- **반복 작업 제거**: 계절별·행사별 이미지 변경 시 `image_log.json` 프롬프트 재사용으로 일관성 유지

### 4.3 확장성 및 개선 로드맵

#### 단기 (v0.2.x)

- 채팅 첨부 이미지 지원 — Cowork 임시 경로 연동
- 커스텀 플랫폼 추가 UI — `platform-configs.md` YAML 기반 온보딩
- 카피 톤 커스터마이징 — 브랜드별 보이스 프로필

#### 중기 (v0.3.x)

- A/B 변형 자동 생성 — 동일 상품 다양한 레이아웃 비교
- GIF 모듈 지원 — 네이버 스마트스토어 GIF 규격 활용
- 상세페이지 성과 연동 — 클릭률·전환율 피드백 루프

#### 장기 (v1.0.x)

- 비디오 모듈 지원 — 상품 시연 영상 자동 편집
- 브랜드 가이드라인 학습 — 브랜드 아이덴티티 일관성 강화
- 마켓플레이스 직접 업로드 — API 연동으로 원클릭 배포

---

## 마무리

PDP 플러그인은 이커머스 운영의 가장 반복적이고 비용 집약적인 작업인 '상세페이지 제작'을 AI 자동화로 대체한다.

기술적으로는 HTML/TailwindCSS 렌더링, fal.ai 멀티 모델 오케스트레이션, Playwright 기반 캡처, Claude Agent SDK 병렬 처리가 유기적으로 결합된 구조다. 비즈니스적으로는 **제작 비용 99% 이상 절감**과 **즉각적인 수정 대응**이 핵심 가치다.

v0.2.x부터는 채팅 이미지 지원과 커스텀 플랫폼 온보딩으로 더 넓은 사용 시나리오를 커버할 예정이다.

---

*PDP Plugin v0.1.0 · 작성자: Insoo · 2026-02-24 · 내부 개발팀 전용*
