---
name: detail-page-generator
description: >
  This skill should be used when the user asks to "create a product detail page",
  "generate a detail page", "make a product page", "상세페이지 만들어줘",
  "상세페이지 생성", "product landing page", or needs to build an e-commerce
  product detail page from product images and information. Also triggers when
  the user wants to edit or modify an existing detail page's modules.
version: 0.1.0
---

# Product Detail Page Generator

상품 이미지와 정보를 입력받아 모듈 기반 상세페이지를 자동 생성하는 스킬.

## Core Workflow

### 1. 상품 정보 수집

사용자로부터 다음 정보를 수집한다:

- **상품명** (필수)
- **상품 카테고리** (식품, 생활용품, 가전, 패션 등)
- **상품 이미지** (업로드된 이미지 파일들)
- **핵심 특장점** (3~5개)
- **상세 스펙/성분 정보**
- **인증 정보** (유기농, HACCP, KC 등)
- **가격 정보**
- **대상 플랫폼** (기본: 롯데마트몰)

정보가 부족하면 AskUserQuestion으로 추가 확인한다.

**이미지 수집 (상품 정보 수집 전, 가장 먼저 수행)**:

아래 우선순위 순서로 이미지를 수집한다.

**① 이미지 URL 직접 제공 (최우선)**
사용자가 `https://...` 이미지 URL을 제공한 경우:
- `image_log.json`의 `source_urls`에 URL 목록을 기록한다
- fal.ai 모델에 `image_url`로 직접 전달 가능하므로 별도 업로드 불필요
- 여러 URL이면 모두 수집하고 모듈별 용도(배너용, 컷샷용 등)를 파악한다

**② 작업 폴더의 이미지 파일**
사용자가 선택한 작업 폴더에서 이미지 파일을 검색한다:
1. `find {작업폴더} -type f \( -name "*.jpg" -o -name "*.jpeg" -o -name "*.png" -o -name "*.webp" \)`
2. 파일이 있으면 `originals/` 디렉토리로 복사한다
3. `mcp__fal-ai__upload`로 fal.ai CDN에 업로드하고 URL을 `uploaded_originals`에 기록한다

**③ 채팅 첨부 이미지 — 사용 불가**
Cowork 채팅에 붙여넣기/드래그한 이미지는 파일 시스템에 저장되지 않아 직접 활용이 불가능하다.
이 경우 사용자에게 안내하고 ①②로 유도한다:

