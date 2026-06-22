# PPT Master — 문서에서 네이티브 편집 가능한 PPTX를 만드는 AI 워크플로

[![Version](https://img.shields.io/badge/version-v2.11.0-blue.svg)](https://github.com/hugohe3/ppt-master/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

[English](./README.md) | [中文](./README_CN.md) | **한국어**

> [!NOTE]
> 이 저장소는 [`hugohe3/ppt-master`](https://github.com/hugohe3/ppt-master)를 기반으로 한 한국어 친화 포크입니다. 원본 프로젝트의 저작권과 크레딧은 원저작자에게 있으며, 이 포크의 한국어 문서와 한국어 글꼴 지원은 `Leafmire/ppt-master`에서 별도로 관리합니다.

> [!IMPORTANT]
> PPT Master는 한 번의 요청으로 언제나 완벽한 발표 자료를 만들어 주는 자동 완성 서비스가 아닙니다. 자료 분석, 시각 설계, SVG 생성, PPTX 내보내기 같은 반복 작업을 크게 줄여 주는 **에이전트용 워크플로**입니다. 결과물은 이미지 한 장이 아니라 PowerPoint에서 개별 요소를 직접 수정할 수 있는 문서이므로, 마지막 다듬기는 사용자가 이어서 할 수 있습니다.

## PPT Master가 만드는 것

원본 자료를 넣으면 다음 특성을 가진 실제 `.pptx`를 생성합니다.

- 텍스트, 도형, 표, 차트가 PowerPoint에서 개별 선택·편집 가능
- 슬라이드 전환, 등장 애니메이션, 발표자 노트 지원
- 발표자 노트를 기반으로 한 음성 내레이션 생성 가능
- 기존 PPTX 템플릿을 활용한 콘텐츠 채우기 가능
- PDF, DOCX, 이미지, 웹 문서 등 다양한 입력 자료 처리
- 로컬 파일 중심의 워크플로로 플랫폼 종속 최소화

AI 프레젠테이션 도구를 결과 형식으로 나누면 다음과 같습니다.

| 방식 | 결과 | PowerPoint에서 요소별 편집 |
|---|---|:---:|
| 고정 템플릿 채우기 | 템플릿 기반 PPTX | 일부 가능 |
| 이미지 슬라이드 | 슬라이드마다 큰 이미지 한 장 | 불가 |
| HTML 프레젠테이션 | 웹 페이지 | 불가 |
| **PPT Master** | **DrawingML 텍스트·도형·차트 기반 PPTX** | **가능** |

## 빠른 시작

### 1. 준비 사항

필수 런타임은 **Python 3.10 이상**입니다.

```bash
python --version
pip install -r requirements.txt
```

Windows 사용자는 [Windows 설치 가이드](./docs/ko/windows-installation.md)를 먼저 읽는 것이 좋습니다.

macOS 예시:

```bash
brew install python
pip install -r requirements.txt
```

Ubuntu / Debian 예시:

```bash
sudo apt install python3 python3-pip
pip install -r requirements.txt
```

`pandoc`은 `.doc`, `.odt`, `.rtf`, `.tex`, `.rst`, `.org`, `.typ` 같은 일부 레거시 형식에서만 필요합니다. 일반적인 `.docx`, `.html`, `.epub`, `.ipynb`는 Python 경로로 처리됩니다.

### 2. 에이전트 선택

PPT Master는 파일 읽기·쓰기, 명령 실행, 여러 차례 대화를 지원하는 에이전트 환경에서 동작합니다.

- IDE 내장 에이전트: Cursor, Windsurf, Zed 계열 등
- IDE 확장: Claude Code, GitHub Copilot, Cline, Continue, Roo Code 등
- CLI 에이전트: Claude Code CLI, Codex CLI, Gemini CLI, Aider 등

모델이 워크플로의 품질 상한을 결정합니다. 복잡한 자료일수록 긴 컨텍스트와 안정적인 도구 사용 능력을 가진 모델이 유리합니다.

### 3. 설치

ZIP을 내려받아 압축을 풀거나 Git으로 이 포크를 복제합니다.

```bash
git clone https://github.com/Leafmire/ppt-master.git
cd ppt-master
pip install -r requirements.txt
```

이후 업데이트:

```bash
python3 skills/ppt-master/scripts/update_repo.py
```

> 업데이트 스크립트의 원격 저장소 설정은 설치 방식에 따라 다를 수 있습니다. 포크의 변경을 유지하려면 Git 원격이 `Leafmire/ppt-master`를 가리키는지 확인하세요.

### 4. 첫 프레젠테이션 만들기

자료를 `projects/` 아래에 넣고 에이전트에게 경로와 목적을 알려 줍니다.

```text
projects/q3-report/sources/report.pdf를 바탕으로
경영진 보고용 10장짜리 16:9 프레젠테이션을 만들어 줘.
한국어로 작성하고, 수치와 출처를 보존해 줘.
```

또는 대화창에 내용을 직접 붙여 넣을 수 있습니다.

에이전트는 먼저 페이지 수, 형식, 템플릿, 분위기, 색상, 이미지 전략 등의 디자인 명세를 확인한 뒤 콘텐츠 분석 → 시각 설계 → SVG 생성 → PPTX 내보내기를 수행합니다.

기본 결과물은 다음 위치에 저장됩니다.

```text
exports/<이름>_<타임스탬프>.pptx
```

SVG 작업본은 재내보내기와 보관을 위해 백업됩니다.

## 한국어 및 글꼴 지원

이 포크는 원본의 텍스트 런별 언어 판별 로직을 한국어까지 확장합니다.

- 한글 완성형, 자모, 호환 자모, 확장 자모를 감지해 DrawingML 언어를 `ko-KR`로 기록
- 한국어 런은 중국어 기본 글꼴 대신 한국어용 East Asian 글꼴 슬롯 사용
- 기본 산세리프: `Malgun Gothic`(맑은 고딕)
- 기본 세리프: `Batang`(바탕)
- Apple SD Gothic Neo, Noto Sans/Serif KR, Source Han KR, 나눔고딕/나눔명조 등의 일반적인 글꼴 이름을 Windows PowerPoint 호환 글꼴로 정규화
- 영문 런은 계속 `en-US`와 라틴 글꼴을 사용하고, 중국어 런은 기존 `zh-CN` 경로를 유지
- 한글을 전각 문자 폭으로 계산해 텍스트 상자 크기와 줄바꿈 추정 개선

SVG에서 글꼴을 직접 지정할 수 있습니다.

```svg
<text font-family="Noto Sans KR, Malgun Gothic, sans-serif">한국어 제목</text>
<text font-family="Noto Serif KR, Batang, serif">한국어 본문</text>
```

배포 대상 PC에 지정 글꼴이 설치되어 있지 않으면 PowerPoint가 대체 글꼴을 사용합니다. 여러 환경에서 열어야 하는 파일은 맑은 고딕과 바탕처럼 Windows 기본 글꼴을 우선하는 편이 안전합니다.

## 기존 PPTX 템플릿 사용

이미 사용 중인 회사·학교 템플릿이 있다면 원본 자료와 함께 에이전트에 전달하고 다음처럼 요청합니다.

```text
projects/template/company-template.pptx의 디자인을 유지하면서
projects/source/brief.docx 내용을 채워 줘.
표와 차트는 편집 가능한 상태로 만들고, 선택한 레이아웃만 사용해 줘.
```

자세한 내용은 [템플릿 사용 가이드](./docs/ko/templates-guide.md)를 참고하세요.

## 이미지 가져오기

사용자가 제공하지 않은 이미지는 두 경로를 섞어 사용할 수 있습니다.

### AI 이미지 생성

`.env`에 백엔드와 API 키를 설정합니다.

```bash
cp .env.example .env
python3 skills/ppt-master/scripts/image_gen.py --list-backends
```

### 웹 이미지 검색

키 없이도 Openverse와 Wikimedia Commons를 사용할 수 있습니다. `PEXELS_API_KEY` 또는 `PIXABAY_API_KEY`를 추가하면 현대적인 스톡 사진 선택지가 늘어납니다.

저작자 표시가 필요한 이미지는 슬라이드에 짧은 출처를 남겨야 합니다. 표지, 인물, 제품, 브랜드 장면처럼 중요도가 높은 이미지는 사용자 제공 고해상도 자산 또는 AI 생성 이미지를 우선하는 것이 좋습니다.

## 오디오 내레이션과 애니메이션

- 발표자 노트를 작성한 뒤 음성으로 변환: [오디오 내레이션](./docs/ko/audio-narration.md)
- 슬라이드 전환과 개체 등장 효과: [애니메이션](./docs/ko/animations.md)

## 문서

| 문서 | 내용 |
|---|---|
| [한국어 문서 안내](./docs/ko/README.md) | 한국어 문서 전체 목차 |
| [시작하기](./docs/ko/getting-started.md) | 설치부터 첫 PPTX 생성까지 |
| [Windows 설치](./docs/ko/windows-installation.md) | PATH, PowerShell, 가상환경 문제 해결 |
| [FAQ](./docs/ko/faq.md) | 품질, 비용, 레이아웃, 내보내기 문제 |
| [왜 PPT Master인가](./docs/ko/why-ppt-master.md) | 다른 AI 프레젠테이션 방식과의 차이 |
| [템플릿 가이드](./docs/ko/templates-guide.md) | 기존 PPTX 및 SVG 템플릿 활용 |
| [템플릿 아키텍처](./docs/ko/templates-architecture.md) | 템플릿 계층과 변환 구조 |
| [기술 설계](./docs/ko/technical-design.md) | 전체 파이프라인과 DrawingML 구조 |
| [오디오 내레이션](./docs/ko/audio-narration.md) | 노트에서 음성 생성·삽입 |
| [애니메이션](./docs/ko/animations.md) | 전환 및 등장 효과 구성 |
| [로드맵](./docs/ko/roadmap.md) | 한국어 포크의 유지보수 방향 |
| [원본 SKILL.md](./skills/ppt-master/SKILL.md) | 에이전트가 따르는 핵심 워크플로 |

## 문제 해결

에이전트가 작업 흐름을 잃었을 때:

```text
skills/ppt-master/SKILL.md를 다시 읽고 현재 프로젝트의 진행 상태를 확인한 뒤 계속해 줘.
```

그 밖의 흔한 문제는 [FAQ](./docs/ko/faq.md)를 확인하세요.

## 원저작자와 라이선스

PPT Master 원본은 [Hugo He](https://www.hehugo.com/)가 만들었습니다.

- 원본 저장소: [`hugohe3/ppt-master`](https://github.com/hugohe3/ppt-master)
- 한국어 포크: [`Leafmire/ppt-master`](https://github.com/Leafmire/ppt-master)
- 라이선스: [MIT](./LICENSE)

원본 프로젝트의 기여 안내는 [CONTRIBUTING.md](./CONTRIBUTING.md)를 참고하세요. 이 포크의 한국어 변경은 원본 저장소로 제출하지 않고 포크 내부에서 관리합니다.
