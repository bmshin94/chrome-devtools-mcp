# Chrome DevTools MCP 전수조사 분석 리포트 🔍

> 작성일: 2026-09-21
> 작성: 카리나 (Claude Code AI 개발 파트너) with 오빠 💖
> 대상 저장소: `chrome-devtools-mcp` v1.9.0

---

## 🔗 관련 링크

| 구분 | 주소 |
|---|---|
| **이 저장소 (포크본)** | https://github.com/bmshin94/chrome-devtools-mcp |
| **원본 저장소 (Google 공식)** | https://github.com/ChromeDevTools/chrome-devtools-mcp |
| **npm 패키지** | https://www.npmjs.com/package/chrome-devtools-mcp |
| **공식 문서 (Chrome for Developers)** | https://developer.chrome.com/docs/devtools/agents |
| **MCP 레지스트리 ID** | `io.github.ChromeDevTools/chrome-devtools-mcp` |
| **이슈 트래커** | https://github.com/ChromeDevTools/chrome-devtools-mcp/issues |
| **DevTools Frontend (서브모듈)** | https://github.com/ChromeDevTools/devtools-frontend |
| **Puppeteer** | https://github.com/puppeteer/puppeteer |
| **MCP 프로토콜 공식** | https://modelcontextprotocol.io |
| **Gemini CLI 브라우저 에이전트 참고** | https://geminicli.com/docs/core/subagents/#browser-agent |

---

