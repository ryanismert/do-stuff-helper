# Discussion Channel Options — Research Spike

Research into candidate mechanisms for the inbox "Discuss" action, which needs to open a conversational channel between the user and an agent about a specific inbox item.

## Claude Code Remote Control

### How It Works

Remote Control is a feature released on 2026-02-25 that connects claude.ai/code (web) or the Claude iOS/Android app to a Claude Code session running on the user's local machine. The key mechanism:

1. **Start a session**: Run `claude remote-control` in the terminal, or use `/remote-control` (alias `/rc`) within an existing Claude Code session.
2. **Get a URL**: The terminal displays a unique session URL (e.g., a claude.ai/code URL) and optionally a QR code (toggled with spacebar).
3. **Connect from any device**: Open the URL in any browser to reach the session on claude.ai/code, scan the QR code to open it in the Claude mobile app, or find the session by name in the Claude app session list.
4. **Local execution**: Claude keeps running locally the entire time. Files, MCP servers, tools, and project configuration all stay on the host machine. Only chat messages and tool results flow through the encrypted bridge.
5. **Reconnection**: If the laptop sleeps or network drops, the session remains alive in the background and reconnects automatically when the host comes back online.

**Configuration**: Can be set to auto-start for all sessions via `/config` -> "Enable Remote Control for all sessions." By default, it only activates when explicitly invoked.

**Plan requirements**: Available as a research preview on Pro ($20/mo) and Max ($100-$200/mo) plans. Not available on Team or Enterprise plans.

**Session limits**: Each Claude Code instance supports one remote connection at a time. The session ends if you close the terminal or stop the claude process.

### Feasibility for Inbox Discuss

**Strengths:**
- **Zero infrastructure to build** — This is a first-party Anthropic feature. No bot server, no relay daemon, no custom code needed.
- **Full agent capabilities** — The remote session has access to the local filesystem, MCP servers, skills, and all Claude Code tools. The discussion agent would have the same power as any worker.
- **Browser accessible** — The session URL opens in any browser, which means the dashboard could link directly to it.
- **Session persistence** — Sessions survive network drops and laptop sleep, with automatic reconnection.

**Challenges:**
- **Programmatic session creation is not supported** — Remote Control is designed as an interactive feature. There is no API or CLI flag to start a remote-control session headlessly and get back a URL. The `--dangerously-skip-permissions` flag is explicitly not supported in remote-control mode.
- **One session per terminal** — Each `claude remote-control` invocation occupies a terminal. To have multiple inbox discussions simultaneously, we would need multiple terminal sessions (e.g., via tmux).
- **Session lifecycle management** — No built-in mechanism to programmatically create, query, or tear down remote-control sessions. We would need a wrapper (tmux + launcher script) to spawn `claude remote-control` in named sessions and capture the session URL from stdout.
- **Plan dependency** — Requires Pro or Max subscription. Our exoself host already has this, but it ties the feature to a specific subscription tier.
- **New feature (research preview)** — Released literally yesterday (2026-02-25). APIs and behavior may change. Not yet battle-tested.

**Potential integration pattern:**
1. Dashboard "Discuss" button triggers a webhook to the exoself host.
2. Webhook handler spawns `claude remote-control` in a new tmux session, passing context about the inbox item via `--resume` or initial prompt.
3. Handler parses the session URL from stdout.
4. Dashboard redirects user to the session URL (opens in browser on claude.ai/code).
5. When discussion resolves, the agent updates the inbox item and exits, or the user closes the session.

This pattern is technically possible but fragile — it requires stdout parsing, tmux orchestration, and lacks official API support.

### Unknowns

- Can the session URL be reliably parsed from `claude remote-control` stdout? What is the exact output format?
- Is there a way to pass an initial prompt or context to a `claude remote-control` session programmatically?
- What happens when the user disconnects from a remote-control session? Does the agent continue working, pause, or terminate?
- Will Anthropic add a programmatic API for remote-control session management?
- What are the rate limits for remote-control sessions on Pro vs Max plans?
- Can a session started via `claude --continue <session-id>` then be upgraded to remote-control via `/rc`?

## Telegram Bot

### How It Works

A Telegram bot acts as a relay between the user's Telegram client and a Claude Code session running on the exoself host. The setup involves:

