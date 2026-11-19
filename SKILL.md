---
name: generating-test-case-artifact
description: Use when the user asks for test cases, a test plan, a QA checklist, or a test matrix based on a project's code, plans, .md files or previously created artifacts, to be delivered as an artifact.
---

# Generating a Test Case Artifact

## Overview

Read everything the project contains (code, plans, markdown, earlier artifacts), then publish ONE artifact: a sectioned table of test cases with a result and a screenshot slot per case. Every test case must trace to something found in the project. Do not invent features.

## Step 1: Gather evidence

Run these in parallel; skip nothing that exists:

| Source | Where to look |
|---|---|
| Code | Routes/controllers, auth/middleware, models/schemas, validators, UI components/pages, error handlers, config |
| Markdown | `README*`, `docs/`, `CLAUDE.md`, `*.md` anywhere (exclude `node_modules`) |
| Plans / specs | `docs/plans`, `plans/`, `specs/`, `~/.claude/plans/`, `ONBOARDING.md`, TODO/PRD files |
| Artifacts | `Artifact` tool `action: "list"`, then `read` those relevant to this project |

For large codebases, delegate reading to an Explore subagent and keep only the conclusions.

If nothing is found, say so and ask the user what to test. Do not fabricate.

## Step 2: Derive sections

Sections come from the evidence, not a fixed list. Candidate catalogue (include one only if the project has related behavior):

Authentication · Authorization/roles · UI and UX · Forms and validation · Duplicate detection · Data integrity/CRUD · API contracts · Error handling · Search/filter/pagination · File upload · Notifications · Integrations · Performance · Accessibility · Security · Responsive/cross-browser

Add project-specific sections the catalogue lacks. Within each section cover: happy path, negative path, edge/boundary, and state/permission variants.

## Step 3: Write cases

- **ID**: section prefix + 3 digits (`AUTH-001`, `DUP-004`, `UX-012`). Unique, never reused.
- **Description**: one sentence, then `Expected:` outcome, then `Source:` file or doc it came from (e.g. `src/auth/login.ts`, `plan.md`).
- **Steps**: numbered, concrete (real URLs, field names, sample data from the code).
- **Result**: starts as `Not run`. Allowed: Not run / Pass / Fail / Blocked, plus a notes field.
- **Result screenshot**: empty slot the viewer fills in.

## Step 4: Build the artifact

1. Call `Artifact` with `action: "quickstart"`, `intent: "other"` first.
2. **REQUIRED SUB-SKILLS:** load `artifact-design` and `artifact-capabilities` before writing the page.
3. Declare the capabilities needed so results and screenshots persist and are shared (shared state for Result and notes; file/asset storage for screenshots). Browser storage alone is not acceptable for results.
4. Page layout:
   - Header: project name, date, totals (cases, passed, failed, not run).
   - One section per area with a heading, case count and its own pass/fail tally; sections are visually separated and collapsible.
   - Each section is a table with columns exactly: **Test Case ID | Test Case Description | Steps | Result | Result Screenshot**.
   - Result cell: status selector + notes. Screenshot cell: paste/upload control with thumbnail preview and click-to-enlarge.
   - Filter by status; works at phone width (table scrolls inside its container, not the page).
5. Publish, return the link plus a one-line summary of sections and case counts.

## Step 5: Verify the page renders its cases (after EVERY publish)

A single syntax error in the inline script leaves the page with no test cases, and nothing flags it. This applies to the first publish and to every later edit, including one-line text fixes.

1. Extract the inline `<script>` from the HTML file and run `node --check` on it. It must print no error. Do not skip this for "small" edits.
2. Count the cases in the data array and confirm it matches the number promised to the user.
3. Do this before telling the user the link is ready. If the check fails, fix the file and republish, then re-run the check.

Escaping rules when editing case text inside JS strings: never put a bare `"` inside a double-quoted string (use `\"`, single quotes or reword), and never put a backtick inside a template literal. Prefer rewording over adding quotes.

```bash
python3 - <<'E'
import re
h = open('PAGE.html').read()
open('chk.js', 'w').write(re.search(r'<script>(.*)</script>', h, re.S).group(1))
E
node --check chk.js && echo SYNTAX_OK; rm chk.js
```

## Common Mistakes

| Mistake | Fix |
|---|---|
| Generic cases ("test login works") | Use real routes, fields and data from the code |
| Sections not backed by code | Drop them |
| Cases with no `Source:` | Add it or delete the case |
| Results kept in localStorage | Use a shared-state capability |
| Extra or renamed columns | Keep the five requested columns |
| Only happy paths | Add negative, boundary and permission cases per section |
| Editing case text and republishing without re-checking | Re-run the Step 5 syntax check after every edit; an unescaped quote blanks the whole page |
