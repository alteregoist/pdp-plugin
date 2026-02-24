# fal.ai 이미지 생성 전략

상세페이지에 필요한 이미지가 부족할 때 fal.ai MCP를 통해 이미지를 생성하거나 편집한다.

> **MCP 도구 사용법**: fal.ai MCP 도구는 `mcp__fal-ai__generate` (생성 요청), `mcp__fal-ai__result` (결과 조회), `mcp__fal-ai__status` (상태 확인), `mcp__fal-ai__upload` (파일 업로드) 등의 이름 패턴으로 호출한다.

---

## 승인된 모델 레지스트리

> **중요**: 아래 목록에 없는 모델은 사용하지 않는다. outdated 모델이나 과도하게 비싼 모델 사용을 방지하기 위한 사전 정의 목록이다.

### A. 텍스트→이미지 (Text-to-Image)

| 모델 ID | 설명 | 가격 | 용도 |
|---------|------|------|------|
| `fal-ai/nano-banana-pro` | Gemini 3 Pro 기반, 최고 품질 | $0.15/장 | 프리미엄 배너, 히어로 이미지 |
| `fal-ai/nano-banana` | Gemini 2.5 Flash 기반, 고품질 가성비 | $0.039/장 | 라이프스타일 씬, 배경 이미지 |
| `fal-ai/flux-2-pro` | FLUX 2 Pro, 최신 FLUX 라인업 | ~$0.03/MP | 배너 배경, 고품질 범용 |
| `fal-ai/flux-2` | FLUX 2 기본, 빠르고 저렴 | ~$0.012/MP | 컷샷 배경, 반복 생성 |
| `fal-ai/kling-image/v3/text-to-image` | Kling V3, 시네마틱 프롬프트 특화 | $0.028/장 (4K: $0.056) | 시네마틱 배너, 분위기 있는 라이프스타일 |
| `fal-ai/kling-image/o3/text-to-image` | Kling O3, 레퍼런스+캐릭터 일관성 | $0.028/장 (4K: $0.056) | 캐릭터/모델 일관성, 시리즈 이미지 |
| `fal-ai/flux/schnell` | FLUX Schnell, 초고속 경량 | ~$0.003/MP | 아이콘, 프로토타입, 빠른 반복 |
| `fal-ai/ideogram/v3` | Ideogram V3, 최고 텍스트 렌더링+포토리얼 | $0.03~$0.09/장 (TURBO~QUALITY) | 텍스트 포함 이미지, 로고, 배너 타이틀 |

### B. 이미지→이미지 (Image-to-Image)

| 모델 ID | 설명 | 가격 | 용도 |
|---------|------|------|------|
| `fal-ai/nano-banana-pro/edit` | Gemini 3 Pro 편집, 자연어 지시 | $0.15/장 | 프리미엄 이미지 편집 (배경 교체, 요소 추가/제거) |
| `fal-ai/nano-banana/edit` | Gemini 2.5 Flash 편집, 가성비 | $0.039/장 | 일반 이미지 편집, 색감 조정 |
| `fal-ai/kling-image/o3/image-to-image` | Kling O3 I2I, 캐릭터 일관성 최상 | $0.028/장 (4K: $0.056) | 상품/모델 일관성 유지 편집, 스타일 변환 |
| `fal-ai/ideogram/v3/edit` | Ideogram V3 Edit, 고정밀 이미지 편집 | $0.03~$0.09/장 (TURBO~QUALITY) | 텍스트 유지 편집, 고정밀 이미지 수정 |
| `fal-ai/ideogram/v3/remix` | Ideogram V3 Remix, 변형 생성 | $0.03~$0.09/장 (TURBO~QUALITY) | 기존 이미지 기반 변형, 스타일 변환 |
| `fal-ai/ideogram/character` | Ideogram V3 Character, 캐릭터 일관성 | $0.10~$0.20/장 (TURBO~QUALITY) | 캐릭터/인물 일관성 유지 생성 |
| `fal-ai/flux-pro/kontext/max` | Kontext Max, 최고 품질 스타일 편집 | $0.08/장 | 스타일 일관성 최우선 편집 |
| `fal-ai/flux-pro/kontext` | Kontext Pro, 균형 잡힌 편집 | $0.04/장 | 일반 스타일 편집, 배경 변경 |
| `fal-ai/flux-2-pro/edit` | FLUX 2 Pro 편집 | ~$0.03/MP | 배경 교체, inpainting |

### C. 인페인팅 (Inpainting)

