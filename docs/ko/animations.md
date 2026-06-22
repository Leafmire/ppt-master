# 슬라이드 전환과 개체 애니메이션

[한국어 문서 목차](./README.md) · [오디오 내레이션](./audio-narration.md) · [원본 레퍼런스](../../skills/ppt-master/references/animations.md)

PPT Master의 네이티브 PPTX는 슬라이드 간 전환과 슬라이드 내부 개체의 등장 애니메이션을 지원합니다. 둘 다 실제 OOXML로 기록되므로 PowerPoint에서 편집할 수 있습니다.

## 기본값

| 계층 | 기본값 | 설명 |
|---|---|---|
| 슬라이드 전환 | `fade`, 0.4초 | 대부분의 덱에 무난한 전환 |
| 개체 애니메이션 | `none` | 과도한 자동 효과를 피하기 위해 기본 비활성 |

개체 애니메이션을 켜려면 `-a auto`를 사용합니다.

```bash
python3 skills/ppt-master/scripts/svg_to_pptx.py <project> -a auto
```

SVG와 콘텐츠를 다시 만들 필요 없이 같은 `svg_output/` 또는 `svg_final/`에서 내보내기 옵션만 바꿔 재생성할 수 있습니다.

## 슬라이드 전환

### 효과 변경

```bash
python3 skills/ppt-master/scripts/svg_to_pptx.py <project> \
  -t push --transition-duration 0.6
```

### 전환 끄기

```bash
python3 skills/ppt-master/scripts/svg_to_pptx.py <project> -t none
```

### 자동 넘김

```bash
python3 skills/ppt-master/scripts/svg_to_pptx.py <project> \
  --auto-advance 5
```

지원 전환:

- `fade`
- `push`
- `wipe`
- `split`
- `strips`
- `cover`
- `random`
- `none`

주요 옵션:

| 옵션 | 설명 |
|---|---|
| `-t`, `--transition` | 전환 효과 또는 `none` |
| `--transition-duration` | 전환 시간(초), 기본 0.4 |
| `--auto-advance` | 지정 시간 후 자동으로 다음 슬라이드로 이동 |

## 개체 애니메이션

개체 애니메이션은 슬라이드의 의미 단위 그룹을 순서대로 보여 줍니다.

```bash
# 자동 효과 선택 + 순차 등장
python3 skills/ppt-master/scripts/svg_to_pptx.py <project> -a auto

# 모든 그룹에 fade 사용
python3 skills/ppt-master/scripts/svg_to_pptx.py <project> --animation fade

# 발표자가 클릭할 때마다 다음 그룹 표시
python3 skills/ppt-master/scripts/svg_to_pptx.py <project> \
  -a auto --animation-trigger on-click

# 속도 조정
python3 skills/ppt-master/scripts/svg_to_pptx.py <project> \
  --animation mixed \
  --animation-duration 0.5 \
  --animation-stagger 0.7
```

## 시작 방식

PowerPoint 애니메이션 창의 “시작”과 대응합니다.

### `on-click`

슬라이드에 들어온 뒤 클릭할 때마다 다음 그룹을 보여 줍니다. 발표자가 공개 순서를 직접 제어하는 라이브 발표에 적합합니다.

`--recorded-narration`과 함께 사용할 수 없습니다. 녹화·동영상 출력은 개체 클릭 타이밍을 자동으로 만들지 않기 때문입니다.

### `with-previous`

모든 그룹이 슬라이드 진입 시 함께 시작합니다. `--animation-stagger`는 무시됩니다.

### `after-previous`

기본값입니다. 첫 그룹은 슬라이드 진입 시 시작하고 다음 그룹은 앞 효과가 끝난 뒤 순차적으로 실행됩니다. 키오스크, 자동 재생, 내레이션 동영상에 적합합니다.

## 지원 개체 효과

단일 효과:

```text
appear, fade, fly, cut, zoom, wipe, split, blinds,
checkerboard, dissolve, random_bars, peek, wheel, box,
circle, diamond, plus, strips, wedge, stretch, expand,
swivel
```

자동 변화 모드:

- `auto`: SVG 그룹 ID의 의미를 바탕으로 효과 선택
- `mixed`: 호환성을 위해 유지되는 결정적 효과 순환
- `random`: 효과 풀에서 무작위 선택
- `none`: 개체 애니메이션 비활성

## `auto` 효과 규칙

대표적인 ID 매핑:

| 그룹 ID 유형 | 대표 효과 |
|---|---|
| `chart`, `table`, `legend`, `timeline`, `track` | `wipe` |
| `card-*`, `step-*`, `item-*`, `pillar-*`, `stage-*` | `fly` |
| `title`, `chapter-*`, `section-*`, `subtitle` | `fade` |
| `takeaway`, `callout`, `quote`, `source`, `conclusion` | `fade` |
| `hero`, `figure-*`, `image`, `img-*`, `kpi` | zoom/dissolve/circle/box/diamond/wheel 순환 |
| 그 밖의 그룹 | fade/wipe/fly/zoom 순환 |

`appear`는 움직임이 보이지 않기 때문에 자동 효과 풀에서 제외됩니다.

## SVG 그룹 앵커

개체 애니메이션은 SVG 루트의 최상위 `<g id="...">`를 의미 단위로 사용합니다.

```svg
<svg ...>
  <g id="background">...</g>
  <g id="title">...</g>
  <g id="chart">...</g>
  <g id="takeaway">...</g>
</svg>
```

