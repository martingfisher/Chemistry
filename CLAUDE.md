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

## Multi-account Cloudflare / Wordify access

This account manages sites across multiple client Cloudflare and Wordify accounts, not just one.

- **Cloudflare**: the `cloudflare-api` MCP tool's account list spans every client account (a
  reseller-style view), including `Tech@thisischemistry.co.uk Account`
  (id `bf8565814ade06e69f0f9741fe538e2b`, the "Chemistry" client). To find a zone without
  knowing which account owns it, `GET /zones?name=<domain>` is account-independent and returns
  the owning `account.id`/`account.name` directly — don't enumerate accounts by hand.
- **Wordify**: separate system from Cloudflare — the connected Wordify identity
  (martin@resultsyoucanmeasure.co.uk) only sees teams it's a *member* of, and `select_team` only
  switches between those; it can't reach a team the account hasn't been invited to. The
  Chemistry Wordify account is a separate team from "RESULTS YOU CAN MEASURE". An invite for
  this identity to join the Chemistry team has been requested — check `get_me`'s `teams[]` for a
  second entry before assuming `select_team` can reach Chemistry sites; if it's still only one
  team, the invite hasn't been accepted yet.
- Direct `curl`/`WebFetch` to arbitrary external domains (client sites, `console.wordify.com`,
  etc.) is blocked by this session's network egress policy — don't retry those, they're policy
  denials, not flakes. For a live look at a page anyway (real response headers, detected tech
  stack, cookies) on a domain that's a Cloudflare zone the connected account can see, use the
  Cloudflare `urlscanner` endpoints: `POST /accounts/{account_id}/urlscanner/scan` (**not**
  `v2/scan` — that variant's response isn't `{success, result}` shaped and trips this tool's
  generic error wrapper) then poll `GET /accounts/{account_id}/urlscanner/scan/{scan_id}` (add
  `/har` for network requests). Always pass `visibility: "Unlisted"` for a client's live site.

## Site incident log

### eizovisualsolutions.com (Chemistry client) — image upload failure, Aug 2026

- Cloudflare zone `7c2a52bc17169421f9dce9ec0d36b162`, account `bf8565814ade06e69f0f9741fe538e2b`
  ("Tech@thisischemistry.co.uk Account"). Hosted on Wordify (`x-server-powered-by: WDFY`),
  assets served via BunnyCDN (`cdn-eizovisualsolutions.b-cdn.net`).
- Stack: WordPress + Uncode theme, Gravity Forms (the support form), CleanTalk anti-spam,
  **Gravity Forms Zero Spam** add-on, WooCommerce, The Events Calendar, Groovy Menu.
- Symptom: image/file uploads on `/customer-support-form-anz/` silently stopped working;
  browser console showed `[Violation] Permissions policy violation: unload is not allowed in
  this document`.
- Ruled out: Cloudflare edge entirely — Rocket Loader off zone-wide, no custom response-header
  Transform Rules, WAF off (its one custom rule correctly exempts `admin-ajax.php`/
  `admin-post.php`), 100MB upload limit, and the live `Permissions-Policy` response header
  doesn't mention `unload` at all (only geolocation/camera/mic/etc.) — so the violation is
  Chrome's own default policy for the deprecated `unload` event, not anything this site or
  Cloudflare configures.
- **Root cause, confirmed by the client**: a then-recent update to the **Gravity Forms Zero
  Spam** plugin broke under Chrome's `unload` deprecation. Disabling that plugin fixed uploads
  immediately (test submission processed correctly); CleanTalk anti-spam remained active as the
  other spam-defense layer. If Zero Spam ships another update, re-test before re-enabling it.