| 모델 ID | 설명 | 가격 | 용도 |
|---------|------|------|------|
| `fal-ai/flux-pro/v1/fill` | FLUX Pro Fill, 영역 채우기 | ~$0.05/MP | 이미지 확장, 부분 채우기 |

### D. 배경 제거 (Background Removal)

| 모델 ID | 설명 | 가격 | 용도 |
|---------|------|------|------|
| `fal-ai/birefnet/v2` | BiRefNet V2, 고정밀 배경 제거 | 무료~컴퓨트 | 상품 누끼, 배경 제거 |

### E. 업스케일 (Upscale)

| 모델 ID | 설명 | 가격 | 용도 |
|---------|------|------|------|
| `fal-ai/aura-sr` | Aura SR, 고속 업스케일 | 무료~컴퓨트 | 저해상도 이미지 2x/4x 업스케일 |

---

## 용도별 모델 선택 가이드

### 텍스트→이미지 선택 플로우

```
최고 품질 배너/히어로가 필요?
├─ YES → nano-banana-pro ($0.15)
└─ NO
   ├─ 시네마틱/분위기 있는 이미지?
   │  ├─ 캐릭터/모델 일관성 필요 → kling-image/o3 ($0.028)
   │  └─ 프롬프트 기반 시네마틱 → kling-image/v3 ($0.028)
   ├─ 고품질 가성비?
   │  ├─ 라이프스타일/배경 → nano-banana ($0.039)
   │  └─ 범용 고품질 → flux-2-pro (~$0.03/MP)
   ├─ 텍스트 포함 이미지/로고? → ideogram/v3 ($0.03 TURBO)
   ├─ 빠른 반복/프로토타입? → flux-2 (~$0.012/MP)
   └─ 아이콘/심플 → flux/schnell (~$0.003/MP)
```

### 이미지→이미지 선택 플로우

```
기존 이미지를 편집해야 할 때:
├─ 캐릭터/상품 일관성 유지가 최우선?
│  └─ kling-image/o3/image-to-image ($0.028) ← 최고 일관성
├─ 자연어 편집 지시? ("배경을 흰색으로 바꿔줘")
│  ├─ 고품질 → nano-banana-pro/edit ($0.15)
│  └─ 가성비 → nano-banana/edit ($0.039)
├─ 텍스트 유지 편집 / 리믹스?
│  ├─ 편집 → ideogram/v3/edit ($0.03 TURBO)
│  └─ 변형 생성 → ideogram/v3/remix ($0.03 TURBO)
├─ 캐릭터/인물 일관성? → ideogram/character ($0.10 TURBO)
├─ 스타일 일관성 편집?
│  ├─ 최고 품질 → kontext/max ($0.08)
│  └─ 균형 → kontext ($0.04)
└─ 배경 교체/inpainting → flux-2-pro/edit (~$0.03/MP)
```

### 유틸리티 선택

```
배경 제거 (누끼) → birefnet/v2 (무료~컴퓨트)
업스케일 → aura-sr (무료~컴퓨트)
이미지 확장 (outpainting) → flux-pro/v1/fill (~$0.05/MP)
```

---

## 모델별 상세 사용법

### Kling Image V3 (텍스트→이미지)

시네마틱 분위기와 영화적 조명에 특화된 모델.

```javascript
// fal.ai MCP generate 호출
{
  app_id: "fal-ai/kling-image/v3/text-to-image",
  input_data: {
    prompt: "Professional food photography of organic honey jar, cinematic warm lighting, shallow depth of field, premium commercial quality",
    negative_prompt: "text, watermark, blurry, low quality",
    aspect_ratio: "16:9",  // 지원: 1:1, 16:9, 9:16, 4:3, 3:4, 3:2, 2:3
    num_images: 1
  }
}
```

**특징**: 프롬프트 해석력이 뛰어나고 자연스러운 조명/그림자 표현. 배너 배경이나 분위기 있는 라이프스타일 씬에 적합.

### Kling Image O3 (텍스트→이미지 & 이미지→이미지)

레퍼런스 이미지 기반 캐릭터/오브젝트 일관성에 특화.

