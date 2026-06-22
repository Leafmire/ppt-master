# 시작하기

이 문서는 PPT Master를 설치하고 첫 번째 네이티브 편집형 `.pptx`를 만드는 과정을 설명합니다.

[한국어 문서 목차](./README.md) · [Windows 설치](./windows-installation.md) · [FAQ](./faq.md)

## 1. 준비 사항

필수 항목:

- Python 3.10 이상
- 파일 읽기·쓰기와 명령 실행이 가능한 AI 에이전트
- 프레젠테이션의 근거가 될 자료

버전 확인:

```bash
python --version
# 환경에 따라 python3 --version
```

저장소 설치:

```bash
git clone https://github.com/Leafmire/ppt-master.git
cd ppt-master
pip install -r requirements.txt
```

Windows에서 `python`이나 `pip` 명령을 찾지 못하면 [Windows 설치 가이드](./windows-installation.md)를 참고하세요.

## 2. 프로젝트 디렉터리 준비

작업마다 `projects/` 아래에 별도 디렉터리를 만드는 것을 권장합니다.

```text
projects/
└── quarterly-report/
    └── sources/
        ├── report.pdf
        ├── metrics.xlsx
        └── brand-logo.png
```

원본 파일은 `sources/`에 모아 두면 에이전트가 참조 범위를 명확히 이해하기 쉽습니다. 경로에는 공백을 사용할 수 있지만, 명령줄에서 다룰 때는 따옴표로 감싸야 합니다.

## 3. 에이전트에 요청하기

좋은 요청에는 목적, 청중, 분량, 언어, 강조할 사실, 시각적 제약이 포함됩니다.

```text
projects/quarterly-report/sources의 자료를 바탕으로
한국어 경영진 보고용 16:9 프레젠테이션을 만들어 줘.

- 10~12장
- 핵심 수치와 출처를 유지
- 결론을 첫 3장 안에 제시
- 차트와 표는 PowerPoint에서 편집 가능하게 구성
- 회사 로고 색을 포인트 컬러로 사용
- 모든 한국어 텍스트는 깨지지 않도록 처리
```

자료가 짧다면 대화창에 직접 붙여 넣을 수도 있습니다.

## 4. 디자인 명세 확인

에이전트는 보통 생성 전에 다음 항목을 확인합니다.

- 출력 형식과 화면 비율
- 예상 페이지 수
- 자유 디자인 또는 템플릿 사용 여부
- 색상과 타이포그래피
- 이미지 검색 또는 AI 이미지 생성 여부
- 전환, 애니메이션, 발표자 노트, 내레이션 여부

이 단계에서 요구를 구체화할수록 뒤의 수정 비용이 줄어듭니다.

한국어 문서의 기본 글꼴 전략을 직접 지정하려면 다음처럼 말할 수 있습니다.

```text
한국어 제목은 맑은 고딕 계열, 본문은 맑은 고딕으로 사용하고,
영문 숫자와 라틴 문자는 Segoe UI 계열을 유지해 줘.
```

## 5. 생성 파이프라인

PPT Master는 대체로 다음 순서로 작업합니다.

1. 입력 자료를 Markdown과 구조화된 정보로 변환
2. 핵심 메시지, 근거, 스토리라인 정리
3. 디자인 명세와 슬라이드별 계획 작성
4. 각 슬라이드를 SVG로 생성
5. 시각 검토와 레이아웃 수정
6. SVG를 DrawingML 기반 PPTX로 변환
7. 필요 시 애니메이션, 발표자 노트, 오디오 삽입
8. 결과와 작업본 보관

결과물 예시:

```text
projects/quarterly-report/
├── svg_output/
├── svg_final/
├── backup/
└── exports/
    └── quarterly_report_20260622_153000.pptx
```

실제 디렉터리 이름은 워크플로와 프로젝트 상태에 따라 달라질 수 있습니다.

## 6. 결과 확인

PowerPoint에서 파일을 열고 다음을 확인하세요.

- 텍스트와 도형을 개별 선택할 수 있는가
- 한국어 글꼴이 의도한 계열로 표시되는가
- 줄바꿈과 자간이 어색하지 않은가
- 차트와 표의 값이 원본과 일치하는가
- 이미지 출처 표기가 필요한 곳에 남아 있는가
- 슬라이드 쇼에서 전환과 애니메이션이 동작하는가
- 발표자 노트가 보존되는가

한국어 글꼴이 다른 글꼴로 바뀌면 해당 PC에 지정 글꼴이 설치되어 있는지 확인하세요. 여러 Windows 환경에 배포할 파일은 `Malgun Gothic`과 `Batang`을 우선하는 편이 안전합니다.

## 7. 수정 요청

전체를 다시 만들기보다 슬라이드와 문제를 특정하면 효율적입니다.

```text
5번 슬라이드만 수정해 줘.
- 제목을 한 줄로 유지
- 왼쪽 차트 범례를 아래로 이동
- 한국어 본문을 18pt 이상으로 확대
- 원본 수치와 주석은 변경하지 말 것
```

에이전트가 현재 상태를 잃은 경우:

```text
skills/ppt-master/SKILL.md를 다시 읽고,
현재 프로젝트 디렉터리의 산출물과 백업을 확인한 뒤 계속해 줘.
```

## 8. 기존 PPTX 템플릿 사용

회사나 학교의 기존 템플릿을 재사용하려면 템플릿 파일과 자료를 함께 전달합니다.

```text
projects/templates/company.pptx를 기반으로
projects/quarterly-report/sources/report.pdf 내용을 채워 줘.
마스터와 브랜드 요소를 유지하고, 선택한 레이아웃만 사용해 줘.
```

자세한 내용은 [템플릿 사용 가이드](./templates-guide.md)를 참고하세요.

## 9. 실시간 미리보기와 편집

프로젝트에 포함된 뷰어와 편집 도구는 SVG 작업본을 빠르게 검토할 때 사용합니다. 사용 가능한 명령은 다음 문서에서 확인하세요.

- `skills/ppt-master/scripts/README.md`
- `skills/ppt-master/SKILL.md`
- `skills/ppt-master/workflows/visual-review.md`

도구의 `--help`를 먼저 실행하면 현재 버전의 옵션을 가장 정확히 확인할 수 있습니다.

```bash
python3 skills/ppt-master/scripts/svg_to_pptx.py --help
```

## 10. 선택 기능

### 이미지 생성과 검색

```bash
cp .env.example .env
python3 skills/ppt-master/scripts/image_gen.py --list-backends
```

웹 이미지 검색은 API 키 없이도 일부 공급자를 사용할 수 있으며, Pexels/Pixabay 키를 설정하면 선택 범위가 넓어집니다.

### 오디오 내레이션

발표자 노트를 먼저 준비한 뒤 [오디오 내레이션 가이드](./audio-narration.md)를 따릅니다.

### 애니메이션

전환과 개체 등장 효과는 [애니메이션 가이드](./animations.md)를 참고하세요.

## 다음 단계

- 결과 품질과 방식의 차이: [왜 PPT Master인가](./why-ppt-master.md)
- 흔한 오류와 해결법: [FAQ](./faq.md)
- 내부 처리 구조: [기술 설계](./technical-design.md)
