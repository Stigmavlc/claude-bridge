# Claude Bridge

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Claude Code plugin](https://img.shields.io/badge/Claude_Code-plugin-7C3AED)](https://code.claude.com/docs/en/plugins)
![Version](https://img.shields.io/badge/version-0.1.0-blue)
[![GitHub stars](https://img.shields.io/github/stars/Stigmavlc/claude-bridge?style=social)](https://github.com/Stigmavlc/claude-bridge/stargazers)

*Let Claude Code do your web tasks for you, instead of stopping to ask you to do them yourself.*

Unlike browser automation that opens a fresh, logged-out browser, Claude Bridge uses your own signed-in Chrome, so there is no re-login and nothing to copy and paste.

<!-- demo GIF goes here once recorded and approved -->

## Quick start

```text
/plugin marketplace add Stigmavlc/claude-bridge
/plugin install claude-bridge@claude-bridge
```

Use it to:
- Log into a dashboard and check or change a setting (ads, hosting, DNS, billing, analytics, a CMS).
- Fill in and submit a web form.
- Look something up on a public site and report back.
- Click through any multi-step web flow you would otherwise do by hand.

(Full requirements and the optional logged-in setup are in [Install](#how-to-install-it) below.)

## What it is

Normally, when Claude Code needs to do something on a website, it can't reach it. It pauses and asks you to go and do it by hand: log into a dashboard, change a setting, look something up, fill in a form.

Claude Bridge changes that. Once it's installed, Claude Code opens a browser, does the steps itself, and tells you what happened, all without you leaving your terminal. It works in two ways:

- On sites you're already logged into (your own dashboards, email, hosting, ads, and so on), it uses your real Chrome browser, so it's signed in as you.
- On public websites that don't need a login, it uses a separate browser that runs quietly in the background.

You don't have to pick which one. Claude works it out from what you ask.

## What you'll need

You only need the pieces for the way you want to use it.

**To use it on your own logged-in sites:**

- Claude Code, version 2.0.73 or newer. [Install it here.](https://code.claude.com/docs/en/setup)
- A paid Claude plan: Pro, Max, Team, or Enterprise. [See the plans.](https://claude.com/pricing) (If you only use Claude through a cloud provider such as AWS Bedrock or Google Vertex, you'd need a separate regular Claude account for this part.)
- The Claude in Chrome extension, version 1.0.36 or newer, in Google Chrome or Microsoft Edge. [Get it from the Chrome Web Store.](https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn) [Here's the setup guide.](https://support.claude.com/en/articles/12012173-getting-started-with-claude-in-chrome)

**To use it on public sites (no login):** just one of these.

- [Node.js](https://nodejs.org/en/download), version 20 or newer. A small browser tool then installs itself automatically the first time it's needed.
- Or the [Playwright](https://playwright.dev) plugin, if you already prefer that.

Not sure whether you already have Claude Code or Node.js? In your terminal, type `claude --version` and `node --version`. If you get a version number back, you have it.

## How to install it

### Step 1: Add the plugin

Open Claude Code and type these two lines, one at a time:

```text
/plugin marketplace add Stigmavlc/claude-bridge
/plugin install claude-bridge@claude-bridge
```

That's the whole install. It adds the part that decides when a browser is needed, and it includes the background browser for public sites.

### Step 2: Turn on access to your logged-in sites

This step is only for when you want Claude to work on sites you're signed into. First make sure you've installed the Claude in Chrome extension (from the list above). Then type:

```text
/chrome
```

Choose "Enabled by default" so you don't have to turn it on every time. The very first time you do this, close Claude Code and open it again, so the connection can finish setting up.

### Step 3 (optional): Stop the permission pop-ups

By default, Claude asks for your okay before each browser action. That's a safe default. If you'd rather it just get on with the task, you can tell it that browser tools are pre-approved. Open the file `~/.claude/settings.json` and add this (or merge it into what's already there):

```json
{
  "permissions": {
    "allow": [
      "mcp__chrome-devtools",
      "mcp__claude-in-chrome",
      "mcp__plugin_playwright_playwright"
    ]
  }
}
```

## How to use it

Just ask, in plain language. For example:

```text
Log into my Vercel dashboard and tell me whether the last deploy worked.
```

```text
Change the daily budget on my Google Ads campaign to 40 dollars.
```

```text
Look up the current pricing on example.com and summarize the tiers for me.
```

Claude decides whether it needs your signed-in browser or the background one, does the work, and reports back in the terminal. If it runs into a login screen or one of those "I'm not a robot" checks, it stops and lets you handle that bit, then carries on.

## Is it safe?

Claude Bridge is built to finish a web task on its own without checking in at every click. It also keeps some firm limits:

- It never types your passwords, one-time codes, or card numbers. That part is always left to you.
- It stops and asks before doing anything it can't undo, such as deleting something, making a payment, or posting in public.
- It treats the contents of a web page as information to read, never as instructions to obey. So if a page hides text telling Claude to do something, Claude ignores it. This protects you from a common trick used against AI assistants.
- It stays away from sensitive categories like banking and adult sites unless you tell it to go there on purpose.

If you'd like it to double-check with you before anything risky, just say so: "confirm risky browser actions before doing them." And for anything involving money, it's wise to watch while it works.

## A note on updates and security

The background browser tool is pinned to one specific, known version (`chrome-devtools-mcp@1.1.1`) rather than always grabbing the newest release. This means everyone gets the same reviewed version, and a future update can't run on your machine until someone deliberately updates it. To update later, change that version number in `.mcp.json` after checking the new release.

---

<details>
<summary><b>For developers and maintainers</b>: how it works, the file layout, publishing your own copy, and troubleshooting</summary>

### How it works under the hood

There is no public way for one AI to send prompts into the Claude browser extension and read its replies back. Claude Bridge works the other way around: Claude Code is in charge, and the browser is a tool it controls. Claude Code connects to a browser, calls browser actions (go to a page, read it, click, type), sees each result in the terminal, and decides the next step on its own.

A skill (`skills/browser-bridge/SKILL.md`) holds the logic. For each web task it first decides whether the task needs your signed-in session, then picks an engine:

| Task | Path | Engine |
|---|---|---|
| Touches your account or dashboard (ads, hosting, DNS, analytics, billing, a CMS, social, posting as you) | Login path | Your real, signed-in Chrome through the Claude in Chrome extension (`mcp__claude-in-chrome__*`) |
| Public or anonymous (docs, prices, public pages, status checks, forms that need no account) | No-login path | A background browser: Chrome DevTools (`mcp__chrome-devtools__*`), or Playwright as an alternative |

If a no-login task unexpectedly hits a sign-in wall, it moves up to the login path rather than guessing. On the login path your real browser is already signed in, so it never needs your username or password.

### Other ways to install

The plugin install in Step 1 is the easy path. Two alternatives:

Install the skill only, without the plugin (it then uses whatever browser engines you already have connected):

```bash
mkdir -p ~/.claude/skills/browser-bridge
cp skills/browser-bridge/SKILL.md ~/.claude/skills/browser-bridge/SKILL.md
```

Load the whole folder for development, without installing it:

```bash
claude --plugin-dir "/path/to/Claude Bridge"
```

### Files

```text
claude-bridge/
├── .claude-plugin/
│   ├── plugin.json          # plugin manifest
│   └── marketplace.json     # one-plugin marketplace (github source)
├── .mcp.json                # bundles the background Chrome DevTools browser
├── skills/
│   └── browser-bridge/
│       └── SKILL.md         # the routing and browser-loop logic (the core)
├── agents/
│   └── browser-runner.md    # optional helper for long browsing sessions
├── docs/                    # design notes and a live-test screenshot
├── CLAUDE.md                # notes for Claude Code when editing this repo
├── Project_Master_and_changelog.md
├── Report.md
└── README.md
```

### Publishing your own copy

Claude Bridge is both a plugin and a one-plugin marketplace, so sharing it is "publish the folder, others add it." This uses git and the [GitHub CLI](https://cli.github.com) (`gh`):

```bash
cd "/path/to/Claude Bridge"
git init
git add .
git commit -m "Claude Bridge v0.1.0"
gh repo create claude-bridge --public --source=. --remote=origin --push
```

Other people then install it with the two commands from Step 1, pointing at your GitHub username instead of `Stigmavlc`, and turn on the login path with `/chrome`.

### Troubleshooting

- **Browser tools for your logged-in sites aren't showing up:** run `/chrome` and choose "Reconnect." Check that the extension is installed at `chrome://extensions` and is version 1.0.36 or newer, and that Claude Code is 2.0.73 or newer. ([More detail here.](https://code.claude.com/docs/en/chrome)) These tools only appear after a fresh start of Claude Code, so restart it if you just enabled the integration.
- **The skill doesn't seem to trigger:** confirm `~/.claude/skills/browser-bridge/SKILL.md` exists, or that the plugin is enabled (run `/plugin`).
- **The background browser is missing:** make sure Node.js 20 or newer is installed, or enable the Playwright plugin.

</details>

## License

MIT. © 2026 Ivan Aguilar
