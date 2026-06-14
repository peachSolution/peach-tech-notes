# Claude Code Playground 플러그인 가이드

> 작성일: 2026-06-14
> 최종 수정: 2026-06-14
> 초점: **md 문서를 HTML 설명 페이지·다이어그램으로 시각화**하는 용도 중심

---

## 목차

1. [개요](#1-개요)
2. [동작 방식 — 3패널 레이아웃](#2-동작-방식--3패널-레이아웃)
3. [제공 템플릿](#3-제공-템플릿)
4. [설치 방법](#4-설치-방법)
5. [기본 사용법](#5-기본-사용법)
6. [개발팀 실무 프롬프트 예시 모음](#6-개발팀-실무-프롬프트-예시-모음)
7. [참고: 이름이 비슷한 다른 것들](#7-참고-이름이-비슷한-다른-것들)
8. [출처](#8-출처)

---

## 1. 개요

**Claude Code Playground**는 Anthropic 공식 마켓플레이스(`claude-plugins-official`)에서 제공하는 플러그인입니다.

텍스트만으로는 한눈에 들어오지 않는 **구조적·시각적 정보**를 자체 포함된 HTML 화면으로 풀어줍니다. 이 문서는 그중에서도 **md 문서를 보기 쉬운 HTML 설명 페이지나 다이어그램으로 변환**하는 용도에 초점을 둡니다.

동작 핵심:

1. `/playground:playground` 스킬에 **무엇을 시각화할지** 요청한다.
2. Claude가 **단일 HTML 파일**을 생성한다 (CSS·JS 내장, 외부 의존성 없음).
3. `open 파일명.html` 로 브라우저가 자동 실행되어 설명 화면·다이어그램을 확인한다.

---

## 2. 동작 방식 — 3패널 레이아웃

| 위치 | 패널 | 설명 |
|------|------|------|
| 좌측 | 대화형 컨트롤 | 표시 범위·레이어·필터 토글 등 |
| 중앙/우측 | 실시간 미리보기 | 설명 화면·다이어그램이 즉시 갱신 |
| 하단 | 프롬프트 출력 | 복사 버튼으로 Claude에 재투입 |

### 언제 쓰나

- 긴 md 문서를 **계층·흐름이 보이는 설명 페이지**로 만들고 싶을 때
- 아키텍처·데이터 흐름·개념 관계를 **다이어그램**으로 표현하고 싶을 때
- 순수 텍스트로는 구조가 한눈에 안 들어올 때

---

## 3. 제공 템플릿

설명 페이지·다이어그램 용도와 직접 맞닿는 템플릿은 **code-map**과 **concept-map**입니다.

| 템플릿 | 용도 | 설명 페이지·다이어그램 적합도 |
|--------|------|------|
| **code-map** | 아키텍처/구성요소 관계, 데이터 흐름, 레이어 다이어그램 | ★ 가장 적합 |
| **concept-map** | 개념·지식 관계, 범위 매핑, 학습 격차 | ★ 적합 |
| **document-critique** | 문서를 문단별로 검토 (승인/거절/댓글) | △ 문서 리뷰용 |
| **design-playground** | 시각 디자인 결정 (간격·색상·타이포) | — |
| **data-explorer** | SQL·API·정규식 쿼리 작성 | — |
| **diff-review** | 코드 diff 라인별 리뷰 | — |

---

## 4. 설치 방법

공식 마켓플레이스(`claude-plugins-official`)에서 설치한다.

### 방법 A — UI에서 토글

`/plugin` 실행 → **Discover** 탭 → `playground` 검색 → 스페이스바로 토글하여 활성화.

### 방법 B — 명령어로 설치

```
# 1. 마켓플레이스 등록
/plugin marketplace add anthropics/claude-plugins-official

# 2. 플러그인 설치
/plugin install playground

# 3. 적용
/reload-plugins
```

---

## 5. 기본 사용법

스킬을 명시해 호출하면 가장 확실하다. 모든 요청은 다음 한 줄 뒤에 **무엇을 시각화할지**를 붙이는 형태다.

```
/playground:playground <무엇을 어떤 화면/다이어그램으로 만들지>
```

### 발동 후 흐름

1. Claude가 주제에 맞는 **단일 HTML 파일**을 생성한다 (외부 의존성 없음).
2. `open 파일명.html` 로 브라우저가 자동 실행된다.
3. 좌측 컨트롤로 표시 범위·레이어를 조정하며 설명 화면·다이어그램을 확인한다.
4. 필요하면 하단 프롬프트를 복사해 실제 문서·코드 작업으로 연결한다.

> 팁: `html로 보기쉽게 만들어줘`처럼 포괄적으로만 요청하면 화면 구성이 모호해진다.
> **다룰 대상(어떤 md/구조)** 과 **표현 형태(설명 페이지/레이어 다이어그램/개념도)** 를 함께 적자.

---

## 6. 개발팀 실무 프롬프트 예시 모음

실제 개발 현장에서 바로 복사해 쓸 수 있도록 상황별로 정리했다.
**대상 파일 경로·테이블명·엔드포인트만 바꾸면** 그대로 동작한다. 각 예시 끝에 발동 템플릿과 **언제 쓰면 좋은지**를 붙였다.

> 프롬프트 첫 줄은 항상 `/playground:playground` — 스킬을 명시하면 가장 확실히 발동된다.

---

### A. 문서를 설명 페이지로 (code-map / concept-map)

장황한 md를 신규 멤버도 5분 만에 이해하는 화면으로.

```
/playground:playground
README.md를 섹션별 접이식 목차 + 진행도 표시가 있는 온보딩 설명 페이지로 만들어줘
```
→ concept-map · **신규 입사자 온보딩 문서로**

```
/playground:playground
이 설치 가이드 md를 1·2·3단계 체크박스로 따라갈 수 있는 단계별 셋업 페이지로 만들어줘
```
→ concept-map · **로컬 환경 셋업 가이드를 따라하기 쉽게**

```
/playground:playground
docs/wiki/ 의 이 기능 설명 md를 용어 클릭 시 정의가 펼쳐지는 설명 페이지로 만들어줘
```
→ concept-map · **도메인 용어가 많은 기능을 팀에 공유할 때**

---

### B. 아키텍처·구조 다이어그램 (code-map)

말로 설명하기 힘든 시스템 구조를 그림으로.

```
/playground:playground
이 아키텍처 md를 프론트→API→Store→DB 레이어가 위아래로 쌓인 다이어그램으로 만들어줘.
레이어를 토글하면 해당 계층만 강조되게 해줘
```
→ code-map · **시스템 전체 구조를 신규 멤버/고객에게 브리핑할 때**

```
/playground:playground
peach-harness 디렉터리 구조를 접고 펼 수 있는 폴더 트리 다이어그램으로 시각화해줘
```
→ code-map · **레포 구조가 복잡해 길을 잃기 쉬울 때**

```
/playground:playground
이 모듈들의 import 의존 관계를 화살표로 연결한 컴포넌트 관계도로 만들어줘.
순환 의존이 있으면 빨갛게 표시해줘
```
→ code-map · **리팩터링 전 결합도/순환 의존을 눈으로 확인할 때**

```
/playground:playground
이 마이크로서비스 간 호출 관계 md를 서비스 노드 + 호출 화살표 다이어그램으로 만들어줘
```
→ code-map · **서비스 경계와 호출 흐름을 합의할 때**

---

### C. 데이터·프로세스 흐름도 (code-map)

요청이 어디로 흐르는지, 상태가 어떻게 바뀌는지.

```
/playground:playground
인증 흐름(로그인→토큰 발급→갱신→만료→로그아웃)을 단계별 플로우 다이어그램으로 만들어줘.
각 단계 클릭 시 어떤 API가 호출되는지 보이게 해줘
```
→ code-map · **인증/세션 로직을 팀에 설명하거나 버그 원인을 추적할 때**

```
/playground:playground
이 주문 상태 머신(장바구니→결제대기→결제완료→배송→취소)을 상태 전이 다이어그램으로 만들어줘
```
→ code-map · **상태값과 전이 규칙을 기획·QA와 합의할 때**

```
/playground:playground
이 배치 작업 md를 입력→검증→변환→저장 단계의 데이터 흐름 다이어그램으로 만들어줘
```
→ code-map · **배치/ETL 파이프라인을 문서화할 때**

---

### D. DB·쿼리 탐색 (data-explorer)

스키마와 쿼리를 조건 토글로 실험.

```
/playground:playground
이 ERD md(테이블·관계)를 테이블 클릭 시 컬럼·FK가 펼쳐지는 스키마 탐색 페이지로 만들어줘
```
→ data-explorer · **DB 설계를 리뷰하거나 신규 멤버에게 스키마를 설명할 때**

```
/playground:playground
orders 테이블에서 기간·상태·금액 범위를 토글하면 SELECT 쿼리가 실시간 생성되는
SQL 빌더 playground 만들어줘
```
→ data-explorer · **복잡한 조회 쿼리를 만들기 전에 조건을 시각적으로 조립할 때**

```
/playground:playground
이 REST API의 쿼리 파라미터(page·size·sort·filter)를 토글하면
요청 URL과 예상 응답 JSON 형태가 갱신되는 playground 만들어줘
```
→ data-explorer · **API 페이징/필터 스펙을 프론트와 합의할 때**

```
/playground:playground
정규식을 입력하면 샘플 로그 텍스트에서 매칭 부분을 즉시 강조해주는 정규식 테스터 만들어줘
```
→ data-explorer · **로그 파싱/검증 정규식을 만들 때**

---

### E. 디자인·UI 결정 (design-playground)

코드 작업 전에 시각 값을 먼저 합의.

```
/playground:playground
공통 버튼 컴포넌트의 모서리 둥글기·그림자·배경색·글자색·패딩을 조정하며
미리보는 playground 만들어줘. 결과를 CSS 변수로 복사할 수 있게 해줘
```
→ design-playground · **디자인 시스템 토큰 값을 팀이 함께 정할 때**

```
/playground:playground
다크/라이트 테마 색상 팔레트를 조합해보고 텍스트 대비(WCAG)를 체크하는 playground 만들어줘
```
→ design-playground · **접근성 기준에 맞는 색상 조합을 고를 때**

```
/playground:playground
카드 리스트 레이아웃의 컬럼 수·간격·반응형 분기점을 조정하며 보는 playground 만들어줘
```
→ design-playground · **반응형 그리드 설계를 확정할 때**

---

### F. 리뷰·검토 (diff-review / document-critique)

변경과 문서를 라인별로 짚어가며.

```
/playground:playground
이번 PR의 변경 diff를 파일별·라인별로 코멘트를 달며 리뷰할 수 있는 playground 만들어줘
```
→ diff-review · **PR 리뷰 포인트를 정리해 코멘트로 옮길 때**

```
/playground:playground
이 기능 명세 초안 md를 문단별로 승인/거절/코멘트하며 검토하는 playground 만들어줘
```
→ document-critique · **Spec/기획서를 팀이 함께 리뷰·확정할 때**

```
/playground:playground
이 API 변경 제안서 md를 항목별로 찬반 의견을 달며 검토하는 페이지로 만들어줘
```
→ document-critique · **호환성 깨지는 API 변경을 합의할 때**

---

### 공통 패턴

좋은 프롬프트는 세 조각으로 이루어진다.

```
[① 대상]            이 ERD md / 이번 PR diff / orders 테이블
[② 표현 형태]        스키마 탐색 페이지 / 라인별 리뷰 / SQL 빌더 / 레이어 다이어그램
[③ 인터랙션(선택)]   클릭 시 펼치기 / 토글로 강조 / 결과 복사 / 순환 의존 빨갛게
```

> ②와 ③이 구체적일수록 컨트롤과 미리보기가 정확히 구성된다.
> 반대로 `html로 보기쉽게 만들어줘`처럼 ②가 빠지면 화면 구성이 모호해진다.

---

## 7. 참고: 이름이 비슷한 다른 것들

검색 시 같은 "Claude Code Playground" 이름으로 혼동되는 항목이 있으니 구분한다.

- **KodeKloud Claude Code Playground** — 실습용 샌드박스 환경 (본 플러그인과 무관)
- **skillsplayground.com** — 스킬·MCP 디렉터리 사이트
- **Shortcuts Playground** — macOS/iOS 단축어 제작용 서드파티 플러그인

---

## 8. 출처

- [anthropics/claude-plugins-official — playground](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/playground)
- [Geeky Gadgets — Claude Code Playground Plugin Guide](https://www.geeky-gadgets.com/claude-code-playground-plugin/)
- [Paddo.dev — Playground: When Text Prompting Isn't Enough](https://paddo.dev/blog/playground-plugin-visual-configuration/)
- [Nathan Onn — How the Claude Code Playground Skill Saved My Sanity](https://www.nathanonn.com/claude-code-playground-skill-visual-design-workflow/)