1. **Create a bot**: Register with @BotFather on Telegram. Get an API token. Takes about 30 seconds.
2. **Run a relay service**: A daemon on the host receives Telegram messages via the Bot API (polling or webhook), forwards them to a Claude Code session (via the Agent SDK or CLI), and sends responses back to Telegram.
3. **User interacts via Telegram**: Send messages to the bot from any Telegram client (mobile, desktop, web at web.telegram.org).

**Deep linking**: Telegram supports deep links in the format `https://t.me/<bot_username>?start=<parameter>`. This opens the bot conversation and passes the parameter to the bot as `/start <parameter>`. This could be used to deep-link to a specific inbox item discussion. However, all conversations with a bot happen in a single chat — there are no separate "threads" per inbox item in the standard Bot API. You cannot deep-link to a specific message in a private bot chat.

**Existing frameworks and libraries:**

| Project | Language | Approach | Maturity |
|---------|----------|----------|----------|
| [claude-code-telegram](https://github.com/RichardAtCT/claude-code-telegram) | Python | Full agent SDK + CLI fallback, SQLite persistence, multi-layer auth | Most mature; tagged releases |
| [claudegram](https://github.com/NachoSEO/claudegram) | Node.js | Bridges Telegram to local Claude Code, real-time streaming | Active; security-focused |
| [claude-telegram-relay](https://github.com/godagoo/claude-telegram-relay) | TypeScript (Bun) | Minimal relay daemon, Supabase memory, systemd/launchd | Lightweight; daemon-first |
| [ccbot](https://github.com/six-ddc/ccbot) | Go? | tmux bridge: 1 topic = 1 window = 1 session | Interesting session model |

**n8n integration**: n8n has native Telegram Trigger and Telegram nodes. A Telegram Trigger node can start a workflow when the bot receives a message. The workflow could relay the message to Claude (via webhook to claude-webhook or via the Agent SDK) and send the response back via the Telegram Send Message node. This is the lowest-code option since we already run n8n.

### Feasibility for Inbox Discuss

**Strengths:**
- **Deep-link from dashboard** — The dashboard can link to `https://t.me/<bot_username>?start=<inbox-item-id>`, which opens the Telegram app/web client and passes the item ID to the bot. The bot can then load context for that item.
- **Multi-platform** — Telegram has native apps for iOS, Android, macOS, Windows, Linux, and a web client (web.telegram.org). The user can discuss from any device without needing a Claude subscription on that device.
- **No subscription dependency** — Works with the Anthropic API key we already use for Claude Code, or with the Agent SDK. Does not require Pro/Max for the end-user device.
- **Existing ecosystem** — Multiple open-source projects already bridge Telegram to Claude Code. We could use one directly or adapt patterns.
- **n8n native support** — Telegram Trigger + Send Message nodes mean we could prototype the relay in n8n with near-zero custom code.
- **Always available** — The bot runs as a daemon on the host. No need to have a terminal open.

**Challenges:**
- **Infrastructure to build and maintain** — Even with existing libraries, we need to run a relay service (Node.js, Python, or n8n workflow). This is a new service to deploy, monitor, and maintain.
- **Single chat, not per-topic threads** — All bot conversations happen in one chat thread. To discuss multiple inbox items, we would need to manage conversation context within the single chat (e.g., "Currently discussing: item X. Send /switch to change topic."). Telegram does support Topics in groups, but not in private bot chats.
- **Message formatting limitations** — Telegram supports Markdown and HTML in messages, but not the rich rendering that claude.ai/code provides (file diffs, tool output, etc.).
- **No native session resume** — If the user closes Telegram and comes back later, the bot needs to restore context. The existing libraries handle this (SQLite persistence), but it adds complexity.
- **Latency** — Messages go: User -> Telegram API -> Bot server -> Claude Code -> Bot server -> Telegram API -> User. This adds a few hundred ms of overhead per round trip compared to direct remote-control.
- **Security surface** — The Telegram bot token and user whitelist need to be managed. The bot has access to Claude Code on the host, so a compromised token could allow unauthorized access.

**Potential integration pattern:**
1. Dashboard "Discuss" button generates a deep link: `https://t.me/exoself_bot?start=inbox_<item-id>`.
2. User clicks the link, which opens Telegram and sends `/start inbox_<item-id>` to the bot.
3. Bot receives the start command, loads the inbox item context, and begins a discussion.
4. Messages relay back and forth between user and Claude via the bot.
5. User sends `/resolve` or `/done` to close the discussion, which updates the inbox item status.
6. Context is persisted in SQLite for potential resumption.

### Unknowns

- Which relay approach is most reliable? Standalone daemon vs n8n workflow?
- How well do the existing libraries (claude-code-telegram, claudegram) handle long-running conversations with context?
- What is the UX for managing multiple concurrent inbox discussions in a single Telegram chat?
- How do we handle authentication — whitelist by Telegram user ID, or require a shared secret?
- What happens when Claude Code is busy with a worker task and a Telegram message arrives? Queue it, reject it, or spawn a new session?
- How do we handle the bot token securely in our Docker/systemd setup?

## Comparison

| Criterion | Remote Control | Telegram Bot |
|-----------|---------------|--------------|
| **Setup effort** | Near-zero (first-party feature) | Medium (bot registration + relay daemon) |
| **Infrastructure** | None (built into Claude Code) | New service to deploy and maintain |
| **Deep-link from dashboard** | Yes (session URL) | Yes (`t.me/bot?start=param`) |
| **Browser accessible** | Yes (claude.ai/code) | Yes (web.telegram.org) |
| **Mobile accessible** | Yes (Claude app, iOS/Android) | Yes (Telegram app, all platforms) |
| **Programmatic session creation** | Not supported (stdout parsing hack) | Fully supported (Bot API) |
| **Per-topic isolation** | Yes (one session per discussion) | No (single chat, context-managed) |
| **Agent capabilities in session** | Full (filesystem, MCP, tools) | Full (via Agent SDK/CLI relay) |
| **Rich output rendering** | Full (file diffs, tool output) | Limited (Markdown/HTML text) |
| **Session persistence** | Built-in reconnection | Requires SQLite or similar |
| **Plan requirements** | Pro or Max subscription | API key only |
| **Concurrent discussions** | One per terminal/tmux window | One chat, context-switched |
| **Maturity** | Research preview (day 1) | Established pattern, multiple libs |
| **n8n integration** | None (would need custom webhook) | Native nodes (Trigger + Send) |
| **User auth on client device** | Claude account required | Telegram account only |
| **Latency** | Low (direct bridge) | Medium (extra API hop) |
| **Maintenance burden** | None (Anthropic maintains) | Ongoing (daemon, updates, monitoring) |

## Recommendation

**Short-term (next 2-4 weeks): Telegram Bot via n8n**

The Telegram bot is the pragmatic choice for the initial implementation:

1. **Programmatic control** — We can create discussion sessions from the dashboard via deep links, which is the core UX requirement. Remote Control has no programmatic session creation API.
2. **Existing infrastructure** — We already run n8n with webhook integration to Claude. Adding a Telegram Trigger node to relay messages is a natural extension of what we have.
3. **No subscription dependency** — Works for anyone with a Telegram account, regardless of Claude plan.
4. **Proven pattern** — Multiple open-source projects demonstrate this works. The relay architecture is well-understood.

**Recommended implementation approach:**
- Register a Telegram bot via @BotFather.
- Build an n8n workflow: Telegram Trigger -> context loader -> Claude webhook -> Telegram Send Message.
- Deep-link format: `https://t.me/exoself_discuss_bot?start=<inbox-item-id>`.
- Manage conversation context per inbox item using n8n's built-in data persistence or a simple JSON file.
- Add `/resolve` command to close discussions and update inbox status.

**Medium-term: Re-evaluate Remote Control**

Remote Control is compelling but premature for our use case:
- It was released yesterday (2026-02-25) as a research preview.
- The lack of programmatic session creation is a dealbreaker for dashboard integration today.
- If Anthropic adds an API for creating remote-control sessions (which is a natural evolution of the feature), it would become the superior option due to zero infrastructure overhead and full agent capabilities.
- Monitor the [Claude Code docs](https://code.claude.com/docs/en/remote-control) and changelog for API additions.

**Hybrid option (if Remote Control gets an API):**
- Use Remote Control for complex discussions that benefit from full Claude Code UI (file diffs, tool approvals).
- Use Telegram for quick back-and-forth discussions that are primarily text-based.
- Dashboard offers both options based on discussion complexity.