권장 그룹 수는 슬라이드당 3~8개입니다. 이 정도는 애니메이션뿐 아니라 PowerPoint에서 그룹 선택·이동하기에도 편리합니다.

### 애니메이션에서 제외되는 크롬 그룹

다음 의미의 ID는 페이지 배경·고정 장식으로 간주해 등장 순서에서 제외됩니다.

```text
background, bg, decoration, decor, header, footer,
chrome, watermark, pagenumber, pagenum, nav, logo, rule
```

예:

```svg
<g id="bg-texture">...</g>
<g id="cover-footer">...</g>
<g id="logo-area">...</g>
```

그룹 자체를 제거하지 말고 의미 있는 ID를 지정하세요. 그룹은 PowerPoint 편집성에도 도움이 됩니다.

### 평면 SVG의 대체 처리

최상위 `<g>`가 없고 루트에 `<rect>`, `<text>`, `<path>`만 있는 경우:

- 보이는 최상위 요소가 8개 이하: 각 요소를 앵커로 사용
- 8개 초과: 지나치게 세분된 애니메이션을 피하기 위해 해당 슬라이드 개체 애니메이션 생략

슬라이드 렌더링 자체는 계속 수행됩니다.

## `animations.json`으로 세부 제어

특정 그룹의 효과, 순서, 지연, 시간을 조정하려면 프로젝트 루트에 `animations.json`을 둡니다.

### 기본 파일 생성

```bash
python3 skills/ppt-master/scripts/animation_config.py scaffold <project>
```

### 검증

```bash
python3 skills/ppt-master/scripts/animation_config.py validate <project>
```

### 예시

```json
{
  "version": 1,
  "slides": {
    "03_market": {
      "groups": {
        "title": {
          "effect": "fade",
          "order": 1
        },
        "chart": {
          "effect": "wipe",
          "order": 2,
          "duration": 0.6
        },
        "insight": {
          "effect": "fly",
          "order": 3,
          "delay": 0.2
        },
        "footer": {
          "effect": "none"
        }
      }
    }
  }
}
```

규칙:

- `slides` 키는 SVG 파일명에서 확장자를 뺀 값
- `groups` 키는 최상위 `<g id>`와 일치
- `effect: none`은 해당 그룹을 등장 순서에서 제외
- `order`는 애니메이션 순서만 바꾸며 레이어 순서는 바꾸지 않음
- `delay`는 `after-previous`에서 시작 전 대기 시간
- `duration`은 해당 그룹의 효과 시간을 덮어씀
- CLI의 `--animation none`은 sidecar보다 우선해 모두 끔

프로젝트에 `animations.json`이 있으면 내보내기가 기본적으로 읽습니다. 다른 경로를 쓰려면 `--animation-config`를 지정합니다.

## 내레이션 덱 권장 설정

```bash
python3 skills/ppt-master/scripts/svg_to_pptx.py <project> \
  -a auto \
  --animation-trigger after-previous \
  --recorded-narration audio
```

주의:

- `on-click`은 녹화 내레이션과 함께 사용 불가
- 슬라이드 자동 전환 시간 안에 모든 개체 애니메이션이 끝나야 함
- 효과가 많으면 말보다 시각 변화가 더 눈에 띌 수 있음
- 내레이션 덱은 2~5개의 핵심 그룹만 순차 공개하는 것이 안전

## 디자인 권장 사항

### 애니메이션은 정보 구조를 드러내는 데 사용

좋은 예:

- 프로세스 단계를 순서대로 공개
- 차트 후 결론 문구 공개
- 비교 항목을 좌우 순서로 공개

피해야 할 예:

- 모든 장식 요소를 따로 움직이기
- 한 슬라이드에 서로 다른 효과를 과도하게 섞기
- 발표 목적과 관계없는 회전·튀기기 효과
- 너무 긴 지연으로 발표 흐름 끊기

### 한국어 텍스트 그룹

한국어 제목과 본문이 별도 그룹이면 효과 순서를 더 명확히 제어할 수 있습니다.

```svg
<g id="title">
  <text font-family="Malgun Gothic">시장 전망</text>
</g>
<g id="chart">...</g>
<g id="takeaway">
  <text font-family="Malgun Gothic">핵심 결론</text>
</g>
```

언어·글꼴 처리는 텍스트 런에서 수행되며, 애니메이션 그룹 ID와는 독립적입니다.

## 한계

- 네이티브 도형 모드에서만 개체 단위 애니메이션 가능
- 슬라이드 전체가 이미지인 레거시 출력은 전환만 지원
- Office 버전에 따라 일부 효과가 단순 `Appear`로 대체될 수 있음
- PNG 호환 렌더링은 시각 대체용이며 애니메이션은 슬라이드 XML에 존재
- PowerPoint와 Keynote의 효과 해석이 완전히 같지 않을 수 있음

## 빠른 명령표

| 목적 | 옵션 |
|---|---|
| 전환 끄기 | `-t none` |
| 전환 변경 | `-t push` |
| 느린 전환 | `--transition-duration 0.8` |
| 자동 넘김 | `--auto-advance 5` |
| 개체 애니메이션 켜기 | `-a auto` |
| 개체 애니메이션 끄기 | `-a none` |
| 클릭 공개 | `--animation-trigger on-click` |
| 함께 시작 | `--animation-trigger with-previous` |
| 순차 시작 | `--animation-trigger after-previous` |
| 효과 시간 | `--animation-duration 0.5` |
| 그룹 간격 | `--animation-stagger 0.7` |
