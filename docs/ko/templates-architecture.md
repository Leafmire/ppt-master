# 템플릿 아키텍처: Brand / Layout / Deck

> 이 문서는 템플릿의 데이터 모델, `design_spec.md` 필드, 여러 템플릿을 함께 사용할 때의 병합 규칙을 설명합니다. 일반 사용자용 절차는 [템플릿 사용 가이드](./templates-guide.md)를 먼저 읽으세요.

[한국어 문서 목차](./README.md) · [기술 설계](./technical-design.md)

## 1. 세 가지 템플릿 종류

| 종류 | 물리 경로 | 담당 범위 | 담당하지 않는 범위 |
|---|---|---|---|
| **Brand** | `templates/brands/<id>/` | 색상, 타이포그래피, 로고, 문체, 아이콘 스타일 | 캔버스, 페이지 구조, SVG 목록 |
| **Layout** | `templates/layouts/<id>/` | 캔버스, 페이지 구조, 페이지 유형, SVG 목록 | 브랜드 로고, 공식 브랜드 색상, 브랜드 문체 |
| **Deck** | `templates/decks/<id>/` | Identity + Structure + 전체 덱 개요 | 없음 |

세 종류는 서로 상속하는 클래스가 아니라 병렬 참조 번들입니다. 디렉터리 종류와 frontmatter의 `kind` 값은 일치해야 합니다.

```yaml
---
brand_id: example_brand
kind: brand
summary: 한국어 B2B 보고서용 브랜드
primary_color: "#2457A7"
---
```

```yaml
---
layout_id: executive_report
kind: layout
summary: 경영진 보고용 표지/요약/비교/결론 구조
canvas_format: ppt169
page_count: 5
page_types: [cover, summary, comparison, content, ending]
---
```

```yaml
---
deck_id: annual_report
kind: deck
summary: 전체 시각 체계가 검증된 연차보고서 덱
canvas_format: ppt169
page_count: 8
primary_color: "#2457A7"
---
```

## 2. 세그먼트 분리

병합 충돌을 예측 가능하게 만들기 위해 필드를 세그먼트 단위로 나눕니다.

| 세그먼트 | 포함 내용 | 우선 소유자 |
|---|---|---|
| **Identity** | Color Scheme, Typography, Logo, Voice & Tone, Icon Style | Brand |
| **Structure** | Canvas Specification, Page Structure, Page Types, SVG Roster | Layout |
| **Middle** | Template Overview, 사용 사례, 디자인 의도, 페이지 리듬 | Deck |

기본 병합 단위는 **세그먼트 전체 교체**입니다. 예를 들어 Deck와 Brand를 함께 지정하면 Deck의 일부 색상만 남기고 Brand의 주색만 섞는 것이 아니라, Identity 전체를 Brand에서 가져옵니다.

## 3. Brand 스키마

Frontmatter 필수 항목:

```yaml
---
brand_id: <slug>
kind: brand
summary: <사용 사례와 대표 색상을 포함한 한 줄 설명>
primary_color: "<HEX>"
---
```

본문 권장 섹션:

1. Brand Overview — 브랜드명, 사용 사례, 분위기
2. Color Scheme — 역할, HEX, 출처(`fact`/`approx`), 설명
3. Typography — 역할, 글꼴, 굵기
4. Logo — 파일, 형태, 여백, 조합 규칙
5. Voice & Tone — 격식, 인칭, 이모지, 약어 정책
6. Icon Style — 선/면/듀오톤 등과 권장 라이브러리

Brand에 넣지 말아야 할 항목:

- `viewBox`와 캔버스 크기
- 표지·목차·본문 같은 페이지 유형
- SVG 파일 목록

### 한국어 타이포그래피 예시

```markdown
## III. Typography

| 역할 | 글꼴 | 굵기 | 대체 글꼴 |
|---|---|---:|---|
| 한국어 제목 | Malgun Gothic | 700 | Noto Sans KR |
| 한국어 본문 | Malgun Gothic | 400 | Dotum |
| 한국어 세리프 강조 | Batang | 400 | Noto Serif KR |
| 영문/숫자 | Segoe UI | 400–700 | Arial |
```

실제 PPTX 변환에서는 이 포크가 한글 런을 `ko-KR`로 표시하고 한국어용 글꼴 슬롯을 선택합니다.

## 4. Layout 스키마

Frontmatter 필수 항목:

```yaml
---
layout_id: <slug>
kind: layout
summary: <한 줄 사용 사례>
canvas_format: <ppt169 | ppt43 | a4 | ...>
page_count: <N>
page_types: [cover, toc, chapter, content, ending]
---
```

본문 권장 섹션:

1. Template Overview — 사용 사례, 디자인 의도, 리듬
2. Canvas Specification — 형식, 치수, `viewBox`, 여백, 콘텐츠 영역
3. Page Structure — 그리드, 장식 규칙, 내비게이션
4. Page Types — 각 페이지 역할과 변형
5. SVG Page Roster — 파일과 역할의 대응

