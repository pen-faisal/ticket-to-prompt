---
name: "ticket-to-prompt"
description: "Turn a ticket (Jira, Linear, GitHub or pasted) into a codebase-grounded implementation prompt, with a preview of the finished task to approve before any code is written."
---

# Ticket to Prompt

Turn a ticket into a clear, codebase-aware prompt, and show the user what the finished task will look like **before** any code is written.

The goal: no wrong builds, no missed requirements, no surprises.

## When to use

- The user shares a ticket, issue or task (pasted text, a ticket ID or a link) and wants a prompt, a plan or an implementation.
- The user says things like "turn this ticket into a prompt", "prepare this for Claude Code" or "what would this look like when it's done?"

## What you need

1. **The ticket.** If a connector for the ticket tool (Jira, Linear, GitHub) is available, fetch the ticket with its comments and linked issues. Otherwise ask the user to paste it.
2. **The codebase.** A repo or folder you can read. If none is available, say so and offer to work from the ticket alone, clearly marking every file path as a guess.

Never invent file paths, functions or conventions. If you didn't find it in the code, don't state it as fact.

## Prompt modes

Pick a mode from the ticket's size, or use the one the user asks for:

- **Short:** small fixes and chores. Goal, files, requirements, done-when. A few lines each.
- **Detailed:** features and anything risky. The full structure in Step 7.
- **Plan-first:** large or unclear work. Write only the step-by-step plan and preview; the implementation prompt comes after approval.

Say which mode you chose and why in one line. The user can switch at any time.

## Workflow

### Step 1: Read the ticket

Collect the title, description, acceptance criteria, comments, linked issues and attachments.

### Step 2: Classify it

Decide the type, because each type needs a different prompt:

- **Bug:** include steps to reproduce, expected vs. actual behavior, and "find the root cause before fixing".
- **Feature:** focus on where it fits, what it reuses and how it's tested.
- **Refactor:** behavior must not change; existing tests must still pass.
- **Chore:** keep the prompt short.

Rewrite vague acceptance criteria as specific, testable statements.

**Estimate the size:** Small, Medium or Large, with a one-line reason (for example, "Medium: new endpoint plus a UI change, follows an existing pattern"). Use it to pick the prompt mode.

**Split big tickets.** If the ticket is Large or covers several separate changes, break it into smaller parts that can each be built and checked on their own, in order (for example: 1. database migration, 2. API, 3. UI, 4. tests). Each part gets its own preview and prompt. Show the split to the user first and let them adjust it.

### Step 3: Explore the codebase

Find:

- **Affected files:** the files and functions the change touches, and their callers.
- **A similar feature:** existing code that does something close to this. This is the most valuable find, because the new work should follow its pattern.
- **Reusable code:** helpers, hooks, services or components that already exist, so nothing is rewritten.
- **Existing tests** for the affected areas.
- **Risk areas:** auth, permissions, payments, migrations, public APIs. Flag these.
- **Git history:** if git is available, check recent commits and past changes to the affected files (`git log`, `git blame`) to learn why the code is built the way it is. Note anything relevant, such as a past bug fix in the same area or a pattern that was deliberately changed. Never undo a deliberate past fix without flagging it.

### Step 4: Learn the conventions

Read the README, CLAUDE.md, contributing guides, lint and formatter config, and the test setup. Note the language, frameworks, validation library, test tools, naming and folder patterns.

**Team rules file.** If the repo has a `PROMPT_RULES.md` (or the user points to another rules file), read it and follow it in every preview and prompt. Its rules take priority over your own defaults. Typical contents: required test types, banned libraries, naming rules, review requirements, the default prompt mode.

### Step 5: Build the preview

This is the key step. Show the user what the finished task will look like before anything is built. Keep it to four parts:

1. **Mockup with state gallery:** for anything visual, show the screen after the change. Draw it inline or build a quick page. Show every relevant state side by side: normal, empty, loading, success, error, no permission. For backend-only work, skip the mockup and say so.
2. **Files to change, with a draft diff:** every file to add or edit, marked **new** or **modified**, with one line on what changes in each. Under each file, sketch the key changes as a short draft diff: new function signatures, routes, types and the core logic, without filling in every detail. This lets reviewers comment on the approach before code exists.
3. **Sample result:** a realistic example of what the feature produces: an API response, a few rows of an export, a message the user will see, or an error. Use made-up data and label it as an example.
4. **Checklist:** each acceptance criterion, with the change and the test that will satisfy it.

