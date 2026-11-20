# AI Employees

A live "office" for your Claude scheduled tasks and Claude Code sessions. Each task or session is a person at a desk, and their mood shows what's happening:

| At their desk | Means |
|---|---|
| Crying | The last run failed |
| Walks over, waving | Waiting on your reply or OK |
| Typing, with a chart | Running right now |
| Thumbs up | Finished in the last day |
| Coffee break | Waiting for the next run, or paused |

Tap a desk to **Run now**, **Pause/Resume**, **Rename** it (on this page only), move it into a department, or open its last run. A **Manager** chat answers questions like "Did anything fail today?" using a snapshot of your tasks.

Built by [Gold St Solutions](https://goldstsolutions.com).

<!-- Add a screenshot or GIF here: the page does not run on GitHub. -->

---

## Before you start: this is not a normal web page

`index.html` only works when it's **published as an artifact on claude.ai**. It gets its live data from the claude.ai artifact runtime (`window.claude.use(...)`), which only exists there.

If you open the file from GitHub, GitHub Pages or your desktop, it loads but shows **"Live status is off here"** with no desks. That's expected.

## Setup

You need:

- A Claude plan with **Claude Code** and **scheduled tasks**.
- The **Claude Code Remote** connector connected in claude.ai (Settings → Connectors). This is how the page reads, runs and pauses your scheduled tasks.
- *Optional, Mac only:* the **Claude desktop app** with a **Filesystem** connection that can read `~/.claude/projects`. This adds desks for the Claude Code sessions on your computer.

### 1. Publish your own copy

Each person publishes their own copy. A copy someone else published reads *their* account, not yours.

The easiest way is to ask Claude Code (or Claude in the desktop app):

> Publish `index.html` as an artifact with the capabilities in `capabilities.json`.

The capabilities it needs (also in [`capabilities.json`](capabilities.json)):

```json
{
  "mcp": {
    "servers": [
      { "server": "Claude Code Remote", "tools": ["list_triggers", "fire_trigger", "update_trigger"] },
      { "server": "host:Filesystem", "tools": ["list_directory", "get_file_info"] }
    ]
  },
  "db": {},
  "user": {},
  "sample": {}
}
```

| Capability | Used for | Without it |
|---|---|---|
| `Claude Code Remote` → `list_triggers` | Reading your scheduled tasks | No task desks |
| `Claude Code Remote` → `fire_trigger` | **Run now** and **Send & run** | Those buttons fail |
| `Claude Code Remote` → `update_trigger` | **Pause / Resume** | Pause fails |
| `host:Filesystem` → `list_directory`, `get_file_info` | Finding local Claude Code sessions | No session desks; tasks still show |
| `db` + `user` | Saving your departments and desk names, privately under your user id | Layout is kept in this browser only |
| `sample` | The Manager chat | Chat is hidden |

If you don't want a feature, remove that line before publishing.

### 2. Open it and allow access

Open the artifact on claude.ai or in the Claude desktop app. The first time, it asks you to OK each connector. If you said no by mistake, use the artifact's **Permissions** menu or click **Allow access** on the banner.

### 3. Show local sessions (optional, Mac)

Session desks only appear when:

1. the page is open **in the Claude desktop app** (not a browser tab), and
2. your **Filesystem** connection is allowed to read `~/.claude/projects`.

If either is missing, the page shows a short note and keeps showing your scheduled tasks. After changing the Filesystem folders, click **Refresh**.

---

## What it does with your data

- **Everything stays in your own account.** The page calls your connectors with your own sign-in. Nothing in this repo contains anyone's data.
- **Your departments and desk names** are stored in the artifact's database under your user id. Nobody else who opens the artifact can read them.
- **Browser storage** remembers small things (whether details are open, which desks you've already seen, your Manager chat history) in your browser only.
- **The Manager chat sends Claude a snapshot** of your tasks: names, schedules, last/next run, and **the first 450 characters of each task's instructions**. If your task prompts contain sensitive details, keep that in mind or remove `sample`.
- **Session desks** only read folder names and file dates in `~/.claude/projects`. They never open the session files.

## Heads-up: the buttons are real

**Run now**, **Send & run** and **Pause** act on your real scheduled tasks right away, with no "are you sure?" step:

- **Run now** starts the task immediately, with everything it normally does (emails, Slack messages, docs...).
- **Send & run** adds your note to that run's instructions and starts it.
- **Pause** stops the task's schedule until you resume it.

**Rename** only changes the name on this page. The task keeps its real name.

## Known limitations

- **It relies on details that aren't official APIs.** It reads plain-text output from the Filesystem connector (`[DIR]`, `[FILE]`, `modified:`), expects session files named `<uuid>.jsonl`, matches run statuses like `ROUTINE_RUN_STATUS_*`, and builds run links as `claude.ai/code/session_...`. If Anthropic changes any of these, parts of the page may stop working without an error.
- **Local session states are guesses** based on file timestamps: active in the last 5 minutes = working, 5 minutes to 2 hours = "probably waiting on you", older = quiet. Only sessions from the last 24 hours are shown (max 8).
- **Schedules without a timezone** are shown as UTC, converted to your local time.
- **Up to 100 scheduled tasks** are read. The list refreshes every 60 seconds; local sessions every 5 minutes while the tab is visible.
- **Tasks that are turned off** are hidden, except ones you pause from this page (so you can resume them).

## Troubleshooting

| You see | Do this |
|---|---|
| "Live status is off here" | You opened the raw file. Publish it as an artifact (step 1). |
| "Not connected" / "can't reach your scheduled tasks" | Connect **Claude Code Remote** in claude.ai, then reload. |
| "This page needs your OK..." | Click **Allow access**, or use the artifact's Permissions menu. |
| "Reading your scheduled tasks is turned off for this page" | The `Claude Code Remote` line is missing from the capabilities. Republish with it. |
| No session desks | Open the page in the Claude desktop app and let Filesystem read `~/.claude/projects`, then **Refresh**. |
| No Manager chat | `sample` isn't declared, or you declined it. |

## Customising

- **Branding:** the header logo is in the `<p class="brand">` line and the boss's name is the `BRAND` constant in the script. Swap them for your own.
- **Colors:** all colors are CSS variables at the top of the `<style>` block, with a dark-mode set below them.
- **Folder labels:** `folderLabel()` turns `-Users-you-Desktop-Projects-Acme` into "Projects · Acme". Change it to fit how you organise folders.

## License

[MIT](LICENSE): free to use, change and share, as long as the copyright notice stays in.

_Last tested: November 2026._
