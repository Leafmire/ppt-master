# 기술 설계

[한국어 문서 목차](./README.md) · [템플릿 아키텍처](./templates-architecture.md) · [원문](../technical-design.md)

## 설계 철학

생성된 PPTX는 인간의 최종 판단을 대체하는 완성품이 아니라, 빈 화면에서 시작하는 작업의 대부분을 줄여 주는 고품질 디자인 초안입니다. AI가 자료 구조화, 시각 설계, 배치, SVG 생성을 담당하고 사용자는 PowerPoint에서 마지막 검수와 다듬기를 수행합니다.

도구의 품질 상한은 모델과 사용자 판단의 품질에 영향을 받습니다. 중요한 수치·출처·표현은 최종 발표 전에 반드시 확인해야 합니다.

## 전체 아키텍처

```text
사용자 입력(PDF/DOCX/XLSX/PPTX/URL/Markdown/이미지)
    ↓
[소스 변환]
    source_to_md/pdf_to_md.py
    source_to_md/doc_to_md.py
    source_to_md/excel_to_md.py
    source_to_md/ppt_to_md.py
    source_to_md/web_to_md.py
    ↓
[프로젝트 생성]
    project_manager.py init <project_name> --format <format>
    ↓
[템플릿 선택 — 선택 사항]
    자유 디자인 또는 Brand/Layout/Deck 병합
    ↓
[Strategist]
    디자인 확인 → design_spec.md + spec_lock.md
    ↓
[이미지 확보 — 필요 시]
    사용자 자산 / AI 생성 / 웹 검색
    ↓
[Executor]
    SVG 페이지 생성 → svg_output/
    품질 검사 → svg_quality_checker.py
    발표자 노트 → notes/total.md
    ↓
[차트 검증·시각 검토 — 선택 사항]
    ↓
[후처리]
    total_md_split.py
    finalize_svg.py
    svg_to_pptx.py
    ↓
[출력]
    exports/<name>_<timestamp>.pptx
    backup/<timestamp>/svg_output/
```

기본 산출물은 네이티브 DrawingML PPTX입니다. `--svg-snapshot`을 사용하면 시각 비교용 SVG 이미지 PPTX를 추가로 만들 수 있지만, 전달·편집의 기준 파일은 네이티브 PPTX입니다.

## 3단계 파이프라인

### 1. 콘텐츠 이해와 설계 계획

입력 문서를 Markdown과 구조화 정보로 정규화합니다. Strategist는 청중, 목적, 페이지 수, 캔버스, 색상, 타이포그래피, 이미지 전략을 정리하고 슬라이드별 스토리라인을 만듭니다.

### 2. SVG 시각 생성

Executor는 슬라이드를 절대 좌표 기반 SVG로 생성합니다. SVG는 브라우저에서 즉시 미리 볼 수 있고, 텍스트로 디버깅할 수 있으며, AI가 DrawingML보다 안정적으로 작성할 수 있습니다.

### 3. DrawingML 변환

후처리 스크립트는 SVG의 도형, 텍스트, 경로, 변환, 그라디언트 등을 PowerPoint DrawingML로 변환합니다. 지원되는 요소는 이미지 한 장이 아니라 클릭·편집 가능한 PowerPoint 객체가 됩니다.

## 왜 SVG인가

### DrawingML 직접 생성의 문제

DrawingML은 매우 장황하고 중첩이 깊습니다. 단순한 도형도 많은 XML이 필요해 AI 생성 안정성과 사람이 디버깅할 가능성이 낮습니다.

### HTML/CSS의 구조적 차이

HTML은 콘텐츠 흐름이 위치를 결정하는 문서 모델이고, PowerPoint는 모든 요소가 독립적인 절대 좌표를 갖는 캔버스 모델입니다. `<table>`이나 흐름형 레이아웃을 독립 도형으로 정확히 매핑하기 어렵습니다.

### SVG 이미지 삽입의 한계

SVG 전체를 그림으로 삽입하면 시각적 일치는 쉬워도 텍스트·도형 편집성이 사라집니다.

### SVG와 DrawingML의 대응

| SVG | DrawingML |
|---|---|
| `<path d="...">` | `<a:custGeom>` |
| `<rect rx="...">` | `<a:prstGeom prst="roundRect">` |
| `<circle>` / `<ellipse>` | `<a:prstGeom prst="ellipse">` |
| `transform` | `<a:xfrm>` |
| `linearGradient` / `radialGradient` | `<a:gradFill>` |
| `fill-opacity` / `stroke-opacity` | `<a:alpha>` |
| `<text>` / `<tspan>` | `<a:p>` / `<a:r>` |