```javascript
// 텍스트→이미지
{
  app_id: "fal-ai/kling-image/o3/text-to-image",
  input_data: {
    prompt: "Korean model using skincare product in modern bathroom, natural lighting, clean aesthetic",
    negative_prompt: "text, watermark, blurry",
    aspect_ratio: "3:4",
    num_images: 1
  }
}

// 이미지→이미지 (상품/캐릭터 일관성 편집)
{
  app_id: "fal-ai/kling-image/o3/image-to-image",
  input_data: {
    prompt: "Same product in outdoor garden setting, natural sunlight",
    image_url: "https://...",  // 원본 이미지
    aspect_ratio: "16:9",
    num_images: 1
  }
}
```

**특징**: 동일 상품이나 모델을 여러 씬에서 일관되게 표현할 때 최적. 시리즈 컷샷(C-1 모듈) 제작에 특히 유용.

### nano-banana-pro / nano-banana

```javascript
// 텍스트→이미지
{
  app_id: "fal-ai/nano-banana-pro",  // 또는 "fal-ai/nano-banana"
  input_data: {
    prompt: "...",
    negative_prompt: "...",
    image_size: "landscape_16_9",  // landscape_16_9, square_hd, portrait_4_3 등
    num_images: 1
  }
}

// 이미지→이미지 (자연어 편집)
{
  app_id: "fal-ai/nano-banana-pro/edit",  // 또는 "fal-ai/nano-banana/edit"
  input_data: {
    prompt: "Change the background to a modern kitchen with marble countertop",
    image_url: "https://...",
    num_images: 1
  }
}
```

### Ideogram V3 (텍스트→이미지 & 편집/리믹스)

최고 수준의 텍스트 렌더링과 포토리얼리즘. 로고, 배너 타이틀, 텍스트가 포함된 이미지에 최적.

```javascript
// 텍스트→이미지
{
  app_id: "fal-ai/ideogram/v3",
  input_data: {
    prompt: "Premium organic honey product banner with text 'PURE HONEY' in elegant gold serif font, clean white background, commercial quality",
    negative_prompt: "blurry, low quality",
    rendering_speed: "TURBO",  // TURBO ($0.03) | BALANCED ($0.06) | QUALITY ($0.09)
    aspect_ratio: "16:9",
    num_images: 1
  }
}

// 이미지 편집
{
  app_id: "fal-ai/ideogram/v3/edit",
  input_data: {
    prompt: "Change the text to 'NEW ARRIVAL' while keeping the same style",
    image_url: "https://...",
    rendering_speed: "TURBO",
    num_images: 1
  }
}

// 리믹스 (변형 생성)
{
  app_id: "fal-ai/ideogram/v3/remix",
  input_data: {
    prompt: "Same product in dark luxury theme with gold accents",
    image_url: "https://...",
    rendering_speed: "TURBO",
    num_images: 1
  }
}

// 캐릭터 일관성
{
  app_id: "fal-ai/ideogram/character",
  input_data: {
    prompt: "Same character model presenting a new product in kitchen setting",
    image_url: "https://...",  // 캐릭터 레퍼런스
    rendering_speed: "TURBO",  // $0.10 TURBO | $0.15 BALANCED | $0.20 QUALITY
    num_images: 1
  }
}
```

**특징**: 텍스트 렌더링 정확도가 타 모델 대비 압도적. 배너에 상품명/가격/프로모션 텍스트를 직접 넣어야 할 때 최적. Style References로 최대 3장의 레퍼런스 이미지 기반 일관된 스타일 생성 가능.

### FLUX Kontext (이미지→이미지)

```javascript
{
  app_id: "fal-ai/flux-pro/kontext",  // 또는 "fal-ai/flux-pro/kontext/max"
  input_data: {
    prompt: "Change the product background to gradient blue, keep the product exactly the same",
    image_url: "https://...",
    num_images: 1
  }
}
```

### FLUX 2 Pro / FLUX 2

```javascript
{
  app_id: "fal-ai/flux-2-pro",  // 또는 "fal-ai/flux-2"
  input_data: {
    prompt: "...",
    negative_prompt: "...",
    image_size: "landscape_16_9",
    num_images: 1
  }
}
```

### 배경 제거

```javascript
{
  app_id: "fal-ai/birefnet/v2",
  input_data: {
    image_url: "https://..."
  }
}
```

### 업스케일

```javascript
{
  app_id: "fal-ai/aura-sr",
  input_data: {
    image_url: "https://..."
  }
}
```

---

## 상세페이지 1건당 예상 비용

일반적인 상세페이지(모듈 A~F) 기준:

