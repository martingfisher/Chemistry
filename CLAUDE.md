# Chemistry

## Cloudflare agent setup

This repository is set up for building on the Cloudflare developer platform, following
[developers.cloudflare.com/agent-setup](https://developers.cloudflare.com/agent-setup/).

`.claude/settings.json` registers the [cloudflare/skills](https://github.com/cloudflare/skills)
plugin marketplace and enables the `cloudflare@cloudflare` plugin. On first launch in this
repository, Claude Code will ask to trust the marketplace and install the plugin, which provides:

- **Skills** (auto-loaded by topic): cloudflare platform, wrangler, durable-objects, agents-sdk,
  workers-best-practices, sandbox, web-perf, building MCP servers and AI agents, Cloudflare One
- **Slash commands**: `/cloudflare:build-agent`, `/cloudflare:build-mcp`
- **Remote MCP servers**: cloudflare-api (Code Mode, full API), cloudflare-docs,
  cloudflare-bindings, cloudflare-builds, cloudflare-observability

The first Cloudflare tool call triggers an OAuth flow to authorize access to the Cloudflare
account. For CI/CD, pass a Cloudflare API token as a bearer token instead.

## Cloudflare docs for agents

Prefer the cloudflare-docs MCP server for current documentation. Without it, fetch Markdown
directly: [developers.cloudflare.com/llms.txt](https://developers.cloudflare.com/llms.txt) indexes
every product (`/<product>/llms.txt` for one product), and any docs page is available as Markdown
by appending `index.md` to its URL.
