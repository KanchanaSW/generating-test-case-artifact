# generating-test-case-artifact

An agent skill that turns a project into a **test case artifact**: one shared, sectioned page of test cases with a result and a screenshot slot per case.

## What it does

When you ask for test cases, a test plan, a QA checklist or a test matrix, the skill:

1. **Gathers evidence** from the project: code (routes, auth, models, validators, UI), markdown docs, plans/specs, and previously created artifacts. It invents nothing; if there is no evidence it asks what to test.
2. **Derives sections** from what the project actually has (Authentication, Forms and validation, API contracts, Error handling, and so on), covering happy path, negative, edge and permission cases.
3. **Writes cases** with a unique ID (`AUTH-001`), a description with `Expected:` and `Source:`, concrete steps, and a result that starts as `Not run`.
4. **Publishes one artifact** with the columns **Test Case ID | Test Case Description | Steps | Result | Result Screenshot**. It has per-section tallies, collapsible sections, a status filter, and shared storage for results and screenshots, so they persist.
5. **Verifies the page** after every publish (`node --check` on the inline script, and the case count), so a stray quote can't silently blank the page.

Full instructions are in [`SKILL.md`](./SKILL.md).

## Requirements

- Claude Code with the `Artifact` tool and the `artifact-design` / `artifact-capabilities` skills (used for publishing).
- `node` and `python3` on `PATH` (for the syntax check).
- Cursor can load the skill, but publishing the artifact needs Claude Code.

## Install

One command, no clone. It downloads `SKILL.md` from this repo into your skills folder (re-run it to update).

**Claude Code**

```bash
curl -fsSL --create-dirs -o ~/.claude/skills/generating-test-case-artifact/SKILL.md https://raw.githubusercontent.com/KanchanaSW/generating-test-case-artifact/main/SKILL.md
```

**Cursor**

```bash
curl -fsSL --create-dirs -o ~/.cursor/skills/generating-test-case-artifact/SKILL.md https://raw.githubusercontent.com/KanchanaSW/generating-test-case-artifact/main/SKILL.md
```

**Both**

```bash
for t in claude cursor; do curl -fsSL --create-dirs -o ~/.$t/skills/generating-test-case-artifact/SKILL.md https://raw.githubusercontent.com/KanchanaSW/generating-test-case-artifact/main/SKILL.md; done
```

For one project only, replace `~/.claude` / `~/.cursor` with `.claude` / `.cursor` and run it from the project root.

Restart Claude Code / Cursor (or start a new session) so the skill is picked up.

## Usage

Ask in plain language, from inside the project you want tested:

```
Generate test cases for this project as an artifact
Create a QA checklist from the plan in docs/plans
```

Claude Code also lets you invoke it directly: `/generating-test-case-artifact`.

## Uninstall

```bash
rm -rf ~/.claude/skills/generating-test-case-artifact ~/.cursor/skills/generating-test-case-artifact
```