| 작업 | 모델 | 수량 | 예상 비용 |
|------|------|------|----------|
| 배너 이미지 생성 | nano-banana | 1장 | $0.039 |
| 라이프스타일 씬 | kling-image/v3 | 2장 | $0.056 |
| 컷샷 배경 | flux-2 | 2장 | ~$0.024 |
| 아이콘 생성 | flux/schnell | 4장 | ~$0.012 |
| 배경 제거 | birefnet/v2 | 3장 | ~$0.00 |
| 업스케일 | aura-sr | 2장 | ~$0.00 |
| **합계** | | **14장** | **~$0.13** |

> 프리미엄 모델(nano-banana-pro, kontext/max) 사용 시 $0.30~$0.50 수준.

---

## 사용 금지 (Deprecated) 모델

> **절대 사용하지 않는다.** 아래 모델들은 outdated이거나, 더 좋은 대체 모델이 있다.

| 사용 금지 모델 | 사유 | 대체 모델 |
|---------------|------|----------|
| `fal-ai/flux-pro/v1.1` | FLUX 2 Pro로 대체됨 | `fal-ai/flux-2-pro` |
| `fal-ai/flux-pro` (v1) | FLUX 2 Pro로 대체됨 | `fal-ai/flux-2-pro` |
| `fal-ai/flux/dev` | FLUX 2로 대체됨 | `fal-ai/flux-2` |
| `fal-ai/flux-realism` | Deprecated | `fal-ai/flux-2-pro` |
| `fal-ai/flux/dev/image-to-image` | Kontext로 대체됨 | `fal-ai/flux-pro/kontext` |
| `fal-ai/rembg` | BiRefNet V2로 대체됨 | `fal-ai/birefnet/v2` |
| `fal-ai/creative-upscaler` | Aura SR로 대체됨 | `fal-ai/aura-sr` |
| `fal-ai/birefnet` (v1) | V2로 대체됨 | `fal-ai/birefnet/v2` |
| `fal-ai/ideogram/v2` | V3로 대체됨 | `fal-ai/ideogram/v3` |
| `fal-ai/stable-diffusion-*` | 전체 deprecated | FLUX 또는 nano-banana 사용 |
| `fal-ai/midjourney-*` | 비공식/불안정 | FLUX 또는 nano-banana 사용 |

---

## 카테고리별 프롬프트 전략

### 공통 프롬프트 규칙

1. **영어로 작성** (fal.ai 모델은 영어 프롬프트에 최적화)
2. **구조**: `[주제], [스타일], [조명], [구도], [배경], [분위기]`
3. **네거티브 프롬프트 기본값**: `"text, watermark, logo, blurry, low quality, distorted, deformed"`
4. **이미지 사이즈**: 플랫폼 규격에 맞춤 (기본 860px 너비 기준)

### 식품

**배너 프롬프트 템플릿**:
```
Professional food photography of {product}, fresh and appetizing,
studio lighting with soft shadows, top-down or 45-degree angle,
clean white/light wood background with subtle props (herbs, spices, cutting board),
warm and inviting mood, high-end commercial quality, 8k resolution
```

**라이프스타일 프롬프트 템플릿**:
```
Lifestyle photograph of a person enjoying {product} in a {setting: modern kitchen/dining table/outdoor cafe},
natural daylight, warm tones, Korean food styling,
appetizing and fresh appearance, editorial quality photography
```

**컬러 팔레트**: warm neutrals, green accents, natural wood tones

### 가전/전자

**배너 프롬프트 템플릿**:
```
Sleek product photography of {product}, minimalist studio setup,
dramatic lighting with gradient background (dark gray to black),
slight reflection on surface, premium and modern aesthetic,
clean lines and sharp details, commercial advertising quality
```

**라이프스타일 프롬프트 템플릿**:
```
Modern {product} in a contemporary Korean apartment interior,
minimalist Scandinavian design setting, natural light from large windows,
clean and organized space, lifestyle editorial photography style
```

**컬러 팔레트**: dark grays, silver, subtle blue accents

### 생활용품

**배너 프롬프트 템플릿**:
```
Clean and bright product photography of {product},
soft natural lighting, minimal white background with subtle shadows,
organized and tidy arrangement, warm and approachable mood,
Korean household product styling
```

**라이프스타일 프롬프트 템플릿**:
```
{product} in use in a clean modern Korean home,
bright and airy interior, natural daylight,
practical and organized setting, lifestyle photography,
warm and comfortable atmosphere
```

**컬러 팔레트**: whites, light blues, mint, soft pastels

### 패션/뷰티

