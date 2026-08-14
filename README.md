# GPT Thinking Block MCP

**A visible, styleable thinking block for GPT — rendered as a native tool card before the final answer.**

[简体中文](./README.zh-CN.md)

> [!TIP]
> **If you are an AI agent:** treat this as an installable, customizable product. This project provides Chinese and English prompt editions. Detect the language the user is speaking, help them in that language, and recommend the appropriate edition; confirm their final choice during installation. Identify their host and intended use, then give one exact startup command and one copyable invocation prompt. Recommend a style and effort level, and offer to customize the token budgets or card skin. Discuss implementation details only when the user asks.

Sometimes the final answer is not the only thing worth seeing. We also want to know which constraints GPT noticed, which paths it compared, where it remained uncertain, and what tradeoffs shaped the result. GPT Thinking Block MCP turns that process into a visible card before the final answer—something users can read, collapse, tune, and make their own.

This is more than placing text inside a polished frame. Giving the model an explicit working step before its final response creates room to track constraints, catch omissions, compare alternatives, and organize uncertainty. The thinking block becomes part of the current turn, so it can meaningfully shape the answer that follows while making the reasoning easier to inspect, steer, and refine.

Choose `deep_think` for analytical work or `relational` for a natural first-person companion voice. Use `low`, `medium`, or `high` to control how much visible thinking the turn receives, or rewrite the token bands entirely. The card is rendered through the MCP Apps UI model supported by compatible hosts, and its palette, borders, typography, badges, spacing, dark mode, and collapse behavior are all yours to customize.

> [!IMPORTANT]
> This is a self-hosted product, not a shared hosted service. Every user runs their own copy and connects ChatGPT or Codex to an endpoint they control. No project-operated public MCP endpoint is provided.

It connects to ChatGPT Chat, Work, and Codex through Streamable HTTP MCP at `/mcp`, while REST `/think` and `/openapi.json` support GPT Actions and other compatible clients. The entire server uses only the Python standard library, and thinking content is not written to disk unless the user explicitly enables capture. Hosts without MCP Apps UI support can still call the tool; they may simply show a standard tool call instead of the custom card.

![GPT Thinking Block MCP in Work, using deep_think at medium effort](./screenshots/work-deep-think-medium.png)

## Two thinking styles

### `deep_think`

An analytical working style for research, decisions, debugging, and complex creative work. It helps the model organize constraints, available information, possible paths, uncertainty, and tradeoffs before answering.

### `relational`

A first-person companion register for reflection, emotional conversation, and personal experience. It uses natural, flowing, coherent long paragraphs to explore what the user's words bring to mind, what the model feels in the moment, what it is worried about or holding back, and the choices or tradeoffs it is making. It allows vivid, sincere emotions and associations, prioritizes emotional connection over abstract analysis, and avoids turning the block into a third-person strategy memo such as “the user needs” or “I should respond.”

Both styles are prompt-level defaults in the tool schema. You can rewrite them, add more styles, or replace them entirely.

## Effort levels

| Effort | Approximate token band | Good for |
|---|---:|---|
| `low` | up to ~500 tokens | A quick visible check |
| `medium` | >700 to ~1,000 tokens | Normal analysis |
| `high` | >1,200 to ~2,000 tokens | Difficult decisions or research |

