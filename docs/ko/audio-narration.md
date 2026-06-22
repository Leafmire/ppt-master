# 오디오 내레이션과 동영상 내보내기

[한국어 문서 목차](./README.md) · [애니메이션](./animations.md) · [원문](../audio-narration.md)

PPT Master는 발표자 노트를 슬라이드별 음성 파일로 만들고, 해당 오디오를 PPTX에 삽입해 자동 재생과 동영상 내보내기에 사용할 수 있습니다.

기본 경로는 `edge-tts`이며, 필요에 따라 ElevenLabs, MiniMax, Qwen TTS, CosyVoice 같은 클라우드 TTS를 사용할 수 있습니다.

## 결과물

- `<project>/audio/` 아래에 슬라이드별 오디오 생성
- SVG 파일명과 대응하는 파일명 사용
- 발표자 노트 보존
- 선택적으로 오디오가 삽입된 새 PPTX 생성
- 오디오 길이에 맞춘 슬라이드 자동 전환 시간 설정

예시:

```text
projects/demo/
├── notes/
│   ├── 01_cover.md
│   └── 02_summary.md
├── audio/
│   ├── 01_cover.mp3
│   └── 02_summary.mp3
└── exports/
    └── demo_narrated.pptx
```

## 동작 방식

1. 발표자 노트를 실제로 읽을 수 있는 자연스러운 문장으로 작성
2. 덱의 주 언어 확인
3. 공급자의 음성 목록에서 후보 선택
4. 말하기 속도와 오디오 삽입 여부 확인
5. 슬라이드별 음성 생성
6. 선택 시 PPTX에 오디오와 자동 전환 시간 삽입

발표자 노트에는 `[강조]`, `Key points:`, `Duration:` 같은 읽히면 안 되는 메타 문구를 넣지 않는 편이 좋습니다.

## 한국어 처리

이 포크는 한국어 발표자 노트 문단을 DrawingML `ko-KR`로 기록합니다. TTS에서도 한국어 덱은 `ko-KR` 로케일 음성을 우선 선택하세요.

현재 사용 가능한 한국어 음성 확인:

```bash
python3 skills/ppt-master/scripts/notes_to_audio.py \
  --list-voices --locale ko-KR
```

음성 목록은 TTS 공급자와 시점에 따라 바뀔 수 있으므로 이름을 문서에 고정하기보다 위 명령으로 확인하는 것이 안전합니다.

## 에이전트에 요청하기

```text
이 PPT의 발표자 노트로 한국어 내레이션을 만들어 줘.
ko-KR 음성 중 차분한 여성 음성 후보를 추천하고,
기본 속도로 생성한 뒤 PPTX에 삽입해 줘.
```

```text
한국어 남성 음성으로 생성하되 숫자와 영문 약어를 또렷하게 읽는 음성을 골라 줘.
첫 슬라이드만 먼저 샘플로 만든 뒤 전체를 생성해 줘.
```

전체 단계는 `skills/ppt-master/workflows/generate-audio.md`가 기준입니다.

## 두 가지 삽입 경로

| 옵션 | 용도 |
|---|---|
| `--recorded-narration audio` | 모든 슬라이드의 완전한 내레이션과 자동 전환 시간을 설정. 동영상·자동 재생용 |
| `--narration-audio-dir audio` | 일부 슬라이드만 포함할 수 있는 저수준 오디오 삽입. 테스트·수동 마무리용 |

`--recorded-narration`은 페이지별 오디오가 완전하게 준비되어야 합니다.

## 수동 실행

### 1. 발표자 노트 분할

```bash
python3 skills/ppt-master/scripts/total_md_split.py <project_path>
```

### 2. Edge TTS

설치:

```bash
python3 -m pip install edge-tts
```

음성 목록:

```bash
python3 skills/ppt-master/scripts/notes_to_audio.py \
  --list-voices --locale ko-KR
```

생성:

```bash
python3 skills/ppt-master/scripts/notes_to_audio.py <project_path> \
  --voice <ko-KR-voice-name> \
  --rate +0%
```

`edge-tts`는 생성 시 인터넷 연결이 필요합니다. 만들어진 오디오와 PPTX 재생에는 네트워크가 필요하지 않습니다.

### 3. ElevenLabs

```bash
export ELEVENLABS_API_KEY="your-elevenlabs-api-key"
python3 skills/ppt-master/scripts/notes_to_audio.py <project_path> \
  --provider elevenlabs \
  --voice-id <voice-id> \
  --elevenlabs-model eleven_multilingual_v2
```

계정의 음성 목록:

```bash
python3 skills/ppt-master/scripts/notes_to_audio.py \
  --provider elevenlabs --list-voices
```

### 4. MiniMax

```bash
export MINIMAX_API_KEY="your-minimax-api-key"
python3 skills/ppt-master/scripts/notes_to_audio.py <project_path> \
  --provider minimax \
  --voice-id <voice-id> \
  --minimax-model speech-2.8-hd
```

해외 엔드포인트가 필요하면 현재 공급자 문서와 `.env.example`의 설정을 확인하세요.

### 5. Qwen TTS

```bash
export DASHSCOPE_API_KEY="your-dashscope-api-key"
python3 skills/ppt-master/scripts/notes_to_audio.py <project_path> \
  --provider qwen \
  --voice-id <voice-id> \
  --qwen-model qwen3-tts-flash \
  --qwen-language-type Korean
```

