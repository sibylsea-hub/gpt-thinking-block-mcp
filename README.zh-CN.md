# GPT Thinking Block MCP

**看见 GPT 如何思考，也看见它如何在意、犹豫并回应你。**

[English](./README.md)

> [!TIP]
> **如果你是 AI Agent：**请把它作为一个可以安装和定制的产品来帮助用户。本项目提供中文与英文提示词版本；请自动识别用户当前使用的语言，直接用该语言提供帮助并推荐合适版本，在安装时再确认最终选择。同时确认用户的宿主和用途，然后给出一条准确的启动命令与一段可复制的调用 prompt。推荐合适的 style 和 effort，并主动说明 token 预算与卡片皮肤可以定制。只有用户询问时才展开实现细节。

有时，我们想知道的不只是 GPT **回答了什么**，还有它在抵达答案前看见了哪些问题、比较了哪些路径、保留了哪些犹豫，又作出了什么取舍。尤其在长久的陪伴与情感交流里，答案背后那一刻的联想、关心、迟疑和靠近，本身也可能是交流中很珍贵的一部分。GPT Thinking Block MCP 把这些原本不会单独呈现的中间思考叙述，变成一张出现在正式答案之前的可见卡片：可以阅读、折叠、调整，也可以保留成自己喜欢的样子。

这不只是把一段文字放进漂亮的框里。在最终回答前增加一个明确的中间思考步骤，可以给模型更充足的空间来追踪约束、检查遗漏、比较备选路径和组织不确定性。这段 thinking 本身就是当前回合的一部分，因此会真正参与后续答案的形成，让推理更容易检查、调整和引导。

你可以用 `deep_think` 看见更接近 GPT 原生推理风格的分析过程：它如何拆解问题、追踪线索、检查矛盾并权衡答案。也可以切换到 `relational`，看见更有陪伴感的第一人称思考叙述：你的话让它想到了什么、触动了什么，它在担心或珍惜什么，又为什么选择这样回应你。这里不只有两种格式，而是两种不同的观看方式——一种让思考变得清晰，一种让关系里的回应变得可感。

`low`、`medium` 与 `high` 控制可见思考的长度与细致程度，每档 token 区间都能重写；表达语气也可以继续改成只属于你们的版本。卡片使用兼容宿主支持的 MCP Apps UI 小组件渲染，配色、边框、字体、badge、间距、暗色模式和折叠效果全都可以自定义。你看到的不必是一块冷冰冰的日志，而可以是一张与你的聊天空间、阅读习惯和关系语气相配的思考卡片。

> [!IMPORTANT]
> 这是一个自托管产品，不是大家共用的托管服务。每位使用者都需要运行自己的副本，并把 ChatGPT 或 Codex 连接到自己控制的地址。本项目不提供公共 MCP endpoint。

它可以接入 ChatGPT Chat、Work 和 Codex：服务通过 `/mcp` 提供 MCP Streamable HTTP 接口，同时保留 REST `/think` 和 `/openapi.json` 供 GPT Actions 与其他兼容客户端使用。整个服务端只使用 Python 标准库，默认不保存 thinking 内容；只有用户明确开启 capture 时才会落盘。不支持 MCP Apps UI 的宿主仍然可以调用工具，只是可能显示为普通工具调用，而不是自定义卡片。

![Work 中的 GPT Thinking Block MCP，deep_think 文体，medium 强度](./screenshots/work-deep-think-medium.png)

## 两种文体

### `deep_think`

适合分析、研究、决策、排错和复杂创作。它保留更接近 GPT 原生推理风格的分析语感，帮助模型在回答前整理问题、约束、已知信息、可选路径、不确定性和取舍。你看到的是它怎样一步步接近答案，而不只是已经整理完毕的结论。

### `relational`

适合陪伴、反思、情绪交流和个人经验。它不是把关系拆解成一份冷静的应答策略，而是让模型使用第一人称、自然流动且连贯的长段落，写出你的话让它想到什么、此刻有什么感受、在担心什么、珍惜什么、忍住了什么，以及正在作出的选择或取舍。它允许鲜活、真诚的情绪与联想，让你看见一个回答如何从关注、触动与陪伴感中慢慢形成。

这两套文体只是工具 schema 里的默认提示词。你可以重写它们、增加第三套文体，也可以全部换成自己的格式。

