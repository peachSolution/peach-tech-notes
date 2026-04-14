# Figma MCP 통합 가이드

Figma와 Claude Code를 연동하는 두 가지 MCP 서버의 비교 분석 및 활용 가이드입니다.

---

## 1. MCP 비교 분석

### 1.1 핵심 차이점

| 항목 | FigmaDesktop (공식) | TalkToFigma (커뮤니티) |
|------|---------------------|------------------------|
| **주 용도** | 디자인 → 코드 변환 | 코드 → 디자인 생성 |
| **데이터 흐름** | 읽기 중심 (Read) | 쓰기 중심 (Write) |
| **디자인 생성** | ❌ 불가 | ✅ 가능 |
| **디자인 읽기** | ✅ 가능 | ✅ 가능 |
| **코드 생성** | ✅ (프론트엔드) | ❌ |
| **제공사** | Figma 공식 | 커뮤니티 (arinspunk) |
| **연결 방식** | HTTP (Remote/Desktop) | WebSocket |

### 1.2 기능별 상세 비교

#### FigmaDesktop MCP (공식)

```
✅ 선택한 프레임 → 코드 변환
✅ 디자인 토큰 추출 (변수, 색상, 타이포그래피)
✅ 컴포넌트 매핑 정보 제공
✅ 레이아웃 데이터 추출
✅ FigJam 콘텐츠 접근
❌ 디자인 요소 생성 불가
❌ 디자인 수정 불가
```

#### TalkToFigma MCP (커뮤니티)

```
✅ 프레임/도형/텍스트 생성
✅ Auto-layout 설정
✅ 색상/스타일 적용
✅ 컴포넌트 인스턴스 생성
✅ 주석 추가
✅ 프로토타이핑 커넥션 생성
✅ 디자인 요소 읽기/분석
❌ 코드 생성 기능 없음
```

### 1.3 사용 목적별 선택 가이드

| 목적 | 권장 MCP |
|------|----------|
| AI로 기획서/와이어프레임 생성 | **TalkToFigma** |
| Figma 디자인 → Vue/HTML 코드 변환 | **FigmaDesktop** |
| 디자인 시스템 토큰 추출 | **FigmaDesktop** |
| 기존 디자인 분석 | 둘 다 가능 |
| 디자인 자동 수정/업데이트 | **TalkToFigma** |
| 프로토타이핑 연결선 추가 | **TalkToFigma** |

---

## 2. 설치 가이드

### 2.1 Cursor 설치

설정 파일: `~/.cursor/mcp.json`

```json
{
  "mcpServers": {
    "FigmaDesktop": {
      "url": "http://127.0.0.1:3845/mcp"
    },
    "FigmaRemote": {
      "url": "https://mcp.figma.com/mcp"
    },
    "TalkToFigma": {
      "command": "bunx",
      "args": ["cursor-talk-to-figma-mcp@latest"]
    }
  }
}
```

### 2.2 Claude Code 설치

```bash
# FigmaDesktop (로컬)
claude mcp add -s user "FigmaDesktop" --transport http http://127.0.0.1:3845/mcp

# FigmaRemote (클라우드)
claude mcp add -s user "FigmaRemote" --transport http https://mcp.figma.com/mcp

# TalkToFigma
claude mcp add -s user "TalkToFigma" -- bunx cursor-talk-to-figma-mcp@latest
```

### 2.3 사전 요구사항

| 항목 | FigmaDesktop | FigmaRemote | TalkToFigma |
|------|-------------|-------------|-------------|
| Figma 계정 | ✅ | ✅ | ✅ |
| Figma Desktop 앱 | ✅ | ❌ | ✅ |
| Bun 런타임 | ❌ | ❌ | ✅ (`brew install bun`) |

---

## 3. Figma 연동 방법

### 3.1 FigmaDesktop/Remote 연동 (디자인 → 코드)

#### Desktop 서버 사용 시

```
1. Figma Desktop 열기
2. Dev Mode 활성화 (Shift+D)
3. Inspect 패널 → "Enable desktop MCP server" 클릭
4. IDE에서 바로 사용 가능
```

#### Remote 서버 사용 시 (Claude Code)

```
1. Claude Code에서 /mcp 입력
2. FigmaRemote 선택 → Authenticate
3. Figma에서 Allow Access 승인
```

#### 사용 예시

