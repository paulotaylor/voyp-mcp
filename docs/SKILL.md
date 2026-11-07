# Voyp skill for AI agents

The **Voyp skill** teaches an AI agent how to use the Voyp MCP server to make real phone
calls well and safely: find the right number, brief the voice agent, get the user's
approval, follow the call, and report the result.

The MCP server gives the agent the *tools* (`start_call`, `get_call`, …). The skill gives it
the *know-how*: when to use them, in what order, and with what care.

- Skill files: [`skills/voyp/`](../skills/voyp)
  - [`SKILL.md`](../skills/voyp/SKILL.md): the main instructions the agent loads
  - [`references/tools.md`](../skills/voyp/references/tools.md): arguments and responses of each tool
  - [`references/call-context.md`](../skills/voyp/references/call-context.md): how to write the call briefing, with a template
  - [`references/examples.md`](../skills/voyp/references/examples.md): example sessions
- Format: [Agent Skills](https://agentskills.io): a folder with a `SKILL.md` file
  (YAML front matter plus Markdown). It works in Claude Code, the Claude apps, the Claude API,
  and other agents that support Agent Skills.

## What the skill makes the agent do

1. **Get the facts.** Who to call, the goal, the details the callee will ask for, and acceptable alternatives.
2. **Find and check the number.** Look it up with `search_place` / `search_places` (local
   server) or ask the user, then convert it to E.164 format (`+14155550123`).
3. **Write a full call context**, because the voice agent knows only what is in it:
   goal, details, alternatives, questions, limits, and how to end the call.
4. **Ask the user before every call**: number, language, and what the agent may agree to.
5. **Start the call** and share the live URL.
6. **Follow the call** with `get_live_call` / `get_call`, and end it with `hangup_call` only when needed.
7. **Report the result**: what was agreed, what is still open, and the next steps.

The skill also has safety rules: no emergency numbers, no spam, harassment or deception,
no sensitive data such as card numbers or passwords, and no redialling without approval.

## Requirements

1. A **Voyp account** with call credits: https://voyp.app/app.html
2. The **Voyp MCP server** connected to your agent, in one of two ways:
   - **Hosted** (recommended): `https://api.voyp.app/mcp/stream`, Streamable HTTP. Sign in
     with your Voyp account through OAuth the first time you connect, or send your Voyp API key
     as `Authorization: Bearer sk_...`.
   - **Local (stdio)**: `npx -y voyp-mcp` with `VOYP_API_KEY` set. This one adds the
     place search tools (`search_places`, `search_place`, `search_place_by_number`).
3. An agent that supports skills. For agents that do not, see
   [Agents without skill support](#agents-without-skill-support).

---

## Installation

### Claude Code: plugin (skill and MCP server in one step)

This repository is also a Claude Code plugin marketplace. The `voyp` plugin installs the
skill **and** sets up the hosted Voyp MCP server.

```text
/plugin marketplace add paulotaylor/voyp-mcp
/plugin install voyp@voyp
```

On Claude Code 2.1.275 or later, one line does both:

```text
/plugin install voyp --marketplace paulotaylor/voyp-mcp
```

Then run `/mcp`, choose **voyp**, and sign in to Voyp in the browser window that opens.
To update later: `/plugin marketplace update voyp`.

### Claude Code: skill only

Use this if you already set up the MCP server yourself, or you want the local server.

```bash
# for you, in all projects
git clone https://github.com/paulotaylor/voyp-mcp.git /tmp/voyp-mcp
mkdir -p ~/.claude/skills
cp -r /tmp/voyp-mcp/skills/voyp ~/.claude/skills/voyp

# or only for one project (commit it to share with your team)
mkdir -p .claude/skills
cp -r /tmp/voyp-mcp/skills/voyp .claude/skills/voyp
```

Add the MCP server if you have not yet:

```bash
# hosted (OAuth), then run /mcp in Claude Code to sign in
claude mcp add --transport http voyp https://api.voyp.app/mcp/stream

# or hosted with an API key (no browser sign-in)
claude mcp add --transport http voyp https://api.voyp.app/mcp/stream \
  --header "Authorization: Bearer your-voyp-api-key"

# or local (API key)
claude mcp add voyp --env VOYP_API_KEY=your-voyp-api-key -- npx -y voyp-mcp
```

Check it: run `/skills` and look for `voyp`, or just ask *"call the pizzeria on Main Street
and order a large margherita for pickup"*.

### Claude apps (claude.ai, Claude Desktop)

1. **Connect the MCP server:** go to **Settings → Connectors → Add custom connector**,
   enter `https://api.voyp.app/mcp/stream`, and sign in to Voyp.
2. **Upload the skill:** make a zip of the skill folder:
   ```bash
   git clone https://github.com/paulotaylor/voyp-mcp.git
   cd voyp-mcp/skills && zip -r voyp-skill.zip voyp
   ```
   Then go to **Settings → Capabilities → Skills**, choose **Upload skill**, pick
   `voyp-skill.zip`, and turn it on. (Skills must be enabled for your plan or organization.)

### Claude API (Messages API)

Upload the folder as a custom skill with the Skills API, and give the model the Voyp server
through the MCP connector (`mcp_servers` with `url: https://api.voyp.app/mcp/stream` and your
Voyp API key, or a Voyp OAuth access token, as `authorization_token`). See Anthropic's documentation on *Agent Skills* and the
*MCP connector* for the request format.

### Other agents with Agent Skills support

Many agents (for example OpenAI Codex, Cursor, GitHub Copilot, Gemini CLI, Goose, OpenCode)
read skills in the same `SKILL.md` format. The easiest way is the open
[`skills`](https://github.com/vercel-labs/skills) CLI, which asks which agents to install for:

```bash
npx skills add paulotaylor/voyp-mcp
```

Or copy `skills/voyp` into your agent's skills folder by hand. Check your agent's
documentation for the path; common ones are `~/.agents/skills/`, `~/.codex/skills/`, and
`.github/skills/`.

Then add the Voyp MCP server in that agent's MCP settings: the hosted URL (with OAuth, or
an `Authorization: Bearer sk_...` header), or the local `npx -y voyp-mcp` command. See the
[main README](../README.md).

### Agents without skill support

Copy the body of [`skills/voyp/SKILL.md`](../skills/voyp/SKILL.md) (everything after the
front matter) into the agent's system prompt or project instructions file (`AGENTS.md`, a
Goose `.goosehints` file, Cursor rules, …). Add
[`references/call-context.md`](../skills/voyp/references/call-context.md) too if you have room.
It is the part that helps call quality the most.

---

## Usage

You do not need to name the skill. The agent loads it when you ask for something that needs
a phone call:

- "Book a table for 4 at Bella Notte on Saturday at 8, under Maria."
- "Call my dentist and move Thursday's appointment to any morning next week."
- "Ask the bike shop on Main Street if they have a 54cm road bike in stock."
- "Call +351 21 000 0000 in Portuguese and ask what time they close today."

What happens next:

1. The agent asks for any missing details.
2. It shows you the number, the language, and a summary of the call, and **waits for your yes**.
3. It starts the call and gives you a live link (`https://www.voyp.app/live/<id>`) where you
   can follow the call.
4. It checks the call while it runs and tells you the result when it ends.
5. Say "hang up" at any time to end the call.

In Claude Code you can also run it directly: `/voyp book a table for 2 at …`.

## Troubleshooting

| Problem | Fix |
|---|---|
| The skill is not used | Check that it is installed (`/skills` in Claude Code, or the Skills settings page) and that the Voyp tools are connected (`/mcp`). Say "phone call" or "call" in your request |
| `401` / invalid or expired token (hosted, OAuth) | Sign in again: in Claude Code run `/mcp` → voyp → re-authenticate; in the Claude apps reconnect the connector |
| `401` "Invalid API key" (hosted with a key, or local) | Check the key: it starts with `sk_`, has no quotes or extra spaces, and is the current key at https://voyp.app/app.html. The header must be `Authorization: Bearer sk_...` |
| "Missing required OAuth scope" | Reconnect and approve all the Voyp permissions (calls and profile) |
| "Unable to start call" | Make sure the number is in E.164 format and that you have credits |
| No place search tools | They are only on the local stdio server. Use it, or give the number yourself |

## Customizing

The skill is plain Markdown, so you can fork it and change it. For example, you can add
your own default callback number or signature, or change how often it checks a call.
Keep the confirmation step and the safety rules.

## License

MIT, the same as this repository.