Layout은 구조만 정의합니다. 브랜드 로고, 공식 브랜드 색상, 브랜드 문체를 넣지 않습니다. 색상과 글꼴은 실행 시 사용자 확인 또는 Brand에서 결정합니다.

## 5. Deck 스키마

Deck는 기존 PPT의 정체성과 구조가 함께 검증된 완전한 참조 번들입니다.

Frontmatter:

```yaml
---
deck_id: <slug>
kind: deck
summary: <한 줄 사용 사례>
canvas_format: <ppt169 | ...>
page_count: <N>
primary_color: "<HEX>"
---
```

본문에는 다음을 모두 포함합니다.

1. Template Overview
2. Canvas Specification
3. Color Scheme
4. Typography
5. Logo
6. Voice & Tone
7. Icon Style
8. Page Structure
9. Page Types
10. SVG Page Roster

Deck는 기본적으로 완전한 해결책이지만, 사용자가 Brand나 Layout을 명시하면 해당 세그먼트가 교체될 수 있습니다.

## 6. 인덱스 파일

각 디렉터리는 선택에 필요한 최소 메타데이터를 인덱스로 제공합니다.

### Brand

`templates/brands/brands_index.json`

```json
{
  "example_brand": {
    "summary": "한국어 엔터프라이즈 보고서용 블루 브랜드",
    "primary_color": "#2457A7"
  }
}
```

### Layout

`templates/layouts/layouts_index.json`

```json
{
  "executive_report": {
    "summary": "경영진 보고용 표지/요약/비교/결론 구조",
    "canvas_format": "ppt169",
    "page_count": 5,
    "page_types": ["cover", "summary", "comparison", "content", "ending"]
  }
}
```

### Deck

`templates/decks/decks_index.json`

```json
{
  "annual_report": {
    "summary": "브랜드와 구조가 결합된 연차보고서 덱",
    "canvas_format": "ppt169",
    "page_count": 8,
    "primary_color": "#2457A7"
  }
}
```

## 7. 다중 경로 병합 규칙

| 사용자 입력 | 병합 결과 |
|---|---|
| 없음 | 자유 디자인 |
| Brand만 | Identity는 Brand, Structure는 자유 설계 |
| Layout만 | Structure는 Layout, Identity는 실행 시 결정 |
| Deck만 | Deck 전체 사용 |
| Brand + Layout | Identity는 Brand, Structure는 Layout |
| Brand + Deck | Identity는 Brand, Structure/Middle은 Deck |
| Layout + Deck | Structure는 Layout, Identity/Middle은 Deck |
| Brand + Layout + Deck | Identity는 Brand, Structure는 Layout, Middle은 Deck |

### 전체 세그먼트 교체 원칙

병합은 기본적으로 필드 단위가 아니라 세그먼트 단위입니다.

잘못된 암묵적 혼합 예:

```text
주색은 Brand, 보조색은 Deck, 로고는 Brand, 아이콘은 Deck
```

권장 결과:

```text
Identity 전체 = Brand
Structure 전체 = Layout 또는 Deck
Middle = Deck가 있을 때 Deck
```

필드 단위 혼합이 꼭 필요하면 실행 전에 사용자가 명시적으로 새로운 통합 `design_spec.md`를 작성해야 합니다.

## 8. 충돌 해결

### 캔버스 충돌

Layout의 캔버스가 Structure를 소유합니다. Deck와 Layout이 함께 지정되면 Layout의 캔버스를 사용하고 Deck의 SVG가 새 캔버스에 맞는지 검토해야 합니다.

### 글꼴 충돌

Brand가 Identity를 소유합니다. 한국어 Brand에는 한국어 글꼴과 Windows 대체 글꼴을 함께 기록하세요.

### 색상 충돌

Brand가 있으면 Brand의 Color Scheme 전체를 사용합니다. 공식 값은 `provenance: fact`, 추정 값은 `provenance: approx`로 구분합니다.

### 로고 충돌

Brand가 제공한 로고 규칙을 우선합니다. Deck의 장식 요소가 새 로고의 안전 여백을 침범하지 않는지 확인해야 합니다.

## 9. 디렉터리 품질 기준

각 템플릿 디렉터리는 다음 조건을 만족해야 합니다.

- frontmatter의 `kind`와 물리 경로가 일치
- ID가 인덱스 키와 일치
- 자신이 소유하지 않는 세그먼트를 포함하지 않음
- SVG 파일 목록과 실제 파일이 일치
- 캔버스·여백·글꼴·색상 값이 명시적
- 예시 문구가 실제 고정 문구처럼 오해되지 않음
- 한국어 글꼴에는 안전한 대체 계열이 정의됨

## 10. 관련 워크플로

- `skills/ppt-master/workflows/create-brand.md`
- `skills/ppt-master/workflows/create-template.md`
- `skills/ppt-master/workflows/template-fill-pptx.md`
- `skills/ppt-master/templates/design_spec_reference.md`

실제 구현과 충돌하면 현재 `SKILL.md`와 워크플로 파일이 우선합니다.
