# PPT Master 한국어 문서

[루트 한국어 README](../../README_KO.md) · [English docs](../) · [中文文档](../zh/)

이 디렉터리는 `Leafmire/ppt-master` 포크의 한국어 사용자를 위한 문서 모음입니다. 명령어, 파일 경로, 설정 키는 코드와 호환되도록 원문 표기를 유지하고 설명을 한국어로 제공합니다.

## 처음 사용하는 경우

1. [시작하기](./getting-started.md)
2. [Windows 설치 가이드](./windows-installation.md) — Windows 사용자
3. [FAQ](./faq.md) — 생성·내보내기·레이아웃 문제

## 기능별 문서

| 문서 | 설명 |
|---|---|
| [왜 PPT Master인가](./why-ppt-master.md) | 네이티브 편집형 PPTX 방식의 장점과 한계 |
| [템플릿 사용 가이드](./templates-guide.md) | 기존 PPTX, 레이아웃, 브랜드 템플릿 활용 |
| [템플릿 아키텍처](./templates-architecture.md) | 템플릿 디렉터리와 변환 계층 |
| [기술 설계](./technical-design.md) | 소스 분석부터 DrawingML 내보내기까지 |
| [오디오 내레이션](./audio-narration.md) | 발표자 노트를 음성으로 생성하고 삽입 |
| [애니메이션](./animations.md) | 슬라이드 전환 및 개체 등장 효과 |
| [로드맵](./roadmap.md) | 한국어 포크의 유지보수 방향 |

## 한국어 글꼴 처리

이 포크는 SVG → PPTX 변환 과정에서 한글을 감지해 DrawingML 언어를 `ko-KR`로 기록합니다. 한글 완성형뿐 아니라 자모·호환 자모·확장 자모도 포함합니다.

기본값:

| 용도 | 글꼴 |
|---|---|
| 한국어 산세리프 | `Malgun Gothic` |
| 한국어 세리프 | `Batang` |
| 영문 | `Segoe UI` 또는 SVG 지정 글꼴 |
| 중국어 | 기존 `Microsoft YaHei` / `SimSun` 경로 유지 |

권장 SVG 글꼴 스택:

```svg
<text font-family="Noto Sans KR, Malgun Gothic, sans-serif">한국어</text>
<text font-family="Noto Serif KR, Batang, serif">한국어 본문</text>
```

## 원문과 차이가 있을 때

코드 동작과 영문 문서가 변경되었지만 한국어 번역이 아직 갱신되지 않은 경우, 다음 순서로 확인하세요.

1. 현재 브랜치의 코드와 `skills/ppt-master/SKILL.md`
2. 같은 이름의 영문 문서
3. 한국어 문서

번역이 구현보다 앞서지 않도록 명령어와 옵션 이름은 가능한 한 원문 그대로 유지합니다.