## 📑 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [쉬운 설명 (비유편)](#2-쉬운-설명-비유편)
3. [폴더 구조 전수조사](#3-폴더-구조-전수조사)
4. [제공 도구 전체 목록](#4-제공-도구-전체-목록)
5. [스킬(Skills) 7종](#5-스킬skills-7종)
6. [설치 및 사용법](#6-설치-및-사용법)
7. [플러그인 vs 스킬 vs MCP](#7-플러그인-vs-스킬-vs-mcp)
8. [API 토큰 필요 여부](#8-api-토큰-필요-여부)
9. [왜 유명한가](#9-왜-유명한가)
10. [로컬 에이전트 구축에 주는 도움](#10-로컬-에이전트-구축에-주는-도움)
11. [React / PHP로 만들 수 있는가](#11-react--php로-만들-수-있는가)
12. [수익화 아이디어 9선](#12-수익화-아이디어-9선)
13. [실행 로드맵](#13-실행-로드맵)
14. [주의사항 정리](#14-주의사항-정리)

---

## 1. 프로젝트 개요

### 한 줄 요약

> **AI 에이전트(Claude, Cursor, Gemini, Copilot 등)에게 "진짜 크롬 브라우저"를 조종할 수 있게 해주는 공식 MCP 서버**

기존 AI는 코드만 읽고 **추측**했다면, 이 도구를 붙이면 AI가 직접 크롬을 띄워
접속 → 클릭 → 개발자도구 분석 → 증거 수집 → 원인 특정까지 수행한다.

### 기본 정보

| 항목 | 내용 |
|---|---|
| 원본 | `ChromeDevTools/chrome-devtools-mcp` (**Google LLC 공식**) |
| 포크본 | `bmshin94/chrome-devtools-mcp` |
| 버전 | v1.9.0 |
| 라이선스 | **Apache-2.0** (상업적 이용 가능) |
| 언어 | TypeScript |
| 소스 규모 | 85개 파일 / 약 22,000줄 |
| 테스트 | 테스트 파일 69개 |
| 핵심 엔진 | Puppeteer 25.10 + Lighthouse 13.4 + DevTools Frontend(서브모듈) |
| Node 요구사항 | `^20.19.0 \|\| ^22.12.0 \|\| >=23` |
| 진입점 | MCP 서버(`chrome-devtools-mcp`) + CLI(`chrome-devtools`) |

### 핵심 기능 3가지 (README 기준)

1. **성능 인사이트** — Chrome DevTools로 트레이스 녹화 후 실행 가능한 인사이트 추출
2. **고급 브라우저 디버깅** — 네트워크 요청 분석, 스크린샷, 콘솔 메시지(소스맵 적용 스택트레이스)
3. **신뢰성 있는 자동화** — Puppeteer 기반, 액션 결과를 자동으로 대기

### 포크본 추가 커밋

- `86498e5` docs: created CLAUDE.md persona guide
- `609f066` Merge pull request #1 from bmshin94/feat/claude-guide

---

## 2. 쉬운 설명 (비유편)

### Before / After

```
❌ 이거 없을 때
오빠: "장바구니 버튼이 안 눌려"
AI:   "이벤트 버블링이나 z-index 문제일 수도... 아마도..." 🤷

✅ 이거 있을 때
오빠: "장바구니 버튼이 안 눌려"
AI:   (크롬 실행 → 접속 → 클릭 시도 → 콘솔/네트워크 확인 → 스크린샷)
      "찾았어요! 투명 배너 div가 버튼을 덮고 있어요.
       /api/cart 요청 자체가 안 갔고, header.css:42의 z-index를
       999 → 10으로 낮추면 해결돼요. 스크린샷 첨부합니다 📸"
```

### 비유 1 — AI에게 "손과 눈"을 달아주는 장치

| 도구 | 비유 |
|---|---|
| `click`, `fill`, `type_text` | 🖐️ 손 |
| `take_screenshot`, `take_snapshot` | 👀 눈 |
| `list_console_messages` | 👂 귀 (에러 비명 듣기) |
| `list_network_requests` | 🩺 청진기 |
| `performance_*` | 📊 MRI |
| `take_heapsnapshot` | 🧬 혈액검사 |
| `skills/` | 📖 의대 교과서 |

### 비유 2 — MCP는 "AI 세계의 USB-C 규격"

```
  [AI 프로그램]          [MCP 규격]          [실제 도구]
  Claude Code ─┐                        ┌─ 크롬 브라우저 ← 이 프로젝트
  Cursor      ─┤                        ├─ 깃허브
  Gemini CLI  ─┼── 동일한 규격으로 연결 ──┼─ 노션
  Copilot     ─┤                        ├─ 슬랙
  VS Code     ─┘                        └─ 구글드라이브
```

예전엔 AI마다 연결 방식이 달랐는데(기기마다 다른 충전기), MCP가 나오면서
C타입 하나로 전부 되는 세상이 됐다. 이 프로젝트는 그중 **"크롬 포트"**이며,
**구글 크롬 팀이 직접 만들었다**는 게 핵심.

### 비유 3 — 실제 동작 흐름

"내 사이트 속도 좀 봐줘" 한마디에 AI가 실행하는 콤보:

```
1. new_page("https://myshop.com")            🌐 크롬 켜고 접속
2. performance_start_trace(reload: true)     🔴 녹화 + 새로고침
3. performance_stop_trace()                  ⏹️ 녹화 종료
4. performance_analyze_insight("LCPBreakdown") 🔬 병목 정밀분석
5. list_network_requests()                   📡 리소스 크기/시간
6. take_screenshot()                         📸 증거
7. 최종 리포트 작성                            💬
```

결과 예시:
> LCP 4.1초(나쁨). 그중 3.2초가 `hero-banner.png`(4.2MB PNG) 다운로드.
> ① WebP 변환 → 380KB (91%↓) ② `fetchpriority="high"` ③ `<link rel="preload">`
> → 예상 LCP **4.1초 → 1.3초**

수동으로 하면 20~30분, AI는 1~2분.

### 핵심 개념 — 스냅샷(Snapshot)

AI는 이미지로 좌표 찍기가 어렵다. 그래서 페이지를 **텍스트 지도**로 바꾼다.

```
uid=1_0 RootWebArea "쇼핑몰 홈" url="https://myshop.com/"
  uid=1_1 heading "신상품" level="1"
  uid=1_5 button "장바구니 담기"      ← 이걸 클릭!
  uid=1_6 textbox "검색어 입력"
```

```
click(pageId: 1, uid: "1_5")
fill(pageId: 1, uid: "1_6", "후드티")
```

**장점**: 접근성 트리(스크린리더가 읽는 구조) 기반이라
정확하고 · 토큰이 가볍고 · 화면 크기가 바뀌어도 안 깨진다.
덤으로 **접근성 검증까지 동시에 된다.**

---

## 3. 폴더 구조 전수조사

### `src/` — 본체 (85파일 / 22,000줄)

| 경로 | 역할 |
|---|---|
| `src/bin/` | 실행 진입점 (MCP 서버 + CLI) |
| `src/tools/` | ⭐ AI가 호출하는 도구 정의 (20개 파일) |
| `src/collectors/` | `PageCollector`, `ServiceWorkerCollector` — 이벤트 수집 |
| `src/processors/` | `PerformanceTrace`, `ChunkedTraceParser`, `HeapSnapshotManager` |
| `src/formatters/` | 7종 — 결과를 AI가 읽기 좋은 텍스트로 변환 |
| `src/devtools/` | 실제 크롬 DevTools 연동 (`DevToolsCommentBridge` 등) |
| `src/daemon/` | CLI용 백그라운드 데몬 (유닉스 소켓 / 네임드 파이프) |
| `src/telemetry/` | 사용 통계 (`ClearcutLogger`, watchdog) |
| `src/config/` | CLI 옵션, 브라우저 옵션, MCP 옵션 |
| `src/third_party/` | DevTools 워커(포매터, 힙스냅샷) |

**핵심 클래스**: `McpContext`, `McpPage`, `McpResponse`, `SlimMcpResponse`,
`TextSnapshot`, `ToolHandler`, `WaitForHelper`

### 나머지 폴더

| 경로 | 내용 |
|---|---|
| `skills/` | ⭐ AI용 전문 노하우 7종 (아래 5장 참고) |
| `docs/` | 문서 10종 (설치/CLI/설정/안드로이드/트러블슈팅/설계원칙 등) |
| `tests/` | 테스트 69개 (목 기반 단위테스트 위주) |
| `scripts/` | 문서 자동생성, Gemini eval(시나리오 25개), 메모리 프로파일링, 토큰 카운팅, 커스텀 ESLint 룰 |
| `.github/workflows/` | CI 8종 (테스트/릴리즈/npm 배포/MCP 레지스트리 배포/메모리 누수 감시) |
| `third_party/devtools-frontend` | 크롬 개발자도구 실제 소스 (git submodule) |

### 멀티 클라이언트 설정 파일

| 파일 | 대상 |
|---|---|
| `.claude-plugin/plugin.json` + `marketplace.json` | Claude Code 플러그인 |
| `.cursor-plugin/plugin.json` | Cursor |
| `gemini-extension.json`, `.gemini/settings.json` | Gemini CLI |
| `mcp.json`, `server.json` | 표준 MCP 레지스트리 |
| `plugin.json` | 범용 agent-plugins.org 규격 |

### 레스토랑 비유

| 폴더 | 비유 |
|---|---|
| `src/tools/` | 🍳 주방 기구 |
| `src/collectors/` | 📝 주문 받는 직원 |
| `src/processors/` | 🔪 재료 손질 |
| `src/formatters/` | 🍽️ 플레이팅 |
| `skills/` | 📖 레시피북 |
| `docs/` | 📋 메뉴판 |
| `tests/` | 👨‍🍳 위생 검사 |
| `.claude-plugin/` 등 | 🚪 각 클라이언트용 출입문 |

---

## 4. 제공 도구 전체 목록

총 **50개 이상**의 도구 제공.

### 🖱️ 입력 자동화 (10)
`click` · `drag` · `fill` · `fill_form` · `handle_dialog` · `hover` ·
`press_key` · `type_text` · `upload_file` · `click_at`

### 🧭 네비게이션 (6)
`new_page` · `navigate_page` · `list_pages` · `select_page` · `close_page` · `wait_for`

### 📱 에뮬레이션 (2)
`emulate` (모바일/느린 네트워크/저사양 CPU) · `resize_page`

### ⚡ 성능 (3)
`performance_start_trace` · `performance_stop_trace` · `performance_analyze_insight`

### 🌐 네트워크 (2)
`list_network_requests` · `get_network_request`

### 🐛 디버깅 (9)
`evaluate_script` · `list_console_messages` · `get_console_message` ·
`get_css_styles` · `take_snapshot` · `take_screenshot` ·
`screencast_start` · `screencast_stop` · `lighthouse_audit`

### 🧠 메모리 (13) — `--memoryDebugging` 필요
`take_heapsnapshot` · `close_heapsnapshot` · `compare_heapsnapshots` ·
`get_heapsnapshot_class_nodes` · `get_heapsnapshot_details` ·
`get_heapsnapshot_dominators` · `get_heapsnapshot_duplicate_strings` ·
`get_heapsnapshot_edges` · `get_heapsnapshot_object_details` ·
`get_heapsnapshot_retainers` · `get_heapsnapshot_retaining_paths` ·
`get_heapsnapshot_summary` · `query_heapsnapshot_objects`

### 🧩 확장프로그램 (5) — `--categoryExtensions` 필요
`install_extension` · `list_extensions` · `reload_extension` ·
`trigger_extension_action` · `uninstall_extension`

### 📲 PWA (4) — `--categoryPwa` 필요
`install_pwa` · `launch_pwa` · `uninstall_pwa` · `get_os_app_state`

### 🔌 확장 연동 (4)
`list_3p_developer_tools` · `execute_3p_developer_tool` ·
`list_webmcp_tools` · `execute_webmcp_tool`

### 🪶 Slim 모드 (3) — `--slim`
`navigate` · `evaluate` · `screenshot` (토큰 절약용 최소 세트)

---

## 5. 스킬(Skills) 7종

단순히 도구만 주는 게 아니라 **"이럴 땐 이렇게 디버깅해라"** 하는
전문가 노하우를 마크다운으로 동봉. 이게 AI를 주니어에서 시니어로 승격시킨다.

| 스킬 | 내용 |
|---|---|
| `chrome-devtools` | 기본 워크플로우 (네비게이트 → 대기 → 스냅샷 → 조작) |
| `chrome-devtools-cli` | 터미널/쉘스크립트 자동화 + 설치 가이드 |
| `debug-optimize-lcp` | **LCP 최적화 완전 공략** — TTFB/로드지연/로드시간/렌더지연 4분할 분석, 최적화 전략, 실전 스니펫 |
| `memory-leak-debugging` | 메모리 누수 진단 (detached DOM, 클로저, 리스너 누락, 무한 캐시) |
| `cookie-debugging` | 401/403, 세션 만료, HttpOnly/SameSite/Partitioned, 쿠키 동의 배너 검증 |
| `a11y-debugging` | 접근성 감사 (ARIA, 키보드 내비, 색 대비, 탭 타겟, web.dev 가이드 연계) |
| `troubleshooting` | 연결 실패 시 진단 마법사 (설정파일 탐색 → 에러 패턴 분류) |

### LCP 4단계 분해표 (스킬 내용 발췌)

| 구간 | 이상적 비율 | 의미 |
|---|---|---|
| TTFB | ~40% | 네비게이션 시작 → HTML 첫 바이트 |
| Resource load delay | <10% | TTFB → LCP 리소스 로드 시작 |
| Resource load duration | ~40% | LCP 리소스 다운로드 시간 |
| Element render delay | <10% | 다운로드 완료 → 실제 렌더 |

> LCP 기준: 좋음 ≤2.5초 / 개선필요 2.5~4.0초 / 나쁨 >4.0초
> 모바일 페이지의 **73%**에서 LCP 요소는 이미지다.

---

## 6. 설치 및 사용법

### 준비물

| 필수 | 버전 |
|---|---|
| Node.js | `^20.19.0` / `^22.12.0` / `>=23` (LTS 권장) |
| Chrome | 최신 stable 이상 |
| npm | Node 포함 |

### 방법 A — MCP 서버 (추천)

**Claude Code (1) CLI — MCP만**
```bash
claude mcp add chrome-devtools --scope user npx chrome-devtools-mcp@latest
```

**Claude Code (2) 플러그인 — MCP + 스킬 전부 ⭐추천**
```bash
/plugin marketplace add ChromeDevTools/chrome-devtools-mcp
/plugin install chrome-devtools-mcp@chrome-devtools-plugins
# Claude Code 재시작 → /skills 로 확인
```
> 기존에 MCP로 설치했다면 먼저 제거할 것.

**공통 설정 (Cursor / VS Code / Cline / Windsurf 등)**
```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

**Codex (OpenAI)**
```bash
codex mcp add chrome-devtools -- npx chrome-devtools-mcp@latest
```

**Antigravity (내장 브라우저 연결)**
```json
{ "mcpServers": { "chrome-devtools": {
  "command": "npx",
  "args": ["-y", "chrome-devtools-mcp@latest", "--browser-url=http://127.0.0.1:9222"]
}}}
```

**설치 확인 첫 프롬프트**
```
https://developers.chrome.com 성능 체크해줘
```
→ 크롬이 뜨고 성능 트레이스를 녹화하면 성공.

> 참고: MCP 연결만으로는 브라우저가 뜨지 않는다.
> **브라우저가 필요한 도구를 처음 호출할 때** 자동 실행된다.

### 방법 B — CLI

```bash
npm i chrome-devtools-mcp@latest -g
chrome-devtools status

chrome-devtools new_page "https://example.com"
chrome-devtools take_snapshot 1
chrome-devtools click 1 "1_5"
chrome-devtools fill 1 "1_6" "검색어"
chrome-devtools take_screenshot 1 --filePath shot.png
chrome-devtools stop
```

> ⚠️ CLI는 기본이 **파일시스템 무제한 접근**.
> 반드시 `--workspace`로 제한할 것:
> ```bash
> chrome-devtools start --workspace=/path/to/project --workspace=/path/to/output
> ```

CLI는 백그라운드 데몬(유닉스 소켓/네임드 파이프)을 써서
**명령어 사이에 브라우저 상태(로그인, 쿠키, 열린 탭)가 유지된다.**

### 주요 옵션

| 옵션 | 설명 | 기본값 |
|---|---|---|
| `--headless` | 창 없이 실행 | `false` |
| `--isolated` | 임시 프로필 사용 후 자동 삭제 (**보안 권장**) | `false` |
| `--slim` | 도구를 3개로 축소 (토큰 절약) | `false` |
| `--userDataDir` | 특정 크롬 프로필 사용 | `$HOME/.cache/chrome-devtools-mcp/chrome-profile` |
| `--browserUrl` / `-u` | 실행 중인 크롬에 연결 (예: `http://127.0.0.1:9222`) | - |
| `--wsEndpoint` / `-w` | WebSocket으로 연결 | - |
| `--wsHeaders` | WS 커스텀 헤더 (JSON) | - |
| `--autoConnect` | Chrome 144+ 로컬 인스턴스 자동 연결 | `false` |
| `--channel` | `canary`/`dev`/`beta`/`stable` | `stable` |
| `--executablePath` / `-e` | 커스텀 크롬 경로 | - |
| `--viewport` | 초기 뷰포트 (예: `1280x720`) | - |
| `--proxyServer` | 프록시 설정 | - |
| `--chromeArg` | 크롬 추가 인자 | - |
| `--acceptInsecureCerts` | 자체서명/만료 인증서 무시 | `false` |
| `--pageIdRouting` | pageId 기반 라우팅 (동시 세션용) | `true` |
| `--memoryDebugging` | 메모리 도구 13종 활성화 | `false` |
| `--categoryExtensions` | 확장프로그램 도구 활성화 | `false` |
| `--categoryPwa` | PWA 도구 활성화 | `false` |
| `--categoryEmulation` / `Performance` / `Network` | 해당 카테고리 포함 | `true` |
| `--categoryExperimentalThirdParty` | 서드파티 개발자 도구 | `false` |
| `--experimentalDevtools` | DevTools 타겟 자동화 | `false` |
| `--experimentalVision` | `click_at(x,y)` 좌표 클릭 | `false` |
| `--logFile` | 디버그 로그 파일 경로 | - |
| `--no-usage-statistics` | 구글 사용통계 끄기 | (수집 ON) |
| `--no-performance-crux` | CrUX API 전송 끄기 | (전송 ON) |

### 추천 설정

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "-y", "chrome-devtools-mcp@latest",
        "--isolated",
        "--no-usage-statistics",
        "--viewport", "1440x900"
      ]
    }
  }
}
```

---

## 7. 플러그인 vs 스킬 vs MCP

### 결론: **셋 다 맞다. 본질은 MCP 서버.**

```
┌─────────────────────────────────────────────────┐
│ 🎁 플러그인 = 포장 상자 (배포 방식)              │
│ ┌─────────────────────────────────────────────┐ │
│ │ 🔌 MCP 서버 = 알맹이 (실제 기능, 도구 50개)  │ │
│ │  + 📖 스킬 7종 = 사용설명서 (AI용 노하우)    │ │
│ └─────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

| 구분 | 역할 | 증거 파일 |
|---|---|---|
| 🔌 MCP 서버 | ⭐ 본질. 도구 50개를 제공하는 실행 프로그램 | `server.json`, `mcp.json`, `src/index.ts` |
| 📖 스킬 | "이럴 땐 이렇게" 마크다운 지침서 7개 | `skills/*/SKILL.md` |
| 🎁 플러그인 | MCP+스킬을 한 번에 설치해주는 패키징 | `.claude-plugin/`, `.cursor-plugin/`, `plugin.json` |
| 💻 CLI | 보너스. 터미널용 명령어 도구 | `src/bin/chrome-devtools.ts`, `src/daemon/` |

| 비교 | MCP 서버 | 스킬 |
|---|---|---|
| 정체 | 실행 프로그램 (Node.js) | 텍스트(마크다운) 파일 |
| 역할 | 실제 브라우저 조종 (**능력**) | 어떻게 쓸지 알려줌 (**지식**) |
| 없으면 | 아무것도 못 함 | 되긴 하지만 AI가 헤맴 |
| 비유 | 🔧 공구세트 | 📕 DIY 교본 |

### 설치 방식별 포함 항목

| 설치 방법 | MCP | 스킬 |
|---|:---:|:---:|
| `claude mcp add ...` | ✅ | ❌ |
| `/plugin install chrome-devtools-mcp@...` | ✅ | ✅ ⭐ |
| `npm i -g chrome-devtools-mcp` | (CLI) | ✅ (패키지 동봉) |

---

## 8. API 토큰 필요 여부

### 결론: **일반 사용자는 토큰 불필요. 완전 무료.**

| 이유 | 설명 |
|---|---|
| 🏠 전부 로컬 실행 | 내 컴퓨터의 크롬에 직접 연결 |
| 🆓 Apache-2.0 | 무료 + 상업적 이용 자유 |
| 🔓 인증 계층 없음 | 로그인/계정 개념 자체가 없음 |

### 단, 외부로 나가는 통신 3가지

| # | 대상 | 시점 | 끄는 법 | 토큰 |
|---|---|---|---|:---:|
| 1 | **Google CrUX API** | 성능 트레이스 시 URL 전송 → 실사용자 데이터 수신 | `--no-performance-crux` | 불필요 |
| 2 | **사용 통계** (기본 ON) | 도구 성공률/지연시간/환경정보 수집 | `--no-usage-statistics` 또는<br>`CHROME_DEVTOOLS_MCP_NO_USAGE_STATISTICS` | 불필요 |
| 3 | **npm 업데이트 체크** | 주기적 최신 버전 확인 | `CHROME_DEVTOOLS_MCP_NO_UPDATE_CHECKS` | 불필요 |

> `CI` 환경변수가 설정돼 있으면 통계 수집은 자동 비활성화.

### 토큰이 필요한 유일한 경우 (기여자 전용)

```bash
# scripts/eval_gemini.ts:181 — AI 평가(eval) 실행 시에만
export GEMINI_API_KEY="..."
npm run eval
```

### 진짜 조심할 것: 토큰이 아니라 "브라우저 내용"

README 경고:
> `chrome-devtools-mcp`는 브라우저 내용을 MCP 클라이언트에 노출시킨다.

**안전 수칙**
- 민감한 작업엔 반드시 `--isolated`
- 개인 프로필 사용 시 은행/이메일 탭 닫고 시작
- 회사 프로젝트면 `--no-usage-statistics` 필수

---

## 9. 왜 유명한가

| # | 이유 | 근거 |
|---|---|---|
| 1 | 🏢 **구글 크롬 팀 공식** | `author: "Google LLC"`, `ChromeDevTools` 조직 소유 |
| 2 | ⏰ **완벽한 타이밍** | "AI가 코드는 짜는데 진짜 되는지 모른다"는 최대 공백을 정확히 메움 |
| 3 | 🔌 **모든 AI 툴 지원 (락인 없음)** | Amp, Antigravity, Bob, Claude Code, Cline, Codex, Copilot, Cursor, Gemini CLI, VS Code, Windsurf, Warp, Zed… |
| 4 | 🧠 **토큰 최적화 설계** | 설계원칙: *"LCP was 3.2s" is better than 50k lines of JSON* |
| 5 | 🎯 **흉내가 아닌 진짜 DevTools** | `third_party/devtools-frontend` 서브모듈로 실제 소스 사용 |
| 6 | 🏗️ **프로덕션급 품질** | 테스트 69개, CI 8종, `any`/`as`/`!`/`@ts-ignore` 전면 금지, 커스텀 린트룰, release-please 자동화, CHANGELOG 13만 자 |
| 7 | 💎 **스킬이라는 신의 한 수** | 도구 + 전문가 노하우를 함께 배포 |

### 설계 원칙 7개 (`docs/design-principles.md`)

1. **Agent-Agnostic API** — 표준(MCP) 사용, 특정 LLM에 락인하지 않는다
2. **Token-Optimized** — 의미 요약 반환. 대용량은 파일로
3. **Small, Deterministic Blocks** — 마법 버튼이 아닌 조합 가능한 작은 도구
4. **Self-Healing Errors** — 에러에 컨텍스트와 해결책을 담아 AI가 스스로 복구
5. **Human-Agent Collaboration** — 기계(구조화) + 사람(요약) 모두 읽을 수 있게
6. **Progressive Complexity** — 기본은 단순, 고급 사용자에겐 옵션
7. **Reference over Value** — 무거운 자산은 경로/URI 반환, 원시 스트림 금지

---

## 10. 로컬 에이전트 구축에 주는 도움

### 결론: **매우 크다. 두 방향 모두.**

### 방향 A — "부품"으로 가져다 쓰기

README 공식 권장:
> **Integrating as a browser subagent** — 에이전트 툴링을 개발하면서 통합 브라우저
> 서브에이전트를 제공하고 싶다면, Chrome DevTools for agents 위에 쌓아올릴 것을 권장.
> (레퍼런스 구현: Gemini CLI browser agent)

| 직접 만들면 | 이걸 쓰면 |
|---|---|
| Puppeteer 래핑 + 대기 로직 | ✅ 완성 |
| 접근성 트리 스냅샷 시스템 | ✅ 완성 |
| 성능 트레이스 파싱 | ✅ 완성 |
| 소스맵 적용 콘솔 스택 | ✅ 완성 |
| 힙 스냅샷 분석 13종 | ✅ 완성 |
| 크롬 버전 호환 대응 | ✅ 구글이 계속 유지보수 |
| **수개월 + 지속 유지보수** | **설정 5줄** |

### 방향 B — "교과서"로 배우기

| 배울 것 | 어디서 | 왜 중요 |
|---|---|---|
| MCP 서버 표준 구조 | `src/index.ts`, `src/ToolHandler.ts` | 툴 등록/디스패치 정석 |
| 툴 정의 패턴 | `src/tools/ToolDefinition.ts` + zod | AI가 헷갈리지 않는 스키마 설계 |
| 카테고리 기반 툴 on/off | `src/tools/categories.ts`, `src/config/` | 툴이 많으면 AI 성능이 떨어진다 |
| 토큰 최적화 | `src/formatters/*` (7종) | 비용이 10배 차이 |
| Slim 모드 계층 설계 | `src/tools/slim/`, `SlimMcpResponse.ts` | 도구 50개 → 3개 축소 전략 |
| 컨텍스트 수명관리 | `src/McpContext.ts`, `McpPage.ts` | 세션 간 상태 관리 |
| 이벤트 수집 | `src/collectors/PageCollector.ts` | 비동기 이벤트 버퍼링 + 쿼리 |
| 자동 대기(auto-wait) | `src/utils/WaitForHelper.ts` | 에이전트 신뢰도의 핵심 |
| 데몬 + IPC | `src/daemon/` | CLI가 상태를 유지하는 비법 |
| 동시 세션 처리 | `--pageIdRouting`, `docs/advanced-usage.md` | 여러 에이전트가 브라우저 공유 |
| 대용량 데이터 처리 | `ChunkedTraceParser.ts` | 스크린샷/트레이스는 경로만 반환 |
| 자가치유 에러 | 설계원칙 #4 | AI 자율 복구 |
| 스킬 작성법 | `skills/*/SKILL.md` | frontmatter + 워크플로우 + references |
| **에이전트 평가(eval)** | `scripts/eval_gemini.ts` + 시나리오 25개 | **성능을 수치로 측정하는 방법론** |
| 메모리 프로파일링 | `scripts/profile/`, `test-memory-leaks.yml` | 장시간 구동 에이전트 필수 |

### 꼭 볼 3가지

1. **`docs/design-principles.md`** — 7줄짜리지만 에이전트 툴 설계의 정수
2. **`scripts/eval_scenarios/`** — 25개 평가 시나리오
   (`navigation`, `input`, `network`, `performance`, `cookie_debugging`,
   `lighthouse_a11y`, `snapshot`, `page_id_routing_concurrent_form`,
   `isolated_context` 등)
3. **`scripts/count_tokens.ts`** — 툴 정의의 토큰 소모량 측정
   (컨텍스트가 작은 로컬 LLM에는 생명줄)

### 로컬 에이전트 구축 시 주의점

| 주의 | 대응 |
|---|---|
| 로컬 LLM은 컨텍스트가 작다 | `--slim` 또는 카테고리 off로 도구 수 축소 |
| 크롬은 메모리를 많이 쓴다 | `--headless` + `--isolated` |
| 작은 모델은 uid 다루기 어려움 | slim 모드 또는 `--experimentalVision` |
| 파일 접근 권한 | CLI는 `--workspace`로 제한 필수 |

---

## 11. React / PHP로 만들 수 있는가

### 결론: **가능하다. 단, "역할 분담"이 핵심.**

> ❗ MCP 서버 **본체**를 React/PHP로 재구현하는 것은 비추천.
> Puppeteer/CDP 생태계가 Node.js 중심이고, DevTools Frontend(TypeScript)를
> 그대로 재사용하는 게 이 프로젝트 최대 강점이기 때문.

### 권장 3-Tier 아키텍처

```
┌───────────────────────────────────────────────────────────┐
│ 🎨 프론트엔드 — React (Next.js)                            │
│    대시보드 / 리포트 뷰어 / 스크린샷 갤러리 / 차트          │
│    결제 UI / 사용자 관리 / 실시간 진행상황(WebSocket)       │
└──────────────────────────┬────────────────────────────────┘
                           │ REST / GraphQL
┌──────────────────────────▼────────────────────────────────┐
│ 🐘 백엔드 — PHP (Laravel) 또는 Node                        │
│    회원/인증 / 구독·결제 / 잡 큐 / 스케줄러 / DB           │
│    ⚠️ PHP는 크롬을 직접 만지지 않음 → 워커에 위임          │
└──────────────────────────┬────────────────────────────────┘
                           │ 큐(Redis/RabbitMQ) or HTTP
┌──────────────────────────▼────────────────────────────────┐
│ 🟢 워커 — Node.js (여기서 chrome-devtools-mcp 사용) ⭐      │
│    MCP 클라이언트 or CLI 호출 → 크롬 조종 → JSON 반환      │
└──────────────────────────┬────────────────────────────────┘
                           │
                      🌐 Chrome (headless, Docker)
```

### React로 만들 것

성능 대시보드 · 스크린샷 갤러리(before/after 슬라이더) · 테스트 리플레이 뷰어
(screencast 영상 + 타임라인) · 노코드 시나리오 빌더 · 리포트 뷰어 ·
AI 채팅 UI · WebSocket 실시간 진행상황

```jsx
const runAudit = async (url) => {
  const res = await fetch('/api/audits', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({url, checks: ['lcp', 'a11y', 'cookies']}),
  });
  const {jobId} = await res.json();
  // WebSocket 또는 폴링으로 진행상황 수신
};
```

### PHP(Laravel)로 만들 것

| 기능 | Laravel 도구 |
|---|---|
| 회원/인증/팀 | Breeze, Jetstream |
| 구독 결제 | **Cashier** (Stripe/Paddle) |
| 잡 큐 | **Queue + Horizon** |
| 정기 스케줄 | **Task Scheduling** |
| 데이터 저장 | Eloquent + MySQL/PostgreSQL |
| 알림 | Mail / Slack Notification |
| REST API | API Resources |

```php
<?php
// app/Jobs/RunBrowserAudit.php
class RunBrowserAudit implements ShouldQueue
{
    public function __construct(public Audit $audit) {}

    public function handle(): void
    {
        // PHP는 Node 워커에 위임만 한다 (크롬을 직접 만지지 않음)
        $response = Http::timeout(300)
            ->post(config('services.audit_worker.url') . '/run', [
                'url'    => $this->audit->url,
                'checks' => $this->audit->checks,
            ]);

        $this->audit->update([
            'status' => 'completed',
            'result' => $response->json(),
        ]);
    }
}
```

### Node 워커 — 방법 1: CLI 호출 (가장 쉬움)

```js
import {execFile} from 'node:child_process';
import {promisify} from 'node:util';
const run = promisify(execFile);

const {stdout} = await run('chrome-devtools', [
  'new_page', url, '--workspace', '/app/output'
]);
```

### Node 워커 — 방법 2: MCP 클라이언트 (정석)

```js
import {Client} from '@modelcontextprotocol/sdk/client/index.js';
import {StdioClientTransport} from '@modelcontextprotocol/sdk/client/stdio.js';

const transport = new StdioClientTransport({
  command: 'npx',
  args: ['-y', 'chrome-devtools-mcp@latest',
         '--headless', '--isolated', '--no-usage-statistics'],
});
const client = new Client({name: 'my-saas', version: '1.0.0'});
await client.connect(transport);

const pages = await client.callTool({name: 'list_pages', arguments: {}});
const shot  = await client.callTool({
  name: 'take_screenshot',
  arguments: {pageId: 1, filePath: '/app/output/shot.png', fullPage: true},
});
```

### 스택 선택 가이드

| 상황 | 추천 |
|---|---|
| 최단 시간 MVP | Next.js(풀스택) + chrome-devtools-mcp 직접 호출 |
| PHP 경험 많음 | Laravel(API+큐) + React(SPA) + Node 워커 |
| 대규모 확장 | React + NestJS + Node 워커 풀(Docker/K8s) + Redis |
| 비용 최소 | React(Vercel) + Node 워커(단일 VPS + Docker 크롬) |

### 서버 배포 실전 팁

```dockerfile
FROM node:22-slim

# 크롬 실행에 필요한 시스템 라이브러리 (누락 시 실행 실패)
RUN apt-get update && apt-get install -y \
    ca-certificates fonts-liberation libasound2 libatk-bridge2.0-0 \
    libatk1.0-0 libcups2 libdbus-1-3 libgbm1 libgtk-3-0 libnspr4 \
    libnss3 libxcomposite1 libxdamage1 libxrandr2 xdg-utils \
    fonts-noto-cjk \
    && rm -rf /var/lib/apt/lists/*

RUN npm i -g chrome-devtools-mcp@latest
```

| 주의 | 대응 |
|---|---|
| root 실행 금지 | 프로젝트에 "root로 실행 시 실패" 경고 존재 → 일반 유저 생성 |
| 한글 깨짐 | `fonts-noto-cjk` 설치 필수 |
| 메모리 | 크롬 1개당 300MB~1GB → 동시 실행 수 제한 |
| 보안 | 외부 URL을 여는 작업 → `--isolated` + 컨테이너 + 네트워크 제한 |
| 타임아웃 | 무거운 페이지 대비 넉넉히 설정 |
| 정리 | 잡 종료 시 `close_page` / 데몬 정리 |

---

## 12. 수익화 아이디어 9선

> ⚖️ **라이선스 조건 (Apache-2.0)**
> 가능: 상업적 이용 / 수정 / 재배포 / 특허 사용권
> 의무: 라이선스 사본 포함 + NOTICE·저작권 표기 유지 + 변경 사실 명시
> 금지: "Google" / "Chrome DevTools"를 자기 제품 브랜드처럼 사용 (상표권은 별개)
> → "Powered by Chrome DevTools MCP" ✅ / "Chrome DevTools Pro" ❌

### 1️⃣ AI 웹 QA 자동화 SaaS ⭐⭐⭐⭐⭐

**컨셉**: "URL과 로그인 정보만 주세요. AI가 전부 테스트하고 버그 리포트를 드립니다."

**흐름**
1. URL + (선택)테스트 계정 등록
2. AI가 크롤링하며 시나리오 자동 발견 (`take_snapshot`으로 폼/버튼 구조 파악)
3. 시나리오 실행 (회원가입/로그인/검색/장바구니/결제/문의)
4. 콘솔 에러 + 네트워크 실패(4xx/5xx) + 스크린샷 + 녹화 수집
5. AI가 리포트 작성 → 슬랙/이메일 알림 + 대시보드

**사용 도구**: `new_page` → `take_snapshot` → `fill_form` → `click` → `wait_for`
→ `list_console_messages` → `list_network_requests` → `take_screenshot` → `screencast_*`

| 플랜 | 가격/월 | 포함 |
|---|---|---|
| Free | ₩0 | 사이트 1개, 주 1회, 시나리오 3개 |
| Starter | ₩49,000 | 사이트 3개, 매일, 시나리오 20개 |
| Pro | ₩199,000 | 사이트 10개, 배포마다, 무제한, 슬랙연동 |
| Business | ₩690,000 | 무제한, 전용 워커, API, SSO |
| Enterprise | 협의 | 온프레미스, SLA |

**차별점 (Playwright/Cypress 대비)**

| 기존 E2E | 우리 |
|---|---|
| 테스트 코드를 사람이 작성 | AI가 자동 생성 |
| UI 변경 시 전부 깨짐 | 스냅샷 기반 자가 적응 |
| 개발자만 사용 가능 | 기획자/PM도 사용 |
| 실패 원인 직접 분석 | AI가 원인+해결책 제시 |

난이도 ⭐⭐⭐ | 수익성 💰💰💰💰💰

---

### 2️⃣ 코어 웹 바이탈 자동 최적화 서비스 ⭐⭐⭐⭐⭐

**컨셉**: "진단만이 아니라, 고치는 PR까지 자동 생성"

기존 서비스(PageSpeed Insights, GTmetrix, Lighthouse CI)는 "느리다"고 알려줄 뿐.
`skills/debug-optimize-lcp`를 활용하면 **정확한 병목 + 실제 수정 코드**까지 가능.

**흐름**
1. GitHub App으로 저장소 연동
2. PR 발생 시 자동 트리거
3. 프리뷰 URL에서 `performance_start_trace(reload)` →
   `performance_analyze_insight("LCPBreakdown")` → `lighthouse_audit()` →
   `list_network_requests()`
4. 병목 특정: "LCP 4.1초 중 3.2초가 hero.png(4.2MB) 다운로드"
5. **수정 PR 자동 생성**: WebP/AVIF 변환, `fetchpriority="high"`,
   `<link rel="preload">`, 폰트 preconnect + `display:swap`, JS defer/async
6. PR 코멘트: "이 수정으로 LCP 4.1s → 1.3s (예상)"

| 플랜 | 가격/월 |
|---|---|
| 개인/오픈소스 | ₩0 (공개 저장소) |
| Starter | ₩39,000 (저장소 3개) |
| Team | ₩149,000 (저장소 10개, 자동 PR) |
| Agency | ₩490,000 (클라이언트 관리, 화이트라벨) |

**추가 수익**: 일회성 감사 리포트 ₩300,000~1,000,000 /
최적화 시공 대행 ₩2,000,000~ / 경쟁사 벤치마크 월 ₩200,000

**한국 시장 포인트**: 검색 순위에 코어 웹 바이탈이 반영 → "속도 = 매출" 메시지가 통함

난이도 ⭐⭐⭐⭐ | 수익성 💰💰💰💰💰

---

### 3️⃣ 웹 접근성 컴플라이언스 SaaS ⭐⭐⭐⭐⭐

**컨셉**: "웹 접근성 인증마크, AI가 준비부터 유지까지"

**한국 시장 근거**
- 장애인차별금지법 → 웹 접근성 준수가 **법적 의무**
- 웹 접근성 인증마크 → 공공기관·금융·대기업 갱신 필요 (유효기간 1년)
- 현재 시세: 컨설팅 수백만~수천만 원, 매년 반복
- 해외: 미국 ADA 소송 리스크, 유럽 EAA(2025 시행) → 글로벌 확장 가능

**흐름** (`skills/a11y-debugging` 활용)
1. `lighthouse_audit(categories: ['accessibility'])` — 자동 점수 + 위반 목록
2. `take_snapshot` — 접근성 트리 직접 분석 (ARIA 누락, 시맨틱 오류)
3. `press_key('Tab')` 반복 — 포커스 순서, 포커스 트랩, 스킵 링크 검증
4. `evaluate_script` — 탭 타겟 크기(44×44px), 색 대비 계산
5. `emulate` — 모바일/200% 확대 상태 검증
6. KWCAG 2.2 / WCAG 2.2 체크리스트 매핑
7. 리포트 + 수정 코드 제안 + 재검사

| 상품 | 가격 |
|---|---|
| 1회 진단 리포트 | ₩500,000 ~ 2,000,000 |
| 월간 모니터링 | ₩99,000 ~ 490,000/월 |
| 인증 준비 패키지 | ₩3,000,000 ~ 10,000,000 |
| 개발사 화이트라벨 | 협의 |

**타겟**: 공공기관 / 금융사 / 대학 / 병원 / 대기업 / **웹 에이전시(B2B2C 레버리지)**

난이도 ⭐⭐⭐ | 수익성 💰💰💰💰 | 경쟁 🟢 낮음

---

### 4️⃣ 쿠키·개인정보 규제 감사 서비스 ⭐⭐⭐⭐⭐

**컨셉**: "우리 사이트, 개인정보보호법 위반 아닌가요?" → AI 자동 검증

**시장 근거**: 개인정보보호법 강화(과징금 리스크), GDPR, 제3자 쿠키 종료 대응
(SameSite/Partitioned/CHIPS). **동의 배너가 실제로 작동하는지** 검증하는 서비스는 희소.

**흐름** (`skills/cookie-debugging` 활용)
1. `new_page(isolatedContext: "consent-audit-1")` — 완전 깨끗한 상태
2. 배너 표시 전 쿠키 확인 → "동의 전에 추적 쿠키를 심었는가?"
3. "거부" 클릭 → 추적 쿠키 실제 차단 여부 검증
4. "동의" 클릭 → 쿠키 정상 설정 여부
5. `list_network_requests` → GA/픽셀/광고 태그 발사 여부
6. `Set-Cookie` 헤더 전수 검사 (Secure/HttpOnly/SameSite/Partitioned)
7. 제3자 쿠키 목록 + 국외 이전 여부 → 법적 리스크 리포트

| 상품 | 가격 |
|---|---|
| 무료 스캔 (리드마그넷) | ₩0 — 점수만 공개, 상세는 유료 |
| 상세 리포트 | ₩300,000/회 |
| 월간 모니터링 | ₩99,000 ~ 290,000/월 |
| 법무 연계 컨설팅 | 제휴 수수료 |

**그로스 전략**: "무료 쿠키 스캐너" 랜딩 → 낮은 점수 → 불안 → 유료 전환

난이도 ⭐⭐ | 수익성 💰💰💰💰 | 진입 🟢 쉬움

---

### 5️⃣ 경쟁사 모니터링 / 가격 추적 SaaS ⭐⭐⭐⭐

**흐름**
1. 감시할 페이지 등록
2. 매일 스케줄 실행: `new_page` → `take_snapshot` → `evaluate_script`로 데이터 추출
3. 이전 스냅샷과 diff
4. 변동 감지 시 슬랙/이메일/카톡 알림
5. `take_screenshot`으로 증거 보관 → 추이 그래프(React)

**장점**: 리텐션이 매우 높음. 이커머스/호텔/항공/렌터카 필수.
스냅샷 기반이라 경쟁사 UI 변경에도 강함.

가격: ₩49,000 ~ 990,000/월
⚠️ robots.txt 존중, 과도한 크롤링 금지, 서비스 약관 확인

난이도 ⭐⭐ | 수익성 💰💰💰💰

---

### 6️⃣ AI 에이전트용 "브라우저 API" 판매 ⭐⭐⭐

Browserbase / Browserless 모델.

```
POST   /v1/sessions            → 격리 크롬 세션 생성
POST   /v1/sessions/{id}/tools → MCP 도구 호출 (50종)
GET    /v1/sessions/{id}/logs  → 콘솔/네트워크 로그
DELETE /v1/sessions/{id}       → 정리
```
+ 원격 MCP 엔드포인트 자체를 SaaS로 제공

가격: 사용량 기반 — ₩50/세션-분 또는 ₩10,000/1,000 도구호출
타겟: AI 스타트업, 에이전트 개발자, 자동화 SaaS 업체
⚠️ 인프라(K8s, 크롬 풀), 보안 격리, 비용 관리가 관건

난이도 ⭐⭐⭐⭐⭐ | 수익성 💰💰💰💰💰

---

### 7️⃣ 노코드 웹 자동화 툴 ⭐⭐⭐

"Zapier인데 브라우저를 직접 조종하는 버전" — **API 없는 사이트도 자동화 가능**

활용 예: 매일 여러 사이트 로그인 → 데이터 수집 → 스프레드시트 정리 /
정부 공고 모니터링 / 쇼핑몰 재고 확인 / 리뷰 수집 및 감성분석 / 반복 서류 제출

가격: ₩29,000 ~ 290,000/월 (실행 횟수 기준)

난이도 ⭐⭐⭐⭐ (UX 난이도 높음) | 수익성 💰💰💰💰

---

### 8️⃣ 교육 & 콘텐츠 & 컨설팅 ⭐⭐⭐⭐⭐ (자본금 ≈ 0)

| 상품 | 가격 |
|---|---|
| 온라인 강의 (인프런/유데미) | ₩55,000 ~ 150,000 |
| 전자책 / 노션 템플릿 | ₩15,000 ~ 50,000 |
| 기업 구축 컨설팅 | 일 ₩500,000 ~ 1,500,000 |
| 기업 출강 교육 | 일 ₩1,000,000 ~ 3,000,000 |
| 유료 뉴스레터 | 월 ₩9,900 |
| 프리미엄 스킬팩 | ₩50,000 ~ 200,000 |
| 유튜브 / 블로그 | 광고 + 제휴 (리드 확보용) |

**조합**: 콘텐츠(무료) → 강의(저가) → 컨설팅(고가) → SaaS(확장)

---

### 9️⃣ 버티컬 특화 상품 ⭐⭐⭐⭐

| 아이디어 | 설명 | 가격 |
|---|---|---|
| 쇼핑몰 결제 모니터링 | 결제 플로우 집중 감시, 장애 즉시 알림 | ₩99,000~/월 |
| 이메일 렌더링 테스트 | 뷰포트별 뉴스레터 렌더링 검증 | ₩49,000~/월 |
| 다국어 사이트 QA | 언어별 레이아웃 깨짐, 번역 누락 | ₩149,000~/월 |
| 디자인 QA (Visual Regression) | 배포 전후 픽셀 비교 | ₩99,000~/월 |
| **PWA 검증** | `install_pwa` 활용, 설치/오프라인/매니페스트 | ₩79,000~/월 |
| **크롬 익스텐션 QA** | `install_extension` 활용 (경쟁자 희소) | ₩149,000~/월 |
| **메모리 누수 감시** | 힙 스냅샷 13종 활용 (경쟁자 희소) | ₩199,000~/월 |
| SEO 렌더링 검증 | JS 렌더링 후 메타/구조화데이터 검증 | ₩79,000~/월 |

---

### 종합 비교표

| # | 아이디어 | 난이도 | 초기비용 | 수익성 | 시장 | 추천 |
|---|---|:---:|:---:|:---:|:---:|:---:|
| 1 | AI 웹 QA SaaS | ⭐⭐⭐ | 중 | 💰💰💰💰💰 | 🌍🌍🌍 | ⭐⭐⭐⭐⭐ |
| 2 | CWV 자동 최적화 | ⭐⭐⭐⭐ | 중 | 💰💰💰💰💰 | 🌍🌍🌍 | ⭐⭐⭐⭐⭐ |
| 3 | 접근성 컴플라이언스 | ⭐⭐⭐ | 낮 | 💰💰💰💰 | 🌍🌍 | ⭐⭐⭐⭐⭐ |
| 4 | 쿠키 규제 감사 | ⭐⭐ | 낮 | 💰💰💰💰 | 🌍🌍 | ⭐⭐⭐⭐⭐ |
| 5 | 경쟁사 모니터링 | ⭐⭐ | 낮 | 💰💰💰💰 | 🌍🌍🌍 | ⭐⭐⭐⭐ |
| 6 | 브라우저 API 판매 | ⭐⭐⭐⭐⭐ | 높 | 💰💰💰💰💰 | 🌍🌍🌍 | ⭐⭐⭐ |
| 7 | 노코드 자동화 | ⭐⭐⭐⭐ | 중 | 💰💰💰💰 | 🌍🌍🌍 | ⭐⭐⭐ |
| 8 | 교육/컨설팅 | ⭐ | ≈0 | 💰💰💰 | 🌍🌍 | ⭐⭐⭐⭐⭐ |
| 9 | 버티컬 특화 | ⭐⭐ | 낮 | 💰💰💰 | 🌍 | ⭐⭐⭐⭐ |

---

## 13. 실행 로드맵

### 0~1개월 — 씨앗 뿌리기 🌱
- 직접 사용하며 블로그/유튜브 콘텐츠 제작
- "무료 쿠키 스캐너" 같은 소형 툴 런칭 (React + Node)
- 이메일 리스트 확보
- 예상 수익: ₩0 ~ 500,000

### 1~3개월 — 첫 수익 💵
- 접근성 진단 리포트를 수동+AI 혼합으로 판매 (건당 ₩500,000)
- 지인 / 중소 에이전시 영업
- 전자책 또는 인프런 강의 출시
- 예상 수익: ₩1,000,000 ~ 5,000,000/월

### 3~6개월 — SaaS 전환 🏗️
- #4(쿠키) 또는 #3(접근성)을 SaaS로 자동화
- Laravel/Next.js + Node 워커 구축
- 첫 유료 고객 10~30명
- 예상 수익: ₩3,000,000 ~ 10,000,000/월

### 6~12개월 — 확장 📈
- #1(QA SaaS) 또는 #2(성능 최적화)로 확장
- 에이전시 화이트라벨 파트너십
- 해외(영어) 버전 런칭
- 예상 수익: ₩10,000,000+/월

> 💡 추천 순서: **#4 쿠키 감사 → #3 접근성 → #1 QA SaaS**
> (진입장벽이 낮은 것부터 단계적으로)

---

## 14. 주의사항 정리

### 사용 시

| 주의 | 대응 |
|---|---|
| 🔐 개인정보 노출 | 브라우저 내용이 AI에 전달됨 → `--isolated` 권장 |
| 📊 사용 통계 기본 ON | `--no-usage-statistics` 또는 환경변수로 차단 |
| 🌍 CrUX API 통신 | `--no-performance-crux` |
| 💻 크롬 전용 | Chrome / Chrome for Testing만 공식 지원 |
| 📦 Node LTS 필수 | 20.19+ / 22.12+ / 23+ |
| 🚫 root 실행 금지 | 실행 실패 원인 |

### 수익화 시

| 항목 | 내용 |
|---|---|
| ⚖️ 라이선스 | Apache-2.0 사본 + NOTICE 포함, 변경사항 명시 |
| 🏷️ 상표권 | "Google"/"Chrome" 브랜드 오용 금지. "Powered by" 수준으로 |
| 🔐 보안/프라이버시 | 고객 사이트 데이터 = 민감정보. 격리·암호화·삭제정책 |
| 📊 텔레메트리 | 상용 서비스는 `--no-usage-statistics` 필수 |
| 🤖 크롤링 윤리 | robots.txt 존중, 과부하 금지 |
| ⚡ 비용 관리 | 크롬은 메모리 소모 큼 → 동시 실행 제한 + 자동 정리 |
| 🏛️ 법적 검토 | 규제 관련 서비스는 변호사 검토 권장 |
| 📉 의존성 리스크 | 구글 정책 변경 대비 → 추상화 레이어 |

---

## 📝 부록: 개발 규칙 (기여 시 참고)

`AGENTS.md` 기준:

**TypeScript 금지 사항**
- `any` 타입 금지
- `as` 타입 캐스팅 금지
- `!` 타입 단언 금지
- `@ts-ignore`, `@ts-nocheck`, `@ts-expect-error` 금지
- `forEach` 대신 `for..of` 선호

**명령어**
```bash
npm run build          # tsc + 빌드
npm run test           # 빌드 + 전체 테스트
npm run test path.ts   # 단일 테스트 파일
npm run format         # eslint --fix + prettier
npm run typecheck      # 타입 체크만
npm run gen            # 빌드 + CLI/문서 생성 + 포맷
npm run eval           # Gemini 기반 AI 평가 (GEMINI_API_KEY 필요)
npm run profile        # 메모리 프로파일링
```

**테스트 원칙**
- 실제 브라우저 대신 **목(mock) 기반 단위테스트 우선**
- 목은 `tests/mocks.ts`에 중앙 집중
- `sinon.createStubInstance(Class)` 사용, 직접 목 객체 작성 금지
- `sinon.assert.calledOnceWithExactly()` 등 sinon assert 사용
- `afterEach(() => sinon.restore())` 필수
- `third_party/devtools-frontend`는 서브모듈이므로 수정 금지

---

## ✅ 요약 한 장

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  Chrome DevTools MCP = 구글 크롬 팀 공식 프로젝트         │
│  "AI에게 진짜 크롬 브라우저를 조종하게 해주는 MCP 서버"    │
│                                                          │
│  🖐️ 손 (클릭/입력)      👀 눈 (스냅샷/스크린샷)           │
│  👂 귀 (콘솔)           🩺 청진기 (네트워크)              │
│  📊 MRI (성능)          🧬 혈액검사 (메모리)              │
│  📖 교과서 (스킬 7종)                                     │
│                                                          │
│  → AI가 "추측"에서 "검증"으로 진화                        │
│  → Apache-2.0, 토큰 불필요, 모든 AI 툴 지원              │
│  → 로컬 에이전트 구축의 최고 레퍼런스 + 수익화 재료        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

*Generated by 카리나 ✨ — Claude Code AI 개발 파트너*
