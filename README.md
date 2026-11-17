# Ticket to Prompt

A Claude skill that turns a ticket (Jira, Linear, GitHub or pasted text) into a clear, codebase-aware implementation prompt, and shows a **preview of the finished task** to approve before any code is written.

## What it does

1. Reads the ticket and works out its type and size
2. Explores your codebase: affected files, a similar feature to follow, reusable code, git history, risk areas
3. Learns your conventions, including your team's `PROMPT_RULES.md`
4. Shows a preview: mockup with all states, files to change with a draft diff, a sample result, and a checklist
5. Waits for you to approve or edit
6. Writes the final prompt, plus a branch name and PR description
7. After implementation, compares the result with the approved preview and flags any drift

## Repository layout

```
.claude-plugin/
  plugin.json          Claude Code plugin details
  marketplace.json     Lets others install it with /plugin
skills/
  ticket-to-prompt/
    SKILL.md           The skill itself
PROMPT_RULES.example.md  Template for your team's rules
```

## Install

### Claude app (web, desktop, mobile)

1. Zip the `skills/ticket-to-prompt` folder (the zip should contain the `ticket-to-prompt` folder with `SKILL.md` inside).
2. In Claude, go to **Customize → Skills → + → Create skill → Upload a skill**, and upload the zip.
3. Turn the skill on.

On Team and Enterprise plans, use **Share** on the skill to give it to colleagues or the whole organization.

### Claude Code: as a plugin

```
/plugin marketplace add YOUR-GITHUB-USER/ticket-to-prompt
/plugin install ticket-to-prompt@ticket-to-prompt
```

### Claude Code: inside one project

Copy `skills/ticket-to-prompt` into your project's `.claude/skills/` folder and commit it. Everyone who clones the project gets the skill.

## Use

Share a ticket and ask, for example:

- "Turn ORD-214 into a prompt"
- "Use ticket-to-prompt on this ticket" (then paste it)
- "Show me what this ticket will look like when it's done"

## Team rules

Copy `PROMPT_RULES.example.md` to your project root as `PROMPT_RULES.md` and edit it. The skill follows it on every ticket and adds lessons to it over time.
