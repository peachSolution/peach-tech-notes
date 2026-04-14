# MarkItDown 소개 정리

> Microsoft의 문서·파일 → Markdown 변환 도구 요약

---

## 개요

`MarkItDown`은 Microsoft가 공개한 Python 기반 도구로, 다양한 문서와 파일을 **Markdown으로 변환**하는 데 초점을 둔다.

핵심 방향:
- 사람용 고충실도 문서 재현보다
- **LLM 입력, 검색, 요약, RAG 전처리**에 적합한 Markdown 출력

공식 README도 이 점을 분명히 말한다.
- 구조 보존은 중요하게 보지만
- 사람에게 보여주기 위한 완벽한 원본 재현용 변환기는 아니라는 뜻이다

한 줄로 정리하면:
- **`MarkItDown` = 여러 파일을 Markdown으로 바꾸는 범용 변환기**

---

## 지원 포맷

공식 README 기준 주요 지원 대상:

- PDF
- PowerPoint
- Word
- Excel
- 이미지
- 오디오
- HTML
- CSV, JSON, XML 같은 텍스트 기반 포맷
- ZIP
- YouTube URL
- EPUB

즉, `PDF 전용` 도구라기보다 **범용 파일 -> Markdown** 도구에 가깝다.

---

## 왜 Markdown인가

공식 설명 기준 장점은 두 가지다.

1. 문서 구조를 어느 정도 유지할 수 있다
- 제목
- 목록
- 표
- 링크

2. LLM 친화적이다
- Markdown은 텍스트 분석에 유리하고
- 토큰 효율도 좋은 편이다

그래서 용도는 대체로 이쪽이다.
- 문서 요약
- 검색 인덱싱
- RAG 입력
- 에이전트 컨텍스트 준비

---

## 설치

공식 요구사항:
- Python 3.10+

권장 설치:

```bash
python -m venv .venv
source .venv/bin/activate
pip install 'markitdown[all]'
```

선택 설치도 가능하다.

```bash
pip install 'markitdown[pdf,docx,pptx]'
```

의미:
- `all`: 모든 선택 의존성 설치
- 필요한 포맷만 골라 설치 가능

---

## CLI 사용법

기본 사용:

```bash
markitdown path-to-file.pdf > document.md
```

출력 파일 지정:

```bash
markitdown path-to-file.pdf -o document.md
```

파이프 입력:

```bash
cat path-to-file.pdf | markitdown
```

실무 예시:

```bash
markitdown proposal.docx -o proposal.md
markitdown report.pdf -o report.md
markitdown slides.pptx -o slides.md
```

---

## Python API

기본 사용:

```python
from markitdown import MarkItDown

md = MarkItDown(enable_plugins=False)
result = md.convert("test.xlsx")
print(result.text_content)
```

즉, CLI 도구이면서 Python 라이브러리로도 붙일 수 있다.

---

## 확장 기능

### 1. MCP 서버

공식 저장소에는 `markitdown-mcp`가 포함되어 있다.  
LLM 앱과 연결해서 Markdown 변환 도구처럼 쓸 수 있다.

### 2. 플러그인

플러그인 시스템을 지원한다.

예:
- `markitdown-ocr`

OCR 플러그인을 쓰면 PDF, DOCX, PPTX, XLSX 안의 이미지 텍스트도 추가로 읽을 수 있다.

### 3. Azure Document Intelligence

PDF 변환에서 Microsoft Document Intelligence를 사용할 수 있다.

---

## 장점

- 여러 포맷을 하나의 인터페이스로 처리
- CLI와 Python API 둘 다 제공
- Markdown 구조 보존에 초점
- LLM/RAG 전처리에 적합
- MCP/플러그인 확장 가능

---

## 한계

- 사람용 원본 문서 재현 도구는 아니다
- 복잡한 레이아웃을 완벽하게 복원하는 용도로 기대하면 안 된다
- `문서 -> PDF 변환기`가 아니다
- `HWPX`는 공식 지원 포맷 목록에 없다

즉:
- `MarkItDown`은 **문서 읽기/추출** 쪽
- `LibreOffice`, `Gotenberg`는 **문서 -> PDF** 쪽
- `hwpx` 스킬은 **한글 문서 자체 조작** 쪽이다

---

## 어디에 잘 맞는가

- PDF/Office 문서를 AI가 읽기 좋은 형태로 바꾸고 싶을 때
- RAG용 원문 전처리를 할 때
- 문서 묶음을 일괄로 Markdown화할 때
- Codex CLI, Claude Code 같은 환경에서 로컬 CLI로 붙일 때

---

## 참고

- 공식 GitHub: https://github.com/microsoft/markitdown
- MCP 패키지: https://github.com/microsoft/markitdown/tree/main/packages/markitdown-mcp
- 현재 로컬 설치 명령 예시: `pip install 'markitdown[all]'`

