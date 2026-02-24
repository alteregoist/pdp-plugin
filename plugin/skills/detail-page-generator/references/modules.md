# 상세페이지 모듈 유형 정의

각 모듈의 역할, 구성 요소, HTML/TailwindCSS 템플릿을 정의한다.
모든 모듈은 `<section>` 태그로 감싸며 `data-module-type`과 `data-module-order` 속성을 필수로 포함한다.

---

## Module A: 상단배너

**역할**: 상품의 첫인상. 메인 비주얼 이미지와 핵심 메시지를 전달.

**구성 요소**:
- 메인 비주얼 이미지 (전체 너비)
- 상품명 (헤드라인)
- 핵심 한줄 카피
- 가격 정보 (선택)

**HTML 템플릿**:
```html
<section data-module-type="A" data-module-order="1" class="relative w-full overflow-hidden">
  <div class="relative">
    <img src="./images/banner_main.png" alt="{상품명} 메인 배너" class="w-full h-auto object-cover" />
    <div class="absolute inset-0 flex flex-col justify-end p-8 bg-gradient-to-t from-black/40 to-transparent">
      <p class="text-white/80 text-sm font-medium tracking-wider uppercase mb-2">{카테고리}</p>
      <h1 class="text-white text-3xl md:text-4xl font-bold mb-3 leading-tight">{상품명}</h1>
      <p class="text-white/90 text-lg mb-4">{핵심 한줄 카피}</p>
      <!-- 가격 정보 (선택) -->
      <div class="flex items-baseline gap-2">
        <span class="text-white text-2xl font-bold">{판매가}</span>
        <span class="text-white/60 text-base line-through">{정가}</span>
      </div>
    </div>
  </div>
</section>
```

**디자인 가이드**:
- 배너 이미지 비율: 16:9 또는 4:3 (플랫폼 규격에 맞춤)
- 텍스트 오버레이 시 가독성 확보를 위한 그라데이션 배경 사용
- 식품: 신선한 느낌의 밝은 톤 / 가전: 세련된 다크 톤 / 생활용품: 따뜻한 내추럴 톤

---

## Module B: 한눈에 보기

**역할**: 상품의 핵심 특장점을 아이콘+텍스트로 빠르게 전달.

**구성 요소**:
- 섹션 타이틀 ("한눈에 보기" 또는 "Why This Product")
- 3~5개 특장점 카드 (아이콘 + 키워드 + 짧은 설명)

**HTML 템플릿**:
```html
<section data-module-type="B" data-module-order="2" class="py-12 px-6 bg-gray-50">
  <div class="max-w-[860px] mx-auto">
    <h2 class="text-center text-2xl font-bold text-gray-900 mb-2">한눈에 보기</h2>
    <p class="text-center text-gray-500 text-sm mb-8">{서브 타이틀}</p>
    <div class="grid grid-cols-2 md:grid-cols-{N} gap-6">
      <!-- 특장점 카드 (반복) -->
      <div class="flex flex-col items-center text-center p-4">
        <div class="w-16 h-16 rounded-full bg-white shadow-md flex items-center justify-center mb-4">
          <img src="./images/icon_{N}.png" alt="{특장점}" class="w-8 h-8" />
        </div>
        <h3 class="font-semibold text-gray-900 text-base mb-1">{특장점 키워드}</h3>
        <p class="text-gray-500 text-sm leading-relaxed">{짧은 설명}</p>
      </div>
      <!-- /특장점 카드 -->
    </div>
  </div>
</section>
```

**디자인 가이드**:
- 아이콘은 라인 스타일 또는 미니멀 일러스트
- 특장점 3개: `grid-cols-3`, 4개: `grid-cols-2 md:grid-cols-4`, 5개: `grid-cols-2 md:grid-cols-5`
- 배경색: 밝은 그레이(`bg-gray-50`) 또는 브랜드 컬러의 연한 톤

---

## Module C-1: 상품 컷샷 (이미지+텍스트 혼합형)

**역할**: 상품의 디테일을 이미지와 설명 텍스트로 교대 배치하여 스토리텔링.

**구성 요소**:
- 상품 이미지 (크롭/확대 컷)
- 설명 텍스트 블록 (헤드라인 + 본문)
- 좌우 교대 레이아웃

