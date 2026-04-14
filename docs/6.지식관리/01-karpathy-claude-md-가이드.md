# Karpathy 영감 CLAUDE.md 가이드

> 출처: [GeekNews - 65줄 마크다운이 AI 코딩을 변화시키다](https://news.hada.io/topic?id=26655)
> 원문 블로그: [tildeweb.nl - 65 lines of Markdown](https://tildeweb.nl/~michiel/65-lines-of-markdown-a-claude-code-sensation.html)
> GitHub 저장소: [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills)

---

## 배경

Andrej Karpathy(전 OpenAI, Tesla)가 LLM 코딩의 고질적 문제를 지적한 것에서 영감을 받아 Michiel Beijen이 65줄 짜리 CLAUDE.md 파일을 작성했다. 이 파일은 GitHub에서 하루 만에 스타 400개, 총 4,000개 근접이라는 폭발적 반응을 얻었다.

### Karpathy가 지적한 LLM 코딩의 문제점

> "The models make wrong assumptions on your behalf and just run along with them without checking.
> They don't manage their confusion, don't seek clarifications, don't surface inconsistencies,
> don't present tradeoffs, don't push back when they should."

> "They really like to overcomplicate code and APIs, bloat abstractions, don't clean up dead code...
> implement a bloated construction over 1000 lines when 100 would do."

> "They still sometimes change/remove comments and code they don't sufficiently understand
> as side effects, even if orthogonal to the task."

**핵심 문제 3가지:**
1. **잘못된 가정** - 물어보지 않고 혼자 해석하고 진행
2. **과도한 복잡화** - 100줄이면 될 것을 1000줄로 구현
3. **의도하지 않은 변경** - 요청과 무관한 코드까지 수정

---

## 4가지 핵심 원칙

### 1. Think Before Coding (코딩 전에 먼저 생각하라)

**문제:** LLM은 불확실한 상황에서도 조용히 가정을 세우고 진행한다.

**규칙:**
- 가정을 명시적으로 드러낼 것
- 혼란스럽거나 불확실한 부분을 숨기지 말 것
- 여러 해석이 가능하면 각각 제시하고 확인을 구할 것
- 구현 전에 불확실한 사항을 질문할 것

**프롬프트 예시:**
```
✅ 좋은 예시:
"이 함수를 수정하기 전에 확인이 필요합니다.
현재 users 배열이 undefined일 수 있는지 확인해주세요.
두 가지 해석이 가능합니다:
A) users가 항상 존재한다고 가정하고 직접 접근
B) null/undefined 체크 후 접근
어떤 방식으로 처리할까요?"

❌ 나쁜 예시:
(아무 질문 없이 null 체크를 추가하거나 제거하고 진행)
```

---

### 2. Simplicity First (단순함이 먼저다)

**문제:** LLM은 요청하지 않은 기능, 추상화, 에러 처리를 추가하는 경향이 있다.

**규칙:**
- 문제를 해결하는 최소한의 코드만 작성
- 요청받지 않은 기능 추가 금지
- 단일 사용 코드에는 추상화 레이어 만들지 않기
- 코드가 더 짧게 가능하다면 다시 작성
- "나중에 필요할 수도 있는" 코드는 작성하지 않기

**프롬프트 예시:**
```
✅ 좋은 예시:
"이 버튼에 클릭 이벤트만 추가해주세요."
→ 버튼에 onClick 핸들러 하나만 추가

❌ 나쁜 예시:
"이 버튼에 클릭 이벤트만 추가해주세요."
→ 클릭 핸들러 + 로딩 상태 + 에러 처리 + 로그 시스템 + 재시도 로직 추가
```

**구체적 금지 사항:**
```
- 요청하지 않은 에러 처리 추가
- 미래를 위한 추상화 레이어 생성
- "편의를 위한" 유틸리티 함수 만들기
- 요청 범위 밖의 코드 리팩토링
```

---

### 3. Surgical Changes (외과적 수정)

**문제:** LLM은 관련 없는 코드까지 수정하는 경향이 있다.

**규칙:**
- 요청된 부분만 정밀하게 수정
- 인접 코드 리팩토링하지 않기
- 기존 코드 스타일과 구조 보존
- **자신이 만든 변경으로 인해 불필요해진 코드만** 제거
- 이해하지 못한 코드는 수정하지 않기

**프롬프트 예시:**
```
✅ 좋은 예시:
"calculateTotal 함수에서 세금 계산 로직만 수정해주세요."
→ 해당 함수의 세금 관련 라인만 수정

❌ 나쁜 예시:
"calculateTotal 함수에서 세금 계산 로직만 수정해주세요."
→ 세금 로직 수정 + 함수명 변경 + 변수명 리네이밍 + 다른 함수들도 리팩토링
```

**외과적 수정 체크리스트:**
```
[ ] 요청된 부분 외 다른 코드를 건드렸는가?
[ ] 기존 주석을 수정하거나 삭제했는가?
[ ] 요청과 무관한 변수명을 변경했는가?
[ ] 요청받지 않은 파일을 수정했는가?
→ 위 항목 중 하나라도 해당하면 롤백
```

---

### 4. Goal-Driven Execution (목표 중심 실행)

**문제:** 명령형 지시(어떻게 하라)는 LLM이 다른 방향으로 흘러가기 쉽다.

**규칙:**
- 요청을 측정 가능한 성공 기준으로 변환
- 다단계 작업은 검증 포인트가 있는 단계로 분리
- 명령형(imperative) → 선언형(declarative) 목표로 변환
- 독립적 검증이 가능하게 작업 구성

**프롬프트 변환 예시:**

```
❌ 명령형 (나쁜 예시):
"사용자 인증 기능을 구현해줘.
JWT 토큰을 사용하고, 로그인/로그아웃을 만들어줘."

✅ 선언형 (좋은 예시):
"다음 성공 기준을 달성하는 사용자 인증을 구현해줘:
1. POST /auth/login 요청 시 유효한 JWT 토큰 반환
2. Authorization 헤더에 토큰 포함 시 보호된 엔드포인트 접근 가능
3. POST /auth/logout 요청 시 토큰 무효화
4. 만료된 토큰으로 요청 시 401 반환

각 단계를 구현한 후 해당 기준이 충족됐는지 확인해줘."
```

**목표 정의 템플릿:**
```
"다음 조건을 만족하는 [기능]을 구현해줘:
1. [검증 가능한 조건 1]
2. [검증 가능한 조건 2]
3. [검증 가능한 조건 3]

각 단계 완료 후 조건 충족 여부를 확인하고 다음 단계로 진행해줘."
```

---

## CLAUDE.md 설치 및 적용 방법

### 방법 1: Claude Code 플러그인으로 설치
```bash
# 플러그인 마켓플레이스에서 추가
/plugin marketplace add forrestchang/andrej-karpathy-skills

# 플러그인 설치
/plugin install andrej-karpathy-skills@karpathy-skills
```

### 방법 2: 프로젝트에 직접 적용
```bash
# 저장소 클론 후 CLAUDE.md를 프로젝트에 복사
curl -o CLAUDE.md https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md

# 또는 기존 CLAUDE.md에 내용 추가
cat karpathy-guidelines.md >> CLAUDE.md
```

### 방법 3: 프로젝트별 적용 (추천)
기존 프로젝트의 CLAUDE.md에 다음 섹션을 추가:

```markdown
## Karpathy 코딩 원칙

### 코딩 전 생각하기
- 가정을 명시하고, 불확실하면 질문하라
- 여러 해석이 가능할 때는 선택지를 제시하라

### 단순함 우선
- 요청한 것만 구현하라
- 불필요한 추상화, 에러 처리, 기능 추가 금지

### 외과적 수정
- 요청된 부분만 수정하라
- 무관한 코드, 주석, 변수명 변경 금지

### 목표 중심 실행
- 성공 기준을 정의하고 검증하며 진행하라
```

---

## 실제 효과

사용자들이 보고한 변화:
- **과도한 창의성 감소** - 요청하지 않은 기능 추가가 줄어듦
- **불필요한 리팩토링 감소** - 관련 없는 코드를 건드리는 일이 줄어듦
- **더 명확한 커뮤니케이션** - AI가 불확실할 때 더 자주 질문함
- **코드 품질 향상** - 더 간결하고 목적에 맞는 코드 생성

> 저자 Michiel Beijen: "진짜 효과가 있는지 100% 확신할 수는 없지만,
> 많은 개발자들이 실질적인 변화를 느끼고 있다."

---

## 이 프로젝트(peach-ai-dev)와의 연관성

현재 프로젝트의 CLAUDE.md와 글로벌 `~/.claude/CLAUDE.md`는 이미 Karpathy 원칙과 유사한 규칙을 포함하고 있다:

| Karpathy 원칙 | peach-ai-dev 대응 규칙 |
|--------------|----------------------|
| Think Before Coding | "추측 금지, 테스트/문서로 검증" |
| Simplicity First | "심플하고 오버엔지니어링 반대", "요청된 것만 구현: scope 엄수" |
| Surgical Changes | "TODO 주석 금지", "요청된 것만 구현" |
| Goal-Driven Execution | "증거 기반: 추측 금지, 테스트/문서로 검증" |

**개선 가능한 영역:**
- "Think Before Coding" 원칙을 더 명시적으로 강화
- 목표 기반 프롬프트 작성 패턴을 스킬/가이드로 문서화

---

## 참고 자료

- [GeekNews 원문](https://news.hada.io/topic?id=26655)
- [tildeweb.nl 블로그](https://tildeweb.nl/~michiel/65-lines-of-markdown-a-claude-code-sensation.html)
- [GitHub 저장소](https://github.com/forrestchang/andrej-karpathy-skills)
- [Karpathy LLM 코딩 진화론](https://cc.deeptoai.com/docs/en/best-practices/karpathy-llm-coding-evolution)
