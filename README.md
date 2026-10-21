# MashNote — MCP 연결 가이드

MashNote는 AI와의 대화 요약을 활동 피드에 기록하는 **원격 HTTP MCP 커넥터**입니다.
MCP를 지원하는 클라이언트(Claude · Codex · ChatGPT · Gemini 등)면 어디서든 붙일 수 있어요.

## 주소 = 워크스페이스

MashNote 커넥터 주소는 **워크스페이스마다 하나**입니다.

```text
https://www.mashnote.app/mcp/<workspaceId>
```

- 주소는 MashNote 앱의 **설정 → 연동 → MCP 연동**에서 워크스페이스를 고르면 그대로 복사할 수
  있습니다(다른 배포를 쓰면 그 앱 도메인). 직접 조합하지 않아도 됩니다.
- 그 주소로 연결한 커넥터는 **그 워크스페이스에, 멤버에게 보이는 팀 활동으로** 기록합니다.
  모델이 어디에 기록할지 고르거나 묻지 않고, `workspaceId` 같은 인자도 없습니다.
  항목별 공개 범위는 앱의 내 매시에서 바꿀 수 있어요.
- 워크스페이스가 여러 개면 커넥터도 여러 개 추가합니다. 연결 한 번 = 워크스페이스 하나.
- 연결하려면 그 워크스페이스의 **정회원**이어야 합니다(게스트는 연결할 수 없어요).
- **활동 도구:** `log_activity`(요약 기록) · `list_recent_activity`(이 워크스페이스의 내 최근 기록)
- **노트 도구(승인 시):** `list_workspaces` · `list_folders` · `list_notes` · `search_notes` · `get_note`
  — 노트 권한도 연결한 워크스페이스 하나에만 적용됩니다.

> 예전 개인 주소(`https://www.mashnote.app/mcp`)는 **더 이상 동작하지 않습니다.** 그 주소로
> 맺은 연결과 토큰은 모두 해제됐으니 워크스페이스 주소로 다시 연결하세요. 개인 액세스
> 토큰(PAT)은 그대로 유효하며 워크스페이스 주소에 그대로 쓸 수 있습니다.

## 인증 — OAuth vs 토큰

연결 방식은 두 가지입니다. 클라이언트가 지원하는 쪽을 쓰세요.

- **OAuth (브라우저 로그인)** — claude.ai 웹·데스크탑, Claude Code, Codex, Gemini CLI, ChatGPT가 지원.
  동의 화면에 어느 워크스페이스에 연결하는지 표시됩니다.
- **개인 액세스 토큰(PAT)** — 브라우저 로그인이 어려운 원격/CI나, 토큰 입력을 선호하는 클라이언트용.
  - MashNote 앱 → **설정 → 연동 → MCP 연동 → "토큰 발급"** 에서 발급. 토큰은 **발급 시 한 번만**
    표시되니 복사해 두세요. 토큰은 활동 기록 전용이며 노트 권한은 주지 않습니다.

---

## claude.ai 웹 · 데스크탑

1. Claude 설정 → **커넥터** → 커스텀 커넥터 추가
2. 워크스페이스 **주소** 붙여넣기(워크스페이스마다 커넥터 하나)
3. 브라우저에서 로그인(OAuth)

## Claude Code

MCP 서버는 **프로젝트 범위**로 추가하는 걸 권장합니다. `.mcp.json`에 저장돼 같은 저장소를
여는 팀원도 같은 워크스페이스에 기록합니다. 서버 이름은 워크스페이스별로 다르게 붙이세요
(앱의 설정 카드가 이름과 명령을 만들어 줍니다).

```bash
claude mcp add --transport http --scope project mashnote-<workspace> https://www.mashnote.app/mcp/<workspaceId>
```

새 세션에서 **`/mcp`로 인증**(브라우저 OAuth 1회)하세요.

`/mashnote:log` 커맨드(현재 대화 요약 기록)는 플러그인으로 설치합니다. 플러그인은 커맨드만
제공하고 MCP 서버는 포함하지 않으니, 위 서버 추가는 따로 해야 합니다.

```text
/plugin marketplace add https://github.com/lymeric-ai/mashnote-plugin.git
/plugin install mashnote@mashnote
```

## Codex CLI

워크스페이스 주소를 이름 붙인 MCP 서버로 추가하고 로그인합니다(설정은 `~/.codex/config.toml`).

```bash
codex mcp add mashnote-<workspace> --url https://www.mashnote.app/mcp/<workspaceId>
codex mcp login mashnote-<workspace>   # 브라우저 로그인
```

토큰으로 쓰려면:

```bash
export MASHNOTE_TOKEN=mn_pat_....   # 앱 설정 → 연동 탭에서 발급
codex mcp add mashnote-<workspace> --url https://www.mashnote.app/mcp/<workspaceId> \
  --bearer-token-env-var MASHNOTE_TOKEN
```

`codex mcp list`에서 로그인 상태를 확인하세요. 이 설정은 Codex CLI·IDE 확장·ChatGPT 데스크톱 앱의
Codex가 함께 읽습니다(앱에서는 Settings → MCP servers, 대화창 `/mcp`, Authenticate 버튼). 저장소 하나에
워크스페이스를 고정하려면 같은 `[mcp_servers.<name>]` 블록을 그 저장소의 `.codex/config.toml`에 넣으세요
(trusted project에서만 읽힙니다).