**HTML 템플릿**:
```html
<section data-module-type="C-1" data-module-order="3" class="py-12 px-6">
  <div class="max-w-[860px] mx-auto space-y-16">
    <!-- 컷샷 블록 (이미지 왼쪽) -->
    <div class="flex flex-col md:flex-row items-center gap-8">
      <div class="w-full md:w-1/2">
        <img src="./images/cutshot_01.png" alt="{설명}" class="w-full rounded-lg shadow-sm" />
      </div>
      <div class="w-full md:w-1/2">
        <span class="text-sm font-semibold text-blue-600 uppercase tracking-wider">{키워드 태그}</span>
        <h3 class="text-xl font-bold text-gray-900 mt-2 mb-3">{헤드라인}</h3>
        <p class="text-gray-600 leading-relaxed">{본문 설명}</p>
      </div>
    </div>
    <!-- 컷샷 블록 (이미지 오른쪽) -->
    <div class="flex flex-col md:flex-row-reverse items-center gap-8">
      <div class="w-full md:w-1/2">
        <img src="./images/cutshot_02.png" alt="{설명}" class="w-full rounded-lg shadow-sm" />
      </div>
      <div class="w-full md:w-1/2">
        <span class="text-sm font-semibold text-blue-600 uppercase tracking-wider">{키워드 태그}</span>
        <h3 class="text-xl font-bold text-gray-900 mt-2 mb-3">{헤드라인}</h3>
        <p class="text-gray-600 leading-relaxed">{본문 설명}</p>
      </div>
    </div>
  </div>
</section>
```

**디자인 가이드**:
- 이미지와 텍스트 좌우 교대 배치 (지그재그 패턴)
- 이미지 비율: 1:1 또는 4:3
- 키워드 태그 색상: 브랜드 컬러 또는 카테고리 컬러

---

## Module C-2: 상품 정보 (텍스트 단독형)

**역할**: 상세 스펙, 사용법, 성분 정보 등 텍스트 중심 정보를 구조화하여 전달.

**구성 요소**:
- 섹션 타이틀
- 정보 테이블 또는 리스트

**HTML 템플릿**:
```html
<section data-module-type="C-2" data-module-order="4" class="py-12 px-6 bg-white">
  <div class="max-w-[860px] mx-auto">
    <h2 class="text-xl font-bold text-gray-900 mb-6 pb-3 border-b-2 border-gray-900">{섹션 타이틀}</h2>
    <div class="space-y-0">
      <!-- 정보 행 (반복) -->
      <div class="flex border-b border-gray-100 py-4">
        <dt class="w-1/3 text-sm font-medium text-gray-500 shrink-0">{항목명}</dt>
        <dd class="text-sm text-gray-900">{항목값}</dd>
      </div>
      <!-- /정보 행 -->
    </div>
  </div>
</section>
```

**변형 - 카드형**:
```html
<section data-module-type="C-2" data-module-order="4" class="py-12 px-6 bg-gray-50">
  <div class="max-w-[860px] mx-auto">
    <h2 class="text-xl font-bold text-gray-900 mb-6">{섹션 타이틀}</h2>
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
      <!-- 정보 카드 (반복) -->
      <div class="bg-white rounded-lg p-5 shadow-sm">
        <h4 class="font-semibold text-gray-900 mb-2">{항목명}</h4>
        <p class="text-sm text-gray-600 leading-relaxed">{항목 설명}</p>
      </div>
      <!-- /정보 카드 -->
    </div>
  </div>
</section>
```

---

## Module D: 인증 정보

**역할**: 상품의 신뢰성을 높이는 인증마크, 수상 이력, 품질 보증 정보.

**구성 요소**:
- 인증마크 이미지들
- 인증 명칭 및 설명
- 인증 번호 (선택)

**HTML 템플릿**:
```html
<section data-module-type="D" data-module-order="5" class="py-10 px-6 bg-blue-50/50">
  <div class="max-w-[860px] mx-auto">
    <h2 class="text-lg font-bold text-gray-900 mb-6 text-center">인증 정보</h2>
    <div class="flex flex-wrap justify-center gap-8">
      <!-- 인증 아이템 (반복) -->
      <div class="flex flex-col items-center text-center w-28">
        <div class="w-20 h-20 rounded-full bg-white shadow-sm flex items-center justify-center mb-3 border border-gray-100">
          <img src="./images/cert_{name}.png" alt="{인증명}" class="w-12 h-12 object-contain" />
        </div>
        <span class="text-xs font-medium text-gray-700">{인증명}</span>
        <span class="text-xs text-gray-400 mt-1">{인증번호}</span>
      </div>
      <!-- /인증 아이템 -->
    </div>
  </div>
</section>
```

---

## Module E: 상품 인포그래픽

**역할**: 수치 데이터, 비교 정보, 프로세스를 시각적으로 전달.

**구성 요소**:
- 인포그래픽 이미지 (전체 너비) 또는 HTML 기반 차트/그래프
- 핵심 수치 하이라이트
- 비교 테이블 (선택)