**배너 프롬프트 템플릿**:
```
High-fashion product photography of {product},
editorial lighting with soft bokeh background,
elegant and sophisticated styling,
luxury brand aesthetic, premium quality image
```

**컬러 팔레트**: beige, blush pink, black, gold accents

### 건강/헬스

**배너 프롬프트 템플릿**:
```
Professional product photography of {product},
clean medical/scientific aesthetic, soft white lighting,
trustworthy and professional mood,
natural ingredients scattered around product,
high-end supplement/health product styling
```

**컬러 팔레트**: clean whites, greens, deep blues

---

## 아이콘 생성

Module B(한눈에 보기)의 아이콘 생성:

**프롬프트 템플릿**:
```
Simple minimalist line icon of {concept},
single color ({brand_color}), white background,
clean vector style, flat design,
centered composition, no text,
icon design for e-commerce
```

**파라미터**:
- 사이즈: 256x256 (후에 120x120으로 리사이즈)
- 모델: `fal-ai/flux/schnell`
- Steps: 4 (schnell 기본)

---

## 이미지 후처리

fal.ai로 생성한 이미지는 다음 후처리를 거친다:

1. **사이즈 조정**: 플랫폼 규격에 맞게 리사이즈
2. **포맷 변환**: PNG (투명 배경 필요 시) 또는 JPEG (일반 사진, quality 90)
3. **파일명 규칙**: `{모듈타입}_{순서}_{설명}.{ext}` (예: `banner_main.png`, `cutshot_01_detail.jpg`)
4. **저장 경로**: `./images/` 디렉토리

후처리는 Node.js sharp 패키지로 수행한다:

```bash
npm install sharp
```

```javascript
const sharp = require('sharp');

// 리사이즈 + JPEG 변환
await sharp('input.png')
  .resize({ width: 860 })
  .jpeg({ quality: 90 })
  .toFile('output.jpg');

// 리사이즈 + PNG 유지 (투명 배경)
await sharp('input.png')
  .resize({ width: 860 })
  .png()
  .toFile('output.png');

// 다수 이미지 일괄 처리
const fs = require('fs');
const path = require('path');

async function processImages(inputDir, outputDir, targetWidth) {
  const files = fs.readdirSync(inputDir).filter(f => /\.(png|jpg|jpeg)$/i.test(f));
  for (const file of files) {
    await sharp(path.join(inputDir, file))
      .resize({ width: targetWidth })
      .toFile(path.join(outputDir, file));
  }
}
```

---

## 이미지 생성 워크플로우

**핵심 원칙: 제공된 이미지 최대 활용 → 부족분만 AI 생성**

### Step 1. 이미지 인벤토리 작성
수집된 이미지 소스를 확인한다:
- `image_log.json`의 `source_urls` — 사용자가 제공한 이미지 URL 목록
- `originals/` — 작업 폴더에서 수집한 이미지 파일 (fal.ai CDN URL 포함)
- 각 이미지가 어떤 모듈에 적합한지 매핑한다 (배너용, 컷샷용, 인증마크용 등)

### Step 2. 모듈별 필요 이미지 목록 작성
- 상세페이지에 필요한 모든 이미지를 모듈 단위로 열거한다

### Step 3. 제공된 이미지 직접 배치 (1순위)
- URL 이미지: fal.ai 모델의 `image_url` 파라미터로 직접 전달하거나, HTML `<img src="">` 에 URL을 연결한다
- 파일 이미지: `images/` 폴더로 복사하고 HTML에 상대경로를 연결한다

### Step 4. 제공된 이미지 기반 편집 (2순위)
- 이미지가 있지만 다른 씬/구도가 필요할 때, 수집된 URL 또는 CDN URL을 `image_url`로 넘겨 image-to-image로 변형한다
- 배경 교체, 씬 변형, 색감 조정 등
- 모델 선택은 위의 "이미지→이미지 선택 플로우" 참고

### Step 5. text-to-image 신규 생성 (3순위, 꼭 필요한 경우에만)
- 위 두 방법으로 채울 수 없는 이미지(아이콘, 인포그래픽 배경 등)만 text-to-image로 생성한다
- 카테고리/톤에 맞는 모델 & 프롬프트 선택 (위 선택 가이드 참조)
- fal.ai MCP로 이미지 생성 후 검토, 필요시 프롬프트 조정 후 재생성

### Step 6. 후처리 및 저장
- 후처리 (리사이즈, 포맷 변환)
- `./images/` 디렉토리에 저장
- `image_log.json`에 생성/편집 기록 추가
