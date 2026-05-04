# qmd + Serena + peach-wiki 개발 프로젝트 활용 가이드

> 개발 프로젝트(Vue3/TypeScript/Bun 모노레포)에서 토큰 절감과 코드 정밀 분석을 위한 3개 도구 조합 가이드.

---

## 0. 운영 방침 (먼저 읽기)

> **기본은 qmd + peach-wiki만 사용한다. Serena는 필요할 때만 활성화한다.**

| 상황 | 권장 도구 |
|---|---|
| 일상 개발 (기능 추가·수정·디버깅) | **qmd + wiki** |
| 대규모 리팩토링, 전역 이름 변경, 심볼 참조 추적 | **+ Serena 활성화** |

**왜 Serena를 기본에서 빼는가:**
- Serena는 세션마다 `activate_project` 호출이 필요해 **운영 마찰**이 있다
- AI가 활성화를 빠뜨리면 **조용히 실패**하고 텍스트 검색으로 폴백한다
- qmd만으로 약 **68% 토큰 절감**을 달성하며, Serena 추가 시 **+10%p**에 그친다
- 즉, 일상 작업에선 마찰 비용 > 추가 절감 이익

**Serena를 켜야 할 때:**
- 함수·클래스 이름을 전체 코드베이스에 안전하게 변경
- "이 심볼 어디서 호출돼?" 정밀 추적 필요
- 인터페이스 구현체 추적, 타입 계층 분석
- 대규모 구조 변경 (파일·심볼 이동)

→ 이때만 Claude에게 **"{프로젝트명} Serena 활성화해줘"** 라고 요청

---

## 1. 도구 역할 분리

| 도구 | 역할 | 강점 |
|---|---|---|
| **qmd** | 로컬 벡터 DB 기반 시맨틱 검색 | 파일 위치 탐색, wiki 문서 통합 검색 |
| **Serena MCP** | LSP 기반 심볼 수준 코드 분석 | 함수 참조 추적, 리팩토링, 타입 계층 |
| **peach-wiki** | 코드베이스 지식 위키 관리 | 도메인 맥락 축적, AI 컨텍스트 압축 |

세 도구는 **대체 관계가 아니라 레이어 관계**다.

```
[넓은 탐색·위키 검색]  →  qmd
[심볼 수준 정밀 분석]  →  Serena
[도메인 맥락 보존]    →  peach-wiki
```

---

## 2. 토큰 절감 효과 (대규모 프로젝트 실측 기준)

규모 예시: 약 670개 파일, 87,000줄 규모의 Vue3/TypeScript 모노레포

| 방법 | 토큰 소비 | 절감율 |
|---|---|---|
| 도구 없음 (파일 직접 읽기) | ~9,400 토큰 | 기준 |
| qmd만 사용 | ~3,040 토큰 | **-68%** |
| qmd + Serena 조합 | ~2,040 토큰 | **-78%** |

**qmd가 절감의 주력(68%), Serena는 추가 10%p + 정확도 향상.**

### 실사용 권장 방침

- **기본: qmd만 사용** — 세션 활성화 마찰 없이 큰 절감 효과
- **Serena 추가 활용**: 리팩토링·전역 이름 변경 등 심볼 정밀 작업 시에만 활성화
  - 이유: Serena는 세션마다 `activate_project` 호출 필요 → 일상 작업엔 마찰

---

## 3. qmd 설치 및 설정

### 설치

```bash
npm install -g @tobilu/qmd
```

### 프로젝트 인덱스 등록

```bash
# 인덱스명 = 프로젝트 폴더명 (격리 목적)
QMD_INDEX=$(basename $(pwd))

# 컬렉션 등록
qmd --index "$QMD_INDEX" collection add . --name "$QMD_INDEX" \
  --mask "**/*.{ts,vue,md,sql,py,go,js}"

# 컨텍스트 설명 추가 (검색 품질 핵심)
qmd --index "$QMD_INDEX" context add "qmd://$QMD_INDEX/" "프로젝트 한 줄 설명"

# 인덱싱 + 임베딩
qmd --index "$QMD_INDEX" update && qmd --index "$QMD_INDEX" embed
```

> Apple Silicon에서 Metal 오류 시 `QMD_LLAMA_GPU=off qmd ...` 로 실행

### 주요 명령

```bash
# 시맨틱 검색 (권장)
qmd --index "$QMD_INDEX" query "키워드" -c "$QMD_INDEX"

# 파일 경로만 출력
qmd --index "$QMD_INDEX" query "키워드" -c "$QMD_INDEX" --files

# 키워드 검색
qmd --index "$QMD_INDEX" search "키워드" -c "$QMD_INDEX"

# 특정 파일 읽기
qmd --index "$QMD_INDEX" get "qmd://$QMD_INDEX/경로/파일.ts"

# 인덱스 갱신 (코드 변경 후)
qmd --index "$QMD_INDEX" update && qmd --index "$QMD_INDEX" embed
```

> **`--index` 없이 plain `qmd update/embed` 절대 금지** — 다른 프로젝트 인덱스 오염

