# Claude Code HUD

A custom statusline for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that shows your rate limits, context usage, running agents, and more — right in your terminal.

No dependencies to install. Just one file.

## Screenshots

**Fresh session** — all green, low usage, latest version:

![Fresh Start](docs/images/hud-fresh-start.png)

**Active development** — agents running, code changes tracked, update available:

![Active Development](docs/images/hud-active-dev.png)

**Heavy session** — yellow warnings as usage climbs, todo progress tracking:

![Heavy Session](docs/images/hud-heavy-session.png)

**Near the limit** — red critical indicators, agent details inline:

![Near Limit](docs/images/hud-near-limit.png)

## What It Shows

| Segment | What it means |
|---------|---------------|
| **5h / 7d** | How much of your rate limit you've used, with a countdown until it resets |
| **Context** | How full the current conversation's context window is |
| **Changes** | Lines of code added and removed this session |
| **Agents** | Number of background agents running (with a detailed list below the bar) |
| **Todos** | Task progress — how many done out of total |
| **Model** | Which Claude model you're using (Opus 4.6, Sonnet 4.5, etc.) |
| **Version** | Your Claude Code version, and whether an update is available |

### Color Coding

Colors shift automatically as values get higher:

| Color | Rate Limits | Context Window |
|-------|-------------|----------------|
| Green | Under 60% | Under 70% |
| Yellow | 60–79% | 70–84% |
| Red | 80%+ | 85%+ |

## Setup

There are two steps: **save the file**, then **point Claude Code to it**.

### Step 1 — Save the script

You need to put `custom-hud.mjs` in a folder where it will stay. The recommended location is inside your Claude Code config folder.

<details>
<summary><strong>macOS / Linux</strong></summary>

Open a terminal and run:

```bash
mkdir -p ~/.claude/hud
curl -o ~/.claude/hud/custom-hud.mjs https://raw.githubusercontent.com/professionalcrastinationco/METRICC/main/custom-hud.mjs
```

This creates the folder and downloads the file in one go.

Your file is now at: `~/.claude/hud/custom-hud.mjs`

</details>

<details>
<summary><strong>Windows (PowerShell)</strong></summary>

Open PowerShell and run:

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\hud"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/professionalcrastinationco/METRICC/main/custom-hud.mjs" -OutFile "$env:USERPROFILE\.claude\hud\custom-hud.mjs"
```

Your file is now at: `C:\Users\<your-username>\.claude\hud\custom-hud.mjs`

</details>

<details>
<summary><strong>Manual download (any OS)</strong></summary>

1. Click on `custom-hud.mjs` in this repo
2. Click the **Raw** button (or **Download**)
3. Save the file to:
   - **macOS / Linux:** `~/.claude/hud/custom-hud.mjs`
   - **Windows:** `C:\Users\<your-username>\.claude\hud\custom-hud.mjs`
4. Create the `hud` folder first if it doesn't exist

</details>

### Step 2 — Tell Claude Code to use it

You need to add one setting to your Claude Code settings file.

**Where is the settings file?**

| OS | Path |
|----|------|
| macOS / Linux | `~/.claude/settings.json` |
| Windows | `C:\Users\<your-username>\.claude\settings.json` |

> If this file doesn't exist yet, just create it.

**Add this to your settings file:**

```json
{
  "env": {
    "CLAUDE_CODE_STATUSLINE_COMMAND": "node ~/.claude/hud/custom-hud.mjs"
  }
}
```

> **Windows users:** Replace the path with your full Windows path:
> ```json
> "CLAUDE_CODE_STATUSLINE_COMMAND": "node C:\\Users\\YourName\\.claude\\hud\\custom-hud.mjs"
> ```

If your settings file already has content, just add the `CLAUDE_CODE_STATUSLINE_COMMAND` line inside the existing `"env"` section. Don't replace what's already there.

<details>
<summary>Example: merging with existing settings</summary>

If your `settings.json` currently looks like this:

```json
{
  "env": {
    "SOME_OTHER_SETTING": "value"
  }
}
```

Add the new line with a comma:

```json
{
  "env": {
    "SOME_OTHER_SETTING": "value",
    "CLAUDE_CODE_STATUSLINE_COMMAND": "node ~/.claude/hud/custom-hud.mjs"
  }
}
```

</details>

### Step 3 — Restart Claude Code

Close and reopen Claude Code. The HUD will appear at the bottom of your terminal automatically.

> **Tip:** You can also ask Claude Code to do Step 2 for you. Just tell it:
> *"Set my statusline command to `node ~/.claude/hud/custom-hud.mjs`"*

### Using with oh-my-claudecode

If you already use [oh-my-claudecode](https://github.com/nicobailon/oh-my-claudecode), the included `omc-hud.mjs` wrapper auto-discovers the OMC plugin — no extra setup needed.

## Requirements

- **Node.js 18 or newer** — you already have this if Claude Code is installed
- **Claude Code** with an active login — the HUD uses your existing session for rate limit data

<details>
<summary><strong>Technical Details</strong></summary>

### How It Works

Claude Code sends JSON data to the statusline command every time the display refreshes. The HUD reads that data and:

1. Calculates context window usage from token counts
2. Fetches your rate limits from the Anthropic API (cached for 60 seconds)
3. Reads the session transcript to find running agents and todo progress
4. Checks npm for the latest Claude Code version (cached for 1 hour)
5. Renders everything as a color-coded status line

All network requests happen simultaneously so the HUD stays fast.

### Agent Tracking

When agents are running, they appear in a tree view below the main status line:

- A letter badge shows the model: **O** (Opus), **s** (Sonnet), **h** (Haiku)
- Elapsed time since the agent started
- The agent type and a short description
- Agents older than 30 minutes are automatically hidden
- Up to 100 agents are tracked per session

### Caching

| Data | Refreshes every | Stored at |
|------|-----------------|-----------|
| Rate limits | 60 seconds | `~/.claude/hud/.usage-cache.json` |
| CC version | 1 hour | `~/.claude/hud/.version-cache.json` |

</details>

## License

MIT
