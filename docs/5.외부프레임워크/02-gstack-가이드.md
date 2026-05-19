# gstack 가이드

> YC 사장 Garry Tan이 공개한 Claude Code용 스킬 팩 — CEO·PM·Eng·Designer·QA·Security를 가상 팀처럼 부르는 48개 슬래시 커맨드

---

## 핵심은 이 두 가지만 기억

| 상황 | 사용할 스킬 |
|---|---|
| **코딩 전 아이디어 점검** | `/office-hours` — "여섯 가지 강제 질문"으로 진짜 만들려는 게 뭔지 재정의하는 디스커버리 컨설턴트 |
| **실제 브라우저 자동화** | `/browse` — 실제 Chromium 브라우저로 클릭·스크린샷 (~100ms/cmd), 안티봇 stealth·쿠키 임포트·도메인 메모리 지원. MFA/CAPTCHA는 `$B` 핸드오프 |

나머지 46개 스킬(`/cso`, `/qa`, `/codex`, `/investigate`, `/autoplan` 등)은 필요할 때 자연어로 트리거되거나 위 두 흐름 안에서 자동으로 따라온다.

---

## 설치 방법

### 사전 조건
- macOS / Linux (Windows는 WSL 권장)
- [bun](https://bun.sh/) 설치 필수
  ```bash
  curl -fsSL https://bun.sh/install | bash
  ```

### 설치 (clone + setup, 2줄)

```bash
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup
```

설치 위치는 `~/.claude/skills/gstack/` 고정. Claude Code가 글로벌 스킬을 자동 인식하는 표준 경로다.

### setup이 하는 일
1. 브라우저 바이너리 빌드 (Playwright Chromium ~91MB 다운로드)
2. `bin/` 도구 PATH 등록
3. 48개 스킬을 `~/.claude/skills/` 아래에 심링크 등록
4. Codex / OpenCode / Factory CLI에도 동시 등록 (감지된 경우)

### 업그레이드

Claude Code 세션 안에서:
```
/gstack-upgrade
```

### 팀 저장소에 강제 적용 (선택)

```bash
(cd ~/.claude/skills/gstack && ./setup --team)
~/.claude/skills/gstack/bin/gstack-team-init required
git add .claude/ CLAUDE.md
git commit -m "require gstack for AI-assisted work"
```

---

## 주요 스킬 분류

| 분류 | 스킬 |
|---|---|
| **디스커버리·기획** | `/office-hours`, `/plan-ceo-review`, `/plan-eng-review`, `/plan-design-review`, `/autoplan` |
| **브라우저·QA** | `/browse`, `/qa`, `/qa-only`, `/scrape`, `/skillify`, `/canary` |
| **품질·보안** | `/cso`, `/review`, `/investigate`, `/health`, `/security-review` |
| **외부 AI 협업** | `/codex` (OpenAI Codex 리뷰/챌린지), `/pair-agent` |
| **출시·문서** | `/ship`, `/land-and-deploy`, `/document-release`, `/document-generate`, `/retro` |
| **세션 관리** | `/context-save`, `/context-restore`, `/freeze`, `/unfreeze`, `/careful` |

---

## 참고

- 저장소: <https://github.com/garrytan/gstack> (MIT)
- 공식 소개: <https://gstacks.org/>
- Product Hunt: <https://www.producthunt.com/products/gstack>
- 작성자: Garry Tan (YC CEO, @garrytan)
