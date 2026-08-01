# Starting tomorrow on the Mac, in VS Code

Written 2026-08-01, at the end of a mobile session. Everything from that session is
pushed — nothing is stranded on the phone.

## Where things stand

| Repo | Branch | What's on it |
|---|---|---|
| `Jenstho/claude-test-project` | `claude/jens-thomas-page-analysis-o8mxug` | `plan.md` — the full analysis of jens-thoemmes.com (4 drafts), plus this file |
| `Jenstho/jens-thoemmes-portfolio` | `main` | The site itself. **Untouched** — analysed read-only, no changes made |

No pull request was opened. Nothing is half-finished in a working directory: the mobile
session's container is disposable and everything worth keeping is on GitHub.

---

## 1. Install the extension (5 minutes)

**Prerequisite:** VS Code 1.94.0 or later (check with **Help → About**).

Open VS Code, press `Cmd+Shift+X` for the Extensions view, search for **Claude Code**,
click **Install**. Or use the direct link: [Install for VS
Code](vscode:extension/anthropic.claude-code).

If the extension doesn't appear afterwards, run **Developer: Reload Window** from the
Command Palette (`Cmd+Shift+P`).

### Also install the CLI — worth doing while you're there

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

The extension bundles its own private copy of Claude for the chat panel, but it does
**not** put `claude` on your PATH. The standalone install gives you the `claude` command
in any terminal, which you need for a few things the panel can't do — `claude mcp add`
being the one that matters here (see step 4). Verify with:

```bash
claude --version
```

Homebrew works too, if you prefer: `brew install --cask claude-code`. Note that Homebrew
installs don't auto-update, while the `curl` install does.

## 2. Sign in

Open the Claude panel — click **✱ Claude Code** in the status bar (bottom-right), or the
Spark icon in the top-right of the editor when a file is open. A sign-in screen appears
on first launch; click **Sign in** and finish in the browser.

Sign in with your **Claude.ai subscription** account, not a Console account. Step 3
depends on it.

## 3. Resume this conversation — the useful part

You don't have to re-explain anything. This session was started from a GitHub repo, so it
shows up in VS Code:

1. Click **Session history** at the top of the Claude panel
2. Switch to the **Web** tab (two tabs: Local and Web)
3. Find this session and click it

It downloads the whole conversation and continues locally — the analysis, the findings,
the visual check, all of it. Note that changes then stay local and don't sync back to
claude.ai, which is fine; from that point on you're working on the Mac.

If you'd rather start clean, `plan.md` stands on its own — point a new conversation at it.

## 4. Get the repos on disk

```bash
# wherever you keep projects
cd ~/Projects

git clone https://github.com/Jenstho/jens-thoemmes-portfolio.git
git clone https://github.com/Jenstho/claude-test-project.git

# the analysis lives on a branch
cd claude-test-project
git checkout claude/jens-thomas-page-analysis-o8mxug
```

Then open the **portfolio** folder in VS Code (`File → Open Folder`), since that's where
the actual work happens. `code ~/Projects/jens-thoemmes-portfolio` from a terminal does
the same thing.

A real advantage over the mobile session: the site is a single self-contained
`index.html`, so you can just open it in Safari or Chrome and see your changes
immediately. No build, no server. The session on the phone had no network access to the
live site at all.

### Optional: GitHub tools in the panel

If you want Claude to read issues and open PRs from inside VS Code, add the GitHub MCP
server once, from the integrated terminal (`` Cmd+` ``):

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer YOUR_GITHUB_PAT"
```

Create the token at <https://github.com/settings/personal-access-tokens>. Check it worked
by typing `/mcp` in the chat panel — the server should read `connected`. Skip this if
you'd rather just use `git` in the terminal; nothing below needs it.

## 5. What to pick up first

From `plan.md`, in the order that pays off. The first item is one piece of work:

1. **P0 — the HAL import + pre-render.** Import from HAL at build time into
   `publications.json`, generate the publication HTML, delete the 2026+ button. This
   fixes the crawlability problem as a side effect and is the whole reason the site
   currently ranks 7th on its own name.
2. **The mobile bug (V-1).** One line: `backdrop-filter` on `.nav` is making the mobile
   control panel land on top of your name. Nice first win — small, visible, and you can
   confirm it in the browser in seconds.
3. **Delete the two data buttons (V-2 / C5).** Biggest visual improvement per keystroke.
4. **The PNG social card (B3).** Every LinkedIn and X share currently previews with no
   image because `og:image` is an SVG.
5. **CERTOP → UTOPI (B1)** in five places, one of which feeds Google Scholar.

Open questions Q1–Q9 at the bottom of `plan.md` are still waiting on you. Q7 (HAL author
ID) and Q8 (does HAL have 2026 records yet) block part of item 1 — worth checking on
hal.science before you start.

## A few things that make the desktop nicer than the phone

- **Plan mode.** Click the mode indicator under the prompt box and pick **Plan**. Claude
  writes what it intends to do as a Markdown document you can comment on inline before
  any edit happens. Good fit for a site you care about.
- **Diffs before they land.** Every file edit shows side by side, and you can edit the
  proposal directly in the diff before accepting.
- **`@`-mentions.** Type `@index` to pull the file in; select lines in the editor and
  press `Option+K` to reference exactly those (`@index.html#2614-2778`).
- **Rewind.** Hover any message for checkpoints — undo Claude's file changes back to that
  point without losing the conversation.
- **`Cmd+Esc`** toggles focus between the editor and Claude. On macOS Tahoe or later this
  is taken by the system Game Overlay: **System Settings → Keyboard → Keyboard Shortcuts
  → Game Controllers**, clear **Game Overlay**.
- **`/usage`** shows where you are against your plan limits.

## If something goes wrong

- `claude doctor` — read-only diagnostics for the install and settings.
- Spark icon missing? You need a file open, not just a folder. Or use the status bar.
- Extension not responding? Run `claude` in the integrated terminal — the CLI usually
  gives a clearer error.
- Full setup reference: <https://code.claude.com/docs/en/setup> and
  <https://code.claude.com/docs/en/vs-code>