> "채팅에 첨부된 이미지는 직접 사용할 수 없습니다. 아래 방법 중 하나를 이용해 주세요:
> - **이미지 URL 붙여넣기** — 상품 이미지의 웹 주소(https://...)를 채팅에 입력해 주세요
> - **작업 폴더에 파일 저장** — 이미지 파일을 선택하신 작업 폴더에 넣어주시면 바로 사용합니다"

이렇게 보관해두면 이후 수정/재생성 시 원본 이미지를 다시 제공하지 않아도 된다.

### 2. 플랫폼 설정 로드

`references/platform-configs.md`를 읽어 **선택된 모든 플랫폼**의 모듈 구성과 사이즈 규격을 확인한다. 플랫폼별로 사용하는 모듈 조합과 이미지 사이즈가 다르다.

- **단일 플랫폼**: 해당 플랫폼 설정만 로드
- **복수 플랫폼**: 선택된 모든 플랫폼의 설정을 각각 로드하고 리스트로 관리

### 3. 모듈 구성 결정

각 플랫폼에 맞는 모듈 파일을 읽어 모듈 템플릿과 디자인 가이드를 확인한다.

**모듈 파일 로드 순서** (플랫폼마다 수행):
1. `references/modules/common.md` — 공통 기본 템플릿 로드
2. `references/modules/{플랫폼명}.md` — 플랫폼 전용 오버라이드 로드
   - 롯데마트몰 → `references/modules/롯데마트몰.md`
   - 쿠팡 → `references/modules/쿠팡.md`
   - 네이버 스마트스토어 → `references/modules/네이버스마트스토어.md`
   - 11번가 → `references/modules/11번가.md`
3. 플랫폼 파일에 정의된 모듈은 해당 템플릿 사용, 미정의 모듈은 common.md를 플랫폼 최대 너비에 맞게 적용

상품 카테고리와 수집된 정보에 따라 **플랫폼별 최적 모듈 조합**을 각각 결정한다.

복수 플랫폼인 경우, 각 플랫폼의 필수/선택 모듈과 순서 규칙이 다르므로 플랫폼마다 독립적으로 모듈 구성을 수립한다.

기본 모듈 유형:
- **A** — 상단배너: 상품 메인 비주얼
- **B** — 한눈에 보기: 핵심 특장점 요약
- **C-1** — 상품 컷샷: 이미지+텍스트 혼합형
- **C-2** — 상품 정보: 텍스트 단독형
- **D** — 인증 정보: 인증마크 및 설명
- **E** — 상품 인포그래픽: 데이터 시각화
- **F** — 표시사항: 법적 고지 및 상세 정보

### 4. 카피라이팅

`references/copywriting-guide.md`를 읽고 각 모듈에 들어갈 카피를 작성한다.

핵심 원칙:
- 헤드라인: 짧고 임팩트 있게 (15자 이내)
- 서브카피: 구체적 혜택 중심 (30자 이내)
- 본문: 쉬운 말, 짧은 문장
- 카테고리별 톤 매칭 (식품→신선/건강, 가전→혁신/편리, 패션→트렌디/세련)

**카피 완성 후 `product_info.yaml`에 즉시 저장한다.** 저장 형식은 아래 `product_info.yaml` 템플릿의 `카피` 섹션을 참고한다. 복수 플랫폼인 경우 플랫폼별로 카피가 다를 수 있으므로 플랫폼명을 키로 구분하여 저장한다. 이 카피 데이터는 Step 6 병렬 Task에 그대로 전달된다.

### 5. 이미지 생성/편집

`references/image-generation.md`를 읽고, 아래 3단계 우선순위 원칙에 따라 모듈별 이미지를 준비한다.

**이미지 활용 우선순위 (반드시 이 순서로 접근한다)**:

1. **원본 이미지 직접 사용**: `originals/`에 보관된 첨부 이미지를 모듈에 그대로 활용한다. 가능한 한 원본을 우선 배치한다.
2. **원본 이미지 기반 편집 (image-to-image)**: 원본만으로 모든 모듈을 채울 수 없을 때, 첨부 이미지의 CDN URL(`uploaded_url`)을 참고 이미지로 활용하여 image-to-image로 변형하거나 새 씬을 만든다.
3. **text-to-image 신규 생성**: 위 두 방법으로도 커버되지 않는 이미지(아이콘, 인포그래픽 배경 등)만 text-to-image로 새로 생성한다.

> 원본 이미지를 최대한 활용하고, text-to-image는 꼭 필요한 경우에만 사용한다.

**반드시 `references/image-generation.md`의 "승인된 모델 레지스트리"에 등록된 모델만 사용한다.**

**텍스트→이미지 (새 이미지 생성):**
- 최고 품질 배너 → `fal-ai/nano-banana-pro` ($0.15/장)
- 가성비 고품질 → `fal-ai/nano-banana` ($0.039/장)
- 시네마틱/분위기 → `fal-ai/kling-image/v3/text-to-image` ($0.028/장)
- 캐릭터/모델 일관성 → `fal-ai/kling-image/o3/text-to-image` ($0.028/장)
- 범용 빠른 생성 → `fal-ai/flux-2-pro` (~$0.03/MP) / `fal-ai/flux-2` (~$0.012/MP)
- 아이콘/프로토타입 → `fal-ai/flux/schnell` (~$0.003/MP)
- 텍스트 포함 이미지/로고 → `fal-ai/ideogram/v3` ($0.03~$0.09/장)

**이미지→이미지 (기존 이미지 편집):**
- 캐릭터/상품 일관성 편집 → `fal-ai/kling-image/o3/image-to-image` ($0.028/장)
- 자연어 기반 편집 → `fal-ai/nano-banana-pro/edit` ($0.15/장) / `fal-ai/nano-banana/edit` ($0.039/장)
- 텍스트 유지 편집/리믹스 → `fal-ai/ideogram/v3/edit` ($0.03/장) / `fal-ai/ideogram/v3/remix` ($0.03/장)
- 캐릭터/인물 일관성 → `fal-ai/ideogram/character` ($0.10~$0.20/장)
- 배경 교체 → `fal-ai/flux-2-pro/edit` (~$0.03/MP)
- 스타일 일관성 편집 → `fal-ai/flux-pro/kontext` ($0.04/장) / `fal-ai/flux-pro/kontext/max` ($0.08/장)

**유틸리티:**
- 배경 제거 → `fal-ai/birefnet/v2` (무료~컴퓨트)
- 업스케일 → `fal-ai/aura-sr` (무료~컴퓨트)
- 이미지 확장 → `fal-ai/flux-pro/v1/fill` (~$0.05/MP)

**프롬프트 작성 규칙:**
- 상품 카테고리와 톤에 맞는 스타일 키워드 사용
- 일관된 색상 팔레트 유지
- 한국 이커머스 스타일 반영 (깔끔, 밝은 톤, 신뢰감)

### 6. HTML 빌드 + 캡처 (플랫폼별 병렬 실행)

공통 카피·이미지(Step 4~5)가 완료된 뒤, **복수 플랫폼인 경우 Task 도구로 병렬 처리한다.**

#### 단일 플랫폼

순차적으로 HTML 빌드 → Playwright 캡처를 수행한다. (아래 HTML 빌드 규칙 및 캡처 스크립트 참고)

#### 복수 플랫폼 — 병렬 처리

**단계 6-P: Task 도구로 플랫폼별 병렬 실행**

선택된 플랫폼 수만큼 `Task` 도구를 **단일 메시지에 동시에** 호출한다. 각 Task에는 아래 정보를 프롬프트로 전달한다:

```
[플랫폼명] 상세페이지 HTML 빌드 및 Playwright 캡처

## 공통 데이터
- 상품명: {상품명}
- 카테고리: {카테고리}
- 출력 폴더: {절대경로}/{상품명}/{플랫폼명}/
- 이미지 폴더(공통): {절대경로}/{상품명}/images/
- 원본 폴더(공통): {절대경로}/{상품명}/originals/

## 모듈 구성
{해당 플랫폼의 모듈 순서 및 조합}

## 카피 데이터
{Step 4에서 생성한 모듈별 카피 전문}

## 이미지 매핑
{모듈별 사용 이미지 파일명 또는 URL}

## 작업 지시
1. {플랫폼명}.md 모듈 템플릿 + common.md 참조하여 HTML 조립
2. 출력 폴더({상품명}/{플랫폼명}/) 생성
3. detail_page.html 저장
4. Playwright로 full.png 및 modules/ 캡처
5. 완료 후 생성된 파일 목록 반환

## 플랫폼 규격
- 최대 너비: {MAX_WIDTH}px
- viewport: {MAX_WIDTH} x 900
```

모든 Task가 완료되면 결과를 취합하여 사용자에게 보고한다.

#### HTML 빌드 규칙 (각 Task 공통)

- TailwindCSS Play CDN 사용 (`<script src="https://cdn.tailwindcss.com"></script>`)
- 한글 웹폰트: 페이퍼로지(Paperlogy)
  - `https://cdn.jsdelivr.net/gh/fonts-archive/Paperlogy/subsets/Paperlogy-dynamic-subset.css`
- 모듈별 `<section>` 태그에 `data-module-type`, `data-module-order` 속성 필수
- 이미지는 공통 폴더 상대경로 사용 (`../images/filename.png`)
- 최대 너비: 플랫폼 규격에 맞춤

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/fonts-archive/Paperlogy/subsets/Paperlogy-dynamic-subset.css" />
  <script>
    tailwind.config = {
      theme: {
        extend: {
          fontFamily: {
            sans: ['Paperlogy', '-apple-system', 'BlinkMacSystemFont', 'Segoe UI', 'Roboto', 'sans-serif'],
          },
        },
      },
    }
  </script>
  <title>{상품명} 상세페이지 — {플랫폼명}</title>
</head>
<body class="bg-white font-sans">
  <div class="max-w-[{MAX_WIDTH}px] mx-auto">
    <section data-module-type="A" data-module-order="1">...</section>
    <section data-module-type="B" data-module-order="2">...</section>
    <!-- ... -->
  </div>
</body>
</html>
```

**폰트 굵기 활용 가이드**:
- 헤드라인(h1, h2): `font-bold` (700) 또는 `font-extrabold` (800)
- 서브 헤드라인(h3): `font-semibold` (600)
- 본문: `font-normal` (400)
- 캡션/보조 텍스트: `font-light` (300)

#### Playwright 캡처 스크립트 (각 Task 공통)

```bash
npm install playwright 2>/dev/null; npx playwright install chromium 2>/dev/null
```

```javascript
const { chromium } = require('playwright');
const path = require('path');
const fs = require('fs');

const htmlPath = process.argv[2];   // 절대경로
const outputDir = process.argv[3];  // modules/ 폴더 절대경로
const viewportWidth = parseInt(process.argv[4]) || 860;

(async () => {
  fs.mkdirSync(outputDir, { recursive: true });
  const browser = await chromium.launch();
  const page = await browser.newPage({ viewport: { width: viewportWidth, height: 900 } });

  await page.goto(`file://${htmlPath}`);
  await page.waitForLoadState('networkidle');

  // 전체 페이지 캡처
  await page.screenshot({
    path: path.join(path.dirname(outputDir), 'full.png'),
    fullPage: true
  });

  // 모듈별 캡처
  const modules = await page.$$('[data-module-type]');
  for (const mod of modules) {
    const type = await mod.getAttribute('data-module-type');
    const order = await mod.getAttribute('data-module-order');
    await mod.screenshot({ path: path.join(outputDir, `module_${order}_${type}.png`) });
  }

  await browser.close();
  console.log('캡처 완료:', outputDir);
})();
```

실행:
```bash
node capture.js /절대경로/{상품명}/{플랫폼명}/detail_page.html \
               /절대경로/{상품명}/{플랫폼명}/modules/ \
               {MAX_WIDTH}
```

### 8. 최종 출력

**출력 폴더**: `{상품명}/` 폴더를 새로 생성하고, 모든 결과물을 그 안에 정리한다.

**단일 플랫폼** 출력 구조:
```
{상품명}/
├── detail_page.html              # 전체 HTML
├── full.png                      # 전체 페이지 이미지
├── product_info.yaml             # 상품 정보 (재생성/수정 시 참고용)
├── image_log.json                # 이미지 생성 로그
├── images/                       # 생성/사용된 이미지
├── originals/                    # 사용자가 제공한 원본 이미지 복사본
└── modules/                      # 개별 모듈 이미지
```

**복수 플랫폼** 출력 구조 (플랫폼별 하위 폴더로 분리):
```
{상품명}/
├── product_info.yaml             # 상품 정보 (공통, 재생성/수정 시 참고용)
├── image_log.json                # 이미지 생성 로그 (공통)
├── images/                       # 공통 생성 이미지
├── originals/                    # 사용자가 제공한 원본 이미지 복사본 (공통)
├── 롯데마트몰/                    # 플랫폼별 결과물 폴더
│   ├── detail_page.html
│   ├── full.png
│   └── modules/
├── 쿠팡/
│   ├── detail_page.html
│   ├── full.png
│   └── modules/
├── 네이버스마트스토어/
│   ├── detail_page.html
│   ├── full.png
│   └── modules/
└── 11번가/
    ├── detail_page.html
    ├── full.png
    └── modules/
```

> 이미지(originals, images)는 공통 폴더에 한 번만 보관하고, 각 플랫폼 HTML에서 상대경로(`../images/`, `../originals/`)로 참조한다.

#### product_info.yaml

상품 정보를 YAML 형식으로 저장한다. 이후 수정/재생성 시 이 파일을 읽어 기존 정보를 참고한다. 사람이 읽고 직접 편집하기 편하도록 YAML을 사용한다.

```yaml
상품명: 유기농 꿀
생성일: "2026-02-20"
수정일: "2026-02-20"

기본정보:
  카테고리: 식품
  대상플랫폼:
    - 롯데마트몰
    - 쿠팡
  정가: 15,000원
  판매가: 12,000원

핵심특장점:
  - 특장점 1
  - 특장점 2
  - 특장점 3

상세스펙:
  항목1: 값1
  항목2: 값2

인증정보:
  - HACCP
  - 유기농

# 단일 플랫폼인 경우: 카피를 직접 기록
# 복수 플랫폼인 경우: 플랫폼명 키로 구분하여 기록
카피:
  롯데마트몰:
    A:
      헤드라인: 자연이 담긴 순수한 단맛
      서브카피: 국내산 아카시아 100%, 첨가물 없는 꿀
    B:
      타이틀: 이 상품의 특장점
      항목:
        - 키워드: 국내산 100%
          설명: 국내 청정 지역에서 직접 채취
        - 키워드: HACCP 인증
          설명: 위생 안전 기준 완벽 충족
    C-1:
      - 헤드라인: 자연 그대로의 풍미
        본문: 열처리 없이 저온 숙성하여 효소와 영양소를 그대로 보존했습니다.
      - 헤드라인: 믿을 수 있는 원산지
        본문: 강원도 산림에서 채취한 아카시아꿀만을 사용합니다.
    F:
      주의사항:
        - 직사광선을 피해 서늘한 곳에 보관하세요
        - 1세 미만 영아에게는 먹이지 마세요
  쿠팡:
    A:
      헤드라인: 자연이 담긴 순수한 단맛
      서브카피: 국내산 아카시아 100%
    B:
      항목:
        - 키워드: 국내산 100%
        - 키워드: HACCP 인증
    C-1:
      - 헤드라인: 자연 그대로의 풍미
        본문: 열처리 없이 저온 숙성

비고: |
  추가 메모나 참고 사항을 여기에 기록한다.
```

#### image_log.json

이미지 생성/편집 시 사용한 모델, 프롬프트, 원본 URL, 결과 파일명을 기록한다. 이후 이미지를 재생성하거나 프롬프트를 조정할 때 참고한다.

```json
{
  "generated_images": [
    {
      "filename": "banner.png",
      "model": "fal-ai/nano-banana",
      "prompt": "Professional food photography of organic honey jar, cinematic warm lighting...",
      "negative_prompt": "text, watermark, blurry, low quality",
      "parameters": {
        "image_size": "landscape_16_9",
        "num_images": 1
      },
      "source_url": "https://fal.media/files/...",
      "created_at": "2026-02-20T12:01:00Z"
    },
    {
      "filename": "cutshot_01.png",
      "model": "fal-ai/kling-image/o3/image-to-image",
      "prompt": "Same product in outdoor garden setting, natural sunlight",
      "negative_prompt": "text, watermark, blurry",
      "parameters": {
        "aspect_ratio": "16:9",
        "num_images": 1
      },
      "original_image_url": "https://fal.media/files/...(편집 전 원본)",
      "source_url": "https://fal.media/files/...(생성 결과)",
      "created_at": "2026-02-20T12:02:00Z"
    }
  ],
  "uploaded_originals": [
    {
      "original_filename": "사용자제공_상품사진.jpg",
      "saved_as": "originals/original_01.jpg",
      "uploaded_url": "https://fal.media/files/...(CDN 업로드 URL)"
    }
  ]
}
```

- HTML 내 이미지 경로는 `./images/filename.png` 상대경로를 유지한다 (상품명 폴더 기준).
- 상품명에 공백이나 특수문자가 포함된 경우, 폴더명은 공백을 `_`로 치환하여 생성한다.
- 사용자가 제공한 원본 이미지는 `originals/` 폴더에 복사하여 보관한다.
- 이미지 생성/편집 시마다 `image_log.json`에 기록을 추가한다. 수정하기(`/수정하기`) 시에도 동일하게 로그를 갱신한다.
- `product_info.yaml`은 상품 정보 수집 완료 후 즉시 저장하고, 카피라이팅 완료 후 `카피` 섹션을 추가 저장한다. 수정 시 `수정일`을 갱신한다.

## Module Editing Workflow

기존 상세페이지 편집 시:

1. HTML 파일을 읽어 `data-module-type` 속성으로 모듈 파싱
2. 사용자 요청에 따라:
   - **모듈 텍스트 수정**: 해당 모듈의 카피 변경
   - **모듈 이미지 교체**: 새 이미지로 교체 또는 fal.ai로 재생성
   - **모듈 순서 변경**: `data-module-order` 재배치
   - **새 모듈 추가**: `references/modules.md`의 템플릿으로 신규 모듈 삽입
   - **모듈 삭제**: 해당 `<section>` 제거
3. 수정된 HTML 저장 및 Playwright로 재캡처

## Reference Files

- **`references/modules.md`** — 모듈 유형별 상세 정의 및 HTML 템플릿
- **`references/platform-configs.md`** — 플랫폼별 모듈 구성 및 사이즈 규격
- **`references/copywriting-guide.md`** — 카테고리별 카피라이팅 가이드라인
- **`references/image-generation.md`** — fal.ai 모델 선택 전략 및 프롬프트 템플릿