Add a short **Open questions** list for anything vague or conflicting in the ticket, and any risk flags from Step 3.

Keep the preview short and easy to scan. It should take less than a minute to review. Put the size estimate and chosen mode at the top.

**Shareable preview link.** When others need to review the preview (a product owner, designer, reviewer or client), or the user asks, publish it as a page they can open and comment on, and give the user the link. Otherwise show it in the conversation.

### Step 6: Ask for approval

End the preview by asking the user to choose:

- **Approve:** build it as shown.
- **Edit:** tell you what to change. Update the preview and ask again.

Do not write the final prompt or start implementation until the user approves. If the user is not available to answer, stop at the preview and say what is waiting on them.

**Post to the ticket.** If a connector for the ticket tool is available, offer to post the preview summary and open questions as a comment on the ticket, so the ticket author can answer there. Only post after the user says yes. When answers come back, update the preview and ask for approval again.

Record what was approved and when, so Step 8 can compare against it.

### Step 7: Write the final prompt

Once approved, write the prompt using this structure:

```
## Goal
One or two lines: what the ticket wants and why. Include the ticket ID and type.

## Context
- Exact file paths and functions involved, and how they connect
- The similar feature to follow, by path
- Reusable helpers to use

## Requirements
Numbered, specific, testable. Taken from the approved checklist.

## Constraints
- Patterns and conventions to follow
- What not to change
- Libraries not to add

## Approved preview
The approved mockup description, file list and sample result: the target to match.

## Done when
- Tests to add or update, by name
- Lint, type check and test commands that must pass

## Open questions
Anything still unanswered. If there are none, say "None".
```

Match the prompt's length to the chosen mode.

### Step 7b: Branch and PR text

Alongside the prompt, suggest:

- **Branch name:** following the repo's existing pattern if there is one, otherwise `type/ticket-id-short-description` (for example `feat/ord-214-orders-csv-export`).
- **PR description draft:** the ticket link, a summary of the change, the approved checklist, how to test it, and any risk flags.

### Step 8: Compare after implementation (if the work is done in this session)

After implementation, put the approved preview next to the real result:

- Each checklist item: done or not done.
- Files changed vs. the planned list. Point out any extra or missing files.
- Anything that differs from the mockup or sample result.

**Drift alert.** Clearly warn when the implementation went beyond the approved plan: extra files changed, a different approach taken, new libraries added, or features the ticket didn't ask for. List each one and ask whether to keep it or roll it back.

## Learning from feedback

After each ticket, ask in one line whether the prompt missed anything (wrong file, missed requirement, convention ignored). If the user gives feedback:

- Add a short, specific lesson to `PROMPT_RULES.md` under a "Lessons" heading (for example, "Always check `src/lib/flags.ts` for feature flags before adding UI"). Ask before creating the file or editing it for the first time.
- Read these lessons in Step 4 on every future ticket.

Keep lessons short and remove ones that no longer apply.

## Quality rules

- **Ground everything in the code.** Every path and function named must exist, unless it's marked as new.
- **Flag, don't guess.** If the ticket is unclear or conflicts with the code (for example, it mentions a function that doesn't exist), add it to Open questions.
- **Prefer reuse.** Always point to existing patterns and helpers.
- **Keep it readable.** Short sentences, clear headings, no filler.
- **Respect scope.** Don't add features the ticket didn't ask for. Suggest them separately if useful.

## Example (short)

Ticket: "ORD-214: Add CSV export to the Orders list. Admin and ops only, respects filters, max 10,000 rows."

Preview (Medium, Detailed mode):

- **Mockup:** Orders page with an "Export CSV" button next to Filters; a loading spinner while exporting; the button hidden for the viewer role.
- **Files:** `server/routes/orders.ts` (modified: add export route), `server/services/export/ordersCsv.ts` (new: follows `invoiceCsv.ts`), `src/pages/orders/OrdersPage.tsx` (modified: add button), `server/routes/__tests__/ordersExport.test.ts` (new).
- **Sample result:** `orders-2026-11-17.csv` with columns order_id, created_at, customer_name, total, status.
- **Checklist:** button on page ✓ filters respected ✓ five columns ✓ row cap with error ✓ admin and ops only ✓
- **Open questions:** Dates in UTC or the user's timezone? Hard cap at 10,000, or a background job for bigger exports?
- **Branch:** `feat/ord-214-orders-csv-export`

"Approve, or tell me what to change? I can also post these questions to the ticket."