두 형식은 절대 좌표 기반 2D 벡터라는 공통 모델을 가지므로 번역이 가능합니다.

## 좌표와 단위

SVG는 픽셀 기반 `viewBox`를 사용하고, 내보내기 시 한 번만 PowerPoint EMU로 변환합니다.

```text
1 SVG px = 9525 EMU (96 DPI 기준)
```

이 방식은 Strategist와 Executor가 사람이 이해하기 어려운 EMU를 직접 다루지 않도록 합니다.

## 텍스트 변환

텍스트 변환은 다음 정보를 런 단위로 처리합니다.

- 텍스트 내용
- 글꼴 계열
- 크기와 굵기
- 이탤릭·밑줄·취소선
- 채우기와 윤곽선
- 언어 태그
- `latin`, `ea`, `cs` 글꼴 슬롯

### 원본 커밋 `4a27eaa`의 역할

원본 커밋은 텍스트 상자 전체가 아니라 **각 텍스트 런별로 언어를 감지**하도록 변경했습니다. CJK 문자가 있으면 `zh-CN`, 그렇지 않으면 `en-US`를 지정하고, 영문 런이 중국어 EA 기본 글꼴로 떨어지는 문제를 줄였습니다. 발표자 노트도 문단별 언어를 기록하도록 연결했습니다.

### 한국어 포크의 확장

이 포크는 해당 구조를 유지하면서 한국어 경로를 추가합니다.

```text
한글 포함 런 → lang="ko-KR"
중국어/CJK 런 → lang="zh-CN"
순수 라틴 런 → lang="en-US"
```

한글 감지 범위:

- Hangul Jamo: U+1100–U+11FF
- Hangul Compatibility Jamo: U+3130–U+318F
- Halfwidth Hangul: U+FFA0–U+FFDC
- Hangul Jamo Extended-A: U+A960–U+A97F
- Hangul Syllables: U+AC00–U+D7A3
- Hangul Jamo Extended-B: U+D7B0–U+D7FF

글꼴 사전은 `latin`, `ea`, `ko` 슬롯을 유지합니다.

```python
{
    'latin': 'Segoe UI',
    'ea': 'Microsoft YaHei',
    'ko': 'Malgun Gothic',
}
```

런별 선택:

- `ko-KR` → `ko`
- `zh-CN` → `ea`
- `en-US` → `latin`

기본 한국어 세리프는 `Batang`, 기본 산세리프는 `Malgun Gothic`입니다. 한국어 글꼴만 지정된 SVG에서는 PowerPoint의 CJK 렌더링 특성을 고려해 라틴 슬롯도 해당 글꼴로 맞출 수 있습니다.

## 한국어 글꼴 정규화

변환기는 다음 계열을 인식합니다.

- Malgun Gothic / 맑은 고딕
- Gulim / 굴림, GulimChe / 굴림체
- Dotum / 돋움, DotumChe / 돋움체
- Batang / 바탕, BatangChe / 바탕체
- Gungsuh / 궁서, GungsuhChe / 궁서체
- Apple SD Gothic Neo / AppleGothic
- Noto Sans/Serif KR 및 CJK KR
- Source Han Sans/Serif KR
- Nanum Gothic / Nanum Myeongjo

Windows PowerPoint 호환성을 우선해 Apple, Noto, Source Han, Nanum 계열 일부는 맑은 고딕 또는 바탕으로 매핑합니다.

## 텍스트 폭과 줄 처리

`estimate_text_width`는 한글을 CJK 전각 문자와 같은 1em 폭으로 추정합니다. 이는 라틴 기본 폭으로 계산할 때 발생하는 텍스트 상자 축소와 예기치 않은 줄바꿈을 줄입니다.

문단 병합 과정에서도 한글 경계를 CJK 경계로 취급해 SVG의 줄바꿈을 합칠 때 불필요한 공백을 추가하지 않습니다.

## 소스 변환

일반 형식은 Python 라이브러리를 우선하고, 일부 레거시 형식에서만 외부 바이너리를 사용합니다. 변환된 Markdown이 Strategist가 읽는 콘텐츠 기준점입니다.

중요 원칙:

- 원본 수치·날짜·단위 보존
- 표와 목록의 구조 유지
- 페이지·슬라이드·시트 출처 추적
- 이미지와 캡션 관계 보존
- 변환 실패 시 원본 파일을 훼손하지 않음

