# Windows 설치 가이드

이 문서는 Windows 10/11에서 PPT Master를 설치하고 첫 실행까지 확인하는 절차를 설명합니다.

[한국어 문서 목차](./README.md) · [시작하기](./getting-started.md) · [FAQ](./faq.md)

## 1. Python 설치

1. [python.org](https://www.python.org/downloads/)에서 Python 3.10 이상 설치 프로그램을 내려받습니다.
2. 설치 첫 화면에서 **Add python.exe to PATH**를 반드시 선택합니다.
3. 기본 설치 또는 사용자 지정 설치를 완료합니다.
4. 열려 있던 PowerShell과 터미널을 모두 닫고 새로 엽니다.

확인:

```powershell
python --version
pip --version
```

`python` 대신 Python Store 화면이 열리면 Windows의 앱 실행 별칭을 확인하세요.

- 설정 → 앱 → 고급 앱 설정 → 앱 실행 별칭
- `python.exe`, `python3.exe`의 Store 별칭을 끄거나 실제 Python 경로가 먼저 오도록 PATH를 수정

설치 관리자가 `py` 런처를 설치했다면 다음 명령도 사용할 수 있습니다.

```powershell
py --version
py -m pip --version
```

## 2. Git 설치 또는 ZIP 다운로드

### 방법 A: Git 사용

[Git for Windows](https://git-scm.com/downloads/win)을 설치한 뒤 PowerShell에서 실행합니다.

```powershell
git clone https://github.com/Leafmire/ppt-master.git
cd ppt-master
```

### 방법 B: ZIP 사용

GitHub 저장소에서 **Code → Download ZIP**을 선택하고, 쓰기 권한이 있는 폴더에 압축을 풉니다. OneDrive 동기화 폴더나 너무 긴 경로는 파일 잠금과 경로 길이 문제를 일으킬 수 있으므로 다음처럼 짧은 위치가 편합니다.

```text
C:\work\ppt-master
```

## 3. 가상환경 만들기

필수는 아니지만 프로젝트별 의존성을 분리하기 위해 권장합니다.

```powershell
cd C:\work\ppt-master
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

PowerShell에서 스크립트 실행이 차단되면 현재 사용자 범위에서 정책을 완화할 수 있습니다.

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

회사 정책으로 변경할 수 없다면 `cmd.exe`에서 다음 파일을 실행합니다.

```cmd
.venv\Scripts\activate.bat
```

또는 가상환경을 활성화하지 않고 해당 Python을 직접 사용합니다.

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

## 4. 설치 확인

```powershell
python skills\ppt-master\scripts\svg_to_pptx.py --help
```

도움말이 표시되면 Python이 스크립트를 읽고 기본 import를 수행할 수 있는 상태입니다.

## 5. 에이전트에서 폴더 열기

VS Code, Cursor 또는 사용하는 IDE에서 저장소 루트를 엽니다.

```powershell
code C:\work\ppt-master
```

에이전트에게 다음 파일을 읽도록 요청합니다.

```text
skills/ppt-master/SKILL.md를 읽고 PPT Master 워크플로를 준비해 줘.
```

## 6. 첫 프로젝트

```powershell
mkdir projects\first-deck\sources
```

PDF나 DOCX를 `sources` 폴더에 복사한 뒤 요청합니다.

```text
projects/first-deck/sources의 자료로 한국어 16:9 PPT를 만들어 줘.
8장 내외로 구성하고 텍스트와 차트는 편집 가능하게 유지해 줘.
```

## 7. 한국어 글꼴 확인

이 포크는 한글 런을 `ko-KR`로 기록하고 기본적으로 다음 글꼴을 사용합니다.

- 산세리프: 맑은 고딕 (`Malgun Gothic`)
- 세리프: 바탕 (`Batang`)

Windows 기본 설치에서 일반적으로 사용할 수 있지만, 조직에서 경량 이미지나 커스텀 Windows를 배포한 경우 글꼴이 제거되어 있을 수 있습니다.

PowerShell에서 글꼴 파일 확인 예시:

```powershell
Get-ChildItem "$env:WINDIR\Fonts" | Where-Object {
    $_.Name -match 'malgun|batang'
}
```

Noto Sans KR, 나눔고딕, Apple SD Gothic Neo 같은 글꼴 이름은 PPTX 변환 단계에서 Windows 호환 한국어 글꼴로 정규화됩니다.

## 8. 자주 발생하는 문제

### `python`을 찾을 수 없음

새 터미널을 열고 다음을 확인합니다.

```powershell
where.exe python
where.exe py
```

경로가 없다면 Python을 다시 설치하면서 PATH 옵션을 선택합니다.

### `pip`를 찾을 수 없음

```powershell
python -m pip install -r requirements.txt
```

`pip` 실행 파일 대신 모듈 방식으로 호출하면 PATH 문제를 피할 수 있습니다.

### `ModuleNotFoundError`

현재 터미널에서 올바른 가상환경이 활성화되어 있는지 확인하고 의존성을 다시 설치합니다.

```powershell
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

### PowerShell 실행 정책 오류

현재 사용자만 변경:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

정책 변경이 허용되지 않으면 `cmd.exe`의 `activate.bat` 또는 가상환경 Python의 전체 경로를 사용합니다.

### 파일 경로가 너무 김

프로젝트를 `C:\work\ppt-master`처럼 짧은 경로로 옮기고, 필요하다면 Windows의 긴 경로 정책을 활성화합니다. Git에서도 다음 설정을 사용할 수 있습니다.

```powershell
git config --global core.longpaths true
```

### 한글 경로에서 도구가 실패함

대부분의 Python 경로는 UTF-8을 지원하지만, 외부 프로그램이 한글 경로를 처리하지 못할 수 있습니다. 진단을 위해 저장소와 프로젝트를 영문·숫자 위주의 짧은 경로로 옮겨 보세요.

### PowerPoint에서 한국어 글꼴이 바뀜

- 해당 PC에 글꼴이 설치되어 있는지 확인
- SVG의 `font-family`에 Windows 대체 글꼴 포함
- 배포용 파일은 `Malgun Gothic` 또는 `Batang` 우선
- PowerPoint의 글꼴 바꾸기 기능으로 문서 전체의 대체 상태 확인

### 생성 파일이 열리지 않음

- 내보내기 로그의 첫 오류를 확인
- 파일 확장자가 `.pptx`인지 확인
- 임시 파일이 아닌 `exports/`의 최종 파일을 열기
- PowerPoint 2016 이상에서 재확인
- 동일 프로젝트의 SVG 백업으로 다시 내보내기

## 9. 업데이트

Git 설치:

```powershell
git status
git pull
python -m pip install -r requirements.txt
```

포크의 한국어 변경을 유지하려면 원격을 확인합니다.

```powershell
git remote -v
```

`origin`이 `https://github.com/Leafmire/ppt-master.git`를 가리키는지 확인하세요.

## 다음 문서

- [시작하기](./getting-started.md)
- [FAQ](./faq.md)
- [오디오 내레이션](./audio-narration.md)