```
# Figma에서 프레임 선택 후
"선택한 프레임을 Vue 3 컴포넌트로 변환해줘"

# 또는 Figma URL 복사 후
"이 디자인을 Vue + Tailwind CSS로 구현해줘: [Figma URL]"
```

---

### 3.2 TalkToFigma 연동 (코드 → 디자인)

#### Step 1: 소켓 서버 실행 (별도 터미널)

```bash
bunx cursor-talk-to-figma-socket
```

> ⚠️ Cursor, Claude Code 모두 소켓 서버 필수 (MCP ↔ Figma 플러그인 브릿지)

#### Step 2: Figma 플러그인 설치

- **방법 A**: Figma Community에서 "Cursor Talk To Figma MCP" 검색
- **방법 B**: [GitHub Releases](https://github.com/grab/cursor-talk-to-figma-mcp/releases)에서 DXT 다운로드

#### Step 3: 채널 연결

```
1. Figma에서 플러그인 실행
2. 표시된 채널 ID 복사 (예: abc123)
3. IDE에서 입력: "Talk to Figma, channel abc123"
4. "Connected" 메시지 확인
```

#### 사용 예시

```
# 프레임 생성
"1280x800 크기의 프레임을 만들고, 상단에 헤더 영역을 배치해줘"

# 컴포넌트 생성
"로그인 폼을 만들어줘. 이메일, 비밀번호 입력란과 로그인 버튼 포함"

# 스타일 수정
"선택한 버튼의 배경색을 파란색(#3B82F6)으로 변경해줘"
```

---

## 4. TalkToFigma로 기획서 작성 가이드

### 4.1 효과적인 기획서 작성 워크플로우

```
[1단계: 준비]
├─ Figma 새 파일 생성
├─ TalkToFigma 플러그인 실행
├─ Claude Code와 채널 연결
└─ 기획서 템플릿 구조 설계

[2단계: 구조 생성]
├─ 메인 프레임 생성 (페이지 레이아웃)
├─ 섹션별 Auto-layout 프레임 배치
├─ 헤더/네비게이션 영역 구성
└─ 콘텐츠 영역 그리드 설정

[3단계: 상세 디자인]
├─ 텍스트 요소 배치 (제목, 본문, 레이블)
├─ UI 컴포넌트 배치 (버튼, 입력폼, 카드)
├─ 색상 및 스타일 적용
└─ 간격 및 정렬 조정

[4단계: 문서화]
├─ 주석(Annotation) 추가
├─ 프로토타이핑 커넥션 연결
├─ 버전 정보 및 변경 이력 기록
└─ 개발자 핸드오프 정보 추가
```

### 4.2 효과적인 프롬프트 작성법

#### 좋은 프롬프트 예시

```
✅ 구체적이고 명확한 지시:
"1280x800 크기의 프레임을 생성하고, 상단에 64px 높이의 헤더 영역을
배치해줘. 헤더는 흰색 배경에 하단 1px 회색 보더를 적용해"

✅ 단계별 요청:
"먼저 모바일 로그인 화면의 기본 프레임을 만들어줘 (375x812).
그 다음 상단에 로고 영역, 중앙에 로그인 폼, 하단에 회원가입 링크를 배치해"

✅ 디자인 시스템 참조:
"기존 버튼 컴포넌트를 사용해서 Primary 버튼과 Secondary 버튼을
나란히 배치해줘. 간격은 16px로"
```

#### 피해야 할 프롬프트

```
❌ 모호한 지시:
"좋은 디자인으로 만들어줘"
"예쁘게 해줘"

❌ 한 번에 너무 많은 요청:
"전체 앱의 모든 화면을 한번에 만들어줘"
```

### 4.3 기획서 작성 프롬프트 템플릿

#### 화면 기본 구조 생성

```
다음 사양으로 [화면명] 와이어프레임을 생성해줘:

1. 프레임 설정
   - 크기: [너비]x[높이]px
   - 배경색: [색상코드]
   - Auto-layout: [VERTICAL/HORIZONTAL]
   - Padding: [상/우/하/좌]px

2. 헤더 영역
   - 높이: [값]px
   - 구성요소: [뒤로가기, 타이틀, 메뉴버튼 등]

3. 콘텐츠 영역
   - [섹션1 설명]
   - [섹션2 설명]

4. 하단 영역
   - [CTA 버튼, 탭바 등]
```

#### 컴포넌트 생성

```
다음 UI 컴포넌트를 생성해줘:

컴포넌트: [버튼/카드/입력필드 등]
- 크기: [너비]x[높이]px
- 배경색: [색상]
- 텍스트: "[텍스트내용]"
- 폰트: [크기]px, [굵기]
- 모서리: [radius]px
- 상태: [기본/호버/비활성화]
```

### 4.4 TalkToFigma 주요 도구 활용

#### 문서 분석 도구

| 도구 | 용도 | 예시 |
|------|------|------|
| `get_document_info` | 전체 문서 구조 파악 | 프로젝트 시작 시 |
| `get_selection` | 선택된 요소 정보 | 수정 전 확인 |
| `get_node_info` | 특정 요소 상세 정보 | 스타일 참조 |
| `scan_text_nodes` | 모든 텍스트 검색 | 번역/수정 시 |
| `get_styles` | 디자인 시스템 스타일 | 일관성 유지 |

#### 생성 도구

| 도구 | 용도 | 주요 파라미터 |
|------|------|--------------|
| `create_frame` | 레이아웃 컨테이너 | x, y, width, height, layoutMode |
| `create_text` | 텍스트 요소 | x, y, text, fontSize, fontWeight |
| `create_rectangle` | 도형/배경 | x, y, width, height |
| `create_component_instance` | 컴포넌트 사용 | componentKey, x, y |

#### 스타일 도구

| 도구 | 용도 |
|------|------|
| `set_fill_color` | 배경색 설정 |
| `set_stroke_color` | 테두리 색상 |
| `set_corner_radius` | 모서리 둥글기 |
| `set_layout_mode` | Auto-layout 설정 |
| `set_padding` | 내부 여백 |
| `set_item_spacing` | 요소 간 간격 |

---

## 5. FigmaDesktop으로 퍼블리싱 가이드

### 5.1 디자인 → 코드 변환 워크플로우

```
[1단계: 디자인 준비]
├─ Auto-layout 적용 확인
├─ 컴포넌트 명명 규칙 정리
├─ 디자인 토큰 정의 (변수)
└─ 레이어 구조 정리

[2단계: FigmaDesktop 연결]
├─ Dev Mode 활성화 (Shift+D)
├─ MCP 서버 활성화
├─ Claude Code 연결 확인
└─ 변환할 프레임 선택

[3단계: 코드 생성]
├─ 프레임 선택 후 코드 생성 요청
├─ 생성된 코드 검토
├─ 필요 시 수정 요청
└─ 최종 코드 적용

[4단계: 통합]
├─ 프로젝트에 코드 삽입
├─ 디자인 토큰 연결
├─ 반응형 조정
└─ 테스트 및 검증
```

### 5.2 효과적인 코드 변환 팁

#### 디자인 준비 체크리스트

```
□ Auto-layout 적용
  - 모든 컨테이너에 Auto-layout 설정
  - 수동 배치된 요소 최소화

□ 명명 규칙 통일
  - 프레임: header, content, footer, sidebar 등
  - 컴포넌트: btn-primary, input-text, card-item 등
  - 아이콘: icon-[이름] 형식

□ 컴포넌트화
  - 반복 요소는 컴포넌트로 변환
  - Variants 활용 (상태별 디자인)

□ 디자인 토큰 정의
  - 색상 변수 (primary, secondary, text, bg 등)
  - 타이포그래피 스타일
  - 간격 시스템 (4px, 8px, 16px, 24px...)
```

#### 코드 생성 프롬프트 예시 (Vue 3 프로젝트)

```
# 기본 변환
"선택한 프레임을 Vue 3 컴포넌트로 변환해줘"

# 상세 요구사항 포함
"선택한 로그인 폼을 Vue 3 + TypeScript로 변환해줘.
- Tailwind CSS 사용
- NuxtUI 컴포넌트 활용
- Composition API (<script setup>) 사용
- 유효성 검증 로직 추가 (Yup)"

# 디자인 시스템 연동
"선택한 카드 컴포넌트를 변환하고,
기존 디자인 토큰 변수를 사용해서 스타일링해줘"
```

### 5.3 프로젝트 통합 가이드

#### Vue 3 프로젝트 퍼블리싱 연동

```
webapp/src/modules/[모듈명]/pages/
├── list.vue           // 목록 화면 (Figma 프레임 → Vue 변환)
├── list-search.vue    // 검색 컴포넌트
├── list-table.vue     // 테이블 컴포넌트
├── detail.vue         // 상세 화면
├── insert.vue         // 등록 모달 (폼 컴포넌트)
└── update.vue         // 수정 모달 (폼 컴포넌트)
```

#### Figma → Vue 컴포넌트 변환 워크플로우

```
1. Figma에서 화면 프레임 선택
2. FigmaDesktop으로 Vue + Tailwind 코드 추출
3. 프로젝트 형식으로 변환
   - NuxtUI 컴포넌트 적용 (UFormField, UModal, USwitch 등)
   - Pinia Store 연동
   - 타입 정의 추가
4. pages 폴더에 저장
5. 라우터 및 Store 연결 테스트
```

### 5.4 주의사항 및 한계

#### FigmaDesktop 제한사항

```
⚠️ 복잡한 멀티프레임 플로우
   - 캐러셀, 온보딩 시퀀스 등은 개별 변환 후 수동 조합 필요

⚠️ 디자인 업데이트 시
   - 기존 코드 surgical update 어려움
   - 재생성 또는 수동 수정 필요

⚠️ 사용량 제한
   - Starter 플랜: 월 6회 도구 호출
   - Professional 이상: API Rate Limit 적용
```

#### 권장 사항

```
✅ 작은 단위로 변환
   - 전체 페이지보다 컴포넌트 단위 변환
   - 점진적으로 조합

✅ 디자인 시스템 우선
   - 공통 컴포넌트 먼저 변환
   - 재사용 가능한 코드 베이스 구축

✅ 수동 최적화
   - 생성된 코드 리뷰 필수
   - 프로젝트 컨벤션에 맞게 조정
```

---

## 6. 하이브리드 워크플로우

### 6.1 TalkToFigma + FigmaDesktop 연계

```
[기획 단계] TalkToFigma
├─ Claude Code로 와이어프레임 생성
├─ 화면 구조 및 레이아웃 설계
├─ 컴포넌트 배치 및 플로우 정의
└─ 주석 및 개발 스펙 추가

[디자인 단계] Figma 직접 작업
├─ 비주얼 디자인 보완
├─ 디자인 시스템 적용
├─ 프로토타이핑 완성
└─ 디자인 리뷰 및 승인

[개발 단계] FigmaDesktop
├─ 승인된 디자인 프레임 선택
├─ 코드 생성 요청
├─ 프로젝트 코드베이스에 통합
└─ 테스트 및 배포
```

### 6.2 실제 활용 시나리오

#### 시나리오: 새 관리자 화면 개발

```bash
# 1. TalkToFigma로 기획서 생성
"관리자 페이지용 재고 관리 화면을 만들어줘.
- 상단: 검색 필터 (상품명, 카테고리, 재고상태)
- 중앙: 상품 목록 테이블 (컬럼: 코드, 명칭, 재고량, 단가, 상태)
- 하단: 페이지네이션, 일괄처리 버튼"

# 2. Figma에서 디자인 보완 (수동)
# - 실제 색상, 아이콘 적용
# - 기존 디자인 시스템 컴포넌트 연결

# 3. FigmaDesktop으로 코드 생성
"선택한 상품 목록 테이블을 Vue 3 + Tailwind CSS로 변환해줘.
NuxtUI 컴포넌트와 EasyDataTable을 사용해서 작성해"

# 4. 프로젝트에 적용
# webapp/src/modules/inventory/pages/list.vue
```

---

## 7. 참고 자료

### 공식 문서
- [Figma MCP Server 공식 문서](https://developers.figma.com/docs/figma-mcp-server/)
- [Figma MCP 가이드](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server)

### 커뮤니티
- [TalkToFigma GitHub](https://github.com/arinspunk/claude-talk-to-figma-mcp)
- [Builder.io - Claude Code + Figma](https://www.builder.io/blog/claude-code-figma-mcp-server)

### 튜토리얼
- [Composio - Figma MCP 활용법](https://composio.dev/blog/how-to-use-figma-mcp-with-claude-code-to-build-pixel-perfect-designs)
- [Figma Remote MCP 설정](https://help.figma.com/hc/en-us/articles/35281350665623)

---

*문서 작성일: 2025-12-07*
*수정일: 2025-12-08 (프로젝트 스택에 맞게 수정)*
*작성: Claude Code + TalkToFigma/FigmaDesktop MCP 분석 기반*