## 프로젝트 수명주기

프로젝트 외부의 사용자 파일은 원본 보존을 위해 복사하고, 저장소 내부의 임시 산출물은 정리를 위해 이동될 수 있습니다. 실제 동작은 `project_manager.py`와 현재 워크플로가 기준입니다.

권장 구조:

```text
projects/<name>/
├── sources/
├── templates/
├── design_spec.md
├── spec_lock.md
├── svg_output/
├── svg_final/
├── notes/
├── audio/
├── backup/
└── exports/
```

## 템플릿 경로

템플릿은 기본값이 아니라 사용자가 명시할 때 활성화됩니다. 자유 디자인이 콘텐츠에 맞춰 구조를 새로 만들 수 있는 반면, 템플릿은 브랜드 고정 보고서나 학술 발표처럼 제약이 중요한 작업에 적합합니다.

Brand, Layout, Deck 병합 규칙은 [템플릿 아키텍처](./templates-architecture.md)를 참고하세요.

## 역할 시스템

PPT Master는 하나의 주 에이전트가 역할을 전환하는 구조를 사용합니다.

- Strategist: 사용자 확인, 스토리라인, 디자인 명세
- Executor: 엄격한 SVG 생성과 노트 작성
- 후처리/검토: 품질 검사, 변환, 선택적 시각 검토

병렬 하위 에이전트나 여러 페이지 동시 생성은 속도를 높일 수 있지만 장기 컨텍스트와 시각 일관성이 흔들릴 수 있어 기본 경로에서 제한됩니다.

## `design_spec.md`와 `spec_lock.md`

- `design_spec.md`: 대상 청중, 스타일 목적, 색상 근거, 페이지 계획 같은 인간 친화적 설명
- `spec_lock.md`: 정확한 HEX, 글꼴 문자열, 아이콘 라이브러리, 이미지 상태 같은 실행 계약

Executor가 페이지마다 `spec_lock.md`를 다시 읽으면 긴 덱에서 색상과 글꼴이 서서히 변하는 현상을 줄일 수 있습니다.

## 이미지 확보

이미지는 다음 우선순위를 사용할 수 있습니다.

1. 사용자 제공 고해상도 자산
2. AI 이미지 생성
3. 라이선스가 확인된 웹 이미지 검색
4. 단순 도형·아이콘 대체

공급자별 키를 별도로 사용하고 `IMAGE_BACKEND`로 활성 백엔드를 명시합니다. 웹 이미지에서 저작자 표시가 필요하면 슬라이드에 크레딧을 추가합니다.

## 품질 검사

SVG 품질 검사는 내보내기 전에 수행합니다. 대표 검사 항목:

- 유효하지 않은 XML
- 캔버스 밖 요소
- 누락된 필수 속성
- 비정상적인 텍스트 크기
- 지원되지 않는 참조
- 잘못된 색상과 경로

품질 검사 통과가 시각적 완성도를 보장하지는 않으므로 PowerPoint 렌더링 검토가 추가로 필요합니다.

## 발표자 노트와 오디오

발표자 노트는 PPTX notes XML로 내보내며 문단별 언어 태그를 사용합니다. 한국어 문단은 `ko-KR`로 기록되므로 TTS 워크플로가 주 언어를 선택하는 데 도움이 됩니다.

오디오 생성과 삽입은 [오디오 내레이션](./audio-narration.md)을 참고하세요.

## 애니메이션

전환과 개체 등장 효과는 OOXML 타이밍 구조로 추가됩니다. 녹화 내레이션과 동영상 내보내기를 사용할 때는 클릭 기반 애니메이션보다 자동 타이밍 기반 효과가 안전합니다.

자세한 내용은 [애니메이션](./animations.md)을 참고하세요.

## 확장 시 원칙

- 공통 로직은 공유 헬퍼에 추가하고 호출부별 복제를 피함
- 기존 영어·중국어 동작을 보존하며 새 언어 경로 추가
- 글꼴 이름과 언어 태그를 명시적으로 테스트
- 자동 테스트 파일을 추가하지 않는 저장소 규칙을 따르고 인라인 스모크 명령으로 검증
- 지원되지 않는 SVG 효과는 조용히 잘못 변환하기보다 경고 또는 명시적 대체 사용
- 결과 PPTX를 실제 PowerPoint에서 확인

코드와 이 문서가 충돌하면 현재 브랜치의 코드, `skills/ppt-master/SKILL.md`, 관련 워크플로가 우선합니다.