> 0.1.2까지 있던 Codex 플러그인 마켓플레이스는 더 이상 제공하지 않습니다. 플러그인에 묶인
> 서버 주소가 개인 주소였기 때문입니다. 설치돼 있다면 제거하고 위 방법으로 추가하세요.

## ChatGPT (사용자 지정 MCP 서버)

> 웹(chatgpt.com)에서만 만들 수 있고, 만든 뒤에는 같은 계정의 데스크톱 앱에서도 보입니다.
> 쓰기 도구(`log_activity`)까지 쓰려면 Business · Enterprise · Education(관리자)이 필요하고,
> Pro는 읽기 도구만 됩니다. 팀원에게 배포하려면 관리자가 워크스페이스에 게시해야 합니다.

1. **Plugins → + → Add custom MCP server** (예전 이름: Apps → Advanced settings → Developer mode → Create app)
2. 워크스페이스 **주소** 붙여넣기(워크스페이스마다 하나)
3. 인증: **OAuth**(브라우저 로그인) 또는 **토큰**(발급한 PAT 붙여넣기)
4. 도구 스캔 뒤 만들기. 노트 읽기를 쓰려면 승인 화면에서 해당 권한을 승인하세요.

전송은 Streamable HTTP + HTTPS입니다.

## Antigravity (앱 · IDE · CLI `agy`)

> 2026-06-18부터 Gemini CLI는 Google AI Pro/Ultra·무료 개인 사용자에게 서비스를 중단하고
> Antigravity CLI로 대체됐습니다. 개인 사용자는 Antigravity를 쓰세요.

`~/.gemini/config/mcp_config.json`(앱·IDE·CLI 공용, 모든 프로젝트) 또는 저장소의
`.agents/mcp_config.json`(그 저장소만, 워크스페이스 고정)에 추가합니다. 원격 서버 키는 `serverUrl`.
앱에서는 Settings → Customizations → **Open MCP Config**로 같은 파일이 열립니다.

```json
{
  "mcpServers": {
    "mashnote-<workspace>": {
      "serverUrl": "https://www.mashnote.app/mcp/<workspaceId>"
    }
  }
}
```

저장 뒤 **Authenticate**를 누르면 브라우저 OAuth가 자동으로 진행됩니다(동적 클라이언트 등록 지원).
OAuth가 막히는 환경이면 `"headers": { "Authorization": "Bearer mn_pat_...." }`를 넣어 토큰으로 쓰세요.

## Gemini CLI (레거시 — Code Assist Standard/Enterprise·유료 API 키 사용자)

```bash
gemini mcp add --transport http --scope user mashnote-<workspace> https://www.mashnote.app/mcp/<workspaceId>
```

`--scope project`로 저장소에 고정할 수 있고, 토큰을 쓰려면 `-H "Authorization: Bearer mn_pat_...."`를
덧붙이세요. 첫 호출에서 401을 받으면 OAuth 로그인이 자동으로 뜹니다.

> 0.1.2까지 있던 Gemini extension(`gemini extensions install …`)은 개인 주소에 묶여 있어 제거했습니다.

---

## 사용

### 자동 기록 — 대부분 이걸로 충분합니다

앱의 **설정 → 연동 → MCP 연동**에서 워크스페이스를 고르면 그 워크스페이스용 **표준 지침**이
생성됩니다. 복사해서 사용하는 클라이언트의 지침 칸(ChatGPT 맞춤 설정 · Claude 프로필
환경설정 · 프로젝트 `CLAUDE.md`/`AGENTS.md` 등)에 한 번 붙여넣으세요. 그러면 모델이 의미
있는 작업을 끝낼 때마다 **알아서 기록을 먼저 제안**합니다. 커넥터가 여러 개 붙어 있고
사용자가 워크스페이스를 말하지 않았을 때만 한 번 물어봅니다.

### 수동 기록 — 직접 남기고 싶을 때

- **`/mashnote:log`** (Claude Code 플러그인) — 현재 대화를 요약해 활동 피드에 기록
  - `/mashnote:log research 워크스페이스에` 처럼 어느 커넥터를 쓸지 힌트를 덧붙일 수 있어요.
- 또는 자연어로 **"이 대화 매시노트에 기록해줘"** — 같은 `log_activity` 도구를 호출합니다.

원문 대화는 서버로 보내지 않고, 요약(`title` · `summary` · `points` · `detail`)만 기록합니다.

### 기존 설치 갱신 (0.1.3)

- 플러그인에서 **MCP 서버를 뺐습니다.** `/plugin marketplace update mashnote` 뒤 플러그인을
  업데이트하고, 위 Claude Code 방법으로 워크스페이스 서버를 프로젝트에 추가하세요.
- 예전 개인 주소로 맺은 연결은 서버에서 해제됐습니다. 각 클라이언트에서 그 커넥터를 지우고
  워크스페이스 주소로 다시 추가하세요.
- 이미 붙여넣은 표준 지침에 "`workspaceId`를 생략/지정" 같은 문구가 남아 있으면 앱에서
  새 지침을 다시 복사해 교체하세요.
