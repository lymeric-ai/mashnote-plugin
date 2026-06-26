# MashNote — MCP 연결 가이드

MashNote는 AI와의 대화 요약을 활동 피드에 기록하는 **원격 HTTP MCP 커넥터**입니다.
MCP를 지원하는 클라이언트(Claude · Codex · ChatGPT 등)면 어디서든 붙일 수 있어요.

- **커넥터 URL:** `https://www.mashnote.app/mcp`
  (다른 배포를 쓰면 그 앱 도메인의 `/mcp`로 바꾸세요.)
- **도구:** `list_workspaces`(기록 전 워크스페이스 확인) · `log_activity`(요약 기록)
- 기록할 워크스페이스는 기록 시점에 클라이언트가 물어봅니다(워크스페이스가 하나면 생략).

## 인증 — OAuth vs 토큰

연결 방식은 두 가지입니다. 클라이언트가 지원하는 쪽을 쓰세요.

- **OAuth (브라우저 로그인)** — claude.ai 웹·데스크탑, Claude Code, Codex, Gemini CLI, ChatGPT가 지원. 브라우저로 한 번 로그인하면 끝.
- **개인 액세스 토큰(PAT)** — 브라우저 로그인이 어려운 원격/CI나, 토큰 입력을 선호하는 클라이언트용.
  - MashNote 앱 → **설정 → 연동 → MCP 연동 → "토큰 발급"** 에서 발급. 토큰은 **발급 시 한 번만** 표시되니 복사해 두세요.

---

## claude.ai 웹 · 데스크탑

1. Claude 설정 → **커넥터** → 커스텀 커넥터 추가
2. 위 **커넥터 URL** 붙여넣기
3. 브라우저에서 로그인(OAuth)

## Claude Code

한 줄 설치(플러그인) — 커넥터와 `/mashnote:log` 커맨드가 같이 들어옵니다:

```text
/plugin marketplace add https://github.com/lymeric-ai/mashnote-plugin.git
/plugin install mashnote@mashnote
```

설치 후 새 세션에서 커넥터가 뜨면 **`/mcp`로 인증**(브라우저 OAuth 1회)하세요.

> 플러그인 자동 연결이 안 되면(클라이언트 버전 차이) 수동으로도 됩니다:
> `claude mcp add --transport http mashnote https://www.mashnote.app/mcp`

## Codex CLI

**플러그인 설치 — 권장(OAuth):**

```bash
codex plugin marketplace add lymeric-ai/mashnote-plugin
codex plugin add mashnote@mashnote
```

설치할 때 브라우저 OAuth로 인증됩니다. 자동으로 안 뜨면 `codex mcp login mashnote`로 인증하세요.

플러그인 경로가 안 되는 버전이면 **수동**으로 추가하세요 — 설정은
`~/.codex/config.toml`의 `[mcp_servers.<name>]`에 저장됩니다.

**수동, 토큰(PAT)으로:**

```bash
export MASHNOTE_TOKEN=mn_pat_....   # 앱 설정 → 연동 탭에서 발급
codex mcp add mashnote --url https://www.mashnote.app/mcp \
  --bearer-token-env-var MASHNOTE_TOKEN
```

생성되는 설정:

```toml
[mcp_servers.mashnote]
url = "https://www.mashnote.app/mcp"
bearer_token_env_var = "MASHNOTE_TOKEN"
```

**OAuth로:**

```bash
codex mcp add mashnote --url https://www.mashnote.app/mcp
codex mcp login mashnote   # 브라우저 로그인
```

> 구버전 Codex나 stdio만 지원하는 클라이언트라면 `npx mcp-remote`로 원격 서버를
> 로컬 stdio로 브리지할 수 있습니다:
> `codex mcp add mashnote -- npx -y mcp-remote https://www.mashnote.app/mcp --header "Authorization: Bearer mn_pat_..."`

## Gemini CLI

**Extension 설치 — 권장(OAuth, 토큰 불필요):**

```bash
gemini extensions install https://github.com/lymeric-ai/mashnote-plugin
```

설치 후 첫 사용 시 브라우저 OAuth로 인증됩니다. 토큰을 선호하면 `~/.gemini/settings.json`의
`mcpServers`에 직접 넣어도 됩니다(원격 서버 키는 `httpUrl`):

```json
{
  "mcpServers": {
    "mashnote": {
      "httpUrl": "https://www.mashnote.app/mcp",
      "headers": { "Authorization": "Bearer mn_pat_...." }
    }
  }
}
```

## Antigravity

마켓플레이스/원클릭 발행 경로가 없어 **수동 설정**입니다. Settings → Customizations →
**Open MCP Config**(`~/.gemini/config/mcp_config.json`)를 열어 추가하세요(원격 서버 키는
`serverUrl`):

```json
{
  "mcpServers": {
    "mashnote": {
      "serverUrl": "https://www.mashnote.app/mcp",
      "headers": { "Authorization": "Bearer mn_pat_...." }
    }
  }
}
```

원격 MCP의 OAuth에 알려진 런타임 버그가 있어 **PAT(헤더) 방식을 권장**합니다. Antigravity는
Gemini CLI와 `~/.gemini/config`를 공유하므로, 위 Gemini extension이 설치돼 있으면 그대로
인식될 수도 있습니다(버전에 따라 다름).

## ChatGPT (Developer Mode)

> Plus · Pro · Business · Enterprise · Education 플랜에서, 웹에서만 가능합니다(무료 티어 불가).

1. 설정 → **Apps → Advanced settings → Developer mode** 켜기
2. **Create app** → 커넥터 URL 붙여넣기
3. 인증: **OAuth**(브라우저 로그인) 또는 **토큰**(발급한 PAT 붙여넣기) 선택
4. 사용할 도구(`log_activity` · `list_workspaces`)를 활성화

전송은 Streamable HTTP + HTTPS이며, `log_activity`(쓰기) 도구를 그대로 쓸 수 있습니다(search/fetch 도구 불필요).

---

## 사용

### 자동 기록 — 대부분 이걸로 충분합니다

앱의 **설정 → 연동 → 어시스턴트 표준 지침**에 있는 안내문을 복사해, 사용하는 클라우드
LLM의 표준 지침 칸(ChatGPT 맞춤 설정 · Claude 프로필 환경설정 등)에 한 번 붙여넣으세요.
그러면 모델이 의미 있는 작업을 끝낼 때마다 **알아서 기록을 먼저 제안**하니, 대부분은
이 자동 기록만으로 충분합니다. 아래 수동 명령은 명시적으로 남기고 싶을 때만 쓰면 됩니다.

### 수동 기록 — 직접 남기고 싶을 때

- **`/mashnote:log`** (Claude Code) — 현재 대화를 요약해 활동 피드에 기록
  - `/mashnote:log research 워크스페이스에` 처럼 힌트를 덧붙일 수 있어요.
- 또는 자연어로 **"이 대화 매시노트에 기록해줘"** — 같은 `log_activity` 도구를 호출합니다.

원문 대화는 서버로 보내지 않고, 요약(`title` · `summary` · `points` · `detail`)만 기록합니다.