---

## 4. Serena MCP 설치 및 구동 구조

### 설치 방법

Claude Code `/plugins` → Discover → `serena` 검색 → Space로 선택 → 설치  
(출처: `claude-plugins-official` 마켓플레이스)

### 구동 구조

```
Claude Code 시작
    ↓
uvx --from git+https://github.com/oraios/serena serena start-mcp-server
    ↓
Serena MCP 서버 자동 실행 (별도 설치·Docker 불필요)
    ↓
Claude Code 종료 시 자동 종료
```

- **항상 켜져있지 않음** — Claude Code 실행 시에만 동작
- **추가 설치 없음** — `uvx`가 GitHub에서 자동으로 받아 실행
- **대시보드** (`http://127.0.0.1:24282`) — 닫아도 기능에 영향 없음, 선택적 모니터링용

### 프로젝트 활성화

Serena는 세션당 하나의 프로젝트를 활성화해야 LSP가 동작한다.

```
방법 1: 해당 프로젝트 디렉토리에서 Claude Code 열기 (자동 감지)
방법 2: Claude에게 "{프로젝트명} 프로젝트 활성화해줘" 요청
```

### 주요 Serena 도구

| 도구 | 용도 |
|---|---|
| `find_symbol` | 함수·클래스·인터페이스 위치 탐색 |
| `find_referencing_symbols` | 특정 심볼을 참조하는 곳 전체 파악 |
| `symbol_overview` | 파일 아웃라인 조회 |
| `replace_symbol_body` | 심볼 본문만 교체 (파일 전체 쓰기 불필요) |
| `rename` | 전체 코드베이스 안전한 이름 변경 |
| `find_implementations` | 인터페이스 구현체 탐색 |

### 지원 언어

직접 지원(완전한 LSP): **TypeScript/JavaScript, Python, Java**  
간접 지원: Go, Rust, C#, Kotlin, Vue 등 40개 이상

---

## 5. peach-wiki 설정

### wiki 초기화

```bash
# 프로젝트 루트에서 실행
/peach:peach-wiki
```

위키는 `docs/wiki/` 하위에 생성된다.

```
docs/wiki/
├── WIKI-AGENTS.md     ← 운영 규칙
├── wiki-index.md      ← 전체 카탈로그
├── wiki-log.md        ← 작업 타임라인
├── concepts/          ← 아키텍처·도메인 개념
├── entities/          ← 모듈·서비스·컴포넌트
└── synthesis/         ← 데이터 흐름·의사결정
```

### AGENTS.md 연동 (필수)

peach-wiki 초기화 후 AGENTS.md에 아래 섹션이 자동 추가된다.  
없으면 수동으로 추가:

```markdown
## wiki 참조 (필수)
코드 생성·분석 전 아래 순서를 따른다.
1. `qmd --index {프로젝트명} query "키워드" -c {프로젝트명}` 으로 wiki + 소스 통합 검색
2. qmd 미설치 시 `docs/wiki/wiki-index.md` → 관련 페이지 직접 Read
```

---

## 6. 권장 워크플로우 (조합 사용)

### 코드 수정 요청 시

```
1. qmd query  →  관련 파일·위키 위치 파악 (토큰 절약)
2. Serena find_symbol  →  심볼 참조·타입 관계 정밀 파악
3. 필요한 파일만 최소 Read
4. 수정 후 qmd update  →  인덱스 갱신
```

### 대규모 리팩토링 시

```
1. qmd query  →  영향 범위 파일 목록 파악
2. Serena find_referencing_symbols  →  심볼 참조 전체 파악
3. Serena rename  →  전체 코드베이스 안전한 이름 변경
4. wiki DRIFT 업데이트  →  변경 반영
```

### 신규 기능 파악 시

```
1. qmd query  →  관련 wiki 문서 + 소스 위치 파악
2. wiki entities 읽기  →  도메인 맥락 파악
3. Serena symbol_overview  →  파일 구조 빠른 파악
```

---

## 7. 자주 묻는 질문

**Q. Serena가 항상 켜져 있나요?**  
A. Claude Code 실행 시에만 자동 시작, 종료 시 같이 종료됩니다.

**Q. Docker나 별도 설치가 필요한가요?**  
A. 불필요합니다. `uvx`가 자동으로 처리합니다.

**Q. 대시보드(127.0.0.1:24282)를 닫아도 되나요?**  
A. 닫아도 됩니다. 모니터링용이며 MCP 기능에 영향 없습니다.

**Q. qmd 없이 Serena만 써도 되나요?**  
A. 안 됩니다. Serena는 코드 심볼만 알고 wiki 문서는 모릅니다.  
qmd가 파일 위치 탐색과 wiki 검색을 담당하며, 두 도구는 대체 관계가 아닙니다.

**Q. qmd 인덱스를 언제 갱신해야 하나요?**  
A. 파일을 새로 추가·이동·대량 수정했을 때 `qmd --index {인덱스} update && embed` 실행.  
소규모 텍스트 수정은 `update`만으로 충분합니다.