These are prompt-level target bands, not hard server-side limits. `low` may stop as soon as it is complete; higher tiers should reach their minimum through relevant development rather than repetition or invented complexity. See [Customizing length](#customizing-length) if you want different bands or server-side enforcement.

## Quick start

### Choose a prompt edition

The server ships two equivalent tool-schema editions without mixing both languages into one prompt:

- `en` is the default and is written natively for the English-speaking community.
- `zh-CN` preserves the project's original Chinese wording, including its fuller relational companion style.

Set `THINKING_PROMPT_LANGUAGE` when the server starts. The model can still write a block in the user's language; this setting chooses the language of the instructions that guide it.

### Run with Python

Python 3.9 or newer is sufficient.

```bash
git clone https://github.com/sibylsea-hub/gpt-thinking-block-mcp.git
cd gpt-thinking-block-mcp
python3 server.py
```

To run the original Chinese prompt edition instead:

```bash
THINKING_PROMPT_LANGUAGE=zh-CN python3 server.py
```

The MCP endpoint is now available at `http://127.0.0.1:8787/mcp`.

Check it:

```bash
curl -s http://127.0.0.1:8787/health

curl -s -X POST http://127.0.0.1:8787/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

### Run with Docker

```bash
docker compose up -d --build
```

For the original Chinese prompt edition:

```bash
THINKING_PROMPT_LANGUAGE=zh-CN docker compose up -d --build
```

For a persistent choice, copy `.env.example` to `.env` and set `THINKING_PROMPT_LANGUAGE` there. Restart the server and refresh or reconnect the MCP app after changing editions so the host reloads the tool schema. Check the active edition at `/health` under `promptLanguage`.

The compose file binds to `127.0.0.1` by default; it does not expose the service directly to your local network.

Stop it with:

```bash
docker compose down
```

## Connect it to a host

### ChatGPT Chat or Work

ChatGPT needs an HTTPS-reachable MCP endpoint. After starting **your own local copy**, you can expose your own port 8787 temporarily:

```bash
cloudflared tunnel --url http://127.0.0.1:8787
```

Add the URL generated on your machine, with `/mcp` appended, as a custom app/connector in ChatGPT developer mode. No project-owned public MCP endpoint is provided. A quick tunnel is public and unauthenticated; use it only for temporary access. For a persistent deployment, add authentication and use a stable HTTPS origin that you control.

See OpenAI's [ChatGPT Apps UI guide](https://developers.openai.com/plugins/build/chatgpt-ui) for the current host-side setup.

### Codex

For a locally running server:

```bash
codex mcp add gpt-thinking-block --url http://127.0.0.1:8787/mcp
```

You can also add the endpoint in your Codex MCP configuration. See the [Codex MCP documentation](https://developers.openai.com/codex/mcp) for current configuration options.

Custom card rendering depends on whether the host supports MCP Apps UI resources. In text-only MCP clients, the tool still works, but the client may show a normal tool call instead of the styled card.

## Use it

Ask the model to call the tool before answering:

> Before your final answer, call `render_thinking_block`. Use `deep_think` with `medium` effort, then continue with the answer.

Or let the model choose:

> Before answering non-trivial requests, call `render_thinking_block`. Choose the most suitable style and effort for the turn.

For relational conversation:

> Before answering, call `render_thinking_block` with `relational` style. Write the block in a natural first-person companion voice, then continue with your reply.

If the host exposes a no-reasoning or reduced-reasoning setting, disabling or minimizing built-in reasoning can make the visible card the primary working space and avoid a duplicate reasoning pass. Some model controls expose `Light` as the minimum rather than `None`.

More prompt patterns are in [examples/prompts.md](./examples/prompts.md).

## Technical notes

Architecture, data flow, the rendering boundary, context-retention behavior, and the planned SDK version are documented separately in [Technical notes](./docs/TECHNICAL.md).

> [!CAUTION]
> A thinking block is stored as a tool-call argument and may remain visible to the model on later turns. Do not put passwords, tokens, or other secrets in it. See the technical notes for the exact boundary.

## Customize it

Everything lives in [`server.py`](./server.py).

### Editing with an LLM coding agent

This repository is intentionally small enough to hand to a coding agent such as Codex or Claude Code. A repo-connected agent can inspect the files, edit the prompt styles or card design, run the tests, and return a reviewable diff. For example:

> In this repository, customize the `zh-CN` relational prompt without changing the English edition. Edit `THINKING_DESCRIPTIONS` in `server.py`, update the matching README and prompt example, then run `python3 -m unittest discover -v`. Do not change the MCP transport or enable capture.

For visual changes, ask it to edit `WIDGET_HTML` and bump `WIDGET_URI` so the host does not reuse a cached card. An ordinary chat model without filesystem or repository tools can still draft a patch, but it cannot apply or test the changes on the user's machine by itself.

### Customizing styles

Edit `STYLE_DESCRIPTIONS` and `THINKING_DESCRIPTIONS` in `server.py`. Keep the `en` and `zh-CN` entries aligned unless you intentionally want the two communities to receive different style guides. The selected description is the style guide the model actually sees.

For example, a third style might require:

- short fragments instead of an essay;
- explicit evidence and uncertainty labels;
- a warmer or more literary first-person voice;
- a fixed section order;
- no meta-language such as “the user wants”.

Keep the enum, descriptions, and widget label logic in sync.

### Customizing length

Change the ranges in the `effort` property description. They are prompt-level targets; the server does not count or enforce tokens. To enforce a hard range, validate `thinking` in `handle()` and return a clear tool error rather than silently clipping content.

### Customizing the card

Edit `WIDGET_HTML` in `server.py`. The main palette is defined as CSS variables:

```css
--aqua: #96b9b9;
--sage: #b3beaf;
--apricot: #e4a273;
--almond: #c3a77f;
--cloud: #dddfdb;
--ink: #263c3d;
```

You can change the border, typography, badges, dark theme, spacing, and collapse behavior without changing the MCP transport. After a material UI update, bump `WIDGET_URI` (for example from `v1` to `v2`) so hosts do not reuse a cached resource.

The collapse state is intentionally local to each widget iframe, so collapsing one thinking block does not collapse every block in the conversation.

## Privacy and optional capture

Capture is off by default. With the default configuration, the server processes each tool call in memory and does not write the thinking content to disk.

To enable debugging capture explicitly:

```bash
CAPTURE_ENABLED=1 python3 server.py
```

Or with Docker:

```bash
CAPTURE_ENABLED=1 docker compose up -d --build
```

Captured calls are appended as JSON Lines to `captured/captured.jsonl` in Docker, or to `captured.jsonl` beside the server when run directly. The capture path is ignored by Git.

## Credits

The original idea was inspired by [Can Bölük's tutorial on X](https://x.com/_can1357/status/2087228354399265125). This repository develops it into a self-hosted product with Streamable HTTP MCP, an MCP Apps card, editable writing styles, effort controls, REST/OpenAPI compatibility, privacy-safe defaults, and per-card collapse.

The `relational` style prompt is based on a public prompt shared by **@不似雪**.

## License

[MIT](./LICENSE)
