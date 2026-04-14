# hwpx 스킬 설치·활용 가이드

> HWPX 문서를 AI 에이전트에서 생성·읽기·편집·복원할 때 쓰는 스킬 요약

---

## 개요

`hwpx` 스킬은 한글(Hancom) `HWPX` 문서를 직접 다루기 위한 전문 스킬이다.  
핵심은 `python-hwpx`를 단순 호출하는 수준이 아니라, 필요할 때는 `HWPX = ZIP 기반 XML 컨테이너`로 보고 구조를 직접 복원하거나 재조립하는 점이다.

주요 역할:
- 기존 `HWPX` 문서 읽기
- 레퍼런스 문서 구조 분석
- 내용만 교체한 새 `HWPX` 생성
- 텍스트 추출
- XML 구조 검증
- 페이지 수 드리프트 검사

한 줄로 정리하면:
- **`hwpx` = 한글 문서 자체를 다루는 스킬**

---

## 설치 방법

### 1. skills.sh 전역 설치

가장 간단한 방법은 `skills.sh`를 사용하는 것이다.

```bash
npx skills add Canine89/hwpxskill --skill hwpx -a '*' -g -y --full-depth
```

의미:
- `--skill hwpx`: 저장소 안의 `hwpx` 스킬만 설치
- `-a '*'`: 지원되는 전체 에이전트 대상
- `-g`: 전역 설치
- `-y`: 확인 생략

### 2. 수동 설치

`skills.sh`를 쓰지 않을 때는 스킬 폴더를 에이전트별 디렉터리에 복사하면 된다.

```bash
git clone https://github.com/Canine89/hwpxskill.git
```

Claude Code:

```bash
cp -r hwpxskill ~/.claude/skills/hwpxskill
```

Codex CLI:

```bash
cp -r hwpxskill ~/.agents/skills/hwpxskill
```

Cursor:

```bash
cp -r hwpxskill ~/.cursor/skills/hwpxskill
```

---

## 기본 요구사항

- Python 3.6+
- `lxml`
- 가상환경 권장

예시:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install lxml
```

---

## 어떻게 쓰는가

### 1. 기존 문서를 기준으로 복원/수정

이 스킬의 핵심 사용법이다.  
기존 `HWPX`를 분석해서 스타일, 표 구조, 셀 병합, 여백, 문단 흐름을 최대한 유지한 채 내용만 바꾼다.

```bash
python3 scripts/analyze_template.py reference.hwpx \
  --extract-header /tmp/ref_header.xml \
  --extract-section /tmp/ref_section.xml

python3 scripts/build_hwpx.py \
  --header /tmp/ref_header.xml \
  --section /tmp/new_section0.xml \
  --output result.hwpx

python3 scripts/validate.py result.hwpx
python3 scripts/page_guard.py --reference reference.hwpx --output result.hwpx
```

### 2. 새 문서 만들기

원본 문서가 없으면 내장 템플릿으로 새 `HWPX`를 만든다.

```bash
python3 scripts/build_hwpx.py --template gonmun --output result.hwpx
```

대표 템플릿:
- `base`
- `gonmun`
- `report`
- `minutes`
- `proposal`

### 3. 텍스트 추출

문서를 Markdown이나 plain text로 뽑을 수 있다.

```bash
python3 scripts/text_extract.py document.hwpx --format markdown
```

### 4. 구조 검증

문서 생성 후 무결성 검사에 사용한다.

```bash
python3 scripts/validate.py result.hwpx
```

---

## 이 스킬이 잘 맞는 상황

- 기존 `HWPX` 양식을 유지하면서 내용만 바꾸고 싶을 때
- 공문, 보고서, 회의록 같은 문서를 자동 생성할 때
- 표 구조와 페이지 수가 중요한 문서를 다룰 때
- `HWPX`를 Markdown 추출이 아니라 **문서 자체로 유지**해야 할 때

---

## 주의할 점

- 이 스킬은 `md 변환기`가 아니다
- 주 역할은 `HWPX` 생성·편집·복원이다
- 단순히 텍스트만 추출하려면 `text_extract.py`만 써도 되지만, 스킬의 진짜 강점은 레퍼런스 복원 워크플로우에 있다

---

## 관련 스크립트

| 스크립트 | 역할 |
|---------|------|
| `analyze_template.py` | 레퍼런스 HWPX 분석 |
| `build_hwpx.py` | XML/템플릿 기반 HWPX 생성 |
| `text_extract.py` | 텍스트 추출 |
| `validate.py` | HWPX 구조 검증 |
| `page_guard.py` | 페이지 수 드리프트 검사 |

---

## 참고

- GitHub: https://github.com/Canine89/hwpxskill
- 로컬 스킬: `/Users/nettem/.agents/skills/hwpx/SKILL.md`

