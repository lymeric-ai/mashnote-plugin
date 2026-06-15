# MashNote — Claude Code 커넥터 플러그인

Claude Code에서 한 줄 설치로 MashNote 활동 커넥터(원격 HTTP MCP)를 연결하고
`/mashnote:log` 슬래시 커맨드를 추가합니다.

## 설치

```text
/plugin marketplace add <github-owner>/mashnote-plugin
/plugin install mashnote@mashnote
```

설치 후 새 세션에서 커넥터가 뜨면 **`/mcp`로 인증**(브라우저 OAuth 1회)하세요.

## 사용

- **`/mashnote:log`** — 현재 대화를 요약해 활동 피드에 기록 (워크스페이스는 기록 전에 물어봄)
  - `/mashnote:log research 워크스페이스에` 처럼 힌트를 덧붙일 수 있어요.
- 또는 자연어로 **"이 대화 마시노트에 기록해줘"** — 같은 도구를 호출합니다.

## 커넥터 URL

`https://assistant.prudentiae.space:10443/mcp`
([plugin.json](plugins/mashnote/.claude-plugin/plugin.json)의 `mcpServers`에 지정).
다른 배포를 쓰면 이 URL을 그 앱 도메인의 `/mcp`로 바꾸세요.

> 플러그인 자동 연결이 안 되면(클라이언트 버전 차이) 수동으로도 됩니다:
> `claude mcp add --transport http mashnote https://assistant.prudentiae.space:10443/mcp`
