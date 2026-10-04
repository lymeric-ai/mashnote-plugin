# MashNote — MCP 연결 가이드

MashNote는 AI와의 대화 요약을 활동 피드에 기록하는 **원격 HTTP MCP 커넥터**입니다.
MCP를 지원하는 클라이언트(Claude · Codex · ChatGPT 등)면 어디서든 붙일 수 있어요.

- **커넥터 URL:** `https://www.mashnote.app/mcp`
  (다른 배포를 쓰면 그 앱 도메인의 `/mcp`로 바꾸세요.)
- **활동 도구:** `log_activity`(요약 기록) · `list_recent_activity`(현재 경로의 최근 기록)
- **노트 도구:** `list_workspaces` · `list_folders` · `list_notes` · `search_notes` · `get_note`
- **기본 기록은 워크스페이스 선택 없이** `workspaceId`를 생략합니다. MashNote의 내 매시에서
  저장한 출처별 워크스페이스 연결을 적용하며, 연결이 없으면 개인 활동으로 남습니다.
  최근 기록 조회도 같은 기본 경로를 따릅니다.
- 현재 사용자나 프로젝트 지침이 명시한 목적지만 직접 지정합니다. 대화 주제·읽은 노트·
  노트 권한 또는 워크스페이스 목록으로 목적지를 추측하지 않습니다. 분류는 기록의 필수 조건이 아닙니다.
- 기록 결과의 실제 목적지와 `visibility`를 확인하세요. 워크스페이스 연결과 팀 공개는 별도입니다.
  서버 0.2.3부터 실제 목적지·출처·공개 범위 설정을 응답합니다. 기존 기록은 이동하지 않습니다.

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

## Codex 앱 · CLI

플러그인 **0.1.1부터** Codex용 MCP 설정 형식을 수정했습니다. 0.1.0은 설치된 것으로
보여도 MCP 로딩 중 `invalid transport` 오류가 나서 도구와 OAuth 승인 화면이 나타나지
않을 수 있습니다. 플러그인 버전은 MashNote 서버 버전과 별개입니다.

기존 설치는 마켓플레이스를 새로고침한 뒤 플러그인을 업데이트하거나 다시 설치하고,
앱을 재시작해 새 대화에서 확인하세요. CLI에서는 다음과 같이 소스를 새로고침합니다:

```bash
codex plugin marketplace upgrade mashnote
```

연결 시 노트 읽기를 승인하려면 OAuth 화면에서 `notes:read`와 허용할 워크스페이스를
선택해야 합니다. 연결 후 `list_workspaces`의 `canReadNotes`를 확인하고 노트 목록·본문을
읽어 보세요. 개인 액세스 토큰(PAT)은 활동 기록 전용이며 노트 권한을 부여하지 않습니다.

**마켓플레이스 추가 후 플러그인 설치(OAuth):**

```bash
codex plugin marketplace add 'https://github.com/lymeric-ai/mashnote-plugin.git'
codex plugin add mashnote@mashnote
```

앱에서도 플러그인 마켓플레이스에 위 저장소를 추가한 뒤 MashNote 플러그인을 설치할 수
있습니다. 로그인(OAuth)하고 노트 읽기를 사용할 워크스페이스를 승인하세요. 설치 후 앱을
재시작하고 새 대화에서 `@MashNote`를 선택해 도구가 보이는지 확인합니다.

이 안내는 마켓플레이스 → 플러그인 설치 경로를 사용합니다. MCP 서버만 직접 등록하는
명령(`codex mcp add`)은 플러그인 설치나 대화의 `@MashNote` 선택을 대체하지 않습니다.

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

### 기존 설치의 지침 갱신

플러그인 0.1.2는 `/mashnote:log`의 필수 워크스페이스 선택과 기존 기록 덮어쓰기 지침을
제거합니다. 마켓플레이스를 갱신한 뒤 플러그인을 업데이트하거나 재설치하세요. 이미 복사한
맞춤 지침에 “먼저 list_workspaces로 선택/질문”이 남아 있으면 앱에서 기본 라우팅 지침을
다시 복사해 교체하세요. 사용자가 의도한 프로젝트별 고정 목적지는 유지할 수 있습니다.
