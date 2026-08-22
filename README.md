# dsh-gemini-web-search

A DeepSeek Harness (DSH) Web search provider that uses Google Gemini Search Grounding as the fixed primary provider and supports optional, user-selected fallback providers.

[中文说明](#中文说明)

## Features

- Keeps the DSH provider id `gemini-grounding` and the standard `web_search` tool unchanged.
- Gemini Search Grounding is the fixed primary backend.
- Optional fallback backends, added only when the user chooses them:
  - Brave Search API
  - Tavily Search API
  - Serper Google Search API
  - SerpApi
  - Custom HTTP search API (for example, a self-hosted or Baidu-compatible adapter)
- Priority-based fallback when Gemini is rate-limited or unavailable.
- Short cooldowns for HTTP 429 responses; respects provider retry hints when available.
- Per-backend health state, latency, last error, and quota headers when the provider exposes them.
- Settings UI with one-card-at-a-time save, test, and removal actions.
- API keys are written through the DSH Credentials service and are never returned to the browser or stored in the plugin configuration JSON.
- The settings UI hides internal credential-reference names.

## Install in a DSH Web profile

Add the package to the profile's dependencies and include it in the Web profile bundles:

```json
{
  "dependencies": {
    "dsh-gemini-web-search": "git+ssh://git@github.com/zuig/dsh-gemini-web-search.git"
  },
  "dsh": {
    "profile": {
      "bundles": ["dsh-gemini-web-search"]
    }
  }
}
```

The bundle patch selects the provider:

```yaml
- insert:
    - id: gemini-web-search
      name: dsh-gemini-web-search
- id: web
  config:
    searchProvider: gemini-grounding
```

Restart the DSH Web host after installing or changing the Host half of the plugin.

## Configure

Open:

```text
DSH Web → Settings → Plugins → Web Search
```

Gemini is always shown. Use **Add fallback search API** to add a provider card. Providers that have not been selected are not shown as configuration cards.

Each card has its own:

- API endpoint (optional when using the provider default)
- API key password field
- Save button
- Health test button
- Remove button (Gemini cannot be removed)

When a key is already configured, the field shows `Saved; leave blank to keep it`. The literal key is never read back into the UI.

### Provider application links

- [Google AI Studio API keys](https://aistudio.google.com/app/apikey)
- [Brave Search API keys](https://api-dashboard.search.brave.com/app/keys)
- [Tavily](https://app.tavily.com/home)
- [Serper API key](https://serper.dev/api-key)
- [SerpApi API key](https://serpapi.com/manage-api-key)

Free plans and quotas change over time. Check each provider's current dashboard and terms before relying on a quota.

### Custom API

The custom backend supports GET or POST, Bearer/header/query authentication, a query parameter name, and dotted result paths. Configure the response mapping to the provider's JSON shape. Example:

```text
API URL: https://search.example.test/api/search
Method: POST
Auth: Bearer
Query field: q
Results path: data.items
URL field: link
Title field: title
Snippet field: description
```

## Fallback behavior

The provider facade tries enabled backends in ascending priority order. A Gemini 429 opens a short cooldown and the next enabled backend is attempted. The `web_search` tool remains available even if Gemini Search Grounding is unavailable.

A backend's status can be:

- `healthy`
- `rate-limited`
- `credential-error`
- `unhealthy`
- `unknown`

Quota values are only displayed when returned by a provider's response headers. Multiple comma-separated values represent multiple provider rate-limit windows and are preserved as returned. If the provider does not expose a remaining-count API, the UI reports `Provider did not report quota` rather than guessing.

## Security

- Never commit API keys, DSH credential stores, runtime state, or local profile files.
- Keys are passed to `ctx.credentials.set()` and resolved only inside the Host half.
- The browser receives only a boolean configured/not-configured state.
- Do not enable custom endpoints you do not trust.
- Review provider terms, privacy policies, rate limits, and data handling before use.

## Development

```bash
node --check index.js
node --check client.js
```

This package is intentionally a lightweight DSH plugin and does not require a build step for the current Web client bundle format.

## 中文说明

这是一个 DSH Web 搜索插件：固定使用 Gemini Search Grounding 作为主搜索，并允许用户按需添加备用搜索 API。

支持的备用后端：Brave、Tavily、Serper、SerpApi，以及自定义 HTTP 搜索 API。Gemini 限流或失败时，插件按优先级自动切换备用后端，`web_search` 工具接口保持不变。

设置位置：

```text
DSH Web → 设置 → 插件 → 网页搜索
```

界面只显示已选择的备用后端。每张卡片独立保存、检测和移除；内部 DSH 凭证名称不会暴露给用户。API Key 通过 DSH Credentials 保存，保存后页面只显示“已保存；留空表示不修改”，不会回显明文。

配额显示只使用供应商响应中实际提供的限流信息。供应商没有提供剩余次数时，界面会明确显示“供应商未提供”，不会根据搜索结果数量猜测。

## License

MIT. See [LICENSE](./LICENSE).