공급자가 요구하는 언어 값은 모델 버전에 따라 다를 수 있으므로 스크립트 `--help`와 공급자 문서를 확인하세요.

### 6. CosyVoice

```bash
export COSYVOICE_API_KEY="your-dashscope-api-key"
python3 skills/ppt-master/scripts/notes_to_audio.py <project_path> \
  --provider cosyvoice \
  --voice-id <voice-id> \
  --cosyvoice-model cosyvoice-v3-flash
```

### 7. 오디오 삽입 재내보내기

```bash
python3 skills/ppt-master/scripts/svg_to_pptx.py <project_path> \
  --recorded-narration audio
```

부분 테스트:

```bash
python3 skills/ppt-master/scripts/svg_to_pptx.py <project_path> \
  --narration-audio-dir audio
```

## 복제 음성 사용

ElevenLabs, MiniMax, Qwen, CosyVoice 등의 일부 공급자는 사용 권한이 있는 음성 샘플로 음성을 복제하고 `voice_id`를 발급합니다. PPT Master는 복제 자체를 수행하기보다 발급된 `voice_id`를 사용해 슬라이드별 음성을 생성합니다.

```text
MiniMax에서 발급받은 voice_id로 한국어 내레이션을 생성해 줘.
voice_id는 <id>이고 PPTX에 삽입해 줘.
```

주의:

- 본인 소유이거나 명시적 허가를 받은 음성만 사용
- 타인 사칭과 기만적 사용 금지
- 공급자의 약관과 지역 법률 확인
- 민감한 음성 샘플의 보관·전송 정책 확인

## 노트 작성 요령

### 슬라이드의 보이는 글을 그대로 읽지 않기

노트는 화면을 보완해야 합니다. 핵심 메시지의 배경, 수치의 의미, 전환 문장을 추가하세요.

### 문장을 짧게 유지

한국어 TTS는 긴 복문과 괄호가 많으면 억양이 불안정해질 수 있습니다. 한 문장에 하나의 핵심을 두는 편이 좋습니다.

### 숫자와 단위 명확히 쓰기

```text
2026년 2분기 매출은 1,250억 원입니다.
전년 동기 대비 12.4퍼센트 증가했습니다.
```

약어는 처음에 풀어 쓰거나 실제로 읽히길 원하는 형태로 적습니다.

### 외래어 샘플 확인

한국어 문장에 영문 제품명이나 기술 약어가 많으면 한 슬라이드를 먼저 생성해 발음을 확인하세요.

## 말하기 속도

기본 노트가 슬라이드당 2~5문장이라면 `+0%`에서 시작하는 것이 안전합니다.

- 내용이 조밀하고 설명형: `-5%` 전후 검토
- 짧은 홍보 문구: `+5%` 전후 검토
- 숫자·고유명사가 많음: 속도보다 문장 분리 우선

실제 음성마다 체감 속도가 다르므로 한 슬라이드 샘플을 먼저 듣는 것이 좋습니다.

## 특정 슬라이드 수정

1. `notes/<page>.md` 수정
2. 오디오 생성 스크립트 재실행
3. PPTX 오디오 삽입 재실행
4. 수정된 슬라이드의 전환 시간 확인

스크립트가 모든 오디오를 덮어쓸 수 있으므로 수동 편집한 파일은 별도 백업하세요.

## 동영상 내보내기

오디오가 삽입된 PPTX를 PowerPoint에서 엽니다.

1. **파일 → 내보내기 → 비디오 만들기**
2. 해상도 선택
3. **기록된 시간 및 설명 사용** 선택
4. MP4로 저장

`--recorded-narration`이 오디오 길이에 맞춘 자동 전환 시간을 기록하므로 별도 녹음 세션 없이 내보낼 수 있습니다.

## 애니메이션과의 관계

동영상·자동 재생에서는 클릭 시 시작하는 개체 애니메이션의 타이밍을 자동으로 만들기 어렵습니다. 내레이션 덱은 자동 시작 또는 슬라이드 시작 기준 애니메이션을 권장합니다.

자세한 내용은 [애니메이션](./animations.md)을 참고하세요.

## 문제 해결

### 음성이 생성되지 않음

- 인터넷 연결 확인
- `edge-tts` 설치 확인
- API 키와 공급자 이름 확인
- 음성 ID가 현재 계정에 존재하는지 확인
- 노트 파일이 비어 있지 않은지 확인

### 한국어 발음이 부자연스러움

- `ko-KR` 음성 사용
- 긴 문장을 나누기
- 숫자·단위를 읽히는 형태로 작성
- 약어를 풀어 쓰기
- 다른 음성 또는 다국어 모델로 한 슬라이드 비교

### PPTX에서 오디오가 재생되지 않음

- 오디오 파일과 슬라이드 파일명의 대응 확인
- 지원되는 MP3/M4A/WAV인지 확인
- `exports/`의 재내보내기 파일을 열었는지 확인
- PowerPoint의 미디어 호환성 최적화 실행

### 자동 전환 시간이 맞지 않음

오디오를 수정한 뒤 PPTX 삽입 단계를 다시 실행해야 합니다. 기존 PPTX에는 이전 오디오 길이의 타이밍이 남아 있을 수 있습니다.