**HTML 템플릿 - 수치 하이라이트형**:
```html
<section data-module-type="E" data-module-order="6" class="py-12 px-6 bg-gradient-to-b from-gray-900 to-gray-800">
  <div class="max-w-[860px] mx-auto">
    <h2 class="text-xl font-bold text-white mb-2 text-center">{인포그래픽 타이틀}</h2>
    <p class="text-gray-400 text-sm text-center mb-10">{서브 타이틀}</p>
    <div class="grid grid-cols-2 md:grid-cols-4 gap-6">
      <!-- 수치 카드 (반복) -->
      <div class="text-center">
        <div class="text-3xl font-bold text-white mb-1">{수치}<span class="text-lg text-blue-400">{단위}</span></div>
        <p class="text-gray-400 text-sm">{설명}</p>
      </div>
      <!-- /수치 카드 -->
    </div>
  </div>
</section>
```

**HTML 템플릿 - 이미지형**:
```html
<section data-module-type="E" data-module-order="6" class="w-full">
  <img src="./images/infographic.png" alt="{인포그래픽 설명}" class="w-full h-auto" />
</section>
```

**HTML 템플릿 - 비교형**:
```html
<section data-module-type="E" data-module-order="6" class="py-12 px-6">
  <div class="max-w-[860px] mx-auto">
    <h2 class="text-xl font-bold text-gray-900 mb-8 text-center">{비교 타이틀}</h2>
    <div class="grid grid-cols-2 gap-4">
      <div class="bg-gray-50 rounded-xl p-6 text-center">
        <p class="text-sm text-gray-500 mb-3">일반 제품</p>
        <img src="./images/compare_before.png" alt="일반 제품" class="w-32 h-32 mx-auto mb-4 object-contain" />
        <p class="text-gray-600 text-sm">{일반 제품 설명}</p>
      </div>
      <div class="bg-blue-50 rounded-xl p-6 text-center border-2 border-blue-200">
        <p class="text-sm font-semibold text-blue-600 mb-3">우리 제품</p>
        <img src="./images/compare_after.png" alt="우리 제품" class="w-32 h-32 mx-auto mb-4 object-contain" />
        <p class="text-gray-900 text-sm font-medium">{우리 제품 설명}</p>
      </div>
    </div>
  </div>
</section>
```

---

## Module F: 표시사항

**역할**: 법적 고지, 주의사항, 배송/교환/반품 정보 등 필수 표시 정보.

**구성 요소**:
- 법적 고지 텍스트
- 주의사항
- 배송/교환/반품 안내
- 소비자 상담 정보

**HTML 템플릿**:
```html
<section data-module-type="F" data-module-order="7" class="py-8 px-6 bg-gray-50 border-t border-gray-200">
  <div class="max-w-[860px] mx-auto">
    <h2 class="text-base font-bold text-gray-700 mb-4">표시사항</h2>
    <div class="space-y-6 text-xs text-gray-500 leading-relaxed">
      <!-- 표시 항목 그룹 (반복) -->
      <div>
        <h3 class="text-sm font-semibold text-gray-600 mb-2">{그룹 타이틀}</h3>
        <div class="space-y-1">
          <div class="flex">
            <span class="w-24 shrink-0 text-gray-400">{항목}</span>
            <span>{내용}</span>
          </div>
        </div>
      </div>
      <!-- /표시 항목 그룹 -->

      <!-- 주의사항 -->
      <div class="bg-yellow-50/50 rounded-lg p-4 border border-yellow-100">
        <h3 class="text-sm font-semibold text-yellow-700 mb-2">주의사항</h3>
        <ul class="space-y-1 list-disc list-inside text-yellow-600/80">
          <li>{주의사항 항목}</li>
        </ul>
      </div>
    </div>
  </div>
</section>
```

---

## 상품이미지 모듈 (보조)

**역할**: 상품 이미지만 단독으로 풀 너비 노출. 모듈 사이사이에 삽입 가능.

**HTML 템플릿**:
```html
<section data-module-type="IMG" data-module-order="{N}" class="w-full">
  <img src="./images/{filename}" alt="{상품명}" class="w-full h-auto" />
</section>
```

---

## 커스텀 모듈 생성 가이드

새로운 모듈을 추가할 때 다음 규칙을 따른다:

1. `data-module-type`에 고유 타입 코드 부여 (예: "G", "H", "CUSTOM-1")
2. `data-module-order`에 삽입 위치 번호 부여
3. TailwindCSS 유틸리티 클래스만 사용
4. `max-w-[860px] mx-auto` 컨테이너 기본 적용 (전체 너비 이미지 제외)
5. 반응형 대응: `md:` 브레이크포인트 활용
6. 이미지는 `./images/` 하위에 저장