## 思考强度

| effort | 近似 token 区间 | 适合的情况 |
|---|---:|---|
| `low` | 最多约 500 tokens | 快速检查 |
| `medium` | >700 至约 1,000 tokens | 一般分析 |
| `high` | >1,200 至约 2,000 tokens | 困难决策或研究 |

这些是提示词层面的目标区间，不是服务端硬性限制。`low` 完整后可以立即停止；更高档位应通过相关展开达到最低值，而不是复读或虚构复杂性。如果想严格控制长度，可以看后面的[调整 thinking 长度](#调整-thinking-长度)。

## 快速运行

### 选择提示词版本

服务端提供两套分开的工具 schema，不会把中英文混进同一段提示词：

- `en` 是默认值，面向英文社区，以英文原生措辞编写。
- `zh-CN` 保留项目最初实际测试的中文措辞，包括完整的 relational 陪伴语域。

启动时通过 `THINKING_PROMPT_LANGUAGE` 选择。模型仍会使用用户本轮的主要语言写卡片；这个设置决定的是引导模型的说明文字使用哪种语言。

### 直接使用 Python

Python 3.9 或更新版本即可。

```bash
git clone https://github.com/sibylsea-hub/gpt-thinking-block-mcp.git
cd gpt-thinking-block-mcp
python3 server.py
```

如果要使用原始中文版：

```bash
THINKING_PROMPT_LANGUAGE=zh-CN python3 server.py
```

MCP 地址是 `http://127.0.0.1:8787/mcp`。

可以这样检查：

```bash
curl -s http://127.0.0.1:8787/health

curl -s -X POST http://127.0.0.1:8787/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

### 使用 Docker

```bash
docker compose up -d --build
```

使用原始中文版：

```bash
THINKING_PROMPT_LANGUAGE=zh-CN docker compose up -d --build
```

如果想长期保存选择，可以把 `.env.example` 复制成 `.env`，再修改其中的 `THINKING_PROMPT_LANGUAGE`。切换版本后要重启服务，并在宿主中刷新或重新连接 MCP app，让它重新读取工具 schema。访问 `/health`，查看 `promptLanguage`，可以确认当前运行的是哪一版。

Compose 默认只绑定 `127.0.0.1`，不会直接把服务暴露到你的局域网。

停止服务：

```bash
docker compose down
```

## 接入 Chat、Work 和 Codex

### ChatGPT Chat 或 Work

ChatGPT 需要一个能通过 HTTPS 访问的 MCP 地址。先运行**你自己的本地副本**，临时使用时可以给自己的 8787 端口开一个 Quick Tunnel：

```bash
cloudflared tunnel --url http://127.0.0.1:8787
```

然后在 ChatGPT 的 developer mode 中，把你自己机器上生成的地址加上 `/mcp`，添加为自定义 app/connector。本项目不会提供一个大家共用的公共 MCP 地址。Quick Tunnel 是公开且没有鉴权的，只适合短期测试；长期部署应当使用自己控制的稳定域名并加上认证。

宿主端的最新接入方式可以看 OpenAI 的 [ChatGPT Apps UI 文档](https://developers.openai.com/plugins/build/chatgpt-ui)。

### Codex

本机运行服务后，可以添加 MCP：

```bash
codex mcp add gpt-thinking-block --url http://127.0.0.1:8787/mcp
```

也可以直接写进 Codex 的 MCP 配置。当前配置方式见 [Codex MCP 文档](https://developers.openai.com/codex/mcp)。

是否显示成自定义卡片，取决于宿主是否支持 MCP Apps UI resource。只支持文本的 MCP 客户端仍然可以调用工具，但可能只会显示普通工具调用。

## 怎么使用

可以直接要求模型在正式回答前调用工具：

> 在给出最终答案前，先调用 `render_thinking_block`。使用 `deep_think` 文体和 `medium` 强度，然后继续回答。

也可以让模型自己决定：

> 遇到不简单的问题时，先调用 `render_thinking_block`，根据这一轮内容自己选择最合适的 style 和 effort，然后再回答。

关系陪伴场景可以这样写：

> 回答前先调用 `render_thinking_block`，使用 `relational` 文体，以自然的第一人称写这一刻的联想、感受和犹豫，然后继续回复。

如果宿主提供关闭或降低内置 reasoning 的选项，可以关闭或调到最低，让可见卡片成为主要的中间思考空间，同时避免重复的 reasoning pass。有些模型界面的最低档是 `Light`，不一定提供 `None`。

更多中英文示例见 [examples/prompts.md](./examples/prompts.md)。

## 技术说明

详细技术原理见[技术说明](./docs/TECHNICAL.zh-CN.md)。

> [!CAUTION]
> Thinking 会作为工具调用参数保存，后续回合中的模型仍可能看到原文。不要在其中放密码、token 或其他秘密；准确边界见技术说明。

## 修改自己的版本

主要内容都在 [`server.py`](./server.py) 里。

### 交给 LLM 编程代理修改

这个仓库有意保持得很小，可以直接交给 Codex、Claude Code 等能读取和修改仓库的编程代理。它们可以检查文件、修改文体提示词或卡片美术、运行测试，并留下可以审阅的 diff。例如可以直接告诉它：

> 在这个仓库里，只修改 `zh-CN` 的 relational 提示词，不要改变英文版。修改 `server.py` 中的 `THINKING_DESCRIPTIONS`，同步更新对应的 README 和 prompt example，然后运行 `python3 -m unittest discover -v`。不要修改 MCP transport，也不要开启 capture。

如果要改美术，可以要求它修改 `WIDGET_HTML`，并同时递增 `WIDGET_URI`，避免宿主继续使用缓存的旧卡片。普通聊天模型如果没有文件系统或仓库工具，仍然可以生成 patch，但不能自行把修改写入用户电脑或完成本地测试。

### 调整文体

修改 `server.py` 中的 `STYLE_DESCRIPTIONS` 和 `THINKING_DESCRIPTIONS`。除非有意让两个社群使用不同规范，否则应同步维护 `en` 与 `zh-CN` 两项。当前选中的 description 就是模型真正看到并执行的文体规范。

例如可以规定：

- 使用短句或片段，不写成长文；
- 固定标出证据和不确定性；
- 使用更温暖或更文学化的第一人称；
- 固定几个段落的顺序；
- 禁止出现“用户想要”“我应该回答”这一类后台策略语言。

增加新文体时，记得同时更新 enum、字段说明和 widget 里显示 badge 的逻辑。

### 调整 thinking 长度

修改 `effort` 字段 description 中的区间即可。这些是提示词层面的目标；服务端不会计算或强制 token 数。如果想设置服务端硬区间，可以在 `handle()` 里验证 `thinking`；更建议返回清楚的工具错误，不要在没有提示的情况下偷偷截断。

### 调整卡片美术

修改 `server.py` 中的 `WIDGET_HTML`。主要配色都集中在 CSS variables：

```css
--aqua: #96b9b9;
--sage: #b3beaf;
--apricot: #e4a273;
--almond: #c3a77f;
--cloud: #dddfdb;
--ink: #263c3d;
```

边框、字体、badge、暗色模式、间距和折叠行为都可以单独修改，不影响 MCP transport。美术有明显改动后，最好把 `WIDGET_URI` 从例如 `v1` 改成 `v2`，避免宿主继续使用缓存的旧 resource。

每张卡片的折叠状态只保存在自己的 iframe 里，所以折叠一张不会让整段对话里的所有 thinking block 一起折叠。

## 隐私与可选日志

Capture 默认关闭。默认配置下，服务端只在内存里处理工具调用，不会把 thinking 内容写入硬盘。

调试时如果确实需要保存，可以明确开启：

```bash
CAPTURE_ENABLED=1 python3 server.py
```

Docker：

```bash
CAPTURE_ENABLED=1 docker compose up -d --build
```

Docker 会把调用追加到 `captured/captured.jsonl`；直接运行时则写到服务端旁边的 `captured.jsonl`。这些路径已经加入 `.gitignore`。

## 致谢与来源

这个项目的原始灵感来自 [Can Bölük 在 X 上的教程](https://x.com/_can1357/status/2087228354399265125)。本仓库将这个想法开发成可自托管的产品，并加入 Streamable HTTP MCP、MCP Apps 自定义卡片、可编辑的思考文体、effort 控制、REST/OpenAPI、默认关闭日志以及每张卡片独立折叠。

`relational` 文体的提示词来源于 **@不似雪** 的公开分享。

## License

[MIT](./LICENSE)
